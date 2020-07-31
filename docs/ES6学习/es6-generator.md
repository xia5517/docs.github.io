## Generator实现

我们从一个简单的Generator使用实例开始，一步步探究Generator的实现原理：
 function关键字与函数名之间有一个星号 "*" 
 ```
function* helloWorldGenerator() {
  yield 'hello';
  yield 'world'; // yield 表达式是暂停执行的标记
  return 'ending';
}

```

我们打印下执行的结果：
```
var hw = helloWorldGenerator(); 
console.log(hw.next()); // {value: "hello", done: false}
console.log(hw.next()); // {value: "world", done: false}
console.log(hw.next()); // {value: "ending", done: true}
console.log(hw.next()); // {value: undefined, done: true}

```

在babel官网上在线转化这段代码，看看ES5环境下是如何实现Generator的：

```
/**
 * 我们就称呼这个版本为简单编译版本吧
 */
var _marked = /*#__PURE__*/ regeneratorRuntime.mark(helloWorldGenerator);

function helloWorldGenerator() {
  return regeneratorRuntime.wrap(
    function helloWorldGenerator$(_context) {
      while (1) {
        switch ((_context.prev = _context.next)) {
          case 0:
            _context.next = 2;
            return "hello";

          case 2:
            _context.next = 4;
            return "world";

          case 4:
            return _context.abrupt("return", "ending");

          case 5:
          case "end":
            return _context.stop();
        }
      }
    },
    _marked,
    this
  );
}
```


generator 函数的核心代码被转换成了 switch 块， 这对于我们探索其内部原理提供了很有价值的线索。我们可以把 generator 想象成一个循环状态机，它根据我们的交互切换不同的状态。变量 context保存了当前的状态，case 语句都在该状态中之行。

如果你想看到完整可用的代码，你可以使用 regenerator，这是 facebook 下的一个工具，用于编译 ES6 的 generator 函数。
```
// 难道就不能编译一个完整可用的代码吗？
npm install -g regenerator
// 然后新建一个 generator.js 文件，里面的代码就是文章最一开始的代码，我们执行命令
regenerator --include-runtime generator.js > generator-es5.js
```


代码咋一看不长，但如果仔细观察会发现有两个不认识的东西 —— regeneratorRuntime.mark和regeneratorRuntime.wrap，这两者其实是 regenerator-runtime 模块里的两个方法。
regeneratorRuntime.mark()
regeneratorRuntime.mark(helloWorldGenerator)这个方法在第一行被调用，我们先看一下runtime里mark()方法的定义。
```
//runtime.js里的定义稍有不同，多了一些判断，以下是编译后的代码

runtime.mark = function(genFun) {
  genFun.__proto__ = GeneratorFunctionPrototype;
  genFun.prototype = Object.create(Gp);
  return genFun;
};

```
这里边GeneratorFunctionPrototype和Gp我们都不认识，他们被定义在runtime里，不过没关系，我们只要知道mark()方法为生成器函数（helloWorldGenerator）绑定了一系列原型。
```
// 构建关系链的目的在于判断关系的时候能够跟原生的保持一致，就比如
function* f() {}
var g = f();
console.log(g.__proto__ === f.prototype); // true
console.log(g.__proto__.__proto__ === f.__proto__.prototype); // true

在完整的编译代码中，为 Gp 添加了这三个函数的方法。
  // 编译后的94行
  function defineIteratorMethods(prototype) {
    ["next", "throw", "return"].forEach(function(method) {
      prototype[method] = function(arg) {
        return this._invoke(method, arg);
      };
    });
  }
  defineIteratorMethods(Gp);

为了简单起见，我们将整个 mark 函数简化为：
runtime.mark = function(genFun) {
  var generator = Object.create({
    next: function(arg) {
      return this._invoke('next', arg)
    }
  });
  genFun.prototype = generator;
  return genFun;
};

```

## regeneratorRuntime.wrap()
从上面babel转化的代码我们能看到，执行helloWorldGenerator()，其实就是执行wrap()，那么这个方法起到什么作用呢，他想包装一个什么东西呢，我们先来看看wrap方法的定义：
```
function wrap(innerFn, outerFn, self) {
  var generator = Object.create(outerFn.prototype);// outerFn.prototype 其实就是 genFun.prototype
  var context = new Context([]);
  generator._invoke = makeInvokeMethod(innerFn, self, context);

  return generator;
}
```
wrap方法先是创建了一个generator，并继承outerFn.prototype；然后new了一个context对象；makeInvokeMethod方法接收innerFn(对应helloWorldGenerator$)、context和this，并把返回值挂到generator._invoke上；最后return了generator。

其实wrap()相当于是给generator增加了一个_invoke方法。
这段代码肯定让人产生很多疑问，outerFn.prototype是什么？ Context又是什么？makeInvokeMethod又做了哪些操作？

outerFn.prototype其实就是genFun.prototype
context可以直接理解为这样一个全局对象，用于储存各种状态和上下文：
```
var ContinueSentinel = {};

var context = {
  done: false,
  method: "next",
  next: 0,
  prev: 0,
  abrupt: function(type, arg) {
    var record = {};
    record.type = type;
    record.arg = arg;

    return this.complete(record);
  },
  complete: function(record, afterLoc) {
    if (record.type === "return") {
      this.rval = this.arg = record.arg;
      this.method = "return";
      this.next = "end";
    }

    return ContinueSentinel;
  },
  stop: function() {
    this.done = true;
    return this.rval;
  }
};
```
每次 hw.next 的时候，就会修改 next 和 prev 属性的值，当在 generator 函数中 return 的时候会执行 abrupt，abrupt 中又会执行 complete，执行完 complete，因为 this.next = end 的缘故，再执行就会执行 stop 函数。

makeInvokeMethod的定义如下，它return了一个invoke方法，invoke用于判断当前状态和执行下一步，其实就是我们调用的next()；
```
//以下是编译后的代码
var ContinueSentinel = {};

function makeInvokeMethod(innerFn, context) {
  // 将状态置为start
  var state = "start";

  return function invoke(method, arg) {
    // 已完成
    if (state === "completed") {
      return { value: undefined, done: true };
    }
    
    context.method = method;
    context.arg = arg;

    // 执行中
    while (true) {
      state = "executing"; // 执行

      var record = {
        type: "normal",
        arg: innerFn.call(self, context)    // 执行下一步,并获取状态(其实就是switch里边return的值)
      };

      if (record.type === "normal") {
        // 判断是否已经执行完成
        state = context.done ? "completed" : "yield";

        // ContinueSentinel其实是一个空对象,record.arg === {}则跳过return进入下一个循环
        // 什么时候record.arg会为空对象呢, 答案是没有后续yield语句或已经return的时候,
        // 也就是switch返回了空值的情况(跟着上面的switch走一下就知道了)
        if (record.arg === ContinueSentinel) {
          continue;
        }
        // next()的返回值
        return {
          value: record.arg,
          done: context.done
        };
      }
    }
  };
}
```

当然这个过程，看文字理解起来可能有些难度，不完整但可用的代码如下，你可以断点调试查看具体的过程：
```
(function() {
  var ContinueSentinel = {};

  var mark = function(genFun) {
    var generator = Object.create({
      next: function(arg) {
        return this._invoke("next", arg);
      }
    });
    genFun.prototype = generator;
    return genFun;
  };

  function wrap(innerFn, outerFn, self) {
    var generator = Object.create(outerFn.prototype);

    var context = {
      done: false,
      method: "next",
      next: 0,
      prev: 0,
      abrupt: function(type, arg) {
        var record = {};
        record.type = type;
        record.arg = arg;

        return this.complete(record);
      },
      complete: function(record, afterLoc) {
        if (record.type === "return") {
          this.rval = this.arg = record.arg;
          this.method = "return";
          this.next = "end";
        }

        return ContinueSentinel;
      },
      stop: function() {
        this.done = true;
        return this.rval;
      }
    };

    generator._invoke = makeInvokeMethod(innerFn, context);

    return generator;
  }

  function makeInvokeMethod(innerFn, context) {
    var state = "start";

    return function invoke(method, arg) {
      if (state === "completed") {
        return { value: undefined, done: true };
      }

      context.method = method;
      context.arg = arg;

      while (true) {
        state = "executing";

        var record = {
          type: "normal",
          arg: innerFn.call(self, context)
        };

        if (record.type === "normal") {
          state = context.done ? "completed" : "yield";

          if (record.arg === ContinueSentinel) {
            continue;
          }

          return {
            value: record.arg,
            done: context.done
          };
        }
      }
    };
  }

  window.regeneratorRuntime = {};

  regeneratorRuntime.wrap = wrap;
  regeneratorRuntime.mark = mark;
})();

var _marked = regeneratorRuntime.mark(helloWorldGenerator);

function helloWorldGenerator() {
  return regeneratorRuntime.wrap(
    function helloWorldGenerator$(_context) {
      while (1) {
        switch ((_context.prev = _context.next)) {
          case 0:
            _context.next = 2;
            return "hello";

          case 2:
            _context.next = 4;
            return "world";

          case 4:
            return _context.abrupt("return", "ending");

          case 5:
          case "end":
            return _context.stop();
        }
      }
    },
    _marked,
    this
  );
}

var hw = helloWorldGenerator();

console.log(hw.next());
console.log(hw.next());
console.log(hw.next());
console.log(hw.next());
```


低配实现 & 调用流程分析
```
function* foo() {
  yield 'result1'
  yield 'result2'
  yield 'result3'
}
  
const gen = foo()
console.log(gen.next().value)
console.log(gen.next().value)
console.log(gen.next().value)


// 生成器函数根据yield语句将代码分割为switch-case块，后续通过切换_context.prev和_context.next来分别执行各个case
function gen$(_context) {
  while (1) {
    switch (_context.prev = _context.next) {
      case 0:
        _context.next = 2;
        return 'result1';

      case 2:
        _context.next = 4;
        return 'result2';

      case 4:
        _context.next = 6;
        return 'result3';

      case 6:
      case "end":
        return _context.stop();
    }
  }
}

// 低配版context  
var context = {
  next:0,
  prev: 0,
  done: false,
  stop: function stop () {
    this.done = true
  }
}

// 低配版invoke
let gen = function() {
  return {
    next: function() {
      value = context.done ? undefined: gen$(context)
      done = context.done
      return {
        value,
        done
      }
    }
  }
} 

// 测试使用
var g = gen() 
g.next()  // {value: "result1", done: false}
g.next()  // {value: "result2", done: false}
g.next()  // {value: "result3", done: false}
g.next()  // {value: undefined, done: true}
```

从中我们可以看出，Generator实现的核心在于上下文的保存，函数并没有真的被挂起，每一次yield，其实都执行了一遍传入的生成器函数，只是在这个过程中间用了一个context对象储存上下文，使得每次执行生成器函数的时候，都可以从上一个执行结果开始执行，看起来就像函数被挂起了一样。
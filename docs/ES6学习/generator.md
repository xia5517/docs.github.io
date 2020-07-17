## generator

> author: yang

### 一、什么是 Generator 函数
Generator函数是ES6提供的一种异步编程解决方案，形式上也是一个普通函数，但有几个显著的特征：
- function关键字与函数名之间有一个星号 "*" （推荐紧挨着function关键字）
```
function * foo(){...}
function *foo(){...}
function* foo(){...}
function*foo(){...}
```
- 函数体内使用 yield 表达式，定义不同的内部状态 （可以有多个yield）
- 直接调用 Generator函数并不会执行，也不会返回运行结果，而是返回一个遍历器对象（Iterator Object）
- 依次调用遍历器对象的next方法，遍历 Generator函数内部的每一个状态

**举个例子：**
```
{
  // 传统函数
  function foo() {
    return 'hello world'
  }

  foo()   // 'hello world'，一旦调用立即执行


  // Generator函数
  function* generator() {
    yield 'hello';         // yield 表达式是暂停执行的标记  
    yield 'world';
    return 'ending';
  }

  let g = generator();   // 调用 Generator函数，函数并没有执行，返回的是一个Iterator对象
  g.next();              // {value: "hello", done: false}，value 表示返回值，done 表示遍历还没有结束
  g.next();              // {value: "world", done: false}
  g.next();              // {value: "ending", done: true} value 表示返回值，done 表示遍历没有结束
}
          
```
上面的代码中可以看到Generator函数和传统函数的运行是完全不同的，传统函数调用后立即执行并输出了返回值；
Generator函数则没有执行而是返回一个Iterator对象，并通过调用Iterator对象的next方法来遍历，函数体内的执行看起来更像是“被人踢一脚才动一下”的感觉。

每次调用Iterator对象的next方法时，内部的指针就会从函数的头部或上一次停下来的地方开始执行，直到遇到下一个 yield 表达式或return语句暂停。换句话说，Generator 函数是分段执行的，yield表达式是暂停执行的标记，而 next方法可以恢复执行

执行过程如下：

1. 第一次调用next方法时，内部指针从函数头部开始执行，遇到第一个 yield 表达式暂停执行后面的操作，将紧跟在yield后面的那个表达式的值，作为返回的对象的value属性值-- 'hello'。
2. 第二次调用next方法时，内部指针从上一个（即第一个） yield 表达式挺值得位置继续往下执行，返回当前状态的值 'world'
3. 第三次调用next方法时，内部指针从第二个 yield 表达式开始，遇到return语句暂停，返回当前状态的值 'ending'，同时所有状态遍历完毕，done 属性的值变为true

### 提问:

1. **手欠想第四次调用next方法？**

第四次调用next方法时，由于函数已经遍历运行完毕，不再有其它状态，因此返回 `{value: undefined, done: true}`。如果继续调用next方法，返回的也都是这个值
```
{
  // Generator函数
  function* generator() {
    yield 'hello';
    yield 'world';
    return 'ending';
  }

  let g = generator();
    g.next();        // {value: "hello", done: false}
    g.next();        // {value: "world", done: false}
    g.next();        // {value: "ending", done: true}
    g.next();        // {value: undefined, done: true}
}
```

2. **如果generator函数没有return语句**
```
{
  // Generator函数
  function* generator() {
    yield 'hello';
    yield 'world';
  }

   let g = generator();
   g.next();     // {value: "hello", done: false}        
  g.next();     // {value: "world", done: false}
  g.next();     //{value: undefined, done: true}
}
```
第三次.next()之后就返回 `{value: undefined, done: true}`，这个第三次的next()唯一意义就是证明g函数全部执行完了。

3. **手欠就想 `generator()` 中 `return` 语句之后写 `yield` 语句?**
```
{
  // Generator函数
  function* generator() {
    yield 'hello';
    return  'ending';
    yield 'world';
  }

   let g = generator();
  g.next();     // {value: "hello", done: false}        
  g.next();     // {value: "ending", done: true}
  g.next();     //{value: undefined, done: true}
}
```
js的老规定：return语句标志着该函数所有有效语句结束，return下方还有多少语句都是无效，白写。

4. **如果generator函数没有yield和return语句呢？**
```
{
  // Generator函数
  function* generator() {
      
  }

   let g = generator();
  g.next();     // {value: undefined, done: true}        
  g.next();     // {value: undefined, done: true}
 g.next();     // {value: undefined, done: true}
}
```

之后也一直是`{value: undefined, done: true}`


5. **如果只有return语句呢？**
```
{
  // Generator函数
  function* generator() {
      return  'ending';
  }

   let g = generator();
  g.next();     // {value: 'ending', done: true}        
  g.next();     // {value: undefined, done: true}
 g.next();     // {value: undefined, done: true}
}
```
第一次调用就返回`{value: 'ending', done: true}`，之后也一直是`{value: undefined, done: true}`。

**总结**：
Generator 函数的特点就是
1. 分段执行，可以暂停
2. 可以知道每个阶段的返回值
3. 可以知道是否执行到结尾

### 二、yield语句

1. `yield` 表达式只能用在 Generator 函数里面，用在其它地方都会报错

```
{
  function (){
    yield 1;
  }
  // SyntaxError: Unexpected number
  // 在一个普通函数中使用yield表达式，结果产生一个句法错误
}

```

2. `yield` 表达式如果用在另一个表达式中，必须放在圆括号里面
```
{
  function* demo() {
    console.log('Hello' + yield); // Uncaught SyntaxError: Unexpected identifier
    console.log('Hello' + yield 123); // Uncaught SyntaxError: Unexpected identifier
  
    console.log('Hello' + (yield)); // OK
    console.log('Hello' + (yield 123)); // OK
  }
}
```

3. `yield` 表达式和 `return` 语句的区别

**相似**：都能返回紧跟在语句后面的那个表达式的值

**区别**：
- 每次遇到 yield，函数就暂停执行，下一次再从该位置继续向后执行；而 return 语句不具备记忆位置的功能
- 一个函数只能执行一次 return 语句，而在 Generator 函数中可以有任意多个 yield

### 三、next() 方法的参数
`yield` 表达式本身没有返回值，或者说总是返回 undefined。next 方法可以带一个参数，该参数就会被当作上一个yield表达式的返回值

先看一个简单的小栗子，并尝试解析遍历生成器函数的执行过程
```
{
  function* generator() {
    let result = yield 3 + 5 + 6
    console.log(result)
    yield result
  }

  let g = generator();
  console.log(g.next());      // {value: 14, done: false}
 console.log(g.next());      // undefined    {value: undefined, done: false}
}

{
  function* generator() {
    let result = yield 3 + 5 + 6
    console.log(result)
    yield result
  }

  let g = generator();
  console.log(g.next());      // {value: 14, done: false}
  console.log(g.next(3));      // 3    {value: 3, done: false}
}
```
如果第一次调用 `next()` 的时候也传了一个参数呢？这个当然是无效的，next方法的参数表示上一个yield表达式的返回值。

从语义上讲，第一个next方法用来启动遍历器对象，所以不用带有参数。
第一次使用 next 方法时，传递参数是无效的，因为第一个 `.next()` 的前面没有 `yield` 语句。
```
{
  function* generator() {
    let result = yield 3 + 5 + 6
    console.log(result)
    yield result
  }

  let g = generator()
  console.log(g.next(10))      // {value: 14, done: false}
  console.log(g.next(3))      // 3    {value: 3, done: false}
}
```
这个功能有很重要的语法意义，Generator 函数从暂停状态到恢复运行，它的上下文状态（context）是不变的。通过next方法的参数，就有办法在 Generator 函数开始运行之后，继续向函数体内部注入值。也就是说，可以在 Generator 函数运行的不同阶段，从外部向内部注入不同的值，从而调整函数行为。

再举个例子：
```
{
  function* gen(x) {
    let y = 2 * (yield (x + 1))   // 注意：yield 表达式如果用在另一个表达式中，必须放在圆括号里面
    let z = yield (y / 3)
    return x + y + z
  }

  let it = gen(5)
    
  console.log(it.next())  // 首次调用next，函数只会执行到 “yield(5+1)” 暂停，并返回 {value: 6, done: false}
  console.log(it.next())  // 第二次调用next，没有传递参数，所以 y的值是undefined，那么 y/3 当然是一个NaN，所以应该返回 {value: NaN, done: false}
  console.log(it.next())  // 同样的道理，z也是undefined，5 + undefined + undefined = NaN，返回 {value: NaN, done: true}
}
```

如果向next方法提供参数，返回结果就完全不一样了
```
{
  function* gen(x) {
    let y = 2 * (yield (x + 1))   // 注意：yield 表达式如果用在另一个表达式中，必须放在圆括号里面
    let z = yield (y / 3)
    return x + y + z
  }

  let it = gen(5)

  console.log(it.next())  // 正常的运算应该是先执行圆括号内的计算，再去乘以2，由于圆括号内被 yield 返回 5 + 1 的结果并暂停，所以返回{value: 6, done: false}
  console.log(it.next(9))  // 上次是在圆括号内部暂停的，所以第二次调用 next方法应该从圆括号里面开始，就变成了 let y = 2 * (9)，y被赋值为18，所以第二次返回的应该是 18/3的结果 {value: 6, done: false}
  console.log(it.next(2))  // 参数2被赋值给了 z，最终 x + y + z = 5 + 18 + 2 = 25，返回 {value: 25, done: true}
}
```

### 四、yield* 表达式
如果在 Generator 函数里面调用另一个 Generator 函数，默认情况下是没有效果的
```
{
  function* foo() {
    yield 'aaa'
    yield 'bbb'
  }

  function* bar() {
    foo()
    yield 'ccc'
    yield 'ddd'
  }

  let iterator = bar()

  for(let value of iterator) {
    console.log(value)
  }
  // ccc
  // ddd
}
```

`yield*` 表达式用来在一个 Generator 函数里面 执行 另一个 Generator 函数
```
{
  function* foo() {
    yield 'aaa'
    yield 'bbb'
  }

  function* bar() {
    yield* foo()      // 在bar函数中 **执行** foo函数
    yield 'ccc'
    yield 'ddd'
  }

  let iterator = bar()

  for(let value of iterator) {
    console.log(value)
  }

  // aaa
  // bbb
  // ccc
  // ddd
}
```
其中`foo()`是迭代器对象，可以把它赋值给变量，然后遍历这个变量。
这里需要注意，一旦 `next()` 方法的返回对象的 done 属性为 true，`for...of` 循环就会终止，且不包含该返回对象
```
function* foo() {
  yield 1;
  yield 2;
  yield 3;
  yield 4;
  yield 5;
  return 6;
}

let a = foo();

for (let v of a) {
  console.log(v);
}
// 1 2 3 4 5
```
遍历器提供一致的符合迭代器协议接口，可以统一可迭代对象遍历方式。 `for...of` 语句可以来迭代包含迭代器的可迭代对象（如 Array、Map、Set、String 等）。

### 五、Generator.prototype.return()
Generator 函数返回的遍历器对象，还有一个 return 方法，可以返回给定的值(若没有提供参数，则返回值的value属性为 undefined)，并且终结遍历 Generator 函数，状态变成 true
```
{
  function* gen() {
    yield 1
    yield 2
    yield 3
  }

  let it = gen()

  it.next()             // {value: 1, done: false}
  it.return('ending')   // {value: "ending", done: true}
  it.next()             // {value: undefined, done: true}
}
```

也就是说，return 的参数值覆盖本次 `yield` 语句的返回值，并且提前终结遍历，即使后面还有yield语句也一律无视。
### 六、Generator用于异步编程？
Generator可以暂停函数执行，返回任意表达式的值。这种特点使得Generator有多种应用场景。

Generator是实现状态机的最佳结构。比如，下面的clock函数就是一个常规写法的状态机。
```
var ticking = true;
var clock = function() {
  if (ticking)
    console.log('Tick!');
  else
    console.log('Tock!');
  ticking = !ticking;
}
```


上面代码的clock函数一共有两种状态（Tick和Tock），每运行一次，就改变一次状态。这个函数如果用Generator实现，就是下面这样。
```
var clock = function*() {
  while (true) {
    console.log('Tick!');
    yield;
    console.log('Tock!');
    yield;
  }
};
```

可以看到，Generator 函数实现的状态机不用设初始变量，不用切换状态，上面的Generator函数实现与ES5实现对比，可以看到少了用来保存状态的外部变量ticking，这样就更简洁，更安全（状态不会被非法篡改）、更符合函数式编程的思想，在写法上也更优雅。Generator之所以可以不用外部变量保存状态，是因为它本身就包含了第一个状态和第二个状态。

<img style="width: 474px; cursor: pointer;" alt="" src="https://note.youdao.com/yws/public/resource/a2e5c9c5d94ed1bd58645f3d02e16474/xmlnote/C1F73AD0EFFC424A88091D15995FCE7E/303" data-media-type="image" data-original="https://note.youdao.com/yws/public/resource/a2e5c9c5d94ed1bd58645f3d02e16474/xmlnote/C1F73AD0EFFC424A88091D15995FCE7E/303">

### 七、异步操作的同步化写法？
举个例子，比如我在测试服务器的某目录建了4个文件，分别是'test.html'、'a.html'、'b.html'、'c.html'，后三个文件的文件内容跟文件名相同，现在我编辑'test.html'的代码，想要先ajax-get相对网址'a.html'，然后再回调里ajax-get相对网址'b.html'，然后在回调里ajax-get相对网址'c.html'，常规的写法是（用上jQuery）：
```
//test.html

$.get('a.html',function(dataa) {
    console.log(dataa);
    $.get('b.html',function(datab) {
        console.log(datab);
        $.get('c.html',function(datac) {
            console.log(datac);
        });
    });
});

// a.html
// b.html
// c.html

回调地域
```
Geenerator写法
```
function request(url) {
  $.get(url, function(response){
    it.next(response);
  });
}

function* ajaxs() {
    console.log(yield request('a.html'));//从上一次停止的地方继续执行
    console.log(yield request('b.html'));
    console.log(yield request('c.html'));
}

var it = ajaxs();

it.next();

// a.html
// b.html
// c.html
```

当执行了 `it.next();` 之后，开始遍历 ajaxs() 对象。
```
console.log(yield request('a.html'));
console.log(yield request('b.html'));
console.log(yield request('c.html'));
```
ajaxs函数执行的第一步是`request('a.html')`，这是一个异步函数，但没关系，JS引擎会耐心等它执行完，它执行的第一步是向a.html发请求，回调执行`it.next(response)`，也就是把response传递给`it.next()`，这就有趣味了，这个next是第几个next？第二个。因为最初已经执行了一个了。现在有种什么感觉？没错，迭代的感觉。再复习一下next的参数，.next(response)意味着什么？意味着覆盖上一个yield语句的返回值。然后，`yield request('a.html')`将迭代暂停，然而下一个迭代已经开始了。

在每一个阶段开始，next(参数)干了两件事，第一件事是用参数覆盖前一个yield语句的值，第二件事是执行本阶段的代码，这样不断迭代下去，最终形成了一个next触发了一串next。这就形成了一个现象：最开始的一个.next()触发了一连串的request函数的执行，无论啥时候我想要执行这一串异步操作，我都只需要两行代码：var it = ajaxs(); it.next();就够了。

**最后**
Async/await 是现在的首选语法，这也是未来。但是，Generator 也在 ECMAScript 标准内，这意味着为了使用它们，除了写几个工具函数，你不需要任何东西。我试图向你们展示一些不那么简单的例子，这些实例的价值取决于你的看法。请记住，没有那么多人熟悉 Generator，并且如果在整个代码库中只有一个地方使用它们，那么使用 Promise 可能会更容易一些 —— 但是另一方面，通过 Generator 某些问题可以被优雅和简洁的处理。
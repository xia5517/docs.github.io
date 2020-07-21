# mudule (二)

> author: baoying

# module(二) ES6 模块加载

ES6 定义了模块的语法，但并未定义如何加载它们，而只对一个未定义的内部操作 **HostResolveImportedModule** 指定了语法以及抽象的加载 机制。 web 浏览器与 Node.js 可以自行决定用什么方式实现 HostResolveImportedModule ， 以便更好契合各自的环境。


## Web 浏览器加载模块

web 浏览器通过通过 `<script type="module">` 加载模块。

```
<!-- 外联 -->
<script type="module" src="module.js"></script> 

<!-- 内联 -->
<script type="module">
import { sum } from "./example.js"; let result = sum(1, 2);
</script>
```

> 当 type 属性无法辨认时，浏览器就会忽略 `<script>` 元素，因此不支持模 块的浏览器也就会自动忽略 `<script type="module"> `声明，从而提供良好的向下兼容性。



### defer 与 type="module" 

模块的加载更像是应用了 defer 属性， 回顾defer：

defer：HTML解析与JS加载并行进行，HTML解析完再执行JS。

当 HTML 解 析到拥有 src 属性的` <script type="module"> `标签时，就会立即开始下载模块文件，但并 不会执行它，直到整个网页文档全部解析完为止。模块也会按照它们在 HTML 文件中出现的 顺序依次执行，这意味着第一个 `<script type="module">` 总是保证在第二个之前执行，即使 其中有些模块不是用 src 指定而是包含了内联脚本。

<img src="https://agroup-bos-bj.cdn.bcebos.com/bj-f222e9f3a7ee777a08de94256a12f97ca9ee8971" width="300px" margin="0 auto;"/>

![图片](https://agroup-bos-bj.cdn.bcebos.com/bj-6fe5eeb511365ca30ffc24c7629410c8743e9262)


看个栗子：

```

<!-- this will execute first -->
<script type="module" src="module1.js"></script>

<!-- this will execute second -->
<script type="module">
import { sum } from "./example.js";
let result = sum(1, 2);
</script>


<!-- this will execute third -->
<script type="module" src="module2.js"></script>

```

加载次序为：

1. 下载并解析 module1.js ;
2. 递归下载并解析在 module1.js 中使用 import 导入的资源;
3. 解析内联模块;
4. 递归下载并解析在内联模块中使用 import 导入的资源;
5. 下载并解析 module2.js ;
6. 递归下载并解析在 module2.js 中使用 import 导入的资源。

一旦加载完毕，直到页面文档被完整解析之前，都不会有任何代码被执行。在文档解析完毕 后，会发生下列行为:

1. 递归执行 module1.js 导入的资源; 
2. 执行 module1.js ;
3. 递归执行内联模块导入的资源;
4. 执行内联模块;
5. 递归执行 module2.js 导入的资源; 6. 执行 module2.js 。


### aysnc 与 type=“module”

sync:  HTML解析与JS加载并行，JS加载完就执行，HTML解析等待，执行完JS再继续解析HTML。

![图片](https://agroup-bos-bj.cdn.bcebos.com/bj-40ddfdaba8673ab2e8be1f8faa9c295d7f119ea3)

举个栗子：

```

<script type="module" async src="module1.js"></script> 
<script type="module" async src="module2.js"></script>

```

两个模块文件被异步加载了。仅查看代码就判断出那个模块会被先执行，这是不可能 的。若 module1.js 首先结束下载(包括它的所有导入资源)，那么它就会首先执行。而对 于 module2.js 来说也是一样。

defer 与 async 的详细见[defer 与 async](./defer-async)

## 模块如何工作

加载一个模块可以分为三个步骤：

**construction**：查找.下载所有的文件并且解析为 模块记录。

**Instantiation**：把所有导出的变量放入内存指定位置(这时候还没有填入数据)。然后让导出和导入都指向内存指定位置,这叫 Linking。

**Evaluation**：执行代码，得到变量的值然后填入到内存对应位置

<img src="https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/03/07_3_phases.png" width="800px"/>



### construction （构建）

在构建阶段，每个模块都会：

1. 找出从哪里下载包含模块的文件（又称**模块解析**）

2. 获取文件（通过从URL下载或从文件系统加载）

3. 将文件解析为模块记录

**查找文件，并获取**

模块加载器将负责查找并下载文件。首先它需要找到入口点文件。在 web 浏览器环境中，通过使用脚本标记告诉加载程序在哪里可以找到它。

<img src="https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/03/08_script_entry.png" width="800px"/>

模块内的模块加载通过 import 语句的模块说明符：

<img src="https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/03/09_module_specifier.png" width="800px"/>

> import语句的模块说明符 依赖外部环境提供的独立机制实现对其字符串解析成指令：

> - 浏览器：内置模块加载器把模块标识符解析成URL；
> - node：内置模块加载器把模块标识符解析成本地文件系统路径。

解析一个文件，然后找出它的依赖项，然后找到并加载这些依赖项。

<img src="https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/03/10_construction.png" width="800px"/>

加载器会将加载的文件以URL为索引缓存成模块映射。

<img src="https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/03/15_module_map.png" width="800px"/>


**模块记录**

模块记录是一种记录模块依赖信息的汇总的数据结构。

<img src="https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/03/05_module_record.png" width="800px">

模块记录放置在模块映射表中，无论何时有人再请求它，加载器都可以从映射表中拉出它。

<img src="https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/03/25_module_map.png" width="800px">

模块解析的最后，就从只有一个入口点的文件变成了一堆模块记录。

<img src="https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/03/27_construction.png" width="800px">

### Instantiation

JS引擎会创建每个模块记录的环境作用域，将其对应到内存地址关联起来，确保相同模块的相同变量都指向相同的内存地址；每个导入变量都能找到对应的导出变量，因此当导出变量发生改变时会影响到所有对应的导入变量。


<img src="https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/03/30_live_bindings_01.png" width="800px"/>


<img src="https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/03/30_live_bindings_02.png"  width="800px"/>


CommonJS 导出拷贝的副本：

<img src="https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/03/31_cjs_variable.png" width="800px"/>


现在，导出/导入变量的所有实例已经和内存位置连接起来了。

### Evaluation

> 注意：通过 Instantiation 后，实例已经与内存关联，但是内存还未真正填入数据。

最后一步是在内存中填充数据。 JS引擎执行顶层作用域的代码（除函数内的）。

<img src="https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/03/40_top_level_code.png" width="800px"/>


## 总结

模块是使用不同方式加载的 JS 文件（与 JS 原先的脚本加载方式相对） 。这种不同模式很有必要，因为它与脚本（script ） 有大大不同的语义：

- 模块代码自动运行在严格模式下，并且没有任何办法跳出严格模式；
- 在模块的顶级作用域创建的变量，不会被自动添加到共享的全局作用域，它们只会在模块顶级作用域的内部存在；
- 模块顶级作用域的 this 值为 undefined ；
- 模块不允许在代码中使用 HTML 风格的注释（这是 JS 来自于早期浏览器的历史遗留特性） ；
- 对于需要让模块外部代码访问的内容，模块必须导出它们；
- 允许模块从其他模块导入绑定；

`type=module`使用默认 defer 加载。


参考：

[ES modules: A cartoon deep-dive](https://hacks.mozilla.org/2018/03/es-modules-a-cartoon-deep-dive/)

[Nodejs模块加载与ES6模块加载实现](https://segmentfault.com/a/1190000020007221)
# defer 与 async

> author: baoying


HTML的解析过程：

<img src="https://camo.githubusercontent.com/51dd00d6b925eedd2d43673add3e8a13ccb0120d/687474703a2f2f74616c696761727369656c2e636f6d2f50726f6a656374732f7765626b6974666c6f772e706e67" width="800px"/>

DOMContentLoaded 发生在HTML解析为DOM后，无论此时CSS解析为CSSOM的过程是否完成。

> MDN: 当初始的 HTML 文档被完全加载和解析完成之后，DOMContentLoaded 事件被触发，而无需等待样式表、图像和子框架的完全加载.

同步、defer 与 async 比较直观的区别在于 DOMContentLoaded 时间的触发点。

**1. 同步：HTML解析 -> 遇到js，加载js -> 执行js -> 继续解析HTML**

![图片](https://agroup-bos-bj.cdn.bcebos.com/bj-11665e5c1a2b618d559f341d13a3506bfdbffa7f)


**2. defer：HTML解析和JS加载同步进行，再JS执行**
如果 script 标签中包含 defer，那么这一块脚本将不会影响 HTML 文档的解析，而是等到 HTML 解析完成后才会执行。而 DOMContentLoaded 只有在 **defer 脚本执行结束后才会被触发**。

如果HTML还没解析完成时，defer脚本已经加载完毕，那么defer脚本将等待HTML解析完成后再执行。defer脚本执行完毕后触发DOMContentLoaded事件：

![图片](https://agroup-bos-bj.cdn.bcebos.com/bj-cb5fe7426719d5296fc79fb8e78ada4e5c12f657)

如果HTML解析完成时，defer脚本还没加载完毕，那么defer脚本继续加载，加载完成后直接执行，执行完毕后触发DOMContentLoaded事件

**3. aysnc: HTML解析和JS加载同步进行 -> 加载完JS就执行 -> 执行完再继续解析HTML**

aysnc DOMContentLoaded 在HTML 解析完即可触发，这意味着async脚本可能会在 DOMContentLoaded 之前或之后执行。

如果HTML 还没有被解析完的时候，async脚本已经加载完了，那么 HTML 停止解析，去执行脚本，脚本执行完毕后触发DOMContentLoaded事件：

<img src="https://agroup-bos-bj.cdn.bcebos.com/bj-217b35ea93323f5e3084ebc57aaec4c5fa8054bf" width="500px">

如果HTML 解析完了之后，async脚本才加载完，然后再执行脚本，那么在HTML解析完毕、async脚本还没加载完的时候就触发 DOMContentLoaded 事件：

![图片](https://agroup-bos-bj.cdn.bcebos.com/bj-1cf9c1f5fb660fc089ad880a5a69b5fb9478f0b0)

defer 中 js的加载、执行顺序是可以预测的，即顺序加载顺序执行。 aysnc中的js 加载、执行顺序是不可预测的，谁先加载谁就先执行。



参考：
https://blog.csdn.net/zyj0209/article/details/79698430
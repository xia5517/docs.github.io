# ES6-Class
## 为什么要引入类
- 代码世界需要对宏观事物进行高度抽象
- 俗话说“物以类聚，XXXX”
- 管理资源的有效方式是分类
- 加之面向对象的语言思想
- 经常重复的实例，相同的能力

万物皆对象，这是对事物的抽象。
对象是类的具象化产物，即实例。
（如同ES与JS一般。）
同一个类的多个实例具备相同能力；多个子类可以获得父类的能力，即继承。

不同的设计语言可能对于继承的实现有所不同。
JS采用关联的方式实现类与继承，即实例通过指针关联类（原型），需要类能力时通过指针查找。

## JS究竟有没有“类”
如果这是一道面试题，我觉得她一定很开放。

无论是与其它语言比较类的实现方式；
或者与其它形式的类思想进行比较

过程可能不同，但做成了相同的事。

## 让我们来看看传统的ES怎么实现这个事
```javascript
function Kitchen(cooker){
	this.dish = 'yellow chicken with rice';
	this.cooker = cooker;
}

Kitchen.prototype.baocaiming = function(){
	console.log(`%ci am ${this.cooker}, and i cooked ${this.dish}`,"color: gold");
}

var K1 = new Kitchen('junlin');
var K2 = new Kitchen('xiaorui');
```

![](ES6-Class/pasted-from-clipboard.png)


## 那我就不new，行不行
```JavaScript
function Kitchen(cooker){
	this.dish = 'yellow chicken with rice';
	this.cooker = cooker;
	return '完犊子';
}
Kitchen.prototype.baocaiming = function(){
	console.log(`%ci am ${this.cooker}, and i cooked ${this.dish}`,"color: gold");
}
var K1 = new Kitchen('junlin');
var K2 = new Kitchen('xiaorui');
var K3 = Kitchen('yangpeng');
```

![](ES6-Class/pasted-from-clipboard.png)

## function你个臭spy，竟然有俩身份？
![](ES6-Class/pasted-from-clipboard.png)

## [[Construct]]与Constructor的区别
![](ES6-Class/pasted-from-clipboard.png)

## ES5如何抓贼？
```javascript
function PersonType(name) {
	// this.__proto__ === PersonType.protoType   this.constructor === PersonType
	if(this instanceof PersonType){ 
	this.name = name;
}else{
	throw new Error('必须通过new关键字来调用Person');
}
	return 1;
}
```

![](ES6-Class/pasted-from-clipboard.png)

## ES5校验的缺陷，糟老头子坏得很
```javascript
var person = new PersonType('yangpeng');
var robot = PersonType.call(person, '擎天柱'); //期望是抛出异常
```

![](ES6-Class/pasted-from-clipboard.png)

## ES6的做法
```javascript
var v8 = {
	find(){
		'new && new.target = fn'
	}
};
function Person(name){
	if(typeof new.target !== void 0){
		this.name = name;
	}else{
		throw new Error('必须通过new关键字来调用Person');
	}
}
```

## 看下new实现
```javascript
function _new(){
  var args = [].slice.call(arguments);
  var constructor = args.shift();
  var context = Object.create(constructor.prototype);
  var result = constructor.apply(context, args);
  return (typeof result === 'object' && result != null) ? result : context;
}
```

![](ES6-Class/pasted-from-clipboard.png)

## 使用ES6类
```javascript
class Person {
	constructor(name){
		this.name = name;
	}
	sayName(){
		console.log(this.name);
	}
}
var person = new Person('yangpeng');
person.sayName();
console.log(typeof Person);
```

## 实现ES6 Class
```javascript
let PersonTypeFaker = (function(){
	"use strict";
	const PersonTypeFaker = function(name){
	if(typeof new.target === void 0){
		throw new Error("必须通过关键字new调用构造函数");
	}
	this.name = name;
}
Object.defineProperty(PersonTypeFaker.prototype, 'sayName', {
	value: function(){
      		if(typeof new.target !== void 0){
          		throw new Error("必须通过关键字new调用构造函数");
      		}
	},
	enumerable: false,
	writable: true,
	configurable: true
});
return PersonTypeFaker;
})()
```

## ES5 构造函数实现 vs ES6 Class
- 类声明不能被提升，TDZ
- 类声明中使用严格模式
- 类中的全部方法不可枚举
- 每个类都确保存在[[construct]]
- 只能通过new调用，没有[[call]]

## 总结
ES6中的类是作为传统ES5继承模型的语法糖出现的，在其基础上增加了一些降低风险的特性，将constructor与prototype结构上划分在了一起。






























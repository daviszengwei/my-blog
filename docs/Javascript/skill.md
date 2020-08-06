---
title: 'js基础'
date: 2020-06-29
tags:
- JS基础
- 基础
categories: 
- javascript
sidebarDepth: 2
---

## js的数据类型的转换  

在js中类型转换只有**三种情况**，分别是：
- 转换为布尔值（调用<code>Boolean()</code>方法）
- 转换为数字（调用<code>Number()</code>、<code>parseInt()</code>、<code>parseFloat()</code>方法）
- 转换为字符串（调用<code>.toString()</code>或者<code>String()</code>方法）

::: danger
<code>null</code> 和 <code>undefined</code> 没有<code>.toString</code> 方法
:::

## JS中数据类型的判断

1. typeof  
  typeof对于原始类型来说，除了null都可以显示正确的类型
```javascript
console.log(typeof 2);               // number
console.log(typeof true);            // boolean
console.log(typeof 'str');           // string
console.log(typeof []);              // object     []数组的数据类型在 typeof 中被解释为 object
console.log(typeof function(){});    // function
console.log(typeof {});              // object
console.log(typeof undefined);       // undefined
console.log(typeof null);            // object     null 的数据类型被 typeof 解释为 object

```
typeof对于对象来说，**除了函数都会显示object，**所以说typeof并不能准确判断变量到底是什么类型，所以想判断一个对象的正确的类型，这时候可以考虑使用<code>instanceof</code>

2. instanceof  
instanceof可以正确的判断对象的类型，因为内部机制是通过判断对象的原型链中是不是能找到类型的<code>prototype</code>
```javascript
console.log(2 instanceof Number);                    // false
console.log(true instanceof Boolean);                // false 
console.log('str' instanceof String);                // false  
console.log([] instanceof Array);                    // true
console.log(function(){} instanceof Function);       // true
console.log({} instanceof Object);                   // true    
// console.log(undefined instanceof Undefined);
// console.log(null instanceof Null);
```
可以看出直接的字面量值判断数据类型，instanceof可以精确判断引用数据类型（Array, Funtion, Object）,而基本数据类型不能被instanceof精确判断。

我们来看一下instanceof在MDN中的解释：instanceof运算符用来测试一个对象在其原型链中是否存在一个构造函数的prototype属性。其意思就是判断**对象是否是某一数据类型（如Array）的实例**，请重点关注一下是判断一个对象是否是数据类型的实例。在这里字面量值，2，true，‘str’不是实例，所以判断值为false。

3. constructor
```javascript
console.log((2).constructor === Number); // true
console.log((true).constructor === Boolean); // true
console.log(('str').constructor === String); // true
console.log(([]).constructor === Array); // true
console.log((function() {}).constructor === Function); // true
console.log(({}).constructor === Object); // true
```
::: danger
这里有一个坑，如果我创建一个对象，更改它的原型，constructor就会变得不可靠了
:::

```javascript
function Fn(){};
 
Fn.prototype=new Array();
 
var f=new Fn();
 
console.log(f.constructor===Fn);    // false
console.log(f.constructor===Array); // true 

```

4. <code>Object.prototype.toString().call()</code>使用Object对象的原型方法toString，使用call进行狸猫换太子，借用Object的toString方法

```javascript
var a = Object.prototype.toString;
 
console.log(a.call(2));
console.log(a.call(true));
console.log(a.call('str'));
console.log(a.call([]));
console.log(a.call(function(){}));
console.log(a.call({}));
console.log(a.call(undefined));
console.log(a.call(null));
```

5. {} 和 [] 的 valueof 和 toString 的结果是什么？  
```javascript
{} 的 valueof 结果为 {}，toString 的结果为 "[object Object]"

[] 的 valueof 结果为 []，toString 的结果为 ""
```

## JavaScript的作用域和作用域链 
**作用域**：作用域是定义变量的区域，它有一套访问变量的规则，这套规则来管理浏览器引擎如何在当前作用域以及嵌套的作用域中根据变量（标识符）进行变量查找。  
**作用域链**：作用域链的作用是保证对执行环境有权访问的所有变量和函数的有序访问，通过作用域，我们可以访问到外层环境的变量和函数。

## JavaScript原型，原型链？有什么特点
在js中我们是使用构造函数来新建一个对象的，每一个构造函数的*内部都有一个prototype属性值*，这个属性值是一个对象，这个对象包含了可以由该构造函数的所有实例共享的属性和方法。当我们使用构造函数新建一个对象后，在这个对象的内部将包含一个指针，这个指针指向构造函数的prototype属性对应的值，在ES5中这个指针被称为**对象的原型**。  
一般来说我们是不应该能够获取到这个值的，但是现在浏览器中都实现了<code>proto</code>属性来让我们访问这个属性，但是我们最好不要使用这个属性，因为它不是规范中规定的。ES5中新增一个<code>Object.getPrototypeOf()</code>方法，我们可以通过这个方法来获取对象的原型。  
当我们访问一个对象的属性时，如果这个对象内部不存在这个属性，那么它就会去它的原型对象里找这个属性，这个原型对象又会有自己的原型，于是就这样一直找下去，也就是**原型链**的概念。原型链的尽头一般来说都是<code>Object.prototype</code>所以这就是我们新建的对象为什么能够使用<code>toString()</code>等方法的原因。  
*特点：*  
JavaScript对象是通过引用来传递的，我们创建的每个新对象实体中并没有一份属于自己的原型副本。当我们修改原型时，与之相关的对象也会继承这一改变。

## js获取原型的方法？
- p.proto
- p.constructor.prototype
- Object.getPrototypeOf(p)

## 什么是 DOM 和 BOM？  
**DOM** 指的是文档对象模型，它指的是把文档当做一个对象来对待，这个对象主要定义了出来网页内容的方法和接口。   
**BOM** 指的是浏览器对象模型，它指的是把浏览器当做一个对象来对待，这个对象主要定义了与浏览器进行交互的方法和接口。BOM的核心是window，而window对象具有双重角色，它既是通过js访问浏览器窗口的一个接口，又是一个Global(全局)对象。这意味着在网页中定义的任何对象，变量和函数，都作为全局对象的一个属性或者方法存在。window对象含有location对象、navigator对象、screen对象等子对象，并且DOM的最根本的对象document对象也是BOM的window对象的子对象。

## display和visibility的区别
相同点：当display为none、visibility为hidden时都可以隐藏元素  
不同点：  
1. display:none 这个元素就变成了一个不显示的元素，会隐藏掉元素空间；但visibility:hidden 虽然不在浏览器显示出来，但是会保留元素空间，只是不显示而已；
2. display有很多值，visibility只有两个常用的值：visible，hidden

## sass, less, css的联系和区别
<code>sass</code>和<code>less</code>都是css扩展，目的都是使css更方便，更强大
1. <code>sass</code>更复杂，<code>less</code>更简单
2. 编译环境不同，<code>sass</code>安装时需要ruby环境，sass是在服务器端处理的；而<code>less</code>需要引入less.js来处理less代码输出css到浏览器
3. 变量符不一样，<code>sass</code>是<code>$</code>, <code>less</code>是<code>@</code>
4. <code>sass</code>允许使用条件编译，列如if{}else{},for{}等，而<code>less</code>不允许

## cookie, sessionStorage, localStorage的区别
| 不同点 | 存储大小 | 有效时间 | 数据与服务器交互方式 |
|-------|:--------:|:---------:|---------:|
| cookie | <=4M |在设置cookie过期之前一直有效（无论窗口浏览器是否关闭）|正常情况下，cookie数据会自动传到服务器，服务器也可以写cookie到客户端|
| sessionStorage| 5M | 数据在当前浏览器关闭后删除（sessionStorage与存储数据的顶级窗口或浏览器选项卡具有相同的生命周期） | 不会发送数据到服务器 |
| localStorage | 5M |持久存储，浏览器关闭后不会丢失除非主动删除（知道web应用程序删除它或用户要求浏览器删除它）|不会发送数据到服务器|
---
title: 'this、call、apply和bind'
date: 2020-06-30
tags:
- javascript
categories: 
- javascript
sidebarDepth: 2
---

## this的初衷
this设计的初衷是**在函数内部使用，用来指代当前的运行环境。**

JavaScript中的对象的赋值行为是将地址赋给一个变量，引擎在读取变量的时候其实就是要了个地址然后再从原始地址中读取对象。而JavaScript允许函数体内部引用当前环境的其他变量，而这个变量是由运行环境提供的。由于函数又可以在不同的运行环境执行（如全局作用域内执行，对象内执行...）所以需要一个机制来表明代码到底在哪里执行！于是this出现了，它的设计目的就是在函数体内部，指代函数当前的运行环境。

## global this（全局this）
1. this等价于window对象；
2. 用var声明一个变量和给this或者window添加属性是等价的；
3. 如果你在声明一个变量的时候没有使用var或者let、const，你就是在给全局的this添加或者改变属性值。

```javascript
// 1
console.log(this === window); //true
//2
var name = "Jake";
console.log(this.name ); // "Jake"
console.log(window.name ); // "Jake"

//3
 age = 23;
 function testThis() {
   age = 18;
 }
 console.log(this.age ); // 23
 testThis();
 console.log(this.age ); // 18
```

## function this(函数中this)
对于函数中的this的指向问题，有一句话很好用：**运行时this永远指向最后调用它的那个对象。**

```javascript
var name = "windowsName";
function sayName() {
var name = "Jake";
console.log(this.name);   // windowsName
console.log(this);    // Window
}
sayName();
console.log(this) // Window
```

:::tip
需要注意的是，对于严格模式来说，默认绑定全局对象是不合法的，this被置为undefined。会报错<code>Uncaught TypeError: Cannot read property 'name' of undefined。</code>
:::

```javascript
function foo() {
    console.log( this.age );
}

var obj1 = {
    age : 23,
    foo: foo
};

var obj2 = {
    age : 18,
    obj1: obj1
};

obj2.obj1.foo(); // 23

```
**还是开头的那句话，最后调用<code>foo()</code>的是<code>obj1</code>，所以this指向<code>obj1</code>，输出23**

## call、apply和bind中的this
call、apply、bind被称之为this的强绑定，用来改变函数执行时的this指向，目前所有关于他们的运用，都是基于这一点来进行的。

```javascript
var name = 'zjk';
  function fun() {
  console.log (this.name);
}

var obj= {
  name: 'jake'
};
fun(); // zjk
fun.call(obj); //Jake
```
:::tip
上面的<code>fun.call(obj)</code>等价于<code>fun.apply(obj)</code>和<code>fun.bind(obj)</code>
:::

## 箭头函数中的this
es5中的this要看函数在什么地方调用（即要看运行时），通过谁是最后调用它该函数的对象来判断this指向。但es6的**箭头函数中没有this绑定，必须通过查找作用域链来决定其值**，如果箭头函数被非箭头函数包含，则this绑定的是最近一层非箭头函数的this，否则，this为undefined。**箭头函数的this始终指向函数定义时的this，而非执行时。**

```javascript
    let name = "zjk";

    let o = {
        name : "Jake",

        sayName: function () {
            console.log(this.name)     
        },

        func: function () {
            setTimeout( () => {
                this.sayName()
            },100);
        }

    };

    o.func()     // Jake
```

::: warning
使用call、apply和bind等方法给this传值，箭头函数会忽略。**箭头函数引用的是箭头函数在创建时设置的this值**
:::

```javascript
let obj = {
  name: "Jake",
  func: (a,b) => {
      console.log(this.name,a,b);
  }
};
obj.func.call(obj,1,2);//'' 1 2
obj.func.apply(obj,[1,2]);//'' 1 2
```

*最后放一道常见的this面试题*
```javascript
var number = 1;

var obj = {

	number:2,

	showNumber:function(){

        this.number = 3;

        (function(){

            console.log(this.number);

        })();

        console.log(this.number);

    }

};

obj.showNumber();
// undefined  
// 3
```

## call & apply

每个函数都包含两个非继承而来的方法：apply()和call()。这两个方法的用途都是在特定的作用域中调用函数，实际上等于设置函数体内this对象的值。

### apply()
apply()方法接受两个参数：一个是在其中运行函数的作用域，另一个是参数数组。其中，第二个参数可以是Array的实例，也可以是arguments对象。

```javascript
function sum(num1, num2){ 
 return num1 + num2; 
} 
function callSum1(num1, num2){ 
 return sum.apply(this, arguments); // 传入 arguments 对象
} 
function callSum2(num1, num2){ 
 return sum.apply(this, [num1, num2]); // 传入数组
} 
console.log(callSum1(10,10)); //20
console.log(callSum2(10,10)); //20
```

:::danger
在严格模式下，未指定环境对象而调用函数，则this值不会转型为window。除非明确把函数添加到某个对象或者调用apply()或call()，否则this值将是undefined
:::

### call()
call()方法和apply()方法的作用相同，它们的唯一区别在于接收参数的方式不同。在使用call()方法时，传递给函数的参数必须逐个列举出来。

```javascript
function sum(num1, num2){
    return num1 + num2;
}
function callsum(num1, num2){
    return sum.call(this, num1, num2);
}
console.log(callsum(10, 10)); //20
```

call()方法与apply()方法返回的结果是完全相同的，至于是使用apply()还是call()，完全取决你采取哪种函数传递参数的方式最方便。

:::tip
- 参数数量/顺序确定就用call，参数数量/顺序不确定就用apply.
- 考虑可读性：参数数量不多就用call，参数数量比较多的话，就把参数合成数组，使用apply。  
:::

## bind()
bind()方法会创建一个函数的实例，其this值会被绑定到传给bind()函数的值。意思就是bind()会返回一个新函数。例如：

```javasript
window.color = "red"; 
var o = { color: "blue" }; 
function sayColor(){ 
 alert(this.color); 
} 
var objectSayColor = sayColor.bind(o); 
objectSayColor(); //blue
```

## call/apply/bind 的区别
*执行：*
- call/apply改变了函数的this上下文后马上执行该函数
- bind 则是返回改变上下文后的函数，不执行该函数

```javascript
function add (a, b) {
    return a + b;
}

function sub (a, b) {
    return a - b;
}

add.bind(sub, 5, 3); // 这时，并不会返回 8
add.bind(sub, 5, 3)(); // 调用后，返回 8
```
*返回值：*
- call/apply 返回fun的执行结果
- bind返回fun的拷贝，并指定了fun的this指向，保存了fun的参数

## call/apply/bind的核心理念
从上面几个简单的例子可以看出<code>call/apply/bind</code>是在向其他对象借用方法，这也符合我们的正常思维，举个简单的栗子。
我和我高中一个同学玩的超级好，衣服鞋子都是共穿的，去买衣服的时候，他买衣服，我买鞋子；回来后某天我想穿他买的衣服了，但是我没有，于是我就借用他的穿。这样我就既达到了穿新衣服的目的，又节省了money~
A对象有个方法，B对象因为某种原因也需要用到同样的方法，这时候就可以让B借用 A 对象的方法啦，既达到了目的，又节省了内存。

**这就是call/apply/bind的核心理念：借。**

**总结**
:::tip
1. 在浏览器里，在全局范围内this指向window对象；
2. 在函数里，this永远指向最后调用他的那个对象；
3. 构造函数中，this指向new出来的那个新的对象；
4. call、apply、bind中的this被强绑定在指定的那个对象上；
5. 箭头函数中this比较特殊，箭头函数this为父作用域的this，不是调用时的this，要知道四种方式，都是调用时确定，也就是动态的，而箭头函数的this指向是静态的，声明的时候就确定下来；
6. apply、call、bind都是js给函数内置的一些API，调用他们可以为函数指定this的指向，同时也可以传参。
:::

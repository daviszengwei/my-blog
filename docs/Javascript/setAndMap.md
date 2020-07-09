---
title: 'Set & Map'
date: 2020-06-10
tags:
- javascript
- ES6
categories: 
- ES6
sidebarDepth: 2
---

# JavaScript 深度剖析

## Set 和 Map

### Set

Set类似于数组，但是，它的成员必须是惟一的。没有重复的值（可以用来做去重）。

**新建一个set**  

```javascript
const set = new Set([1,2,3,4,5])
```
 Set的构造器函数接收一个参数来生成set。那就是可迭代对象。只要部署了Iterator接口的类数组对象，都可以用来生成set；

 **Set的主要属性**  
 Set的主要属性有两个：  
 * <code>size</code> (用来获取Set的长度，类似于数组的length)  
 * <code>constructor</code> (指向构造器函数Set)

 **Set的基础方法**  
 Set作为一种新的数据结构，主要有下面四种方法：  
 * <code>add(value)</code>: 添加某个值，返回Set结构本身，类似于数组的push操作
 * <code>delete(value)</code>: 删除某个值，返回一个布尔值，表示删除是否成功
 * <code>has(value)</code>: 返回一个布尔值，表示该值是否为Set的成员
 * <code>clear(value)</code>: 清除所有成员，返回undefined
 
 **Set的遍历方法**  
 Set主要有下面几种遍历方法：  
 * <code>keys()</code>: 返回一个键的遍历器
 * <code>values()</code>: 返回键值的遍历器
 * <code>entries()</code>: 返回一个带键值对的遍历器
 * <code>forEach()</code>: 同数组的forEach行为一致

 **注意点**  
 因为set是没有键的，所以，**所有的键实际上都等于value**。因为set没有键，所以无法像数组一样直接去的其中某个值，类似于<code>set[1]</code>是取不到值的。

 **set的主要应用场景**  
 数组的去重，既然set不允许出现重复的元素，那么我们可以这么实现**数组的去重**

 ```javascript
let arr = [1,2,3,5,5,6];
arr = [...new Set(arr)]
console.log(arr)      // [1,2,3,5,6]
 ```
**这个技巧的适用范围是数组中的数值类型为<code>undefined</code> <code>null</code> <code>boolean</code> <code>string</code> <code>number</code>**  
**当包含<code>object</code> <code>function</code> <code>array</code>时，则不适用**  

 ### Map  

 Map是ES6中的新的数据类型，在过去，我们知道对象总是以**键值对**的形式存在，我们的对象的**键仅能使用字符串**来充当，在ES6中，我们可以使用map的结构，他的结构类似于对象，但是允许**键为任何数据类型**  

 ```javascript
const obj = {name:'张三'}
const m = new Map();
m.set(obj, "描述对象")
console.log(m)      // { key:{ name: '张三' },value: '描述对象' }
 ```









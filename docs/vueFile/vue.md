---
title: Vue 基础
date: 2020-06-12
tags:
- vue
categories: 
- 基础
- 面试
sidebarDepth: 2
---

## vue生命周期各阶段发生的事

**beforeCreate:**  
实例初始化

**created:**  
已完成：数据观测，事件。  
接下来：
1. 判断是否有el选项，没有就停止生命周期，直到vue实例调用vm.$mount(el)；有就继续向下编译；
2. 再判断有没有template选项，有就编译成render函数，没有就将外部HTML作为模板编译；
3. 如果在HTML结构中增加一串html，在vue对象中增加template选项，会显示template中的内容，如果隐藏template选项，则会显示HTML的内容，vue中还有一个render函数，它是以createElment作为参数，然后做渲染，我们可以直接嵌入js。

在官方文档上也有这段描述：  
:::danger
Vue 选项中的 render 函数若存在，则 Vue 构造函数不会从 template 选项或通过 el 选项指定的挂载元素中提取出的 HTML 模板编译渲染函数。
:::

:::tip
所以：编译优先级是：render函数 > template选项 > outer HTML
::: 


**beforeMount:**  
开始给实例对象vm创建$el，并且替换掉el选项 

**mounted:**  
这里页面还没有渲染出来，还是{{message}}占位的，还是JavaScript中的虚拟DOM形式存在的，  
在这之后，页面才显示完成

**beforeUpdate:**  
数据改变时调用，虚拟DOM更新之前访问现有的DOM；  
*这个钩子可以监听到data的变化，但是view层没有被重新渲染，view层数据没有变化*

**updated:**  
虚拟DOM改变之后调用；
*view层才被重新渲染，数据更新*

**beforeDestroy:**  
实例销毁前调用，这一步，实例仍然完全可用

**destroyed:**  
实例销毁后调用，*该钩子被调用后，对应 Vue 实例的所有指令都被解绑，所有的事件监听器被移除，所有的子实例也都被销毁*

## 对MVVM的理解

*MVVM是Model-View-ViewModel的缩写。*  

**Model** 代表数据模型，也可以在Model中定义数据修改和操作的业务逻辑  
**View** 代表UI组件，它负责将数据模型转化成UI展现出来  
**ViewModel** 监听模型数据的改变和控制视图行为、处理用户交互，简单理解就是一个同步View和Model的对象，连接Model和View。  
在MVVM架构下，View和Model之间并没有直接的联系，而是通过ViewModel进行交互，Model和ViewModel之间的交互是双向的，因此View数据的变化会同步到Model中，而Model数据的变化也会立即反应到View上。  
ViewModel 通过双向数据绑定吧View层和Model层连接了起来，而View和Model之间的同步工作完全是自动的，无需人为干涉，因此开发者只需关注业务逻辑，不需要手动操作DOM，不需要关注数据状态的同步问题，复杂的数据状态维护完全由MVVM来统一管理。 

**MVVM(Model-View-ViewModel), 源自于经典的 Model–View–Controller（MVC）模式。MVVM 的出现促进了 GUI 前端开发与后端业务逻辑的分离，极大地提高了前端开发效率。MVVM 的核心是 ViewModel 层，它就像是一个中转站（value converter），负责转换 Model 中的数据对象来让数据变得更容易管理和使用，该层向上与视图层进行双向数据绑定，向下与 Model 层通过接口请求进行数据交互，起呈上启下作用。View 层展现的不是 Model 层的数据，而是 ViewModel 的数据，由 ViewModel 负责与 Model 层交互，这就完全解耦了 View 层和 Model 层，这个解耦是至关重要的，它是前后端分离方案实施的最重要一环。**

![alt MVVM]('/my-vuepress-blog/image.png')

## 一句话就能回答的面试题  
1. DOM 渲染在 哪个周期中就已经完成？  
答：DOM渲染在mounted中就已经完成了。  
2. $route 和 $router 的区别  
答：*$route* 是“路由信息对象”，包括path, params, hash, query, fullpath, matched, name等路由信息参数。而*$router*是“路由实例”对象，包括了路由的跳转方法，钩子函数等  
3. vue.js的两个核心是什么？  
答：数据驱动、组件系统  
4. vue常用的指令  
答：v-for, v-if, v-else, v-bind, v-on, v-show
5. vue常用的修饰符  
答：<code> .prevent</code> : 提交事件不再重载页面; <code>.stop</code> :阻止单击事件冒泡; <code>.self</code> : 当事件发生在该元素本身而不是子元素的时候触发; <code>.capture</code> : 事件侦听，事件发生的时候会调用  
6. vue中key值的作用？  
答：当vue.js用v-for正在更新已渲染过的元素列表时，它默认用**就地复用**策略。如果数据项的顺序被改变，Vue将不会移动DOM元素来匹配数据项的顺序，而是简单复用此处每个元素，并且确保它在特定索引下显示已被渲染过的每个元素。**key的作用主要是为了高效的更新虚拟DOM**  
7. 什么是vue的计算属性？  
答：在模板中放入太多的逻辑会让模板过重且难以维护，在需要对数据进行复杂处理，且可能多次使用的情况下，尽量采取计算属性的方式。  
    **好处**：
    - 使得数据处理结构清晰；
    - 依赖于数据，数据更新，处理结果自动更新；
    - 计算属性内部<code>this</code>指向vm实例；
    - 在<code>template</code>调用时，直接写计算属性名即可；
    - 常用的是<code>getter</code>方法，获取数据，也可以使用set方法改变数据；
    - 相较于<code>methods</code>，不管依赖的数据变不变，<code>methods</code>都会重新计算，但是依赖数据不变的时候<code>computed</code>从缓存中获取，不会重新计算   

8. vue等单页面应用及其优缺点  
答：**优点** ：Vue的目标是通过尽可能简单的API实现响应的数据绑定和组合的视图组件，核心是一个响应的数据绑定系统。MVVM、数据驱动、组件化、轻量、简洁、高效、快速、模块友好。  
    **缺点**：不支持低版本的浏览器，最低只支持到IE9;不利于SEO的优化（如果要支持SEO，建议通过服务端进行渲染组件）；第一次加载首页耗时相对长一些；不可以使用浏览器的导航按钮，需要自行实现前进、后退。
9. 怎么定义vue-router的动态路由？怎么获取传过来的值  
答：在router目录下的**index.js**文件中，对path属性加上<code>/:id</code>，使用router对象的params.id获取

## Vue实现数据双向绑定的原理：<code>Object.defineProperty()</code>  
vue实现数据双向绑定主要是：**采用数据劫持结合发布者-订阅者模式的方式**，通过<code>Object.defineProperty()</code>来劫持各个属性的setter、getter,在数据变动是发布消息给订阅者，触发相应监听回调。当把一个普通的<code>JavaScript</code>对象传给vue实例来作为它的<code>data</code>选项时，Vue将遍历它的属性，用<code>Object.defineProperty()</code>将它们转为getter/setter。用户看不到getter/setter，但是在内部它们让Vue追踪依赖，在属性被访问和修改时通知变化。

vue的数据双向绑定将MVVM作为数据绑定的入口，整合<code>Observer, Compile</code>和<code>Watcher</code>三者，通过<code>Observer</code>来监听自己的model的数据变化，通过<code>Compile</code>来解析模板指令（vue中是用来解析{{}}），最终利用<code>watcher</code>搭起<code>observer</code>和<code>compile</code>之间的通信桥梁，达到**数据——>视图更新；视图交互变化（input）——>数据model变更**双向绑定效果  

**js实现简单的双向绑定**
```javascript
 <body>
    <div id="app">
    <input type="text" id="txt">
    <p id="show"></p>
</div>
</body>
<script type="text/javascript">
    var obj = {}
    Object.defineProperty(obj, 'txt', {
        get: function () {
            return obj
        },
        set: function (newValue) {
            document.getElementById('txt').value = newValue
            document.getElementById('show').innerHTML = newValue
        }
    })
    document.addEventListener('keyup', function (e) {
        obj.txt = e.target.value
    })
</script>   
```

## Vue组件间的参数传递
### 父组件与子组件传值
父组件传给子组件： 子组件通过<code>props</code>方法接受数据；  
子组件传给父组件：<code>$emit</code>方法传递参数  
### 非父子组件间的数据传递，兄弟组件传值 
eventBus, 就是创建一个事件中心，相当于中转站，可以用它来传递事件和接收事件。项目比较小时，用这个比较合适。





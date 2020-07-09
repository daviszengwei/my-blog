---
title: Vuex
date: 2020-06-15
tags:
- vuex
categories: 
- vuex
sidebarDepth: 2
---

## 概念性解读

Vuex采用MVC模式中的Model层，规定所有的数据必须通过<code>action——>mutaion——>state</code>这个流程进行来改变状态的。再结合VUE的数据视图双向绑定实现页面的更新。*统一页面状态管理*，可以让复杂的组件交互变得简单清晰，同时在调试时也可以通过DEVtools去查看状态  

在当前前端的spa模块化项目中不可避免的是某些变量需要在全局范围内引用，此时**父子组件的传值**，**子父组件间的传值**，**兄弟组件间的传值**成了我们需要解决的问题。虽然vue中提供了 <code>props(父传子)</code> <code>commit(子传父)</code> 兄弟间也可以用 <code>localstorage</code> 和 <code>sessionstorage</code>。但是这种方式在项目开发中带来的问题比他解决的问题（*难管理，难维护，代码复杂，安全性低*）更多。vuex的诞生也是**为了解决这些问题**，从而大大提高我们vue项目的开发效率。

看一下这个vue响应式的列子，vue中的data、methods、computed，可以实现响应式。

视图通过点击事件，触发methods中 <code>increment</code> 方法，可以更改state中 <code>count</code> 的值，一旦 <code>count</code> 值发生变化，computed中的函数能够把 <code>getCount</code> 更新到视图

```vue
<div id="app">
  <button @click="increment"></button>
  {{getCount}}
</div>

new Vue({
  el: "#app",
  // state
  data() {
    return {
      count: 0
    }
  },
  // view
  computed: {
    gotCount() {
      return this.count
    }
  },
  // actions
  methods: {
    increment () {
      this.count++
    }
  }
})
```
那么vuex和vue这个响应式的列子有什么关系呢？  
**我们可以用vuex实现和vue同样的响应式功能**

其实他们原理上是一样的，vuex中也有四个属性值：state、getters、mutations、actions。

在没有actions的情况下：
- 数据： <code>state --> data</code>
- 获取数据： <code>getters --> computed</code>
- 更改数据：<code>mutations --> methods</code>  

**视图通过点击事件，触发<code>mutations</code>中方法，可以更改<code>state</code>中的数据，一旦<code>state</code>数据发生更改，<code>getters</code>把数据反映到视图。**
那么actions，可以理解处理异步，而单纯多加的一层。

## 源码分析
### store注入组件install方法
*解答问题：vuex的store是如何注入到组件中的？*  
首先使用vuex，需要安装插件：
```vue
Vue.use(Vuex); //vue的插件机制，安装vuex插件
```
当use（Vuex）时候，会调用vuex中的install方法，装在vuex！  
下面是install的核心源码：
```vue
Vue.mixin({
  beforeCreate() {
    if (this.$options && this.$options.store) {
      // 找到根组件 main 上面挂一个$store
      this.$store = this.$options.store
    } else {
      // 非根组件指向其父组件的$store
      this.$store = this.$parent && this.$parent.$store
    }
  }
})
```
可见，store注入vue的实例组件的方式，是通过vue的mixin机制，借助vue组件的生命周期钩子<code>beforeCreate</code> 完成的。即每个vue组件实例化过程中，会在<code>beforeCreate</code>钩子调用vuexInit方法。


## 原理总结
Vuex是通过全局注入store对象，来实现组件间的状态共享。在大型复杂的项目中（多级组件嵌套），需要实现一个组件更改某个数据，多个组件自动获取更改后的数据进行业务逻辑处理，这时候使用vuex比较合适。假如只是多个组件间传递数据，使用vuex未免有点大材小用，其实只用使用组件间常用的通信方法即可。
---
title: vue的路由实现
date: 2020-06-16
tags:
- vue-router
categories: 
- vue
sidebarDepth: 2
---

## VUE的路由实现：hash模式和history模式

**hash模式：** 在浏览器中符号“#”，#以及#后面的字符称之为hash，用window.location.hash读取；  
*特点*：hash虽然在URL中，但不被包括在HTTP请求中；用来指导浏览器动作，对服务器安全无用，hash不会重加载页面。  
hash模式下，仅hash符号之前的内容会被包含在请求中，如<code>http://www.xxx.com</code>，因此对于后端来说，即使没有做到对路由的全覆盖，也不会返回404错误。

**history模式**：history采用HTML5的新特性；且提供了两个新方法：<code>pushState(), replaceState()</code>可以对浏览器历史记录栈进行修改，以及<code>popState</code>事件的监听到状态变更。  
history模式下，前端URL必须和实际向后端发起请求的URL一致，如<code>http://www.xxx.com/items/id</code>。后端如果缺少对<code>/items/id</code>的路由处理，将返回404错误。**Vue-Router官网里如此描述：**"不过这种模式要玩好，还需要后台配置支持...所以呢，你要在服务端增加一个覆盖所有情况的候选资源：如果URL匹配不到任何静态资源，则应该返回同一个<code>index.html</code>页面，这个页面就是你APP依赖的页面。"

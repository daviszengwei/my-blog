---
title: 跨域解决方案
date: 2020-07-27
tags: 
- js
categories: 
- 基础
sidebarDepth: 2
---

## 什么是跨域？
跨域是指**一个域下的文档或脚本试图去请求另一个域下的资源**，这里跨域是广义的。

其实我们通常所说的跨域是狭义的，是由浏览器同源策略限制的一类请求场景。

### 什么是同源策略？
同源策略/SOP(Same origin policy)是一种约定，由Netscape公司1995年引入浏览器，它是**浏览器最核心最基本的安全功能**，如果缺少了同源策略，浏览器很容易受到**XSS、CSFR**等攻击。所谓同源是**指"协议+域名+端口"三者相同**，即便两个不同的域名同一个IP地址，也非同源。

同源策略限制以下几种行为：  
::: warning
1. `Cookie、LocalStorage`和`IndexDB`无法读取
2. `DOM`和`JS`对象无法获得
3. `AJAX`请求不能发送
:::

## 跨域解决方案
::: tip
1. 通过`jsonp`跨域
2. `document.domain + iframe`跨域
3. `location.hash + iframe`
4. `window.name + iframe`跨域
5. `postMessage`跨域
6. 跨域资源共享（CORS）
7. `nginx`代理跨域
8. `nodejs`中间件代理跨域
9. `WebSocket`协议跨域
:::

**这里挑几个用的多的讲**

## 通过jsonp跨域
通常为了减轻web服务器的负载，我们把js、css、img等静态资源分离到另一台独立域名的服务器上，在HTML页面中再通过相应的标签从不同域名下加载静态资源，而被浏览器允许，基于此原理，我们通过动态创建script，再请求一个带参网址实现跨域通信。

1. 原生实现：
```javascript
<script>
    var script = document.createElement('script');
    script.type = 'text/javascript';

    // 传参一个回调函数名给后端，方便后端返回时执行这个在前端定义的回调函数
    script.src = 'http://www.domain2.com:8080/login?user=admin&callback=handleCallback';
    document.head.appendChild(script);

    // 回调执行函数
    function handleCallback(res) {
        alert(JSON.stringify(res));
    }
 </script>
```

服务端返回如下（返回时即执行全局函数）：

```javascript
handleCallback({"success": true, "user": "admin"})
```

2. jquery Ajax实现：

```javascript
$.ajax({
    url: 'http://www.domain2.com:8080/login',
    type: 'get',
    dataType: 'jsonp',  // 请求方式为jsonp
    jsonpCallback: "handleCallback",  // 自定义回调函数名
    data: {}
});
```

3. Vue axios 实现：

```js
this.$http = axios;
this.$http.jsonp('http://www.domain2.com:8080/login', {
    params: {},
    jsonp: 'handleCallback'
}).then((res) => {
    console.log(res); 
})
```

后端node.js代码：
```js
var querystring = require('querystring');
var http = require('http');
var server = http.createServer();

server.on('request', function(req, res) {
    var params = querystring.parse(req.url.split('?')[1]);
    var fn = params.callback;

    // jsonp返回设置
    res.writeHead(200, { 'Content-Type': 'text/javascript' });
    res.write(fn + '(' + JSON.stringify(params) + ')');

    res.end();
});

server.listen('8080');
console.log('Server is running at port 8080...');
```

::: danger
**jsonp的缺点：只能发送get一种请求。**
:::

## nginx代理跨域
Nginx代理跨域，实质和CORS跨域原理一样，通过配置文件设置请求响应头`Access-Control-Allow-Origin`…等字段

1. nginx配置解决iconfont跨域

浏览器跨域访问js,css,img等常规静态资源被同源策略许可，但iconfont字体文件例外，此时可在Nginx的静态资源服务器中加入以下配置

```js
location / {
  add_header Access-Control-Allow-Origin *;
}
```

2. nginx反向代理接口跨域  
::: tip
跨域问题：同源策略仅是针对浏览器的安全策略。服务器端调用http接口只是使用HTTP协议，不需要同源策略，也就不存在跨域问题。
:::

实现思路：通过Nginx配置一个代理服务器域名与domain1相同，端口不同做跳板机，反向代理访问domain2接口，并且可以顺便修改cookie中domain信息，方便当前域cookie写入，实现跨域访问。

Nginx具体配置：
```js
#proxy服务器
server {
    listen       81;
    server_name  www.domain1.com;

    location / {
        proxy_pass   http://www.domain2.com:8080;  #反向代理
        proxy_cookie_domain www.domain2.com www.domain1.com; #修改cookie里域名
        index  index.html index.htm;

        # 当用webpack-dev-server等中间件代理接口访问nignx时，此时无浏览器参与，故没有同源限制，下面的跨域配置可不启用
        add_header Access-Control-Allow-Origin http://www.domain1.com;  #当前端只跨域不带cookie时，可为*
        add_header Access-Control-Allow-Credentials true;
    }
}
```

## nodejs中间件代理跨域
node中间件实现跨域代理，原理大致与Nginx相同，都是通过启一个代理服务，实现数据的转发，也可以通过设置cookieDomainRewrite参数修改相应头中cookie中域名，实现当前域的cookie写入，方便接口登录认证。

1. 非`vue`框架的跨域  
**使用node + express + http-proxy-middleware 搭建一proxy服务器**

- 前端代码：

```js
var xhr = new XMLHttpRequest();

// 前端开关：浏览器是否读写cookie
xhr.withCredentials = true;

// 访问http-proxy-middleware代理服务器
xhr.open('get', 'http://www.domain1.com:3000/login?user=admin', true);
xhr.send();
```

- 中间件服务器代码：

```js
var express = require('express');
var proxy = require('http-proxy-middleware');
var app = express();

app.use('/', proxy({
    // 代理跨域目标接口
    target: 'http://www.domain2.com:8080',
    changeOrigin: true,

    // 修改响应头信息，实现跨域并允许带cookie
    onProxyRes: function(proxyRes, req, res) {
        res.header('Access-Control-Allow-Origin', 'http://www.domain1.com');
        res.header('Access-Control-Allow-Credentials', 'true');
    },

    // 修改响应信息中的cookie域名
    cookieDomainRewrite: 'www.domain1.com'  // 可以为false，表示不修改
}));

app.listen(3000);
console.log('Proxy server is listen at port 3000...');
```

2. `vue`框架的跨域  
`node + vue + webpack + webpack-dev-server`搭建的项目，跨域请求接口，直接修改`webpack.config.js`配置。开发环境下，vue渲染服务和接口代理服务都是webpack-dev-server同一个，所以页面与代理接口之间不再跨域。

webpack.config.js部分配置：

```js
module.exports = {
  entry: {},
  module: {},
  ...
  devServer: {
    historyApiFallback: true,
    proxy: [{
      context: '/login',
      target: 'http://www.domain2.com:8080', //代理跨域目标接口
      changeOrigin: true,
      secure: false, //当代理某些https服务报错时用
      cookieDomainRewrite: 'www.domain1.com' //可以为false，表示不修改
    }],
    noInfo: true
  }
}
```
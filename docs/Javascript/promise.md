---
  title: 慢谈Promise
  date: 2020-06-09
  tags: 
  - javascript
  categories: 
  - javascript
  - promise
  sidebarDepth: 2
---

# JavaScript 深度剖析

## 慢谈Promise  

  ### 深入理解promise  

  **Promise是什么**
  从字面上来看，Promise就是一个承诺。那么，在ES6中，Promise通常用来控制异步操作。当一个异步操作成功的时候，触发.then的操作。当一个异步操作不成功的时候，触发.catch操作。  

  **Promise能帮我们解决什么**  
  JavaScript实现异步执行，在Promise未出现前，我们通常是使用嵌套的回调函数来解决的。但是使用回调函数来解决异步问题，简单还好说，但是问题比较复杂的，就会面临回调金字塔的问题

  ```javascript
  var a = function() {
    console.log('a');
  };

  var b = function() {
      console.log('b');
  };

  var c = function() {
      for(var i=0;i<100;i++){
          console.log('c')
      }  
  };

  a(b(c()));    // 100个c -> b -> a
  ```
  我们要桉顺序的执行a，b，c三个函数，我们发现嵌套回调函数确实可以实现异步操作（在c函数中循环100次，发现确实是先输出100个c，然后在输出b，最后是a）。但是你发现没这种实现*可读性极差*，如果是几十上百且回调函数异常复杂，那么代码维护起来将更加麻烦。

  那么，接下来我们看一下使用 <code>promise</code>（promise的实例可以传入两个参数表示两个状态的回调函数，第一个是 <code>resolve</code>，必选参数；第二个是 <code>reject</code> ，可选参数）的方便之处。

  ```javascript
  var promise = new Promise(function(resolve, reject) {
    console.log('......');
    resolve();  //这是promise的一个机制，只有promise实例的状态变为resolved，才会触发then回调函数
  })

  promise.then(function() {
    for(var i = 0; i < 100; i++) {
      console.log('c')
    }
  })
  .then(function() {
    console.log('b');
  })
  .then(function() {
    console.log('c');
  })
  ```

  **promise的3种状态**

  上面提到了promise的 <code>resolved</code> 状态，那么我们就来说promise的3种状态，未完成（unfulfilled）、完成（fulfilled）、失败（failed）。  
  在promise中我们使用resolved代表fulfilled，使用rejected表示failed。

  **ES6的Promise有哪些特性**

  1.<code>promise</code> 的状态只能从 <code>未完成-->完成</code>，<code>未完成-->失败</code> 且状态不可逆转。  
  2.<code>promise</code> 的异步结果，只能在完成状态时才能返回，而且我们在开发中是根据结果来选择状态的，然后根据状态来选择是否执行<code>then()</code>。  
  3.实例化的 <code>promise</code> 内部会立即执行，<code>then</code> 方法中的**异步回调**函数会在**脚本中所有同步任务完成时才会执行**。因此，<code>promise</code> 的异步回调结果最后输出。实例代码如下：

  ```javascript
  var promise = new Promise(function(resolve, reject) {
    console.log('Promise instance');
    resolve();
  })

  promise.then(function() {
    console.log('resolved result');
  });

  for(var i = 0; i < 100; i++) {
    console.log(i)
  }

  /*
   Promise instance
   1
   2
   3
   ...
   99
   100
   resolved result
   */
  ```
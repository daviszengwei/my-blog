---
  title: 慢谈Promise
  date: 2020-06-09
  tags: 
  - javascript
  categories: 
  - promise
  - async
  - await
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

## async, await, promise
  async、await、promise三者是es6新增的关键字，async-await是建立在promise机制之上的，并不能取代其地位。

  async：作为一个关键字*放在函数前面*，用于表示函数是一个异步函数，因为async就是异步的异步，异步函数也就是意味着**这个函数的执行不会阻塞后面代码的执行**。

  async基本语法：
  ```javascript
  async function func() {
    ------
  }
  func();
  ```
  async表示函数异步，定义的函数会返回一个promise对象，可以使用then方法添加回调函数。

  ```javascript
  async function demo() {
    return 'hello async!';
  }
  console.log(demo());
  demo().then((data) => {
    console.log(data);
  });
  console.log('first exec')
  /*
  若 async 定义的函数有返回值，return 'hello async!';
  相当于Promise.resolve('hello async!'),没有声明式的 return则相当于执行了Promise.resolve();
  Promise { 'hello async!' }
  first exec
  hello async!
  */
  ```
  如果async内部发生错误，使用throw抛出，catch捕获

  ```javascript
  async function demo(flag) {
    if(flag) {
      return 'hello world!';
    } else {
      throw 'happend err!'
    }
  }
  demo(0).then((val) => {
    console.log(val);
  }).catch((val) => {
    console.log(val)
  });
  console.log('first exec');
  /*
  first exec
  happend err!
  */
  ```
  await: 是等待的意思，那么它在等待什么呢，它后面跟着什么呢？  
  其实它后面可以放任何表达式，不过我们更多的是放一个promise对象的表达式。  
  *注意：* await关键字，**只能放在async函数里面，不能单独使用**。

  ```javascript
  async function Func() {
    await Math.random();
  }
  Func();
  /*
  SyntaxError: await is only valid in async function
  */
  ```
  await后面可以跟任何的js表达式。虽然说await可以等很多类型的东西，但是**它主要的意图是用来等待promise对象的状态被resolved**。  
  如果await的是promise对象会造成异步函数停止执行那就等待promise的解决；  
  如果等的是正常的表达式则立即执行。

  ```javascript
  function demo() {
    return new Promise((resolve, reject) => {
      resolve('hello promise!');
    });
  }
  (async function exec() {
    let res = await demo();
    console.log(res)
  })();
  // hello promise
  ```

promise:对象用于表示一个异步操作的最终状态（完成或失败），以及该异步操作的结果值。它有三种状态：pending，resolved，rejected
- 1、 promise从pending状态改为resolved或rejected状态只有一次，一旦变成resolve或rejected之后，这个promise的状态就再也不会改变了。
- 2、 通过resolve(value)传入的value可以是任何值，null也可以，它会传递给后面的then方法里的function去使用。通过rejected(err)传入的err理论上也是没有限制类型的，但我们一般都会传入一个error，比如reject(new Error('Error'))

**await若等待的是promise就会停止下来**

业务场景：  
  有三个异步请求需要发送，相互没有关联，只是需要当请求都结束后将界面的loading清除掉即可。

```javascript
function sleep(second) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resoleve("request done!" + second + Math.random())
    },second)
  })
}

async function bugDemo() {
  console.log(await sleep(2000));
  console.log(await sleep(3000));
  console.log(await sleep(1000));
  console.log("clear the loading~")
}

bugDemo()
console.log("second part")
/*
> "second part"
> "request done! 20000.474574137382767"
> "request done! 30000.04033940416849768"
> "request done! 10000.36716999803500894"
> "clear the loading~"
*/
```

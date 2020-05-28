---
title: JavaScript工作原理：事件循环
tags:
  - JavaScript
url: 275.html
id: 275
categories:
  - node.js
date: 2019-06-24 21:19:22
---

单线程异步
-----

js早期对性能无要求，与此一开始就采用了单线程的选择，好处显而易见是可以非常清晰的确定执行顺序，如果多线程则很难判断在需要某个值时其他是否处理完毕。 虽然后来引入了web service，但其主线程一直是单线程的，并且主线程的变量不得操作dom，辅助线程仍然受主线程控制，相对于对主线程进行辅助计算。 引入异步是，避免某个延时任务（如定时器）阻塞了整个进场，使得加载和渲染无法继续下去，如果没有异步的话，在单线程的js中一旦延时，那么必须要等到这个定时器执行完才可以继续进行渲染或者其他操作，这在客户端基本是不可容忍的。

执行栈
---

执行栈认为是一个存储函数调用的栈结构，遵循先进后出的原则 ![](http://img.bugzhang.com/1670d2d20ead32ec.gif) js代码的执行过程就是往执行栈里放入函数然后先进后出的执行。

任务队列
----

JavaScript将任务分为两类：同步任务和异步任务。同步任务会直接进入调用栈先进后出的执行，而异步任务则会放入任务队列，一旦执行栈为空，EventLoop会从任务队列中取出应该被执行的任务放入执行栈执行，**本质上异步代码也是同步执行的**。 ![](http://img.bugzhang.com/16740fa4cd9c6937.webp) 将异步任务又分为了两类：宏任务（macrotask）和微任务（microtask）。执行顺如如下： 1.首先执行同步代码，这属于宏任务 2.当执行完所有同步代码后，执行栈为空，查询是否有异步代码需要执行 3.执行所有微任务 4.当执行完所有微任务后，如有必要会渲染页面 5.然后开始下一轮 Event Loop，执行宏任务中的异步代码，也就是 setTimeout 中的回调函数 微任务包括 process.nextTick ，promise ，MutationObserver。 宏任务包括 script ， setTimeout ，setInterval ，setImmediate ，I/O ，UI rendering。 一个常考是示例：
```js
    console.log('script start')

    async function async1() {
      await async2()
      console.log('async1 end')
    }
    async function async2() {
      console.log('async2 end')
    }
    async1()

    setTimeout(function() {
      console.log('setTimeout')
    }, 0)

    new Promise(resolve => {
      console.log('Promise')
      resolve()
    })
      .then(function() {
        console.log('promise1')
      })
      .then(function() {
        console.log('promise2')
      })

    console.log('script end')

    // 执行顺序是：
    // script start
    // async2 end
    // Promise
    // script end
    // promise1
    // promise2
    // async1 end
    // setTimeout
```

node.js中的事件循环
-------------

node.js的异步机制是采用了libuv这个c++的异步库实现的，因此其时间循环就是libuv中规定是循环——分为六个阶段执行：

       ┌───────────────────────────┐
    ┌─>│           timers          │
    │  └─────────────┬─────────────┘
    │  ┌─────────────┴─────────────┐
    │  │     pending callbacks     │
    │  └─────────────┬─────────────┘
    │  ┌─────────────┴─────────────┐
    │  │       idle, prepare       │
    │  └─────────────┬─────────────┘      ┌───────────────┐
    │  ┌─────────────┴─────────────┐      │   incoming:   │
    │  │           poll            │<─────┤  connections, │
    │  └─────────────┬─────────────┘      │   data, etc.  │
    │  ┌─────────────┴─────────────┐      └───────────────┘
    │  │           check           │
    │  └─────────────┬─────────────┘
    │  ┌─────────────┴─────────────┐
    └──┤      close callbacks      │
       └───────────────────────────┘


*   **timers**
    *   检查 timer 队列是否有到期的 timer 回调，如果有，将到期的 timer 回调按照 timerId 升序执行。
    *   检查是否有 process.nextTick 任务，如果有，全部执行。
    *   检查是否有microtask，如果有，全部执行。
*   **pending callbacks**
    *   检查是否有 pending 的 I/O 回调。如果有，执行回调。如果没有，退出该阶段。
    *   检查是否有 process.nextTick 任务，如果有，全部执行。
    *   检查是否有microtask，如果有，全部执行。
*   **idle, prepare**
    *   libuv内部为poll做准备工作。
*   **poll**
    *   首先检查是否存在尚未完成的回调。如果有未完成回调：
    *   如果有未完成回调。
        *   执行所有可用回调。
        *   检查是否有 process.nextTick 回调，如果有，全部执行。
        *   检查是否有微任务，如果有，全部执行。
    *   如果没有未完成回调
        *   检查是否有 immediate 回调，如果有，退出 poll 阶段。如果没有，阻塞在此阶段，等待新的事件通知重启循环。
    *   如果不存在尚未完成的回调，退出poll阶段。
*   **check**
    *   如果有immediate回调，则执行所有immediate回调。
    *   检查是否有 process.nextTick 回调，如果有，全部执行。
    *   检查是否有 microtaks，如果有，全部执行。
*   **close callbacks**
    *   如果有immediate回调，则执行所有immediate回调。
    *   检查是否有 process.nextTick 回调，如果有，全部执行。
    *   检查是否有 microtaks，如果有，全部执行。

上述过程比较复杂，对于一个实际的问题而言，自从node.js升级到10后，执行顺序其实是和游览器端一致的，所以不需要非常详细的知道上述过程，安装与游览器游览器端的去编码即可。
---
title: 使用Web Worker
tags:
  - 前端优化
url: 42.html
id: 42
categories:
  - 前端
date: 2017-06-28 22:48:00
---

![](http://7xqgks.com1.z0.glb.clouddn.com/head-0022.jpg)

> js作为单线程的语言，无法同时运行多个处理程序，但面对一些不得不耗费资源才能完成的任务时，单线程弊端就显现出来，为了解决这个问题，Wob Worker应身而出：“Worker”是指执行代码的并行线程，可以在一个不与用户交互的独立环境中，就行一些耗资源任务的处理，与游览器文档中的js相当于多线程，比如图像处理等任务。Web Worker有效解决了js的缺点，同时是提升性能的一个方法之一：将耗资源的代码在worker中执行。

# worker对象
```js
    //创建worker对象并执行
    var worker = new Worker("js/decrypt.js");

    //添加事件处理程序，完成后给主线程发送信息时执行
    worker.onmessage = function(e){
        console.log('finash');
    }

    //发送信息给worker
    worker.postMessage('file.txt');

    //注册用来接收来自主线程信息的处理程序
    worker.onmessage = function(e){
        //从事件对象获取消息
        var valueToDecrypt = e.data;
        //进行操作
        console.log('end');
        //返回给主线程
        postMessage(decryptedValue);
    }
```

# Worker执行模型与作用域

Worker线程从上到下执行代码与文件，如果Worker注册了onmessage事件，只要message事件可能触发，将永远不会退出。 一个Worker对象是运行在完全隔离的环境中，其作用域大于主线程所在的JavaScript全局对象，小于整个客户端的Window对象。
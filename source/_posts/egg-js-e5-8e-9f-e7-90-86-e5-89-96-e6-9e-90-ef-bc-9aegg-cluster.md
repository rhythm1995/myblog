---
title: egg.js原理剖析：egg-cluster
tags:
  - egg.js
  - node.js
url: 331.html
id: 331
comments: false
categories:
  - node.js
date: 2019-10-19 16:51:01
---

本文主要是egg-cluster的原理理解，包括了一些前置知识：操作系统进程同学、node创建进程、node进程通信。我对这些前置知识只会略讲，如果看到后没有任何印象或者之前不掌握，请阅读本文最后参考文档中的文章来熟悉这些前置知识再阅读本文。

node-cluster 简介
===============

「cluster」意为“集群”，顾名思义，就是增加node.js这种单进程单线程的有效利用多核系统的能力。 「cluster」 模块可以创建共享服务器端口的子进程。

node.js 进程创建方式
==============

通信方式与进程产生方式有关，而Node有4种创建进程的方式： spawn() ， exec() ， execFile() 和 fork()。

*   spawn：spawn() 方法默认不会创建shell去执行传入的命令，性能相对较好
*   exec：exec() 方法会创建一个shell，完全支持shell语法 ，可以直接传入任意shell脚本，故可以调用shell脚本
*   execFile：execFile() 相比 exec() 是不会通过shell，而是接受一个可执行文件
*   fork：fork() 是 spawn() 的变体，用来创建Node进程，最大的特点是父子进程自带通信机制——管道

操作系统 进程通信
=========

*   共享内存：不同进程共享同一段内存空间。通常还需要引入信号量机制，来实现同步与互斥。这种方式性能极高，但实现困难。
*   消息传递：这种模式下，进程间通过发送、接收消息来实现信息的同步。
*   信号量：信号量简单说就是系统赋予进程的一个状态值，未得到控制权的进程会在特定地方被强迫停下来，等待可以继续进行的信号到来。如果信号量只有 0 或者 1 两个值的话，又被称作“互斥锁”。这个机制也被广泛用于各种编程模式中。
*   管道：管道本身也是一个进程，它用于连接两个进程，将一个进程的输出作为另一个进程的输入。可以用 pipe 系统调用来创建管道。我们经常用的“ | ”命令行就是利用了管道机制。
*   socket：网络通信，不仅能跨进程（端口与端口），还能跨机器（IP与IP）

node.js 进程通信
============

*   通过stdin/stdout传递消息：属于操作系统进程通信方式的消息传递法。拿到子进程的handle后，可以访问其 stdio 流，然后约定一种 message 格式通信。
*   IPC管道：默认通信方式，父进程与process.on('message') 收 | child.send() 发，子进程process.on('message') 收 | process.send() 发，和
*   socket：借助socket网络通信，不仅能跨进程（端口与端口），还能跨机器（IP与IP）。

node-cluster 工作原理
=================

通常，一个集群由主从架构组成，就是一个「master」主进程和多个「worker」子进程来组成一个集群。 工作进程由「child_process.fork」方法创建，这些「worker」可以用IPC管道和父进程通信。子进程和父进程具有相同的代码段、数据段、堆栈，但是它们的内存空间不共享。父进程「master」负责监听端口，接收到新的请求派发给下面的「worker」进程。

egg-cluster 架构模型
================

egg-cluster把进程分为三类： master：主进程，负责 agent 的启动、退出、重启；各个 worker 进程调度；负责 agent 和各个 worker 之间的通信；负责各个 worker 之间的通信 agent：「master」 的子进程，可以理解为「master」的“助理”，帮「master」打杂做一些派发之类的工作：处理公共资源的访问，如文件监听，或者帮 「worker」 处理一些公共事务，如一些事情是不需要每个 「worker」 都做一次的，「agent」 帮忙做完之后通知它们执行之后的操作 worker：「master」 的子进程，一般是根据服务器有多少个 CPU 启动多少个这样的 「worker」 进程，主要用于对外服务，处理各种业务层面的事情

进程顺序
====

*   master 启动后先启动 agent 进程
*   agent 初始化成功后，通过 IPC 通道通知 master
*   master 根据 CPU 的个数启动相同数目的 worker 进程
*   worker 进程初始化成功后，通过 IPC 通道通知 master
*   所有的进程初始化成功后，master 通知 agent 和各个 worker 进程应用启动成功

进程通信
====

master 和 agent/worker 是 直接通信的，因为 master 是 agent/worker fork()后的父进程，拥有IPC管道进行通信 agent 和 worker 之间以及各个 worker 之间是 间接通信的，依靠master做中转 https://github.com/eggjs/egg-cluster#readme

参考：
《现代操作系统》
进程间的五种通信方式介绍： https://blog.csdn.net/wh_sjc/article/details/70283843
node.js的cluster文档：http://nodejs.cn/api/cluster.html
Nodejs进程间通信： https://www.cnblogs.com/rubyxie/articles/8949417.html
egg-cluster：https://github.com/eggjs/egg-cluster#readme
Egg 源码解析之 egg-cluster：https://zhuanlan.zhihu.com/p/29374045#
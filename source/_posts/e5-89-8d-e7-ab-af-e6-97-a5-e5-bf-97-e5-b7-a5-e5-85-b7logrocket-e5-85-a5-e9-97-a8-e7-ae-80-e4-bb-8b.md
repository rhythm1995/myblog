---
title: 前端日志工具logrocket入门简介
tags:
  - 前端
  - 前端工程化
  - 调试
url: 103.html
id: 103
categories:
  - 前端
date: 2018-01-30 21:45:25
---

![](http://7xqgks.com1.z0.glb.clouddn.com/head-0067.jpg)

工具简介
====

LogRocket 是一个前端日志工具，它可以让你像发生在自己的浏览器中那样重现问题。无需猜测错误发生的原因，或者要求用户截图以及日志转储，LogRocket 可以让你重现会话以便快速了解发生了什么错误。无需考虑框架，它适用于任何应用程序，也有插件可以从 Redux、Vuex和@ngrx/tore 上记录额外的上下文。 LogRocket通过从应用程序中捕获日志和会话记录来帮助您修复错误并了解用户的行为。 ![](https://files.readme.io/b38e152-developer_hero.png)

开始
==

1.LogRocket开始非常简单，访问https://app.logrocket.com 获取应用程序ID。 2.通过npm安装LogRocket并且在应用中进行集成。

    $ npm i --save logrocket
    
    // Code:
    
    import LogRocket from 'logrocket';
    LogRocket.init('app/id');
    

3.添加相关中间件 可以选择添加三大框架对应的中间件，进一步提高集成度 [redux](https://docs.logrocket.com/docs/redux-middleware) [ngrx](https://docs.logrocket.com/docs/ngrx-middleware) [vuex](https://docs.logrocket.com/docs/vuex-plugin)

原理
==

LogRocket是一个JavaScript SDK，通过拦截网站/应用程序中的操作（如网络请求，触摸操作和Redux事件）并将其发送到远程服务器来工作。 您的应用中的每个用户会话都与唯一的LogRocket标识符关联。这些用户会话可以加入到适当的URL，以集成方式提供给开发人员。

LogRocket能做什么
=============

前端调试一向比较坑爹，LogRocket就是解决这个问题的。具体包括： **1.修复错误** ![](https://files.readme.io/325de23-Group_13.png) LogRocket向您显示影响用户的最高错误，并为您提供每个错误报告的回放与日志。观看回放与日志有助于您了解用户如何触发错误。 网络请求，JavaScript异常和代码错误都在LogRocket中捕获和重放，可以检查原始HTML / CSS以了解和重现UI错误。 **2.集成后端日志记录工具** 在对后端错误进行分类时，可能不清楚为什么前端提出了意外的请求。LogRocket集成了后端日志记录和错误报告工具，以显示每个后端错误和日志条目的相应前端会话日志。 **3.支持度** ![](https://files.readme.io/f891fbf-Group_11.png) 通常情况下，用户上报问题时，并不能给出足够的上下文来判断到底发生了什么。 如果你正在使用一个像Intercom的聊天工具，你可以直接将LogRocket整合进去，从而无论用户何时开始聊天，你都可以插入一条记录链接。 **4.依靠流量并发现用户体验问题** ![](https://files.readme.io/2b33467-Group_14.png) 由于LogRocket记录了一切，因此可以在事实之后智能地发现问题。我们可以根据事件进行搜索和过滤，例如让我们的系统自动检测到用户感到沮丧的时刻——比如愤怒点击了按钮。 LogRocket可以通过一些属性（如用户电子邮件，名称，注册日期，位置，状态，持续时间，页面访问等等）来搜索和过滤会话。如果它发生在应用程序中，可以使用LogRocket搜索来找到它。
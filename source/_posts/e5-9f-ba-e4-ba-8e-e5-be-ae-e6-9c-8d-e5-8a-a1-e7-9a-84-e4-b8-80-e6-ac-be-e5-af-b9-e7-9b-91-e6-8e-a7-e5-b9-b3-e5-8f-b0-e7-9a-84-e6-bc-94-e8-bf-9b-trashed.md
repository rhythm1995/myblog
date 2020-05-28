---
title: 基于前端微服务的对一款监控平台的技术方案改进
tags:
  - node.js
  - 架构
url: 347.html
id: 347
categories:
  - node.js
  - 前端
date: 2019-11-01 12:04:53
---

原始方案
----

![](http://bugzhang.com/wp-content/uploads/2019/11/image.png)

后端：Java框架Spring Boot整合数据库数据，推送给Grafana。 前端：完全依靠Grafana做前端，定制化很弱，无法完成大屏看板等刚需。

改进方案1：完全定制化的前端
--------------

![](http://bugzhang.com/wp-content/uploads/2019/11/image-1.png)

干掉Grafana，完全定制化开发前端，采用Vue框架，以及数据可视化组件库Echarts和大屏组件库iDataV完成业务开发。

改进方案2：完全配置化的前端
--------------

![](http://bugzhang.com/wp-content/uploads/2019/11/image-2.png)

方案1有个明显确定是复用性非常差，以后类似项目又要重头再来，不如Grafana方便。 方案2用Node.js，替代Grafana的后端功能。引入MongoDB存储业务后端推送的数据。

每个页面可配置组件及其组件形式和样式，在运行时依靠配置编译出主题文件并且采用。可推送数据到不同组件实现监控。

改进方案3：基于微服务的对当前方案的改进
--------------------

![](http://bugzhang.com/wp-content/uploads/2019/11/image-3.png)

方案2相比之前，功能非常齐全，可维护性很佳。难度在于实施工期非常长，并且目前业务Grafana基本能支持绝大多数场景，除了大屏展示效果不佳外是勉强可用的。

如果把大屏功能和Grafana数据可视化分离成两个微服务，然后用一个基座整合在一起，大屏功能和方案2一致，而Grafana完整保留，好处在于开发时间大大降低。得到方案3。

# 前端微服务技术可选方案

方式 | 开发成本 |  维护成本 | 可行性 | 同一框架要求 | 实现难度 | 潜在风险 |
- | - | - | - | - | - | - |
路由分发 | 低 | 低 | 高 | 否 | ★ | 不支持多种框架 |
iFrame | 低 | 低 | 高 | 否 | ★ | XSS风险，性能差 |
应用微服务化 | 高 | 低 | 中 | 否 | ★★★★ | 针对每个框架做定制 |
微件化 | 高 | 中 | 低 | 是 | ★★★★★ | 针对构建系统，如 webpack 进行 hack |
微应用化 | 中 | 中 | 高 | 是 | ★★★ | 统一不同应用的构建规范 |
纯 Web Components | 高 | 低 | 高 | 否 | ★★ | 新技术，浏览器的兼容问题 |
结合 Web Components | 高 | 低 | 高 | 否 | ★★ | 新技术，浏览器的兼容问题 |


![](http://bugzhang.com/wp-content/uploads/2019/11/image-4.png)

我们的业务：Y（不限框架） -> N（不需要支持IE） -> N （不是新项目因为有Grafana了）

得到选型：**WebComponent 集成应用的方式**

其他关于web component是什么，怎么用，微服务是什么怎么用的。可以靠看参考文章来了解。

* * *

参考文章：

*   前端微服务：[https://github.com/phodal/microfrontends](https://github.com/phodal/microfrontends)
*   WebComponent入门：[http://www.ruanyifeng.com/blog/2019/08/web_components.html](http://www.ruanyifeng.com/blog/2019/08/web_components.html)
*   vue-cli3中使用**WebComponent**：[https://www.cnblogs.com/Adiodanza/p/9641708.html](https://www.cnblogs.com/Adiodanza/p/9641708.html)
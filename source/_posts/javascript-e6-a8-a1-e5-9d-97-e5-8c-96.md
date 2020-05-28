---
title: JavaScript模块化
tags:
  - 前端工程化
url: 44.html
id: 44
categories:
  - 前端
date: 2017-08-14 22:53:07
---

![](http://7xqgks.com1.z0.glb.clouddn.com/head-0029.jpg)

# 从古代前端说起

早起的前端，是没有工程体系的，基本就是div+css，以及用js做个动画验证个表单之类的工作，因为工程不复杂，变量少，代码重构难度低，开发团队也不大（当时很多前端都是后端甚至设计师来写），所以这门语言一开始对模块化的需求不大，可以统统全局变量，反正没几个。在需要的时候写端脚本然后引入即可，甚至很多js代码是混入html属性中也不足为奇。

# 模块

而对于javascript来说，模块代表就是一个特定函数，这种模式使用了闭包
```js
    var module =
    (function() {
        //私有变量 函数
        ...
        return{
        //公有函数
        ...
        };
    })();
```

这种javascript模块，将内部功能与对外功能划定了清晰的界限。防止了命名冲突。

# CommonJS

CommonJS认为js不应该只是简单脚本语言，应该可以运行在任何地方，因此开始制定一系列api。CommonJS的崛起主要是随着node.js的大热，而被熟知了的。
```js
    //定义模块
    exports.sum = {};

    //使用模块
    var xxx = require('xxx');
    exports.add = function(n){
         return xxx.ss(val,n);
    };
```

这种写法适合服务端，因为在服务器读取模块都是在本地磁盘，加载速度很快。但是如果在客户端，加载模块的时候有可能出现“假死”状况。因为使用时候必须等模块加载完毕。但即使如此，实际还是不能使用的主要原因是少了node中才有的几个全局变量：

*   module
*   exports
*   require
*   global

现在，browserify和webpack，通过commonjs在游览器的实现，可以进行模块打包处理，其中webpack已经成为了前端工程标准的打包工具。

# AMD

AMD，即 (Asynchronous Module Definition)，异步模块定义，require.js应用的是这一规范。先定义所有依赖，然后在加载完成后的回调函数中执行。
```js
    //使用模块
    require([module], callback);
```

# CMD

CMD， Common Module Definition，命令模块定义。实现是淘宝玉伯大神开发的sea.js相关的。 CMD和AMD一样也是异步的，其使用方式是：
```js
    //模块的定义
    define(function(require, exports, module) {
       var xxx = require('xxx');
       xxx.start();
    });
```

AMD和CMD最大的区别是对依赖模块的执行时机处理不同。 CMD对模块的态度是懒执行, 而AMD对模块的态度是预执行。CMD是在需要的时候才会执行，带来的好处就是开发者能理解因为过程更接近与人脑所想，而AMD是所有的模块全部提前，带来结果就是如果不考虑这点就会被坑。可以说CMD通过机器打包的效率提升了编码效率，这可能也是sea.js在webpack但是之前在国内更火的原因。
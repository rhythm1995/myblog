---
title: Zepto源码分析——Zepto核心
tags:
  - zepto.js
  - 源码分析
url: 62.html
id: 62
categories:
  - 前端
date: 2017-09-11 11:16:07
---

![](http://7xqgks.com1.z0.glb.clouddn.com/head-0046.jpg)

前言
--

一直计划想写一写某个框架的源码分析，之前读过jQuery的，但感觉jQuery太大，同时网上研究也已经很多而且很全了，所以选取迷你版jQuery之称的Zepto作为分析对象，边读边写，同时司徒正美大神的《JavaScript框架设计》这本书的对js框架的讲解，来写写自己理解。

目录结构
----

源码下载地址：https://github.com/madrobby/zepto ，源码在src目录下，目录结构如下：

    ├── src
       ├── ajax.js
       ├── amd_layout.js
       ├── assets.js
       ├── callbacks.js
       ├── data.js
       ├── deferred.js
       ├── detect.js
       ├── event.js
       ├── form.js
       ├── fx.js
       ├── fx_methods.js
       ├── gesture.js
       ├── ie.js
       ├── ios3.js
       ├── selector.js
       ├── stack.js
       ├── touch.js
       └── zepto.js


命名空间
----
```js
    var Zepto = (function() {
      return $
    })()

    window.Zepto = Zepto
    window.$ === undefined && (window.$ = Zepto)
```

Zepto借鉴了类似jQuery的挂载全局变量的方式实现，核心是返回$传递给Zepto。然后把Zepto和$作为window的属性。这样不会发现变量冲突，即使冲突只需要修改这两个变量名。 此外补充下，因为很多框架都采用了$作为标配，因此经常会出现$这个变量名冲突的问题，不过一般情况下Zepto是不会和jQuery冲突（这两个只选取一个就足够了），jQuery中的解决方案是：

```js
    //先把可能存在的变量名存放
    var _jQuery = window.jQuery,
        _$ = window.$;

    //没冲突时候放进去
    jQuery.extend({
      noConflict:function(deep){
        window.$ = _$;
        if(deep){
          window.jQuery = _jQuery;
        }
        return jQuery;
      }
    });
```

类型判定
----

这段代码是相对比较简单的，就不再复习一遍JavaScript的类型了。直接上代码：

```js
    function type(obj) {
      return obj == null ? String(obj) :
        class2type[toString.call(obj)] || "object"
    }

    function isFunction(value) {
      return type(value) == "function"
    }

    function isWindow(obj) {
      return obj != null && obj == obj.window
    }

    function isDocument(obj) {
      return obj != null && obj.nodeType == obj.DOCUMENT_NODE
    }

    function isObject(obj) {
      return type(obj) == "object"
    }

    function isPlainObject(obj) {
      return isObject(obj) && !isWindow(obj) && Object.getPrototypeOf(obj) == Object.prototype
    }

    function likeArray(obj) {
      var length = !!obj && 'length' in obj && obj.length,
        type = $.type(obj)

      return 'function' != type && !isWindow(obj) && (
        'array' == type || length === 0 ||
        (typeof length == 'number' && length > 0 && (length - 1) in obj)
      )
    }
```


JavaScript的类型判断有一些坑，如下：
```js
    //type的坑
    typeof null // 'object'
    typeof document.childNodes  //safari 'function'
    typeof document.createElement('embed')  //firefox3-10  'function'
    typeof document.createElement('object')  //firefox3-10  'function'
    typeof document.createElement('applet')  //firefox3-10  'function'
    typeof /\d/i  //  'function'
    typeof window.alert  //ie6-8  'object'
    //跨文档比较原型，会不一致
    var iframe = document.createElement('iframe');
    document.body.appendChild(iframe);
    xArray = window.frames[window.frames.length - 1].Array;
    var arr = new xArray[1,2,3];
    arr instanceof Array;   //false
    arr.constructor === Array;  //false
    //旧版本AE中DOM和BOM对象的constructor不存在
    window.onload = function(){
        alert(window.constructor);  //ie67 undefined
        alert(document.constructor);  //ie67 undefined
        alert(document.body.constructor);  //ie67 undefined
        alert((new ActiveXObject('Microsoft.XMLHTTP')).constructor);  //ie6-9 undefined
    }
    //isNaN会把字符串、数组放回去返回true
    isNaN('aaa')  //true
```

框架入口
----
```js
      zepto.init = function(selector, context) {
        var dom
        // 如果没有选择器，返回一个空Zepto对象
        if (!selector) return zepto.Z()
        // 如果是字符串类型的选择器
        else if (typeof selector == 'string') {
          // 去除收尾空白符
          selector = selector.trim()
          // 如果传入的字符串是以<开头且符合HTML代码规则，创建元素
          if (selector[0] == '<' && fragmentRE.test(selector))
            dom = zepto.fragment(selector, RegExp.$1, context), selector = null
          // 如果不是规范但又有内容，在指定位置查找
          else if (context !== undefined) return $(context).find(selector)
          // 如果是CSS选择器，调用CSS选择器查找
          else dom = zepto.qsa(document, selector)
        }
        // 如果是函数，当dom创建时调用
        else if (isFunction(selector)) return $(document).ready(selector)
        // 如果是一个Zepto对象，直接返回这个对象
        else if (zepto.isZ(selector)) return selector
        else {
          // 如果是数组，转成类数组对象
          if (isArray(selector)) dom = compact(selector)
          // 如果是一个对象，将其包括到一个数组中
          else if (isObject(selector))
            dom = [selector], selector = null
          // 如果是HTML片段，以此创建节点
          else if (fragmentRE.test(selector))
            dom = zepto.fragment(selector.trim(), RegExp.$1, context), selector = null
          // 如果有context参数，在context上创建
          else if (context !== undefined) return $(context).find(selector)
          // 如果是CSS选择器，调用CSS选择器查找
          else dom = zepto.qsa(document, selector)
        }
        // 从查找到的节点创建一个Zepto对象
        return zepto.Z(dom, selector)
      }
```

一个基本概念就是Zepto对象，Zepto是一个类数组对象，它具有链式方法来操作它指向的DOM节点，除了$(Zepto)对象上的直接方法外(如$.extend)，文档对象中的所有方法都是集合方法。 调用Zepto框架的方法，都是作用于Zepto()对象的，init函数所做的工作就是讲正常的DOM对象转换成具有Zepto方法的Zepto对象。

对象拓展
----

对象拓展是一种机制，用来把新功能添加到命名空间上。实质就是深浅拷贝。 `javascript function extend(target, source, deep) { for (key in source) if (deep && (isPlainObject(source[key]) || isArray(source[key]))) { if (isPlainObject(source[key]) && !isPlainObject(target[key])) target[key] = {} if (isArray(source[key]) && !isArray(target[key])) target[key] = [] extend(target[key], source[key], deep) } else if (source[key] !== undefined) target[key] = source[key] }`
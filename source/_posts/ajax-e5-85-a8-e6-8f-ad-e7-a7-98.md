---
title: ajax全揭秘
tags:
  - JavaScript
url: 250.html
id: 250
categories:
  - HTTP
  - 前端
date: 2019-04-20 14:28:21
---

> 项目中一直都使用ajax的封装库，时间久了难免忘记，前不久看到公司笔试题有实现promise的ajax，居然想不出来完整的。所以把ajax复习一遍，并且封装下。文章会花很大篇幅讲对象基础和原理，如果直接库封装看实现的可以看最后两小节。

XMLHttpRequest
==============

XMLHttpRequest是用于与后台交换数据的对象，XMLHttpRequest不仅仅能传递XML而是允许任何类型的数据传输。 所有现代的浏览器都支持 XMLHttpRequest 对象。 XMLHttpRequest是ajax的实现基础。 XMLHttpRequest 继承了 XMLHttpRequestEventTarget，XMLHttpRequestEventTarget继承了EventTarget。 简单说明下这两个父对象的属性方法

# EventTarget

|方法 | 功能 |
| - | - |
| addEventListener() | 注册特定事件类型的事件处理程序 |
| removeEventListener() | 删除事件侦听器 |
| dispatchEvent() | 将事件分派到此EventTarget |


EventTarget的简单实现
```js
    class EventTarget {
      constructor() {
        this.listeners = {};
      }

      addEventListener(type, callback) {
        if(!(type in this.listeners)) {
          this.listeners[type] = [];
        }
        this.listeners[type].push(callback);
      }

      removeEventListener(type, callback) {
        if(!(type in this.listeners)) {
          return;
        }
        let stack = this.listeners[type];
        for (let i = 0, l = stack.length; i < l; i++) {
          if(stack[i] === callback) {
            stack.splice(i, 1);
            return this.removeEventListener(type, callback);
          }
        }
      }

      dispatchEvent(event) {
        if(!(event.type in this.listeners)) {
          return;
        }
        var stack = this.listeners[event.type];
        event.target = this;
        for (let i = 0, l = stack.length; i < 1; i++) {
          stack[i].call(this. event);
        }
      }
    }
```

# XMLHttpRequestEventTarget

XMLHttpRequestEventTarget是一个描述事件处理程序的接口

| 属性 | 功能 |
| - | - |
| onabort | 当请求失败时调用该方法 |
| onerror | 当请求发生错误时调用该方法 |
| onload | 当一个HTTP请求正确加载出内容后返回时调用 |
| onloadstart | 当一个HTTP请求开始加载数据时调用 |
| onprogress | 间歇调用该方法用来获取请求过程中的信息 |
| ontimeout | 当时间超时（设置了timeout）时调用 |
| onloadend | 当内容加载完成（不管成功与否）时调用 |


# XMLHttpRequest

| 属性 | 功能 |
| - | - |
| onreadystatechange | 当readyState属性发生变化时调用的EventHandler |
| readyState | 请求的状态码 |
| responseType | 响应数据的类型 |
| responseURL | 响应的序列化URL |
| responseXML | 一个Document，其中包含该请求的响应 |
| status | 返回无符号短整型请求响应状态 |
| statusText | 返回字符串型请求响应状态和信息 |
| timeout | 超时时间 |
| upload | 上传过程 |
| withCredentials | 用来指定跨域的请求是否应该使用证书 |
| withCredentials | 用来指定跨域的请求是否应该使用证书 |
| withCredentials | 用来指定跨域的请求是否应该使用证书 |
| abort() | 终止请求 |
| getAllResponseHeaders() | 以字符串的形式返回所有用CRLF分隔的响应头 |
| getResponseHeader() | 返回包含指定响应头的字符串 |
| open() | 初始化一个请求 |
| send() | 发送请求 |
| setRequestHeader() | 设置HTTP请求头的值 |


## readyState的取值

| 值 | 属性 | 功能 |
| - | - | - |
| 0 | UNSENT | 代理被创建，但尚未调用 open() 方法 |
| 1 | OPENED | open() 方法已经被调用 |
| 2 | HEADERS_RECEIVED | send() 方法已经被调用，并且头部和状态已经可获得 |
| 3 | LOADING | 下载中； responseText 属性已经包含部分数据 |
| 4 | DONE | 下载操作已完成 |


封装简单的Ajax方法
===========

大概流程是：

1.通过兼容性判断创建XMLHttpRequest对象还是ActiveXObject对象

2.设置请求头，区分get与其他方法，调用open()，send()方法建立连接

3.每当 readyState 改变时，就会触发 onreadystatechange 事件。当 readyState 是

4 并且 status 是介于200（含）与400（不包含）之间时，可以获取到responseText。

5.用sent()方法发送请求 首先定义一下入参data


| 参数 | 类型 | 说明 |
| - | - | - |
| option | string | 请求方法 |
| url | string | 请求地址 |
| body | object | 请求参数 |

```js
    function ajax(data){
        // 创建对象
        var xhr = null;
        var params = formsParams(data.body);
        if(window.XMLHttpRequest){
            xhr = new XMLHttpRequest()
        } else {
            xhr = new ActiveXObject("Microsoft.XMLHTTP");
        }
        // 设置请求头，建立连接
        if(data.option == "GET"){
            xhr.open(data.option, data.url + "?"+ params,data.async);
            xhr.send(null)
        } else if(data.option !== "GET"){
            xhr.open(data.option,data.url,data.async);
            xhr.setRequestHeader("Content-Type","application/x-www-form-urlencoded");
            xhr.send(params);
        }
        // onreadystatechange监听并相应
        xhr.onreadystatechange = function(){
            if(xhr.readyState == 4 && xhr.status >= 200 && xhr.status < 400) {
              data.success(xhr.responseText);
            }
        }
        function formsParams(data){
            var arr = [];
            for(var prop in data){
                arr.push(prop + "=" + data[prop]);
            }
            return arr.join("&");
        }
```


实现Promise版的Ajax方法
=================

其主要是返回一个Promise对象，把回调版的ajax的逻辑写入Promise中。
```js
    let ajax = ({url = '', option = 'get', body = {} }) => {
      return new Promise((resolve, reject) => {
        const xhr = new XMLHttpRequest()
        xhr.open(option, url, true);
        xhr.onreadystatechange = () => {
          if(xhr.readyState == 4 && xhr.status >= 200 && xhr.status < 400) {
            resolve(xhr.responseText);
          }
        }
        xhr.onerror = (error) => {
                reject(error);
            }
        xhr.send()
      });
    }
```
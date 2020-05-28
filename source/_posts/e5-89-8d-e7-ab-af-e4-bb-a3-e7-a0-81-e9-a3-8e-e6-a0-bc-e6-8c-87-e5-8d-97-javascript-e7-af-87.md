---
title: 前端代码风格指南——javascript篇
tags:
  - 前端优化
  - 前端工程化
url: 317.html
id: 317
comments: false
categories:
  - 前端
date: 2018-11-20 20:17:27
---

*   严格模式：开启（必须）
    
    > 理由：能够避免一些不精通js特性导致的坑，如==和===，得益于js早期糟糕的设计，有太多这样的问题。需要注意的是，严格模式在ie9下不兼容，但该项目是vue体系项目至少运行在ie10以上故不需要考虑。
    
*   引号：一律使用单引号（必须）
    
    > 理由：可以和html的双引号更好的工作
    
*   分号：一律在语句结尾加分号（必须）
    
    > 理由：首先官方新规范是要求分号的，可以避免不必要的坑。
    
        /* 不推荐的 */
        let fn = {
        return 0
        }
        
        /* 推荐的 */
        let fn = {
        return 0;
        }; //这里有分号
        
    
*   空白与格式（必须）

在二元和三元运算符的符号与操作数之间添加空格，在非行末的 , ; } 后添加空格，在 { 前添加空格。并在每个逻辑块中间添加空白行。 特别的，在 if、while 等关键字后加空格，与函数调用做区分

    /* 不推荐的 */
    let foo='bar',hello=foo+2,test=true;
    function hi(){
      // ...
    }
    if(foo&&hello){
      // ...
    }else if(foo){
      // ...
    }else if(! test){
      // ...
    }
    
    /* 推荐的 */
    let foo = 'bar';
    let hello = foo + 2;
    let test = true;
    function hi(arg1, arg2) {
      // ...
    }
    if (foo && hello) {
      // ...
    } else if (foo) {
      // ...
    } else if (!test) {
      // ...
    }
    

*   变量声明（必须）
    *   一个函数作用域中所有的变量声明尽量提到函数首部。
    *   如果可以使用 let 和 const 的，要使用 let 和 const。
    *   除了特殊情况如挂载全局jquery等外，不要定义全局变量。 > 理由：第一条是保证可读性，第三条和第四条是安全的作用域，
*   变量命名（必须）
    
    *   变量全部使用小驼峰，如
    *   常量使用大写匈牙利式命名法，如MAX_COUNT
    *   组件名必须为多个单词，如ConfigCenter
    *   组件名使用大驼峰命名，如ConfigCenter
    *   构造函数与类，大写第一个字母，如Person()
    *   jquery 对象必须以'$'开头命名
    *   对象私有属性和方法，用'#'开头命名
    *   不要使用复数，对于变量而言，复数还是非负数都一样 \> 理由：可读性好，实际这是js与vue.js官方推荐的命名方式整合。
    
        /* 推荐的 */
        var thisIsMyName; // 变量
        var MAX_COUNT = 10; // 常量
        function Person(name) { // 构造函数
        this.name = name;
        }
        var $body = $("body"); // jquery对象
        class Point { // 类
        #x; // 私有属性
        #y;
        
        constructor(x, y) {
          this.#x = x;
          this.#y = y;
        }
        }
        
    
*   括号（可选）
    

下列关键字后必须有大括号（即使代码块的内容只有一行）：if, else, for, while, do, switch, try, catch, finally, with。大括号如果是单行，前后要加一个空格。

> 理由：能够避免一些bug，提升可读性，方便复制代码而不复制到括号。

    /* 不推荐的 */
    if (condition) doSomething();
    
    /* 推荐的 */
    if (condition) { doSomething() };
    import { x, y } from myModule;
    

*   对于循环和判断代码，不要循环或判断三层以上。（可选）
    
*   能用ES6、ES7等新版本ES语法的，尽量用新语法，包括不仅限于：（部分必须）
    
    *   使用对象和数组的解构赋值的方式实现多个变量的赋值。
    
        /* 不推荐的 */
        let [a, b, c] = [1, 'a', [ 2 , 3]]
        
        /* 推荐的 */
        let []
        
    
    *   使用拓展运算符...对参数对象中的所有可遍历属性进行拷贝。
        
    *   小函数使用箭头函数来替代function，这里要注意箭头函数作用域问题。
        
    *   使用字符串模板进行字符串拼接。
        
    *   使用安全的块级作用域语法let和const。
        
    *   使用Object.assign()进行对象的浅拷贝。
        
    *   使用Set去重，以及替代元素不重复的数组。
        
    *   使用class语法替代原有的方式。
        
    *   使用函数参数的默认值替代 || 语法
        
    
        /* 不推荐的 */
        function f(x, y, z) {
        y = 1 || undefined;
        z = 2 || undefined
        return x + y + z;
        }
        
        /* 推荐的 */
        function f1(x, y = 4, z = 12) {
        return x + y + z;
        }
        
    
    *   使用对象中函数的缩写形式
    
        /* 不推荐的 */
        let obj = {
        add: function() {}
        }
        
        /* 推荐的 */
        let obj =  {
        add() {}
        }
        
    
    *   使用Promise，async-await解决回调函数多层嵌套问题。

注释与文档规范
-------

*   单行注释（必须）
    *   注释单独一行的情况下，注释的//后面要跟一个空格
    *   注释如果和代码同一行，代码分号结束后，要跟一个空格，注释的//后也要跟一个空格 \> 理由：写起来简单，看上去直观。

    // 调用函数
    foo();
    
    var maxCount = 10; // 这是一个变量
    

*   多行注释（必须）

多行注释使用下面这种形式

> 理由：js统一的规范，官方也是这么写的。

    /**
     * 代码注释1
     * 代码注释2
     */
     ```
    
      多行注释建议在以下几种情况使用：
      - 难于理解的代码段
      - 可能存在错误的代码段
      - 浏览器特殊的 HACK 代码
      - 业务逻辑强相关的代码
    
    - 函数注释（必须）
    
    复杂的函数，所有类，都必须进行标准函数注释，示例如下。
    > 理由：函数注释使用业界统一的规范，方便后续使用 jsdoc 生成文档。
    ```js
    /**
     * 获取任务的名称
     * @param id {Number} 传入需要获取名称的人物id
     * @return {String} 返回的姓名
     * @author shi 2015/07/21 可以不写
     * @version 1.1.0 可以不写
     * @example 示例代码，可以不写
     */
    function getTaskName(id) {
      let name = "test";
      return name;
    }
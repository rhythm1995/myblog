---
title: JavaScript工作原理：五种JavaScript代码的优化方式
tags:
  - JavaScript
url: 268.html
id: 268
categories:
  - node.js
  - 前端
date: 2019-06-12 15:45:02
---

在了解这些优化方式前，先强烈建议阅读本博客前两篇文章，前两篇是对V8原理的一个入门，因为这些优化方式，都是针对原理而引入的可行方式。

对象属性的顺序
-------

始终以相同的顺序实例化对象属性，以便可以共享隐藏的类和随后优化的代码。
```js
    // bad
    function Point(x, y) {
        this.x = x;
        this.y = y;
    }
    var a = new Point(1,2);
    var b = new Point();  // 顺序不一致
    b.x = 2;
    b.y = 1;

    // good
    function Point(x, y) {
        this.x = x;
        this.y = y;
    }
    var a = new Point(1,2);
    var b = new Point(2,1);
```

动态属性
----

在实例化之后向对象添加属性或者修改属性的类，都会修改隐藏类，使得优化无效。所以对象实例化时尽量添加好属性，并且尽量不再做对属性类型的修改。
```js
    // bad
    function Point(x, y) {
        this.x = x;
        this.y = y;
    }
    var a = new Point(1,2);
    var b = new Point(2,1);
    b.z = 3; // 实例化后添加了属性
    a.y = null // 实例化后修改了类型

    // good
    function Point(x, y) {
        this.x = x;
        this.y = y;
    }
    var a = new Point(1,2);
    var b = new Point(2,1);
```

对象的方法
-----

对同一个类new出来的对象多次执行同一个方法，第一次后的执行速度会非常快，因为内联缓存。
```js
    // bad
    function Rectangle(x, y) {
        this.x = x;
        this.y = y;
    }
    function Square(a) {
        this.a = a;
    }

    var a = new Rectangle(1, 2);
    var b = new Square(2);
    function SquareArea(square) {
        var a = square.a;
        return a * a;
    }
    function RectangleArea(rectangle) {
        var x = rectangle.x;
        var y = rectangle.y;
        return x * y;
    }

    // good
    function Rectangle(x, y) {
        this.x = x;
        this.y = y;
    }
    function Area (rectangle) {
        var x = rectangle.x;
        var y = rectangle.y;
        return x * y;
    }
    var a = new Rectangle(1, 2);
    var b = new Rectangle(2, 2);
    var b = new Rectangle(5, 6);
    Area(a);
    Area(b);
    Area(c);
```

内存回收
----

有效使用内存总是可以提高代码的性能，对于使用了垃圾回收的语言来说，并不意味着有了垃圾回收就没有内存泄漏的问题，如果及时回收不再使用的内存，可以避免内存泄漏的同时减轻编译器的工作。简单的做法就是把不再使用的对象设置为null，以及通过delete关键字，来删除对象的一个属性。

数组
--

避免稀疏数组，其中键值不是自增的数字。并没有存储所有元素的稀疏数组是哈希表。这种数组中的元素访问开销较高。另外，尽量避免预分配大数组。最好是按需增长。最后，不要删除数组中间的元素，即使用队列和栈方法删除。这会使键值变得稀疏。
```js
    var a = [];
    a[0] = 1;
    a[5] = 2; // 稀疏数组

    var b = [];
    b.length = 10000; // 预先分配了大数组

    var c = [1,2,3,4,5];
    c[3] = undefined; // 删除后变为稀疏数组
```

数据表示
----

V8 使用 32 位表示对象和数值。由于数值是 31 位的，它使用了一位来区分它是一个对象（flag = 1）还是一个称为 SMI（SMall Integer）整数（flag = 0）。那么，如果一个数值大于 31 位，V8会将该数字装箱，把它变成一个双精度数，并创建一个新的对象来存放该数字。尽可能使用 31 位有符号数字，以避免对 JS 对象的高开销的装箱操作。 V8 的简单数据存放在句柄，句柄的操作是非常快的，而对大整数和大浮点数会使用堆，因为大整数是不可避免的，所以能使用整数的，尽量不要使用浮点数。
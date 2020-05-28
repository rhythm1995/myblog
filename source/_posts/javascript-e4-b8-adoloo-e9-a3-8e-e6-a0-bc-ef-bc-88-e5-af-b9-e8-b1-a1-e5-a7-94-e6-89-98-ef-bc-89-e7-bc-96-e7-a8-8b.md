---
title: javascript中OLOO风格（对象委托）编程
tags:
  - JavaScript
  - 前端
url: 191.html
id: 191
categories:
  - 前端
date: 2018-04-24 15:11:51
---

什么是OLOO
=======

在《you don't know js》中，作者有提到"OLOO"(objects-linked-to-other-objects)编程风格更适合js，而非传统的面向对象风格。原因是：js本身不提供类，所以是对类的一种模拟，实际实现非常繁琐（从es5的多种继承写法可知），是一种舍本求末的行为，而OLOO风格在该书中被认为是最适合JS的风格。 这是一个Student-Person的OLOO风格例子：

    var Person = {
      init: function(name) {
        this.name = name;
      },
      getName: function() {
        return this.name;
      }
    };
    
    var Student = Object.create(Person);
    
    Student.set = function(name, school) {
      this.init(name);
      this.school = school;
    };
    
    Student.getSchool = function() {
      return this.school;
    };
    
    var bugzhang = Object.create(Student);
    bugzhang.set('Bugzhang', 'CSU');
    
    bugzhang.getName(); // Bugzhang
    bugzhang.getSchool(); // CSU
    

OLOO与OOP的对比
===========

在绝大多数编程语言中，面向对象的实现有两个范式：基于类的面向对象与基于原型的面向对象，前者如c++，java，后者如javacript。 在js中，如果利用构造函数实现继承，那么构造函数不会在原型链上，原型链只有prototype。如果利用prototype实现，其实prototype式继承不好用。 OLOO没有解决property为引用类型时会共享的问题。 在OOP中：采用的是**类-继承**的思想实现。 在OLOO中：采用的是**行为代理模式**的设计思想。OLOO只代表对象利用原型链连接到其他对象。不再使用构造函数来创建对象，而是使用Object.create()为主的方法，利用对象创建对象。

ES6时代的选择
========

在es6中，class关键字的出现，让js模拟类成为了历史。虽然class本质上任然是构造函数原型混合继承的语法糖，但本身简单好用，简化了重复并且没有意义的模板代码，比如`xxx.prototype.xxx`等；把 class 的 `constructor` 和普通函数严格区分，避免了 new 的暗坑；加上统一的 `super` 关键字作为继承的关键字，而且继承链是成线性的。种种有点表明，class应该作为未来主流的方式。
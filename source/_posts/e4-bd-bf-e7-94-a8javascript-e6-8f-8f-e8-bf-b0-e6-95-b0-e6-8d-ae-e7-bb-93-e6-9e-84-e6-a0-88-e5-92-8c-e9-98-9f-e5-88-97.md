---
title: 使用javascript描述数据结构——栈和队列
tags:
  - JavaScript
  - 数据结构
url: 206.html
id: 206
categories:
  - 算法
date: 2018-06-28 22:35:46
---

栈
-

栈是一种先入后出的数据结构：从末尾添加、从末尾删除，末尾的元素成为栈顶，最先进入的元素称为栈底。 js的数组可以用来模拟数据结构中的栈。
```js
    class Stack {

        constructor() {
            this.items = [];
        }

        // 入栈：推入一个元素
        push(element) {
             this.items.push(element);
        }

        // 出栈：移除顶部元素
        pop() {
            return this.items.pop();
        }

        // 获取栈底元素
        get peek() {
            return this.items[this.items.length - 1];
        }

        // 是否为空栈
        get isEmpty() {
            return !this.items.length;
        }

        // 获取长度
        get size() {
            return this.items.length;
        }

        // 清空栈
        clear() {
            this.items = [];
        }

        // 打印栈数据
        print() {
            console.log(this.items.toString());
        }
    }
```

队列
--

队列是一种遵循先进先出的数据机构；队列在尾部添加新元素，从头部移除元素。最新添加的元素必须排在队列的末尾。
```js
    class Queue {

        constructor(items) {
            this.items = items || [];
        }

            // 添加元素
        enqueue(element){
            this.items.push(element);
        }

            // 删除元素
        dequeue(){
            return this.items.shift();
        }

            // 返回第一个元素
        front(){
            return this.items[0];
        }

            // 清空队列
        clear(){
            this.items = [];
        }

            // 队列大小
        get size(){
            return this.items.length;
        }

            // 是否为空
        get isEmpty(){
            return !this.items.length;
        }

            // 打印
        print() {
            console.log(this.items.toString());
        }
    }
```
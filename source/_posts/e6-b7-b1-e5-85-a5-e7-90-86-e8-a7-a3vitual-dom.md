---
title: 深入理解Vitual DOM
tags:
  - 前端底层
url: 58.html
id: 58
categories:
  - 前端
date: 2017-08-29 23:12:57
---

![](http://7xqgks.com1.z0.glb.clouddn.com/head-0049.jpg)

> Vitual DOM是被诸如React,Vue,Preact等框架采用的一种内部黑盒技术，用来作为到最终dom操作的一个中介。 ![](http://7xqgks.com1.z0.glb.clouddn.com/v-dom1.png) v-dom在前端的工作图

# v-dom是什么

v-dom是一个真实dom的模拟表示，它像是原始dom的一个轻量级的副本，不同的是v-dom减少了冗余，采用了高性能的diff算法比较更新dom，同时只通过最终的正式dom操作来渲染入游览器，这最终特性保证了v-dom的高效。

# 为什么要用v-dom

DOM操作是把js实现到游览器的核心，一切的交互都需要用过dom。不幸的是，don操作是一件性能非常底下的工作：因为dom对象中有很多和操作无关的冗余字段，而任何dom操作不得不遍历整颗冗余度非常高的dom树。 比如，目前有一个列表，包含多个项目，当检查第一项，dom将重建整个列表，这是十倍以上的非必须工作，在早起前端，因为业务的不复杂性，dom操作尚能工作顺利，但目前越来越复杂的前端场景，现代网站可以使用大量dom操作，低效更新已经是一个严重的问题。 ![](http://7xqgks.com1.z0.glb.clouddn.com/v-dom2.png)真实的游览器渲染过程，元素的更新会涉及到重绘和回流，进一步降低性能。 这种缓慢的现实，使得很多js框架采用自己的方式去更新dom，v-dom就是其中一个被推广的佼佼者。

# v-dom如何被建立

![](http://efe.baidu.com/blog/the-inner-workings-of-virtual-dom/1.png)Preact的dom工作流程。

## 虚拟元素

我们可以用如下的虚拟元素替代真实dom：

    /**
     * 一个类似的dom结构元素的v-dom设计
     * <div id="container">
     *  <h1>Hello v-dom</h1>
     * </div>
     */

    var element = {
        tagName: 'div',
        attr: {
            props: {
                id: 'container'
            },
            style: {
                color: 'black'
            }
        },
        children: {
            tagName: 'h1',
            value: 'Hello v-dom'
        }
    }

    //用构造函数模拟一下
    function Element(tagName, attr, children, value) {
        this.tagName = tagName;
        this.attr = attr;
        this.children = children;
        this.value = value;
    };
    var headline = new Element('h1', null, null, 'Hello world');
    var container = new Element('div', {
        props: {
            id: 'container'
        },
        style: {
            color: 'black'
        }
    }, headline, null);


上述代码就是一个对v-dom的描述以及简单实现，实际上React的JSX转换后真正调用的API也是类似的API，这个API是`React.createElement()`。 v-dom有一些通用的特点：轻量级、无状态、不可改变。

## render方法

有了上述的v-dom对象后，再通过类似`render()`的方法，就可以创建真实dom，render函数的简单实现类似这样：

    function render(element, root) {
        var realDOM = document.createElement(element.tagName);
        //循环设置属性和样式
        var props = element.attr.props;
        var styles = element.attr.style;
        for (var i in props) {
            realDOM.setAttrbute(i, props[i]);
        }
        for (var j in styles) {
            realDOM.style[j] = styles[j];
        }
        //循环子节点，如果是对象递归该方法，否则创建文本节点
        element.children.forEach(function (child) {
            if (child instanceof Element) {
                render(child, realDOM);
    } else {
                raedlDOM.appendChild(document.createTextNode(child));
            }
        });
        //插入真实dom
        root.appendChild(realDOM);
        return realDOM;
    }


## diff算法比较差异

当Virtual DOM发生更新时候，会进行变化生成一颗新的dom树，为了比较两棵树的异同，引入了一种Diff算法完成比较，diff算法非常高效，当对v-dom完成差异比较后，这个差异会作用到真实dom，过程如下：

*   1.构建Virtual Dom树
*   2.将Virtual Dom插入真实dom
*   3.构建变化后的Virtual Dom树
*   4.通过diff算法比较差异
*   5.仅将差异在真实DOM中更新

给定任意两颗树，进行转换的差异算法，一般复杂度是O(n^3)，react的diff算法复杂度是O(n)，它基于v-dom的两个基本事实：

*   两个相同组件产生类似的DOM结构，不同的组件产生不同的DOM结构；
*   对于同一层次的一组子节点，它们可以通过唯一的id进行区分。

组件的树是自带层级的，diff算法按照层级比较，如图： ![](http://cdn.infoqstatic.com/statics_s2_20170822-0429/resource/articles/react-dom-diff/zh/resources/0909000.png) 如果节点类型不同，直接删除Before中的节点，插入新节点；如果节点类型相同，继续层序遍历属性，属性不同则替换属性；直到遍历到最底层。通过一次遍历，即可比较出不同，更新了整个dom。

    //一个参考算法，链接：https://www.zhihu.com/question/29504639/answer/73607810
    // diff 函数，对比两棵树
    function diff (oldTree, newTree) {
      var index = 0 // 当前节点的标志
      var patches = {} // 用来记录每个节点差异的对象
      dfsWalk(oldTree, newTree, index, patches)
      return patches
    }

    // 对两棵树进行深度优先遍历
    function dfsWalk (oldNode, newNode, index, patches) {
      // 对比oldNode和newNode的不同，记录下来
      patches[index] = [...]

      diffChildren(oldNode.children, newNode.children, index, patches)
    }

    // 遍历子节点
    function diffChildren (oldChildren, newChildren, index, patches) {
      var leftNode = null
      var currentNodeIndex = index
      oldChildren.forEach(function (child, i) {
        var newChild = newChildren[i]
        currentNodeIndex = (leftNode && leftNode.count) // 计算节点的标识
          ? currentNodeIndex + leftNode.count + 1
          : currentNodeIndex + 1
        dfsWalk(child, newChild, currentNodeIndex, patches) // 深度遍历子节点
        leftNode = child
      })
    }


一篇很好的diff算法原理的解析：http://www.infoq.com/cn/articles/react-dom-diff/

# 生命周期问题

v-dom的整个过程，即产生了生命周期，一切和v-dom相关的组件的生命周期也与之相关。如： ![](https://cn.vuejs.org/images/lifecycle.png)vue组件的生命周期 ![](http://7xqgks.com1.z0.glb.clouddn.com/3-3-component-lifecycle.jpg)react组件的生命周期 一般分为初始化、运行中、销毁三个状态，三个状态有关的生命周期钩子函数有： **初始化阶段**： 　　getDefaultProps:获取实例的默认属性(即使没有生成实例，组件的第一个实例被初始化CreateClass的时候调用，只调用一次,) 　　getInitialState:获取每个实例的初始化状态（每个实例自己维护） 　　componentWillMount：组件即将被装载、渲染到页面上（render之前最好一次修改状态的机会） 　　render:组件在这里生成虚拟的DOM节点（只能访问this.props和this.state；只有一个顶层组件，也就是说render返回值值职能是一个组件；不允许修改状态和DOM输出） 　　componentDidMount:组件真正在被装载之后，可以修改DOM **运行中状态**： 　　componentWillReceiveProps:组件将要接收到属性的时候调用（赶在父组件修改真正发生之前,可以修改属性和状态） 　　shouldComponentUpdate:组件接受到新属性或者新状态的时候（可以返回false，接收数据后不更新，阻止render调用，后面的函数不会被继续执行了） 　　componentWillUpdate:不能修改属性和状态 　　render:只能访问this.props和this.state；只有一个顶层组件，也就是说render返回值只能是一个组件；不允许修改状态和DOM输出 　　componentDidUpdate:可以修改DOM **销毁阶段**： 　　componentWillUnmount:开发者需要来销毁（组件真正删除之前调用，比如计时器和事件监听器）
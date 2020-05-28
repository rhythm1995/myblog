---
title: javascript中的相等"==="与"=="详解
tags:
  - JavaScript
  - 前端底层
url: 165.html
id: 165
categories:
  - 前端
date: 2018-02-25 13:16:44
---

全等
==

js对全等"==="的实现逻辑是如下的： ![](http://img.bugzhang.com/%E5%85%A8%E7%AD%89.png) 即： 1\. 如果 Type(x) 和 Type(y) 不同，返回 false 2. 如果 Type(x) 为 Undefined，返回 true 3. 如果 Type(x) 为 Null，返回 true 4. 如果 Type(x) 为 Number，则进入下面的判断逻辑 4.1. 如果 x 为 NaN，返回 false
4.2. 如果 y 为 NaN，返回 false
4.3. 如果 x 的数字值和 y 相等，返回 true
4.4. 如果 x 是 +0 且 y 是 -0，返回 true
4.5. 如果 x 是 -0 且 y 是 +0，返回 ture
4.6. 返回 false 5. 如果 Type(x) 为 String，则当且仅当 x 与 y 的字符序列完全相同（长度相等，每个位置上的字符相同）时返回 true，否则返回 false 6. 如果 Type(x) 为 Boolean，则若 x 与 y 同为 true 或同为 false 时返回 true，否则返回 false7. 如果 x 和 y 引用的是同一个对象，返回 true，否则返回 false

不全等==
=====

不全等的逻辑更加冗长，如下： ![](http://img.bugzhang.com/%E9%9D%9E%E5%85%A8%E7%AD%89.png)

1.  如果 Type(x) 和 Type(y) 相同，则

2.  1.1. 如果 Type(x) 为 Undefined，返回 true
    1.2. 如果 Type(x) 为 Null，返回 true
    1.3. 如果 Type(x) 为 Number，则
    1.3.1. 如果 x 是 NaN，返回 false
    1.3.2. 如果 y 是 NaN，返回 false
    1.3.3. 如果 x 的数值与 y 相同，返回 true
    1.3.4. 如果 x 是 +0 且 y 是 -0，返回 true
    1.3.5. 如果 x 是 -0 且 y 是 +0，返回 true
    1.3.6. 返回 false
    1.4. 如果 Type(x) 为 String，则当且仅当 x 与 y 的字符序列完全相同（长度相等，每个位置上的字符相同）时返回 true，否则返回 false
    1.5. 如果 Type(x) 为 Boolean，则若 x 与 y 同为 true 或同为 false 时返回 true，否则返回 false
    1.6. 如果 x 和 y 引用的是同一个对象，返回 true，否则返回 false

3.  如果 x 是 null 且 y 是 undefined，返回 true

4.  如果 x 是 undefined 且 y 是 null，返回 ture

5.  如果 Type(x) 为 Number 且 Type(y) 为 String，以 x == ToNumber(y) 的比较结果作为返回

6.  如果 Type(x) 为 String 且 Type(y) 为 Number，以 ToNumber(x) == y 的比较结果作为返回值

7.  如果 Type(x) 为 Boolean，以 ToNumber(x) == y 的比较结果作为返回值

8.  如果 Type(y) 为 Boolean，以 x == ToNumber(y) 的比较结果作为返回值

9.  如果 Type(x) 为 String 或 Number 且 Type(y) 为 Object，以 x == ToPrimitive(y) 的比较结果作为返回值

10.  如果 Type(x) 为 Object 且 Type(y) 为 String 或 Number，以 ToPrimitive(x) == y 的比较结果作为返回值

11.  返回 false

所以实际就是这样的表现： ![](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1519546038753&di=433922788363eb439577b9e06a9758d0&imgtype=0&src=http%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F912254%2F201610%2F912254-20161025235619187-741352385.png)

完全等同函数
======

无论全等还是非全等，对于NaN，以及+0、-0的处理都是非常失败的，因此可以有一个完全等同函数处理这个问题

```js
    function identity(val1, val2) {
        if(val1===val2){
            if((val1 === -0 || val2 === +0)&&(val1 === +0 || val2 === -0)){
                return false
            }
            return true;
        }else {
            if(val1 === NaN || val2 === NaN){
                return true
            }
            return false;
        }
    }
```

此外需要注意的就是，在严格模式下，只允许采用全等，另外在代码中，也更加提倡全等以避免不必要的麻烦。
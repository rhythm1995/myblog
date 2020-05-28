---
title: 使用gulp减少HTTP请求
tags:
  - 前端
  - 前端工程化
url: 40.html
id: 40
categories:
  - 前端
date: 2017-06-18 22:42:56
---

![](http://7xqgks.com1.z0.glb.clouddn.com/head-0026.jpg)

> 游览器与服务器每进行一个http连接，需要进行TCP三次握手的确认，如果是https更是多出了7次安全确认来完成连接。每个http报文中，又有很多重复的多余信息，因此减少http请求，是最为可行的前端优化方式之一。以下是现代前端中，常用的减少http请求的方式及其实现。

# CSS Sprites

CSS Sprites，又叫雪碧图或者CSS精灵，是指把多个图片合并在一张图片上，然后通过`background-positon`属性制定CSS prites偏移量。 因为实现很简单，这个就不粘代码了。<br> 总之雪碧图可以让项目变得：更少的图片，干净的标签，更少的HTTP请求，甚至更小的文件（因为图片压缩编码的原因，合并后的文件往往比分开的文件总和要小，空白区域并不会增加额外的大小）。总而言之使用CSS Sprites可以提升网站性能。

# 合并js脚本与css样式表

每多合并一个js脚本或者css样式表，就可以减少一次http请求，进而提升性能。对于现代前端而言，合并脚本与样式表的工作，往往交给自动化工具完成，我常用的是Gulp，下边是使用Gulp自动化工具进行压缩的配置文件gulpfile.js的示例：

    const gulp = require('gulp');
    const concat = require('gulp-concat');
    const concatCss = require('gulp-concat-css');

    gulp.task('testConcat', function () {
        gulp.src('src/js/*.js')
            .pipe(concat('main.js'))        //合并后的文件名
            .pipe(gulp.dest('dist/js'));
    });

    gulp.task('testConcatCss', function () {
      return gulp.src('assets/**/*.css')
        .pipe(concatCss("styles/bundle.css"))
        .pipe(gulp.dest('out/'));
    });


附gulp中文文档：[www.gulpjs.com.cn](www.gulpjs.com.cn)

# 将小图片进行base64编码后写入css

用编码工具，将小图片进行base64编码，然后将编码后的数据拷贝出来，放在background-image: url的位置。该方法也同样减少了http请求，但会增大体积，故不适合于大图片仅适合于小图片。实现同样很简单，就不写了。

# 采用icon-font的方式引入矢量图标

实现很简单，把svg丢到一个自动化的工具里就可以打包出来了，使用通过css的类来使用，相信用过bootstrap等此类响应式框架的同学都知道。我常用的打包网站是：https://icomoon.io/ 。个人感觉，这是最适合的对图片的处理方式，未来小图片将全部矢量化。 <br>
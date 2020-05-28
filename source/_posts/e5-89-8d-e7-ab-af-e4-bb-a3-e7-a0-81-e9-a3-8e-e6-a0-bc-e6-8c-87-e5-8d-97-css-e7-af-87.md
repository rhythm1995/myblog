---
title: 前端代码风格指南——CSS篇
url: 216.html
id: 216
categories:
  - 前端
date: 2018-11-07 15:35:27
tags:
---

是我常用的一套开发标准，借鉴、删减、自创的一些规则，保留的基本都是比较实用的。用于统一代码风格。并给出了每条标准的理由。

* * *

*   空白与格式（可选）
    
    *   大括号与选择器之间留空，冒号后面留空 \> 理由：这样比较好看而且多数已有代码是这样。
    
        /* 不推荐的 */
        .el-pager{
        width:$--pagination-number-after-width;
        }
        
        /* 推荐的 */
        .el-pager {
        width: $--pagination-number-after-width;
        }
        
    
    *   在只有一条样式时允许和选择器写到同一行，但大括号首尾最好留一个空格。 \> 理由：写三行太浪费屏幕空间。
    
        /* 不推荐的 */
        .el-pager {
          width:$--pagination-number-after-width;
        }
        
        /* 推荐的 */
        .el-pager { width: $--pagination-number-after-width; }
        
    
    *   一个选择器中有多个样式声明时每条写一行 \> 理由：使报错可以精确到具体的规则上，便于排错。
    
        /* 不推荐的 */
        .el-pager {width:$--pagination-number-after-width; height: $--pagination-number-after-height; color: $--pagination-number-after-color;}
        
        /* 推荐的 */
        .el-pager {
          width: $--pagination-number-after-width;
          height: $--pagination-number-after-height;
          color: $--pagination-number-after-color;
        }
        
    
    *   多个选择器使用逗号隔开时写在不同的行，大括号不要另起一行 \> 理由：修改时不容易漏掉逗号后面的选择器。
    
        /* 不推荐的 */
        .el-pager, div {
          width:$--pagination-number-after-width;
        }
        
        /* 推荐的 */
        .el-pager,
        div {
          width:$--pagination-number-after-width;
        }
        
    
    *   每条样式声明后面都加上分号
        
        > 理由：复制起来方便。
        
    *   所有最外层引号使用双引号
        
        > 理由：与HTML保持一致。
        
    
        /* 不推荐的 */
        @import url(//www.google.com/css/maia.css);
        html { font-family: 'open sans', arial, sans-serif; }
        
        /* 推荐的 */
        @import url("//www.google.com/css/maia.css");
        html {
          font-family: "open sans", arial, sans-serif;
        }
        .selector[type="text"] { }
        
    
    *   用逗号分隔的多个样式值写成多行
    
    > 理由：便于阅读和编辑。
    
        /* 不推荐的 */
          .block {
            box-shadow: 0 0 0 rgba(#000, 0.1), 1px 1px 0 rgba(#000, 0.2), 2px 2px 0 rgba(#000, 0.3), 3px 3px 0 rgba(#000, 0.4), 4px 4px 0 rgba(#000, 0.5);
          }
        
        /* 推荐的 */
        .block {
          box-shadow: 0 0 0 rgba(#000, 0.1),
                    1px 1px 0 rgba(#000, 0.2),
                    2px 2px 0 rgba(#000, 0.3),
                    3px 3px 0 rgba(#000, 0.4),
                    4px 4px 0 rgba(#000, 0.5);
          }
        
    
*   功能限定（可选）
    
    *   避免使用ID选择器，如无必要禁止使用!important
        
        > 理由：权重太高，不易维护。
        
    *   禁止使用 @import 引入 CSS 文件，但在SCSS等预编译处理器中是允许的
        
    
    > 理由：兼容性差，并且会打破资源下载顺序有性能问题
    
*   属性顺序（可选）
    *   位置属性(position, top, right, z-index, display, float等)
    *   大小(width, height, padding, margin)
    *   文字系列(font, line-height, letter-spacing, color- text-align等)
    *   背景(background, border等)
    *   其他(animation, transition等) > 理由：顺序从高到低依次和使用频率直接相关。
*   变量与属性命名（可选）
    
    *   0 值的单位建议省略，但不强制。
        
        > 理由：css中所有 0 值的单位是没用的。
        
    *   16进制颜色值中的字母统一为小写。
        
        > 理由：大小写对CSS是一样的，但切换大写麻烦。
        
    *   类名中的字母一律小写
        
        > 理由：大小写对CSS是一样的，但难道统一大写或者首字母大写？
        
    *   类名中只是用用字母、数字以及“-”，并且尽量不要使用数字。
        
        > 理由：CSS类名可以用任何字符，但命名还是和js语言变量统一为好。
        
    
        .hello {} /* OK */
        .module-title {} /* OK */
        .panel-level1 {} /* OK */
        
        .导航栏 /* Fuck */
        
    
*   CSS模块化（可选）
    
    *   基本命名
    *   除非是非常常见的缩写，否则类名使用完整英文单词或者抽调空格的英文词组 \> 理由：正常阅读，缩写可能不统一
    
        /* 不推荐的 */
        .konnichiwa {} /* 非英文单词会导致大家无法正常阅读 */
        .modl {} /* 每个人的缩写未必一致，会造成不统一 */
        .hello-world {} /* 类名请只使用一个没有分隔[-_]的词 */
        
        /* 推荐的 */
        .module {}
        .helloworld {}
        .nav {}
        
    
    *   仅当有层级关系时候使用“-”连接 \> 理由：使css与html的层级一致
    
          .form-submit {} /* 推荐 */
          .form-submittingbutton {} /* 不推荐 */
        
    
    *   当要对选择器进行样式的修饰时，可以使用多个类而非对已有类进行定语的限制 \> 理由：先前规范已经规定了必须层级关系采用“-”，并且修饰符可能与组件名冲突
    
        .success-popup. {} /* 不推荐的 */
        .successPopup. {} /* 不推荐的 */
        
        .popup.success {} /* 推荐的 */
        .popup.error {} /* 推荐的 */
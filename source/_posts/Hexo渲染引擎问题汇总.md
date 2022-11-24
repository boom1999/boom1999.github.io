---
title: 渲染问题汇总
date: 2021-07-24
tags: 
    - debug
categories: 总结
---


- Hexo将Markdown文件渲染成静态页面，但是每次都会出现不少问题，本地的渲染引擎和hexo的有所区别，在这里总结一些问题
<!--more-->

### 1.英文引号和中文引号问题

- 比较莫名其妙，Hexo的默认渲染邀请自动将英文引号转化为中文引号，特别是在一些数学公式中，显示会出现问题
- `hexo-renderer-markdown`默认配置`quotes`配置为`quotes: '“”‘’'`，然后英文双引号就会被解析为英文双引号而非坑爹的中文双引号。但是讲道理更改配置`quotes: '""'''`可以取消这种奇怪的逻辑，但是无果。

#### 在一个Issue中找到解决办法

##### `适用hexo-renderer-markdown-it`和`hexo-renderer-markdown-it-plus`

``` yml
markdown:
  render:
-    typographer: true
+    typographer: false
```

##### 适用默认的`hexo-renderer-marked`

``` yml
marked:
-  smartypants: true
+  smartypants: false
```

### 2.公式渲染问题

这次是一个算法的综述，里面有一些多行公式还有矩阵，之前埋下的坑打算这次一起解决了。

- `hexo-renderer-markdown-it`没法渲染`Latex`的矩阵以及多行公式，在`Vscode`或者`Typora`中可以渲染出来，但是`hexo`不太行，所以换了`hexo-renderer-kramed`，版本是`^0.1.4`
- 但是出现新的问题，`kramed`会把`*`和`_`渲染成斜体，所以反而出现了很多问题

#### 解决方案

- 首先是用`\ast`替代公式中可能出现的`*`这里主要是会出现`A*`算法
- 然后取消`_`的斜体转义，`/node_modules/kramed/lib/rules/inline.js`的第20行修改如下：`em: /^\*((?:\*\*|[\s\S])+?)\*(?!\*)/,`，第11行也做如下修改：``escape: /^\\([`*\[\]()#$+\-.!_>])/,``，这里要注意，其实行内代码最好用四个引号````
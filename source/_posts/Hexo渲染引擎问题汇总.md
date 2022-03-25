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

---
title: 渲染问题汇总
date: 2022-12-03
tags: 
    - Debug
    - Hexo
categories: Blog
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
- 然后取消`_`的斜体转义，`/node_modules/kramed/lib/rules/inline.js`的第20行的`/^\b_((?:__|[\s\S])+?)_\b|^\*((?:\*\*|[\s\S])+?)\*(?!\*)/,`修改如下：`em: /^\*((?:\*\*|[\s\S])+?)\*(?!\*)/,`，即去除`_`的转义，以后斜体使用`*`即可，不使用`_`；第11行也做如下修改：``escape: /^\\([\\`*\[\]()#$+\-.!_>])/,``改为``escape: /^\\([`*\[\]()#$+\-.!_>])/,``，这里要注意，其实行内代码最好用四个引号````
- 还有一点要注意就是，改了源码之后要重启`hexo`服务，修改模块后不会自动刷新

### 3.待办事项渲染问题

- 使用`kramed`渲染不了待办事项，但在`hexo-renderer-marked`的`PR`里找到了相关更新：

  ``` javascript
    // Support To-Do List
    Renderer.prototype.listitem = function(text) {
      if (/^\s*\[[x ]\]\s*/.test(text)) {
        text = text.replace(/^\s*\[ \]\s*/, '<input type="checkbox"></input> ').replace(/^\s*\[x\]\s*/, '<input type="checkbox" checked></input> ');
        return '<li style="list-style: none">' + text + '</li>\n';
      } else {
        return '<li>' + text + '</li>\n';
      }
    };
  ```

- 把渲染函数加入到本地的`hexo`文件夹的`/node_modules/hexo-renderer-kramed/lib/renderer.js`，但注意和`-`标记要间隔至少两行，否则第一个待办会出错


- [x] 待办事项1
- [ ] 待办事项2
- [x] 待办事项3
- [ ] 待办事项4

### 4.表格渲染问题

- 问题：换用`hexo-renderer-kramed`不就后发现之前用`marked`能成功渲染Tables的地方无法渲染
- 找了`n`小时后来发现只要`GFM`的Table语法，和前一行严格保持一行以上的间距，并且在Table块之前本行无空格，就能和原来的`marked`一样渲染出正常的表格
- 但是紧接着会多一个不算问题的问题，如果使用多个`-`标记的用于分层，并且在非第一层的层与层之间之间存在上述方法修改的Table，会导致Table后面跟着的下一个层级变丑，例如：


- 原来
  - 原来
    - 这一层的样式

- - - 变成了这样

- 其实合理怀疑是`Kramed`的空格渲染问题，有空排查一下

> 下面附上`GFM`的Table语法

Tables aren't part of the core Markdown spec, but they are part of GFM and Markdown Here supports them. They are an easy way of adding tables to your email -- a task that would otherwise require copy-pasting from another application.

```
Colons can be used to align columns.

| Tables        | Are           | Cool  |
| ------------- |:-------------:| -----:|
| col 3 is      | right-aligned | $1600 |
| col 2 is      | centered      |   $12 |
| zebra stripes | are neat      |    $1 |

There must be at least 3 dashes separating each header cell.
The outer pipes (|) are optional, and you don't need to make the 
raw Markdown line up prettily. You can also use inline Markdown.

Markdown | Less | Pretty
--- | --- | ---
*Still* | `renders` | **nicely**
1 | 2 | 3
```

Colons can be used to align columns.

| Tables        | Are           | Cool  |
| ------------- |:-------------:| -----:|
| col 3 is      | right-aligned | $1600 |
| col 2 is      | centered      |   $12 |
| zebra stripes | are neat      |    $1 |

There must be at least 3 dashes separating each header cell. The outer pipes (|) are optional, and you don't need to make the raw Markdown line up prettily. You can also use inline Markdown.

Markdown | Less | Pretty
--- | --- | ---
*Still* | `renders` | **nicely**
1 | 2 | 3
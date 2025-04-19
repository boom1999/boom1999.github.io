---
title: Nunjucks Errors
date: 2021-08-26
tags: 
   - Debug
categories: NunjucksErrors
copyright: true
---

- :sunny::clock1030::sleeping:
- Nunjucks模板标签导致MD文件解析报错的问题:

``` bash
FATAL {
  err: Error [Nunjucks Error]: _posts/mathBasic.md [Line 13, Column 14] expected variable end
      =====               Context Dump               =====
      === (line number probably different from source) ===
    8 | <p>$f’({ {x}<em>{0} })=\underset{\Delta x\to 0}{\mathop{\lim } },\frac{f({ {x}</em>{0} }+\Delta x)-f({ {x}_{0} })}{\Delta x}$    （1）</p>
    9 | <p>或者：</p>
    10 | <p>$f’({ {x}<em>{0} })=\underset{x\to { {x}</em>{0} }}{\mathop{\lim } },\frac{f(x)-f({ {x}<em>{0} })}{x-{ {x}</em>{0} }}$           （2）</p>

```
<!--more-->

### 分析

- Markdown文件中有标签与nunjucks模板引擎的标签冲突，比如

``` bash
{{
{{{
}}}    
}}
```

- 这些标签都是模板引擎的，如果Markdown文件中有这些标签，那么在解析的时候就会把Markdown中的标签动态解析
- 处理方法就是**避免出现这些标签**

### 解决方案1

- 考虑过将在两个标签中增加一个空格来避免冲突，可以成功渲染，本地markdown也可以解析出公式，但hexo上的markdowm无法解析，故建议不要使用，本质上没有完全解决问题。

### 解决方案2

参考两个Issues

- [hexo genereate faile Template render error: (unknown path) #2679][1]
- [Template render error: (unknown path) #2384][2]

网上目前很多解决方案都是进行类似如下的替换，需要对所有公式进行替换，量比较大并且治标不治本，暂不采用

``` python
{% raw %}
{{name}}
{% endraw %}
```

### 解决方案3

- 直接修改`Nunjucks`源码

`node_modules/nunjucks/src/lexer.js`

``` js
'use strict';

var lib = require('./lib');

var whitespaceChars = " \n\t\r\xA0";
var delimChars = '()[]{}%*-+~/#,:|.<>=!';
var intChars = '0123456789';
var BLOCK_START = '{%';
var BLOCK_END = '%}';
var VARIABLE_START = '{$';
var VARIABLE_END = '$}';
var COMMENT_START = '{#';
var COMMENT_END = '#}';
var TOKEN_STRING = 'string';
var TOKEN_WHITESPACE = 'whitespace';
var TOKEN_DATA = 'data';
var TOKEN_BLOCK_START = 'block-start';
var TOKEN_BLOCK_END = 'block-end';
var TOKEN_VARIABLE_START = 'variable-start';
```

冲突部分：

``` js
var VARIABLE_START = '{{';
var VARIABLE_END = '}}';
```

作如下替换：

``` js
var VARIABLE_START = '{$';
var VARIABLE_END = '$}';
```

找到定义部分的`VARIABLE_START`和`VARIABLE_END`，这次的`{{`和`}}`错误就是这个导致，上述代码已经将两个变量从`{{`改为`{$`，`}}`改为`$}`
通过直接修改渲染标签，解析的时候就不会和markdown中的复杂公式冲突，并且对所文件都有效，为了防止之后重新`npm install`的时候还原，所以在这里记录一笔

### Others

其他的hexo依赖插件目前还没有发现因为修改`Nunjucks`源码而出现的问题，例如`hexo-generator-searchdb`、`hexo-generator-sitemap`等，有些插件也是依赖于`Nunjucks`的，如果出现问题，同步修改插件的`xml`模板文件就可以解决。

### Update 2025.04.19

> 果然记一笔是有用的，最近发现全局搜索出了问题，开始排查，果然是改`Nunjucks`源码引起的。

- `hexo-generator-searchdb`插件的模板文件中也有`{{`和`}}`，所以在搜索的时候会报错，解决方法是将插件中的`{{`和`}}`作相同的替换，`node_modules/hexo-generator-searchdb/templates/search.xml`：

``` xml
<?xml version="1.0" encoding="utf-8"?>
<search>
  {%- for article in articles %}
  <entry>
    <title>{$ article.title $}</title>
    <url>{$ article.url $}</url>
    <content><![CDATA[{$ article.content | safe $}]]></content>
    {%- if article.categories %}
      <categories>
        {%- for category in article.categories %}
        <category>{$ category $}</category>
        {%- endfor %}
      </categories>
    {%- endif %}
    {%- if article.tags %}
      <tags>
        {%- for tag in article.tags %}
        <tag>{$ tag $}</tag>
        {%- endfor %}
      </tags>
    {%- endif %}
  </entry>
  {%- endfor %}
</search>
```

- `hexo-generator-sitemap`插件的模板文件中也有`{{`和`}}`，`xml`文件中的`{{`和`}}`作相同的替换，`node_modules/hexo-generator-sitemap/sitemap.xml`：

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  {% for post in posts %}
  <url>
    <loc>{$ post.permalink | uriencode $}</loc>
    {% if post.updated %}
    <lastmod>{$ post.updated | formatDate $}</lastmod>
    {% elif post.date %}
    <lastmod>{$ post.date | formatDate $}</lastmod>
    {% endif %}
    <changefreq>monthly</changefreq>
    <priority>0.6</priority>
  </url>
  {% endfor %}

  <url>
    <loc>{$ config.url | uriencode $}</loc>
    <lastmod>{$ sNow | formatDate $}</lastmod>
    <changefreq>daily</changefreq>
    <priority>1.0</priority>
  </url>

  {% for tag in tags %}
  <url>
    <loc>{$ tag.permalink | uriencode $}</loc>
    <lastmod>{$ sNow | formatDate $}</lastmod>
    <changefreq>weekly</changefreq>
    <priority>0.2</priority>
  </url>
  {% endfor %}

  {% for cat in categories %}
  <url>
    <loc>{$ cat.permalink | uriencode $}</loc>
    <lastmod>{$ sNow | formatDate $}</lastmod>
    <changefreq>weekly</changefreq>
    <priority>0.2</priority>
  </url>
  {% endfor %}
</urlset>
```

- `hexo-generator-feed`插件也要改

[1]: https://github.com/hexojs/hexo/issues/2679
[2]: https://github.com/hexojs/hexo/issues/2384

---
title: Graphviz
date: 2024-05-18
tags: 
    - Hexo
    - Graphviz
categories: Blog
copyright: true
---

:pushpin: 博客系统的Graphviz支持

- [Graphviz（Graph Visualization Software）][1]是一个开源的图形可视化软件，用DOT描述语言来定义图形，让用户专注于内容而忽略布局的设计
- 在先前的博客中，图片可以用对象存储在云端，而简单的结构图也如此未必太过复杂，因此想到用Graphviz来绘制简单的图结构。在本地的Vscode或者Typora中已经内嵌了Graphviz的渲染，标记代码块为`graphviz`即可，但在博客中没有，两端存在差异化，这篇用于解决这个问题。
:dash::clock11::sleeping:

<!--more-->

## 添加支持

### Install

- Win/Mac/Linux
  - [Graphviz Download][2]，主流系统内的使用也比较简单，可以直接安装，不再赘述
- Blog System
  - 安装插件`hexo-graphviz`

    ``` bash
    yarn add hexo-graphviz   # 通过yarn安装
    npm install hexo-graphviz --save # 通过npm安装
    ```

  - 模板增加相应的js代码，目录为`{blogRoot}/themes/{themeName}/layout/_partial/after-fotter.ejs`

    ``` js
    <% if (theme.graphviz.enable) { %>
        <script src='https://cdnjs.cloudflare.com/ajax/libs/viz.js/1.7.1/viz.js'></script>
        <script>
            String.prototype.replaceAll = function(search, replacement) {
                var target = this;
                return target.split(search).join(replacement);
            };

            let vizObjects = document.querySelectorAll('.graphviz')

            for (let item of vizObjects) {
                let svg = undefined
                try {
                    svg = Viz(item.textContent.replaceAll('–', '--'), 'svg')
                } catch(e) {
                    svg = `<pre class="error">${e}</pre>`
                }
            item.outerHTML = svg
            }
        </script>
    <% } %>
    ```

  - 添加启动选项（Theme配置下的`_config.yml`）

    ``` yml
    # hexo-graphviz
    graphviz:
        enable: true
    ```
  
  - 注意：可能在安装过程中npm自动升级了，导致之前配置的Nunjucks模板标签又和Markdown的部分标签冲突了，需要重新解决，详细见之前写的[Nunjucks Errors][3]的解决方案3。

## 示例

- 详细的可以查看[Graphviz的文档][1]

- 简单的有向图

    ``` graphviz
    digraph G {
        A -> B;
        B -> C;
        C -> D;
        D -> A;
    }
    ```

- 带节点标签和边标签的有向图

    ``` graphviz
    digraph G {
        A [label="Start"];
        B [label="Middle"];
        C [label="End"];
        
        A -> B [label="Step 1"];
        B -> C [label="Step 2"];
    }
    ```

- 带颜色和形状的无向图

    ``` graphviz
    graph G {
        A [shape=circle, color=red, fillcolor=lightblue, style=filled];
        B [shape=square, color=blue];
        C [shape=diamond, color=green];
        
        A -- B;
        B -- C;
        C -- A;
        A -- D -- E;
    }
    ```

- 二叉树

    ``` graphviz
    digraph G {
        bgcolor="beige"
        node [shape="record", height=.1]
        node0[label="<f0> | <f1> A | <f2>"]
        node1[label="<f0> | <f1> B | <f2>"]
        node2[label="<f0> | <f1> C | <f2>"]
        node0:f0 -> node1:f1
        node0:f2 -> node2:f1
    }
    ```

- 大括号子图

    ``` graphviz
    graph G {
        "数据结构三要素" -- {"数据的逻辑结构", "数据的存储结构", "数据的运算"}
    }
    ```

- 总示例

    ``` graphviz
    digraph G {
        label=<<B>Graphviz基本组成结构</B>>;
        labelloc=top;   // 标题位置
        bgcolor=transparent; // 背景透明
    
        node[shape=box];  // 结点形状为方形
        //edge[style=bold];
    
        // 独立出现的为结点或属性声明, 中括号前为结点名称
        graphviz[label="Graphviz"];
    
        subgraph{
            layout[label="Layouts"];
            script[label="Script Files"];
            api[label="APIs"];
            rank=same;
        }
    
        graphviz -> layout;
        graphviz -> script;
        graphviz -> api;
        
        // 设置子图
        api -> 
        subgraph{
            layout_etc[label="......"];
        }
    
        script ->
        subgraph{
            element[label="Elements"];
            attribute[label="Attributes"];
            rank=same;
        }
    
        layout ->
        subgraph{
            layout_dot[label="dot"];
            layout_neato[label="neato"];
        }
        
        element ->
        subgraph{
            ele_graph[label="Graph"];
            ele_node[label="Node"];
            ele_edge[label="Edge"];
        }
    }
    ```

<!-- markdownlint-disable-file MD025 MD033 -->
[1]: https://graphviz.org
[2]: https://graphviz.org/download/
[3]: https://www.lingzhicheng.cn/2021/08/26/NunjucksErrors/

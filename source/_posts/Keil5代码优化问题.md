---
title: Keil调试代码优化问题
date: 2020-12-08
tags: Debug
categories: AT89S51
copyright: true
---

- 在使用Keil进行调试的过程中，有的时候你可能会发现有几行代码怎么也得不到执行，下断点也会变成灰色感叹号。甚至有时候在做条件判断的时候，明明两个不相等的值却被判定为相等。
- 这是因为Keil对我们的代码进行了**优化**处理，将优化等级降低即可解决问题。![请输入图片描述][1]

  [1]: https://www.lingzhicheng.cn/usr/file/picture/Keil/Keil_optimism.png
  
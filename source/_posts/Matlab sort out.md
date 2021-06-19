---
title: Matlab sort out
date: 2021-06-19
tags:
   - sort out
categories: Matlab
---

## 1.Basics ##

- ### Scalars ###

``` matlab
a = 10
a = 10+1i       % 推荐使用1i
```

- ### Vectors ###

``` matlab
a = [1 3 5]     % 1行3列
a = [1,3,5]     % 使用,与否效果相同
a = 1:2:7       % begin:step:end
a = [1;3;5]     % 3行1列
```

- ### Matrices ###

``` matlab
A = [1,2,3;4,5,6;7,8,9] % 三行三列矩阵
```

- ### Operations ###

1. `*` 和`.*`的区别
    <font size=2>
    >在进行数值运算和数值乘矩阵运算，两者无区别，`a*b=a.*b; a*B=a.*B; B*a=B.*a`（其中小写字母表示数值，大写字母表示矩阵，下同）。
    >在处理矩阵乘矩阵时，`*`表示普通的矩阵乘法，要求前面矩阵的列数等于后面矩阵的行数；`.*`表示两个矩阵对应元素相乘，要求两个矩阵行数列数都相等。
    </font>

    ``` matlab
    >> [1,2,3]*[1,2;3,4;5,6]       % 矩阵乘法
    ans =
    22    28

    >> [1,2,3].*[4,5,6]            % 矩阵点乘
    ans =
    4    10    18
    ```

2. `/` 和`./`的区别
    <font size=2>
    >数值运行时，这两种没有区别，`a/b=a./b`
    >数值与矩阵运算，数值在前时只能用`./`;数值在后时`a/b=a./b`
    >矩阵与矩阵，`A/B`可以看作是`A*inv(B)`;`A./B`表示A矩阵和B矩阵对应元素相除，需要矩阵维度保持一致。
    >另外有`\`和`.\`表示前除，类似小学时候详细区的分除和被除运算，一般习惯不使用这种。
    </font>

    ``` matlab
    >> [4,5]/[1,2;3,4]                    % 矩阵除法
    ans =
    -0.5000    1.5000

    >> [4,5,6]./[1,2,3]                   % 矩阵点除
    ans =
    4.0000    2.5000    2.0000
    ```

3. `A(1,1)`、`A(:,1)`、`A(1,:)`、`A(:,:)`

4. `[1+2*1i,3+4*1i]'`和`[1+2*1i,3+4*1i].'`都表示矩阵的转置,实数情况下`A'`和`A.'`没有区别；复数情况下`A'`会产生`A`的转置共轭矩阵，`A.'`产生普通的转置矩阵。
   >`[1+2*1i,3+4*1i]'=[1-2*1i;3-4*1i]`
   >`[1+2*1i,3+4*1i].'=[1+2*1i;3+4*1i]`

- ### Special number ###

    `pi`
    `Inf`、`inf`:infinity,i.e.,1/0
    `NaN`、`nan`:not a number,e.g.,0/0
    `eps`:accuracy of the matlab,eps= 2-52 ≈ 2.2204×10-16

- ### General functions ###

  - Trigonometric functions:
    `cos(x)`、`sin(x)`、`tan(x)`
  - Complex:
    z = a+bi = $re^{i\theta}$
    `real(z)`、`imag(z)`、`abs(z)`、`angle(z)`、`conj(z)`
  - Exponetial and logarithm functions
    | Name | Description | Name | Description |
    |-------|-------|-------|-------|
    |  `exp(x)`  |  $e^{x}$  |  `log1p(x)`  |  ln(1+x)  |
    |  `pow2(x)`  |  $2^{x}$  |  `log2(x)`  |  $log_{2}{x}$  |
    |  `log(x)`  |  lnx  |  `log10(x)`  |  $log_{10}{x}$、$lg(x)$  |

  - Array and matrix
    | Name | Description | Name | Description |
    |-------|-------|-------|-------|
    |  `ones`  |  All one  |  `zeros`  |  All zeros  |
    |  `length`  |  length  |  `size`  |  Dimension of array  |
    |  `sum`  |  sum  |  `mean`  |  Mean of array  |
    |  `reshape`  |  Reshape the array  |  `sort`  |  Sort the array  |
    |  `min`  |  minmum  |  `max`  |  maxmum  |

  - Rounding functions
    `round`、`fix`、`floor`、`ceil`
  - Figure plotting
    `plot()`、`subplot(n,m,i)`、`figure(i)`、`xlabel()`、`ylabel()`
    `title()`、`legend()`、`hold on`、`grid on`、`semilogy()`...
  - Function format
    `function [outputs]=function_name(inputs)`

<!-- markdownlint-disable-file MD033 -->

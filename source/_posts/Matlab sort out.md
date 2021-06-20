---
title: Matlab sort out
date: 2021-06-19
tags:
   - sort out
categories: Matlab
---

Personal matlab sort out finally.
<!--more-->

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

   ```matlab
   [1+2*1i,3+4*1i]'=[1-2*1i;3-4*1i]
   [1+2*1i,3+4*1i].'=[1+2*1i;3+4*1i]
   ```

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
    |:-------:|:-------:|:-------:|:-------:|
    |  `exp(x)`  |  $e^{x}$  |  `log1p(x)`  |  ln(1+x)  |
    |  `pow2(x)`  |  $2^{x}$  |  `log2(x)`  |  $log_{2}{x}$  |
    |  `log(x)`  |  lnx  |  `log10(x)`  |  $log_{10}{x}$、$lg(x)$  |

  - Array and matrix
    | Name | Description | Name | Description |
    |:-------:|:-------:|:-------:|:-------:|
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

## 2.Signals and linear systems ##

- ### Basic sequence ###

  - Implulse signal and sequence
  
  ``` matlab
  t = -5:0.01:5;
  y = (t==0);
  subplot(1,2,1);
  plot(t, y, 'r’);

  n = -5:5;
  x = (n==0);
  subplot(1,2,2);
  stem(n, x);
  ```

  - Step signal and sequence

  ``` matlab
  t =-5:0.01:5;
  y=(t>=0);
  subplot(1,2,1);
  plot(t, y, 'r')

  n = -5:5;
  x = (n>=0);
  subplot(1,2,2);
  stem(n, x);
  ```

  - Real exponential sequence
  x(n)=$a^n$, $\forall n$,  $a\in R$

  ``` matlab
  n=[ns:nf];
  x=a.^n;
  ```

  - Exponential sequence
  x(n)=$e^{(\delta +j\omega)n}$

  ``` matlab
  n=[ns:nf];
  x=exp((delta+jw)*n);
  ```

  - Sinine, cosine sequence
  $x(n)=cos(\omega n+\theta)$

  ``` matlab
  n=[ns:nf];
  x=cos(w*n+sita);
  ```

  - Other signal generation functions
  
  | Name | Description | Name | Description |
  |:-------:|:-------:|:-------:|:-------:|
  |  `sawtooth`  |  Sawtooth or triangle wave  |  `pulstran`  |  Pulse train  |
  |  `square`  |  Square wave  |  `rectpule`  |  Aperiod squre wave  |
  |  `sinc`  |  Sinc wave  |  `tripuls`  |  Aperiod triangle wave  |

- ### Signal Operations ###
  
  - Moving
  y(n)=x(n-m)

  ```matlab
  y(n)=x(n-m)
  ```

  - Periodic extension
  $y(n)=x((n))_{M}$

  ```matlab
  y(n)=x(mod(n,M)+1)
  ```

  - Flipping
  y(n)=x(-n)

  ```matlab
  y=fliplr(x)
  ```

  - Correlation with two sequences
  $y(m)=\sum^n x_1(n+m)x_2^*(n)$

  ```matlab
  y=xcorr(x1,x2)
  ```
  
  - Cummulative sum
  $y(n)=\sum_{i=1}^n x(i)$

  ```matlab
  y=cumsum(x)
  ```

  - Convolution of two sequences
  $y(n)=x_1(n)*x_2(n)=\sum^mx_1(m)x_x(n-m)$

  ```matlab
  y=conv(x1,x2)
  ```

  - Convolution of two _continuous-time_ signals：
  $f(t)=\int_{-\infty}^{+\infty}f_1(\tau)f_2(t-\tau)d\tau$
  $f(t)=\sum_{k=-\infty}^{+\infty}f_1(k\Delta)f_2(t-k\Delta)\Delta$

  ```matlab
  y=conv(x1,x2)*dt
  ```

- ### Fourier transform ###

  - Continuous-time，continuous-frequency：FT
  
  | T2F | F2T |
  |:-------:|:-------:|
  |  $X(f)=\int_{-\infty}^{+\infty}x(t) e^{-j2\pi ft}dt$  |  $x(t)=\int_{-\infty}^{+\infty}X(f) e^{j2\pi ft}df$  |
  - Discrete-time, discrete-frequency: DFT / FFT
  
  | T2F | F2T |
  |:-------:|:-------:|
  |  $X(k)=\sum_{n=0}^{N-1}x(n) e^{-j\frac{2pi}{N}nk}$  |  $x(t)=\frac{1}{N}\sum_{n=0}^{N-1}X(k) e^{j\frac{2pi}{N}nk}$  |

- ### Energy and Power ###
  
  - Energy
    - T-domain
    $x[n]=x(n\Delta t)$
    $E=\int_{-\infty}^{+\infty} |x(t)|^2dt \approx\sum_{n=0}^M x[n]·x^*[n]·\Delta t$

    ``` matlab
    % E=sum(x.*conj(x))*dt;
    E=sum(abs(x).^2)*dt;
    ```

    - F-domain
    $X[k]=X(k\Delta f)$
    $E=\int_{-\infty}^{+\infty} |X(f)|^2df \approx\sum_{k=0}^{k-1} X[k]·X^*[k]·\Delta f$

    ``` matlab
    % E=sum(x.*conj(x))*df;
    E=sum(abs(x).^2)*df;
    ```

  - Power
    - T-domain
    $P=\lim_{T\rightarrow\infty}\frac{1}{T}\int_0^T |x(t)|^2dt \approx\frac{1}{N}\sum_{n=0}^{N-1} |x[n]|^2$

    ``` matlab
    % P=sum(x.*conj(x))/N;
    P=sum(abs(x).^2)/N;
    ```

    - F-domain
    _$X_Tf$ is the spectrum of x(t) within [0,T]_
    $P=\lim_{T\rightarrow\infty}\frac{1}{T}\int_{-\infty}^{+\infty} |X_T(f)|^2df \approx\frac{1}{N}\sum_{k=0}^{K-1} |X[k]|^2\Delta f$

    ``` matlab
    % P=sum(x.*conj(x))*df/T;
    P=sum(abs(x).^2)*df/T;
    ```

- ### Autocorrelation and Power spectral density(PSD) ###
  
  - Autocorrelation
    $x[n]=x(n\Delta t)$
    $R(\tau)=\int_{-\infty}^{+\infty} x(t)x^* (t+\tau)dt \approx\sum_{n=0}^M x[n]·x^*[n+\tau]·\Delta t$
    $R_T(\tau)=\lim_{T\rightarrow\infty}\frac{R(\tau)}{T}$

    ```matlab
    R(tau)=sum(x(t).*conj(x(t+tau))*dt/(N*dt);
    ```

  - PSD
    $P_T(f)=\lim_{T\rightarrow\infty}\frac{|X_T(f)|^2}{T}$
  - Wiener-Khinchin theorem
    $FT(R_T(\tau))=P_T(f)$

## 3.Randon process and analog modulation ##

- ### ex3 ###

  - ex3.1
    - ex3.1.1

## 4.Baseband signal transmission I ##

- ### ex4 ###

## 5.Baseband signal transmission II ##

- ### ex5 ###

## 6.Digital Transmission via carrier modulation I ##

- ### ex6 ###

## 7.Digital Transmission via carrier modulation II ##

- ### ex7 ###

<!-- markdownlint-disable-file MD033 -->

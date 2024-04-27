---
title: Latex Grammar
date: 2023-02-09
tags: 
    - Latex
categories: Summary
---

$LaTeX$ (/ˈlɑːtɛx/ LAH-tekh or /ˈleɪtɛx/ LAY-tekh, often stylized as LATEX) is a **software system** for document preparation. When writing, the writer uses plain text as opposed to the formatted text found in WYSIWYG word processors like Microsoft Word, LibreOffice Writer and Apple Pages. The writer uses **markup tagging conventions** to define the general structure of a document to stylise text throughout a document (such as bold and italics), and to add citations and cross-references. A TeX distribution such as TeX Live or MiKTeX is used to produce an output file (such as PDF or DVI) suitable for printing or digital distribution.

<!--more-->

----------

## Introduction

- $ LaTeX $ is widely used in **academia** for the communication and publication of scientific documents in many fields, including **mathematics, computer science, engineering, physics, chemistry, economics, linguistics, quantitative psychology, philosophy, and political science**. It also has a prominent role in the preparation and publication of books and articles that contain complex multilingual materials, such as Arabic and Greek. LaTeX uses the TeX typesetting program for formatting its output, and is itself written in the **$TeX$ macro language**.

- $ LaTeX $ can be used as **a standalone document preparation system**, or as **an intermediate format**. In the latter role, for example, it is sometimes used as part of a pipeline for translating `DocBook` and `other XML-based formats` to `PDF`. The typesetting system offers programmable desktop publishing features and extensive facilities for automating most aspects of typesetting and desktop publishing, including **numbering and cross-referencing of tables and figures, chapter and section headings, graphics, page layout, indexing and bibliographies**.

## Install :sweat_smile:
> Leave a hole here, and I will attach an installation tutorial when I have time later.
> Okey, let's go! I used to use `VsCode` + `TexLive`, which is more convenient, because `VsCode` has a built-in `LaTeX Workshop` extension, which can be used directly after installation. 
> For the convenience of reading, a seprarate article about installation and use is written, and the link is [here](https://www.lingzhicheng.cn/2023/03/27/TexLive+VsCode/).

## Attention:exclamation:	

> Before starting, it is important to note that:
>
> $ Markdown $ is rendered as `HTML` and its syntax conflicts with $ LaTeX $, which affects the effect of the article. E.g. braces should be avoided repeatedly, spaces should be added.

``` latex
}} % It's bad.
} } % It's good!
```

## Formula Insertion

- Formulas are divided into inline formulas and formula blocks, the former is embedded in a line, and the latter is a separate line.

- Inline formulas
$ f(x) = a+b $

``` latex
$ f(x) = a+b $
```

​​​​​

- Formula blocks

$$
x+1 = 2*3
$$
```
$$
x+1 = 2*3
$$
```

- Numbering

$$
x+1 = 2*3 \tag{1.1}
$$

``` latex
$$
x+1 = 2*3 \tag{1.1}
$$
```

## Basic operations

### Four operations and brackets

> The basic four-rule arithmetic can be done directly with the keyboard's +-*

- `\times` stands for $ \times $, `\div` stands for $ \div $, `\cdot` stands for $ \cdot $

Example: `$ a + b - c*2 + d/e = 5 \times 3 + 8 \div 4 - f \cdot g $`
$ a + b - c*2 + d/e = 5 \times 3 + 8 \div 4 - f \cdot g $

- The curly braces need to be escaped with a slash.

Example: `$ (1+2) \quad [1+2] \quad \{1+2\} $`
$ (1+2) \quad [1+2] \quad \{1+2\} $

- The `\left` and `\right` commands are used to generate auto-match height brackets or parenthetical characters.

Example: `$ (\dfrac{1}{2})^2 \quad \left( \dfrac{1}{2} \right) ^2 $`
$ (\dfrac{1}{2})^2 \quad \left( \dfrac{1}{2} \right) ^2 $

- `<>`, `Abs` and `Norms()`

`$ \left\langle \dfrac ab \right\rangle \quad \left| \dfrac ab \right| \quad \left\| \dfrac ab \right\| $`
$ \left\langle \dfrac ab \right\rangle \quad \left| \dfrac ab \right| \quad \left\| \dfrac ab \right\| $

- Rounding function

`$ \left\lfloor \dfrac ab \right\rfloor \quad \left\lceil \dfrac ab \right\rceil $`
$ \left\lfloor \dfrac ab \right\rfloor \quad \left\lceil \dfrac ab \right\rceil $

- Slashes with arrows

`\left/ \dfrac ab \right\backslash`
$ \left/ \dfrac ab \right\backslash $

`\left\uparrow \dfrac ab \right\downarrow \quad \left\Uparrow \dfrac ab \right\Downarrow \quad \left\updownarrow \dfrac ab \right\Updownarrow`
$ \left\uparrow \dfrac ab \right\downarrow \quad \left\Uparrow \dfrac ab \right\Downarrow \quad \left\updownarrow \dfrac ab \right\Updownarrow $

- Mixed parentheses

`\left[ \dfrac{1}{2}, 1 \right) \quad \left\langle \psi \right|`
$ \left[ \dfrac{1}{2}, 1 \right) \quad \left\langle \psi \right| $
 
- Single parentheses

> Use `\left.` or `\right.` to omit.

``` latex
$$
\left\{
\begin{array}{l}
x+y=1\\
x-y=2
\end{array}
\right.
$$ 
```
$$
\left\{
\begin{array}{l}
x+y=1\\
x-y=2
\end{array}
\right.
$$ 

`$ \left. \dfrac ab \right\} $`
$ \left. \dfrac ab \right\} $

- The **size** of the brackets

> Use `\big`, `\Big`, `\bigg`, `\Bigg` to control.

`$ \left\{ \left[ \left( \left| \left\| a \right\| +1 \right| -2 \right) +3 \right] -4 \right\}*5 $`
$ \left\{ \left[ \left( \left| \left\| a \right\| +1 \right| -2 \right) +3 \right] -4 \right\}*5 $
​
`$ \Bigg\{ \bigg[ \Big( \big| \left\| a \right\| +1 \big| -2 \Big) +3 \bigg] -4 \Bigg\}*5 $`
$ \Bigg\{ \bigg[ \Big( \big| \left\| a \right\| +1 \big| -2 \Big) +3 \bigg] -4 \Bigg\}*5 $

### Superscript and subscript

- Use `_` for subscript, `^` for superscript, and handle only one character, and multiple characters enclosed in `{}`. `'` indicates derivation, multiple can be used.

Example: `x_1^{3,4} + x_{1,2}^{2} = y_0 + y'_{6} + y''`
$ x_1^{3,4} + x_{1,2}^{2} = y_0 + y'_{6} + y'' $

### Fractional and radical

- Use `frac{a}{b}` for fractions, `sqrt` for square roots, and `sqrt[n]` for nth roots.

Example: `\frac{x}{2} + \sqrt x = \sqrt[3] {x^2+y^2}`
$ \frac{x}{2} + \sqrt x = \sqrt[3] {x^2+y^2} $

## Symbols, functions and special characters

### Tones and markers

- Tones
`\dot{a}` $ \dot{a} $
`\ddot{a}` $ \ddot{a} $
`\acute{a}`  $ \acute{a} $
`\grave{a}` $ \grave{a} $
`\check{a}` $ \check{a} $
`\breve{a}` $ \breve{a} $
`\tilde{a}` $ \tilde{a} $
`\widetilde{a}` $ \widetilde{a} $
`\bar{a}` $ \bar{a} $
`\mathring{a}` $ \mathring{a} $

- Markers
`\hat{a}` $ \hat{a} $
`\widehat{a}` $ \widehat{a} $
`\vec{a}` $ \vec{a} $

### Basic functions

- Exponential function

Example: `\exp_a b, \exp b, x^2`
$ \exp_a b, \exp b, x^2 $

- Logarithm

Example: `\log a, \lg b, \ln c, \log_{2} d`
$ \log a, \lg b, \ln c, \log_{2} d $

- Trigonometric function

Example: `\sin a, \cos b, \tan c, \cot d, \sec e, \csc f`
$ \sin a, \cos b, \tan c, \cot d, \sec e, \csc f $

- Inverse trigonometric functions

Example: `\arcsin a, \arccos b, \arctan c`
​​$ \arcsin a, \arccos b, \arctan c $

> `\arccot`, `\arcsec` and `\arccsc` in $Latex$ is not defined and can be replaced by `\text{arccot}x` or `\operatorname{function}` $-->$ $\text{arccot}x$

- Hyperbolic functions

Example: `\sinh a, \cosh b, \tanh c, \coth d`
$ \sinh a, \cosh b, \tanh c, \coth d $

- Absolute value

Example: `\left\vert a \right\vert, |b|, | \dfrac cd |, \left| \dfrac cd \right|`
$ \left\vert a \right\vert, |b|, | \dfrac cd |, \left| \dfrac cd \right| $
 

- Maximum and minimum

Example:`\max(x,y), \min(x,y)`​​
$ \max(x,y), \min(x,y) $

- Others

Example: `\operatorname{function}`
$ \operatorname{sgn} x $

### Limits

`\min x, \max y, \inf a, \sup b`
$ \min x, \max y, \inf a, \sup b $

`\lim x, \liminf y, \limsup z`
$ \lim x, \liminf y, \limsup z $

`\dim p, \deg q, \det m, \ker\phi`
​$ \dim p, \deg q, \det m, \ker\phi $

### Projection

`\Pr j, \hom l, \lVert z \rVert, \arg z`
$ \Pr j, \hom l, \lVert z \rVert, \arg z $

### Differential and derivative

> Treatment of non-italic derivative $ \mathrm{d}x $ with `\mathrm{d}x`

`dx, \mathrm{d}x, \partial t, \nabla\psi`
​$ dx, \mathrm{d}x, \partial t, \nabla\psi $

`\mathrm{d}y/\mathrm{d}x, \frac{\mathrm{d}y}{\mathrm{d}x}, \frac{\partial^2}{\partial x \partial y}z`
$ \mathrm{d}y/\mathrm{d}x, \frac{\mathrm{d}y}{\mathrm{d}x}, \frac{\partial^2}{\partial x \partial y}z $
 
 
`', \prime, \backprime, f^\prime, f', f'', f^{(3)}, \dot y, \ddot y`
$ ', \prime, \backprime, f^\prime, f', f'', f^{(3)}, \dot y, \ddot y$

### Special symbols

`\infty, \aleph, \complement, \backepsilon, \eth, \Finv, \hbar`
$ \infty, \aleph, \complement, \backepsilon, \eth, \Finv, \hbar $

`\imath, \jmath, \Bbbk, \ell, \mho, \wp, \Re, \circledS​`
$ \imath, \jmath, \Bbbk, \ell, \mho, \wp, \Re, \circledS​ $

### Mod

`\pmod{m}, a \bmod b`
$ \pmod{m}, a \bmod b $

`\gcd(m,n), \operatorname{lcm}(m,n)`
$ \gcd(m,n), \operatorname{lcm}(m,n) $​

`\mid, \nmid, \shortmid, \nshortmid`
$ \mid, \nmid, \shortmid, \nshortmid $

### Sqrt

`\surd, \sqrt{2}, \sqrt[n]{}, \sqrt[3]{x^2+y^2}`
$ \surd, \sqrt{2}, \sqrt[n]{}, \sqrt[3]{x+y} $

### Operations

` +, -, \pm, \mp, \dotplus `
$ +, -, \pm, \mp, \dotplus$ 

` *, /, \times, \div, \divideontimes, \cdot, \ast, \backslash `
$ *, /, \times, \div, \divideontimes, \cdot, \ast, \backslash $

` \star, \circ, \bullet `
$ \star, \circ, \bullet $

` \boxplus, \boxminus, \boxtimes, \boxdot `
$ \boxplus, \boxminus, \boxtimes, \boxdot $

`\oplus, \ominus, \otimes, \oslash, \odot`
$ \oplus, \ominus, \otimes, \oslash, \odot $

`\circleddash, \circledcirc, \circledast`
$ \circleddash, \circledcirc, \circledast $
​
`\bigoplus, \bigotimes, \bigodot`
$ \bigoplus, \bigotimes, \bigodot $

### Sets

`\{, \}, \emptyset, \varnothing`
​$ \{, \}, \empty, \varnothing $


`\in, \notin, \not\in, \ni, \not\ni`
​​$ \in, \notin, \not\in, \ni, \not\ni $

`\cap, \Cap, \sqcap, \bigcap`
​$ \cap, \Cap, \sqcap, \bigcap $

`\cup, \Cup, \sqcup, \bigcup, \bigsqcup, \uplus, \biguplus`
$ \cup, \Cup, \sqcup, \bigcup, \bigsqcup, \uplus, \biguplus $

`\setminus, \smallsetminus`
$ \setminus, \smallsetminus $

`\subset, \Subset, \sqsubset`
$ \subset, \Subset, \sqsubset $

`\supset, \Supset, \sqsupset`
$ \supset, \Supset, \sqsupset $

`\subseteq, \nsubseteq, \subsetneq, \varsubsetneq, \sqsubseteq`
$ \subseteq, \nsubseteq, \subsetneq, \varsubsetneq, \sqsubseteq $

`\supseteq, \nsupseteq, \supsetneq, \varsupsetneq, \sqsupseteq`
$ \supseteq, \nsupseteq, \supsetneq, \varsupsetneq, \sqsupseteq $

`\subseteqq, \nsubseteqq, \subsetneqq, \varsubsetneqq`
$ \subseteqq, \nsubseteqq, \subsetneqq, \varsubsetneqq $

### Relationship symbol

`=, \ne, \neq, \equiv, \not\equiv`
$ =, \ne, \neq, \equiv, \not\equiv $

`\doteq, \doteqdot, :=, \overset{\underset{\mathrm{def} } {} } {=}`
$ \doteq, \doteqdot, :=, \overset{\underset{\mathrm{def} } {} } {=} $

`\sim, \nsim, \backsim, \thicksim, \simeq, \backsimeq, \eqsim, \cong, \ncong`
$ \sim, \nsim, \backsim, \thicksim, \simeq, \backsimeq, \eqsim, \cong, \ncong $

`\approx, \thickapprox, \approxeq, \asymp, \propto, \varpropto`
$ \approx, \thickapprox, \approxeq, \asymp, \propto, \varpropto $

`<, \nless, \ll, \not\ll, \lll, \not\lll, \lessdot`
$ <, \nless, \ll, \not\ll, \lll, \not\lll, \lessdot $

`>, \ngtr, \gg, \not\gg, \ggg, \not\ggg, \gtrdot`
$ >, \ngtr, \gg, \not\gg, \ggg, \not\ggg, \gtrdot $

`\le, \leq, \lneq, \leqq, \nleq, \nleqq, \lneqq, \lvertneqq`
$ \le, \leq, \lneq, \leqq, \nleq, \nleqq, \lneqq, \lvertneqq $

`\ge, \geq, \gneq, \geqq, \ngeq, \ngeqq, \gneqq, \gvertneqq`
$ \ge, \geq, \gneq, \geqq, \ngeq, \ngeqq, \gneqq, \gvertneqq $

`\lessgtr, \lesseqgtr, \lesseqqgtr, \gtrless, \gtreqless, \gtreqqless`
$ \lessgtr, \lesseqgtr, \lesseqqgtr, \gtrless, \gtreqless, \gtreqqless $

`\leqslant, \nleqslant, \eqslantless`
$ \leqslant, \nleqslant, \eqslantless $

`\geqslant, \ngeqslant, \eqslantgtr`
$ \geqslant, \ngeqslant, \eqslantgtr $

`\lesssim, \lnsim, \lessapprox, \lnapprox`
$ \lesssim, \lnsim, \lessapprox, \lnapprox $

`\gtrsim, \gnsim, \gtrapprox, \gnapprox`
$ \gtrsim, \gnsim, \gtrapprox, \gnapprox $

`\prec, \nprec, \preceq, \npreceq, \precneqq`
​$ \prec, \nprec, \preceq, \npreceq, \precneqq $

`\succ, \nsucc, \succeq, \nsucceq, \succneqq`
$ \succ, \nsucc, \succeq, \nsucceq, \succneqq $

`\preccurlyeq, \curlyeqprec`
$ \preccurlyeq, \curlyeqprec $

`\succcurlyeq, \curlyeqsucc`
$ \succcurlyeq, \curlyeqsucc $

`\precsim, \precnsim, \precapprox, \precnapprox`
$ \precsim, \precnsim, \precapprox, \precnapprox $

`\succsim, \succnsim, \succapprox, \succnapprox`
$ \succsim, \succnsim, \succapprox, \succnapprox $

### Geometric symbols

`\parallel, \nparallel, \shortparallel, \nshortparallel`
$ \parallel, \nparallel, \shortparallel, \nshortparallel $

`\perp, \angle, \sphericalangle, \measuredangle, 45^\circ`
$ \perp, \angle, \sphericalangle, \measuredangle, 45^\circ$

`\Box, \blacksquare, \diamond, \Diamond, \lozenge, \blacklozenge, \bigstar, \bigcirc`
​$ \Box, \blacksquare, \diamond, \Diamond, \lozenge, \blacklozenge, \bigstar, \bigcirc $

`\triangle, \triangledown ,\bigtriangleup, \bigtriangledown, \vartriangle`
​$ \triangle, \triangledown ,\bigtriangleup, \bigtriangledown, \vartriangle $

`\blacktriangle, \blacktriangledown, \blacktriangleleft, \blacktriangleright`
$ \blacktriangle, \blacktriangledown, \blacktriangleleft, \blacktriangleright $

### Logical symbols

`\forall, \not\forall, \exists, \nexists`
​$ \forall, \not\forall, \exists, \nexists $

`\because, \therefore, \And`
$ \because, \therefore, \And $

`\lor, \vee, \curlyvee, \bigvee`
$ \lor, \vee, \curlyvee, \bigvee $
​
`\land, \wedge, \curlywedge, \bigwedge`
$ \land, \wedge, \curlywedge, \bigwedge $

> Use `\lor, \land` instead of `\or, \and`

`\bar{x}, \bar{abc}, \overline{x}, \overline{abc}`
$ \bar{x}, \bar{abc}, \overline{x}, \overline{abc} $

`\lnot, \neg, \not\operatorname{R}, \bot, \top`
$ \lnot, \neg, \not\operatorname{R}, \bot, \top $

`\vdash, \dashv, \vDash, \Vdash, \Vvdash, \models`
​​$ \vdash, \dashv, \vDash, \Vdash, \Vvdash, \models $

`\nvdash, \nVdash, \nvDash, \nVDash`
$ \nvdash, \nVdash, \nvDash, \nVDash $

`\ulcorner, \urcorner, \llcorner, \lrcorner`
$ \ulcorner, \urcorner, \llcorner, \lrcorner $

### Arrowhead

`\to, \rightarrow, \nrightarrow, \longrightarrow`
$ \to, \rightarrow, \nrightarrow, \longrightarrow $

`\gets, \leftarrow, \nleftarrow, \longleftarrow`
$ \gets, \leftarrow, \nleftarrow, \longleftarrow $

`\leftrightarrow, \nleftrightarrow, \longleftrightarrow`
$ \leftrightarrow, \nleftrightarrow, \longleftrightarrow $ 

`\uparrow, \downarrow, \updownarrow`
$ \uparrow, \downarrow, \updownarrow $

`\Rightarrow, \nRightarrow, \Longrightarrow, \implies`
$ \Rightarrow, \nRightarrow, \Longrightarrow, \implies $

`\Leftarrow, \nLeftarrow, \Longleftarrow`
$ \Leftarrow, \nLeftarrow, \Longleftarrow $

`\Leftrightarrow, \nLeftrightarrow, \Longleftrightarrow, \iff`
$ \Leftrightarrow, \nLeftrightarrow, \Longleftrightarrow, \iff $

`\Uparrow, \Downarrow, \Updownarrow`
$ \Uparrow, \Downarrow, \Updownarrow $

`\Rrightarrow, \Lleftarrow`
$ \Rrightarrow, \Lleftarrow $

`\nearrow, \swarrow, \nwarrow, \searrow`
$ \nearrow, \swarrow, \nwarrow, \searrow $

`\mapsto, \longmapsto`
$ \mapsto, \longmapsto $

`\rightharpoonup, \rightharpoondown, \leftharpoonup, \leftharpoondown, \upharpoonleft, \upharpoonright, \downharpoonleft, \downharpoonright, \rightleftharpoons, \leftrightharpoons`
$ \rightharpoonup, \rightharpoondown, \leftharpoonup, \leftharpoondown, \upharpoonleft, \upharpoonright, \downharpoonleft, \downharpoonright, \rightleftharpoons, \leftrightharpoons $

`\curvearrowleft, \circlearrowleft, \Lsh, \upuparrows, \rightrightarrows, \rightleftarrows, \rightarrowtail, \looparrowright`
$ \curvearrowleft, \circlearrowleft, \Lsh, \upuparrows, \rightrightarrows, \rightleftarrows, \rightarrowtail, \looparrowright $

`\curvearrowright, \circlearrowright, \Rsh, \downdownarrows, \leftleftarrows, \leftrightarrows, \leftarrowtail, \looparrowleft`
$ \curvearrowright, \circlearrowright, \Rsh, \downdownarrows, \leftleftarrows, \leftrightarrows, \leftarrowtail, \looparrowleft $

`\hookrightarrow, \hookleftarrow, \multimap, \leftrightsquigarrow, \rightsquigarrow, \twoheadrightarrow, \twoheadleftarrow`
$ \hookrightarrow, \hookleftarrow, \multimap, \leftrightsquigarrow, \rightsquigarrow, \twoheadrightarrow, \twoheadleftarrow $

### Ellipsis

`\cdots, \ldots, \vdots, \ddots`
$ \cdots, \ldots, \vdots, \ddots $

### Others

`\amalg, \%, \&, \dagger, \ddagger`
​​​$ \amalg, \%, \&, \dagger, \ddagger $

`\smile, \frown, \wr, \triangleleft, \triangleright`
$ \smile, \frown, \wr, \triangleleft, \triangleright $

`\diamondsuit, \heartsuit, \clubsuit, \spadesuit, \Game, \flat, \natural, \sharp`
$ \diamondsuit, \heartsuit, \clubsuit, \spadesuit, \Game, \flat, \natural, \sharp $

`\diagup, \diagdown, \centerdot, \ltimes, \rtimes, \leftthreetimes, \rightthreetimes`
$ \diagup, \diagdown, \centerdot, \ltimes, \rtimes, \leftthreetimes, \rightthreetimes $

`\eqcirc, \circeq, \triangleq, \bumpeq, \Bumpeq, \doteqdot, \risingdotseq, \fallingdotseq`
$ \eqcirc, \circeq, \triangleq, \bumpeq, \Bumpeq, \doteqdot, \risingdotseq, \fallingdotseq $

`\intercal, \barwedge, \veebar, \doublebarwedge, \between, \pitchfork`
$ \intercal, \barwedge, \veebar, \doublebarwedge, \between, \pitchfork $

`\vartriangleleft, \ntriangleleft, \vartriangleright, \ntriangleright`
$ \vartriangleleft, \ntriangleleft, \vartriangleright, \ntriangleright $

`\trianglelefteq, \ntrianglelefteq, \trianglerighteq, \ntrianglerighteq`
$ \trianglelefteq, \ntrianglelefteq, \trianglerighteq, \ntrianglerighteq $

### Vector

`\vec{a}, \boldsymbol{b}, \overleftarrow{ab}, \overrightarrow{cd}, \overleftrightarrow{ab}, \widehat{abc}`

$ \vec{a}, \boldsymbol{b}, \overleftarrow{ab}, \overrightarrow{cd}, \overleftrightarrow{ab}, \widehat{abc} $
​​

### Up and down lines

`\overset{\frown} {AB}, \overline {abc}, \underline{def}`
$ \overset{\frown} {AB}, \overline {abc}, \underline{def} $

`\overbrace{1+2+\cdots+n} \quad \overbrace{1+2+\cdots+n}^{n}`
$ \overbrace{1+2+\cdots+n} \quad \overbrace{1+2+\cdots+n}^{n} $
 
`\begin{matrix} n \\ \overbrace{1+2+\cdots+n} \end{matrix}`
$ \begin{matrix} n \\ \overbrace{1+2+\cdots+n} \end{matrix} $

`\underbrace{a+b+\cdots+z} \quad \underbrace{a+b+\cdots+z}_{26}`
$ \underbrace{a+b+\cdots+z} \quad \underbrace{a+b+\cdots+z}_{26} $

`\begin{matrix} \underbrace{a+b+\cdots+z} \\ 26 \end{matrix}`
$ \begin{matrix} \underbrace{a+b+\cdots+z} \\ 26 \end{matrix} $ 

### Big operations

> Inlines formulas and formula blocks are different.

`$ \sum_{i=1}^{n} i^2 $`
$ \sum_{i=1}^{n} i^2 $

```
$$
\sum_{i=1}^{n} i^2
$$
```
$$
\sum_{i=1}^{n} i^2
$$

To display an inline formula as positive top and bottom, you can use the limits command to follow the operator:

`$ \sum\limits_{i=1}^{n} i^2 $`
$ \sum\limits_{i=1}^{n} i^2 $
 

To display the formula block as top-right-right, you can use the first-order frameless matrix form or use the nolimits command:

``` latex
$$
\begin{matrix} \sum_{i=1}^{n} i^2 \end{matrix}
$$
```
$$
\begin{matrix} \sum_{i=1}^{n} i^2 \end{matrix}
$$
 
``` latex
$$
\sum\nolimits_{i=1}^{n} i^2
$$
```
$$
\sum\nolimits_{i=1}^{n} i^2
$$

- Accumulate and multiply

`\sum_{i=1}^{n} i^2 \quad \prod_{i=1}^{n} x_i`
$$
\sum_{i=1}^{n} i^2 \quad \prod_{i=1}^{n} x_i
$$

- Limit

`\lim_{x \to \infty} (1+\frac{1}{x})^x = e`
$$
\lim_{x \to \infty} (1+\frac{1}{x})^x = e
$$
 
 
- Ordinary integration

`\int_{a}^{b} e^x \, \mathrm{d}x`
$$
\int_{a}^{b} e^x \, \mathrm{d}x
$$

- Multiple integration

`\iint_{D} f(x,y) \, \mathrm{d}\sigma \quad \iiint_{\Omega} f(x,y,z) \, \mathrm{d}V`
$$
\iint_{D} f(x,y) \, \mathrm{d}\sigma \quad \iiint_{\Omega} f(x,y,z) \, \mathrm{d}V
$$

- Closure curve integral

`\oint_{L} f(x,y) \, \mathrm{d}s`
$$
\oint_{L} f(x,y) \, \mathrm{d}s
$$

- Others

`\int, \iint, \iiint, \oint`
$ \int, \iint, \iiint, \oint $

- Intersection, union and reunion

`\bigcap_{i=1}^{n} A_i \quad , \bigcup_{j=1}^{m} B_j , \coprod_{i \in I} A_i`
$$
\bigcap_{i=1}^{n} A_i \quad , \bigcup_{j=1}^{m} B_j , \coprod_{i \in I} A_i
$$


## Fractional

### Basic operations 

> You can use `\over` and `{}`

`x = { {-b \pm \sqrt{b^2-4ac} } \over {2a} }`
$$ x = { {-b \pm \sqrt{b^2-4ac} } \over {2a} } $$ 
 
### Typesetting

> Normal fractions are automatically scaled down in in-line formulas and displayed as full size in formula blocks.

`$ \frac{1}{2}=0.5 $`
$ \frac{1}{2}=0.5 $

``` latex
$$
\frac{1}{2}=0.5
$$
```
$$
\frac{1}{2}=0.5
$$
​
- The `\tfrac` command is used to make fractions appear as inline formula styles.
``` latex
$$
\tfrac{1}{2}=0.5
$$
```
$$
\tfrac{1}{2}=0.5
$$

- The `\dfrac` command is used to make fractions appear as formula block styles.

`$ \dfrac{1}{2}=0.5 $`
$ \dfrac{1}{2}=0.5 $
 
​

- Attention

In scenarios such as exponential functions, limits, and integrals, try not to use the frac command, and use / express as horizontal fractions.

``` latex
$$
e^{\frac{i\pi}{2} } \quad \int_{-\frac{\pi}{2} }^{\frac{\pi}{2} } \sin x \, \mathrm{d}x
% Bad
$$
```
$$
e^{\frac{i\pi}{2} } \quad \int_{-\frac{\pi}{2} }^{\frac{\pi}{2} } \sin x \, \mathrm{d}x
% Bad
$$

``` latex
$$
e^{i\pi/2} \quad \int_{-\pi/2}^{\pi/2} \sin x \, \mathrm{d}x
% Good
$$
```
$$
e^{i\pi/2} \quad \int_{-\pi/2}^{\pi/2} \sin x \, \mathrm{d}x
% Good
$$

### Continuous fractions

> Enter the conjunction with the `\cfrac` command, which automatically handles the height of the numerator and denominator.

``` latex
$$
\cfrac{1}{2 + \cfrac{1}{2 + \cfrac{1}{2 + \cdots} } }
$$
```
$$
\cfrac{1}{2 + \cfrac{1}{2 + \cfrac{1}{2 + \cdots} } }
$$
 
### Binomial coefficients

``` latex
$$
\mathrm{C}_n^r = \binom{n}{r}
$$
```
$$
\mathrm{C}_n^r = \binom{n}{r}
$$

> Enter with the binom command, tbinom to display binomial coefficients as inline formula styles, and dbinom to display binomial coefficients as formula block styles


``` latex
$$
\tbinom{n}{r} = \tbinom{n}{n-r}
$$
```
$$
\tbinom{n}{r} = \tbinom{n}{n-r}
$$

`$ \dbinom{n}{r} = \dfrac{n!}{k!\,(n-k)!} $`
$ \dbinom{n}{r} = \dfrac{n!}{k!\,(n-k)!} $
​

##  Matrices, conditional expressions and equations

- Grammar

``` latex
\begin{type}
content
\end{type}
type: matrix、pmatrix、bmatrix、Bmatrix、vmatrix、Vmatrix;
	coditional cases;
	equations aligned、alignedat.
```

> In content, the `&` symbol indicates the aligned content of each line, and `\\` indicates a line break at the end.

### Frameless matrix

- Separate matrix columns with `&` and matrix rows with `\\`

``` latex
$$
\begin{matrix}
a & b \\
c & d
\end{matrix}
$$
```
$$
\begin{matrix}
a & b \\
c & d
\end{matrix}
$$

### Box matrix

- `pmatrix` is parentheses, `bmatrix` is square brackets, `Bmatrix` is curly braces, `vmatrix` is vertical (determinant), and `Vmatrix` is double vertical bar.

- Enter ellipses using `cdots`, `ddots` and `vdots`.

``` latex
$$
\begin{pmatrix}
0 & \cdots & 0 \\
\vdots & \ddots & \vdots \\
0 & \cdots & 0 \\
\end{pmatrix}
$$
```
$$
\begin{pmatrix}
0 & \cdots & 0 \\
\vdots & \ddots & \vdots \\
0 & \cdots & 0 \\
\end{pmatrix}
$$

``` latex
$$
\begin{bmatrix}
a & b \\
c & d
\end{bmatrix}
\quad
\begin{Bmatrix}
a & b \\
c & d
\end{Bmatrix}
\quad
\begin{vmatrix}
a & b \\
c & d
\end{vmatrix}
\quad
\begin{Vmatrix}
a & b \\
c & d
\end{Vmatrix}
$$
```
$$
\begin{bmatrix}
a & b \\
c & d
\end{bmatrix}
\quad
\begin{Bmatrix}
a & b \\
c & d
\end{Bmatrix}
\quad
\begin{vmatrix}
a & b \\
c & d
\end{vmatrix}
\quad
\begin{Vmatrix}
a & b \\
c & d
\end{Vmatrix}
$$

### Matrix segmentation

- Add `|` to the definition where it needs to be divided.

``` latex
$$
\left[
\begin{array}{cc|c}
1 & 2 & 3 \\
4 & 5 & 6
\end{array}
\right]
$$
```
$$
\left[
\begin{array}{cc|c}
1 & 2 & 3 \\
4 & 5 & 6
\end{array}
\right]
$$

### Conditional expressions

- Use `&` to separate formulas from conditions.

``` latex
$$
f(n)=
\begin{cases}
n/2, & \text{if } n \text{ is even} \\
3n+1, & \text{if } n \text{ is odd}
\end{cases}
$$
```
$$
f(n)=
\begin{cases}
n/2, & \text{if } n \text{ is even} \\
3n+1, & \text{if } n \text{ is odd}
\end{cases}
$$

> Use `\text` to distinguish between text and mathematical characters

### Multi-line equation

- Use `\alignedat` to perform it
``` latex
$$
\begin{aligned}
f(x) & = (m+n)^2 \\
& = m^2+2mn+n^2 \\
\end{aligned}
$$
```
$$
\begin{aligned}
f(x) & = (m+n)^2 \\
& = m^2+2mn+n^2 \\
\end{aligned}
$$

### Equations

- `\case` is used to indicate left-alignment.

``` latex
$$
\begin{cases}
x+2y3z=0 \\
4x-5y+6z=0 \\
-7x+8y+9z=0 \\
\end{cases}
$$
```
$$
\begin{cases}
x+2y3z=0 \\
4x-5y+6z=0 \\
-7x+8y+9z=0 \\
\end{cases}
$$

- Aligned  according to the equal sign is indicated by `\aligned`.

``` latex
$$
\begin{cases}
x+2y3z=0 \\
4x-5y+6z=0 \\
-7x+8y+9z=0 \\
\end{cases}
$$
```
$$
\left\{ \begin{aligned}
x+2y3z=0 \\
4x-5y+6z=0 \\
-7x+8y+9z=0 \\
\end{aligned} \right.
$$

## Arrays and tables

### Basic

- Arrays and tables begin with `begin{array}{definite}` and end with `end{array}`. Define the alignment of each column in the definition, and `l`, `c` and `r` can be used to represent left, center, and right, respectively. If you insert a horizontal dividing line, insert `\hline` between the line contents; If you insert a vertical dividing line, insert `|`. The table content is separated by `&` columns and rows by `\\`.

``` latex
$$
\begin{array}{c|lcr}
x & 1\text{(left)} & 2\text{(center)} & 3\text{(right)} \\
\hline
f(x) & 0 & 1 & 2 \\
g(x) & 3 & 4 & 5 \\
h(x) & 6+i & 7-i & 1+10i
\end{array}
$$
```
$$
\begin{array}{c|lcr}
x & 1\text{(left)} & 2\text{(center)} & 3\text{(right)} \\
\hline
f(x) & 0 & 1 & 2 \\
g(x) & 3 & 4 & 5 \\
h(x) & 6+i & 7-i & 1+10i
\end{array}
$$

- You can use arrays and tables to implement similar functions as aligned.

``` latex
$$
\begin{array}{lcr}
z & = & a \\
f(x,y,z) & = & x+y+z
\end{array}
$$
```
$$
\begin{array}{lcr}
z & = & a \\
f(x,y,z) & = & x+y+z
\end{array}
$$

### Nesting of arrays and tables

``` latex
$$
% outer
\begin{array}{c}
	% inner row 1
	\begin{array}{cc}
		% inner row 1 column 1
		\begin{array}{c|cccc}
		\min & 0 & 1 & 2 & 3 \\
		\hline
		0 & 0 & 0 & 0 & 0 \\
		1 & 0 & 1 & 1 & 1 \\
		2 & 0 & 1 & 2 & 2 \\
		3 & 0 & 1 & 2 & 3
		\end{array}
	&
		% inner row 1 column 2
		\begin{array}{c|cccc}
		\max & 0 & 1 & 2 & 3 \\
		\hline
		0 & 0 & 1 & 2 & 3 \\
		1 & 1 & 1 & 2 & 3 \\
		2 & 2 & 2 & 2 & 3 \\
		3 & 3 & 3 & 3 & 3
		\end{array}
	\end{array}
\\
	% inner row 2
	\begin{array}{c|cccc}
	\Delta & 0 & 1 & 2 & 3 \\
	\hline
	0 & 0 & 1 & 2 & 3 \\
	1 & 1 & 0 & 1 & 2 \\
	2 & 2 & 1 & 0 & 1 \\
	3 & 3 & 2 & 1 & 0
	\end{array}
\end{array}
$$
```
$$
% outer
\begin{array}{c}
	% inner row 1
	\begin{array}{cc}
		% inner row 1 column 1
		\begin{array}{c|cccc}
		\min & 0 & 1 & 2 & 3 \\
		\hline
		0 & 0 & 0 & 0 & 0 \\
		1 & 0 & 1 & 1 & 1 \\
		2 & 0 & 1 & 2 & 2 \\
		3 & 0 & 1 & 2 & 3
		\end{array}
	&
		% inner row 1 column 2
		\begin{array}{c|cccc}
		\max & 0 & 1 & 2 & 3 \\
		\hline
		0 & 0 & 1 & 2 & 3 \\
		1 & 1 & 1 & 2 & 3 \\
		2 & 2 & 2 & 2 & 3 \\
		3 & 3 & 3 & 3 & 3
		\end{array}
	\end{array}
\\
	% inner row 2
	\begin{array}{c|cccc}
	\Delta & 0 & 1 & 2 & 3 \\
	\hline
	0 & 0 & 1 & 2 & 3 \\
	1 & 1 & 0 & 1 & 2 \\
	2 & 2 & 1 & 0 & 1 \\
	3 & 3 & 2 & 1 & 0
	\end{array}
\end{array}
$$

## Fonts

- Use Large and Small to control font size

`$ A, \large{A}, \small{A} $`
$ A, \large{A}, \small{A} $

### Greek symbols

`$\digamma, \varepsilon, \vartheta, \varkappa, \varpi, \varrho, \varsigma, \varphi $
`
$\digamma, \varepsilon, \vartheta, \varkappa, \varpi, \varrho, \varsigma, \varphi $

$$
\begin{array}{c|l|c|l}
A \alpha & \verb|A \alpha| & N \nu & \verb|N \nu| \\
\hline
B \beta & \verb|B \beta| & \Xi \xi & \verb|\Xi \xi| \\
\hline
\Gamma \gamma & \verb|\Gamma \gamma| & O \omicron & \verb|O \omicron| \\
\hline
\Delta \delta & \verb|\Delta \delta| & \Pi \pi & \verb|\Pi \pi| \\
\hline
E \epsilon & \verb|E \epsilon| & P \rho & \verb|P \rho| \\
\hline
Z \zeta & \verb|Z \zeta| & \Sigma \sigma & \verb|\Sigma \sigma| \\
\hline
H \eta & \verb|H \eta| & T \tau & \verb|T \tau| \\
\hline
\Theta \theta & \verb|\Theta \theta| & \Upsilon \upsilon & \verb|\Upsilon \upsilon| \\
\hline
I \iota & \verb|I \iota| & \Phi \phi & \verb|\Phi \phi| \\
\hline
K \kappa & \verb|K \kappa| & X \chi & \verb|X \chi| \\
\hline
\Lambda \lambda & \verb|\Lambda \lambda| & \Psi \psi & \verb|\Psi \psi| \\
\hline
M \mu & \verb|M \mu| & \Omega \omega & \verb|\Omega \omega| \\
\end{array}
$$

### Hebrew symbols

`$\aleph, \beth, \gimel, \daleth$`
$\aleph, \beth, \gimel, \daleth$

### Special font form

#### Bold

- Use `\mathbf{text}` or `{\bf text}`, `\mathbb{text}` or `\Bbb{text}`. However, it is not valid for special symbols.

``` latex
$$
\begin{array}{c|ccc}
\text{defalut} & SAMPLE & sample & 0123 \\
\hline
\text{mathbf} & \mathbf{SAMPLE} & \mathbf{sample} & \mathbf{0123} \\
\hline
\text{bf} & {\bf SAMPLE} & {\bf sample} & {\bf 0123} \\
\hline
\text{mathbb} & \mathbb{SAMPLE} & \mathbb{sample} & \mathbb{0123} \\
\hline
\text{Bbb} & \Bbb{SAMPLE} & \Bbb{sample} & \Bbb{0123}
\end{array}
$$ 
```
$$
\begin{array}{c|ccc}
\text{defalut} & SAMPLE & sample & 0123 \\
\hline
\text{mathbf} & \mathbf{SAMPLE} & \mathbf{sample} & \mathbf{0123} \\
\hline
\text{bf} & {\bf SAMPLE} & {\bf sample} & {\bf 0123} \\
\hline
\text{mathbb} & \mathbb{SAMPLE} & \mathbb{sample} & \mathbb{0123} \\
\hline
\text{Bbb} & \Bbb{SAMPLE} & \Bbb{sample} & \Bbb{0123}
\end{array}
$$

#### Bold symbols

- Use `\boldsymbol{text}`, which is valid for special symbols.

``` latex
$$
\begin{array}{c|cccccc}
\text{defalut} & SAMPLE & sample & 0123 & \delta\theta\sigma\omega & \Delta\Theta\Sigma\Omega & \sin\in\to \\
\hline
\text{blodsymbol} & \boldsymbol{SAMPLE} & \boldsymbol{sample} & \boldsymbol{0123} & \boldsymbol{\delta\theta\sigma\omega} & \boldsymbol{\Delta\Theta\Sigma\Omega} & \boldsymbol{\sin\in\to} \\
\end{array}
$$
```
$$
\begin{array}{c|cccccc}
\text{defalut} & SAMPLE & sample & 0123 & \delta\theta\sigma\omega & \Delta\Theta\Sigma\Omega & \sin\in\to \\
\hline
\text{blodsymbol} & \boldsymbol{SAMPLE} & \boldsymbol{sample} & \boldsymbol{0123} & \boldsymbol{\delta\theta\sigma\omega} & \boldsymbol{\Delta\Theta\Sigma\Omega} & \boldsymbol{\sin\in\to} \\
\end{array}
$$

#### Italian font

- Use `\mathit{text}` or `{\it text}`

``` latex
$$
\begin{array}{c|ccccc}
\text{defalut} & SAMPLE & sample & 0123 & \delta\theta\sigma\omega & \Delta\Theta\Sigma\Omega \\
\hline
\text{mathit} & \mathit{SAMPLE} & \mathit{sample} & \mathit{0123} & \mathit{\delta\theta\sigma\omega} & \mathit{\Delta\Theta\Sigma\Omega} \\
\hline
\text{it} & {\it SAMPLE} & {\it sample} & {\it 0123} & {\it \delta\theta\sigma\omega} & {\it \Delta\Theta\Sigma\Omega} \\
\hline
\end{array}
$$
```
$$
\begin{array}{c|ccccc}
\text{defalut} & SAMPLE & sample & 0123 & \delta\theta\sigma\omega & \Delta\Theta\Sigma\Omega \\
\hline
\text{mathit} & \mathit{SAMPLE} & \mathit{sample} & \mathit{0123} & \mathit{\delta\theta\sigma\omega} & \mathit{\Delta\Theta\Sigma\Omega} \\
\hline
\text{it} & {\it SAMPLE} & {\it sample} & {\it 0123} & {\it \delta\theta\sigma\omega} & {\it \Delta\Theta\Sigma\Omega} \\
\hline
\end{array}
$$

#### Roman font

- Use `\mathrm{text}` or `{\rm text}`

$$
\begin{array}{c|ccccc}
\text{defalut} & SAMPLE & sample & 0123 & \delta\theta\sigma\omega & \Delta\Theta\Sigma\Omega \\
\hline
\text{mathrm} & \mathrm{SAMPLE} & \mathrm{sample} & \mathrm{0123} & \mathrm{\delta\theta\sigma\omega} & \mathrm{\Delta\Theta\Sigma\Omega} \\
\hline
\text{rm} & {\rm SAMPLE} & {\rm sample} & {\rm 0123} & {\rm \delta\theta\sigma\omega} & {\rm \Delta\Theta\Sigma\Omega} \\
\end{array}
$$

> Tips: Lowercase Greek letters do not support Roman script; For Arabic numerals and uppercase Greek letters, the default font is `\rm`.

#### Others

$$
\begin{array}{c|ccccc}
\text{defalut} & SAMPLE & sample & 0123 & \delta\theta\sigma\omega & \Delta\Theta\Sigma\Omega \\
\hline
\text{mathsf} & \mathsf{SAMPLE} & \mathsf{sample} & \mathsf{0123} & \mathsf{\delta\theta\sigma\omega} & \mathsf{\Delta\Theta\Sigma\Omega} \\
\hline
\text{sf} & {\sf SAMPLE} & {\sf sample} & {\sf 0123} & {\sf \delta\theta\sigma\omega} & {\sf \Delta\Theta\Sigma\Omega} \\
\hline
\text{mathscr} & \mathscr{SAMPLE} & \mathscr{sample} & \mathscr{0123} & \mathscr{\delta\theta\sigma\omega} & \mathscr{\Delta\Theta\Sigma\Omega} \\
\hline
\text{mathcal} & \mathcal{SAMPLE} & \mathcal{sample} & \mathcal{0123} & \mathcal{\delta\theta\sigma\omega} & \mathcal{\Delta\Theta\Sigma\Omega} \\
\hline
\text{cal} & {\cal SAMPLE} & {\cal sample} & {\cal 0123} & {\cal \delta\theta\sigma\omega} & {\cal \Delta\Theta\Sigma\Omega} \\
\hline
\text{mathtt} & \mathtt{SAMPLE} & \mathtt{sample} & \mathtt{0123} & \mathtt{\delta\theta\sigma\omega} & \mathtt{\Delta\Theta\Sigma\Omega} \\
\hline
\text{tt} & {\tt SAMPLE} & {\tt sample} & {\tt 0123} & {\tt \delta\theta\sigma\omega} & {\tt \Delta\Theta\Sigma\Omega} \\
\hline
\text{mathfrak} & \mathfrak{SAMPLE} & \mathfrak{sample} & \mathfrak{0123} & \mathfrak{ \delta\theta\sigma\omega} & \mathfrak{\Delta\Theta\Sigma\Omega} \\
\hline
\text{frak} & {\frak SAMPLE} & {\frak sample} & {\frak 0123} & \frak{ \delta\theta\sigma\omega} & \frak{\Delta\Theta\Sigma\Omega} \\
\hline
\text{scriptstyle} & {\scriptstyle SAMPLE} & {\scriptstyle sample} & {\scriptstyle 0123} & \scriptstyle{ \delta\theta\sigma\omega} & \scriptstyle{\Delta\Theta\Sigma\Omega}  \\
\hline
\text{scriptstyle+text} & {\scriptstyle \text{SAMPLE} } & {\scriptstyle \text{sample} } & {\scriptstyle \text{0123} } \\
\end{array}
$$

#### Mix fonts

``` latex
$$
f(n)=
\begin{cases}
n/2, & \text{if $n$ is even} \\
3n+1, & \text{if $n$ is odd}
\end{cases}
$$
```
$$
f(n)=
\begin{cases}
n/2, & \text{if $n$ is even} \\
3n+1, & \text{if $n$ is odd}
\end{cases}
$$

- Attention block

``` latex
$$
\begin{matrix}
\text{if} n \text{is even} \\ % Bad
\text{if } n \text{ is even} \\ % Good
\text{if}~n~\text{is even} \\ % Good
\text{if}\ n\ \text{is even} \\ % Good
\end{matrix}
$$
```
$$
\begin{matrix}
\text{if} n \text{is even} \\ % Bad
\text{if } n \text{ is even} \\ % Good
\text{if}~n~\text{is even} \\ % Good
\text{if}\ n\ \text{is even} \\ % Good
\end{matrix}
$$

## Space

> $Latex$ automatically ignores spaces, and the space control from wide to narrow is:

- Double quad space: 2 characters wide

`$ \alpha \qquad \beta $`
$ \alpha \qquad \beta $

- Quad space: 1 character wide

`$ \alpha \quad \beta $`
$ \alpha \quad \beta $
​
- Space: 1/3 character wide

`$ \alpha \ \beta ~ \gamma $`
$ \alpha \ \beta ~ \gamma $

- Medium space: 2/7 characters wide

`$ \alpha \; \beta $`
$ \alpha \; \beta $
​
- Small space: 1/6 character width

`$ \alpha \, \beta $`
$ \alpha \, \beta $
​
- No spaces: 0 characters wide

`$ \alpha \beta $`
$ \alpha \beta $
​
- Snap: -1/6 character width

`$ \alpha \! \beta $`
$ \alpha \! \beta $
​

## Color

- Use `{\color{color}{text} }` to control color of text.

`$ \color{red}{text} \quad \color{yellow}{text} \quad \color{blue}{text} \quad \color{green}{text} \quad \color{purple}{text} $`
$ \color{red}{text} \quad \color{yellow}{text} \quad \color{blue}{text} \quad \color{green}{text} \quad \color{purple}{text} $
​

- You can also use capital initials to represent complex colors.

`$ \color{Red}{text} \quad \color{Orange}{text} \quad \color{RoyalBlue}{text} \quad \color{Violet}{text} \quad \color{LimeGreen}{text} $`
$ \color{Red}{text} \quad \color{Orange}{text} \quad \color{RoyalBlue}{text} \quad \color{Violet}{text} \quad \color{LimeGreen}{text} $

- Use `{\color{ #rgb}{text} }` to control more complex colors. However, this is not possible in some renderings, so it is not shown here.

`$ \color{ #0FF}{text} \quad \color{ #00F}{text} \quad \color{ #F0F}{text} \quad \color{ #0F0}{text} \quad \color{ #6CF}{text} $`

## Tips

- Use the `\boxed` command or with the border of a 1*1 table

`$ \boxed {E=mc^2} $`
$ \boxed {E=mc^2} $
``` latex
$$
\begin{array}{|c|}
\hline
E=mc^2 \\
\hline
\end{array}
$$
```
$$
\begin{array}{|c|}
\hline
E=mc^2 \\
\hline
\end{array}
$$

- Some useful $ Latex $ websites.

1. [Tables Generator](https://www.tablesgenerator.com)
2. [Classify Symbols](http://detexify.kirelabs.org/classify.html)
3. [Equation Editor](http://www.codecogs.com/latex/eqneditor.php)
---
title: Skills for TexLive under VsCode
date: 2023-07-12
tags: 
    - 总结
categories: Latex
copyright: true
---

> Record some $ LaTeX $ packages and installation and usage skills.

<!--more-->

## :books: 1. Packages

### 1.1. Packages for Code: `minted`

- import package `minted` for code highlighting.
```latex
\usepackage{minted}
```

- Before using `minted`, you should have `python version>2.7` and install `pygments` first.
```bash
python -V

pip install pygments
```


- It may be show error message
```bash
You must invoke LaTeX with the -shell-escape flag`. 
```
- The reason is that the `minted` package needs to call the `pygments` package to highlight the code, and the `pygments` package needs to call the system command. The `-shell-escape` parameter is used to allow the `minted` package to call the system command.
- It's necessary to add `-shell-escape` parameter to the setting file, the following json file is a part of it.
```json
"latex-workshop.latex.tools": [
        {
            "name": "xelatex",
            "command": "xelatex",
            "args": [
                "--shell-escape",
                "-synctex=1",
                "-interaction=nonstopmode",
                "-file-line-error",
                "%DOCFILE%"
            ]
        },
        {
            "name": "pdflatex",
            "command": "pdflatex",
            "args": [
                "--shell-escape",
                "-synctex=1",
                "-interaction=nonstopmode",
                "-file-line-error",
                "%DOCFILE%"
            ]
        },
        {
            "name": "latexmk",
            "command": "latexmk",
            "args": [
                "--shell-escape",
                "-synctex=1",
                "-interaction=nonstopmode",
                "-file-line-error",
                "-pdf",
                "-outdir=%OUTDIR%",
                "%DOCFILE%"
            ]
        },
        {
            "name": "bibtex",
            "command": "bibtex",
            "args": [
                "%DOCFILE%"
            ]
        }
    ],
```

- Then it may also show error message
```bash
You must have `pygmentize' installed to use this package.
```
- This is because you installed `pygments` failed or the `pygments` package is not in the system path. In windows 10 or later versions, there is a inside python version in `Disk C`.
- You will find `pygmentize.exe` in path: `C:\Users\username\AppData\Local\Programs\Python\Python37\Scripts`, add the path to the system path, then it will work.

- Example for `minted` package
```latex
\begin{minted}[linenos=true,frame=lines,framesep=2mm]{python}
def quick_sort(arr):
    if len(arr) < 2:
        return arr
    else:
        pivot = arr[0]
        less = [i for i in arr[1:] if i <= pivot]
        greater = [i for i in arr[1:] if i > pivot]
        return quick_sort(less) + [pivot] + quick_sort(greater)
\end{minted}
```
![minted_file][1]

<!-- markdownlint-disable-file MD025 MD028 MD033 -->
[1]: https://www.lingzhicheng.cn/usr/file/picture/tex/minted.jpg
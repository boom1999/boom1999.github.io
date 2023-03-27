---
title: Installation for TexLive under VsCode
date: 2023-03-27
tags: 
    - 总结
categories: Latex
---

> Okey, let's go! I used to use `VsCode` + `TexLive`, which is more convenient, because `VsCode` has a built-in `LaTeX Workshop` extension, which can be used directly after installation. 

<!--more-->

## :books: 1.Forwards

- $ LaTeX $ is widely used in **academia** for the communication and publication of scientific documents in many fields, including **mathematics, computer science, engineering, physics, chemistry, economics, linguistics, quantitative psychology, philosophy, and political science**. It also has a prominent role in the preparation and publication of books and articles that contain complex multilingual materials, such as Arabic and Greek. LaTeX uses the TeX typesetting program for formatting its output, and is itself written in the **$TeX$ macro language**.

- $ LaTeX $ can be used as **a standalone document preparation system**, or as **an intermediate format**. In the latter role, for example, it is sometimes used as part of a pipeline for translating `DocBook` and `other XML-based formats` to `PDF`. The typesetting system offers programmable desktop publishing features and extensive facilities for automating most aspects of typesetting and desktop publishing, including **numbering and cross-referencing of tables and figures, chapter and section headings, graphics, page layout, indexing and bibliographies**.

- Without further ado, let’s go directly to the picture! 
![Tex example][1]


## :pushpin: 2.VsCode installation and language configuration

> By default, everyone can install vscode and be familiar with the basic operations, so I won't repeat it here. If you don't know how to install it, you can refer to the [ official website][2], only list a few points here.

1. Pay attention to modifying the **installation path**, because more **plug-ins** may be installed later (not only $LaTeX$, but also other scenarios).
2. For convenience, it is recommended to create a desktop shortcut, and add the "Open with Code" action to the Windows Explorer file context menu, and add it to the `PATH`.
3. 如果你需要改为中文环境:cn:，可以安装中文插件，可以在左边栏的拓展中搜索[Chinese (Simplified) Language Pack for Visual Studio Code][3].


## :file_folder: 3.TexLive Download and Installation

> `TexLive` is used here, if you want to use `MikTex` or others, you can refer to other articles.

- Install TexLive via [ISO image][4], you can choose download from a nearby CTAN mirror or manually choose CTAN mirror.
![ISO image][5]

- :cd: You can choose a **suitable mirror** to download according to your network status. If the download speed is too *slow*, you can try to choose a mirror station that is closer to download. It may take about ten minutes.
  - This directory contains the ISO image for the official TeX Live release; md5 and sha512 checksums are provided, and the sha checksum is GPG-signed. The generic names (texliveYYYY.iso and texlive.iso) are symlinks to the dated release .iso.
  - If you have problems with installation or running TeX after installation, please check your environment variables: settings, including PATH, that end up referencing previously-installed TeX systems (TeX Live or otherwise), can cause trouble, especially on Windows.

- :clipboard: Wait for the download to complete, open the ISO file, and open the batch file `install-tl-windows.bat` as an administrator.
  - If you are using Windows 10 or 11, you can right-click the ISO file and select "Mount" to mount the ISO file as a virtual disk. Then you can open the batch file as an administrator.
  - If you are using Windows 7, you can use the [7-Zip](https://www.7-zip.org/) software to open the ISO file, and then open the batch file as an administrator.
![Open ISO][6]

- Then it *freezes* for a while, which is normal, and the GUI for installation appears.
<div align=center>
  <img src="https://www.lingzhicheng.cn/usr/file/picture/tex/installer_gui.jpg" alt="installer gui" width="30%" height="30%" text-align:center>
</div>

- :wrench: Modify the installation path, usually `Disk:/texlive/year`, such as`D:/texlive/2023`, and then click `Install`. It may take a while, depending on your computer performance, so take a break with a cup of coffee.
- There is an Advanced option. Generally speaking, after changing `TEXDIR`, the `TEXMFLOCAL` path will also be modified. If you are not familiar with other options, don't to modify them (especially for novices).

- :white_check_mark: After the installation is complete, click `Finish` to exit the installation interface.
<div align=center>
  <img src="https://www.lingzhicheng.cn/usr/file/picture/tex/close_gui.jpg" alt="close gui" width="50%" height="50%" text-align:center>
</div>

- Check if the installation is successful. Use `xelatex -v` in *CMD*.
![Installation successfully][7]

## :hammer: 4. Extension installation and environment settings

### :nut_and_bolt: 4.1 Install `LaTex Workshop` extension in Vscode.

- Open the Extensions sidebar in VS Code. `View → Extensions` or `Ctrl+Shift+X`.
- Search for `LaTeX Workshop`.
- Click Install. It will be installed into the `~/.vscode/extensions` directory.
- Reload VS Code after installation.
![Install LaTex Workshop][8]

- If your `Tex Live` version is relatively old, additional manual configuration of environment variables may be required.

### :bookmark_tabs: 4.2 Environment settings

- There are two environment configuration schemes in Vscode, one is to use buttons and option settings directly, and the other is to use `json` to set.
![workshop_setting][9]
- Click `Open settings (JSON)` in the upper right corner to use the code settings.*[JSON](https://www.json.org/json-en.html) is a lightweight text data format with a certain structure. If you are not familiar with it, you can spend a few minutes to understand it.*
- **Add** the following settings after other settings.

``` json
{
    "latex-workshop.latex.autoBuild.run": "never",
    "latex-workshop.showContextMenu": true,
    "latex-workshop.intellisense.package.enabled": true,
    "latex-workshop.message.error.show": false,
    "latex-workshop.message.warning.show": false,
    "latex-workshop.latex.tools": [
        {
            "name": "xelatex",
            "command": "xelatex",
            "args": [
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
    "latex-workshop.latex.recipes": [
        {
            "name": "XeLaTeX",
            "tools": [
                "xelatex"
            ]
        },
        {
            "name": "PDFLaTeX",
            "tools": [
                "pdflatex"
            ]
        },
        {
            "name": "BibTeX",
            "tools": [
                "bibtex"
            ]
        },
        {
            "name": "LaTeXmk",
            "tools": [
                "latexmk"
            ]
        },
        {
            "name": "xelatex -> bibtex -> xelatex*2",
            "tools": [
                "xelatex",
                "bibtex",
                "xelatex",
                "xelatex"
            ]
        },
        {
            "name": "pdflatex -> bibtex -> pdflatex*2",
            "tools": [
                "pdflatex",
                "bibtex",
                "pdflatex",
                "pdflatex"
            ]
        },
    ],
    "latex-workshop.latex.clean.fileTypes": [
        "*.aux",
        "*.bbl",
        "*.blg",
        "*.idx",
        "*.ind",
        "*.lof",
        "*.lot",
        "*.out",
        "*.toc",
        "*.acn",
        "*.acr",
        "*.alg",
        "*.glg",
        "*.glo",
        "*.gls",
        "*.ist",
        "*.fls",
        "*.log",
        "*.fdb_latexmk"
    ],
    "latex-workshop.latex.autoClean.run": "onFailed",
    "latex-workshop.latex.recipe.default": "lastUsed",
    "latex-workshop.view.pdf.internal.synctex.keybinding": "double-click"
}
```

### :black_nib: 4.3 Detailed introduction of environment configuration parameters

> If you don't want to read it, you can just skip it, it's not that important.

1. Set when to automatically build `LaTeX` using the default (first) compilation chain.
   1. `onSave` builds the project upon saving a tex file in vscode.
   2. `onFileChange` builds the project upon detecting a file change in any of the dependencies, even modified by other applications.

``` json
"latex-workshop.latex.autoBuild.run": "never"
```

2. Enable contextual LaTeX menu: Right click will have **two more options**, more convenient.
   1. Default: `false`
   2. `true` enables the menu.
   3. `Build LaTex project.` *(Ctrl+Alt+B)*
   4. `SyncTex from cursor.` *(Ctrl+Alt+J)*
``` json
"latex-workshop.showContextMenu": true
```

3. `"latex-workshop.intellisense.package.enabled": true` enables the package name completion in the `\usepackage{}` command.
4. `"latex-workshop.message.error.show": true` shows the error message in the status bar.
5. `"latex-workshop.message.warning.show": true` shows the warning message in the status bar.
6. `"latex-workshop.latex.tools":[...]` defines default settings for the compilation.
7. `"latex-workshop.latex.recipes": [...]` defines default settings and compilation order for the recipe compilation chains:
   1. Recipe: XeLaTex
   2. Recipe: PDFLaTex
   3. Recipe: BibTeX
   4. Recipe: LaTeXmk
   5. Recipe: xelatex -> bibtex -> xelatex*2
   6. Recipe: pdflatex -> bibtex -> pdflatex*2
8. Difference between XeLaTex and PDFLaTex:
   1. **PDFLaTeX uses TeX standard fonts**, so when generating PDF, all non-TeX standard fonts will be replaced, and the generated PDF files will embed all fonts by default; and compiled with XeLaTeX, if there are many pictures or other elements in the paper If no fonts are embedded, the generated PDF.
   2. **The XeTeX corresponding to XeLaTeX has better support for fonts**, allows users to use operating system fonts instead of TeX's standard fonts, and has better support for non-Latin fonts.
   3. **PDFLaTeX compiles faster** than XeLaTeX.
9. `"latex-workshop.latex.clean.fileTypes"` It is to clear some unnecessary files in the result generated during the compilation process.
10. `"latex-workshop.latex.autoClean.run"`
    1. `onFailed` cleans the project when compilation fails.
    2. `onBuilt` cleans the project when compilation is done, whether successful or failed.
11. `"latex-workshop.latex.recipe.default": "lastUsed"` sets the default compilation chain to the last used one. Get used to using the compiled chain you used last time, or you can change it to first to use the first compiled chain.
12. `"latex-workshop.view.pdf.internal.synctex.keybinding": "double-click"` sets the default key binding for the SyncTex function. The default is double-click, and you can also set it to `ctrl+alt+j` or `ctrl+alt+click`.

## :notebook: 5. SumatraPDF

> Vscode's built-in PDF reading cannot fully satisfy large-scale use, and an external reader must be used if necessary. It is recommended to use the combination of `SumatraPDF` and $Tex$, mainly because it can be synchronized in two directions, and it is also relatively lightweight.

- [Official download link][10]

- Add settings in `settings.json`. *Pay attention to modify the three paths below.*
``` json
{
    "latex-workshop.view.pdf.viewer": "external",
    "latex-workshop.view.pdf.ref.viewer":"auto",
    "latex-workshop.view.pdf.external.viewer.command": "D:/SumatraPDF/SumatraPDF.exe",
    "latex-workshop.view.pdf.external.viewer.args": [
        "%PDF%"
    ],
    "latex-workshop.view.pdf.external.synctex.command": "D:/SumatraPDF/SumatraPDF.exe",
    "latex-workshop.view.pdf.external.synctex.args": [
        "-forward-search",
        "%TEX%",
        "%LINE%",
        "-reuse-instance",
        "-inverse-search",
        "code \"D:/VSCode/resources/app/out/cli.js\" -r -g \"%f:%l\"",
        "%PDF%"
    ]
}
```

- `"latex-workshop.view.pdf.viewer": "external"` If you need to change the Tex PDF back to the internal reader, just change `external` to `tab`.

<!-- markdownlint-disable-file MD025 MD028 MD033 -->
[1]: https://www.lingzhicheng.cn/usr/file/picture/tex/tex_example.jpg
[2]: https://code.visualstudio.com/
[3]: https://code.visualstudio.com/docs/getstarted/locales
[4]: https://tug.org/texlive/acquire-iso.html
[5]: https://www.lingzhicheng.cn/usr/file/picture/tex/ISO_image.jpg
[6]: https://www.lingzhicheng.cn/usr/file/picture/tex/open_ISO.jpg
[7]: https://www.lingzhicheng.cn/usr/file/picture/tex/installation_success.jpg
[8]: https://www.lingzhicheng.cn/usr/file/picture/tex/install_latex_workshop.jpg
[9]: https://www.lingzhicheng.cn/usr/file/picture/tex/workshop_setting_1.jpg
[10]: https://www.sumatrapdfreader.org/download-free-pdf-viewer
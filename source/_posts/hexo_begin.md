---
title: Hexo_begin
date: 2021-01-31
tags: 
    - hexo
categories: blog
---

## 机制介绍 ##

`hexo d`上传部署到Github的其实是hexo编译后的文件静态文件，用于生成网页，不包含源文件，也就是上传`.deploy_git`中的文件，其他文件，包括`source`里面的和一些配置、主题文件都不会上传到GitHub，考虑到可能会在不同主机发布，因此将`hexo d`操作不会上传的文件主动`git`到另一个分支。
<!--more-->

----------

## 新建备份分支 ##

1. 首先在Github上新建一个`hexo_backup`分支并且设置为默认分支，以便在每次同步的时候不用指定分支，比较方便，用于备份源文件。

2. 在本地`blog_local`目录下`git clone git@github.com:boom1999/boom1999.github.io.git`,此时由于默认分支为hexo_backup，所以只是clone了hexo_backup。在clone下的目录里将`.git`以外的文件全部删除，将源文件（包括`.gitignore`）全部复制过来。（`.deploy_git`不需要，这是生成的静态文件，也就是会push到master分支中的文件）。如果没有`.gitignore`，忽略push这部分文件，请手动添加：

        .DS_Store
        Thumbs.db
        db.json
        *.log
        node_modules/
        public/
        .deploy*/

3. 将源文件push到hexo_backup分支,上传后GitHub上的hexo_backup分支备份完成

        git add .
        git commit –m "your commit"
        git push 

----------

## 更换环境操作 ##

- 创建一个新的`blog_local`文件夹
- 安装`git`和`nodejs`
- 设置git全局邮箱和用户名

        git config --global user.name "yourgithubname"
        git config --global user.email "yourgithubemail"
- 设置`ssh key`

        ssh-keygen -t rsa -C "youremail"
        #生成后公钥添加到GitHub仓库中
        #验证是否成功
        ssh -T git@github.com
- 安装hexo（不需要hexo init，因为整体框架会被clone下来）

        npm install hexo-cli -g
        #或者局部安装
        npm install hexo
- `clone`到本地操作

        git clone git@github.com:boom1999/boom1999.github.io.git
- 进入本地文件

        cd boom1999.github.io
        npm install
- 安装名为`hexo-deployer-git`的部署器插件，通过该插件就能实现一键部署

        npm install hexo-deployer-git --save
- 添加新文件`source/_posts`，生成，部署

        hexo g
        hexo d

### Tips ###

- 每次写完不要忘了备份源文件

        git add .
        git commit –m "your commit"
        git push 
- 如果是在已经目前使用的机器，如在其他机器上更新过，每次**使用前**只要远端同步一下就行`git pull`

----------

- `2021/01/30测试通过`
- 下一步准备部署到自己的服务器，GitHub作备份用

---
title: Git常用操作
date: 2021-03-25
tags: 
   - Git
   - Github
categories: 总结
copyright: true
---

:whale:总结Git日常基本操作，不定时添加一些遇到的问题，以及从小白入门连接本地git和github等
:bell:默认已有Github或Gitee等账号,已下载安装git

- ## [本地:computer:Git配置](#LocalGit) ##

    > ### [:pushpin:配置用户信息](#UserInfo) ###

    > ### [:pushpin:生成SSH Key](#SSH) ###

- ## [Upload repositories from :computer:to:cloud:](#UploadRepo) ##

- ## [Clone repositories from :cloud:to:computer:](#CloneRepo) ##

- ## [:blue_book:Git基本指令汇总](#GitBasic) ##

<!--more-->

---

<h2 id="LocalGit">一、本地Git配置</h2>

<h3 id="UserInfo">1.1配置用户信息</h3>

- Git bash here
    ` $ git config --global user.name "boom1999" `
    ` $ git config --global user.email "lingzhicheng@zjut.edu.cn" `
    ` $ git config --list `

![UserInfo][1]

<h3 id="SSH">1.2生成SSH Key</h3>

- Git bash here
    ` $ ssh-keygen -t rsa -C "lingzhicheng@zjut.edu.cn" `

![SSHKey][2]

- 添加SSH Key
    > 将提示位置(/c/Users/xxx/.ssh/id_rsa)的公钥（id_rsa.pub）内容复制粘贴至github > setting > SSH and GPG keys> key
    > 注意不要弄混公钥和私钥
    > 若只在某一个Repo中使用，可在此Repo中setting > Deploy Keys中添加公钥 

![DeployKeys][3]

<h2 id="UploadRepo">二、从本地上传Repositories</h2>

### :memo:本地创建Repo ###

- 新建目录/选择目录
- Git bash here 进入命令行操作
    > 新建文件
    ` $ mkdir 文件名 `
    > 进入文件夹
    ` $ cd 文件名 `
- 初始化git仓库
    ` $ git init `
- 查看文件内容检查是否配置成功
    ` $ ls -la `

```至此git仓库初始化结束，默认叫master分支```

---

- 新建测试（dev）分支
    ` $ git checkout -b dev `
- 检查git状态
    ` $ git status `

![GitInit][4]

- 使用编辑器在文件夹中编辑内容，测试和执行
- 提交文件至缓存区(用`git add .`提交目录下所有文件)
    ` $ git add index.md `

![GitAdd][5]

- 撤销缓存内容
    ` $ git rm --cached index.md `

![GitRm][6]

- 发布版本
    ` $ git commit -m 'This is a commit.' `
- 查看版本日志（--graph）
    ` $ git log `

![GitCommit][7]

- 将测试分支合并到主分支
    > 切换回主分支
    ` $ git checkout master `
    > 执行合并操作
    ` $ git merge dev `
    > 删除dev分支
    ` $ git branch -d dev `

![GitMerge][8]

### :computer:Repo上传 ###

- 在Github上新建项目，复制生成项目对应的SSH地址

![GitDemo][9]

```$ git remote add origin git@github.com:boom1999/gitdemo.git```

![GitDemoUpload][10]

- 第一次推送本地分支(+u将本地和远程分支关联)
    ` $ git push -u origin master `
- 以后推送本地分支
    ` $ git push origin master `
- 推送其他分支（如dev）
    ` $ git push origin dev `

![GitRemote][11]

> 若有提示👇，请参照上文配置SSH Keys

``` shell
ERROR: Permission to boom1999/gitdemo.git denied to deploy key
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```

<h2 id="CloneRepo">三、从远程clone Repositories到本地</h2>

- 这是第二种方式，即现在GitHub上创建项目，或从已有的项目clone到另一台机器。
- 在Github上新建项目，复制生成项目对应的SSH地址
` $ git clone git@github.com:boom1999/gitdemo.git `
` $ cd gitdemo `
` $ ls -la `

<h2 id="GitBasic">四、Git基本指令汇总</h2>

- 配置全局个人信息

``` shell
git config --global user.name "your user name"    
git config --global user.email "your email address"
```

- :dart:More

``` shell
git config --list       //显示所有配置列表
ls                      //显示目录下可见文件 
ls -la                  //显示当前目录所有文件包括隐藏文件
ls | grep *.js          //通过管道筛选所有js文件
git init                //初始化git仓库 
git add                 //添加该文件到暂存区 
git status              //查看当前状态
git rm --cached         //从暂存区移除到工作区 
git add .               //添加所有的文件到暂存区 
git commit . -m ""      //将缓存区的内容提交到历史区("-m"表示不跳到另一个页面)  
git commit .            //添加所有文件到历史区 
git checkout            //将工作区的修改撤销；取回暂存区的文件 
git status -s           //查看哪些文件被更改过 
git log                 //查看提交历史 
git diff                //查看工作区和暂存区文件的区别 
git diff --cached       //查看暂存区和历史区的区别 
git diff HEAD           //查看工作区与上一个版本的不同（更改文件后不提交到历史区可用该命令查看）  
git log --oneline       //提交显示到一行 
git log --graph         //图形化显示提交的 
git reset --hard HEAD^  //所有文件回到上一个版本 
git reflog              //查看所有的提交 
git reset --hard xxxxxx //回到指定版本(前5/6位版本号即可) 
git log --oneline --grep="project"      
                        //查找文件中带有project的文件并显示到一行 
git reset --mixed HEAD^ //暂存区和历史区回到上一个版本，工作区不变 
```

[1]: https://www.lingzhicheng.cn/usr/file/picture/Git/UserInfo.png
[2]: https://www.lingzhicheng.cn/usr/file/picture/Git/SSHKey.png
[3]: https://www.lingzhicheng.cn/usr/file/picture/Git/DeployKeys.png
[4]: https://www.lingzhicheng.cn/usr/file/picture/Git/GitInit.png
[5]: https://www.lingzhicheng.cn/usr/file/picture/Git/GitAdd.png
[6]: https://www.lingzhicheng.cn/usr/file/picture/Git/GitRm.png
[7]: https://www.lingzhicheng.cn/usr/file/picture/Git/GitCommit.png
[8]: https://www.lingzhicheng.cn/usr/file/picture/Git/GitMerge.png
[9]: https://www.lingzhicheng.cn/usr/file/picture/Git/GitDemo.png
[10]: https://www.lingzhicheng.cn/usr/file/picture/Git/GitDemoUpload.png
[11]: https://www.lingzhicheng.cn/usr/file/picture/Git/GitRemote.png

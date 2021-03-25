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


- ## [本地Git配置](#LocalGit) ##
    > ### [配置用户信息](#UserInfo) ###
    > ### [生成SSH Key](#SSH) ###
- ## [Upload repositories from :computer:to:cloud:](#UploadRepo) ##
- ## [Clone repositories from :cloud:to:computer:](#CloneRepo) ##
- ## [Git基本指令](#GitBasic) ##

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

<h2 id="CloneRepo">三、从远程clone Repositories到本地</h2>

<h2 id="GitBasic">四、Git基本指令</h2>

[1]: https://www.lingzhicheng.cn/usr/file/picture/Git/UserInfo.png
[2]: https://www.lingzhicheng.cn/usr/file/picture/Git/SSHKey.png
[3]: https://www.lingzhicheng.cn/usr/file/picture/Git/DeployKeys.png

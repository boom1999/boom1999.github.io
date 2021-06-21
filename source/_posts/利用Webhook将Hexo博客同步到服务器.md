---
title: Hexo同步部署再Github和服务器
date: 2021-02-01
tags: 
    - hexo
    - 同步
    - webhook
categories: blog
copyright: true
---


> 寄人篱下不是长久之计，下文展开介绍利用Webhook将Hexo博客部署到自己的服务器上，同时github-peges作为镜像站使用，接上一篇，默认分支hexo_backup作为源文件备份，master作主页

<!--more-->

- 前提条件：已有github.io仓库，hexo博客已经上线测试可以使用，已有`nginx`、`git`、`node`等基础，可参考上一篇笔记
- 省略了一些次要步骤，域名解析、备案等

----

## 服务器Webhook部分 ##

- 利用宝塔平台的Webhook插件

> 名称随意，实例脚本如下
> gitPath="/www/wwwroot/YourWebsiteLocation/"
> gitHttp="https://github.com/YourGithubName/xxxxxxxx.github.io.git"
> 若失败，可考虑加上`sudo`使用管理员权限

    #!/bin/bash
    echo ""
    #输出当前时间
    date --date='0 days ago' "+%Y-%m-%d %H:%M:%S"
    echo "Start"
    
    #git项目路径
    gitPath=" "
    #git 网址
    gitHttp=" "
    #延迟高可以加https://ghproxy.com/代理

    echo "Web站点路径：$gitPath"
     
    #判断项目路径是否存在
    if [ -d "$gitPath" ]; then
        cd $gitPath
        #判断是否存在git目录
        if [ ! -d ".git" ]; then
                echo "在该目录下克隆 git"
                sudo git clone $gitHttp gittemp
                sudo mv gittemp/.git .
                sudo rm -rf gittemp
        fi
        echo"拉取最新的项目文件"
        sudo git reset --hard origin/master
        sudo git pull origin master
        echo"设置目录权限"
        sudo chown -R www:www $gitPath
        date --date='0 days ago' "+%Y-%m-%d %H:%M:%S"
        echo "End"
        exit
    else
        echo "该项目路径不存在"
        echo "End"
        exit
    fi
![宝塔Webhook][1]

## Github设置Webhook ##

- 查看密钥
![宝塔Webhook查看密钥][2]

- 切换到Github项目的Settings-Webhooks

- Add Webhook

> **Payload URL:** `http://服务器ip:8888?hook?access_key=xxxxx`
> **Content type:** `application/json`
> **Secret:** `Key`
> **Which events would you like to trigger this webhook?**
> choose：Just the push event.
> √**Active**

- 测试，本地更改sources后`hexo g`+`hexo d`
- 第一次红了是因为将Github中Webhook的Content type设置成了`application/x-www-form-urlencoded`，改成`application/json`通过
![Webhook测试][3]

----

## Tips ##

1. 服务器需要开放8888端口，如果使用其他端口同样需要打开
2. 两者都是表单数据发送时的编码类型
    application/x-www-form-urlencoded --> key=value&key=value
3. 一般情况下都会正常同步，如果长时间未update，可能是DNS的问题，可以考虑选择一个连接速度较快的GitHub的ip添加到/etc/hosts并且刷新

[1]: https://www.lingzhicheng.cn/usr/file/picture/Hexo_synchronization/Webhook01.png
[2]: https://www.lingzhicheng.cn/usr/file/picture/Hexo_synchronization/Webhook02.png
[3]: https://www.lingzhicheng.cn/usr/file/picture/Hexo_synchronization/Webhook03.png

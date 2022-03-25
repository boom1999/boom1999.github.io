---
title: debug Gitalk 403
date: 2022-01-02
tags: 
   - debug
categories: Gitalk
copyright: true
---

- 留言系统使用Gitalk，最近发现无法使用GitHub账号登陆，出现403
- Error: Request failed with status code 403

### Headers

> 网络请求403，拿不到token，导致登陆不上
> Request URL：<https://cors-anywhere.herokuapp.com/https://github.com/login/oauth/access_token>
>
> ``` None
> Missing required request header. Must specify one of: origin,x-requested-with
> ```

<!--more-->

### cors-anywhere

> 开源框架解决跨域问题的反向代理
>
>``` None
> CORS Anywhere is a NodeJS proxy which adds CORS headers to the proxied request.
>
> The url to proxy is literally taken from the path, validated and proxied. The protocol part of the proxied URI is optional, and defaults to "http". If port 443 is specified, the protocol defaults to "https".
>
> This package does not put any restrictions on the http methods or headers, except for cookies. Requesting user credentials is disallowed. The app can be configured to require a header for proxying a request, for example to avoid a direct visit from the browser.
>```

### more

> 看上去像是被滥用，然后从2021.1.31后用户必须手动获得这个网站的访问权限后才能使用，这也就是为什么最近登陆不上了
>
> ``` None
> Public demo server (cors-anywhere.herokuapp.com) will be very limited by January 2021, 31st
> The demo server of CORS Anywhere (cors-anywhere.herokuapp.com) is meant to be a demo of this project. But abuse has become so common that the platform where the demo is hosted (Heroku) has asked me to shut down the server, despite efforts to counter the abuse (rate limits in #45 and #164, and blocking other forms of requests). Downtime becomes increasingly frequent (e.g. recently #300, #299, #295, #294, #287) due to abuse and its popularity.
> ```
>

### Gitalk Document

>``` None
>const gitalk = new Gitalk({
>  clientID: 'GitHub Application Client ID',
>  clientSecret: 'GitHub Application Client Secret',
>  repo: 'GitHub repo',      // The repository of store comments,
>  owner: 'GitHub repo owner',
>  admin: ['GitHub repo owner and collaborators, only these guys can initialize github issues'],
>  id: location.pathname,      // Ensure uniqueness and length less than 50
>  distractionFreeMode: false  // Facebook-like distraction free mode
>})
>
>gitalk.render('gitalk-container')
>```

### add

> 在gitalk相应js文件中添加
>
> ``` None
> proxy: '<%- theme.gitalk.proxy %>'
> ```
>
> 在配置文件中添加
>
> ``` None
> proxy: https://netnr-proxy.cloudno.de/https://github.com/login/oauth/access_token
> 
> ```
>
> 重新部署

暂时采用了Issue中找到的可替代的cors，表示感谢.

----------

### Update 2022.01.02

- 很久没更新blog了，趁这次有时间索性把积攒下来的问题一起解决:wave:
- 这次还是Gitalk的问题，~~虽然可以使用其他评论插件，~~ 但还是更加喜欢Gitalk的UI
- 上文已经解决了`403`的问题，是采用了一个Issue里的方法，使用了热心网友的代
- 但是发现实际上只用了三四个月左右就由于使用人数过多，所有访问都进了这个代理，后来应该是网友取消了，不得而知，出现了好几个月的`Error:Net error`
- 继续找问题会发现又有网友提出了上面这个方法（**好家伙搁这儿转圈圈呢**）
- 很机智的把`_config.yml`里的代理清了，发现有希望，不再是直接`error`了，有登录验证界面，但变成`405`了好家伙。
- 好在有个Taiwan的网友Kalin Lai[发现了问题][1]，是gitalk.ejs中的代理没删干净，手动狗头。
- 删干净后重新出现了`403`，继续找文档，loading···
- 最后终于找到了，忙活一个小时成功解决，吃饭去了

> 针对403的问题，Gitalk开发团队已经做了修复，解决方法如下：更新版本到 1.7.2 或者修改配置增加`proxy: https://cors-anywhere.azm.workers.dev/https://github.com/login/oauth/access_token`
> 当然也可以重新配置个人代理，有机会试一下，这里插个眼

### Update 2022.01.07

- 来更新了，试了用Cloudflare创建代理
- 有免费的版本，每天十万条，每十分钟一千条记录
- 但是发现代理后整个网站速度受到影响比较大，故放弃使用，选择用Gitalk团队的官方代理，结束。

[1]:https://github.com/gitalk/gitalk/issues/437

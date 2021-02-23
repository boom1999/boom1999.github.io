---
title: debug Gitalk 403
date: 2021-02-23
tags: 
   - debug
categories: Gitalk, plug-in
copyright: true
---

- 留言系统使用Gitalk，最近发现无法使用GitHub账号登陆，出现403
- Error: Request failed with status code 403

### Headers ###

> 网络请求403，拿不到token，导致登陆不上
> Request URL：<https://cors-anywhere.herokuapp.com/https://github.com/login/oauth/access_token>
>
> ``` None
> Missing required request header. Must specify one of: origin,x-requested-with
> ```

### cors-anywhere ###

> 开源框架解决跨域问题的反向代理
>
>``` None
> CORS Anywhere is a NodeJS proxy which adds CORS headers to the proxied request.
>
> The url to proxy is literally taken from the path, validated and proxied. The protocol part of the proxied URI is optional, and defaults to "http". If port 443 is specified, the protocol defaults to "https".
>
> This package does not put any restrictions on the http methods or headers, except for cookies. Requesting user credentials is disallowed. The app can be configured to require a header for proxying a request, for example to avoid a direct visit from the browser.
>```

### more ###

> 看上去像是被滥用，然后从2021.1.31后用户必须手动获得这个网站的访问权限后才能使用，这也就是为什么最近登陆不上了
>
> ``` None
> Public demo server (cors-anywhere.herokuapp.com) will be very limited by January 2021, 31st
> The demo server of CORS Anywhere (cors-anywhere.herokuapp.com) is meant to be a demo of this project. But abuse has become so common that the platform where the demo is hosted (Heroku) has asked me to shut down the server, despite efforts to counter the abuse (rate limits in #45 and #164, and blocking other forms of requests). Downtime becomes increasingly frequent (e.g. recently #300, #299, #295, #294, #287) due to abuse and its popularity.
> ```
>

### Gitalk Document ###

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

### add ###

> 在gitalk相应js文件中添加
>
> ``` None
> proxy: '<%- theme.gitalk.proxy %>'
> ``` None
> 在配置文件中添加
>
> ``` None
> proxy: https://netnr-proxy.cloudno.de/https://github.com/login/oauth/access_token
> 
> ```
>
> 重新部署

---

暂时采用了Issue中找到的可替代的cors，表示感谢.

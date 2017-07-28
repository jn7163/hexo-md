---
title: nginx自定义错误页
date: 2016-10-18 17:04:22
categories:
- linux
- 网络服务
- nginx
tags:
- linux
- 网络服务
- nginx
- 404错误页
keywords: nginx, 404
---
> 
如果碰巧网站出了问题，或者用户试图访问一个并不存在的页面时，此时服务器会返回代码为404的错误信息，此时对应页面就是404页面。404页面的默认内容和具体的服务器有关。如果后台用的是nginx服务器，那么404页面的内容则为：404 Not Found, 这样是非常影响用户体验的，我们可以自定义错误页

<!-- more -->

### 自定义错误页
<pre><code class="language-nginx line-numbers">server {
    error_page 404  /404.html;
    error_page 500 502 503 504 /50x.html;   # 500等错误同理
}

## 然后创建自定义错误页
</code></pre>

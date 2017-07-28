---
title: pip阿里云镜像加速
date: 2016-11-28 16:21:00
categories:
- python
tags:
- python
- pip镜像加速
keywords: pip, python, 阿里云, 加速
---
> 
由于伟大的墙，使用pip在线安装python模块的时候非常的慢
好在阿里云提供了国内pip仓库，还算有点良心

<!-- more -->

### 配置pip
<pre><code class="language-bash line-numbers"><script type="text/plain">mkdir .pip
vim .pip/pip.conf
[global]
index-url=http://mirrors.aliyun.com/pypi/simple/
[install]
trusted-host=mirrors.aliyun.com
[list]
format=columns
</script></code></pre>

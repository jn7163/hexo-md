---
title: hexo + nginx 博客搭建
date: 2016-10-11 15:50:57
categories:
- hexo
tags:
- hexo
- nginx
keywords: hexo, blog, next主题
---
> 
为什么要写blog？
写博客可以强化自己对知识点的理解，有些知识点只有自己真正写起来才知道，并不是当初想的那么简单；
我们平时学的东西都是很零散的，可能今天接触这个东西，过几天学另外的东西，学的快也就忘得快，时间久了之后，慢慢就会丢了；
好记性不如烂笔头，我们可以把接触的新东西记录到blog里，等哪天要用到的时候，不需要重新Google，直接看自己的blog就好了，因为来自你手笔的文章，总比看别人的文章理解的快！

<!-- more -->

### hexo篇
[hexo - markdown写作](https://hexo.io)

<pre><code class="language-bash line-numbers">## hexo安装
yum -y install git-core
wget -qO- https://raw.github.com/creationix/nvm/master/install.sh | sh  # 重新登陆shell
nvm install stable
npm install -g hexo-cli
hexo init www       # 新建www并初始化
cd www
npm install         # 安装依赖包

## hexo命令
hexo n "page_name"      # 新建页面
hexo cl                 # 清除public及缓存
hexo g                  # 生成静态页面
hexo d                  # 推送至github.io (1)
hexo s                  # 在本地启动hexo,监听127.0.0.1:4000
hexo s -p 80            # 指定端口号


(1) hexo github配置
github新建仓库 "user.github.io" 注意 user 是你的github用户名

然后配置 ~/www/_config.yml
deploy:
  type: git
  repo: git@github.com:user/user.github.io.git
  branch: master

安装git插件
npm install hexo-deployer-git --save

推送至github
hexo d

访问 https://user.github.io 就能看到你的blog了
</code></pre>

### nginx篇
> 
如果你手上有vps或者云主机的话，直接用nginx来部署blog会更好

<pre><code class="language-bash line-numbers"># centos 安装 nginx
yum -y install epel-release
yum -y install nginx

# 配置
1. cp -af ~/www/public/ /usr/share/nginx/html/www && chown -R nginx:nginx /usr/share/nginx/html/
### 复制之前先执行 hexo g 生成页面！

2. vim /etc/nginx/conf.d/www.conf
--- www.conf ---
server {
    listen 80;
    server_name www.example.com;
    root /usr/share/nginx/html/www;
    index index.html;
}
----------------

3. nginx -t # 检查nginx配置文件
systemctl start nginx
</code></pre>

### next主题
> 默认主题有点单调，建议更换为[next主题](http://theme-next.iissnan.com)

<pre><code class="language-bash line-numbers">cd ~/www/
git clone https://github.com/iissnan/hexo-theme-next themes/next
vim __config.yml
theme: next     # 启用next主题

hexo cl &amp;&amp; hexo g
hexo s -p 80        # 查看next主题是否生效
</code></pre>

[next官方文档](http://theme-next.iissnan.com/)

### hexo参考
<a href="http://lovenight.github.io/2015/11/10/Hexo-3-1-1-%E9%9D%99%E6%80%81%E5%8D%9A%E5%AE%A2%E6%90%AD%E5%BB%BA%E6%8C%87%E5%8D%97/" rel="nofollow">hexo3.1.1搭建指南</a>
<a href="http://hp256.com/2014/12/23/post-1/" rel="nofollow">markdown语法基础版</a>
<a href="http://hp256.com/2014/12/24/post-0002/" rel="nofollow">markdown语法进阶版</a>
<a href="http://theme-next.iissnan.com/faqs.html" rel="nofollow">hexo设置阅读全文</a>
<a href="https://www.v2ex.com/t/205208" rel="nofollow">next字体库更换</a>
<a href="https://github.com/iissnan/hexo-theme-next/issues/759#issuecomment-202242848" rel="nofollow">next_pisces主题留白太多调整</a>
<a href="http://fanjun.im/2016/09/hexo_next_seo.html" rel="nofollow">hexo SEO优化</a>
<a href="http://www.jianshu.com/p/86557c34b671#" rel="nofollow">hexo SEO优化 - 简书</a>
<a href="http://codecloud.net/16342.html" rel="nofollow">hexo SEO优化 - 程序员的资料库</a>

> 对SEO有兴趣的，推荐读一下这本书 **SEO实战密码**

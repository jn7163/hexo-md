---
title: nginx location匹配规则
date: 2016-10-18 13:25:22
categories:
- linux
- 网络服务
- nginx
tags:
- linux
- 网络服务
- nginx
- location
keywords: nginx, location
---
> location指令是http模块当中最核心的一项配置，根据预先定义的URL匹配规则来接收用户发送的请求，根据匹配结果，将请求转发到后台服务器、非法的请求直接拒绝并返回403、404、500错误处理等。下面说说location的匹配规则和优先级

<!-- more -->

## location 匹配语法
### 精确匹配
<pre><code class="language-nginx line-numbers">location = /images/a.png {
...
}
</code></pre>

### 前缀匹配
#### 普通前缀匹配
<pre><code class="language-nginx line-numbers">location /images/blog/ {
...
}
</code></pre>

#### 优先前缀匹配
<pre><code class="language-nginx line-numbers">location ^~ /images/blog/ {
...
}
</code></pre>

### 正则匹配
#### 区分大小写
<pre><code class="language-nginx line-numbers">location ~ .*\.php {
...
}
</code></pre>

#### 不区分大小写
<pre><code class="language-nginx line-numbers">location ~* .*\.PHP {
...
}
</code></pre>

#### 取反(区分大小写)
<pre><code class="language-nginx line-numbers">location !~ .*\.php {
...
}
</code></pre>

#### 取反(不区分大小写)
<pre><code class="language-nginx line-numbers">location !~* .*\.PHP {
...
}
</code></pre>

## location 匹配优先级
> 对于请求 https://www.zfl9.com/images/blog/php7.png

### 若命中精确匹配，如
<pre><code class="language-nginx line-numbers">location = /images/blog/php7.png {
...
}
</code></pre>

**则优先精确匹配，并终止匹配**

### 若命中多个前缀匹配，如
<pre><code class="language-nginx line-numbers">location /images/ {
...
}
location /images/blog/ {
...
}
</code></pre>

**则暂存最长前缀匹配，并继续搜索正则匹配**

### 若命中最长优先前缀匹配，如
<pre><code class="language-nginx line-numbers">location /images/ {
...
}
location ^~ /images/blog/ {
...
}
</code></pre>

**则命中最长优先前缀匹配，并终止匹配**

### 若命中多个正则匹配，如
<pre><code class="language-nginx line-numbers">location /images/ {
...
}
location /images/blog/ {
...
}
location ~ /images/ {
...
}
location ~ /images/blog/ {
...
}
</code></pre>

**则按照配置文件的上下顺序匹配第一个正则，并终止匹配**

### 若命中最长前缀匹配后没有搜索到正则，如
<pre><code class="language-nginx line-numbers">location /images/ {
...
}
location /images/blog/ {
...
}
</code></pre>

**则命中最长优先前缀匹配，并终止匹配**

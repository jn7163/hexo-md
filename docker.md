---
title: docker 容器
date: 2017-01-25 11:20:00
categories:
- 容器技术
- docker
tags:
- docker容器
keywords: docker, 容器
---
> 
Docker 使用 Google 公司推出的 Go 语言 进行开发实现，基于 Linux 内核的 cgroup，namespace，以及 AUFS 类的 Union FS 等技术，对进程进行封装隔离，属于操作系统层面的虚拟化技术。由于隔离的进程独立于宿主和其它的隔离的进程，因此也称其为容器

<!-- more -->

## 环境要求

- CentOS7
- 内核版本3.10以上

## 安装docker

<pre><code class="language-bash line-numbers">--- /etc/yum.repos.d/docker.repo ---
[docker]
name=Docker
baseurl=https://yum.dockerproject.org/repo/main/centos/7/
enabled=1
gpgcheck=1
gpgkey=https://yum.dockerproject.org/gpg
--- /etc/yum.repos.d/docker.repo ---

yum -y install docker-engine
</code></pre>

## 启动docker

<pre><code class="language-bash line-numbers">## 添加当前用户至docker用户组
usermod -aG docker $USER

systemctl enable docker
systemctl start docker
</code></pre>

## 镜像加速

<pre><code class="language-bash line-numbers">## 阿里云docker镜像加速 https://hpc.aliyun.com/doc/docker%E9%95%9C%E5%83%8F%E6%9C%8D%E5%8A%A1
## 注册并创建namespace后获得一个docker加速域名

## 配置加速
--- /etc/docker/daemon.json ---
{
  "registry-mirrors": ["docker加速域名"]
}
--- /etc/docker/daemon.json ---

systemctl restart docker
</code></pre>

## 常用参数

<pre><code class="language-bash line-numbers">docker search nginx     # 搜索镜像
docker pull centos      # 下载镜像，不指定tag则默认latest

docker images                   # 列出本机镜像
docker images -f dangling=true  # 列出虚悬镜像(dangling image)，虚悬镜像没有价值，应当删除！

docker run centos /bin/echo 'hello,docker'  # 运行容器
docker run -it centos /bin/bash             # 运行容器&伪终端
docker run -it --name test --rm centos /bin/bash    # 退出容器时删除该容器
docker run -d --name test --rm centos /bin/ping www.baidu.com   # -d参数，后台运行
docker run -d --name nginx -P nginx         # 映射端口(随机)
docker run -d --name nginx -p 80:80 nginx   # 映射端口(手动指定)

docker logs test    # 查看test容器的日志(输出)

docker port nginx   # 查看端口映射

docker top test     # 查看容器运行的进程

docker inspect test # 查看容器底层信息(json)

docker exec -it test /bin/bash      # 在运行的容器中执行命令

docker ps       # 查看当前运行的容器
docker ps -a    # 查看当前所有容器

docker start|stop|restart test  # 启动|停止|重启 容器

docker create       # 创建容器但不启动它

docker rm test      # 删除容器

docker rmi nginx    # 删除镜像

docker diff test    # 查看变动的文件

docker commit   # 将容器的存储层保存下来成为镜像
eg:
docker commit \
--author "Tao Wang <twang2218@gmail.com>" \
--message "修改了默认网页" \
webserver \
nginx:v2
# 不建议用commit创建镜像，太臃肿！

docker build -t nginx:v3 .      # docker build [选项] <上下文路径/URL/->
# 从Dockerfile构建镜像

docker export test > test.tar.gz    # 导出容器(支持url路径)
docker import test.tar.gz test:v2   # 导入容器(支持url路径)

## 数据卷

数据卷可以在容器之间共享和重用
对数据卷的修改会立马生效
对数据卷的更新，不会影响镜像
数据卷默认会一直存在，即使容器被删除

如果需要在删除容器的同时移除数据卷。可以在删除容器的时候使用 docker rm -v
挂载数据卷的默认权限是读写，用户也可以通过 :ro 指定为只读

docker run -d -v /opt/data:/data centos     # 本地目录/opt/data挂载至容器中的/data目录
docker run -d -v /opt/data centos           # 本地目录/opt/data挂载至容器中的/opt/data目录
docker run -d -v /opt/data:/data:ro centos  # 只读方式
</code></pre>

## 推送镜像

<pre><code class="language-bash line-numbers">## 此例使用阿里云加速节点！

docker login --username=UserName registry.cn-hangzhou.aliyuncs.com                 # 登录仓库
docker tag [ImageId] registry.cn-hangzhou.aliyuncs.com/otokaze/centos7:[镜像版本号] # 设置标签
docker push registry.cn-hangzhou.aliyuncs.com/otokaze/centos7:[镜像版本号]          # 推送镜像
</code></pre>

## Dockerfile

<pre><code class="language-bash line-numbers">Dockerfile 是一个文本文件，其内包含了一条条的指令(Instruction)，每一条指令构建一层，因此每一条指令的内容，就是描述该层应当如何构建。

在一个空白目录中，建立一个文本文件，并命名为 Dockerfile：
$ mkdir mynginx
$ cd mynginx
$ touch Dockerfile
其内容为：
-----------------------------------
FROM nginx
RUN echo 'Hello, Docker!' > /usr/share/nginx/html/index.html
-----------------------------------
这个 Dockerfile 很简单，一共就两行。涉及到了两条指令，FROM 和 RUN

## FROM 指定基础镜像

所谓定制镜像，那一定是以一个镜像为基础，在其上进行定制。就像我们之前运行了一个 nginx 镜像的容器，再进行修改一样，基础镜像是必须指定的。而 FROM 就是指定基础镜像，因此一个 Dockerfile 中 FROM 是必备的指令，并且必须是第一条指令

## RUN 执行命令
RUN 指令是用来执行命令行命令的
shell 格式：RUN <命令>，就像直接在命令行中输入的命令一样。
exec 格式：RUN ["可执行文件", "参数1", "参数2"]，这更像是函数调用中的格式。

## COPY复制文件
COPY package.json /usr/src/app/
COPY hom* /mydir/
COPY hom?.txt /mydir/

## CMD容器启动命令
正确的做法是直接执行 nginx 可执行文件，并且要求以前台形式运行。
比如  CMD ["nginx", "-g", "daemon off;"]

## ENTRYPOINT入口点

场景一：让镜像变成像命令一样使用

docker run myip -i # 无法传递参数！
docker: Error...

---------------------------------
FROM ubuntu:16.04
RUN apt-get update \
    && apt-get install -y curl \
    && rm -rf /var/lib/apt/lists/*
ENTRYPOINT [ "curl", "-s", "http://ip.cn" ]
---------------------------------

场景二：应用运行前的准备工作

启动容器就是启动主进程，但有些时候，启动主进程前，需要一些准备工作。

ENTRYPOINT ["docker-entrypoint.sh"]

## ENV设置环境变量

格式有两种：
ENV <key> <value>
ENV <key1>=<value1> <key2>=<value2>...

## VOLUME定义匿名卷
mysql等需要写入数据的容器

格式为：
VOLUME ["<路径1>", "<路径2>"...]
VOLUME <路径>

## EXPOSE声明端口

格式为 EXPOSE <端口1> [<端口2>...]。

EXPOSE 指令是声明运行时容器提供服务端口，这只是一个声明，并不影响运行时的端口设置！

## WORKDIR指定工作目录

格式为 WORKDIR <工作目录路径>。
使用 WORKDIR 指令可以来指定工作目录（或者称为当前目录），以后各层的当前目录就被改为指定的目录，该目录需要已经存在，WORKDIR 并不会帮你建立目录

## USER 指定当前用户

格式：USER <用户名>
USER 指令和 WORKDIR 相似，都是改变环境状态并影响以后的层。WORKDIR 是改变工作目录，USER 则是改变之后层的执行 RUN, CMD 以及 ENTRYPOINT 这类命令的身份

## HEALTHCHECK 健康检查

格式：
HEALTHCHECK [选项] CMD <命令>：设置检查容器健康状况的命令
HEALTHCHECK NONE：如果基础镜像有健康检查指令，使用这行可以屏蔽掉其健康检查指令

HEALTHCHECK 支持下列选项：

--interval=<间隔>：两次健康检查的间隔，默认为 30 秒；
--timeout=<时长>：健康检查命令运行超时时间，如果超过这个时间，本次健康检查就被视为失败，默认 30 秒；
--retries=<次数>：当连续失败指定次数后，则将容器状态视为 unhealthy，默认 3 次。
和 CMD, ENTRYPOINT 一样，HEALTHCHECK 只可以出现一次，如果写了多个，只有最后一个生效。

0：成功；1：失败；2：保留，不要使用这个值

----------------------------------------
FROM nginx
RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*
HEALTHCHECK --interval=5s --timeout=3s \
CMD curl -fs http://localhost/ || exit 1
----------------------------------------

为了帮助排障，健康检查命令的输出（包括 stdout 以及 stderr）都会被存储于健康状态里，可以用 docker inspect 来查看。

docker inspect --format '{{json .State.Health}}' web | python -m json.tool
</code></pre>

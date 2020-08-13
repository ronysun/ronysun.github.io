---
layout: post
title: "Dockerfile介绍"
categories:
  - test
---
## Dockerfile
Dockerfile常用命令：

ADD & COPY ：拷贝文件到镜像中。ADD能够自动解压.gz .zip等压缩文件，并能够添加URL文件  
ADD test.tar.gz /root/tmp/  
ADD http://172.28.15.92/rpmdir/python2-netmiko-2.0.1.2-1.el7.noarch.rpm /root

RUN：执行一条命令  
RUN yum install -y httpd

FROM: 基于某一个baseImage构建docker镜像  
FROM centos：7.4.1708

WORKDIR: 为RUN CMD等命令配置工作目录  
WORKDIR /root

LABEL:为docker镜像指定标签  
LABEL maintainer='sunln2008@foxmail.com'

ENTRYPOINT & CMD :容器启动时运行的命令,CMD在docker run时能够被覆盖，ENTRYPOINT则不会
ENTRYPOINT ls /root/  
CMD ls /root/

VOLUME:进行宿主机与容器或者容器之间卷的挂载。可实现数据持久化和共享  
VOLUME /DATA  
 screen ~/Library/Containers/com.docker.docker/Data/com.docker.driver.amd64-linux/tty  
 cd /var/lib/deocker/volumes

EXPOSE:对外暴露网络端口，实际使用时需要配置docker run -P（大写）参数  
EXPOSE 80/tcp

USER:设置容器启动时的默认用户，需要提前添加用户  
USER jenkins

ARG & ENV:定义环境变量or参数，ARG能够在docker build时通过--build-arg <varname>=<value>进行传递  
ENV test_value1='foo' test_value2='foo2'  
ARG usernmae=jenkins


Dokcerfile的所有命令：
1. FROM
1. RUN
1. CMD
1. LABEL
1. MAINTAINER [deprecated]
1. EXPOSE
1. ENV
1. ADD
1. COPY
1. ENTRYPOINT
1. VOLUME
1. USER
1. WORKDIR
1. ARG
1. ONBUILD
1. STOPSIGNAL
1. HEALTHCHEECK
1. SHELL

Dockerfile example：
```Dockerfile
FROM centos:7.4.1708
LABEL maintainer='sunln2008@foxmail.com'
ARG username=jenkins
ADD testfile1 testfile2 /root/
ADD test.tar.gz /root/tmp/
ADD http://172.28.15.92/rpmdir/openstacksdk-0.11.4.10.dev4.tar.gz /root
RUN mkdir /data
RUN echo "xxxx" > /data/testfile3
VOLUME /data
EXPOSE 80/tcp
ENV test_value1='foo' test_value2='foo2'
RUN useradd ${username}
USER ${username}
# RUN nc -l 80 &
# ENTRYPOINT ls /data/
# CMD ls /data
```


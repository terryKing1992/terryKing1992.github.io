---
layout: post
title: 制作centos基础镜像
date: 2018-10-18 22:30:00
tags: [docker, 镜像, centos]
---

### 前言

> 现在我所在的公司使用的操作系统都是centos7.4版本的, 当然应用也是部署在centos上面, 那么如果使用docker部署的话, 也自然而然的想到基于centos镜像来构建自己的应用镜像; 但是centos基础镜像与应用镜像之间也可以构建一下基础框架的镜像, 比如: 基础JDK镜像, 基于Python环境镜像等等; 相信大家也遇到过, 构建了一个镜像发现部署应用的时候打印出来的中文是乱码, 日志的时间显示的是UTC时间, 比北京时间少了8个小时, 想看某一个进程是否起来的时候发现telnet命令 ps命令 netstat命令不可用, 想要编辑文件的时候发现vim命令也不能用; 今天我们就来看一下如何基于centos镜像构建一个增强版的centos镜像;

### 镜像仓库的搭建

在构建自己的镜像之前, 需要有一个地方能够存放我们的镜像便于我们后面使用直接拿来可以用; 大概有两种思路: 第一种、在自己的机器上构建, 然后通过命令 ```docker save -o xxx.tar.gz xxx-image:v1.0```, 然后上传到自己的云盘上, 使用的时候下载到本地, 然后 ```docker load -i xxx.tar.gz```; 这种方式是没问题的, 但是就是比较繁琐(打包，上传下载，load的时候还要找到目录神马的); 第二种、创建自己的一个docker image registry, 对于本地构建的镜像 只需要重新 tag一下, docker push即可完成;下载的时候只需要docker pull就可以了, 全部都可以在命令行上面搞定, 想想都有一种裤裤的感觉;

说了这么多, 让我们一起看一下如何搭建一个自己的镜像仓库;(本文中仅仅使用了阿里云的仓库, 建了自己的namespace而已),当然也可以自己申请云主机, 然后搭建私服都是没有问题的;

首先登录阿里云, 然后从服务中找到容器镜像服务所在的位置, 如下图所示:

![搜索容器镜像服务](/assets/images/2018-10-18-aliyun_docker_registry_home_page.png)

点击容器镜像服务进入到镜像仓库创建界面如下图

![创建镜像仓库](/assets/images/2018-10-18-aliyun-docker-registry-create.png)

点击创建, 进入到创建仓库界面

![创建镜像仓库](/assets/images/2018-10-18-aliyun-docker-registry-create-details.png)

填写完信息之后, 点击下一步选择如何上传镜像

![镜像上传方式](/assets/images/2018-10-18-aliyun-docker-registry-select-build-method.png)

因为我的需求就是从本地仓库构建, 不过我认为本地仓库构建是最灵活的一种方式;

创建好镜像存储的位置之后, 就可以在本地创建Dockerfile来构建自己的镜像了;

### 构建centos增强版镜像

1、首先, 创建Dockerfile文件, 将如下内容放入文件中

```text
FROM centos
MAINTAINER terry.king "1575639478@qq.com"

# 定义时区参数
ENV TZ=Asia/Shanghai
RUN ls -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo '$TZ' > /etc/timezone
# 设置编码
RUN localedef -c -f UTF-8 -i zh_CN zh_CN.utf8
ENV LC_ALL "zh_CN.UTF-8"

# 安装基础yum包
RUN yum install -y gcc gcc-c++ pcre pcre-devel zlib zlib-devel openssl openssl-devel patch net-tools iproute telnet bind-utils wget kde-l10n-Chinese glibc-common

```

2、登录阿里云Docker Registry

```sudo docker login --username=你的阿里云账号 registry.cn-shenzhen.aliyuncs.com```

用于登录的用户名为阿里云账号全名，密码为开通服务时设置的密码。

您可以在产品控制台首页修改登录密码。就是第二张创建镜像仓库按钮的左边设置registry登录密码

3、构建镜像并将镜像推送到Registry

```shell
$ docker build -t terrylmay/centos .
$ docker tag terrylmay/centos registry.cn-shenzhen.aliyuncs.com/terrylmay/centos:[镜像版本号]
$ docker push registry.cn-shenzhen.aliyuncs.com/terrylmay/centos:[镜像版本号]
```

默认的镜像版本号为latest

### 总结

这样在构建应用镜像的时候就可以解决序言中提到的一系列问题了. 方便自己快速构建自己的应用镜像;

---
title: Docker基础及深度学习应用
date: 2019-06-5 18:11:35
tags: Docker
categories: "tool"
archives: "2019-06"
---
使用Docker进行环境管理， 包含Docker的基础操作和深度学习应用。


# 一、基础

## 1.1概念

- 官方概念:
类比大货轮(生产环境:ubuntu centos)上的集装箱,集装箱可以将化学品 食物 玩具等等一个个独立分割(即可以满足不同版本的tensorflow版本模型),只需要一艘货轮就可以运送食物 化学品等,并且在同一艘大货轮中可以一直持续的增加集装箱,并且进行统一管理(消耗低 通信方便)
- 解决痛点:
    1. 问题: "我的本地机器可以运行啊, 线上怎么不可以运行啊"
    1. 问题2: 深度学习繁杂的框架和依赖, 我太懒了, 对linux环境也不是很熟, 也不想花费很多时间和精力去折腾环境, 这时候怎么办?
    1. 问题3: 假设构建一个算法平台, 包含如下三个算法, 这种情况, 你如何在一台服务器上, 整合所有的算法集成为一个算法平台?
        - 文本摘要: tensorflow-gpu==1.10 | cudnn==7.0 | gcc=4
        - 命名实体识别: tensorflow-gpu==1.4 | cudnn==6.0 | gcc=4
        - 文本分类: pytroch=1.1 | gcc==4.7
        - 构建三个docker容器, 指定相关端口进行通讯, 直接集成所有算法
    - 解决方案: docker 大法好, 下载相关的docker镜像, 运行镜像创建环境, 直接保证无痛运行

## 1.2容器

**不虚拟化操作内核, 只虚拟化相关操作系统接口**
概念
- 一种虚拟化方案
- 操作系统级别的虚拟化
- 只能运行相同or相似内核的操作系统
- 依赖于linux内核特性: namespace和cgroups (control group)

优点
- 占用磁盘空间小
- 资源占用小
- 对CPU 内存资源消耗小

## 1.3框架图

wKiom1jCbLPTOe1iAAERmX6iMrE621.png

## 1.4安装

- 主要系统安装(google 搜索以下关键字)
    - [ubuntu18 install docker](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-18-04)
- 系统要求
    - x64架构: uname -r 可以直接查看
- 注意:
    - 下载安装docker(root用户)
        - 直接使用国外镜像安装可以会报超时错误,直接通过国内源的方式进行安装
    - 非sudo用户使用docker
        - sudo usermod -aG docker your_username
        - 设置完毕后需要重新登录该用户才能通过不加sudo直接使用docker
    - 修改docker **image**仓库
```
#对于使用 systemd 的系统，请在 /etc/docker/daemon.json 中写入如下内容（如果文件不存在请新建该文件）
{
  "registry-mirrors": [
    "https://registry.docker-cn.com"
  ]
}
```

# 二 Docker

## 2.1基础命令

- Dockerfile: (相关命令, 构建镜像)
- image(镜像):
    - docker image ls: 显示当前镜像
    - docker pull image(镜像名): 从docker仓库中拉取镜像
    - docker image rm image_id: 移除相关镜像
        - 必须先删除相关容器才能删除镜像
- contanin(容器):
    - docker run -it imageName bash: 运行镜像并进入bash界面
    - docker ps: 查看当前运行的容器
    - docker contanin ls -a: 查看所有运行过的容器
    - docker contanin rm contanin_id: 根据容器id删除容器
    - docker commit -m="modify msg" -a="author" containerId imageName
        1. docker ps : 查看当前运行容器id
        1. 需要在当前未退出容器的基础上, 打开另外一个shell界面, 保存当前容器为新镜像

## 2.2常用操作

- 常用操作:
    - docker run -it 镜像名 bash: 运行镜像并进入bash界面
    - docker run -it -v 共享文件夹路径:/容器内共享文件夹路径 镜像名 bash
    - docker run -it -p 8000:8000 -v 共享文件夹路径:/容器内共享文件夹路径 镜像名 bash
        - -p: 指定网络端口进行通信
        - -v: 指定共享文件夹
    - **CTRL + P +Q: 退出容器但保证后台运行**
- 常用仓库:
    - [Nvidia docker GPU加速docker](https://github.com/NVIDIA/nvidia-docker)
    - [depo-国人一站式深度学习docker仓库](https://hub.docker.com/r/ufoym/deepo)
- 保存及备份操作:

# 参考

1. [如何通俗易懂的说明docker](https://www.zhihu.com/search?type=content&q=docker)
1. [docker 从入门到实践](https://yeasy.gitbooks.io/docker_practice/network/port_mapping.html)
1. [阮一峰Docker入门教程](http://www.ruanyifeng.com/blog/2018/02/docker-tutorial.html)
1. [阮一峰Docker微服务](http://www.ruanyifeng.com/blog/2018/02/docker-wordpress-tutorial.html)
1. [docker 基础概念和框架](https://blog.51cto.com/tangyade/1905232)
1. [docker 优势-知乎](https://www.zhihu.com/question/22871084)


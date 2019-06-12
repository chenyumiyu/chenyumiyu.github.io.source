---
title: 空间向量搜索构建流程（Sptag+Bert+Docker）
date: 2019-06-13 0:11:35
tags: 向量搜索
categories: "深度学习"
archives: "2019-06"
---
微软SPTAG向量搜索工具工程化构建总结。

## Spatg

### 1.概述

微软官方在Github 开源的大规模最近邻向量搜索库， [Sptag](https://github.com/microsoft/SPTAG)github地址。
- 主要流程： 样本->转化为向量->根据样本向量构建KDT图
- 提供构建方法：
    - SPTAG-KDT：优势在于索引构建花费较低
    - SPTAG-BKT：在高维向量搜索更加准确

### 2.依赖安装：

- 第一种依赖安装：
    - swig >= 3.0
    - cmake >= 3.12.0
    - boost >= 1.67.0
    - tbb >= 4.2
    - 优点：需要基于linux环境进行相关依赖安装，不需要Docker的学习成本
    - 缺点：可能会破坏原有linux系统环境
- 基于Docker的环境构建：
    - git clone Sptag仓库：git clone https://github.com/microsoft/SPTAG.git
    - 修改SPTAG仓库下Dockerfile内容：
        - 14行内容改为如下：
        > RUN wget "https://github.com/Kitware/CMake/releases/download/v3.14.4/cmake-3.14.4-Linux-x86_64.tar.gz" -O - \
        - 18行内容改为如下:
        > RUN wget "https://dl.bintray.com/boostorg/release/1.67.0/source/boost_1_67_0.tar.gz" -O - \
        - 最后添加Docker镜像中文支持：
            > ENV LANG C.UTF-8
    - docker build -t sptag . : 生成sptag docker 镜像，时间约为：20分钟
    - [docker 基础入门](https://www.chenyumiyu.club/2019/06/05/Docker%E5%9F%BA%E7%A1%80%E5%8F%8A%E6%B7%B1%E5%BA%A6%E5%AD%A6%E4%B9%A0%E5%BA%94%E7%94%A8/)

### 3.训练向量搜索图：

- [官方Sptag详细用法,亲测可用](https://github.com/microsoft/SPTAG/blob/master/docs/GettingStart.md)

## Bert:

自Google开源以来，Bert在NLP各大领域都取得比原先词向量（word2vec、glove等）更好的效果, 我们可以在Google 开源的bert基础之上，直接使用 Bert 来进行样本向量表示， 最后基于bert向量的样本通过SPTAG构建向量搜索图。

### 相关资源下载：

- [Bert 中文向量下载](https://github.com/google-research/bert)

### 构建Bert 句向量生成服务（todo）

- 先占坑先

# 参考

- [肖涵博士-两行代码玩转句向量](https://zhuanlan.zhihu.com/p/50582974)
- [puluwen-微软SPTAG向量搜索工具](https://puluwen.github.io/2019/05/Microsoft-SPTAG-introduction/)
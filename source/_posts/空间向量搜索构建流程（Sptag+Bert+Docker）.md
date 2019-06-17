---
title: 空间向量搜索构建流程（Sptag+Bert+Docker）
date: 2019-06-13 0:11:35
tags: 向量搜索
categories: "深度学习"
archives: "2019-06"
---
微软SPTAG向量搜索工具工程化构建总结。

## 流程

- 构建流程及指标： 
    - 构建流程： 句子 -> Bert句向量 -> Sptag构建空间向量搜索图
    - 训练时间参考： 原始200W数据， 因为目前Sptag仅支持单核CPU构建的情况，训练时间约24h
- 搜素流程及指标：
    - 句子 -> Bert句向量： 60ms，基于cpu， 可以使用GPu并行加速获取句向量
    - Bert句向量 -> Sptag空间向量搜索： 40ms
    

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
    - 缺点：可能会破坏原有linux系统环境, cmake 和 swig版本可能会有区别
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

### 构建Bert句向量生成服务

- 安装相关依赖：
    - pip install bert-serving-server： 如果仅需要使用Bert模型提供句向量服务给Python程序，需要安装该依赖
    - pip install bert-serving-client： Python端Client，获取句向量
    - pip install -U bert-serving-server[http]： 如果需要将Bert句向量服务提供http请求的方式，需要安装该依赖（基于flask框架，提供一个单进程的http服务，如果对并发有要求还是最好自己去封装吧）
- 支持http请求版本：
    - bert-serving-start -model_dir=/YOUR_MODEL -http_port 8125
- 多线程版本：
    - 设置基于cpu提供Bert句向量
    - 设置并发数为：10
    - 对外端口为： 5555
```python
# -*- coding: utf-8 -*-
# @Time  : 6/12/19 10:03 AM
# @Author : zhongzhaochang
# @Project : bert
# @Desc : 开启bert服务， 对外提供句子转bert向量
import os

from bert_serving.server.helper import get_args_parser
from bert_serving.server import BertServer


cur_dir = os.path.dirname(os.path.realpath(__file__))
model_dir = os.path.join(cur_dir, "models")

bert_model = os.path.join(model_dir, "chinese_L-12_H-768_A-12")

# start： python bert_service.py
# stop: bert-serving-terminate -port 5555(or 你指定监听客户端的端口)
args = get_args_parser().parse_args(['-model_dir', bert_model,
                                     '-port', '5555',
                                     '-port_out', '5556',
                                     '-max_seq_len', 'NONE',
                                     '-mask_cls_sep',
                                     '-cpu',
                                     '-num_worker', '10'])
server = BertServer(args)
server.start()

```
```python
# -*- coding: utf-8 -*-
# @Time  : 6/11/19 6:34 PM
# @Author : zhongzhaochang
# @Project : bert
# @Desc : 简单bert_client 获取句向量代码
from bert_serving.client import BertClient


def get_vector_by_bert_client():


    bc = BertClient(ip="127.0.0.1", port=5555)
    while True:
        question = input("input sentence:\n")
        vectors = bc.encode([question])
        print(str(vectors), vectors.shape)


if __name__ == '__main__':
    get_vector_by_bert_client()

```

#### issue：
- 启动Bert model模型时，可能会出现该问题：
    - INFO：tensorflow: Could not find trained model in model_dir: /tmp/xxxx, running initialization to predict
    ```
    #Bert 仓库的解释是：
    Note: You may see a message like Could not find trained model in model_dir: /tmp/tmpuB5g5c, running initialization to predict. 
    This message is expected, it just means that we are using the init_from_checkpoint() API rather than the saved model API. If you don't specify a checkpoint or specify an invalid checkpoint, this script will complain.
    我的做法是： 直接忽略它， 一个原因是该信息的重要等级只是INFO级别， 二是因为官方仓库的解释
    ```
    
# 参考

- [知乎-两行代码玩转句向量](https://zhuanlan.zhihu.com/p/50582974)
- [puluwen-微软SPTAG向量搜索工具](https://puluwen.github.io/2019/05/Microsoft-SPTAG-introduction/)
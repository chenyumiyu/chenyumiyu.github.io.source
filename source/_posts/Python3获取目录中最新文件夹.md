---
title: Python3获取目录中最新文件夹
date: 2019-06-17 10:18:35
tags: 常用代码
categories: "Python"
archives: "2019-05"
---
获取一个目录中的最新文件夹，个人主要应用：深度学习会生成多个模型，直接使用该代码可以获取最新的模型名。

### 常用代码

```python
import os


def find_newest_folder(parent_dir):
    """
    :param parent_dir: 父目录绝对路径
    :return: 当前最新修改的目录绝对路径
    """
    all_subdirs = [os.path.join(parent_dir, dir)
                   for dir in os.listdir(parent_dir)
                   if os.path.isdir(os.path.join(parent_dir, dir))]

    latest_subdir = max(all_subdirs, key=os.path.getmtime)
    return latest_subdir
```

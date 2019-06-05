---
title: 正则删除非中文字符
date: 2019-05-24 16:18:35
tags: 常用代码
categories: "Python"
archives: "2019-05"
---
使用github+hexo搭建个人blog，并使用appveyor进行版本控制和持续集成。

### 常用代码
在做开发的过程中，偶尔会遇到需要去除掉相关字符串中的非中文字符的情况，以下是相关的代码。

```python
import re


valid_string_template = re.compile(r"[^\u4e00-\u9fa5|\w]+")  # 匹配中文和数字英文正则

def filter_all_punctuation_and_non_utf8(self, text):
    """
    :des 进行输入预处理:过滤所有的标点符号and non-utf8 string
    :param text: 输入,一般为字符串
    :return: 过滤掉标点符号的结果
    """
    result = re.sub(valid_string_template , '', text)
    return result
```

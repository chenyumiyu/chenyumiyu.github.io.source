---
title: github+hexo搭建个人博客
date: 2019-05-23 15:11:35
tags: blog
categories: "tool"
archives: "2019-05"
---
使用github+hexo搭建个人blog，并使用appveyor进行版本控制和持续集成。

## 流程
    
### 1.github

全球程序最大同性交友网站，代码搬运的不二之选，程序猿的必备工具之一。
本文中最重要的目的是进行代码的版本控制和博客内容的存放。

- [github 帐号注册](https://github.com/)
- [git安装](https://git-scm.com/book/zh/v2/%E8%B5%B7%E6%AD%A5-%E5%AE%89%E8%A3%85-Git)
- [git简单使用](https://www.liaoxuefeng.com/wiki/896043488029600/896067074338496)
- github blog 内容仓库创建

### 2.node.js

- [node.js安装](https://nodejs.org/zh-cn/download/)
- 检测node.js是否被正确安装： node -v(命令行下)
- 检测npm是否正确安装： npm -v

### 3.hexo

Hexo是一款基于Node.js的静态博客框架，依赖少易于安装使用，可以生成静态网页直接托管在
github和heroku上。

- hexo安装: npm install -g hexo-cli
- hexo创建blog: hexo init blog
- blog项目主要路径说明：
    - node_modules： hexo的node.js模块
    - source： blog源代码
        - _posts
            - blog.md: Markdown格式的个人blog
        - CNAME: 如果你有域名的话，可以直接将域名指向到github的博客内容，文件内容为你的域名
    - themes： hexo博客主题
    - _config.yml： blog主配置文件， 只需要简单配置，就可以实现
    - .gitignore： git上传文件控制，可以控制部分敏感or无用的文件提交
    
- hexo框架主要命令：
    - hexo new blog.md: 新建blog
    - hexo g： 根据新建blog生成静态页面
    - hexo s： 本地debug测试
    - hexo d: 发布到github上（需要进行全局配置）

- hexo博客主要配置_config.yml：
```
# Hexo Configuration
## Docs: https://hexo.io/docs/configuration.html
## Source: https://github.com/hexojs/hexo/

# Site
# 需要设置的部分
title: Zzc's Blog
subtitle: hello, world!
description: Python, 自然语言处理, 深度学习, 羽毛球爱好者.
keywords: Python, 自然语言处理, 机器学习, 深度学习
author: Zzc
language: zh-Hans   # 语言设置
timezone: Asia/Shanghai # 时区设置

# URL
# 修改域名指向需要设置的部分
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
url: https://www.chenyumiyu.club
root: /
permalink: :year/:month/:day/:title/
permalink_defaults:

# 主题，可以自己在github上寻找自己需要的主题
# 默认为： hexo
# 存储文件夹： themes
theme: replica

# Deployment
# 需要设置的部分
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git(托管平台类型)
  repo: 你的github博客内容仓库
  branch: master（分支）

```
- 域名指向user_name.github.io
    - 购买域名： 需要进行域名备案才能进行解析（阿里云or腾讯云）
    - user_name.github.io的ip： ping user_name.github.io
    - 解析域名： 参考域名提供商的官方文档
        - 修改相关参数可以直接将域名直接指向到博客ip
    
### 4.hexo+git工具安装

    Hexo此时创建的blog只能在本地运行，不能外网访问的blog还有啥意义呢

- npm install hexo-deployer-git --save: hexo部署到git工具安装 
- hexo clean: 清除缓存
- hexo g: 生成blog静态页面
- hexo d： 发布到github博客内容仓库

### 5.创建blog源代码仓库

### 6.Appveyor进行hexo博客持续集成（注意： 需要科学上网）

有了源代码管理了，可以在不同的电脑进行blog编写了，但是如果只是下载下来了hexo的源代码版本库，
难道又要重新走一边在上面的流程嘛？Appveyor就是这样的一个平台，监控你的源博客提交，并自动部署。
当然它的功能不知如此，有兴趣的可以深入研究。
    
- [Appveyor官网](https://www.appveyor.com/docs/)
- 登录： 直接使用github帐号登录
- 源代码新建appveyor.yml文件（修改相关注明文件，其他不用修改）
```
clone_depth: 5
 
environment:
  nodejs_version: "11"  # 最好填写对应的nodejs版本，因为有些hexo主题对nodejs对版本有要求
  access_token:
    secure: appveyor 转换后的github personal key
 
install:
  - ps: Install-Product node $env:nodejs_version
  - node --version
  - npm --version
  - npm install
  - npm install hexo-cli -g
 
build_script:
  - hexo generate
 
artifacts:
  - path: public
 
on_success:
  - git config --global credential.helper store
  - ps: Add-Content "$env:USERPROFILE\.git-credentials" "https://$($env:access_token):x-oauth-basic@github.com`n"
  - git config --global user.email "%GIT_USER_EMAIL%"
  - git config --global user.name "%GIT_USER_NAME%"
  - git clone --depth 5 -q --branch=%TARGET_BRANCH% %STATIC_SITE_REPO% %TEMP%\static-site
  - cd %TEMP%\static-site
  - del * /f /q
  - for /d %%p IN (*) do rmdir "%%p" /s /q
  - SETLOCAL EnableDelayedExpansion & robocopy "%APPVEYOR_BUILD_FOLDER%\public" "%TEMP%\static-site" /e & IF !ERRORLEVEL! EQU 1 (exit 0) ELSE (IF !ERRORLEVEL! EQU 3 (exit 0) ELSE (exit 1))
  - git add -A
  - if "%APPVEYOR_REPO_BRANCH%"=="master" if not defined APPVEYOR_PULL_REQUEST_NUMBER (git diff --quiet --exit-code --cached || git commit -m "Update Static Site" && git push origin %TARGET_BRANCH% && appveyor AddMessage "Static Site Updated")
```
- [appveyorKey生成](https://ci.appveyor.com/tools/encrypt)
- 持续集成： git clone 你的hexo博客源代码仓库，直接在source下新建相关文件，然后再提交到github远程仓库就可以了，appveyor会自动监控并且部署
- 新建blog：
    - 直接复制原有的博客，然后修改名字就可以了
    - 修改文件首部的tag， 标题， 分类， 归档

### 7.未完待续

- 加入搜索引擎收录
- 加入站点访问统计
    - [站点统计示例](https://www.jianshu.com/p/c311d31265e0)
- 加入打赏
- 博客迁移到云服务器，因为国内访问太慢了
- 博客加入评论功能

## 坑

- 1.域名映射
    - user_name.github.io原博客地址为国外域名， 直接在域名提供商解析域名处选择为国外域名，不翻墙基本无法访问
- 1.github can not find tags(categories or ...)
    - hexo初始化hexo的source下不存在分类、tag目录
    - 创建： hexo new page "categories"
    - 创建： hexo new page "tags"
    - 创建： hexo new page "archives"
- 2.github生成personal key在appveyor使用中遇到的坑
    - 无法直接使用，需要使用appveyor的加密，然后在填入appveyor.yml文件中

# 参考

- [Markdown语法](https://www.appinn.com/markdown/)
- [hexo搭建个人blog](https://zhuanlan.zhihu.com/p/26625249)
- [hexo博客迁移到云服务器](https://zhuanlan.zhihu.com/p/58654392)
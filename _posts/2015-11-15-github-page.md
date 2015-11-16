---
layout: post
title: 使用Github Page做技术博客
categories: [github, jekyll]
tags: git
---

* 目录
{:toc}

## 基本使用

### 注册Github

略过

### 配置本地Git

1. 用户名和邮箱，如果之前用过可以跳过

```
git config --global user.name “dlworld"  
git config --global user.email “linwendeng@mail.com”  
```

1. 还可以配置保存密码，可选

```
dlw@dlw:Workspace$ git config --global credential.helper cache
dlw@dlw:Workspace$ git config --global credential.helper 'cache --timeout=3600000'
```


### 创建Repo
创建格式为**username**.github.io的项目，如：dlworld.github.io

### 选择模板
使用的是基于poole的lanyon，比较简洁。

```
dlw@dlw$ git clone https://github.com/poole/lanyon.git dlworld.github.io
```

### 配置
配置文件为_config.yml

```
dlw@dlw:dlworld.github.io$ cat _config.yml 
# Permalinks
#
# Use of `relative_permalinks` ensures post links from the index work properly.
permalink:           pretty
#relative_permalinks: true
# Setup
title:               Hello DLWorld
tagline:             'Blog'
description:         ''        
url:                 http://dlworld.github.io
github_username:     dlworld    
baseurl:             ''
gems:                [jekyll-paginate]    
paginate:            5
paginate_path:       "page:num" 
baidu_analytics:     dlworld    
# About/contact
author:
  name:              dlworld
  email:             linwendeng@gmail.com
# Build settings
markdown: kramdown
kramdown:
  input: GFM                   # use Github Flavored Markdown !important
```
- paginate，分页插件，
- kramdown，必须指明input GFM，否则代码无法正常换行

### 提交

本地编辑完成/测试（见高级配置）后，就可提交到github。

```
dlw@dlw:dlworld.github.io$ git push origin master
```
提交后会自动编译，如果没有错误，就可以通过**username**.github.io访问。

*注：*如果远程版本库包含您本地尚不存在的提交，可以强制提交

```
dlw@dlw:dlworld.github.io$ git push origin +master
```

## 高级配置

### 本地调试

1. 修改ruby仓库
由于rubygems在国内不便访问，可改用淘宝的仓库

```
dlw@dlw:Workspace$ gem source --remove https://rubygems.org/
https://rubygems.org/ removed from sources
dlw@dlw:Workspace$ gem source -a https://ruby.taobao.org/
https://ruby.taobao.org/ added to sources
```

1. 安装github-pages包

```
dlw@dlw:dlworld.github.io$ sudo gem install github-pages -V
```

1. 运行

```
dlw@dlw:dlworld.github.io$ sudo bundle exec jekyll serve
```

1. 查看
浏览器打开127.0.0.1:4000

### 文章归档
categories

### 文章目录
Jekyll的Markdown渲染器kramdown已经支持目录树（Table Of Content）功能，只需在文章内标识toc生成的位置即可。

```
* 目录
{:toc}
```

### 使用SSH替换HTTPS

```
dlw@dlw:dlworld.github.io$ git remote -v
origin	https://github.com/dlworld/dlworld.github.io (fetch)
origin	https://github.com/dlworld/dlworld.github.io (push)
dlw@dlw:dlworld.github.io$ git remote set-url origin git@github.com:dlworld/dlworld.github.io
dlw@dlw:dlworld.github.io$ git remote -v
origin	git@github.com:dlworld/dlworld.github.io (fetch)
origin	git@github.com:dlworld/dlworld.github.io (push)
```


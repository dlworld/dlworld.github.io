---
layout: post
title: 使用Github Page做技术博客
tag: git
---

## 注册Github帐号
略过

## 配置本地Git
用户名和邮箱，如果之前用过可以跳过

还可以配置保存密码


## 创建Repo
dlworld.github.io

## 选择模板
使用的是基于poole的lanyon，比较简洁。

```
dlw@dlw$ git clone https://github.com/poole/lanyon.git dlworld.github.io
```

## 配置
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
```

## 提交

```
dlw@dlw:dlworld.github.io$ git push origin master
```

*注：*如果远程版本库包含您本地尚不存在的提交，可以强制提交

```
dlw@dlw:dlworld.github.io$ git push origin +master
```


# 本地调试

## 安装ruby包
1. 修ruby仓库
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


# 其它
## 使用SSH替换HTTPS

```
dlw@dlw:dlworld.github.io$ git remote -v
origin	https://github.com/dlworld/dlworld.github.io (fetch)
origin	https://github.com/dlworld/dlworld.github.io (push)
dlw@dlw:dlworld.github.io$ git remote set-url origin git@github.com:dlworld/dlworld.github.io
dlw@dlw:dlworld.github.io$ git remote -v
origin	git@github.com:dlworld/dlworld.github.io (fetch)
origin	git@github.com:dlworld/dlworld.github.io (push)
```


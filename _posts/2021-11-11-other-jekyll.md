---
layout: article
title: "Jekyll搭建blog"
data: 2021-11-11
tags:
  - other

---

## 摘要  

通过Jekyll搭建静态Blog网站

## Jekyll安装

基于CentOS7.9

```bash

 #安装rvm
 gpg2 --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB
 curl -sSL https://get.rvm.io | bash -s stable
 rvm install 3.0.0
 #安装jekyll bundle
 gem install jekyll bundle
 jekyll new my-awesome-site
 cd my-awesome-site
 bundle install
 bundle exec jekyll serve
```

## Jekyll主题

在jekyllthemes.org 上有很多主题可用挑选使用

## Jekyll pulgin

在<https://github.com/jekyll> 上有很多plugin可供使用，典型的有：  
[jekyll-admin](https://github.com/jekyll/jekyll-admin)  
[jekyll-archives](https://github.com/jekyll/jekyll-archives)

## Liquid是Jekyll的模板语言

<https://liquid.bootcss.com/>

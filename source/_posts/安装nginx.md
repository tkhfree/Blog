---
title: nginx搭建静态网站
tags:
  - Nginx
comments: true
toc: true
mathjax: true
abbrlink: c1018d6
date: 2018-09-18 00:21:52
urlname:
categories:
thumbnail:
---

# 安装nginx

```
yum install nginx -y
```

# 启动nginx

```
nginx
```
# 更改默认静态文件存储位置

```
vim /etc/nginx/nginx.conf 
```
root 更改/data/www/

# www文件夹创建index.html

```
#vi index.html
<!DOCTYPE html>

```
# 登陆IP/index.html
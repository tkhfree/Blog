---
title: wsgi、uwsgi、uWSGI、nginx、apache
tags:
  - Python
comments: true
toc: true
mathjax: true
abbrlink: fec6eec3
date: 2020-06-01 10:36:29
urlname:
categories:
thumbnail:
---

python web开发需要理解这几个概念

# wsgi

wsgi是一个网络通信协议，具体内容就是规范application和server的通信格式。

wsgi接口定义很简单，只要求Web开发者实现一个函数，来相应Http请求。

```python
def application(environ, start_response):
    start_response('200 OK', [('Content-Type', 'text/html')])
    return '<h1>Hello, web!</h1>'
```

> 上面的`application()`函数就是符合WSGI标准的一个HTTP处理函数，它接收两个参数：
>  `environ`：一个包含所有HTTP请求信息的dict对象；
>  `start_response`：一个发送HTTP响应的函数。

> `application()`函数必须由WSGI服务器来调用

#  uwsgi

uwsgi是一个**线路协议**

与wsgi是两种东西，是`uWSGI服务器`的独占协议

# uWSGI

uWSGI是一个**web应用服务器**，这个web服务器是跟application进行交互的

# Nginx

一般做代理服务器/反向代理，uWSGI作为web服务器承担不了大规模的http请求，使用Nginx代理把request请求分配到分布式的uWSGI服务器上

# Apache

apache是国际上最通用的server服务器
---
title: python WSGI框架
tags:
  - Python
comments: true
toc: true
mathjax: true
abbrlink: 358532ce
date: 2018-11-18 00:21:52
urlname:
categories:
thumbnail:
---
[toc]
#wsgi

----------
python WSGI框架详解
1.web应用的本质
1)浏览器发送一个HTTP请求
2)服务器收到请求，生成一个HTML文档
3)服务器把HTML文档作为HTTP响应的body发个浏览器
4)浏览器收到HTTP响应，从HTTP Body取出HTML文档并显示

2.什么是WSGI,为什么需要WSGI
上面的web应用过程，如果由我们自己来实现是复杂的，接收HTTP请求，解析HTTP请求，响应HTTP请求等。
通常这些操作都由WSGI服务器来完成，WSGI（Web Server Gateway Interface）定义了WSGI服务器执行的接口，我们只需要编写服务WSGI规范的接口，然后由WSGI服务器来执行，就可以了。

3.WSGI接口编写示例

def application(environ,satrt_response):
    start_response('200 OK',[('Content-Type','text/html')])
    return '<h1>Hello,web!</h1>'
上面的application()函数就是符合WSGI标准的一个HTTP处理函数
参数说明：
environ：包含HTTP请求信息的dict对象
satrt_response:发送HTTP响应的函数
函数说明：
start_response('200 OK', [('Content-Type', 'text/html')])
发送HTTP响应的Header，Header只能发送一次，意思是start_response函数只能执行一次
'200 OK'是HTTP响应码参数，[('Content-Type', 'text/html')]表示HTTP Header

函数的返回值return '<h1>Hello,web!</h1>'作为HTTP响应body发送给服务器。

接收HTTP请求，解析HTTP请求，发送HTTP请求等操作就交由WSGI服务器去完成，WSGI接口只负责业务逻辑。

4.python WSGI服务器
python内置了一个WSGI服务器，这个模块叫做wsgiref，不过这个模块没有考虑运行效率，只是为了开发和测试使用。

5.python编写运行WSGI web应用示例
1)编写WSGI接口

def application(environ,satrt_response):
    start_response('200 OK',[('Content-Type','text/html')])
    return '<h1>Hello,web!</h1>'
2)编写server.py，启动初始化WSGI服务器，加载application()函数

复制代码
# server.py
# 从wsgiref模块导入:
from wsgiref.simple_server import make_server
# 导入我们自己编写的application函数:
from hello import application

# 创建一个服务器，IP地址为空，端口是8000，处理函数是application:
httpd = make_server('', 8000, application)
print "Serving HTTP on port 8000..."
# 开始监听HTTP请求:
httpd.serve_forever()
复制代码
运行：python server.py
打开浏览器，输入http://localhost:8000/，就可以看到结果了。

----------

----------


wsgi全称Web Server Gateway Interface，是一个规范，定义web服务器如何与python应用程序进行交互。

wsgi的目的有两个：
1、让Web服务器知道如何调用Python应用程序，并且把用户的请求告诉应用程序。
2、让Python应用程序知道用户的具体请求是什么，以及如何返回结果给Web服务器。

在WSGI中定义了两个角色，Web服务器端称为server或者gateway，应用程序端称为application或者framework。

server端会先收到用户的请求，然后会根据规范的要求调用application端，如下所示：

**SERVER/GATEWAY----------------APPLICATION/FRAMEWORK**

调用的结果会被封装成HTTP响应后再发送给客户端。

#server如何调用application
首先，每个application的入口只有一个，也就是所有的客户端请求都同一个入口进入到应用程序。


> 这个application就是root


接下来，server端需要知道去哪里找application的入口。这个需要在server端指定一个Python模块，也就是Python应用中的一个文件，并且这个模块中需要包含一个名称为application的可调用对象（函数和类都可以），这个application对象就是这个应用程序的唯一入口了。WSGI还定义了application对象的形式：

```
def simple_app(environ, start_response):
      pass
```
所有支持WSGI的python框架都有一个application的对象。框架的使用者不需要关心application是怎么工作的，只需要关心路由定义、请求处理等具体的业务逻辑。

#application对象需要做什么
当server调用application了之后，application就开始处理请求了，请求处理之后，application需要返回处理结果给server。**处理请求**和**返回结果**这两个动作都和environ、start_response参数有关。

##environ参数
environ参数是一个Python的字典，里面存放了所有和客户端相关的信息，这样application对象就能知道客户端请求的资源是什么，请求中带了什么数据等。environ字典包含了一些CGI规范要求的数据，以及WSGI规范新增的数据，还可能包含一些操作系统的环境变量以及Web服务器相关的环境变量。我们来看一些environ中常用的成员：

首先是CGI规范中要求的变量：

```
REQUEST_METHOD： 请求方法，是个字符串，'GET', 'POST'等
SCRIPT_NAME： HTTP请求的path中的用于查找到application对象的部分，比如Web服务器可以根据path的一部分来决定请求由哪个virtual host处理
PATH_INFO： HTTP请求的path中剩余的部分，也就是application要处理的部分
QUERY_STRING： HTTP请求中的查询字符串，URL中?后面的内容
CONTENT_TYPE： HTTP headers中的content-type内容
CONTENT_LENGTH： HTTP headers中的content-length内容
SERVER_NAME和SERVER_PORT： 服务器名和端口，这两个值和前面的SCRIPT_NAME, PATH_INFO拼起来可以得到完整的URL路径
SERVER_PROTOCOL： HTTP协议版本，HTTP/1.0或者HTTP/1.1
HTTP_： 和HTTP请求中的headers对应。
WSGI规范中还要求environ包含下列成员：

wsgi.version：表示WSGI版本，一个元组(1, 0)，表示版本1.0
wsgi.url_scheme：http或者https
wsgi.input：一个类文件的输入流，application可以通过这个获取HTTP request body
wsgi.errors：一个输出流，当应用程序出错时，可以将错误信息写入这里
wsgi.multithread：当application对象可能被多个线程同时调用时，这个值需要为True
wsgi.multiprocess：当application对象可能被多个进程同时调用时，这个值需要为True
wsgi.run_once：当server期望application对象在进程的生命周期内只被调用一次时，该值为True
```

上面列出的这些内容已经包括了客户端请求的所有数据，足够application对象处理客户端请求了。

## start_resposne参数
start_response是一个可调用对象，接收两个必选参数和一个可选参数：

```
status: 一个字符串，表示HTTP响应状态字符串
response_headers: 一个列表，包含有如下形式的元组：(header_name, header_value)，用来表示HTTP响应的headers
exc_info（可选）: 用于出错时，server需要返回给浏览器的信息
```

当application对象根据environ参数的内容执行完业务逻辑后，就需要返回结果给server端。我们知道HTTP的响应需要包含status，headers和body，所以在application对象将body作为返回值return之前，需要先调用start_response()，将status和headers的内容返回给server，这同时也是告诉server，application对象要开始返回body了。

#application对象的返回值

application对象的返回值用于为HTTP响应提供body，如果没有body，那么可以返回None。如果有body的化，那么需要返回一个可迭代的对象。server端通过遍历这个可迭代对象可以获得body的全部内容。

```
def simple_app(environ, start_response):
      status = '200 OK'
      response_headers = [('Content-type', 'text/plain')]
      start_response(status, response_headers)
      return ['hello, world']
```

app.py 一般包含了Pecan应用的入口，包含应用初始化代码
config.py 包含Pecan的应用配置，会被app.py使用
controllers/ 这个目录会包含所有的控制器，也就是API具体逻辑的地方
controllers/root.py 这个包含根路径对应的控制器
controllers/v1/ 这个目录对应v1版本的API的控制器。如果有多个版本的API，你一般能看到v2等目录。



---

title: mariadb数据库里mysql.sock的作用
date: 2022-11-27 12:13
tags:
  -  数据库

categories:
  - 方法：方法论（经验谈）

comments: true
toc: true

---
有时候在服务器上用mysql指令启动会报缺少mysql.sock的错误，一般有两种情况：
+. 一种是把相关的依赖包全都安装上

```
yum search mariadb
yum install xxx xxx -y
```
+. 另一种是mysql客户端client和server端的连接方式
一般在服务器上安装mariadb是把客户端和服务端全都安装上。使用mysql指令时，是使用localhost的命令登录的，即
`mysql -uroot -p 等于 mysql -uroot -p -h loacalhost`
 而mariadb的client端和server端是有两种建立连接的方式，一种是通过tcp/ip的协议，比如远程navicat登录什么的；另一种是通过sock连接，需要mysql.sock的文件。
 在服务器上的client和server连接就是使用sock连接，如果没有找到sock文件就会登陆不上。一般来说重启mariadb服务可以自动生成sock文件，sock文件一般放在/var/lib/mysql/mysql.sock。
 如果不是放在这里，可以`ln -s xxx.sock xxx`软连接起来，或者使用`mysql -uroot -p -S xx.sock`指定sock文件。
 如果没有生成sock文件，`find / -name mysql.sock`找不到文件，可以强制使用tcp/ip协议连接，这时候虽然是本地localhost登录，但是要强制使用127.0.0.1登录`mysql -uroot -p -h 127.0.0.1`。
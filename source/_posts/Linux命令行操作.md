---
title: Linux命令行操作
tags:
  - Linux
comments: true
toc: true
mathjax: true
abbrlink: 4916d74f
date: 2017-11-18 00:21:52
urlname:
categories:
thumbnail:
---

### 目录操作
**创建目录**

```
mkdir
```
**切换目录**

```
cd ..
cd
```
**移动目录**

```
mv /home/test /var/tmp
```
**删除目录**

```
rm -rf var/test
```
**查看目录**

```
ls
```
##文件操作

**创建文件**

```
touch ~/testfile
```
**删除文件**

```
rm ~/testfile
```
**复制文件**

```
cp ~/testfile ~/testnewfile
```
**查看文件内容**

```
cat ~/.bash_history
```
##过滤、管道与重定向
**过滤文件内内容**

```
grep 'root' /etc/pwd
```
**过滤文件夹内内容**

```
grep -r 'linux' /var/log
```
**管道**
管道的作用是将上一个命令的输出和下一个命令的输入连接起来

```
cat ~/.bash_history | grep 'ls' 
```
**重定向**
使用>或者<将内容重定向文件中 

```shell
 echo 'hello,world!'> ~/test.txt
```

### 运维常用命令

**ping**
四个ping包

```
ping -c 4 www.baidu.com
```
**网络状态**

```
netstat -lt
netstat -tulpn
```
**过滤得到当前系统ssh信息**

```
ps -aux | grep 'ssh'
```

```

----------

anaconda切换环境
conda info -e
activate python2

----------

```

### Linux离线安装apt包

首先需要了解的是在网络正常情况下使用apt -d install 安装，会把包下载在*/var/cache/apt/archives/*下。

因此整体思路就是在正常的安装，把下载的包和依赖包拷贝到离线机器上，构建包索引，执行安装

如果正常机器上已经安装需要的包了，那么使用aptitude工具卸载安装的包

（0）aptitude可以卸载包及依赖的包

`sudo aptitude remove`

（1）清理apt的下载缓存区。

```html
sudo rm -rf /var/cache/apt/archives/*
```

（2）下载所需要的组件

```html
sudo apt-get -d install <包名>
```

（3）创建一个目录，将下载的包拷贝到该目录下

```html
cp -r /var/cache/apt/archives  /yout-path
```

（4）修改目录权限

```html
 chmod 777 -R /your-path
```

（5） 建立deb包的依赖关系

```html
sudo touch /your-path/Packages.gz



sudo dpkg-scanpackages /your-path/ /dev/null  | gzip > /your-path/Packages.gz
```

（6）将所有下载的文件和生成的gz文件拷贝到离线的ubuntu机器上，将 /etc/apt/sources.list原有内容注释掉，新增：

```html
deb file:/var debs/
```

（7）执行 sudo apt-get update，之后就可以直接使用apt-get install 包名 来安装了

### sudo -E

简单来说，就是加上`-E`选项后，用户可以在sudo执行时保留当前用户已存在的环境变量，不会被sudo重置，另外，如果用户对于指定的环境变量没有权限，则会报错。

### 文件大小

`df -h`查看系统中文件的使用情况

`du -sh *`查看当前目录下各个文件及目录占用空间大小

可以排序查看 `du -sh \root\* | sort -nr`
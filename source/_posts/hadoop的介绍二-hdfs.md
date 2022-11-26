---
title: hadoop的介绍二-hdfs
tags:
  - Hadoop
comments: true
toc: true
mathjax: true
abbrlink: b042e9aa
date: 2019-10-25 04:23:11
urlname:
categories:
thumbnail:
---

hdfs是大概工作流程是把一个大文件分割成若干个确定大小的数据块，每个数据库会进行备份分布在不同的主机上，当有某个数据块损坏时会自动从别的主机那里拷贝新的数据块。如果一个文件的大小小于这个设定的数据块，则按照一个数据块的空间进行存储。

##hdfs的架构

hdfs是一个master/slave的架构，一个master带多个slaves。一个hdfs包含一个NameNode（master）和多个DataNode（slaves）。NameNode管理文件的namespace和通过客户端访问文件系统的文件。DataNode管理文件的具体存储，每个节点部署一个。

NameNode执行打开一个文件，关闭，重命名等，负责管理文件的数据块到底存储在哪一个DataNode上面。**基于文件的操作**
DataNode执行数据块的读写操作。**基于数据块的操作**

hdfs有两个环境要求：
1. java环境，java的版本应该在1.7以上
2. 必须安装ssh，hadoop里各个主机的通信应该是基于ssh进行的


hadoop伪分布式环境部署手册
`https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/SingleCluster.html`

hadoop生态包安装地址
`http://archive.cloudera.com/cdh5/cdh/5/`

hdfs 运行后相当于一个在服务器上的云盘，可以在浏览器上查看文件和数据块的信息，可以传文件，删文件，读取，创建文件夹，移动，拷贝等等，可以使用shell指令，也可以用java api在java 程序里使用。
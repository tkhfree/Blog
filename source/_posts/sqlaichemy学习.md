---
title: sqlaichemy学习
tags:
  - Python
comments: true
toc: true
mathjax: true
abbrlink: a1ec0fd2
date: 2019-09-18 00:21:52
urlname:
categories:
thumbnail:
---

数据库操作
采用mysql数据库

登录数据库
mysql ‐uroot ‐p

创建数据库
create database name

显示数据库
show databases

删除数据库
drop database name

进入数据库
use databasename

sqlalchemy
object-relation mapping（ORM）技术，把关系数据库的表结构映射到对象上
安装mysql数据库、mysql-python驱动、sqlalchemy包

自增长
auto_increment

类型
Type

非空
not null

默认值
default ‘xx’

唯一
unique

指定字符集
charset

主键
primary key

外键

增加两个表之间的联系

导入sqlalchemy包
from
from
from
from

sqlalchemy import create_engine
sqlalchemy.ext.declarative import declarative_base
sqlalchemy import Column, String, Integer
sqlalchemy.orm import sessionmaker

与数据库连接
from sqlalchemy import create_engine
engine = create_engine('mysql+mysqldb://root:1221@localhost:3306/pytest')
数据库类型+驱动：//数据库账号：密码@ip：端口/数据库名字

创建基类
from sqlalchemy.ext.declarative import declarative_base
Base = declarative_base()

描述表结构
创建一个类User1，继承基类

class User1(base):
__tablename__:'username'
id = Colume(Integer,primary_key = True)
name = Colume(String(30),nullable = False,index=True)
password = colume(Integer)
def __repr__(self):
return '%s(%r)' %(self.__class__.__name__,self.username)
tabelename指定表的名字，除了String、Integer类型还有Text，Boolean，DateTime等。nullable=False 代表这一
列不可以为空，index=True 表示在该列创建索引。定义 repr 是为了方便调试，你可以不定义，也可以定义的更详细一
些

创建实例（建立表）
Base.metadata.create_all(engine)

建立会话

om sqlalchemy.orm import sessionmaker
Session = sessionmaker(bind=engine)
session = Session()
可以把 sessionmaker 想象成一个手机，engine 当做 MySQL 的号码，拨通这个“号码”我们就创建了一个 Session 类，
下面就可以通过这个类的实例与 MySQL 愉快的交谈了！

简单 CURD
Create
ssion.add_all([User1(name = 'ed',password = '1111111'),
User1(name = 'ad',password = '1111112'),
])
session.commit()

Retrieve
for instance in session.query(tang).order_by(tang.name):
print(instance.name, instance.age)
for name, fullname in session.query(tang.name, tang.age):
print(name, fullname)
print(session.query(tang).get(5))
for t1, in session.query(tang.name).filter(tang.age == 24):
print t1
session.query(class).get(index)
或者
session.query(class.colume)
session.query(class).all()
session.query(class.colume).filter(条件)
#相等
query.filter(User.name == 'ed')
#不相等
query.filter(User.name != 'ed')
#like
query.filter(User.name.like('%ed%'))
#in
query.filter(User.name.in_(['ed', 'wendy', 'jack']))
query.filter(User.name.in_(
session.query(User.name).filter(User.name.like('%ed%'))
))
#not in
query.filter(~User.name.in_(['ed', 'wendy', 'jack']))
#IS NULL
query.filter(User.name == None)
query.filter(User.name.is_(None))
#IS NOT NUKK
query.filter(User.name != None)
query.filter(User.name.isnot(None))
#And
from sqlalchemy import and_
query.filter(and_(User.name == 'ed', User.fullname == 'Ed Jones'))
query.filter(User.name == 'ed', User.fullname == 'Ed Jones')
query.filter(User.name == 'ed').filter(User.fullname == 'Ed Jones')

#OR
from sqlalchemy import or_
query.filter(or_(User.name == 'ed', User.name == 'wendy'))
#Match
query.filter(User.name.match('wendy'))

update
a = session.query(User1).get(1)
a.password=88888
session.add(a)
session.commit()

delete
a = session.query(User1).get(1)
session.delete(a)
session.commit()


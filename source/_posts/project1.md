---
title: 进存销管理开发1
tag:
  - java
comments: true
toc: true
mathjax: true
abbrlink: 7ff365b2
date: 2019-10-19 16:52:00
urlname:
categories:
thumbnail:
---
项目开发首要是先做好规划，因为之前是用python开发的aistation系统，学的Java都已经忘完了，因此这次计划采用Java结合mariadb数据库进行开发。具体在开发过程中遇到的问题在后续一点点记录和解决吧。
1. 要开始进行开发首先一定要把业务流程搞清楚，知道要做什么，取得什么结果，之前aistation项目做业务流程做了两三个月，一直在等人员到位，项目经费审批之类的，结果把业务搞清楚了之后开发就很快。

```shell
st=>start: 开始
op1=>operation: 登陆成功
op2=>operation: 登陆失败
cond=>condition: 登录验证
op3=>opperation: 商品管理
sub32=>subroutine: 库存管理（存）
sub33=>subroutine: 销售管理（销）
sub31=>subroutine: 进货管理（进）
op4=>opperation: 客户管理
op5=>opperation: 供应商管理
op6=>opperation: 系统维护
sub61=>subroutine: 数据库备份和恢复
sub62=>subroutine: 修改密码
sub63=>subroutine: 退出系统
op7=>opperation: 发送告警通知
op8=>opperation: 关于
e=>end: 结束

st->cond(no,left)->op2
cond(yes,bottom)->op1(bottom)->op3(bottom)->sub31(bottom)->sub32(bottom)->sub33
sub33(bottom)->op4(bottom)->op5(bottom)->op6(bottom)->sub61
sub61(bottom)->sub62(bottom)->sub63
sub63(bottom)->op7(bottom)->op8(bottom)->e
```

（如果上面流程图不能正常显示，则如下：）
{% asset_img yewu.jpg 业务流程图 %}
![业务流程图.png](https://i.loli.net/2019/10/20/dGm8UaIqirJCDfY.png)

ps:为了画这个流程图忙活一下午，一开始是想用hexo的一个flowchart插件，但是这个插件用不了parallel框，而且排版有问题，不能实时预览，也没有报错信息，总之很不方便，还是用visio画了一下。


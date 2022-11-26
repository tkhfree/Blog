---
title: python脚本
tags:
  - Python
comments: true
toc: true
mathjax: true
abbrlink: d3dca0a6
date: 2019-09-18 00:21:52
urlname:
categories:
thumbnail:
---
# 使用python自动化打开音乐下载网站自动下载音乐

使用的是python中色selenium包，selenium2.0接入了webdriver的api

webdriver需要先安装相应浏览器的驱动放到浏览器和python的安装位置（chrome、safari、ie、fox）


```
#coding=utf-8
from selenium import webdriver
from selenium.webdriver.common.keys import Keys
import time

chrome_obj = webdriver.Chrome()
#chrome_obj.maximize_window()

chrometitle = chrome_obj.title
print(chrometitle)

chrome_obj.get("https://music.sounm.com/yinyue/")
chrome_obj.find_element_by_xpath('//div[@class="btn-box"]').click()
chrome_obj.find_element('id','search-wd').send_keys('不要说话')
chrome_obj.find_element_by_xpath('//input[@value="netease"]').send_keys(Keys.SPACE)
chrome_obj.find_element_by_xpath('//button[@class="search-submit"]').submit()
time.sleep(100)
chrome_obj.close()
```

其中使用两种寻找方式，find_element_by_id根据id查找，还是根据相对路径find_element_by_xpath；导入keys包，发送space按键；导入time包，停止100秒


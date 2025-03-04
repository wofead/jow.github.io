---
layout:     post
title:      Python的100天学习
subtitle:   1
date:       2019-6-8
author:     Jow
header-img: img/timg.jpg
catalog: 	 true 
tags:
    - Python
    - study

---

### 目录
1. 开发环境的配置
2. 请求库安装
3. 解析库安装
4. 数据库的安装
5. Web库的安装
6. App爬取相关库的安装
7. 爬虫框架的安装
8. 部署相关库的安装


> 爬虫，很久之前自己就行学习了，但是自己一直没有去认真的准备和一步一个脚印的留下些学习的痕迹，在五一放假的第一天，自己决定好好的静下心来进行学习


## 开发环境的配置

首先当然是Python的安装，直接从官网上下载安装就行了，接下里就是请求库的安装了，要想能够快速的得到自己想要的东西，工具要好，而这些请求库就是我们的好工具。

爬虫可以简单的分为几步：抓取界面、分析界面和储存数据。

## 请求库安装 ##

1. requests库的安装 pip install requests    -->常用的用于http请求的模块
2. Selenium的安装 pip install selenium    -->自动化测试测试工具，利用它我们可以驱动浏览器执行特定的动作，如点击、下拉等操作，针对js渲染的页面来说，这种抓取非常的有效。
3. ChromeDriver的安装   -->用它来驱动Google浏览器
4. PhantomJS的安装   -->这个是一个无界面的、可脚本编程的Webkit浏览器引擎，他原生支持多种web标准:DOM操作、css选择器、json、canvas以及svg。
5. aiohttp的安装  -->requests是一个阻塞式的http请求库，当我们发送请求的时候程序会一直等待服务器响应，才会进行下一步操作。这个时间是非常的长的，如果程序可以在这段时间做一些请求的调度响应的处理，那么爬取效率会非常的高。aiohttp就是这样一个提供异步Web服务的库。
6. cchardnet以及aiodns  --> 字符编码检测库和加速DNS解析的库。

在使用PhantomJS的时候，发现Python已经对其不在支持，并且推荐使用Google的headless模式

```python
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
chrome_options = Options()
chrome_options.add_argument("--headless")
browser = webdriver.Chrome(options=chrome_options)
browser.get("https://www.jianshu.com")
print(browser.current_url)
```

## 解析库安装 ##

1. lxml的安装  --> 支持html和xml解析，支持xpath的解析方式。
2. beautifulsoup4 
3. pyquery
4. tesserocr的安装   -->图形验证码的识别，tesserocr是Python的一个ocr识别库，是对tesseract的封装，所以需要首先安装tesseract。

## 数据库的安装 ##

1. mysql的安装
2. PyMysql的安装  -->使Python和数据库进行交互
3. Redis安装

## Web库的安装 ##

存在一些这样的web服务器，比如flask、django等,我们可以拿他来开发网站和接口等。在这里我们可以使用它们搭建一些API接口供爬虫使用。

1. Flask的安装

## App爬取相关库的安装 ##

1. charles的安装   -->通过charles来抓取手机的包，小米原来的浏览器安装不了证书，需要换个浏览器。
2. mitmproxy的安装
3. Appium的安装

## 爬虫框架的安装 ##

1. pyspider的安装
2. Scrapy的安装
3. Scrapy-Splash的安装
4. Scrapy-Redis的安装

## 部署相关库的安装

1. docker的安装
2. scrapyd的安装
3. scrapyd-client的安装
4. scrapyd api的安装
5. scrapyrt的安装
6. gerapy的安装


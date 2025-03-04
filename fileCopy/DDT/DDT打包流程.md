---
layout:     post
title:      DDT 打包流程
subtitle:   ddt客户端学习
date:       2019-3-14
author:     Jow
header-img: img/post-bg-2015.jpg
catalog: 	 true 
tags:
    - DDT
    - Package

---

### 目录
1. 版本的管理
2. 地区的管理
3. 包名的配置
4. 更新包的配置
5. 资源压缩配置
6. Jenkins
7. 包体的放置

> Tools目录下的配置文件都是服务器打包配置器的的，即replaceParamsTools下的main.bat        
> Time has gone, but people are still there.

##版本的管理
配置目录地址：Publish/apk/config/release  和 test1

**版本的管理分为底包版本和GCloud版本**

其中底包版本是应用商店需要的版本在 。。。 上管理

GCloud版本管理是为了方便我们自己管理

GCloud分为底包和资源部分

资源的的前两个参数和底包相同，资源的第三个参数在Jenkins的svn_revision来配置

底包的版本第三位默认为10，后面一位（第四位）每打包一次加一不管这个包成功or失败

当提审通过的时候，下次再打包第二个参数加一后面的第四位参数如下

isShowChangeIPView：是否展示选服界面

isPassSdk：是否需要跳过SDK的打包

**注意release是正式发布的配置 而 test1是测试版本 正式发布的配置要仔细检查**

##地区的管理

配置目录地址：LuaScript/region/RegonConfig 和 ZY_INTERNATIONAL目录

添加新的地区在Configs.AREA和Configs中添加配置

ZY_INTERNATIONAL目录下的文件是针对于当前地区进行的定制，它们需要继承上级目录的父类文件，然后进行重写

也可以使用Develop\Tools\replaceParamsTools\config.cfg 来生成拷贝

工具是main客户端Develop\Tools\replaceParamsTools下的main.bat  

正常情况下配置好主干和分支先从主干修改，然后合并到分支，当然也有一些操作直接操作分支

## 包名的配置

文件目录：Client\Pack\packConfig

name and cnname 这两个配置只是起着标志的作用，根据版本特性来命名即可

## 更新包的配置

文件目录：Client\Publish\updateSave\update_config.text

配置版本的前两位 + 更新到那个位置的svn版本号

##  资源压缩配置（已经修改）

目录地址：Client\Compress\Android_new_lz4\Compressed\rescompressed.cof

IOS一样的修改

这个配置是固定的 即第一次切分支时的svn版本号

## Jenkins

在Jenkins新建 参照 从切分支的那个项目进行配置

**有几个地方需要修改**

1. appId
2. packageID
3. doPackageUrl
4. doTestPackageUrl

## 包体的放置

需要共享空间上防止包体 \\10.10.1.20\10.10.4.60\手游弹弹堂\Packages\海外共享\

目录的创建仿照\\10.10.1.20\10.10.4.60\手游弹弹堂\Packages\海外共享\港台Android\正式\google\1.5.10.15_143955




---
layout:     post
title:      再谈https
subtitle:   https
date:       2019-10-28
author:     Jow
header-img: img/about-bg-walle.jpg
catalog: 	 true 
tags:
    - network

---

### 目录
1. 数字证书
2. SSL协议的握手的过程
3. session的恢复


> 家里的https放置了许久自己一直也没有看完，现在开始继续的学习，先从整体开始学习，然后再开始学习细节。


## 数字证书

每个https网址都存在一份数字证书，那什么是数字证书呢？

数字证书就是一个电子文档，其中包含了持有者的信息、公钥以及证明该证书有效的数字签名。一般来说，数字证书是由数字证书认证机构(Certificate authority,CA)来负责签发和管理的，并承担PKI体系中公钥的合法性的检测责任；数字证书的类型有很多，而https使用的ssl证书。

那怎么验证证书由CA签发的而不是伪造的呢？

数字正式的签发机构CA，在接收到申请者资料后进行核对并确认信息的真实有效，然后就会制作一份符合X.509标准文件。证书中的证书内容包含了持有者信息和公钥等都是由申请者提供的，而数字签名则是CA机构对证书内容进行hash加密后得到的，这个数字签名就是我们验证证书是否是可信CA签发的数据。

接收端接到一份数字证书Cer1后，对证书的内容做Hash等到H1；然后在签发该证书的机构CA1的数字证书中找到公钥，对证书上数字签名进行解密，得到证书Cer1签名的Hash摘要H2；对比H1和H2，假如相等，则表示证书没有被篡改。但这个时候还是不知道CA是否是合法的，我们看到上图中有CA机构的数字证书，这个证书是公开的，所有人都可以获取到。而这个证书中的数字签名是上一级生成的，所以可以这样一直递归验证下去，直到根CA。根CA是自验证的，即他的数字签名是由自己的私钥来生成的。合法的根CA会被浏览器和操作系统加入到权威信任CA列表中，这样就完成了最终的验证。所以，一定要保护好自己环境（浏览器/操作系统）中根CA信任列表，信任了根CA就表示信任所有根CA下所有子级CA所签发的证书，不要随便添加根CA证书。

## SSl协议的握手过程

开始加密通信之前客户端和服务器首先必须建立连接和交换参数，这个过程叫做握手（handshake）

握手阶段分成五部：
1. 客户端给出协议版本号、一个客户端生成的随机数（Client Random),以及客户端支持的加密方法。
2. 服务器确认双方使用的加密方式，并给出数字证书和一个服务器生成的随机数。
3. 客户端确认数字证书有效，然后生成一个新的随机数，并使用数字证书中的公钥，加密这个随机数，发给服务器。
4. 服务器使用自己的私钥，获取客户端发来的随机数。
5. 客户端和服务器根据约定的加密方法，使用前面的三个随机数，生成“对话秘钥”（session key），用来加密接下来整个对话过程。

## 私钥的作用

1. 生成对话秘钥一共需要三个随机数。
2. 握手之后的对话使用“对称秘钥”加密，服务器的公钥和私钥只用于加密和解密“对话秘钥”，无其他作用。
3. 服务器公钥放在服务器的数字证书之中。

## session的恢复

握手阶段用来建立SSL连接。如果出于某种原因对话中断，就需要重新握手。这时有两种方法可以恢复原来你的session，一种叫做session id，另一种叫做session ticket。

session ID的思想很简单，就是每一次对话都有一个编号（session ID）。如果对话中断，下次重连的时候，只要客户端给出这个编号，且服务器有这个编号的记录，双方就可以重新使用已有的"对话密钥"，而不必重新生成一把。

客户端不再发送session ID，而是发送一个服务器在上一次对话中发送过来的session ticket。这个session ticket是加密的，只有服务器才能解密，其中包括本次对话的主要信息，比如对话密钥和加密方法。当服务器收到session ticket以后，解密后就不必重新生成对话密钥了。


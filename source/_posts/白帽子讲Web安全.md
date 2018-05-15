---
title: 白帽子讲Web安全Note
date: 2018-05-10 19:58:53
updated:
tags: [信息安全]
categories: [专业]
---


计划看好久了，一直没看，现在终于简单过了一遍。

<!--more-->

# 我的安全世界观
## 威胁分析
* STRIDE模型

|               威胁               |       定义       | 对应的安全属性 |
| -------------------------------- | ---------------- | -------------- |
|          Spoofing(伪装)          |   冒充他人身份   |      认证      |
|         Tampering(篡改)          |  修改数据或代码  |     完整性     |
|        Repudiation(抵赖)         |  否认做过的事情  |   不可抵赖性   |
| InformationDisclosure(信息泄露)  |   机密信息泄露   |     机密性     |
|   Denial of Service(拒绝服务)    |     拒绝服务     |     可用性     |
| Elevation of Privilege(提升权限) | 未经授权获得许可 |      授权      |

## 风险分析
* DREAD模型

windows Vista的UAC功能，询问用户是否允许该行为，这里说如果用户能分辨什么样的行为是安全的，那么还要安全软件做什么。这里说的很对，不过也是存在一种解释，这里的提醒只是一种用户个人对行为承担责任的方式吧，已经予以提醒，之后的操作的安全保障已经不是操作系统层面的安全性能够提供了。

# 客户端安全

* <script>\<img>\<iframe>\<link>等标签都可以跨域加载资源，不同于XMLHttpRequest，通过Src属性加载的资源，JS权限被浏览器限制了，不能读写返回的内容，XMLHttpRequest可以读写同源，访问跨域需要遵循标准，通过目标域返回的HTTP头来授权

EVSSL相比普通的SSL,在浏览器里会有绿色高亮

# 跨站脚本攻击(XSS)
* 反射型XSS
* 存储型XSS
* Dom Based XSS

除了User Agent以外，还可以通过一些浏览器限定的操作来判断实际浏览器

XSS也是有一些攻击框架，如Attack API、BeEF等等

XSS Worm 蠕虫属于终极XSS

* 可以通过location.hash来发送XSS payload
* base标签
* window.name
* Anehta回旋镖

XSS攻击主要发生在MVC的View层，模板引擎支持htmlEncode来对抗XSS

Anti-Samy 最好的XSS filter

# CSRF

这个问题在之前写前后端分离部署的时候倒是遇到过……然后为了能跑起来，似乎对安全性完全没有考虑过

P3P头和Firefox等浏览器是支持发送第三方Cookie的

Anti CSRF Token

# ClickJacking

唔，很多盗版内容网站通过这个来获取广告点击

## XSIO图片覆盖攻击

## fram busting
防止跨域iframe攻击

## X-Frame-Options

# HTML5安全
## sandbox

# 注入攻击

使用类似xp_cmdshell等存储过程获取webshell

# 文件上传漏洞

避免方法:
1. 文件上传的目录设置为不可执行
2. 判断文件类型
3. 使用随机数改写文件名和文件路径
4. 单独设置文件服务器的域名

# 认证与会话管理

# 访问控制

## 垂直权限管理(RBAC)
## 水平权限管理

## OAuth

# 加密算法与随机数

## ECB(Electronic Codebook)模式

## 伪随机数问题

如果伪随机数的空间太小，可能被预测到，那的确很危险

## 小结
1. 不要使用ECB模式
2. 不要使用流密码(比如RC4)
3. 使用HMAC-SHA1代替MD5(甚至是代替SHA1)
4. 不要使用相同的key做不同的事情
5. salts与IV需要随机产生
6. 不要自己实现加密算法，尽量使用安全专家已经实现好的库
7. 不要依赖系统的保密性

当你不知道该如何选择时
1. 使用CBC模式的AES256用于加密
2. 使用HMAC-SHA512用于完整性检查
3. 使用带salt的SHA-256或SHA-512用于Hashing

# Web框架安全
利用好如django自身提供的安全解决方案，也仔细想想框架自身是否存在漏洞。

# 应用层拒绝服务攻击

即对消耗较大的应用页面不断发起正常的请求，如select * from xxx这种数据库查询操作阻塞数据库，或者占用服务端的最大连接数，亦或是通过https申明一个较大的content-length,小字节包慢慢发，占用资源。

* 业务前进行一个判断访问频次异常
* 验证码
* Yahoo设计的算法(Detecting system abuse)
	* 根据IP地址和cookie，计算客户端的请求频率并进行拦截(好眼熟，好像吴军老师的《浪潮之巅》里也提到过这个样例)

# WebServer配置安全

# 互联网业务安全

# 安全开发流程（SDL)





# Reference
1. [《白帽子讲Web安全》]
2. [URL的井号](http://www.ruanyifeng.com/blog/2011/03/url_hash.html)

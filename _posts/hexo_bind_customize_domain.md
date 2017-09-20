---
title: Hexo 绑定自定义域名
date: 2016-08-03 10:25:36
tags: hexo
categories: 个人博客
toc: true
---
搭建好属于自己的博客并托管到之后GitHub会分配一个二级域名`name.github.io`，当然也可以绑定自定义的域名。

<!--more-->

<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="http://music.163.com/outchain/player?type=2&id=33556489&auto=0&height=66"></iframe>
# 1.申请域名
国内申请域名可以在万网或者是腾讯云上等购买，域名并不是想象中的那么贵，我买的域名也就5软妹币/年，挺划算的。

# 2.添加CNAME
在站点根目录的的source目录下新建一个名为`CNAME`的文件，里面写上自定义域名，比如我的CNAME是这样的：  
```
junhan.win
```
之后`hexo d`提交到github即可。

# 3.添加域名解析
万网上可以解析域名，dnspod也可以。既然我的域名是在万网上买的，所以就直接在万网上进行解析。
### a.添加两个指向GitHub服务器的A记录，如下图
![添加A记录](http://i2.piimg.com/567571/ceaefa6e7d2a64e8.png)
### b.添加指向github二级域名的CNAME记录，如下图
![添加CNAME记录](http://i2.piimg.com/567571/c39a8896b97d2f29.png)

# 4.使用自定义域名直接访问HEXO博客
http://junhan.win: [Demo](http://junhan.win)

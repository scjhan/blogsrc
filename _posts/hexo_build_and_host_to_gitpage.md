---
title: Hexo搭建博客(一) 使用Hexo搭建博客并托管到Github
date: 2016-07-10 16:11:35
tags: hexo
categories: 个人博客
toc: true
---
# 前言
听光哥讲完怎么用Hexo搭建博客之后就立马捣鼓了起来。一开始还蛮顺利的，一下子就搭建好。到了后面修改主题时，没有前端基础的弱势彻底显现出来。于是今天早上起来就把昨天弄的全删了，查了很多教程，现在重新来一遍！
<!--- more --->

# 准备工作
1. 安装Node.js，我下载的是Node.js的4.4.7版本。
  [Node.js下载链接][1]
2. 安装Git，我下载的是Git的2.9.0版本。
  [Git下载链接][2]

# 安装和配置Hexo
1. 安装Hexo
  随便找个地方，右键`Git Bush Here`，输入下面的指令：
```cpp
npm install hexo -g
```
ps. 翻墙的话速度会快很多。
2. 初始化博客
  新建一个文件夹，用来保存网站，例如我新建的文件夹是`E:\HEXO`，然后在该目录下右键`Git Bush Here`，输入下面的指令：
```cpp
hexo init
```
执行完毕之后会在该目录下生成一些配置文件和模板文件。
3. 生成博客
  继续执行：
```cpp
hexo g
```
然后会生成一个public文件夹，里面存放的就是博客静态文件。
# 本地部署
在`E:\HEXO`目录下执行：
```cpp
hexo server
```
如果看到以下输出，说明本地部署成功。
```cpp
INFO  Hexo is running at http://localhost:4000/. Press Ctrl+C to stop.
```
然后打开浏览器访问[http://localhost:4000][3]，就可以预览到我们的博客。
# 部署到Github
只有部署到Github(或者其他代码托管网站)，我们的博客才能被其他人访问。
1. 注册Github账号，[Github注册][4]
2. 新建一个repository，这里要注意repository命名格式必须是下面这样的：
```cpp
username.github.io
```
例如我的就是`ChenJunhan.github.io`
3. 配置`_config.yml`文件
  在`E:\HEXO`下找到`_config.yml`文件，打开编辑，在最后面找到下面这段代码：
```cpp
# Deployment
## Docs: http://hexo.io/docs/deployment.html
deploy:
  type:
```
替换成下面的代码：
```cpp
# Deployment
## Docs: http://hexo.io/docs/deployment.html
deploy:
  type: git
  repository: https://github.com/用户名/用户名.github.io.git
  branch: master
```
**这里要注意repository、branch字段前加两个空格，冒号后面一个空格**
4. 在开始部署之前，要先将相应的依赖文件先装好，所以先执行：
```cpp
npm install hexo-deployer-git --save
```
5. 正式部署
  这个步骤很简单，只需要两个命令
```cpp
hexo g
hexo d
```
执行完毕之后本地文件就被upload到Github上了，接下来使用`http://用户名.github.io`就可以访问我们的博客。
# 写新博文
执行以下命令会在E:\source\_post下生成一个`new_blog_name.md`文件。
```cpp
hexo new "new_blog_name"
```
这个文件就是我们的博文，可以使用`MarkDown`进行编辑，直接txt编辑也可以。其中开头几段是这篇博文的具体信息，例如本文：
```cpp
---
title: Hexo搭建博客(一) 使用Hexo搭建博客并托管到Github
date: 2016-07-10 15:27:07
tags:
---
```
写完博文记得要执行以下命令：
```cpp
hexo g
hexo d
```
# 其他
在`E:\HEXO`目录下的`_config.yml`文件可以配置博客的一些基础属性，例如博客标题、作者、语言、博客主题等。
```cpp
# Site
title: 
subtitle: 
description:
author: 
language: 
```
Hexo配置博客前期还是挺简单的，通过上面的几个步骤，博客就搭建好了，不过界面有点low。网上有很多教程教怎么修改主题的。有兴趣可以[点击这里][5]。

<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="http://music.163.com/outchain/player?type=2&id=2080139&auto=1&height=66"></iframe>

[1]: https://nodejs.org/en/download/
[2]: http://git-scm.com/
[3]: http://localhost:4000/
[4]: https://github.com/
[5]: https://www.zhihu.com/question/24422335

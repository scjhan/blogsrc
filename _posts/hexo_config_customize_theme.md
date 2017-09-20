---
title: Hexo搭建博客(二) 自定义博客主题
date: 2016-07-10 18:30:59
tags: hexo
categories: 个人博客
toc: true
---
说实话，Hexo默认的landspace主题实在是太low了，于是上网查了下修改主题的教材。没有前端基础，所以搞得乱七八糟的，终于搞定！
Hexo主题直接上Github搜就有了，很多。我用的是`yilia`主题，下面来个预览图。
<!--- more --->
![][1]
***
# Hexo根目录
要配置主题，就要了解下Hexo博客根目录下的基本信息
```
├── _config.yml
├── package.json
├── scaffolds
├── source
|   ├── _drafts
|   └── _posts
└── themes
```
`_config.yml`:网站的 配置 信息，您可以在此配置大部分的参数。
`package.json`:应用程序的信息。
`scaffolds`:模版文件夹。当您新建文章时，Hexo会根据scaffold 来建立文件。
`source`:资源文件夹是存放用户资源的地方。除posts文件夹之外，开头命名为(下划线)的文件/文件夹和隐藏的文件将会被忽略。Markdown和HTML文件会被解析并放到public文件夹，而其他文件会被拷贝过去。
`themes`:主题 文件夹。Hexo会根据主题来生成静态页面。
***
# 下载主题包
下载主题有两个方法
1. 直接从Github上下载，然后放在`E:\HEXO\themes`里面(这里的`E:\HEXO`是上篇讲的本地博客根目录)
例如我使用的`yilia`主题，它的链接：[https://github.com/litten/hexo-theme-yilia][2]。点击下载Zip压缩包到本地，然后放在`E:\HEXO\themes`里面。
2. 直接执行命令下载
```
git clone https://https://github.com/litten/hexo-theme-yilian.git themes/hexo-theme-yilia
```
之后会自动下载`yilia`，并保存在`E:\HEXO\themes\yilia`。
# 启用主题
打开`E:\HEXO\_config.yml`，找到`theme`字段，将`landspace`改为`yilia`。
# 更新
```
cd themes/yilia
git pull
```
# 配置主题
在`themes\yilia`目录下有个`_config.yml`，这是来配置主题用的。
具体语法我也不是很懂，不过看起来理解应该不难。主要用到的有下面这些：
1.菜单和友链
```
menu:
  主页: /
  所有文章: /archives
  随笔: /tags/随笔
```
`menu`是用来创建菜单的，我这里创建了三个菜单。
```
# SubNav
subnav:
  github: "https://github.com/ChenJunhan"
  weibo: "http://weibo.com/2633996855"
```
这个是来编辑友情链接的。
其他的应该也没什么了，我了解的就这么多。
2.头像和图标
`_config.yml`中，有两个字段用来配置头像和网站图标的。
```
# Miscellaneous
google_analytics: ''
favicon: /img/favicon.png

#你的头像url
avatar: /img/avatar.jpg
```
`url`可以使用外链，也可以使用本地链接，本地链接是`E:\HEXO\themes`的相对链接。例如头像我放在`E:\HEXO\themes\img\avatar.png`。
3.其他配置
其他配置例如css，这些很前端。是的，我一点也不会。因为代码块的显示出了点问题，所以我捣鼓了下`E:\HEXO\themes\yilia\source\css\_partial`下的`highlight.styl`，这个是用来配置代码高亮的，其他的看名字应该也可以猜到它的功能。
***
# 为Hexo博文添加标签
写博文的时候，在最上面的`tags`字段声明即可。
之后public下会生成一个`tags`文件夹，和一个`tags.html`文件，这个不用理它。
单个tag
```
---
title: xxx
date: xxx
tags: tag
---
```
多个tags
```
---
title: xxx
date: xxx
tags: 
	- tag1
	- tag2
---
```

  [1]: /image/preview.jpg
  [2]: https://github.com/litten/hexo-theme-yilia

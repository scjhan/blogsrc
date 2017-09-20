---
title: 闲着没事乱折腾
date: 2016-07-30 23:00:06
tags: hexo
categories: 个人博客
toc: true
---
本来今晚是要干活的，有点懒散，索性继续瞎搞Hexo博客配置。  
今天把我的博客主题换成Next了，看来看去还是这个比较简约。之后又胡乱配置了一些东西，例如标签云，背景图片，背景音乐。  
这里记下Next主题官方教程位置[开始使用][1]  

<!--more-->

---

## 添加标签云
1. 先在Hexo站目录下执行
```
hexo new page tags
```
2. 之后生成`/source/tags/tags.md`，打开修改为如下格式，即添加type: "tags"：
```
---
title: tags
date: 2016-07-30 21:23:21
type: "tags"
---
```

## 添加背景图片
打开`\themes\next\source\css\_schemes\Pisces\index.sty`，可以看到第一句是这样的：
```
body { background: #f5f7f9; }
```
把他改成如下形式：
```
body { background:url(/images/background.jpg); }
```
其中`/images/background.jpg`是事先放好的背景图片

## 添加背景音乐
这里使用网易云音乐，很简单。  
![网易云][2]
如上图，点`生成外链播放器`，然后直接把HTML代码贴到markdown博文下就可以了，效果如下：


<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="http://music.163.com/outchain/player?type=2&id=85621&auto=1&height=66"></iframe>




  [1]: http://theme-next.iissnan.com/getting-started.html#install-next-theme
  [2]: http://i1.piimg.com/567571/17fed33033acc95d.png

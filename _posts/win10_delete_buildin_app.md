---
title: Windows1 0 删除内置app
date: 2016-08-06 08:40:36
tags: windows
categories: 系统优化
toc: true
---
Windows10自带了一大堆乱七八糟的app，而且在控制面板的卸载程序找不到这些= =，其实只要两条命令就可以卸载。

## 1.使用Windows PowerShell卸载
- 命令**Get-AppxPackage -AllUsers**获取app列表
先打开PowerShell，在Cortana直接搜就可以，然后右键管理员运行。
<!--more-->
![PowerShell](http://i4.piimg.com/567571/226061d39aba3748.png)
输入命令`Get-AppxPackage -AllUsers`，然后就会输出所有app列表，大致上长的和下面一样：
```
Name                   : Microsoft.BingWeather
Publisher              : CN=Microsoft Corporation, O=Microsoft Corporation, L=Redmond, S=Washington, C=US
Architecture           : X86
ResourceId             :
Version                : 4.7.118.0
PackageFullName        : Microsoft.BingWeather_4.7.118.0_x86__8wekyb3d8bbwe
...
```
- 命令**Remove-AppxPackage PackageFullName**卸载程序
例如卸载上面说的BingWeather，可以执行：
```
Remove-AppxPackage Microsoft.BingWeather_4.7.118.0_x86__8wekyb3d8bbwe
```

## 2.删除安装包
使用PowerShell卸载程序之后，这些内置app的安装包还残留在C盘中，作为一个强迫症患者，不能忍！
这些安装包的路径如下
```
C:\Program Files\WindowsApps
```
选择对应的安装包删除即可。


每博一首歌
<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="http://music.163.com/outchain/player?type=2&id=27759600&auto=0&height=66"></iframe>

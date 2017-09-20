---
title: Ubuntu 系统美化
date: 2016-09-10 11:15:47
tags: linux
categories: 系统优化
toc: true
---
# 主题美化
这里使用的是[Flatabulous][1]主题。
### 1. 安装 [Ubuntu tweak tool][2]
```
sudo add-apt-repository ppa:tualatrix/ppa
sudo apt-get update
sudo apt-get install ubuntu-tweak
```
### 2. 安装`flatabulous-theme`
```
sudo add-apt-repository ppa:noobslab/themes
sudo apt-get update
sudo apt-get install flatabulous-theme
```
<!--more-->
### 3. 安装`ultra-flat-icons`
```
sudo add-apt-repository ppa:noobslab/icons
sudo apt-get update
sudo apt-get install ultra-flat-icons
```
>Alternatively, you could also run `sudo apt-get install ultra-flat-icons-orange` OR `sudo apt-get install ultra-flat-icons-green`.  

### 4. 加载主题
按Super键，搜索`Ubuntu Tweak`，主题选择`Flatabulous`，图标选择`ultra-flat-icons`，重启即可。
![Screenshots][3]

****

# Shell 美化
Shell是Linux/Unix的一个外壳，它负责外界与Linux内核的交互，接收用户或其他应用程序的命令，然后把这些命令转化成内核能理解的语言，传给内核。Linux/Unix提供了很多种Shell，常用的Shell有这么几种，sh、bash、csh等，想知道你的系统有几种shell，可以通过以下命令查看：
```
cat /etc/shells
```
显示如下：
```
/bin/bash
/bin/csh
/bin/ksh
/bin/sh
/bin/tcsh
/bin/zsh
```
### 1. 安装zsh
```
sudo apt-get install zsh
```
安装完成后设置当前用户使用 zsh：
```
chsh -s /bin/zsh
```
根据提示输入当前用户的密码就可以了，如果执行上面命令没效果的话，再执行：`vim ~/.bashrc`，在末尾添加`bash -c zsh`，重启终端即可。
### 2.安装oh-my-zsh
需要先安装Git，然后执行以下指令自动安装oh-my-zsh：
```
wget https://github.com/robbyrussell/oh-my-zsh/raw/master/tools/install.sh -O - | sh
```
### 2. 修改zsh主题
oh my zsh 提供了数十种主题，相关文件在~/.oh-my-zsh/themes目录下。在`~/.zshrc`里找到`ZSH_THEME`，就可以设置主题了。如设置`ZSH_THEME="ys"`。
![ys theme][4]


  [1]: https://github.com/anmoljagetia/Flatabulous
  [2]: https://github.com/tualatrix/ubuntu-tweak
  [3]: https://camo.githubusercontent.com/98ca90f73825cadbb1a8c2c45c8be8840acd9355/687474703a2f2f692e696d6775722e636f6d2f544b5465334d6e2e706e673f31
  [4]: https://cloud.githubusercontent.com/assets/1415488/13198621/f2c1320c-d848-11e5-8f22-7fac1baeec2f.jpg

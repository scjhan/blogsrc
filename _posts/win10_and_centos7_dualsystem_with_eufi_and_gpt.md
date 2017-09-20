---
title: Windows10+CentOS7双系统(UEFI+GPT)
date: 2016-07-26 22:26:42
tags: 
	- windows
	- linux
categories: 系统优化
toc: true
---
# 前言
眼馋双系统有一段时间了，然而之前安装失败格盘的惨痛教训历历在目。这几天闲着没事查阅了好多资料，怂了一个星期，终于决定再来一次尝试。总的来说安装过程还是挺顺利的，但是后期Windows引导的问题折腾了三天。。。  
技术不好，最终搞出个另类的双系统：默认启动Windows10，同时支持Windows Quick Boot；BIOS下切换到CentOS。  
# 设备信息
> PC： Thinkpad E431，Microsoft Windows10 Pro 64Bit (10240)  
> CentOS版本：CentOS-7-x86_64-DVD-1511(这个版本的CentOS支持UEFI)  
<!---more--->

# 前期准备  
### 1. 分配CentOS安装盘符  
直接使用Windows的磁盘管理，用磁盘压缩切一个空间出来就好了。我是切了50G出来。
### 2. 关闭`Windows Quick`  
执行`Win+R`输入`gpedit.msc`，计算机配置->管理模块->关机，双击右边，选择`已禁用`。  ![关闭快速启动][1]
### 3. 关闭`Secure Boot`  
这个要在BIOS下执行。  

# 安装CentOS
### 1. `UltraISO`制作CentOS启动盘。
### 2. 设置CentOS镜像位置
BISO选择U盘启动，接下来应该会看到黑色界面，如下：
![安装CentOS7][2]
将光标移到第一行，然后这里不是直接点`Install CentOS7`，要按Tab键先配置CentOS镜像位置。  
按下Tab之后可以看到一下三行英文：  
```
setparams "Install CentOS 7" Install
    limuze /image/vmlinuz inst.stage2=hd:LABEL=CentOS\x207\x20x86_64 quiet  
    initrdefi /image/pxeboot/initrd.img
```

 这个是用来选择镜像位置的，因为CentOS它不会自动定位到正确的位置。。。所以接下来要先修改上面的内容。    
① 把第二句改成：  
```
   limuze /image/vmlinuz initrd=initrd.img linux dd quiet
```
② 接下来按`Ctrr+x`执行，就可以看到所有盘符和编号了。类似于下面这样：
![修改CentOS位置][3]
可以看到，CentOS镜像的位置（也就是我的U盘）是`sdb4`。记住这个sdb4，然后关掉这个界面重新再来一次。
③ 重新来一次又来到了步骤2的那个图，还是按Tab键，这一次将第二行改成如下形式:
```
limuze /image/vmlinuz inst.stage2=hd:/dev/sdb4 quiet
```
接下来按`Ctrr+x`执行，CentOS就开始安装了。记得要勾选一个桌面（如gnome桌面）。  
### 4. 设置CentOS磁盘分区
进来安装界面之后，选择前面切出来的那个盘。
![此处输入图片的描述][4]
CentOS安装过程中要设置磁盘分区，这个就涉及到Linux的磁盘分区。下面是我的设置情况:
```
/        ：大小30G，设备类型`LAM`，文件系统`ext4`
/boot    ：大小200M，设备类型`标准分区`，文件系统`ext4`
/boot/efi：大小128M，设备类型、文件系统默认值（这个efi分区是放CentOS的uefi文件的，貌似最后也就占10M左右的空间）
/swap    ：大小8G，设备类型`LAM`，文件系统`ext4`（据说swap分区要为物理内存的两倍，不过觉得我8G内存给它8G已经算多了）
/home：  ：剩下的空间都给它，设备类型`LAM`，文件系统`ext4`
```
之后就开始漫长的安装了。。。。

# 修复引导
CentOS安装完毕时候会重启电脑，这个时候你会看到系统选项有一个`Windows10`和 `CentOS`，选择`Windows10`，“卧槽！我的Win10居然没事，网上那群骗子，害我虚惊一场，重启看看CentOS先”。选择CentOS，然后就看到下面的东东：
![Windows未启动][5]
“特么我这个是Linux啊，你提示Windows未启动是什么意思？”
然后就开始了我的折腾之旅。。。。。
（这里省略上万字的心酸历程）
下面是解决方案
### 1. 网友建议  
网上说的在Windows下使用`easybcd`添加CentOS的引导，反正我试了很多遍就是没成功。事实上`easybcd`只能添加CentOS的mbr引导，这个可以在`easybcd`看出，然而我是通过UEFI来装的，应该就不行。
### 2. 几番折腾，新办法  
① 前面CentOS分区的时候实际上`/boot/efi`是一个`ESP`分区（UEFI 系统分区）。里面放的是CentOS的EFI引导文件。
```
$ ls -R EFI/
EFI/:
BOOT/  centos/

EFI/BOOT:
BOOTX64.EFI*  fallback.efi*

EFI/centos:
BOOT.CSV  gcdx64.efi*  grub.cfg.bak  grubx64.efi*     shim.efi*
fonts/    grub.cfg     grubenv       MokManager.efi*  shim-centos.efi*

EFI/centos/fonts:
unicode.pf2
```
其中最重要的文件是`grubx64.efi`，开机时，BIOS先通过ESP分区找到相应的efi程序，然后加载启动系统，这里的`grubx64.efi`就是用来加载CentOS的。
② 以此类推，Windows下肯定也有类似的文件。的确，在装Windows10的时候，会自动分配一个隐藏的ESP分区，盘符别名为`SYSTEM_DRV`:
```
BOOT/
EFI/
```
其中`EFI/Microsoft/Boot`目录里面放的就是加载Windows系统的efi文件。即`EFI/Microsoft/Boot/bootmgr.efi`。  
由上可知，整个硬盘共有两个ESP分区，常理上讲好像有点不科学，具体我也不知道可不可以。我觉得可能是不可以的，测试了下，发现BIOS每次都是从`SYSTEM_DRV`里面搜索efi程序，而CentOS的efi又不在`SYSTEM_DRV`目录下，这应该就是CentOS无法启动的原因。  
③ 所以接下来我就把CentOS的ESP分区里面的`EFI/centos`整个文件夹都拷贝到`SYSTEM_DRV`盘下的`/EFI`目录下。重启电脑发现还是不行，原因很简单
**a**.如果想要出现两个系统的选择项，那就要使用Win10引导CentOS或者有个程序来专门引导两个系统，前者我查了很多资料还是没弄出来，好像是要修改Windows的`BCD`文件，有点麻烦。至于后者，有个叫`rEFind`的程序（[rEFind下载][6]）可以达到目的，不过弄出来界面太丑了，我放弃了。
**b**.如果想要使用BIOS引导，就要把`EFI/centos`里面的路径写到一些特殊的文件，这个要用到一个叫`BOOTICE`的工具[BOOTICE下载][7]。  
### 3. `BOOTICE`使用教程  
① 打开BOOTICE，选择UEFI，点`修改启动序列`
![BOOTICE][8]
![修改启动序列][9]
② 选择左边的添加，先随便选一个本地磁盘的efi文件，然后把左边的`启动文件`改为
```
\EFI\centos\grubx64.efi
```
`启动分区`选择和Windows系统一样的项。
最后把它移动到第二个，保存。
③ 使用PE把`\EFI\centos`从CentOS的ESP目录移动到Windows的ESP目录下。这一步是为了让上面设置`启动文件:\EFI\centos\grubx64.efi`生效。  
不得不说，PE真是个好工具。

# 成功
通过上述步骤之后，重启电脑，电脑应该还是自动进入Win10，因为`BOOTICE`工具是把CentOS添加到BIOS的启动序列中= = 
重启，进入BIOS（Thinkpad是F12），可以看到BIOS启动列表有`Windows10`、`CentOS`、`USB HDD`等等，这个`USB HDD`就是U盘，点`CentOS`，就可以进入CentOS的引导了，然后启动CentOS。到这里就成功了。

# 心得
装这个双系统，修复引导花了我好长时间，不过也学到了很多东西，比如UEFI和传统Legacy的区别、UEFI的工作原理、PE的作用等等，最终文件没有发生丢失，也算是值了。下面是总结。
1. **UEFI+GPT装双系统真麻烦**
2. **微软垄断心态真可怕**
3. **Google搜索东西靠谱多了**
4. PE真是个好工具（进入磁盘修改EFI文件）  
5. 我装的双系统怎么和大家的不一样= =  
(又要继续干活了。。。)

# 参考资料
#### CentOS7安装教程
1. [U盘安装CentOS7的最终解决方案][10]
#### UEFI引导修复
1. [UEFI主板GPT方式安装CentOS6.4][11]
2. [UEFI+GPT安装Windows8和CentOS双系统][12]
3. [Windows10与CentOS的完美结合][13]
4. [UEFI的两种启动模式][14]
5. [支持 efi 的主板 双系统安装 ubuntu - 学习 EFI 和 gpt][15]
6. [如何在UEFI模式下Win8与Ubuntu多系统的安装？][16]
7. [如何在UEFI+GPT下使用rEFind实现Win10 + Kali2.0 双引导！][17]


  [1]: http://i2.piimg.com/567571/33d722ea2e1be7f3.png
  [2]: http://i2.piimg.com/567571/9bf538617d6ee03e.png
  [3]: http://i1.piimg.com/567571/f7904f40f4b33ecb.png
  [4]: http://i4.piimg.com/567571/b4c9a622f50ebe11.png
  [5]: http://i2.piimg.com/567571/f236c2bc1a183fc5.png
  [6]: https://sourceforge.net/projects/refind/
  [7]: http://www.ipauly.com/wp-content/uploads/2015/11/BOOTICEx64_v1.332.rar
  [8]: http://i4.piimg.com/567571/ae36600142436d51.png
  [9]: http://i4.piimg.com/567571/fb61a15fd564471e.png
  [10]: http://www.augsky.com/599.html
  [11]: http://blog.csdn.net/smstong/article/details/9408451
  [12]: http://blog.csdn.net/smstong/article/details/9422809
  [13]: http://m.blog.csdn.net/article/details?id=51207807
  [14]: http://blog.sina.com.cn/s/blog_53918a450102uzi6.html
  [15]: http://blog.csdn.net/icycolawater/article/details/51242999
  [16]: https://www.zhihu.com/question/22502670
  [17]: http://tshare365.com/archives/1774.html

---
title: Linux文件与目录管理
date: 2016-07-14 12:49:04
tags: linux
categories: Linux
toc: true 
---

# 一、目录与路径
```
.           代表此目录
..          代表上层目录
-           代表前一个工作目录
~           代表当前用户的主文件夹
~accout     代表用户account的主文件夹
```
与目录处理相关的命令
```
cd:     切换目录
pwd:    显示当前目录
mkdir:  新建目录
rmdir:  删除空目录
```
<!--- more --->
1. `cd`命令
> `cd [绝对路径 or 相对路径]`
2. `pwd`命令
> `pwd [-P] ` 
> 参数：-P 显示出当前路径，而非使用连接(link)路径  

例如：
```
[junhan@localhost test_dir]$ cd /var/mail
[junhan@localhost mail]$ pwd
/var/mail
[junhan@localhost mail]$ pwd -P
/var/spool/mail
[junhan@localhost mail]$ ls -ld /var/mail
lrwxrwxrwx. 1 root root 10 7月   7 18:39 /var/mail -> spool/mail
```
3. `mkdir`命令  
> `mkdir [-mp] 目录名称`
> -m: 直接配置权限（默认使用umask）
> -p: 递归创建  

示例：
```
[junhan@localhost test_dir]$ mkdir t1/t2/t3/t4
mkdir: 无法创建目录"t1/t2/t3/t4": 没有那个文件或目录
[junhan@localhost test_dir]$ mkdir -p t1/t2/t3/t4
[junhan@localhost test_dir]$ mkdir -m 711 test2
[junhan@localhost test_dir]$ ls -ld test2
drwx--x--x. 2 junhan junhan 6 7月  15 01:52 test2
```
4. `rmdir`命令
> `rmdir [-p] 目录名称`
> -P: 连同上层空目录一起删除

# 二、文件目录管理
```
ls: 查看文件与目录
cp: 复制
rm: 删除
mv: 移动
basename/dirname: 文件名/目录名 
```
1. `ls`命令
> `ls [-aAdfFhilnrRSt] 目录名称`  
> **参数：**  
> -a : 全部文件，包括.和..（常用来显示所有文件）    
> -A : 全部文件，不包括.和..  
> -d : 只列出目录本身（常用来显示目录本身信息）  
> -F : 给予附加数据结构，如：`*：`代表可执行文件；`/:`表示目录；`=：`表示socket文件；`|：`表示管道文件.  
> -i : 列出i-node  
> -l : 列出长数据串，包含文件属性和权限等信息（常用于查看权限）   
> -R : 连同子目录一同显示  
> `ls [--color={never,auto,always}] 目录名称`   
> --color={never,auto,always} : 设置显示颜色  
> `ls [--full-time] 目录名称`  
> --full-time : 显示完整时间  
> --time={atime,ctime} : 显示访问时间(atime)，权限改变时间(ctime)  
 
2. `cp`命令
> `cp [-adfilprus] source destination`  
> `cp option source1 source2 source3 ... directory`  
> **参数**  
> -a : 相当于-pdr  
> -d : 如果是连接文件，则复制连接文件（默认复制的是源文件）  
> -f : 即force，强制复制
> -i : 若已经存在，则询问（常用）  
> -l : 用来硬连接的连接文件创建  
> -P : 连同属性一同复制（默认复制不会复制属性，此命令常用于备份文件）  
> -r : 递归复制（常用于复制整个目录）  
> -s : 复制成符号链接文件(symbolice link)  
> -u : 即update，源文件相对较旧才更新  
3. `rm`命令
> `rm [-fir]`  
> **参数：** 
> -f : 即force  
> -i : 询问  
> -r : 递归删除
4. `mv`命令
> `mv [-fiu] source destination`  
> `mv [options] source1 source2 ... directory`  
> **参数:**  
> -f : force  
> -i : 询问  
> -u : 比较旧才更新  
5. `basename/dirname`命令  
示例：
```
[junhan@localhost junhan]$ dirname test_dir/
file   t1/    test2/ 
[junhan@localhost junhan]$ dirname test_dir/file 
test_dir
[junhan@localhost junhan]$ basename test_dir/file 
file

```

# 三、文件内容查阅
```
cat : 第一行开始显示
tac : 最后一行开始显示
nl : 显示行号
more : 一页页显示
less : 一页页显示，课往前翻页
head : 看头几行
tail : 看末几行
od : 指定进制查看
```
1. `cat`命令(concatenate)  
> `cat [-AbEnTv]`  
> -A : 等价于-vET，显示特殊字符  
> -b : 显示行号，不包括空白行  
> -n : 显示行号，包括空白行  
2. `nl`命令  
> `nl [-bnw] 文件`  
> -b a : 同cat -n  
> -b t : 同cat -b (默认值)  
3. `more`和`less`  
> **more:**  
> 空格：向下翻页  
> 回车：向下一行  
> /seq：向下搜索seq  
> :f：显示文件名和当前行号  
> q ：退出  
> **less:**  
> [PgDn]：向下翻页  
> [PgUp]：向上翻页  
> /seq：向下查  
> ?seq：向上查  
> n ：重复查  
4. `head`和`tail 文件`  
> head/tail [-n number]:number+表示前number行，number-表示除后number行。  
> tail -f 文件:  动态监测  
5. `od`命令
> `od [it Type] 文件`  
> a         : 默认字符  
> c         : ASCII  
> d[size]   : 十进制，占用size位  
> o[size]   : 八进制  
> x[size]   : 十六进制  
> f[size]   : 浮点数  

# 四、修改文件时间或创建新文件  
1. 时间
> `mtime`   : 文件修改时间(modification time)    
> `ctime`   : 权限或属性修改时间(status time)  
> `atime`   : 被读取时间，如cat,cp(access time)  
使用`ls -l --time={ctime,atime} 文件或目录`即可显示，默认显示mtime  
2. `touch`命令  
> `touch [-acdmt]`    
> -a : 修改atime  
> -c : 修改ctime  
> -m : 修改mtime  
> -d : 修改日期，后接日期  
> -t : 修改日期，后接日期 [YYMMDDhhmm]   

# 五、文件权限  
1. `umask`命令  
> umask  
> umask -S  
```
# 示例
[junhan@localhost junhan]$ umask
0022
[junhan@localhost junhan]$ umask -S
u=rwx,g=rx,o=rx
```
由umask可知，新建一个目录或者文件的默认权限：  
文件：`-rw-rw-rw- - 022 = -rw-r--r--`  
<<<<<<< HEAD
目录：`drwxrwxrwx - 022 = drwxr-wr-x`  
=======
目录：`drwxrwxrwx - 022 = drwxr-wr-x`  
>>>>>>> ADD:commit

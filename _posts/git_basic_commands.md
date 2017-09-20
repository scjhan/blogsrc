---
title: Git基础命令
toc: true
comments: true
date: 2017-07-11 09:19:56
tags: git
categories: 工具使用
---

git提交代码的基本步骤

<!--more-->

#### 1. Git环境部署

```shell
git config --global alias.ll "log --graph --all --pretty=format:'%Cred%h %Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit --date=relative"
git config --global color.status auto
git config --global color.branch auto
git config --global color.diff auto
git config --global color.interactive auto
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.st status
git config --global alias.last "log -1 HEAD"
git config --global alias.df diff
git config --global color.ui true
```

#### 2. Git提交规范

1. git st 查看修改内容
2. git add <file> 添加修改文件
3. git commit -am "" 提交到本地仓库
4. git ll 查看head指针位置，判断是否需要fetch
   1. 如果head指针没有指向当前
   2. git fetch
   3. git rebase origin/branch
      1. 如果有冲突（REBASE 1/2）
      2. 解决冲突
      3. git add <冲突文件>
      4. git rebase --continue
   4. git push origin HEAD:master
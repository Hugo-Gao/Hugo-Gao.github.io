---
title: Git命令合集
date: 2016-09-22 13:14:08
tags: [Git]
---

git config --global user.name "yourname"  --------提交你的用户名

----------
git config --global user.email "youremailname" -----------提交你的邮箱地址

----------
git config user.name     ----------查看你当前的用户名

----------
git config user.email ------------查看你当前邮箱名

----------
pwd  ------------------查看当前路径名

----------
git init -------------------创建代码仓库

----------
ls -ah  -------------------查看当前代码仓库所有文件

----------
 git add .    -------------------------添加当前目录所有文件到代码仓库



----------
git status  ------------------------获取当前代码仓库所在分支以及状态

----------
git commit -m "your description"  ----------------------提交文件到代码仓库并描述

----------
git diff  ---------------------查看上次改动的内容

----------
git log ---------------------查看最近3次历史提交记录

----------
git log --pretty=oneline ----------------查看最近历史纪录的commit ID

----------
git reset --hard HEAD^  ---------------------将当前版本换回多少个^之前

----------
git reflog  -----------------记录你的当前版本未来commit的id、

----------
git reset --hard 3628164   ----------指定回到某个版本号

----------
git checkout -- file  -----------撤销某个文件的提交

----------
git checkout -- readme.txt -----------------丢弃工作区的修改

----------
git rm test.txt   --------------------删除某个文件

----------
$ ssh-keygen -t rsa -C "youremail@example.com" ---------创建ssh文件

----------
git remote add origin git@github.com:githubname/repositorityName.git --------关联你github上的远程仓库

----------
git push -u origin master     ----------------第一次提交到Github

----------
git push  origin master   -------------以后提交到github

----------
git clone git@github.com:githubname/repositorityName.gi  -----克隆远程项目到本地

----------






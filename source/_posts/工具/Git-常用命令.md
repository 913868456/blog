---
title: Git 常用命令
date:  2018-08-28 12:35
categories:
- 工具
tags:
- Git
---


#### Git 常用命令
配置
```
git config --global user.name "你的名字"
git config --global user.email "邮箱"

```
远程仓库操作
```
git remote add origin "仓库地址" //链接远程仓库

git push -u origin master        //第一次将本地仓库推送到远程仓库

git branch -r                    //查看远程分支列表

git clone 仓库地址               //将远程仓库克隆到本地

git push origin master           //将本地库改动提交远程

git pull                         //更新本地库至远程库的最新改动

git push origin --delete 远程分支名  //删除远程分支
```
SSH Key 操作
```
$ ls -al ~/.ssh
# Lists the files in your .ssh directory, if they exist

ssh -keygen -t rsa -C "你的邮箱"   //生成SSH key,保存路径为/root/.ssh
```
创建忽略文件
```
touch .gitignore  //不需要服务端提交的内容可以写到忽略文件里
```
添加
```
git init 将当前目录初始化为git仓库
git add 文件名 将文件添加到暂存区
git add -A   //将所有修改文件全部添加到缓存区
git commit -m "描述"  将暂存区提交到仓库
```
查询
```
git status                //查询仓库状态
git diff 文件名           //比较文件差异
git log                   //查看仓库历史记录(详细)
git log --oneline         //查看仓库历史记录(单行)
git reflog                //查看所有版本的commitID  (本地分支Git记录不小心删除时,可以用此查看删除的Git记录)
```
撤销(回滚)
```
git  checkout -- 文件名     //撤销工作区修改
git reset HEAD 文件名       //撤销暂存区的修改
git reset --hard 该版本ID   //回退到历史版本
git reset --hard HEAD^      //回退到上个版本
git reflog                  //回滚到未来
```
删除
```
rm filename                 //删除本地文件
git rm index.html --cached  //保证当前工作区中没有index.html,使用--cached表示只删除换缓存区中的内容
```
标签
```
git tag 标签名           //为当前版本打标签
git tag 标签名 该版本ID  //为历史版本打标签
git tag                  //查看所有标签
git show  标签名         //查看某一标签
git tag -d 标签名        //删除某一标签
```
分支管理
```
git branch dev           //创建分支
git checkout dev         //切换分支
git checkout -b dev      //创建并切换分支
git branch -d dev        //删除分支
git commit -a -m 'dev1'  //在分支上提交新的版本
git merge dev            //合并分支
git stash            //在分支开发过程中遇到问题,需要切换到其他分支,保留写好的内容再切换到主干
git stash apply          //再次切换分支后需要应用一下保留的内容
git stash drop           //丢弃保存的内容
git stash pop            //使用并丢掉
```
![git flow](https://upload-images.jianshu.io/upload_images/3340896-9aefe571129ba75d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### Git Submodule 管理项目子模块

```
git clone <repository> --recursive                   //递归的方式克隆整个项目

git submodule add <repository> <path>                //添加子模块

git submodule init                                   //初始化子模块(远程仓库子模块添加或删除后,需要进行此操作)

git submodule update                                 //更新子模块

git submodule foreach git pull                      //拉取所有子模块

git rm moduleA                                      //删除子模块

```
#### git flow 命令

```
brew install git-flow    //安装git-flow
```
![gitFlow命令.png](https://upload-images.jianshu.io/upload_images/3340896-c6377d2d53ee3e4d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

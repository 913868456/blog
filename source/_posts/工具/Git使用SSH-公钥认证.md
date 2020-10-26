
---
title: Git SSH认证
date:  2018-03-12 11:08
categories:
- 工具
tags:
-  SSH
-  Git
---

###### 许多 Git 服务器都使用 SSH 公钥进行认证。 为了向 Git 服务器提供 SSH 公钥，如果某系统用户尚未拥有密钥，必须事先为其生成一份。 这个过程在所有操作系统上都是相似的。

1.  首先，你需要确认自己是否已经拥有密钥。 默认情况下，用户的 SSH 密钥存储在其 ` ~/.ssh` 目录下。 

![文件目录](http://upload-images.jianshu.io/upload_images/3340896-1b010822f8d11e9e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


进入该目录并列出其中内容，你便可以快速确认自己是否已拥有密钥,进去以后是这个样子,说明已经拥护密钥,将 `id_rsa.pub`文件发送给git管理员,让管理员添加进去.
![密钥文件](http://upload-images.jianshu.io/upload_images/3340896-44cfaed89c9a17a1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
如果没有找到,直接看步骤3

2. 打开SouceTree 选择 `新建` -> `从URL克隆`

![选择从URL克隆](http://upload-images.jianshu.io/upload_images/3340896-646609abed39b165.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

`源URL` 填写 `git`仓库地址, `目标路径` 选择本地仓库地址,如果仓库链接成功,底下会显示`git`仓库名称,选择 `克隆` ,ok,完成项目拷贝,可以开心地使用SourceTree了.
![添加源路径,目标路径](http://upload-images.jianshu.io/upload_images/3340896-369777c21dcf1a2d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


3. 如果没有在目录下找到`id_rsa.pub`文件,进入终端,执行以下命令

```
ssh-keygen -t rsa -C "你的邮箱"
```
显示结果
```
$ ssh-keygen -t rsa -C "你的邮箱"
Generating public/private rsa key pair.
Enter file in which to save the key (/home/schacon/.ssh/id_rsa):
Created directory '/home/schacon/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/schacon/.ssh/id_rsa.
Your public key has been saved in /home/schacon/.ssh/id_rsa.pub.
The key fingerprint is:
d0:82:24:8e:d7:f1:bb:9b:33:53:96:93:49:da:9b:e3 你的邮箱
```
首先 `ssh-keygen` 会确认密钥的存储位置（默认是 `.ssh/id_rsa`），点`回车键`,选择默认存储位置,然后它会要求你输入两次密钥口令。如果你不想在使用密钥时输入口令，将其留空即可。重新执行步骤1和2就ok了


###### 想要了解更多Git相关内容,请参考
[git-scm.com](https://git-scm.com/book/zh/v2/Git-%E5%9F%BA%E7%A1%80-%E8%8E%B7%E5%8F%96-Git-%E4%BB%93%E5%BA%93)





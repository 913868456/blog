---
title:  Linux文件命令
date:  2018-12-11 19:10
categories:
- [Linux, 文件及权限命令]
tags: 
- Linux
---

## 权限
```
-rwxr--r--

//总共9个字符
//-表示类型,`d`为目录,`-`为文件
//rwx 表示所有者权限,读写执行
//r-- 表示所述用户组的权限,只读
//r--表示其他权限,只读

ls -al //查阅文件详细信息
-rw-r--r-- 1 root   root   239 5 6 13:22 test.txt 
 // [文件类型与权限] [文件连接数][所有者账号] [用户组账号] [文件容量 单位 B] [修改日期] [文件名]

```
## 用户切换
```
//切换到root 用户
sudo -i

//root 切换到指定用户
su - username   //注意- 和 用户名之间有空格
```

## 文件与目录
```
cd filePath //切换目录
pwd //打印当前路径

//新建(增加)
mkdir dirname //新建目录
mkdir -p test1/test2/test3  //创建多层目录
mkdir -m 711 test2 //新建权限为rwx--x--x的目录
touch filename //创建文件

//删除
rm filename   //移除文件
rm -r dirName //移除目录
rmdir dirname //移除目录
rmdir -p test1/test2/test3 //连同上层空的目录也一起删除

//复制
cp  源路径 目标路径
cp -a sourcefile desfile //完全复制
cp -i sourcefile desfile  //询问
cp -r sourcefile desfile  //递归复制

//移动
mv [-fiu] souce destination
-f //强制
-i  //询问是否覆盖
-u //更新为最新

//修改
//修改的用户组: groupname在 etc/group 文件中存在
chgrp [-R] groupname dirname/filename

//修改拥有者:  ownername 在 etc/passwd 文件中存在
chown ownername dirname/filename
chown :groupname dirname/filename
chown ownername:groupname dirname/filename

//修改文件权限
//数字修改权限  各个权限加权分 r: 4 w: 2 x: 1 
chmod 777 dirname/filename //-rwxrwxrwx
chmod 755 dirname/filename //-rwxr-xr-x
//修改权限(通过字符)
//chmod      u g o a       + - =      rwx        文件目录
chmod u=rwx,go=r filename  //-rwxr--r--
chmod a+w filename   //增加写权限
chmod a-x filename  //减少执行权限 

basename  dir/finename  //获取文件名
dirname  dir/filename  //获取目录名

//修改新建文件和目录的默认权限
umask 002  //owner group others  002表示拿掉others的写权限

root的umask默认是022
一般用户umask默认是002

//查看文件类型
file filePath 

//脚本文件名查找
which command
//文件查找
whereis 文件名
locate 文件名或部分文件名
find /home -user vbird //查找/home下面属于vbird的文件
find  /  name 文件名 //查找文件名
```










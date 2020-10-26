
---
title:  ShellScript
date:  2018-03-09 18:22
categories:
- [Linux, Shell]
tags: 
- Shell
- Linux
---

```
#str    #输出字符串长度
#array[@]     #输出数组长度
echo $array[@]    #打印数组

source ./test1.sh  #导入外部文件,可以引用外部文件的变量

# shell script 追踪与debug
sh [-nvx] xxx.sh
Options:
-n: 不需要执行script,仅检查语法问题
-v: 在执行script前,先将script内容输出到屏幕上
-x:将使用到的script内容显示到屏幕

```


---
title: SVN 配置
date: 2017-03-16 11:17
categories:
- 工具
tags:
-  SVN
-  Mac
---

SVN 配置

1、创建SVN Repositroy
打开终端 输入：svnadmin create /path/svn/pro
svn是目录，pros是版本库，输入完成即可看见生成文档
2、配置svn用户权限。
/path/svn/pro/conf/目录下存在3个文件：authz,passwd,svnserve.conf
（1) 配置svnserve.conf
将里面的
＃anon-access = read
＃auth-access = write
＃password-db = passwd
＃authz-db = authz
四行前的＃号去掉，再将anon-access = read改为anon-access = none，这样禁止匿名访问
**注意**:
- 这句话必须使用，这样子才能显示Log的信息
- 在＃号后是有空格的，去掉空格，上文字顶格,否则也有错误
（2)  配置passwd
里面存的是用户与密码，有示例，直接按照它的格式添加用户和密码就可以了
test1=123
test2=456
（3)   配置authz
[groups] 后面跟的是用户组设置，可以将你在passwd里设置的用户添加到不同的用户组里，那么之后，可以对不同用户组设置不同的权限，以免多用户设置麻烦，多个用户用,号分隔。可按它的示例做
[groups]
testgroups=test1,test2

之后，可以对不同的版本库进行权限设置，底下有一个示例，按它的写法写就可以了，如果需要对所有的版本库设置，利用[/]就可以了。如：
[/]
@testgroups=rw
个人配置只需在[/]下面添加
用户名=rw即可
**PS**：用户组前要用@符号，如果是用户，直接写用户名就可以了。rw代表可读写，显然只读就是r了
3、启动SVN服务
svnserve -d -r /path/svn  特别注意，路径一定是SVN的目录，不是其中一个版本库的目录，不然，能正常启动，就是访问有问题
没有任何输出，则启动成功
4.配置conrnerStone
点击ConerStone左下角 如下图的“+”，添加SVN地址，
需要填写的方格
server： IP地址 不包括Svn后面的其他路径 例如：主机IP 192.168.。。。
Repisitory path:Ip后面的路径  例如：上文的Pro
Name：用户名
Password：密码
5.让SVN支持.a文件的上传
（1）打开终端：vi ~/.subversion/config 打开config文件
（2）找到：

#global-ignores = *.o *.lo *.la *.al .libs *.so *.so.[0-9]* *.a *.pyc *.pyo 

# *.rej *~ #*# .#* .*.swp .DS_Store

这是svn默认的忽略列表，更改这个即可上传和下载被忽略的文件
去掉开头第一行的#和#前面的空格，第一行global前面不能有空格。
去掉第二行的#，空格不用管。
删除上问红色的*.a，这样即可上传.a文件。

#**Cornerstone Svn简单使用指南**

一、安装并拷贝项目

1.第一步：安装svn.
2.第二步：第一个使用svn，找到“Check Out Working Copy”选项，选择并点击。
目的：从服务器上拷贝一份全新的项目工程。
3.第三步：可以正常使用了。。。

二、在项目中使用

1.查看日志
找到“Log”选项，选择并点击。
查看自己当前的版本是否是最新的，如果不是最新的版本，从第2步开始执行；
如果自己当前的版本是最新的，从第3步开始执行；
2.更新到最新版本
当前程序员在打开工程项目之前，找到“Update to Latest Revision”选项，选择并点击。
目的：保持当前程序员客户端的项目版本是最新的。
3.编辑项目
4.提交之前，再次点击“Update to Latest Revision”，保持项目是最新版本。
5.更新最新版本后，如果有错误，冲突等情况，解决，直到没错误！
6.提交项目，找到“Commit Changes”选项，选择并点击。
目的：把当前编辑后的项目提交的服务器。
三、"lock"和“unlock”的使用
当我们正在编辑某个文件时，为了防止被其他人修改，可以在编辑之前，使该文件处于锁定状态，当我们编辑后，要提交的时候，再解锁。
四、每次提交项目，都要写详细备注并署名。
五、恢复到以前的版本，以前其他出错情况，请参考稍后的文档说明或上网搜索。 
Cornerstone的逻辑很清晰，界面打开后，左边栏上下分开，上面是working copies的列表，下面是REPOSITORIES的列表。常见的功能基本上跟windows一样，在上下文中可以得到。

1、连接到HTTP server

RESPOSITORIES栏上，标题栏的右手边有+和-，点击+号（如果第一次打开这个软件，这一步会自动跳出来），出现的对话框中，选择HTTP Server

Server：http server的地址
Port: 空着就可以

Repository path: 填入http server的Repository地址。一般这个地址，开源项目host在你创建项目后都会给你地方拷贝。

提示：在这个栏目下面，有一行最后形成的地址，你可以跟开源项目给你的路径看一下是否一致。

NickName: 空着就可
Name: 填入登录的账户名称
Password: 填入登录的密码

然后点击Add，就可以把这个Repository加入了。 

2、Check out
按照1做好自己的REPOSITORIES，选中一个的时候，左上角的Check Out就会高亮，点击该图标，就会出现一个路径选择的对话框。填好相应的地方后，就开始check out了。
##**注意**：
***第一次在check out没有完成之前，你选择的本地work space目录是不会出现在 working COPIES上的。***
3、Commit delete file
先把本地的文件删除，然后在cornerstone的working copies栏目中，找到相应的文件所在的位置，可以看到这个文件有个M标志，表示missing。在该文件上右键点击，上下文菜单中找到Delete，然后确认。刚才的M标识换成了D，表示Deleted。这个时候你就可以commit了。
4、Relocate
如果服务器上改了原始的某个目录的路径，那么在WORKING COPIES上，右键可以找到Relocate To，后面会跟上相应的目录，点击就可以重新定位。定位好后，相应的REPOSITORIES会自动更改。 

##参考：

  [conerStone的配置及使用说明](http://www.tuicool.com/articles/n6fyq2)
[MAC下配置SVN](http://www.cnblogs.com/onlyfu/archive/2012/05/08/2489814.html)
http://blog.csdn.net/wisdom605768292/article/details/19068601

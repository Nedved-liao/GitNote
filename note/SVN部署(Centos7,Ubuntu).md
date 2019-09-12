---
title: SVN部署(Centos7,Ubuntu)
date: 2018-10-23 17:22:37
tags: CSDN迁移
---
 [ ](http://creativecommons.org/licenses/by-sa/4.0/) 版权声明：本文为博主原创文章，遵循[ CC 4.0 BY-SA ](http://creativecommons.org/licenses/by-sa/4.0/)版权协议，转载请附上原文出处链接和本声明。  本文链接：[https://blog.csdn.net/Nedved_L/article/details/83310012](https://blog.csdn.net/Nedved_L/article/details/83310012)   
    
   # SVN 简介 

 SVN是Subversion的简称，是一个开放源代码的版本控制系统，相较于RCS、CVS，它采用了分支管理系统，它的设计目标就是取代CVS。互联网上很多版本控制服务已从CVS迁移到Subversion。说得简单一点SVN就是用于多个人共同开发同一个项目，共用资源的目的

 

 
# SVN 的主要功能

 

 （1）目录版本控制  
 CVS 只能跟踪单个文件的历史, 不过 Subversion 实作了一个 "虚拟" 的版本控管文件系统, 能够依时间跟踪整个目录的变动。 目录和文件都能进行版本控制。  
 （2）真实的版本历史  
 自从CVS限制了文件的版本记录，CVS并不支持那些可能发生在文件上，但会影响所在目录内容的操作，如同复制和重命名。除此之外，在CVS里你不能用拥有同样名字但是没有继承老版本历史或者根本没有关系的文件替换一个已经纳入系统的文件。在Subversion中，你可以增加（add）、删除（delete）、复制（copy）和重命名（rename），无论是文件还是目录。所有的新加的文件都从一个新的、干净的版本开始。  
 （3）自动提交  
 一个提交动作，不是全部更新到了档案库中，就是完全不更新。这允许开发人员以逻辑区间建立并提交变动，以防止当部分提交成功时出现的问题。  
 （4）纳入版本控管的元数据  
 每一个文件与目录都附有一組属性关键字并和属性值相关联。你可以创建, 并儲存任何你想要的Key/Value对。 属性是随着时间来作版本控管的,就像文件內容一样。  
 （5）选择不同的网络层  
 Subversion 有抽象的档案库存取概念, 可以让人很容易地实作新的网络机制。 Subversion 可以作为一个扩展模块嵌入到Apache HTTP 服务器中。这个为Subversion提供了非常先进的稳定性和协同工作能力，除此之外还提供了许多重要功能: 举例来说, 有身份认证, 授权, 在线压缩, 以及文件库浏览等等。还有一个轻量级的独立Subversion服务器， 使用的是自定义的通信协议, 可以很容易地通过 ssh 以 tunnel 方式使用。  
 （6）一致的数据处理方式  
 Subversion 使用二进制差异算法来异表示文件的差异, 它对文字(人类可理解的)与二进制文件(人类无法理解的) 两类的文件都一视同仁。 这两类的文件都同样地以压缩形式储存在档案库中, 而且文件差异是以两个方向在网络上传输的。  
 （7）有效的分支(branch)与标签(tag)  
 在分支与标签上的消耗并不必一定要与项目大小成正比。 Subversion 建立分支与标签的方法, 就只是复制该项目, 使用的方法就类似于硬连接（hard-link）。 所以这些操作只会花费很小, 而且是固定的时间。  
 （8）Hackability  
 Subversion没有任何的历史包袱; 它主要是一群共用的 C 程序库, 具有定义完善的API。这使得 Subversion 便于维护, 并且可被其它应用程序与程序语言使用。

 
# 优于CVS之处

 1、原子提交。一次提交不管是单个还是多个文件，都是作为一个整体提交的。在这当中发生的意外例如传输中断，不会引起数据库的不完整和数据损坏。  
 2、重命名、复制、删除文件等动作都保存在版本历史记录当中。  
 3、对于二进制文件，使用了节省空间的保存方法。（简单的理解，就是只保存和上一版本不同之处）  
 4、目录也有版本历史。整个目录树可以被移动或者复制，操作很简单，而且能够保留全部版本记录。  
 5、分支的开销非常小。  
 6、优化过的数据库访问，使得一些操作不必访问数据库就可以做到。这样减少了很多不必要的和数据库主机之间的网络流量。

 
# Centos 部署

 官网下载： [http://subversion.apache.org/packages.html](http://subversion.apache.org/packages.html)  
 SVN客户端TortoiseSVN :[https://tortoisesvn.net/downloads.html](https://tortoisesvn.net/downloads.html)

 
## 1.使用yum安装

 
> [root@78778e06dc0a /]# yum install subversion -y
> 
>  
 
## 2.查看安装版本

 
> [root@78778e06dc0a /]# svn --version  
>  svn, version 1.7.14 (r1542130)  
>  compiled Apr 11 2018, 02:40:28
> 
>  Copyright (C) 2013 The Apache Software Foundation.  
>  This software consists of contributions made by many people; see the NOTICE  
>  file for more information.  
>  Subversion is open source software, see http://subversion.apache.org/
> 
>  The following repository access (RA) modules are available:
> 
>  * ra_neon : Module for accessing a repository via WebDAV protocol using Neon.  
>  - handles 'http' scheme  
>  - handles 'https' scheme  
>  * ra_svn : Module for accessing a repository using the svn network protocol.  
>  - with Cyrus SASL authentication  
>  - handles 'svn' scheme  
>  * ra_local : Module for accessing a repository on local disk.  
>  - handles 'file' scheme
> 
>  
 

 
## 3.创建一个目录用于储存于SVN目录

 
> [root@78778e06dc0a /]# mkdir /SVN
> 
>  
 
## 4.新建一个仓库

 
> [root@78778e06dc0a /]# svnadmin create /SVN/No-1/  
>  [root@78778e06dc0a /]# ll /SVN/No-1/  
>  total 8  
>  -rw-r--r--. 1 root root 229 Oct 23 07:00 README.txt  
>  drwxr-xr-x. 2 root root 54 Oct 23 07:00 conf  
>  drwxr-sr-x. 6 root root 233 Oct 23 07:00 db  
>  -r--r--r--. 1 root root 2 Oct 23 07:00 format  
>  drwxr-xr-x. 2 root root 231 Oct 23 07:00 hooks  
>  drwxr-xr-x. 2 root root 41 Oct 23 07:00 locks
> 
>  
 
> 以下关于目录的说明：  
>  hooks目录：放置hook脚步文件的目录  
>  locks目录：用来放置subversion的db锁文件和db_logs锁文件的目录，用来追踪存取文件库的客户端  
>  format目录：是一个文本文件，里边只放了一个整数，表示当前文件库配置的版本号  
>  conf目录：是这个仓库配置文件（仓库用户访问账户，权限）
> 
>  conf目录下有三个文件authz，passwd，svnserve.conf，其作用如下：  
>  authz：负责账号权限的管理，控制账号是否读写权限  
>  passwd：负责账号和密码的用户名单管理  
>  svnserve.conf：svn服务器配置文件
> 
>  
 
## 5.配置SVN配置文件svnserver.conf

 
> >  [root@78778e06dc0a /]# vim /SVN/No-1/conf/svnserve.conf
> 
>  ...  
>  ### users have read-only access to the repository, while authenticated  
>  ### users have read and write access to the repository.  
> anon-access = read  ##没有授权的用户，用户对存储库具有只读访问权限  
> auth-access = write ##拥有授权的用户，用户对存储库具有只读写访问权限  
>  ...  
>  ### Uncomment the line below to use the default password file.  
> password-db = passwd ##指定用户密码文件。通过该文件可以实现以路径为基础的访问控制。 除非指定绝对路径，否则文件位置为相对conf目录的相对路径。  
>  ...  
>  ### Uncomment the line below to use the default authorization file.  
> authz-db = authz ##指定权限配置文件名。通过该文件可以实现以路径为基础的访问控制。 除非指定绝对路径，否则文件位置为相对conf目录的相对路径。  
>  ...  
>  ### is repository's uuid.  
> realm = This is My First Test Repository ##指定版本库的认证域，即在登录时提示的认证域名称。若两个版本库的 认证域相同，建议使用相同的用户名口令数据文件。
> 
>  注意: 注释时,要把前面的空格也删掉
> 
>  
 
## 6.使用默认文件.配置用户及密码

 版本库的目录格式如下:

 [<版本库>:/项目/目录]  
 @<用户组名> = 权限  
 <用户名> = 权限

 
> [/SVN/No-1/]  
>  @groupname = rw  
>  username = r  
>  * =
> 
>  
 其中[]內容有許多写法：  
 [/],表示根目录及其一下的路径，根目录是svnserver启动时指定好的  
 [No-1:/],表示对版本库No-1设置权限；  
 [No-1:/svnadmin],表示对svnadmin设置权限；  
 [No-1:/svnadmin/second],表示对second设置权限；

 权限的主体可以是用户组，用户或者*  
 用户组在前面要以@开头  
 *表示全部用户  
 权限分为：r ,w, rw和null  
 null空表示没有任何权限。  
 auhtz配置文件中的每个参数，开头不能有空格，对于组要以@开头，用户不需要。  
 *= 表示除了上面设置的权限用户组以外，其他所有用户都设置空权限，空权限表示禁止访问本目录,记得要加上。

 
### 添加用户

 

 
> >  [root@78778e06dc0a /]# vim /SVN/No-1/conf/authz  
>  最后添加如下  
>  #No-1  
>  Noadmin = Fist,Second  
>  [No-1:/]  
>  @Noadmin = rw  
>  test = r  
>  *=
> 
>  
 
### 配置对应的密码

 
> >  [root@78778e06dc0a /]# vim /SVN/No-1/conf/passwd
> 
>  [users]  
>  # harry = harryssecret  
>  # sally = sallyssecret  
>  First = 123  
>  Second = 456  
>  test = 1
> 
>  
 
## 7.使用命令svnserve启动服务

 
> svnserver -d -r /SVN/ --listen-port 3690
> 
>  
 svnserve -d -r 目录 --listen-port 端口号

 -r: 配置方式决定了版本库访问方式。  
 --listen-port: 指定SVN监听端口，不加此参数，SVN默认监听3690

 
## 8.关闭(通过kill)

 
> >  kill -9 `ps -elf | grep svn|head -1|awk '{print $4}'`
> 
>  
 

 
# Ubuntu

 如果 Subversion 客户端没有安装，命令将报告svn命令找不到的错误。  
 

 
> root@78778e06d22a:~# svn --version  
>  The program 'svn' is currently not installed. You can install it by typing:  
>  apt-get install subversion
> 
>  
 我们可以使用 apt-get 命令进行安装

 
> root@78778e06d22a:~# apt-get install subversion  
>  Reading package lists... Done  
>  Building dependency tree   
>  Reading state information... Done  
>  The following packages were automatically installed and are no longer required:  
>  augeas-lenses hiera libaugeas0 libxslt1.1 ruby-augeas ruby-deep-merge ruby-json ruby-nokogiri ruby-rgen ruby-safe-yaml ruby-selinux ruby-shadow  
>  Use 'apt-get autoremove' to remove them.  
>  The following extra packages will be installed:  
>  libserf-1-1 libsvn1  
>  ...
> 
>  
 安装成功之后，执行 svn --version 命令。

 
> root@78778e06d22a:~# svn --version  
>  svn, version 1.8.13 (r1667537)  
>  compiled Sep 8 2015, 14:59:01 on x86_64-pc-linux-gnu
> 
>  
 Ubuntu下的SVN安装完成。

 

 

 

 

 

 

   
 
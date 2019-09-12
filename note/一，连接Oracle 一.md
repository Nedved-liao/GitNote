---
title: 一，连接Oracle 一
date: 2018-03-20 18:30:52
tags: CSDN迁移
---
 [ ](http://creativecommons.org/licenses/by-sa/4.0/) 版权声明：本文为博主原创文章，遵循[ CC 4.0 BY-SA ](http://creativecommons.org/licenses/by-sa/4.0/)版权协议，转载请附上原文出处链接和本声明。  本文链接：[https://blog.csdn.net/Nedved_L/article/details/79629464](https://blog.csdn.net/Nedved_L/article/details/79629464)   
    
  连接Oracle数据库方法：   
 一，使用sqlplus连接   
 二，使用第三方软件连接

 **sqlplus**

 sqlplus 工具简介   
 (1)、概述：sqlplus是在Linux下操作oracle的工具   
 (2)、操作如下：在命令行中中输入“sqlplus”即可

 sqlplus 语法： 

 
```
sqlplus  用户/密码@数据库名  [as sysdba/sysoper]
```
 说明：当用特权用户身份连接时，必须带上as sysdba或是as sysoper   
 sqlplus常用命令：

 **1，以无用户身份登录连接sqlplus**

 
```
[oracle@Nedved ~]$ sqlplus /nolog

SQL*Plus: Release 11.2.0.1.0 Production on Tue Mar 20 18:07:40 2018

Copyright (c) 1982, 2009, Oracle.  All rights reserved.

SQL> 
```
 说明：此处登录的并不是Oracle而是sqlplus工具，

 **2.conn 连接Orace了数据库**   
 以system登录Oracle数据库

 
```
SQL> conn system/oracle
Connected.
SQL> show user
USER is "SYSTEM"
SQL> conn scott/oracle
Connected.
SQL> show user
USER is "SCOTT"

```
 如上可以通过conn 以实现切换用户

 **3.disc/disconn/disconnect**   
 说明: 该命令用来断开与当前数据库的连接

 **4.pssw[ord]**   
 说明: 该命令用于修改用户的密码，如果要想修改其它用户的密码，需要用sys/system登录。

 **5.clear screen**   
 清屏

 **6.启动 | 关闭 Oracle状态**

 关闭

 
```
SQL> shutdown immediate；
SP2-0717: illegal SHUTDOWN option
SQL> shutdown immediate
Database closed.
Database dismounted.
ORACLE instance shut down.
```
 启动

 
```
SQL> startup
ORACLE instance started.

Total System Global Area  759943168 bytes
Fixed Size          2217224 bytes
Variable Size         566233848 bytes
Database Buffers      184549376 bytes
Redo Buffers            6942720 bytes
Database mounted.
Database opened.
```
 **特别说明：**   
 oracle启动模式有3种：

 l Startup nomount （nomount模式）启动实例不加载数据库。

 l Startup mount （mount模式）启动实例加载数据库但不打开数据库

 l Startup （open 模式）启动实例加载并打开数据库，就是我们上面所用的命令

 Nomount模式中oracle仅为实例创建各种内存结构和服务进程，不会打开任何数据库文件   
 1） 创建新数据库

 2） 重建控制文件

 
```
这2种操作都必须在这个模式下进行。

Mount模式中oracle只装载数据库但不打开数据库
1)    重命名数据文件

2)    添加、删除和重命名重做日子文件

3)    执行数据库完全恢复操作

4)    改变数据库的归档模式

这4种操作都必须在这个模式下进行

Open模式（就是我们上面的startup不带任何参数的）正常启动。

```
 当然这3种模式之间可以转换：

 Alter database mount(nomount模式)—〉alter database open(mount 模式)—〉（open模式）

 当然还有其它一些情况，在我们open模式下可以将数据库设置为非受限状态和受限状态

 在受限状态下，只有DBA才能访问数据库   
 1） 执行数据导入导出

 2） 使用sql*loader提取外部数据

 3） 需要暂时拒绝普通用户访问数据库

 4） 进行数据库移植或者升级操作

 在打开数据库时使用startup restrict命令即进入受限状态。

 使用alter system disable restricted session命令即可以将受限状态改变为非受限状态。

 使用alter system enable restricted session命令可以将非受限状态变为受限状态 

 使用alter database open read only可以使数据库进入只读状态。

 使用alter database open read write 可以使数据库进入读写状态。

 当然在某些情况下可能是用上述各种启动方式都无法成功启动数据库，这个时候就要使用startup force命令来强行启动数据库。当然谁都不想碰到这种情况：）

 关闭数据库   
 1）正常关闭 shutdown   
 2) 立即关闭 shutdown immediate   
 3) 关闭事务 shutdown transactional   
 4) 强行关闭 shutdown abort,当然谁都不想碰到这种情况。

   
  
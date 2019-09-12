  ## 搭建数据库服务器

 版本众多，但为了追求稳定选择的是5.7

 在使用YUM REPOSITORY官方给出的版本如下：   
 The MySQL Yum repository includes the latest versions of:

 
```
MySQL 8.0 (Development)
MySQL 5.7 (GA)
MySQL 5.6 (GA)
MySQL 5.5 (GA - Red Hat Enterprise Linux and Oracle Linux Only)
MySQL Cluster 7.5 (GA)
MySQL Cluster 7.6 (Development)
MySQL Workbench
MySQL Fabric
MySQL Router (GA and preview)
MySQL Utilities
MySQL Connector / ODBC
MySQL Connector / Python
MySQL Shell (GA and preview)

```
 下面就详细讲解如何搭建数据库服务器

 
## 前期准备

 1.服务器（DELL HP 联想……）   
 2.安装系统（Windows Unix Linux）   
 3.安装提供数据库服务的基本管理   
 （商业or开源，是否跨平台，软件来源，rpm或是源码包）   
 4.安装MySQL软件（5.7）   
 5.关闭防火墙，selinux

 
## 安装MySQL流程

 官网下载MySQL软件包MySQL 5.7   
 ![这里写图片描述](https://img-blog.csdn.net/20171122124140778?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvTmVkdmVkX0w=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)   
 1）装包前检查环境，是否已安装过数据库软件。不同版本之间会有小许不兼用

 rpm -q mariadb mariadb-server

 2）解压

 
```
tar -xvf mysql-5.7.17-1.el7.x86_64.rpm-bundle.tar
```
 ![这里写图片描述](https://img-blog.csdn.net/20171122125040326?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvTmVkdmVkX0w=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)   
 3）删除带最少安装的RPM包，

 rm -rf mysql-community-server-minimal-5.7.17-1.el7.x86_64.rpm 

 4）准备安装环境

 [root@BD4 09.mysql]# rpm -Uvh mysql-community-*.rpm   
 警告：mysql-community-client-5.7.17-1.el7.x86_64.rpm: 头V3 DSA/SHA1 Signature, 密钥 ID 5072e1f5: NOKEY   
 错误：依赖检测失败：   
 perl(Data::Dumper) 被 mysql-community-test-5.7.17-1.el7.x86_64 需要   
 perl(JSON) 被 mysql-community-test-5.7.17-1.el7.x86_64 需要

 可以看到需要Data::Dumper，JSON   
 依赖关系可以用yum解决

 yum install perl-Data-Dumper.x86_64 perl-JSON -y

 5）解决后就可以装包

 
```
rpm -Uvh mysql-community-*.rpm
```
 ![这里写图片描述](https://img-blog.csdn.net/20171122131145663?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvTmVkdmVkX0w=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

 提及下MySQL配置文件   
 主配置文件: /etc/my.cnf   
 默认存储数据目录: /var/lib/mysql   
 默认监听端口: 3306   
 日志文件: /var/log/mysqld.log

 ps -C 进程名 //查看进程   
 rpm -qf 命令 //查看命令由来

 可用ps 来看下你的数据库服务起来了没有

 
## 客户端把数据存储到数据库服务器的过程

 1.连接数据库服务器   
 2.选择库（文件夹）   
 3.选择/创建表（文件）   
 4.插入记录（数据）   
 5.断开连接

 
## 首次登陆MySQL

 确保MySQL服务已启动,服务启动后才会在/var/log/mysqld.log生成随机初始密码   
 1）查看初始密码

 
```
[root@BD4 09.mysql]# systemctl start mysqld
[root@DB4 09.mysql]#grep     -i         'password'  /var/log/mysqld.log  
2017-11-20T02:29:59.176287Z 1 [Note] A temporary password is generated for root@localhost: spalif)3uh/Q
```
 2)利用初始密码登陆

 
```
mysql -uroot -p"spalif)3uh/Q"
```
 3）修改密码（临时）

 
```
mysql> set global validate_password_policy=0;           //只验证
mysql> set global validate_password_length=6;           //修改密码长度默认值为6
mysql> alter user root@"localhost" identified by "123456";  //设置新密码
```
 4)修改配置文件（永久）   
 打开 /etc/my.cnf 添加   
 validate_password_policy=0   
 validate_password_length=6

 到这里最基本的搭建数据库服务已经完毕了，

   
  
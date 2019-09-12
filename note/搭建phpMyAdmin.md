---
title: 搭建phpMyAdmin
date: 2017-11-24 15:20:17
tags: CSDN迁移
---
 [ ](http://creativecommons.org/licenses/by-sa/4.0/) 版权声明：本文为博主原创文章，遵循[ CC 4.0 BY-SA ](http://creativecommons.org/licenses/by-sa/4.0/)版权协议，转载请附上原文出处链接和本声明。  本文链接：[https://blog.csdn.net/Nedved_L/article/details/78621694](https://blog.csdn.net/Nedved_L/article/details/78621694)   
    
  ## MySQL常见的管理工具

 ![这里写图片描述](https://img-blog.csdn.net/20171124094922784?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvTmVkdmVkX0w=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

 今天选择的phpMyAdmin   
 一款基于浏览器管理数据库的工具。   
 下载可以去官网下载[https://files.phpmyadmin.net/phpMyAdmin/4.7.5/phpMyAdmin-4.7.5-all-languages.tar.gz](https://files.phpmyadmin.net/phpMyAdmin/4.7.5/phpMyAdmin-4.7.5-all-languages.tar.gz)

 
## 准备环境

 httpd php php-mysql 

 
```
[root@DB3 ~]#yum install httpd php php-mysql -y
```
 
## 解包并指定释放路径

 phpMyAdmin-4.7.5-all-languages.tar.gz

 
```
[root@DB3 ~]#tar -xvf phpMyAdmin-4.7.5-all-languages.tar.gz -C /var/www/html/

```
 
## 重命名解压得到目录以方便访问

 
```
[root@DB3 html]# mv phpMyAdmin-4.7.5-all-languages phpadmin
```
 
## 安全起见更改权限

 
```
[root@DB3 html]# chown -R apache:apache phpadmin
```
 
## 准备phpmyadmin配置文件

 
```
[root@DB3 phpadmin]# cp config.sample.inc.php config.inc.php 
```
 
## 修改配置文件

 修改config.inc.php的第17行和31行

 
```
[root@DB3 phpadmin]# vim config.inc.php
```
 
```
 17 $cfg['blowfish_secret'] = 'heya'; /* YOU MUST FILL IN THIS FOR COOKIE AUTH! */
 31 $cfg['Servers'][$i]['host'] = 'localhost';
```
 17行的需要在$cfg[‘blowfish_secret’] = ’ ’ 在两单引号间随便添加字符，作用是COOKIE缓存验证。   
 31行则是改动[‘host’] = ‘localhost’; localhost改为你数据库的IP地址，内德的数据是在本机上所以就时localhost啦。

 
## 起服务，并在客户端测试

 
```
[root@DB3 phpadmin]#systemctl restart httpd
```
 客户端测试，就随便找一台能够连上刚设置的机子测试就可以了   
 在浏览器中输入http://机子的IP/phpadmin/

   
  
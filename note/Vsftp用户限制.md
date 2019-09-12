
 要求

 1.可以登录服务器

 2.可以拉取dmp文件

 3.仅限在dmp文件的目录下,不能cd其他路径,ls其他目录

 

 解决过程

 yum 安装ftp服务

 
> [root@78778e06dc0a /]# yum install vsftpd -y
> 
>  
 

 修改vsftp配置文件,开启限制

 
> [root@78778e06dc0a /]# vim /etc/vsftpd/vsftpd.conf
> 
>  最后加上以下几行:  
>  chroot_list_enable=YES  
>  chroot_local_user=YES  
>  chroot_list_file=/etc/vsftpd/chroot_list  
>  user_config_dir=/etc/vsftpd/user.d
> 
>  
 **vsftpd.conf配置文件说明** 

 
> >  anonymous_enable=YES #设置是否允许匿名用户登录  
>  local_enable=YES #设置是否允许本地用户登录  
>  local_root=/home #设置本地用户的根目录  
>  write_enable=YES #是否允许用户有写权限  
>  local_umask=022 #设置本地用户创建文件时的umask值  
>  anon_upload_enable=YES #设置是否允许匿名用户上传文件  
>  anon_other_write_enable=YES #设置匿名用户是否有修改的权限  
>  anon_world_readable_only=YES #当为YES时，文件的其他人必须有读的权限才允许匿名用户下载，单单所有人为ftp且有读权限是无法下载的，必须其他人也有读权限，才允许下载  
>  download_enbale=YES #是否允许下载  
>  chown_upload=YES #设置匿名用户上传文件后修改文件的所有者  
>  chown_username=ftpuser #与上面选项连用，表示修改后的所有者为ftpuser  
>  ascii_upload_enable=YES #设置是否允许使用ASCII模式上传文件  
>  ascii_download_enable=YES #设置是否允许用ASCII模式下载文件
> 
>  chroot_local_user=YES #设置是否锁定本地用户在自己的主目录中，（登录后无法cd到父目录或同级目录中）  
>  chroot_list_enable=YES #设置是否将用户锁定在自己的主目录中  
>  chroot_list_file=/etc/vsftpd/chroot_list #定义哪些用户将会锁定在自己的主目录中  
>  userlist_enable=YES #当为YES时表示由userlist_file文件中指定的用户才能登录ftp服务器  
>  userlist_file=/etc/vsftpd/user_list #当userlist_enable为YES时才生效
> 
>  
> 
>  
 为每个用户建立对应的配置文件 

 
> >  如  
>  [root@78778e06dc0a /]#mkdir /etc/vsftpd/user.d  
>  [root@78778e06dc0a /]#vim /etc/vsftpd/user.d/ftp  
>  加入以下内容  
>  local_root=/home/ftp
> 
>  
 

 然后启动ftp服务

 
> >  [root@78778e06dc0a /]#service vsftpd start  
>  
> 
>  
 登录测试

 问题

 
> 1. 500 OOPS: could not bind listening IPv4 socket  
>  原因:因为同时指定了 inetd和standalone 两种运行方式,端口冲突了.  
>  解决方法:  
>  1).使用XINET模式  
>  去掉/etc/rc.local文件中的vsftpd的启动脚本/usr/local/sbin/vsftp &;  
>  重启xinetd服务, service xinetd restart  
>  运行service vsftpd restart命令启动vsftpd
> 
>  2).使用STANDALONE独立模式  
>  在服务器的负担比较重的情况下最好用这个模式  
>  或者直接修改/etc/xinetd.d/vsftpd文件,把disable=no改成disable=yes就行了!
> 
>  
> 
>  
 

 

 
> 2. service vsftpd start时出现Starting vsftpd for vsftpd: [ FAILED ]  
>  修改/etc/logrotate.d/vsftpd.log  
>  把 missingok 注释掉
> 
>  3. 500 OOPS: cannot change directory:/home/...
> 
>  sestatus -b | grep ftp  
>  setenforce 0  
>  setenforce: SELinux is disabled  
>  [root@78778e06dc0a /]# getenforce  
>  Disabled  
>  [root@78778e06dc0a /]# setsebool -P ftp_home_dir=1  
>  setsebool: SELinux is disabled.  
>  [root@78778e06dc0a /]# setsebool -P allow_ftpd_full_access 1  
>  setsebool: SELinux is disabled.
> 
>  再次尝试,OK
> 
>  
 

 

   
 
 6.可通过Web页面显示对象状态，日志，警告信息

 
## Nagios 可以监控的功能有：

 1、监控网络服务（SMTP、POP3、HTTP、NNTP、PING等）；   
 2、监控主机资源（处理器负荷、磁盘利用率等）；   
 3、简单地插件设计使得用户可以方便地扩展自己服务的检测方法；   
 4、并行服务检查机制；   
 5、具备定义网络分层结构的能力，用”parent”主机定义来表达网络主机间的关系，这种关系可被用来发现和明晰主机宕机或不可达状态；   
 6、当服务或主机问题产生与解决时将告警发送给联系人（通过EMail、短信、用户定义方式）；   
 7、可以定义一些处理程序，使之能够在服务或者主机发生故障时起到预防作用；   
 8、自动的日志滚动功能；   
 9、可以支持并实现对主机的冗余监控；   
 10、可选的WEB界面用于查看当前的网络状态、通知和故障历史、日志文件等；   
 11、可以通过手机查看系统监控信息；   
 12、可指定自定义的事件处理控制器；

 
## 部署Nagios

 1.环境准备   
 gcc gcc-c++ php httpd nagios压缩包   
 2.起http服务，并测试php是否可用   
 3.添加nagios用户与组   
 useradd nagios   
 groupadd nagcmd   
 usermod -G nagcmd nagios   
 4.解压nagios.zip   
 5.编译安装   
 tar -xvzf nagios-4.2.4.tar.gz   
 ./configure –with-nagios-user=nagios –with-nagios-group=nagcmd –with-command-user=nagios –with-command-group=nagcmd   
 make all //准备安装环境   
 make install //安装主程序program, CGIs, and HTML files   
 make install-init //安装启动程序(修改权限)   
 make install-commandmode //安装命令行   
 make install-config //安装配置文件   
 make install-webconf //安装nagios与http工作时的配置文件   
 make install-exfoliation或者install-classicui //Web页面显示样式

 
## 安装nagios-plugins

 提供监控插件的软件包   
 nagios-plugins-2.1.4.tar.gz   
 1.解包   
 2.编译安装   
 ./configure&&make&&make install   
 ls /usr/local/nagios/libexec/check_* //验证

 起服务

 /etc/rc.d/init.d/nagios status   
 /etc/rc.d/init.d/nagios start   
 echo “/etc/rc.d/init.d/nagios start” >>/etc/rc.local //设置开机自启

 测试   
 1）重启httpd(重新加载nagios配置文件)   
 2）设置访问监控页面管理员nagiosadmin的密码   
 3）可在nagios配置文件找到密码放置目录/etc/httpd/conf.d/nagios.conf 在第34行

 34 AuthUserFile /usr/local/nagios/etc/htpasswd.users

 4）手动生成密码文件   
 htpasswd -c 密码放置路径 用户   
 htpasswd -c /usr/local/nagios/etc/htpasswd.users nagiosadmin   
 5）访问监控服务器查看   
 firefox [http://127.0.0.1/nagios](http://127.0.0.1/nagios)

   
  
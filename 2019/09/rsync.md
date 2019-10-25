#  Linux下Rsync+sersync实现数据实时同步



## 前言



### sersync介绍

sersync主要用于服务器同步，web镜像等功能。

基于boost1.43.0,inotify api,rsync command.开发。

目前使用的比较多的同步解决方案是inotify-tools+rsync ，另外一个是google开源项目Openduckbill（依赖于inotify- tools），这两个都是基于脚本语言编写的。



### sersync优点

1. sersync是使用c++编写，而且对linux系统文件系统产生的临时文件和重复的文件操作进行过滤，所以在结合rsync同步的时候，节省了运行时耗和网络资源。因此更快。
2.  sersync配置起来很简单，其中bin目录下已经有基本上静态编译的2进制文件，配合bin目录下的xml配置文件直接使用即可。
3. sersync相比较其他脚本开源项目，使用多线程进行同步，尤其在同步较大文件时，能够保证多个服务器实时保持同步状态。 
4. sersync有出错处理机制，通过失败队列对出错的文件重新同步，如果仍旧失败，则按设定时长对同步失败的文件重新同步。 
5. sersync自带crontab功能，只需在xml配置文件中开启，即可按您的要求，隔一段时间整体同步一次。无需再额外配置crontab功能。 
6. sersync的socket与http插件扩展，满足二次开发的需要。



### Rsync+Inotify-tools与Rsync+sersync的区别

1. Rsync+Inotify-tools

   >1.Inotify-tools只能记录下被监听的目录发生了变化（包括增加、删除、修改），并没有把具体是哪个文件或者哪个目录发生了变化记录下来；
   >
   >2.rsync在同步的时候，并不知道具体是哪个文件或者哪个目录发生了变化，每次都是对整个目录进行同步，当数据量很大时，整个目录同步非常耗时（rsync要对整个目录遍历查找对比文件），因此，效率很低。

2. Rsync+sersync

   >1.sersync可以记录下被监听目录中发生变化的（包括增加、删除、修改）具体某一个文件或某一个目录的名字；
   >
   >2.sync在同步的时候，只同步发生变化的这个文件或者这个目录（每次发生变化的数据相对整个同步目录数据来说是很小的，rsync在遍历查找比对文件时，速度很快），因此，效率很高。

**小结**

> 当同步的目录数据量不大时，建议使用Rsync+Inotify-tools；当数据量很大（几百G甚至1T以上）、文件很多时，建议使用Rsync+sersync。

[取自系统运维](https://www.osyunwei.com/archives/7447.html)



部署总结：

>
>步骤一、 配置slave上的rsync服务
>步骤二、 配置master上sersync客户端



## 1.0 关闭 SElinux 和防火墙

```bash
sed -i  "s/SELINUX=enforcing/SELINUX=disabled/"  /etc/selinux/config
systemctl  stop firewalld
systemctl disable firewalld
```





## 2.0 Salve服务器上安装Rsync服务

```bash
yum install -y rsync
```



### 2.1 配置rsync.conf

```bash
cat > /etc/rsync/rsyncd.conf <<EOF
# Minimal configuration file for rsync daemon
# See rsync(1) and rsyncd.conf(5) man pages for help

# This line is required by the /etc/init.d/rsyncd script
# GLOBAL OPTIONS
uid = root                         
gid = root      

#Base env
log file = /var/log/rsyncd.log 
pidfile = /var/run/rsyncd.pid  
lock file = /var/run/rsync.lock
#User auth file
secrets file = /etc/rsync/rsyncd.secrets
#Welcome file  
motd file = /etc/rsync/rsyncd.Motd

uid = root
gid = root 
use chroot = no 
read only = no
list = no
port=873

#This will log every file transferred - up to 85,000+ per user, per sync
transfer logging = yes
log format = %t %a %m %f %b
syslog facility = local3
max connections = 200 
timeout = 600  
auth users = root

#limit access to private LANs
hosts allow = 172.10.128.200/16
hosts deny = *

# MODULE OPTIONS
[nedved]
comment = nedved
path = /RMS_PUB/

EOF
```

> rsync服务的配置文件表明允许sersync主服务器(IP为192.168.0.100)访问，
>
> rsync同步模块名为[nedved]将同步过来的文件分别放入对应的path指定的目录/RMS_PUB/
>
> 有多台目标服务器，则每台都需要进行类似的rsync服务配置，上面的uid，gid要换成本服务器对应的同步用户。
>
> 注意rsync服务账户(上述用了root用户)要有对被同步目录(/RMS_PUB/)的写入更新权限。



### 2.2 创建同步目录

```bash
mkdir -p /RMS_PUB/
```





### 2.3 配置用户认证文件

```bash
mkdir /etc/rsync
cat > /etc/rsync/rsyncd.pass <<EOF
rsyncd:rsyncd!@#
EOF

chmod 600 /etc/rsyncd.conf
chmod 600 /etc/rsync/rsyncd.pass
```



### 2.4 启动并设置开机自启

#### 针对centos7+

> 使用systemctl cat rsyncd
>
> 可以看到默认的systemctl 配置文件是指定了(ConditionPathExists)检查配置文件是否存在
>
> 和提供了一个(EnvironmentFile)启动选项配置文件

```bash
[root@lab ~]# systemctl cat rsyncd
# /usr/lib/systemd/system/rsyncd.service
[Unit]
Description=fast remote file copy program daemon
ConditionPathExists=/etc/rsyncd.conf

[Service]
EnvironmentFile=/etc/sysconfig/rsyncd
ExecStart=/usr/bin/rsync --daemon --no-detach "$OPTIONS"

[Install]
WantedBy=multi-user.target

#指定配置文件检查路径
sed -i '/^Co/s#tc/#tc/rsync/#' /usr/lib/systemd/system/rsyncd.service
#指定加载配置文件路径
sed -i 's#=""#="--config=/etc/rsync/rsyncd.conf"#' /etc/sysconfig/rsyncd

systemctl enable rsyncd
systemctl start rsyncd
ss -antpul | grep 873
```



#### 针对centos 6以下

启动

```bash
rsync --daemon --config=/etc/rsync/rsyncd.conf
ps -ef|grep rsync
netstat -tunpl|grep :873
```

开机自启

```bash
echo "/usr/bin/rsync --daemon --config=/etc/rsync/rsyncd.conf" >> /etc/rc.local
```

重启

```bash
pkill rsync
rsync --daemon --config=/etc/rsync/rsyncd.conf
ps -ef |grep rsync
```



## 3.0  Master上配置rsync客户端



### 3.1 Master上配置rsync权限

```bash
echo 'rsyncd!@#' > /etc/rsync.password
chmod 600 /etc/rsync.password 
cat /etc/rsync.password 
ll /etc/rsync.password
```



### 3.2 Master上测试rsync同步情况

> 在后面在master上进行部署sersync，sersync主服务器上必须要确保可以把文件salve上

```bash
#创建同步目录
mkdir -p /var/rsync
touch /var/rsync/1

#执行同步命令(注意防火墙)
rsync -avzP /var/rsync rsyncd@172.10.128.200::nedved/ --password-file=/etc/rsync.password
```






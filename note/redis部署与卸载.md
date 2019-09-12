```
 2、解压  
 

 
```
 tar -xvf redis-4.0.10.tar.gz
```
 3、cp到/usr/local下 (纯属个人习惯)  
 

 
```
 cp redis-4.0.10 /usr/local/redis

cd /usr/local/redis
```
 4、编译

 
> [root@Service redis]# pwd  
>  /usr/local/redis
> 
>  
 
```
 make
cd src && make install
```
 5、可以看到在src目录下生成了几个新的文件。并且在/usr/local/bin下已经有了redis相关程序

 
> [root@localhost redis]# ll -tr src  
>  -rw-rw-r--. 1 root root 3779 7月 24 22:58 zmalloc.h  
>  .  
>  .  
>  .  
>  -rwxr-xr-x. 1 root root 5768648 8月 3 10:05 redis-server  
>  -rwxr-xr-x. 1 root root 5768648 8月 3 10:05 redis-sentinel  
>  -rw-r--r--. 1 root root 396768 8月 3 10:05 redis-cli.o  
>  -rwxr-xr-x. 1 root root 2617232 8月 3 10:05 redis-cli  
>  -rw-r--r--. 1 root root 109120 8月 3 10:05 redis-benchmark.o  
>  -rwxr-xr-x. 1 root root 2451208 8月 3 10:05 redis-benchmark  
>  -rwxr-xr-x. 1 root root 5768648 8月 3 10:05 redis-check-rdb  
>  -rwxr-xr-x. 1 root root 5768648 8月 3 10:05 redis-check-aof  
>  -rw-r--r--. 1 root root 16088 8月 3 10:06 Makefile.dep
> 
>  
> 
>  [root@Service src]# ll /usr/local/bin/  
>  总用量 21860  
>  -rwxr-xr-x. 1 root root 2451208 8月 3 10:42 redis-benchmark  
>  -rwxr-xr-x. 1 root root 5768648 8月 3 10:42 redis-check-aof  
>  -rwxr-xr-x. 1 root root 5768648 8月 3 10:42 redis-check-rdb  
>  -rwxr-xr-x. 1 root root 2617232 8月 3 10:42 redis-cli  
>  lrwxrwxrwx. 1 root root 12 8月 3 10:42 redis-sentinel -> redis-server  
>  -rwxr-xr-x. 1 root root 5768648 8月 3 10:42 redis-server
> 
>  
> 
>  
 6、修改配置文件

 先做一个链接(个人习惯)

 
```
 mkdir /etc/redis
ln -s /usr/local/redis/redis.conf /etc/redis/redis.conf

```
 

 redis默认启动是会挂在前台的，若没有修改配置文件启动就需要加&

 所以就设置为后台启动。

 在redis.conf的配置文件里面。做如下的修改：

 
> vim /etc/redis/redis.conf
> 
>  daemonize no
> 
>  修改为：
> 
>  daemonize yes
> 
>  
 7、设置开机自启

 要先让redis服务自动启动的话，首先需要在/etc/init.d目录下创建redis的启动脚本。

 将redis安装目录下的utils/redis_init_script复制到/etc/init.d目录下，命名为redis

 
```
 cp utils/redis_init_script /etc/init.d/redis
chmod 755 /etc/init.d/redis
```
 脚本修改,修改其中指定的pid和配置文件。 

 
> vim
> 
>  PIDFILE=/var/run/redis_${REDISPORT}.pid
> 
>  CONF="/etc/redis/${REDISPORT}.conf"
> 
>  
> 
>  修改为
> 
>  PIDFILE=/var/redis/run/redis_${REDISPORT}.pid
> 
>  CONF="/etc/redis/redis.conf"
> 
>  
> 
>  
> 
>  
> 
>  
> 
>  
> 
>  
 创建存放pid的目录为/var/redis/run 

 
```
 mkdir -p /var/redis/run

```
 修改redis.conf配置文件

 
> vim /etc/redis/redis.conf
> 
>  pidfile /var/run/redis_6379.pid
> 
>  修改为
> 
>  pidfile /var/redis/run/redis_6379.pid
> 
>  
 现在我们已经可以通过service redis start/stop来启动和关闭redis服务了。

 最后只需要通过chkconfig redis on命令来设置开机启动即可。

 如果提示redis 服务不支持 chkconfig的话，只需要在/etc/init.d/redis这个启动脚本的第二行后面加上下面的内容即可。

 
> vim /etc/init.d/redis
> 
>  # chkconfig:2345 90 10  
>  # description:Redis is a persistent key-value database
> 
>  
> 
>  
 8、启动redis,并设置开机启动

 
```
 chkconfig redis on
service redis start
```
 9、卸载

 首先把redis服务关闭

 
```
 service redis stop
```
 确认是否已关闭

 
> [root@Service ~]# ps -elf | grep redis  
>  0 S root 3307 2947 0 80 0 - 28180 - 11:44 pts/1 00:00:00 grep --color=auto redis
> 
>  
 由于redis命令都安装到/usr/local/bin目录下面了，并且添加到环境变量PATH里面了，所以可以直接运行。

 删除make的时候生成的几个redisXXX的文件

 
```
 rm -f /usr/local/bin/redis*
rm -rf /usr/local/redis
rm -f /etc/redis
rm -f /var/redis
```
 redis就卸载完成了。 

 

 

 

   
 
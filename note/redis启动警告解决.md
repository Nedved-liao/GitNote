   ## 背景

 最近在测试环境重启后,redis启动遇到了三个警告

 
> 第一个警告：The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.  
>  第二个警告：overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to/etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.  
>  第三个警告：you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix thisissue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain thesetting after a reboot. Redis must be restarted after THP is disabled.
> 
>  
 
## 解决方案

 第一个警告

 
> echo 'echo 511 > /proc/sys/net/core/somaxconn' >> /etc/rc.local  
>  source /etc/rc.local
> 
>  
 第二个警告

 
> echo "vm.overcommit_memory = 1" >> /etc/sysctl.conf  
>  sysctl -p
> 
>  
 第三个警告

 
> echo 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' >> /etc/rc.local  
>  source /etc/rc.local
> 
>  
 其实redis已经做到很详细了,在报错中已经给出了详细的方案

 

 
# 详细参考

 报错一: [http://blog.csdn.net/raintungli/article/details/37913765](http://blog.csdn.net/raintungli/article/details/37913765)

 报错二: [http://blog.csdn.net/whycold/article/details/21388455](http://blog.csdn.net/whycold/article/details/21388455)

 报错三: [http://www.cnblogs.com/kerrycode/archive/2015/07/23/4670931.html](http://www.cnblogs.com/kerrycode/archive/2015/07/23/4670931.html)

 

   
 
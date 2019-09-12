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

 

   
 
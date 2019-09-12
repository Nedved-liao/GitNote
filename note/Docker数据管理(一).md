---
title: Docker数据管理(一)
date: 2018-08-27 22:07:14
tags: CSDN迁移
---
 [ ](http://creativecommons.org/licenses/by-sa/4.0/) 版权声明：本文为博主原创文章，遵循[ CC 4.0 BY-SA ](http://creativecommons.org/licenses/by-sa/4.0/)版权协议，转载请附上原文出处链接和本声明。  本文链接：[https://blog.csdn.net/Nedved_L/article/details/82119761](https://blog.csdn.net/Nedved_L/article/details/82119761)   
    
   # 数据卷挂载

 在生产环境中,需要对数据进行持久化,冗余化,或者在需要在多个容器之间进行数据共享

 数据卷:容器内数据直接映射到本地主机环境

 数据卷容器:使同特定容器维护数据卷

 -v 进行映射

 
## 1.在容器内生成一个数据卷

 
> docker run -id --rm --name dbdata -v dbdata docker.io/busybox
> 
>  
 
## 2.挂载主机目录作为数据卷, 将webroot挂载到容器的test中(绝对路径)

 
> docker run -id --rm --name -P web -v /root/webroot:/test docker.io/busybox
> 
>  
 
## 3.挂载一个本机文件作为数据卷, 将web.xml挂载到容器的test中(不推荐)

 
> docker run -id --rm --name -P web -v /root/web.xml:/test docker.io/busybox
> 
>  
 
## 总结:

 如果使用文件挂载,当使用vim或者sed --in-place时候,可能造成inode改变,所以不推荐以文件挂载

 
# 数据卷容器

 生成一个专门放数据的容器,这个数据卷容器可以在多个容器之间共享一些持续更行的数据

 
## 1.生成数据卷容器

 
> docker run -it --name dbdata -v /dbdata docker.io/busybox
> 
>  查看结果  
>  / # ls  
>  bin dbdata dev etc home proc root run sys tmp usr var
> 
>  
 
## 2.创建其他容器,其实可用到 --volumes-from来挂载dbdata容器中的数据卷

 
> docker run -it --name web1 --volumes-from dbdata docker.io/busybox  
>  docker run -it --name web2 --volumes-from dbdata docker.io/busybox
> 
>  
 在其中一个容器中创建一个文件,可以在另外两个看到

 
## 总结:

 可以多次使用--volumes-from来挂载dbdata,也可以从其他已经挂载的容器卷的容器挂载数据卷

 如果删除了挂载的容器(包括dbdata,web1,web2),数据卷并不会被删除.只有删除最后一个还挂载着它的容器 显示使用docker rm -v 命令来指定 同时删除关联的容器

 
# 利用数据卷来迁移数据

 利用数据容器对其中的数据卷进行备份,恢复以实现数据迁移

 
## 1.备份

 
> >  docker run --volumes-from dbdata -v /root/back:/backup --name back docker.io/busybox tar -cvf /backup/backup.tar /dbdata
> 
>  
 利用目录挂载,就可以把备份放到物理机的/root/back里了

 
## 2.恢复

 
> >  docker run --volumes-from dbdata -v /root/back:/backup --name recover docker.io/busybox tar -xvf /backup/backup.tar
> 
>  
 总结:  
 通过数据卷和数据卷容器对容器内数据进行共享,备份,恢复等操作,即使出现了运行故障,用户也不必担心数据丢失,只需要快速创建容器即可  
 在生产环境中,定期在物理机上进行数据备份,使用支持容错的存储系统(RAID,分布式文件系统{Ceph,GPFS,HDFS}).可以大大提升数据安全

 

 入门容器操作见[https://blog.csdn.net/Nedved_L/article/details/79067732](https://blog.csdn.net/Nedved_L/article/details/79067732)

   
   
   
   
 
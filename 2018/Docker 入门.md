   Docker 是一个[开源](https://baike.baidu.com/item/%E5%BC%80%E6%BA%90/246339)的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的容器中，然后发布到任何流行的[Linux](https://baike.baidu.com/item/Linux) 机器上，也可以实现[虚拟化](https://baike.baidu.com/item/%E8%99%9A%E6%8B%9F%E5%8C%96)。容器是完全使用[沙箱](https://baike.baidu.com/item/%E6%B2%99%E7%AE%B1/393318)机制，相互之间不会有任何接口





Docker

优点：

更加轻量简洁高效

传统虚拟机需要给每个VM安装系统，Docker技术代替安装系统

容器使用共享公共库和程序

缺点：

容器的隔离性没有虚拟化强

共用Linux内核，安全性有先天缺陷

SElnux难以控制

监控容器和容器排错麻烦



镜像：

在Docker中容器是基于镜像启动的

镜像是启动容器的核心

镜像采用分层设计

使用快照的COW技术，确保底层数据不丢失

  


Docker基本操作

docker images //查看系统镜像

docker history //查看镜像制作历史

docker inspect //查看镜像底层信息

docker search //从官方源搜索镜像

docker pull //下载镜像

docker push //上传镜像

docker rmi //删除本地镜像

docker tag //修改镜像名称和标签

docker save busybox >busybox.tar //导出

docker load <busybox.tar //导入



容器相关命令

docker run -it centos bash //以交互模式启动一个容器

docker run -itd centos bash //启动的容器放在后台

docker ps //显示正在运行的容器

docker ps -a //显示所有容器

docker ps -aq //显示所有容器，单只显示 id

docker start|stop|restart 容器id //启动，停止，重启容器

docker exec -it 容器id /bin/bash //进入容器并执行/bin/bash

docker inspect 容器id  //显示容器详细信息

docker top //容器id查看容器内运行的进程

docker rm //容器id删除容器

docker stop $(docker ps -aq) //关闭所有容器

docker rm $(docker ps -aq) //删除所有容器

docker attach 容器id //进入容器





  
   
 
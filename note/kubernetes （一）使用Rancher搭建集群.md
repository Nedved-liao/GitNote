   **目录**

 

 [如何快速高效部署K8s集群](#%E5%A6%82%E4%BD%95%E5%BF%AB%E9%80%9F%E9%AB%98%E6%95%88%E9%83%A8%E7%BD%B2K8s%E9%9B%86%E7%BE%A4)

 [Rancher是什么](#Rancher%E6%98%AF%E4%BB%80%E4%B9%88)

 [为什么是Rancher](#%E4%B8%BA%E4%BB%80%E4%B9%88%E6%98%AFRancher)

 [1.0、安装Rancher](#%E5%AE%89%E8%A3%85Rancher)

 [1.1、环境](#%E7%8E%AF%E5%A2%83)

 [1.2、选择Rancher版本](#%C2%A0%E9%80%89%E6%8B%A9Rancher%E7%89%88%E6%9C%AC)

 [1.3、拉取镜像](#%E6%8B%89%E5%8F%96%E9%95%9C%E5%83%8F)

 [2.0、容器启动高级选项](#%E5%AE%B9%E5%99%A8%E5%90%AF%E5%8A%A8%E9%AB%98%E7%BA%A7%E9%80%89%E9%A1%B9)

 [2.1、SSL加密方式访问Rancher](#%E9%80%89%E6%8B%A9SSL%E5%8A%A0%E5%AF%86%E6%96%B9%E5%BC%8F%E8%AE%BF%E9%97%AE%EF%BC%88%E5%8F%AF%E9%80%89%EF%BC%89)

 [默认自签名证书：](#%E9%BB%98%E8%AE%A4%E8%87%AA%E7%AD%BE%E5%90%8D%E8%AF%81%E4%B9%A6%EF%BC%9A)

 [自定义自签名证书：](#%E8%87%AA%E5%AE%9A%E4%B9%89%E8%87%AA%E7%AD%BE%E5%90%8D%E8%AF%81%E4%B9%A6%EF%BC%9A)

 [2.2、启用API审核日志](#%E5%90%AF%E7%94%A8API%E5%AE%A1%E6%A0%B8%E6%97%A5%E5%BF%97)

 [2.3、Air Gap](#air-gap)

 [2.4、持久化数据](#%E6%8C%81%E4%B9%85%E5%8C%96%E6%95%B0%E6%8D%AE)

 [3.0、启动容器](#%E5%90%AF%E5%8A%A8%E5%AE%B9%E5%99%A8)

 [4.0、访问UI](#%E8%AE%BF%E9%97%AEUI)

 [5.0、Rancher多节点HA部署](#h2_5)

 [5.1、准备：](#%E5%87%86%E5%A4%87%EF%BC%9A)

 [5.2、部署需求：](#%E9%83%A8%E7%BD%B2%E9%9C%80%E6%B1%82%EF%BC%9A)

 [HA 节点](#HA%20%E8%8A%82%E7%82%B9)

 [MySQL数据库](#MySQL%E6%95%B0%E6%8D%AE%E5%BA%93)

 [外部负载均衡服务器](#%E5%A4%96%E9%83%A8%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1%E6%9C%8D%E5%8A%A1%E5%99%A8)

 [5.3、HA模式下的RANCHER SERVER节点](#h3_7)

 
--------


 
# 如何快速高效部署K8s集群

 众所周知的K8s集群的部署非常麻烦而且坑又多，为了提升效率必须借助一些工具

 
# Rancher是什么

 Rancher是一个开源的企业级容器管理平台。通过Rancher，企业再也不必自己使用一系列的开源软件去从头搭建容器服务平台。Rancher提供了在生产环境中使用的管理Docker和Kubernetes的全栈化容器部署与管理平台。

 
# 为什么是Rancher

 目前创建K8S集群的安装程序最受欢迎的有Kops，Kubespray，kubeadm，rancher，以及个人提供的脚本集等。

 (对于Kubernetes容器化的方式运行，容灾恢复比在物理机优秀。所以忽略二进制部署在物理机上)

 
  * Kops和Kubespary在国外用的比较多，国内因为GFW原因使用不多。 
  * kubeadm是Kubernetes官方提供的k8s部署工具，不过不支持HA，且支持的docker版本、K8S版本也有限，因此也不是生产首选的安装程序。 
  * Rancher2016年的新起之秀，可以做到极简快速部署管理Docker，并支持多种编排方式：Cattle、Kubernetes、Mesos、Swarm等。通过修改镜像库的方式可以实现在国内的使用。所以我们选择Rancher作为Docker管理部署框架。 

 Kubernetes官方文档:[https://kubernetes.io/docs/reference/](https://kubernetes.io/docs/reference/)

 Kubernetes官方Git地址:[https://github.com/kubernetes/kubernetes](https://github.com/kubernetes/kubernetes)

 Rancher官方地址: [https://www.cnrancher.com/](https://www.cnrancher.com/)

 

 
# 1.0、安装Rancher

 
## 1.1、环境

 Centos7.4 

 docker 18.06.1-ce

 Rancher 2.0  
 三台服务器

 
> k8s-master 192.168.146.50
> 
>  k8s-node1 192.168.146.51
> 
>  k8s-node1 192.168.146.52
> 
>  
 
## 1.2、选择Rancher版本

 Rancher Server 镜像版本有两个latest 和stable

 
     Tag                            | Description                                 
     ------------------------------ | -------------------------------------------- 
     rancher/rancher:latest         | 最新的开发版本。这些版本是通过我们的CI自动化框架进行验证。这些版本不推荐用于生产环境。
     rancher/rancher:stable         | 最新的稳定版本。这个标签是生产建议。                          
     rancher/rancher:&lt;v2.X.X&gt; | 你可以安装特定版本的Rancher，在DockerHub上查看那些tag可用。     

**Notes：**

 
> **" **不要使用任何带有 rc{n} 前缀的release，这些是Rancher测试团队使用的
> 
>  
 本文使用rancher/server:v2.0.0-beta4

 
## 1.3、拉取镜像

 
```
 [root@K8s-master ~]# docker pull registry.docker-cn.com/rancher/server:v2.0.0-beta4 
v2.0.0-beta4: Pulling from rancher/server
f56c16792d1e: Pull complete 
e2f3fcbcaacd: Pull complete 
29f1503c00f8: Pull complete 
b5ce6675ff39: Pull complete 
a29a9bdc981f: Pull complete 
4758641d8c60: Pull complete 
a30a6a05c4ed: Pull complete 
1ec93119bf5e: Pull complete 
74bc3b84fcb6: Pull complete 
567d9245728b: Pull complete 
Digest: sha256:09507919e8c16395393ab170b3129d3919ab6797d8f7fc1bdfc94cb4f989c151
Status: Downloaded newer image for registry.docker-cn.com/rancher/server:v2.0.0-beta4

```
 
# 2.0、容器启动高级选项

 
## 2.1、SSL加密方式访问Rancher

 处于安全的目的，我们选择通过https方式

 SSL可以保护所有Rancher网络通信，例如登录或与群集交互时。

 **证书生成方式：**

 
  * 默认自签名证书 
  * 自定义自签名证书 
### **默认自签名证书：**

 
> #直接暴露端口运行就可以
> 
>  docker run -d --restart=unless-stopped -p 80:80 -p 443:443 rancher/rancher:latest
> 
>  
 
### **自定义自签名证书：**

 使用功能OpenSSl生成

 
> #生成根证书私钥(pem文件)   
>  openssl genrsa -out cakey.pem 2048   
>  #生成根证书签发申请文件(csr文件)   
>  openssl req -new -key cakey.pem -out ca.csr -subj "/C=CN/ST=myprovince/L=mycity/O=myorganization/OU=mygroup/CN=myCA"   
>  #自签发根证书(cer文件)   
>  openssl x509 -req -days 365 -sha1 -extensions v3_ca -signkey cakey.pem -in ca.csr -out cacert.pem  
>  #生成服务端私钥   
>  openssl genrsa -out key.pem 2048   
>  #生成证书请求文件   
>  openssl req -new -key key.pem -out server.csr -subj "/C=CN/ST=myprovince/L=mycity/O=myorganization/OU=mygroup/CN=myServer"  
>  #使用根证书签发服务端证书   
>  openssl x509 -req -days 365 -sha1 -extensions v3_req -CA cacert.pem -CAkey cakey.pem -CAserial ca.srl -CAcreateserial -in server.csr -out cert.pem  
>  #使用CA证书验证server端证书   
>  openssl verify -CAfile cacert.pem cert.pem
> 
>  
 容器启动

 
> docker run -d --restart=unless-stopped \  
>  -p 80:80 -p 443:443 \  
>  -v /<CERT_DIRECTORY>/<FULL_CHAIN.pem>:/etc/rancher/ssl/cert.pem \  
>  -v /<CERT_DIRECTORY>/<PRIVATE_KEY.pem>:/etc/rancher/ssl/key.pem \  
>  -v /<CERT_DIRECTORY>/<CA_CERTS.pem>:/etc/rancher/ssl/cacerts.pem \  
>  rancher/rancher:latest
> 
>  
 

 
## 2.2、启用API审核日志

 API审核日志记录通过Rancher服务器进行的所有用户和系统事务。

 默认情况下，API审核日志会在rancher容器内写入/ var / log / auditlog。将该目录共享为卷并设置AUDIT_LEVEL以启用日志。

 
> docker run -d --restart=unless-stopped \  
>  -p 80:80 -p 443:443 \  
>  -v /var/log/rancher/auditlog:/var/log/auditlog \  
>  -e AUDIT_LEVEL=1 \  
>  rancher/rancher:latest
> 
>  
 

 
## 2.3、Air Gap

 If you are visiting this page to complete an air gap installation, you must pre-pend your private registry URL to the server tag when running the installation command in the option that you choose. Add  `<REGISTRY.DOMAIN.COM:PORT>`  with your private registry URL in front of  `rancher/rancher:latest` .

 **Example:**

 
```
  <REGISTRY.DOMAIN.COM:PORT>/rancher/rancher:latest
```
 --取值[官网](https://rancher.com/docs/rancher/v2.x/en/installation/single-node/)，不过没有弄明白作用是什么。。

 
## 2.4、持久化数据

 持久性数据位于容器中的/var/lib/rancher

 
> docker run -d --restart=unless-stopped \  
>  -p 80:80 -p 443:443 \  
>  -v /host/rancher:/var/lib/rancher \  
>  rancher/rancher:latest
> 
>  
 

 
# 3.0、启动容器

 启用ssl，api审核，数据持久化

 
```
 #环境准备
[root@K8s-master ~]# mkdir -p /home/rancher/{cert,log,data}
#生成证书过程省略
[root@K8s-master /home/rancher/cert]# ll
总用量 28
-rw-r--r-- 1 root root 1216 1月  25 15:10 cacert.pem
-rw-r--r-- 1 root root 1013 1月  25 15:10 ca.csr
-rw-r--r-- 1 root root 1679 1月  25 15:10 cakey.pem
-rw-r--r-- 1 root root   17 1月  25 15:11 ca.srl
-rw-r--r-- 1 root root 1224 1月  25 15:11 cert.pem
-rw-r--r-- 1 root root 1675 1月  25 15:10 key.pem
-rw-r--r-- 1 root root 1017 1月  25 15:10 server.csr
#生成容器
[root@K8s-master ~]#docker run -d --restart=unless-stopped \
	-p 8090:80 -p 8443:443 \
	-v /home/rancher/cert/cert.pem:/etc/rancher/ssl/cert.pem \
	-v /home/rancher/cert/key.pem:/etc/rancher/ssl/key.pem \
    -v /home/rancher/cert/cacert.pem:/etc/rancher/ssl/cacerts.pem \
	-v /home/rancher/log/auditlog:/var/log/auditlog \
	-e AUDIT_LEVEL=1 \
	-v /home/rancher/data/rancher:/var/lib/rancher \
	registry.docker-cn.com/rancher/server:v2.0.0-beta4
```
 等待容器启动完成访问对应IP的端口

 
# 4.0、访问UI

 权限管理，2.X 在第一次登录就会提示你写入密码

 ![](https://img-blog.csdnimg.cn/20190125161424589.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L05lZHZlZF9M,size_16,color_FFFFFF,t_70)

 

 通过右下角更改语言

 ![](https://img-blog.csdnimg.cn/20190125161927389.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L05lZHZlZF9M,size_16,color_FFFFFF,t_70)

 

 相对于1.6 认证减少了 ，现在只有两项了

 而且好像默认这是了允许本地数据库中的账号访问

 ![](https://img-blog.csdnimg.cn/20190125162301949.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L05lZHZlZF9M,size_16,color_FFFFFF,t_70)

 

 Rancher部署就先到这里了。

 

 说一下HA吧

 
# 5.0、Rancher多节点HA部署

 在高可用(HA)的模式下运行Rancher Server与使用外部数据库运行Rancher Server一样简单，需要暴露一个额外的端口，添加额外的参数到启动命令中，并且运行一个外部的负载均衡就可以了。

 HA Rancher安装了第4层负载均衡器，描述了入口控制器的SSL终止

 ![Rancher HA](https://rancher.com/docs/img/rancher/ha/rancher2ha.svg)

 

 --取值[官网](https://rancher.com/docs/rancher/v2.x/en/installation/ha/)

 

 
## **5.1、准备：**

 
  * [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl) - Kubernet中的命令行工具。 
  * [rke](https://rancher.com/docs/rke/v0.1.x/en/installation/) - Rancher Kubernetes Engine，用于构建Kubernetes集群的cli。 
  * [helm](https://docs.helm.sh/using_helm/#installing-helm) - Kubernetes的包管理。 **note：**

 
> **" **v2.12.0和cert-manager存在问题，请使用Helm v2.12.1或更高版本
> 
>  
 
## **5.2、****部署需求：**

 
  * ### HA 节点
    
     
    > >   * 1GB内存 
    >   * 9345, 8080 端口需要在各个节点之间能够互相访问 
    >   * 所有安装有支持的Docker版本的现代Linux发行版 RancherOS, Ubuntu, RHEL/CentOS 7 都是经过严格的测试。 
    >       * 对于 RHEL/CentOS, 默认的 storage driver, 例如 devicemapper using loopback, 并不被Docker推荐。 请参考Docker的文档去修改使用其他的storage driver。 
    >       * 对于 RHEL/CentOS, 如果你想使用 SELinux, 你需要 安装额外的 SELinux 组件.  
      
  * ### MySQL数据库
    
     
    > >   * 至少 1 GB内存 
    >   * 每个Rancher Server节点需要50个连接 (例如：3个节点的Rancher则需要至少150个连接) 
    >   * MYSQL配置要求 
    >       * 选项1: 用默认COMPACT选项运行Antelope 
    >       * 选项2: 运行MySQL 5.7，使用Barracuda。默认选项ROW_FORMAT需设置成Dynamic  
      
  * ### 外部负载均衡服务器
    
     
    > >   * 负载均衡服务器需要能访问Rancher Server节点的 8080 端口 
      

 在每个需要加入Rancher Server HA集群的节点上，运行以下命令：

 
```
 docker run -d --restart=unless-stopped -p 8080:8080 -p 9345:9345 rancher/server \
     --db-host DB_host.com --db-port 3306 --db-user username --db-pass password --db-name cattle \
     --advertise-address <Node IP>

```
 **note：**

 
> **" **每个节点的<Node IP> 唯一，这个IP会被添加到HA的设置中。
> 
>  **" **如果你修改了 -p 8080:8080 并在host上暴露了一个不一样的端口，你需要添加 --advertise-http-port <host_port> 参数到命令中。
> 
>  
 

 
## 5.3、HA模式下的RANCHER SERVER节点

 如果你的Rancher Server节点上的IP修改了，你的节点将不再存在于Rancher HA集群中。

 你必须停止在--advertise-address配置了不正确IP的Rancher Server容器并启动一个使用正确IP地址的Rancher Server的容器。

 

 **note：**

 
> **" **每一个Rancher Server节点需要有4 GB 或者8 GB的堆空间，意味着需要8 GB或者16 GB内存
> 
>  **" **MySQL数据库需要有高性能磁盘
> 
>  **" **对于一个完整的HA，建议使用一个有副本的Mysql数据库。另一种选择则是使用Galera集群并强制写入一个MySQL节点。
> 
>  
 

 **参考：**

 [https://rancher.com/docs/rancher/v2.x/en/overview/](https://rancher.com/docs/rancher/v2.x/en/overview/)

 [https://my.oschina.net/wenzhenxi/blog/1822075](https://my.oschina.net/wenzhenxi/blog/1822075)

 

   
 
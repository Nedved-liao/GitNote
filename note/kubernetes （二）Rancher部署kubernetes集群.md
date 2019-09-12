   V1.2 

 上一遍详细说了rancher的优缺还有在centos的部署，还没有到真题-快速部署kubernetes

 本来应该是第二天就写部署kubernetes集群的，但是遇到了个问题，Google花了大量时间没找到解决办法。

 问题是如下

 
> ** " **msg="Connecting to proxy" url="wss://192.168.146.10:8443/v3/connect/register" ... x509: cannotvalidate certificate for 192.168.146.10 because it doesn't contain any IP SANs
> 
>  
 咋一看这不是在搭建docker私有仓库时候的报错吗？

 好的，根据一般去设置了，但是最后依然报错这个。

 然后第三天就高烧40。。就一直搁在这里了,后来直接放弃了

 

 回归正题

 最后把启动rancher启动命令改为了如下

 
```
 docker run -d --restart=unless-stopped \
	-p 8080:80 -p 8443:443 \
	-v /home/rancher/log/auditlog:/var/log/auditlog \
	-e AUDIT_LEVEL=1 \
	-v /home/rancher/data/rancher:/var/lib/rancher \
	registry.docker-cn.com/rancher/rancher

```
 
> **" **把镜像换成了换成了稳定版的V2.1，并关掉了私人ca证书，使用自带的证书（实在不想折腾了）
> 
>  
 

 

 环境说明：

 
     Host       | Description                 
     ---------- | ---------------------------- 
     k8s-master | rancher-server，etcd， Control
     k8s-node1  | Worker                      
     k8s-node2  | Worker                      


> **" **rancher部署一大缺点非常吃内存，server+master最好8G以上，node上的worker 4G以上
> 
>  
 

 
# 添加集群

 rancher2.X 支持导入现有的集群和定制

 ![](https://img-blog.csdnimg.cn/20190213113126937.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L05lZHZlZF9M,size_16,color_FFFFFF,t_70)

 

 
## 选择custom 定制

 成员角色默认

 网络组建有Flannel，Calico，Canal

 默认使用Canal，这里使用Calico

 云提供商，因为是定制无需选择。

 ![](https://img-blog.csdnimg.cn/20190213134604835.png)

 

 ![](https://img-blog.csdnimg.cn/20190213134517504.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L05lZHZlZF9M,size_16,color_FFFFFF,t_70)

 高级选项

 ![](https://img-blog.csdnimg.cn/20190213134923602.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L05lZHZlZF9M,size_16,color_FFFFFF,t_70)

 
## 复制命令，在主机上运行

 选择etcd， Control复制命令到k8s-master上运行

 等待一会便可以看到提示1台新主机注册成功

 ![](https://img-blog.csdnimg.cn/20190212163444393.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L05lZHZlZF9M,size_16,color_FFFFFF,t_70)

 复制命令到主机上运行

 
```
 [root@K8s-master ~]# sudo docker run -d --privileged --restart=unless-stopped --net=host -v /etc/kubernetes:/etc/kubernetes -v /var/run:/var/run rancher/rancher-agent:v2.1.6 --server https://192.168.146.10 --token h4l97k96vb9sf2gn9z7sq22l77rtw74vpqbdzjvl8z79mpfb2hqzs2 --ca-checksum 72e778d0cbac037ff51a96256f30456931d10a9edd30320e062935f1350bae7a --etcd --controlplane
Unable to find image 'rancher/rancher-agent:v2.1.6' locally
v2.1.6: Pulling from rancher/rancher-agent
38e2e6cd5626: Already exists 
705054bc3f5b: Already exists 
c7051e069564: Already exists 
7308e914506c: Already exists 
a2711e11baba: Pull complete 
f9087afb8ba1: Pull complete 
727b818fbd8d: Pull complete 
c2cd62200e43: Pull complete 
d1c7ea3be957: Pull complete 
Digest: sha256:b413508a3c82460b2d4d13928989bb2ef7f03501f4dc88f9e7491e2de4d273bc
Status: Downloaded newer image for rancher/rancher-agent:v2.1.6
e3f0022539633cc283857e5f21d567931b28bbc8e89daabf1c72c4ceec8142cb

```
 

 选择Worker，在node1,2上执行

 
```
 [root@K8s-node1 ~]# sudo docker run -d --privileged --restart=unless-stopped --net=host -v /etc/kubernetes:/etc/kubernetes -v /var/run:/var/run rancher/rancher-agent:v2.1.6 --server https://192.168.146.10 --token g8bhvznct2652nhdgn2nx7qjwtl69mrnxlgh64g79hps7ps2vm646n --ca-checksum 72e778d0cbac037ff51a96256f30456931d10a9edd30320e062935f1350bae7a --worker

```
 运行Worker会拉取大量镜像，等待时间有点长。

 

 node上的worker都注册完后（这里只运行了master，node1所以主机数为2）

 ![](https://img-blog.csdnimg.cn/20190213135556117.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L05lZHZlZF9M,size_16,color_FFFFFF,t_70)

 

 可以看到主界面右上角有两个按钮

 执行命令是rancher集成在控制面板的终端

 kubeconfig则是Linux主机上kubectl的配置文件

 ![](https://img-blog.csdnimg.cn/20190213140145768.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L05lZHZlZF9M,size_16,color_FFFFFF,t_70)

 

 

 点击导航栏的主机可以查看到此时已经注册的主机

 ![](https://img-blog.csdnimg.cn/20190213140016157.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L05lZHZlZF9M,size_16,color_FFFFFF,t_70)

 

 

 管理查看 当前拥有的namespace

 ![](https://img-blog.csdnimg.cn/20190213140357144.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L05lZHZlZF9M,size_16,color_FFFFFF,t_70)

 Rancher部署k8s集群大概就到这里了，接下来就是使用rancher部署服务了。

 

 

 

 

 

 

 

 

 

 

 

 

   
 
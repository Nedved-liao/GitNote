#SELinux禁用
setenforce 0
sed -i '/^SE/s/enforcing/disabled/' /etc/selinux/config 
```
 
## 1.2、创建修改内核文件/etc/sysctl.d/k8s.conf

 
```
 #写入文件
cat <<EOF > /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

#使用sysctl使其生效
sysctl -p /etc/sysctl.d/k8s.conf
```
 
## 1.3、禁用swap

 
```
 
#关闭系统的Swap
swapoff -a

#永久生效
echo "vm.swappiness=0" >> /etc/sysctl.d/k8s.conf

#修改 /etc/fstab 文件，注释掉 SWAP 的自动挂载
#/dev/mapper/centos-swap swap                    swap    defaults        0 0
```
 
# 2、配置yum源

 
```
 #配置阿里的docker-ce 
yum-config-manager --add-repo  https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

#配置kubernetes
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

```
 
# 3、安装docker

 
```
 #建立缓存
yum makecache fast

#安装依赖
yum install -y xfsprogs yum-utils device-mapper-persistent-data lvm2

#安装最新版本的docker
yum install -y --setopt=obsoletes=0 docker-ce-18.06.1.ce-3.el7

#可以使用以下命令查看docker版本
yum list docker-ce.x86_64  --showduplicates |sort -r

#启动docker
systemctl start docker
systemctl enable docker

```
 

 
# 4、在各节点安装kubeadm、kubelet和kubectl

 
```
 #安装kubelet kubeadm kubectl
yum install -y kubelet kubeadm kubectl

```
 
# 5、准备需要用到的镜像

 
```
 #因为kubeadm使用的镜像源是k8s.gcr.io，但是网站已被墙。所以需要提前准备所需要的镜像
#可以使用kubeadm 查看默认需要准备的镜像。并且在此可以看到k8s.gcr.io是无法连接的

[root@K8s-master ~]# kubeadm config images list
I0116 10:11:34.091410   91268 version.go:94] could not fetch a Kubernetes version from the internet: unable to get URL "https://dl.k8s.io/release/stable-1.txt": Get https://dl.k8s.io/release/stable-1.txt: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
I0116 10:11:34.107299   91268 version.go:95] falling back to the local client version: v1.13.2
k8s.gcr.io/kube-apiserver:v1.13.2
k8s.gcr.io/kube-controller-manager:v1.13.2
k8s.gcr.io/kube-scheduler:v1.13.2
k8s.gcr.io/kube-proxy:v1.13.2
k8s.gcr.io/pause:3.1
k8s.gcr.io/etcd:3.2.24
k8s.gcr.io/coredns:1.2.6


```
 根据提示，到docker hub 搜索到对应的镜像后pull下来再tag标签

 
> kube-apiserver:v1.13.0   
>  kube-controller-manager:v1.13.0  
>  kube-scheduler:v1.13.0  
>  kube-proxy:v1.13.0  
>  pause:3.1  
>  etcd:3.2.24  
>  coredns:1.2.6
> 
>  
 下载完后，tag标签为k8s.gcr.io/***

 如：

 docker pull mirrorgooglecontainers/kube-apiserver:v1.13.0

 docker tag mirrorgooglecontainers/kube-apiserver:v1.13.0 k8s.gcr.io/kube-apiserver:v1.13.0   
 

 
```
 [root@K8s-master ~]# docker images
REPOSITORY                             TAG                 IMAGE ID            CREATED             SIZE
k8s.gcr.io/kube-proxy                  v1.13.0             8fa56d18961f        6 weeks ago         80.2MB
k8s.gcr.io/kube-apiserver              v1.13.0             f1ff9b7e3d6e        6 weeks ago         181MB
k8s.gcr.io/kube-controller-manager     v1.13.0             d82530ead066        6 weeks ago         146MB
k8s.gcr.io/kube-scheduler              v1.13.0             9508b7d8008d        6 weeks ago         79.6MB
k8s.gcr.io/coredns                     1.2.6               f59dcacceff4        2 months ago        40MB
k8s.gcr.io/etcd                        3.2.24              3cab8e1b9802        3 months ago        220MB
k8s.gcr.io/pause                       3.1                 da86e6ba6ca1        13 months ago       742kB

```
 镜像准备完毕，开始集群初始化

 

 
# 6、初始化集群

 master 节点

 
## 6.1初始化您的主节点

 主节点是集群里运行控制面的机器，包括 etcd (集群的数据库)和 API 服务（kubectl CLI 与之交互）。

 
  1. 选择一个 Pod 网络插件，并检查是否在 kubeadm 初始化过程中需要传入什么参数。这个取决于 您选择的网络插件，您可能需要设置  `--Pod-network-cidr`  来指定网络驱动的 CIDR。请参阅[安装网络插件](https://kubernetes.io/zh/docs/setup/independent/create-cluster-kubeadm/#Pod-network)。 
  3. (可选) 除非特别指定，kubeadm 会使用默认网关所在的网络接口广播其主节点的 IP 地址。若需使用其他网络接口，请 给  `kubeadm init`  设置  `--apiserver-advertise-address=<ip-address>`  参数。如果需要部署 IPv6 的集群，则需要指定一个 IPv6 地址，比如  `--apiserver-advertise-address=fd00::101` 。 
  5. (可选) 在运行  `kubeadm init`  之前请先执行  `kubeadm config images pull`  来测试与 gcr.io 的连接。 -- 取自kubernetes官网 

 [https://kubernetes.io/zh/docs/setup/independent/create-cluster-kubeadm/](https://kubernetes.io/zh/docs/setup/independent/create-cluster-kubeadm/)

 开始初始化

 
```
 #master 集群初始化
[root@K8s-master ~]# kubeadm init --kubernetes-version=v1.13.0 --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=192.168.146.10
[init] Using Kubernetes version: v1.13.0
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Activating the kubelet service
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [k8s-master localhost] and IPs [192.168.146.10 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [k8s-master localhost] and IPs [192.168.146.10 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [k8s-master kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 192.168.146.10]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[kubelet-check] Initial timeout of 40s passed.
[apiclient] All control plane components are healthy after 184.547282 seconds
[uploadconfig] storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-1.13" in namespace kube-system with the configuration for the kubelets in the cluster
[patchnode] Uploading the CRI Socket information "/var/run/dockershim.sock" to the Node API object "k8s-master" as an annotation
[mark-control-plane] Marking the node k8s-master as control-plane by adding the label "node-role.kubernetes.io/master=''"
[mark-control-plane] Marking the node k8s-master as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]
[bootstrap-token] Using token: fl099d.dxy00288pqxl2xj0
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstraptoken] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstraptoken] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstraptoken] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstraptoken] creating the "cluster-info" ConfigMap in the "kube-public" namespace
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes master has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of machines by running the following on each node
as root:

  kubeadm join 192.168.146.10:6443 --token f0hxzt.q34cvw84otvdnca8 --discovery-token-ca-cert-hash sha256:76c15a976e3bd80c5cea54afeba0587682a131cfc5485cb28e980000102bd945


```
 kubeadm init --kubernetes-version=v1.13.0 --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=192.168.146.10

 选择flannel作为Pod网络插件，所以指定–pod-network-cidr=10.244.0.0/16，API server 用来告知集群中其它成员的地址 

 

 
## 6.2按提示配置常规用户如何使用kubectl访问集群

 
```
   mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
 
## 6.3查看集群状态

 
```
 [root@K8s-master ~/kubernetes]# kubectl get cs
NAME                 STATUS    MESSAGE              ERROR
scheduler            Healthy   ok                   
controller-manager   Healthy   ok                   
etcd-0               Healthy   {"health": "true"}   
```
 
## 7、准备接下来用到的yml文件

 
  1. [kube-flannel.yml](https://github.com/Nedved-liao/kubernetes.1.30_CN/blob/master/kube-flannel.yml) 
  3. [kube-dashboard.yaml](https://github.com/Nedved-liao/kubernetes.1.30_CN/blob/master/kube-dashboard.yaml) 
  5. [dashboard-user-role.yaml](https://github.com/Nedved-liao/kubernetes.1.30_CN/blob/master/dashboard-user-role.yaml) 使用git 拉取准备文件

 
```
 [root@K8s-master ~]# git clone https://github.com/Nedved-liao/kubernetes.1.30_CN
正克隆到 'kubernetes.1.30_CN'...
remote: Enumerating objects: 23, done.
remote: Counting objects: 100% (23/23), done.
remote: Compressing objects: 100% (22/22), done.
remote: Total 23 (delta 6), reused 0 (delta 0), pack-reused 0
Unpacking objects: 100% (23/23), done.

[root@K8s-master ~]# ll kubernetes.1.30_CN/
总用量 32
-rw-r--r--. 1 root root   515 1月  16 10:44 dashboard-user-role.yaml
-rw-r--r--. 1 root root  4675 1月  16 10:44 kube-dashboard.yaml
-rw-r--r--. 1 root root 10739 1月  16 10:44 kube-flannel.yml
-rw-r--r--. 1 root root   276 1月  16 10:44 kubernetes.repo
-rw-r--r--. 1 root root   490 1月  16 10:44 README.md

```
 
## 8、安装Pod Network

 
```
 
#安装flannel
[root@K8s-master ~/kubernetes.1.30_CN]# kubectl create -f kube-flannel.yml 
clusterrole.rbac.authorization.k8s.io/flannel created
clusterrolebinding.rbac.authorization.k8s.io/flannel created
serviceaccount/flannel created
configmap/kube-flannel-cfg created
daemonset.extensions/kube-flannel-ds-amd64 created
daemonset.extensions/kube-flannel-ds-arm64 created
daemonset.extensions/kube-flannel-ds-arm created
daemonset.extensions/kube-flannel-ds-ppc64le created
daemonset.extensions/kube-flannel-ds-s390x created


#粗略查看
[root@K8s-master ~/kubernetes]# kubectl get ds -l app=flannel -n kube-system
NAME                      DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR                     AGE
kube-flannel-ds-amd64     1         1         1       1            1           beta.kubernetes.io/arch=amd64     113m
kube-flannel-ds-arm       0         0         0       0            0           beta.kubernetes.io/arch=arm       113m
kube-flannel-ds-arm64     0         0         0       0            0           beta.kubernetes.io/arch=arm64     113m
kube-flannel-ds-ppc64le   0         0         0       0            0           beta.kubernetes.io/arch=ppc64le   113m
kube-flannel-ds-s390x     0         0         0       0            0           beta.kubernetes.io/arch=s390x     113m

#确保结果无疑
[root@K8s-master ~/kubernetes]# kubectl get pod --all-namespaces -o wide
NAMESPACE     NAME                                    READY   STATUS    RESTARTS   AGE    IP               NODE         NOMINATED NODE   READINESS GATES
kube-system   coredns-86c58d9df4-65nbv                1/1     Running   1          179m   10.244.0.4       k8s-master   <none>           <none>
kube-system   coredns-86c58d9df4-gnczd                1/1     Running   1          179m   10.244.0.5       k8s-master   <none>           <none>
kube-system   etcd-k8s-master                         1/1     Running   1          178m   192.168.146.10   k8s-master   <none>           <none>
kube-system   kube-apiserver-k8s-master               1/1     Running   1          179m   192.168.146.10   k8s-master   <none>           <none>
kube-system   kube-controller-manager-k8s-master      1/1     Running   1          179m   192.168.146.10   k8s-master   <none>           <none>
kube-system   kube-flannel-ds-amd64-9x8mc             1/1     Running   0          113m   192.168.146.10   k8s-master   <none>           <none>
kube-system   kube-proxy-l9pgs                        1/1     Running   1          179m   192.168.146.10   k8s-master   <none>           <none>
kube-system   kube-scheduler-k8s-master               1/1     Running   1          179m   192.168.146.10   k8s-master   <none>           <none>
kube-system   kubernetes-dashboard-6cb88fb59c-fccp4   1/1     Running   0          51m    10.244.0.7       k8s-master   <none>           <none>


```
 
## 9、添加 Slave节点

 
## 9.1、在两个 Slave节点上分别执行jion

 如下命令来让其加入Master上已经就绪了的 k8s集群：

 kubeadm join --token <token> <master-ip>:<master-port> --discovery-token-ca-cert-hash sha256:<hash>

 
## 9.2、node1取于前面kubeadm init 生成的（node2省略）

 
```
 [root@K8s-node1 ~]# kubeadm join 192.168.146.10:6443 --token f0hxzt.q34cvw84otvdnca8 --discovery-token-ca-cert-hash sha256:76c15a976e3bd80c5cea54afeba0587682a131cfc5485cb28e980000102bd945
[preflight] Running pre-flight checks
[discovery] Trying to connect to API Server "192.168.146.10:6443"
[discovery] Created cluster-info discovery client, requesting info from "https://192.168.146.10:6443"
[discovery] Requesting info from "https://192.168.146.10:6443" again to validate TLS against the pinned public key
[discovery] Cluster info signature and contents are valid and TLS certificate validates against pinned roots, will use API Server "192.168.146.10:6443"
[discovery] Successfully established connection with API Server "192.168.146.10:6443"
[join] Reading configuration from the cluster...
[join] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[kubelet] Downloading configuration for the kubelet from the "kubelet-config-1.13" ConfigMap in the kube-system namespace
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Activating the kubelet service
[tlsbootstrap] Waiting for the kubelet to perform the TLS Bootstrap...
[patchnode] Uploading the CRI Socket information "/var/run/dockershim.sock" to the Node API object "k8s-node1" as an annotation

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the master to see this node join the cluster.

```
 如果 token忘记，则可以去 Master上执行如下命令来获取：

 
```
 [root@K8s-master ~/kubernetes.1.30_CN]# kubeadm token list
TOKEN                     TTL       EXPIRES                     USAGES                   DESCRIPTION                                                EXTRA GROUPS
f0hxzt.q34cvw84otvdnca8   23h       2019-01-17T11:16:24+08:00   authentication,signing   The default bootstrap token generated by 'kubeadm init'.   system:bootstrappers:kubeadm:default-node-token


```
 
## 9.3、效果验证

 查看节点状态 

 
```
 [root@K8s-master ~/kubernetes.1.30_CN]# kubectl get no
NAME         STATUS   ROLES    AGE     VERSION
k8s-master   Ready    master   5m28s   v1.13.1
k8s-node1    Ready    <none>   107s    v1.13.2
k8s-node2    Ready    <none>   110s    v1.13.2

```
 查看所有 Pod状态 

 
```
 [root@K8s-master ~/kubernetes.1.30_CN]# kubectl get pods --all-namespaces -o wide
NAMESPACE     NAME                                 READY   STATUS    RESTARTS   AGE     IP               NODE         NOMINATED NODE   READINESS GATES
kube-system   coredns-86c58d9df4-xrz2j             1/1     Running   0          5m26s   10.244.0.19      k8s-master   <none>           <none>
kube-system   coredns-86c58d9df4-z2lcj             1/1     Running   0          5m26s   10.244.0.18      k8s-master   <none>           <none>
kube-system   etcd-k8s-master                      1/1     Running   0          4m53s   192.168.146.10   k8s-master   <none>           <none>
kube-system   kube-apiserver-k8s-master            1/1     Running   0          5m2s    192.168.146.10   k8s-master   <none>           <none>
kube-system   kube-controller-manager-k8s-master   1/1     Running   0          4m51s   192.168.146.10   k8s-master   <none>           <none>
kube-system   kube-flannel-ds-amd64-dwpcr          1/1     Running   0          4m9s    192.168.146.10   k8s-master   <none>           <none>
kube-system   kube-flannel-ds-amd64-h2tm9          1/1     Running   0          2m4s    192.168.146.20   k8s-node1    <none>           <none>
kube-system   kube-flannel-ds-amd64-ssh2d          1/1     Running   0          2m7s    192.168.146.21   k8s-node2    <none>           <none>
kube-system   kube-proxy-4lr8q                     1/1     Running   0          5m26s   192.168.146.10   k8s-master   <none>           <none>
kube-system   kube-proxy-7b22t                     1/1     Running   0          2m7s    192.168.146.21   k8s-node2    <none>           <none>
kube-system   kube-proxy-j6qkx                     1/1     Running   0          2m4s    192.168.146.20   k8s-node1    <none>           <none>
kube-system   kube-scheduler-k8s-master            1/1     Running   0          4m58s   192.168.146.10   k8s-master   <none>           <none>

```
 
## 10、安装 dashboard

 就像给elasticsearch配一个可视化的管理工具一样，我们最好也给 k8s集群配一个可视化的管理工具，便于管理集群。  
 因此我们接下来安装 v1.10.0版本的 kubernetes-dashboard，用于集群可视化的管理。

 
## 10.1、创建

 
```
 [root@K8s-master ~/kubernetes.1.30_CN]# kubectl create -f kube-dashboard.yaml 
secret/kubernetes-dashboard-certs created
serviceaccount/kubernetes-dashboard created
role.rbac.authorization.k8s.io/kubernetes-dashboard-minimal created
rolebinding.rbac.authorization.k8s.io/kubernetes-dashboard-minimal created
deployment.apps/kubernetes-dashboard creat
```
 
## 10.2、查看 dashboard的 pod是否正常启动

 如果正常说明安装成功:

 
```
 [root@K8s-master ~/kubernetes.1.30_CN]# kubectl get pods --namespace=kube-system
NAME                                   READY   STATUS    RESTARTS   AGE
coredns-86c58d9df4-xrz2j               1/1     Running   0          165m
coredns-86c58d9df4-z2lcj               1/1     Running   0          165m
etcd-k8s-master                        1/1     Running   0          164m
kube-apiserver-k8s-master              1/1     Running   0          164m
kube-controller-manager-k8s-master     1/1     Running   0          164m
kube-flannel-ds-amd64-dwpcr            1/1     Running   0          163m
kube-flannel-ds-amd64-h2tm9            1/1     Running   0          161m
kube-flannel-ds-amd64-ssh2d            1/1     Running   0          161m
kube-proxy-4lr8q                       1/1     Running   0          165m
kube-proxy-7b22t                       1/1     Running   0          161m
kube-proxy-j6qkx                       1/1     Running   0          161m
kube-scheduler-k8s-master              1/1     Running   0          164m
kubernetes-dashboard-98b7c88bb-lqggg   1/1     Running   0          150m

```
 
## 10.3、查看 dashboard的外网暴露端口 

 
```
 [root@K8s-master ~/kubernetes.1.30_CN]# kubectl get service --namespace=kube-system
NAME                   TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)         AGE
kube-dns               ClusterIP   10.96.0.10     <none>        53/UDP,53/TCP   167m
kubernetes-dashboard   NodePort    10.106.23.70   <none>        443:31280/TCP   151m

```
 
## 10.4、生成私钥和证书签名 

 
```
 [root@K8s-master ~/kubernetes.1.30_CN]# openssl genrsa -des3 -passout pass:x -out dashboard.pass.key 2048
Generating RSA private key, 2048 bit long modulus
.....+++
.......................................................................................................................................................................................................................................................................................................+++
e is 65537 (0x10001)

[root@K8s-master ~/kubernetes.1.30_CN]# openssl rsa -passin pass:x -in dashboard.pass.key -out dashboard.key
writing RSA key

[root@K8s-master ~/kubernetes.1.30_CN]# rm -f dashboard.pass.key

#一路按回车即可
[root@K8s-master ~/kubernetes.1.30_CN]# openssl req -new -key dashboard.key -out dashboard.csr
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:
State or Province Name (full name) []:
Locality Name (eg, city) [Default City]:
Organization Name (eg, company) [Default Company Ltd]:
Organizational Unit Name (eg, section) []:
Common Name (eg, your name or your server's hostname) []:
Email Address []:

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:

```
 
## 10.5、 生成SSL证书,并移动到 /etc/kubernetes/pki

 将生成的 dashboard.key 和 dashboard.crt置于路径 /etc/kubernetes/pki下， 路径在dashboard.yaml 里面已经定义好

 
```
 #生成证书
[root@K8s-master ~/kubernetes.1.30_CN]# openssl x509 -req -sha256 -days 365 -in dashboard.csr -signkey dashboard.key -out dashboard.crt
Signature ok
subject=/C=XX/L=Default City/O=Default Company Ltd
Getting Private key

#移动到/etc/kubernetes/pki
[root@K8s-master ~/kubernetes.1.30_CN]# cp dashboard.c* /etc/kubernetes/pki/

#确认移动
[root@K8s-master ~/kubernetes.1.30_CN]# ll /etc/kubernetes/pki/dashboard.*
-rw-r--r--. 1 root root 1103 1月  16 14:08 /etc/kubernetes/pki/dashboard.crt
-rw-r--r--. 1 root root  952 1月  16 14:08 /etc/kubernetes/pki/dashboard.csr


```
 
## 10.6、创建 dashboard用户 

 
```
 [root@K8s-master ~/kubernetes.1.30_CN]# kubectl create -f dashboard-user-role.yaml
clusterrolebinding.rbac.authorization.k8s.io/admin created
serviceaccount/admin created

```
 
## 10.7、获取登陆token 

 
```
 [root@K8s-master ~/kubernetes.1.30_CN]# kubectl describe secret/$(kubectl get secret -nkube-system |grep admin|awk '{print $1}') -nkube-system
Name:         admin-token-q6v79
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: admin
              kubernetes.io/service-account.uid: 9ef5dbe7-1955-11e9-a360-000c29b4a7f7

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1025 bytes
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi10b2tlbi1xNnY3OSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJhZG1pbiIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjllZjVkYmU3LTE5NTUtMTFlOS1hMzYwLTAwMGMyOWI0YTdmNyIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlLXN5c3RlbTphZG1pbiJ9.BS93rVuy_61e5NKU20ZbXMybtr239tLgv9cmfL1knu9YZf66GBwQvERUtnHcYACt8vaD55RNLVk_9uHAKESSo0iJMv1-doKbAPZDrL-PT7XomrSgHleVzSyPHMixFRZVcQXpi5l1DcBC2QdNdfZL7h5SAnrs2NFuoGRv5IQXMMnlRVvbWFhBXIbVqRU7lEJo7VXglOYFjNPOC8JkTxxk2GsWJmp1zT-8ZRpajhfhe9VFxi-JLcKgMgv4d5IYGXr1CGcwMIChJz7jnPg7itSTpyYLGGTinZx0HhBivMw9hRm6RqAQgsr4g9sgGCeBRrFcZMbjKsvaQ3dnb7Dnupyyag

```
 访问https://IP:port   
 port 在dashboard.yaml 中的nodeport定义

 
## 10.8、访问UI页面

 查看暴露端口

 
```
 [root@K8s-master ~/kubernetes.1.30_CN]# kubectl get svc --all-namespaces
NAMESPACE     NAME                   TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)         AGE
default       kubernetes             ClusterIP   10.96.0.1      <none>        443/TCP         5h27m
kube-system   kube-dns               ClusterIP   10.96.0.10     <none>        53/UDP,53/TCP   5h27m
kube-system   kubernetes-dashboard   NodePort    10.106.23.70   <none>        443:31280/TCP   5h12m

```
 如上所示，Dashboard已经在 `31280` 端口上公开，现在可以在外部使用 `https://<cluster-ip>:31280` 进行访问。

 需要注意的是，在多节点的集群中，必须找到运行Dashboard节点的IP来访问，而不是Master节点的IP

 MatserIP为 `192.168.146.10` ，ClusterIP为 `192.168.146.20,192.168.146.21` 

 直接访问master 的可能会出现如下 ：![](https://img-blog.csdnimg.cn/20190116164004952.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L05lZHZlZF9M,size_16,color_FFFFFF,t_70)

 

 我们使用kubectl describe 来看下实际上是哪个node在运行dashboard

 
```
 [root@K8s-master ~/kubernetes.1.30_CN]# kubectl describe po kubernetes-dashboard-98b7c88bb-lqggg  -n kube-system 
Name:               kubernetes-dashboard-98b7c88bb-lqggg
Namespace:          kube-system
Priority:           0
PriorityClassName:  <none>
Node:               k8s-node2/192.168.146.21
Start Time:         Wed, 16 Jan 2019 11:31:42 +0800
Labels:             k8s-app=kubernetes-dashboard
                    pod-template-hash=98b7c88bb
Annotations:        <none>
Status:             Running
IP:                 10.244.1.2
Controlled By:      ReplicaSet/kubernetes-dashboard-98b7c88bb
Containers:
  kubernetes-dashboard:
    Container ID:  docker://4fe074b5a90e86bcdb1d055d26204523c0cd72ecd5502af5a805d992ff77436f
    Image:         mirrorgooglecontainers/kubernetes-dashboard-amd64:v1.10.0
    Image ID:      docker-pullable://mirrorgooglecontainers/kubernetes-dashboard-amd64@sha256:e4b764fa9df0a30c467e7cec000920ea69dcc2ba8a9d0469ffbf1881a9614270
    Port:          8443/TCP
    Host Port:     0/TCP
    Args:
      --auto-generate-certificates
      --token-ttl=5400
    State:          Running
      Started:      Wed, 16 Jan 2019 11:35:47 +0800
    Ready:          True
    Restart Count:  0
    Liveness:       http-get https://:8443/ delay=30s timeout=30s period=10s #success=1 #failure=3
    Environment:    <none>
    Mounts:
      /certs from kubernetes-dashboard-certs (rw)
      /tmp from tmp-volume (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kubernetes-dashboard-token-zm8rg (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kubernetes-dashboard-certs:
    Type:          HostPath (bare host directory volume)
    Path:          /etc/kubernetes/pki
    HostPathType:  Directory
  tmp-volume:
    Type:    EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:  
  kubernetes-dashboard-token-zm8rg:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  kubernetes-dashboard-token-zm8rg
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node-role.kubernetes.io/master:NoSchedule
                 node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:          <none>

```
 可以看到实际上是node2在运行

 我们再用node2的ip访问，此时就看到久违的页面了

 ![](https://img-blog.csdnimg.cn/20190116164553559.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L05lZHZlZF9M,size_16,color_FFFFFF,t_70)

 选择令牌。把10.7步骤中找到的token粘贴进去，登录dashboard

 由于在正式环境中，并不推荐使用NodePort的方式来访问Dashboard

 

 
# 11、集成Heapster

 Heapster是容器集群监控和性能分析工具，天然的支持Kubernetes和CoreOS。

 
## 11.1、准备相应的yaml文件

 Heapster支持多种储存方式，

 
  1. heapster 
  3. influxdb 
  5. grafana Git clone

 
```
 git clone https://github.com/Nedved-liao/kubernetes.1.30_CN/tree/master/heapster
```
 官方 GitHub 

 [https://github.com/kubernetes-retired/heapster](https://github.com/kubernetes-retired/heapster) 

 个人修改后[GitHub](https://github.com/Nedved-liao/kubernetes.1.30_CN/tree/master/heapster)

 使用默认的yaml文件， 创建的pod日志一直报错如下：

 
```
 E1028 07:39:05.011439       1 manager.go:101] Error in scraping containers from Kubelet:XX.XX.XX.XX:10255: failed to get all container stats from Kubelet URL "http://XX.XX.XX.XX:10255/stats/container/": Post http://XX.XX.XX.XX:10255/stats/container/: dial tcp XX.XX.XX.XX:10255:
 getsockopt: connection refused
```
 经过googling后

 [https://brookbach.com/2018/10/29/Heapster-on-Kubernetes-1.11.3.html](https://brookbach.com/2018/10/29/Heapster-on-Kubernetes-1.11.3.html)

 参照里面的解析修改了部分官方给出的内容

 **heapster.yaml**

 
> - --source=kubernetes:https://kubernetes.default 
> 
>  - --source=kubernetes.summary_api:''?useServiceAccount=true&kubeletHttps=true&kubeletPort=10250&insecure=true
> 
>  name: system:heapster
> 
>  name: heapster
> 
>  #并把 `k8s.gcr.io` 修改为国内镜像。
> 
>  
 `并新增了 heapster-role.yaml` 

 
```
 apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: heapster
rules:
- apiGroups:
  - ""
  resources:
  - pods
  - nodes
  - namespaces
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - extensions
  resources:
  - deployments
  verbs:
  - get
  - list
  - update
  - watch
- apiGroups:
  - ""
  resources:
  - nodes/stats
  verbs:
  - get

```
 
# 11.2、创建Pod

 
```
 [root@K8s-master ~/kubernetes.1.30_CN/heapster]# ll
总用量 20
-rw-r--r-- 1 root root 2353 5月   1 2018 grafana.yaml
-rw-r--r-- 1 root root  269 1月  23 14:46 heapster-rbac.yaml
-rw-r--r-- 1 root root  368 1月  23 14:47 heapster-role.yaml
-rw-r--r-- 1 root root 1214 1月  23 14:45 heapster.yaml
-rw-r--r-- 1 root root 1005 5月   1 2018 influxdb.yaml
[root@K8s-master ~/kubernetes.1.30_CN/heapster]# kubectl create -f .

```
 
## 11.3、验证结果

 稍等后，查看一下Pod的状态处于running：

 
```
 [root@K8s-master ~/kubernetes.1.30_CN/heapster]# kubectl -n kube-system get po 
NAME                                   READY   STATUS    RESTARTS   AGE
coredns-86c58d9df4-xrz2j               1/1     Running   8          7d4h
coredns-86c58d9df4-z2lcj               1/1     Running   8          7d4h
etcd-k8s-master                        1/1     Running   3          7d4h
heapster-7c46fbf7cf-sffcg              1/1     Running   0          69m
kube-apiserver-k8s-master              1/1     Running   11         7d4h
kube-controller-manager-k8s-master     1/1     Running   14         7d4h
kube-flannel-ds-amd64-dwpcr            1/1     Running   12         7d4h
kube-flannel-ds-amd64-h2tm9            1/1     Running   10         7d4h
kube-flannel-ds-amd64-ssh2d            1/1     Running   10         7d4h
kube-proxy-4lr8q                       1/1     Running   3          7d4h
kube-proxy-7b22t                       1/1     Running   2          7d4h
kube-proxy-j6qkx                       1/1     Running   2          7d4h
kube-scheduler-k8s-master              1/1     Running   15         7d4h
kubernetes-dashboard-98b7c88bb-2fzlw   1/1     Running   0          20h
monitoring-grafana-5f5c6c4c4b-zmhkf    1/1     Running   0          69m
monitoring-influxdb-68df588786-zz9h5   1/1     Running   0          69m

```
 再查看日志，确保没有问题

 
```
 #查看heapster的日志，若有报错进行相对应的修复。influxdb，grafana日志此处省略
[root@K8s-master ~/kubernetes.1.30_CN/heapster]# kubectl -n kube-system logs --tail=200 heapster-7c46fbf7cf-sffcg
I0123 06:50:51.635860       1 heapster.go:72] /heapster --source=kubernetes.summary_api:''?useServiceAccount=true&kubeletHttps=true&kubeletPort=10250&insecure=true --sink=influxdb:http://monitoring-influxdb.kube-system.svc:8086
I0123 06:50:51.636028       1 heapster.go:73] Heapster version v1.4.2
I0123 06:50:51.637662       1 configs.go:61] Using Kubernetes client with master "https://10.96.0.1:443" and version v1
I0123 06:50:51.637859       1 configs.go:62] Using kubelet port 10250
I0123 06:50:52.102425       1 influxdb.go:278] created influxdb sink with options: host:monitoring-influxdb.kube-system.svc:8086 user:root db:k8s
I0123 06:50:52.103093       1 heapster.go:196] Starting with InfluxDB Sink
I0123 06:50:52.103158       1 heapster.go:196] Starting with Metric Sink
I0123 06:50:53.072648       1 heapster.go:106] Starting heapster on port 8082
I0123 06:51:12.521512       1 influxdb.go:241] Created database "k8s" on influxDB server at "monitoring-influxdb.kube-system.svc:8086"
```
 确认无误后，刷新浏览器，查看到cpu使用率证明部署成功

 ![](https://img-blog.csdnimg.cn/20190123160603482.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L05lZHZlZF9M,size_16,color_FFFFFF,t_70)

 

 

 
# 12、集群拆卸 

 
## 12.1、拆卸集群

 首先处理各节点：

 
> kubectl drain <node name> --delete-local-data --force --ignore-daemonsets  
>  kubectl delete node <node name>
> 
>  
 一旦节点移除之后，则可以执行如下命令来重置集群：  
 kubeadm reset

 
## 12.2、master处理节点

 
```
 [root@K8s-master ~]# kubectl drain k8s-node1 --delete-local-data --force --ignore-daemonsets
node/k8s-node1 cordoned
WARNING: Ignoring DaemonSet-managed pods: kube-flannel-ds-amd64-r2qnb, kube-proxy-cvbbf
node/k8s-node1 drained
[root@K8s-master ~]# kubectl delete node k8s-node1
node "k8s-node1" deleted

```
 
## 12.3节点重置（node2省略）

 
```
 [root@K8s-node1 ~]# kubeadm reset
[reset] WARNING: changes made to this host by 'kubeadm init' or 'kubeadm join' will be reverted.
[reset] are you sure you want to proceed? [y/N]: y
[preflight] running pre-flight checks
[reset] no etcd config found. Assuming external etcd
[reset] please manually reset etcd to prevent further issues
[reset] stopping the kubelet service
[reset] unmounting mounted directories in "/var/lib/kubelet"
[reset] deleting contents of stateful directories: [/var/lib/kubelet /etc/cni/net.d /var/lib/dockershim /var/run/kubernetes]
[reset] deleting contents of config directories: [/etc/kubernetes/manifests /etc/kubernetes/pki]
[reset] deleting files: [/etc/kubernetes/admin.conf /etc/kubernetes/kubelet.conf /etc/kubernetes/bootstrap-kubelet.conf /etc/kubernetes/controller-manager.conf /etc/kubernetes/scheduler.conf]

The reset process does not reset or clean up iptables rules or IPVS tables.
If you wish to reset iptables, you must do so manually.
For example: 
iptables -F && iptables -t nat -F && iptables -t mangle -F && iptables -X

If your cluster was setup to utilize IPVS, run ipvsadm --clear (or similar)
to reset your system's IPVS tables.

[root@K8s-node1 ~]# iptables -F && iptables -t nat -F && iptables -t mangle -F && iptables -X

```
 
## 12.4、master重置

 

 
```
 [root@K8s-master ~]# kubeadm reset
[reset] WARNING: changes made to this host by 'kubeadm init' or 'kubeadm join' will be reverted.
[reset] are you sure you want to proceed? [y/N]: y
[preflight] running pre-flight checks
[reset] no etcd config found. Assuming external etcd
[reset] please manually reset etcd to prevent further issues
[reset] stopping the kubelet service
[reset] unmounting mounted directories in "/var/lib/kubelet"
[reset] deleting contents of stateful directories: [/var/lib/kubelet /etc/cni/net.d /var/lib/dockershim /var/run/kubernetes]
[reset] deleting contents of config directories: [/etc/kubernetes/manifests /etc/kubernetes/pki]
[reset] deleting files: [/etc/kubernetes/admin.conf /etc/kubernetes/kubelet.conf /etc/kubernetes/bootstrap-kubelet.conf /etc/kubernetes/controller-manager.conf /etc/kubernetes/scheduler.conf]

The reset process does not reset or clean up iptables rules or IPVS tables.
If you wish to reset iptables, you must do so manually.
For example: 
iptables -F && iptables -t nat -F && iptables -t mangle -F && iptables -X
[root@K8s-master ~]# iptables -F && iptables -t nat -F && iptables -t mangle -F && iptables -X
```
 

 

 

 

 

 

 

 

 

 

 

 

 

   
 
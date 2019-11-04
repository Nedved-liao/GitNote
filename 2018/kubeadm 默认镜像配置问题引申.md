   背景：

 每次使用功能kubeadm的时候都需要提前准备好镜像，为什么自定义使用的镜像源呢？  
 

 在没有翻越围墙时

 
```
 kubeadm init --kubernetes-version=v1.13.0 --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=192.168.146.10

	[ERROR ImagePull]: failed to pull image k8s.gcr.io/kube-apiserver:v1.13.0: output: Trying to pull repository k8s.gcr.io/kube-apiserver ... 
Get https://k8s.gcr.io/v1/_ping: dial tcp 108.177.97.82:443: getsockopt: connection refused
, error: exit status 1
	[ERROR ImagePull]: failed to pull image k8s.gcr.io/kube-controller-manager:v1.13.0: output: Trying to pull repository k8s.gcr.io/kube-controller-manager ... 
Get https://k8s.gcr.io/v1/_ping: dial tcp 108.177.97.82:443: getsockopt: connection refused
, error: exit status 1
	[ERROR ImagePull]: failed to pull image k8s.gcr.io/kube-scheduler:v1.13.0: output: Trying to pull repository k8s.gcr.io/kube-scheduler ... 
Get https://k8s.gcr.io/v1/_ping: dial tcp 74.125.204.82:443: getsockopt: connection refused
, error: exit status 1
	[ERROR ImagePull]: failed to pull image k8s.gcr.io/kube-proxy:v1.13.0: output: Trying to pull repository k8s.gcr.io/kube-proxy ... 
Get https://k8s.gcr.io/v1/_ping: dial tcp 108.177.97.82:443: getsockopt: connection refused
, error: exit status 1
	[ERROR ImagePull]: failed to pull image k8s.gcr.io/pause:3.1: output: Trying to pull repository k8s.gcr.io/pause ... 
Get https://k8s.gcr.io/v1/_ping: dial tcp 108.177.97.82:443: getsockopt: connection refused
, error: exit status 1
	[ERROR ImagePull]: failed to pull image k8s.gcr.io/etcd:3.2.24: output: Trying to pull repository k8s.gcr.io/etcd ... 
Get https://k8s.gcr.io/v1/_ping: dial tcp 74.125.204.82:443: getsockopt: connection refused
, error: exit status 1
	[ERROR ImagePull]: failed to pull image k8s.gcr.io/coredns:1.2.6: output: Trying to pull repository k8s.gcr.io/coredns ... 
Get https://k8s.gcr.io/v1/_ping: dial tcp 108.177.125.82:443: getsockopt: connection refused
, error: exit status 1

```
 去官网翻翻资料，看下是否可以直接配置

 [https://kubernetes.io/zh/docs/setup/independent/create-cluster-kubeadm/](https://kubernetes.io/zh/docs/setup/independent/create-cluster-kubeadm/)

 

 使用kubeadm images list 可以查看到kubeadm默认使用的镜像列表

 同样我们是不是可以改变kubeadm使用的默认镜像呢？

 首先我们先看一下kubeadm命令

 
```
 #让我们看下config选项
[root@K8s-master ~]# kubeadm config -h

There is a ConfigMap in the kube-system namespace called "kubeadm-config" that kubeadm uses to store internal configuration about the
cluster. kubeadm CLI v1.8.0+ automatically creates this ConfigMap with the config used with 'kubeadm init', but if you
initialized your cluster using kubeadm v1.7.x or lower, you must use the 'config upload' command to create this
ConfigMap. This is required so that 'kubeadm upgrade' can configure your upgraded cluster correctly.

Usage:
  kubeadm config [flags]
  kubeadm config [command]

Available Commands:
  images        Interact with container images used by kubeadm.
  migrate       Read an older version of the kubeadm configuration API types from a file, and output the similar config object for the newer version.
  print         Print configuration
  upload        Upload configuration about the current state, so that 'kubeadm upgrade' can later know how to configure the upgraded cluster.
  view          View the kubeadm configuration stored inside the cluster.

Flags:
  -h, --help                help for config
      --kubeconfig string   The kubeconfig file to use when talking to the cluster. If the flag is not set, a set of standard locations are searched for an existing KubeConfig file. (default "/etc/kubernetes/admin.conf")

Global Flags:
      --log-file string   If non-empty, use this log file
      --rootfs string     [EXPERIMENTAL] The path to the 'real' host root filesystem.
      --skip-headers      If true, avoid header prefixes in the log messages
  -v, --v Level           log level for V logs

Use "kubeadm config [command] --help" for more information about a command.


```
 
> images 查看kubeadm使用的镜像
> 
>  print 打印配置
> 
>  
 因为我需要更改的是kubeadm使用的镜像所以继续使用kuberadm config images -h 查看帮助

 
```
 [root@K8s-master ~]# kubeadm config images -h
Interact with container images used by kubeadm.

Usage:
  kubeadm config images [flags]
  kubeadm config images [command]

Available Commands:
  list        Print a list of images kubeadm will use. The configuration file is used in case any images or image repositories are customized.
  pull        Pull images used by kubeadm.

Flags:
  -h, --help   help for images

Global Flags:
      --kubeconfig string   The kubeconfig file to use when talking to the cluster. If the flag is not set, a set of standard locations are searched for an existing KubeConfig file. (default "/etc/kubernetes/admin.conf")
      --log-file string     If non-empty, use this log file
      --rootfs string       [EXPERIMENTAL] The path to the 'real' host root filesystem.
      --skip-headers        If true, avoid header prefixes in the log messages
  -v, --v Level             log level for V logs

Use "kubeadm config images [command] --help" for more information about a command.

```
 发现这里只有一个list 和pull了可以判断查看配置可以直接用 kubeadm config print

 
```
 #先尝试打印出配置文件
[root@K8s-master ~]# kubeadm config print 
Error: missing subcommand; "print" is not meant to be run on its own
Usage:
  kubeadm config print [flags]
  kubeadm config print [command]

Available Commands:
  init-defaults Print default init configuration, that can be used for 'kubeadm init'
  join-defaults Print default join configuration, that can be used for 'kubeadm join'

Flags:
  -h, --help   help for print

Global Flags:
      --kubeconfig string   The kubeconfig file to use when talking to the cluster. If the flag is not set, a set of standard locations are searched for an existing KubeConfig file. (default "/etc/kubernetes/admin.conf")
      --log-file string     If non-empty, use this log file
      --rootfs string       [EXPERIMENTAL] The path to the 'real' host root filesystem.
      --skip-headers        If true, avoid header prefixes in the log messages
  -v, --v Level             log level for V logs

Use "kubeadm config print [command] --help" for more information about a command.

error: missing subcommand; "print" is not meant to be run on its own

#发现报错，根据提示我们是需要打印init-defaults init的默认配置文件
[root@K8s-master ~]# kubeadm config print init-defaults
apiVersion: kubeadm.k8s.io/v1beta1
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: abcdef.0123456789abcdef
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 1.2.3.4
  bindPort: 6443
nodeRegistration:
  criSocket: /var/run/dockershim.sock
  name: k8s-master
  taints:
  - effect: NoSchedule
    key: node-role.kubernetes.io/master
---
apiServer:
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta1
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controlPlaneEndpoint: ""
controllerManager: {}
dns:
  type: CoreDNS
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: k8s.gcr.io
kind: ClusterConfiguration
kubernetesVersion: v1.13.0
networking:
  dnsDomain: cluster.local
  podSubnet: ""
  serviceSubnet: 10.96.0.0/12
scheduler: {}

```
 yaml格式的配置文件，

 可以看到这里的默认image仓库使用的是k8s.gcr.io

 
> imageRepository: k8s.gcr.io
> 
>  
 此时我们把打印的配置文件copy到一个文件中new.yaml

 把其中的imageRepository:改成我们的仓库 如 mirrorgooglecontainers

 在使用kubeadm config images list --config ./new.yaml

 
```
 [root@K8s-master ~]# kubeadm config images list --config new.yaml 
W0116 10:29:05.964789  100887 common.go:86] WARNING: Detected resource kinds that may not apply: [InitConfiguration JoinConfiguration]
[config] WARNING: Ignored YAML document with GroupVersionKind kubeadm.k8s.io/v1beta1, Kind=JoinConfiguration
mirrorgooglecontainers/kube-apiserver:v1.13.0
mirrorgooglecontainers/kube-controller-manager:v1.13.0
mirrorgooglecontainers/kube-scheduler:v1.13.0
mirrorgooglecontainers/kube-proxy:v1.13.0
mirrorgooglecontainers/pause:3.1
mirrorgooglecontainers/etcd:3.2.24
mirrorgooglecontainers/coredns:1.2.6

```
 可以看到此时使用的镜像已经改过来了

 

 但是问题又来了，mirrorgooglecontainers并没有所以的组件镜像。。

 可以替换一个有所以组件的镜像源网站，

 

 亦或者在我们先前看到的yaml文件中定义，但是碍于刚上手k8s这个实在没有办法自定义这个yaml文件。

 但不妨这也是一个解决路径。

 

 先留着

   
 
---
title: Cent OS 7 添加  EPEL  Nux Dextop  ELRepo等源
date: 2017-11-11 19:03:38
tags: CSDN迁移
---
 [ ](http://creativecommons.org/licenses/by-sa/4.0/) 版权声明：本文为博主原创文章，遵循[ CC 4.0 BY-SA ](http://creativecommons.org/licenses/by-sa/4.0/)版权协议，转载请附上原文出处链接和本声明。  本文链接：[https://blog.csdn.net/Nedved_L/article/details/78508044](https://blog.csdn.net/Nedved_L/article/details/78508044)   
    
  ## Cent OS 7 添加第三方yum源

 CentOS由于很追求稳定性，所以官方源中自带的软件不多，因而需要一些第三方源。   
 比如EPEL、ATrpms、ELRepo、Nux Dextop、RepoForge等。   
 网上有很多添加第三方yum源的方法，内德整理了部分方法

 
## 添加阿里yum

 wget -O /etc/yum.repos.d/CentOS-Base.repo [http://mirrors.aliyun.com/repo/Centos-7.repo](http://mirrors.aliyun.com/repo/Centos-7.repo)

 
## EPEL

 EPEL即Extra Packages for Enterprise Linux，为CentOS提供了额外的10000多个软件包，而且在不替换系统组件方面下了很多功夫，因而可以放心使用。   
 rpm -ivh [http://dl.fedoraproject.org/pub/epel/7/x86_64/e/epel-release-7-5.noarch.rpm](http://dl.fedoraproject.org/pub/epel/7/x86_64/e/epel-release-7-5.noarch.rpm)

 倘若网站连接失效，可以到以下网站搜索epel-release-7-5.noarch.rpm   
 [http://www.baidu.com/link?url=MdhH8ypAtivcLZJAQD4JbBcQKrx9X7yNDU–6evVPcO&wd=&eqid=ca19ff090001cdc7000000025a092c44](http://www.baidu.com/link?url=MdhH8ypAtivcLZJAQD4JbBcQKrx9X7yNDU--6evVPcO&amp;wd=&amp;eqid=ca19ff090001cdc7000000025a092c44)

 
## Nux Dextop

 Nux Dextop中包含了一些与多媒体相关的软件包，作者尽量保证不覆盖base源。官方说明中说该源与EPEL兼容，实际上个别软件包存在冲突，但基本不会造成影响:   
 rpm -Uvh [http://li.nux.ro/download/nux/dextop/el7/x86_64/nux-dextop-release-0-5.el7.nux.noarch.rpm](http://li.nux.ro/download/nux/dextop/el7/x86_64/nux-dextop-release-0-5.el7.nux.noarch.rpm)

 
## ELRepo

 ELRepo包含了一些硬件相关的驱动程序，比如显卡、声卡驱动:

 
```
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-2.el7.elrepo.noarch.rpm
```
 
## Remi

 Remi源大家或许很少听说，不过Remi源GoFace强烈推荐，尤其对于不想编译最新版的linux使用者，因为Remi源中的软件几乎都是最新稳定版。或许您会怀疑稳定不？放心吧，这些都是Linux骨灰级的玩家编译好放进源里的，他们对于系统环境和软件编译参数的熟悉程度毋庸置疑。

 rpm -Uvh [http://rpms.famillecollet.com/enterprise/remi-release-7.rpm](http://rpms.famillecollet.com/enterprise/remi-release-7.rpm)

 RPMForge   
 RPMForge是CentOS系统下的软件仓库，拥有4000多种的软件包，被CentOS社区认为是最安全也是最稳定的一个软件仓库。

 rpm -Uvh [http://pkgs.repoforge.org/rpmforge-release/rpmforge-release-0.5.3-1.el7.rf.x86_64.rpm](http://pkgs.repoforge.org/rpmforge-release/rpmforge-release-0.5.3-1.el7.rf.x86_64.rpm)

 
## RPMFusion

 如果您现在正在使用Fedora 15，对RPMFusion一定不陌生吧，各种音频软件如MPlayer在标准源中是没有的，一般先安装RPMFusion源，之后就可以放便地yum install各种需要的软件啦。   
 添加阿里云的RPMFusion源

 
```
rpm -Uvh http://mirrors.aliyun.com/rpmfusion/free/el/updates/6/x86_64/rpmfusion-free-release-6-1.noarch.rpm
```
 
```
sudo rpm -Uvh http://mirrors.aliyun.com/rpmfusion/nonfree/el/updates/6/x86_64/rpmfusion-nonfree-release-6-1.noarch.rpm
```
   
  
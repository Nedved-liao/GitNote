---
title: CentOS 7.4 安装网易云音乐
date: 2017-11-10 15:47:02
tags: CSDN迁移
---
 [ ](http://creativecommons.org/licenses/by-sa/4.0/) 版权声明：本文为博主原创文章，遵循[ CC 4.0 BY-SA ](http://creativecommons.org/licenses/by-sa/4.0/)版权协议，转载请附上原文出处链接和本声明。  本文链接：[https://blog.csdn.net/Nedved_L/article/details/78500524](https://blog.csdn.net/Nedved_L/article/details/78500524)   
    
  1.下包–>网易云音乐   
 Ubuntu14.04（推荐14.04依赖包网上能找到）   
 ![这里写图片描述](https://img-blog.csdn.net/20171111101837468?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvTmVkdmVkX0w=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)   
 提示:16.04有部分依赖包还找不到，有兴趣可以自行打包RPM安装。   
 2.解包   
 （1）使用 ar -vx解压ubuntu的压缩包 netease-cloud-music_1.0.0-2_amd64_ubuntu14.04.deb

 
```
[root@room8pc205 杂项]# ls 
netease-cloud-music_1.0.0-2_amd64_ubuntu14.04.deb
[root@room8pc205 杂项]#ar -vx netease-cloud-music_1.0.0-2_amd64_ubuntu14.04.deb
```
 得到如下三个文件   
 * - debian-binary   
 * - control.tar.gz   
 * - data.tar.xz   
 （2） 继续解压data.tar.xz（所需的软件程序）

 
```
[root@room8pc205 杂项]#tar -xvf data.tar.xz 

```
 得到一个usr的目录   
 cp或移动usr内的文件到/usr/lib64(默认X64放置程序目录)

 
```
[root@room8pc205 杂项]#cp -r usr/*   /usr/
```
 3.解决安装依赖问题（两种方法）   
 第一种下载高版本的gcc进行编译，   
 第二种手动转码编译。   
 以下介绍第二种方法   
 （1）运行/bin/netease-cloud-music（网易云音乐）   
 看提示添加7455权限给 /usr/lib/netease-cloud-music/chrome-sandbox   
 运行后看报错，找到要安装的依赖包

 
```
[root@room8pc205 杂项]#chmod 7445  /usr/lib/netease-cloud-music/chrome-sandbox
```
 可以到Rpm serach（[http://rpm.pbone.net/](http://rpm.pbone.net/)）下载   
 找到对应需要的依赖包下载并安装

 对于centos7.4一般只需要三个依赖包，解决依赖包后再运行就可以启动网易云音乐了.   
 提示：root身份无法打开网易云音乐（吐核，大牛们可以自行去解决），以usr运行

 4.在播放歌曲时发现网络设置错误，   
 由于没有安装解码器导致，所以呢需要安装streamers解码器   
 ![这里写图片描述](https://img-blog.csdn.net/20171111120252273?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvTmVkdmVkX0w=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)   
 环境准备首先安装epel、elrepo、nux-dextop第三方源并确你的yum可用（内德用的是阿里yum，163yum较稳定）。   
 简单介绍：   
 EPEL即Extra Packages for Enterprise Linux，为CentOS提供了额外的10000多个软件包，而且在不替换系统组件方面下了很多功夫，因而可以放心使用。   
 ELRepo包含了一些硬件相关的驱动程序，比如显卡、声卡驱动。   
 Nux Dextop中包含了一些与多媒体相关的软件包，作者尽量保证不覆盖base源。官方说明中说该源与EPEL兼容，实际上个别软件包存在冲突，但基本不会造成影响。

 （2）安装解码器：   
 yum -y install gstreamer-ffmpeg   
 yum -y install gstreamer-plugins-ugly   
 yum -y install gstreamer-plugins-bad   
 yum -y install ffmpeg   
 yum -y install libvdpau   
 yum -y install mpg123   
 yum -y install mplayer   
 yum -y install mplayer-gui   
 yum -y install gstreamer1-libav   
 yum -y install vlc   
 （包太多就不多注释了，有兴趣的可以去问度娘）   
 重启   
 注意i 运行需以usr身份，

   
  
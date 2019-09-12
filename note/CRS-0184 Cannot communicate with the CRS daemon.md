---
title: CRS-0184 Cannot communicate with the CRS daemon
date: 2018-08-10 10:50:53
tags: CSDN迁移
---
 [ ](http://creativecommons.org/licenses/by-sa/4.0/) 版权声明：本文为博主原创文章，遵循[ CC 4.0 BY-SA ](http://creativecommons.org/licenses/by-sa/4.0/)版权协议，转载请附上原文出处链接和本声明。  本文链接：[https://blog.csdn.net/Nedved_L/article/details/81559420](https://blog.csdn.net/Nedved_L/article/details/81559420)   
    
   # 事件背景

 rman清理脚本异常.导致磁盘空间爆满(一个环境变量没有设置正确)  
 释放磁盘空间,进行rman清理  
 之后,领导把实例重启,但是ams实例没有关闭

 

 
## 环境

 系统 : AIX

 数据库: Oracle10g

 

 

 
# 问题处理

 下午,客户反应页面无法登陆.  
 事发第一时间,检查集群

 
### 1.进行集群检查

 
> #crs_stat -t  
>  CRS-0184: Cannot communicate with the CRS daemon.
> 
>  
 
### 2.尝试启动(root)

 
> #crsctl start crs  
>  CRS-4640: Oracle High Availability Services is already active  
>  CRS-4000: Command Start failed, or completed with errors.
> 
>  
 根据提示Services is already active

 
### 3.检查crs服务

 
> #crsctl check crs  
>  CRS-4638: Oracle High Availability Services is online  
>  CRS-4535: Cannot communicate with Cluster Ready Services  
>  CRS-4530: Communications failure contacting Cluster Synchronization Services demon  
>  CRS-4534: Cannot communicate with Event Manager‘
> 
>  
 
### 4.看似没问题,进行快速检查vip是否正常

 查看本地配置文件

 
> #cat /etc/hosts  
>  10.205.128.21 ORA1  
>  10.205.128.22 ORA2  
>  100.100.100.21 ORA1-priv  
>  100.100.100.22 ORA2-priv  
>  10.205.128.25 ORA1-vip  
>  10.205.128.26 ORA2-vip  
>  10.205.128.27 ORA-scan
> 
>  
> 
>  #ifconfig -a
> 
>  en0: flags=1e084863,480<UP,BROADCAST,NOTRAILERS,RUNNING,SIMPLEX,MULTICAST,GROUPRT,64BIT,CHECKSUM_OFFLOAD(ACTIVE),CHAIN>  
>  inet 10.205.128.22 netmask 0xffffffc0 broadcast 10.205.128.63  
>  tcp_sendspace 262144 tcp_recvspace 262144 rfc1323 1
> 
>  
 发现vip不在节点上,仅剩下物理ip

 判断问题为VIP进行漂移了

 

 
## 解决方法

 
### 1.关闭实例

 
> #su - oracle  
>  $sqlplus / as sysdba  
>  SQL>shu immediate
> 
>  
 
### 2.关闭asm

 
> #su - grid  
>  $sqlplus / as sysasm  
>  SQL>shu immediate
> 
>  
 
### 3.尝试重启crs

 
> #crsctl disable crs  
>  CRS-4621: Oracle High Availability Services autostart is disabled.
> 
>  #crsctl stop crs  
>  CRS-2796: The command may not proceed when Cluster Ready Services is not running  
>  CRS-4687: Shutdown command has completed with errors.  
>  CRS-4000: Command Stop failed, or completed with errors.
> 
>  
 再次尝试启动，也是报错。

 
> #crsctl enable crs  
>  CRS-4622: Oracle High Availability Services autostart is enabled.
> 
>  #crsctl start crs  
>  CRS-4640: Oracle High Availability Services is already active  
>  CRS-4000: Command Start failed, or completed with errors.
> 
>  
 
### 4.直接kill掉crs

 最后看到mos上有一个workaround，可以手动Kill掉那些crs的进程。

 
> #ps -fea | grep ohasd.bin | grep -v grep  
>  root 30541310 1 1 16:19:17 - 3:33 /u01/grid/11.2.0/bin/ohasd.bin reboot  
>  #ps -fea | grep gipcd.bin | grep -v grep  
>  grid 2098638 1 1 16:19:40 - 2:29 /u01/grid/11.2.0/bin/gipcd.bin  
>  #ps -fea | grep mdnsd.bin | grep -v grep  
>  grid 31916408 1 0 16:19:36 - 0:01 /u01/grid/11.2.0/bin/mdnsd.bin  
>  #ps -fea | grep gpnpd.bin | grep -v grep  
>  grid 5177660 1 0 16:19:38 - 0:27 /u01/grid/11.2.0/bin/gpnpd.bin  
>  #ps -fea | grep evmd.bin | grep -v grep  
>  grid 3342370 1 0 16:33:17 - 1:28 /u01/grid/11.2.0/bin/evmd.bin  
>  #ps -fea | grep crsd.bin | grep -v grep  
>  root 3408064 1 0 16:33:46 - 4:17 /u01/grid/11.2.0/bin/crsd.bin reboot
> 
>  #kill -9 30541310 2098638 31916408 5177660 3342370 3408064
> 
>  
 
### 5.启动 crs 

 
> #crsctl start cluster  
>  CRS-2672: Attempting to start 'ora.cluster_interconnect.haip' on 'ORA2'  
>  CRS-2676: Start of 'ora.cluster_interconnect.haip' on 'ora102' succeeded  
>  CRS-2679: Attempting to clean 'ora.asm' on 'ORA2'  
>  CRS-2681: Clean of 'ora.asm' on 'ora102' succeeded  
>  CRS-2672: Attempting to start 'ora.asm' on 'ORA2'  
>  CRS-2676: Start of 'ora.asm' on 'ora102' succeeded  
>  CRS-2672: Attempting to start 'ora.crsd' on 'ORA2'  
>  CRS-2676: Start of 'ora.crsd' on 'ora102' succeeded
> 
>  
 如果还是有问题那么清理节点2的配置信息，然后重新运行root.sh

 
> #/u01/app/11.2.0/grid/crs/install/rootcrs.pl -verbose -deconfig -force  
>  #/u01/app/11.2.0/grid/crs/install/roothas.pl -verbose -deconfig -force  
>  #/u01/app/11.2.0/grid/root.sh  
>  然后检查状态是否正常，如果不正常，再次重启crs,就好了。
> 
>  
 
### 6.检查CRS状态

 
> # crs_stat -t  
>  Name Type Target State Host   
>  ------------------------------------------------------------  
>  ora....DATA.dg ora....up.type ONLINE ONLINE ORA1   
>  ora....LRDG.dg ora....up.type ONLINE ONLINE ORA1   
>  ora.GRIDDG.dg ora....up.type ONLINE ONLINE ORA1   
>  ora....ER.lsnr ora....er.type ONLINE ONLINE ORA1   
>  ora....N1.lsnr ora....er.type ONLINE ONLINE ORA1   
>  ora....DB.lsnr ora....er.type ONLINE ONLINE ORA1   
>  ora....DATA.dg ora....up.type ONLINE ONLINE ORA1   
>  ora....LRDG.dg ora....up.type ONLINE ONLINE ORA1   
>  ora.asm ora.asm.type ONLINE ONLINE ORA1   
>  ora.cvu ora.cvu.type ONLINE ONLINE ORA1   
>  ora.epm.db ora....se.type ONLINE ONLINE ORA2   
>  ora.gsd ora.gsd.type OFFLINE OFFLINE   
>  ora....network ora....rk.type ONLINE ONLINE ORA1   
>  ora.oc4j ora.oc4j.type ONLINE ONLINE ORA1   
>  ora.ons ora.ons.type ONLINE ONLINE ORA1   
>  ora....ry.acfs ora....fs.type ONLINE ONLINE ORA1   
>  ora.scan1.vip ora....ip.type ONLINE ONLINE ORA1   
>  ora....SM1.asm application ONLINE ONLINE ORA1   
>  ora....S1.lsnr application ONLINE ONLINE ORA1   
>  ora....S1.lsnr application ONLINE ONLINE ORA1   
>  ora....ss1.gsd application OFFLINE OFFLINE   
>  ora....ss1.ons application ONLINE ONLINE ORA1   
>  ora....ss1.vip ora....t1.type ONLINE ONLINE ORA1   
>  ora....SM2.asm application ONLINE ONLINE ORA2   
>  ora....S2.lsnr application ONLINE ONLINE ORA2   
>  ora....S2.lsnr application ONLINE ONLINE ORA2   
>  ora....ss2.gsd application OFFLINE OFFLINE   
>  ora....ss2.ons application ONLINE ONLINE ORA2   
>  ora....ss2.vip ora....t1.type ONLINE ONLINE ORA2   
>  ora....ssdb.db ora....se.type ONLINE ONLINE ORA1   
>  ora....db1.svc ora....ce.type ONLINE ONLINE ORA1   
>  ora....db2.svc ora....ce.type ONLINE ONLINE ORA1 
> 
>  
 OK.没问题了

 
### 7.启动asm

 
> >  #su - oracle  
>  $sqlplus / as sysdba  
>  SQL>startup
> 
>  
 
### 8.启动实例

 
> #su - oracle  
>  $sqlplus / as sysdba  
>  SQL>startup
> 
>  
 
## 检查结果

 
### 1.检查ip,发现vip已经回来

 
> #ifconfig -a  
>  en0: flags=1e084863,480<UP,BROADCAST,NOTRAILERS,RUNNING,SIMPLEX,MULTICAST,GROUPRT,64BIT,CHECKSUM_OFFLOAD(ACTIVE),CHAIN>  
>  inet 10.205.128.22 netmask 0xffffffc0 broadcast 10.205.128.63  
>  inet 10.205.128.26 netmask 0xffffffc0 broadcast 10.205.128.63  
>  tcp_sendspace 262144 tcp_recvspace 262144 rfc1323 1
> 
>  
 
### 2.检查ocr状态

 
> # ocrcheck  
>  Status of Oracle Cluster Registry is as follows :  
>  Version : 3  
>  Total space (kbytes) : 262120  
>  Used space (kbytes) : 3524  
>  Available space (kbytes) : 258596  
>  ID : 450202465  
>  Device/File Name : +GRIDDG  
>  Device/File integrity check succeeded
> 
>  Device/File not configured
> 
>  Device/File not configured
> 
>  Device/File not configured
> 
>  Device/File not configured
> 
>  Cluster registry integrity check succeeded
> 
>  
 
### 3.检查crs状态

 
> # crsctl check crs  
>  CRS-4638: Oracle High Availability Services is online  
>  CRS-4537: Cluster Ready Services is online  
>  CRS-4529: Cluster Synchronization Services is online  
>  CRS-4533: Event Manager is online
> 
>  
 
### 4.所有Oracle实例（数据库状态）

 
> #srvctl status database -dsdd  
>  Instance sdd1 is running on node ORA1  
>  Instance sdd2 is running on node ORA2
> 
>  
 
### 5.节点应用程序状态

 
> #srvctl status nodeapps  
>  VIP ORA1-vip is enabled  
>  VIP ORA1-vip is running on node: ORA1  
>  VIP ORA2-vip is enabled  
>  VIP ORA2-vip is running on node: ORA2  
>  Network is enabled  
>  Network is running on node: ORA1  
>  Network is running on node: ORA2  
>  GSD is disabled  
>  GSD is not running on node: ORA1  
>  GSD is not running on node: ORA2  
>  ONS is enabled  
>  ONS daemon is running on node: ORA1  
>  ONS daemon is running on node: ORA2
> 
>  
 
### 6.ASM状态以及ASM配置

 
> #srvctl status asm -a  
>  ASM is running on rac2,rac1
> 
>  
 

 
### 7.TNS监听器状态以及配置

 
> #srvctl config listener -a  
>  Name: LISTENER  
>  Network: 1, Owner: grid  
>  Home:  
>  /u01/app/grid/11.2.0 on node(s) ORA2,ORA1  
>  End points: TCP:1521
> 
>  
 OK问题处理完毕,客户登录成功

 
# 附上正确重启Oracle RAC服务步骤

 oracle rac服务器需要重启的时候，oracle相关资源关闭的的流程：

 
> 1）关闭实例  
>  su - oracle  
>  sqlplus / as sysdba  
>  shu immediate
> 
>  2）关闭asm实例  
>  su - grid  
>  sqlplus / as sysasm  
>  shu immediate
> 
>  3)关闭crs  
>  crsctl stop cluster -all (root执行)
> 
>  4)启动crs  
>  crsctl start cluster -all (root执行)
> 
>  5)启动asm  
>  su - oracle  
>  sqlplus / as sysdba  
>  startup
> 
>  6)启动实例  
>  su - oracle  
>  sqlplus / as sysdba  
>  startup
> 
>  
 
# 总结

 参考: [http://blog.itpub.net/29654823/viewspace-2148123/](http://blog.itpub.net/29654823/viewspace-2148123/)

 简单概述CRS架构 ：  
 1）Cluster Synchronization Services (CSS)—管理群集配置，谁是成员、谁来、谁走，通知成员。  
 2）Cluster Ready Services (CRS)—管理群集内高可用操作的主要程序，crs管理的全部内容都被看作资源，包括数据库、实例、服务、监听器、vip地址、应用进程等。Crs进程根据OCR中的配置信息管理群集资源，包括启动、停止、监视和容错操作。当某个资源的状态发生改变时，crs进程产生事件。RAC安装完成后，crs进程监视各种资源，发生异常时自动重启该资源，一般来说重启5次，如不成功不再尝试。  
 3）Event Management (EVM)—后台进程发布由crs生成的事件。  
 4）Oracle Notification Service (ONS)—通信FAN消息的发布和订阅服务。  
 5）RACG—扩展集群支持oracle特定的需求和复杂的资源。  
 6）Process Monitor Daemon (OPROCD)—锁定在内存中监视集群运行并执行I/O隔离。利用 hangchecker，监测、停止、再监测、再停止，如果醒来时时间不对则重启该节点。

 注意：  
 CRS进程栈默认随着操作系统的启动而自启动，有时出于维护目的需要关闭这个特性，可以用root用户执行下面命令。

 
> crsctl disable crs  
>  crsctl enable crs
> 
>  
 这个命令实际是修改了/etc/oracle/scls_scr/raw/root/crsstart这个文件里的内容  
 CRS由CRS，CSS，EVM三个服务组成，每个服务又是由一系列module组成，crsctl允许对每个module进行跟踪，并把跟踪内容记录到日志中。

 
> crsctl lsmodules css  
>  crsctl lsmodules evm
> 
>  
 跟踪CSSD模块，需要root用户执行： 

 
> crsctl debug log css "CSSD:1"  
>  Configuration parameter trace is now set to 1.  
>  Set CRSD Debug Module: CSSD Level: 1 
> 
>  
 查看跟踪日志

 
> pwd  
>  /u01/app/oracle/product/crs/log/rac1/cssd  
>  more ocssd.log
> 
>  
 Oracle Cluster Registry (OCR)：  
 管理Oracle集群软件和Oracle RAC数据库配置信息；类似于windows的注册表；这也包含Oracle Local Registry (OLR)，存在于集群的每个节点上，管理Oracle每个节点的集群配置信息。Oracle Clusterware 把整个集群的配置信息放在共享存储上，这个存储就是OCR Disk.在整个集群中，只有一个节点能对OCR Disk进行读写操作，这个节点叫作Master Node，所有节点都会在内存中保留一份OCR的拷贝，同时有一个OCR Process从这个内存中读取内容。OCR内容发生改变时，由Master Node的OCR Process负责同步到其他节点的OCR Process。

 Ocrcheck命令用于检查OCR内容的一致性，命令执行过程会在$CRS_HOME\log\nodename\client目录下产生ocrcheck_pid.log日志文件。 这个命令不需要参数。  
 ocrcheck

 oracle rac集群，是一个整体，需要同时启动和关闭，如果你只启动其中一个，那么另一个节点的vip就会飘到这个节点，voting disk投票把这个节点踢出集群，也就是脑裂。解决脑裂问题的基本思路就是：首先重启被踢出集群的节点的crs（crsctl stop crs ,然后crsctl start crs )，如果不行，那就清理节点2的配置信息，然后重新运行root.sh,然后执行crsctlstart crs开启crs即可。

 

 参考: [http://blog.itpub.net/29654823/viewspace-2148123/](http://blog.itpub.net/29654823/viewspace-2148123/)

   
   
   
   
   
   
   
   
   
   
 
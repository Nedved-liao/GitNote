## 一、 ASM（自动存储管理）的来由：

 ASM是Oracle 10g R2中为了简化Oracle数据库的管理而推出来的一项新功能，这是Oracle自己提供的卷管理器，主要用于替代操作系统所提供的LVM，它不仅支持单实例，同时对RAC的支持也是非常好。ASM可以自动管理磁盘组并提供有效的数据冗余功能。使用ASM（自动存储管理）后，数据库管理员不再需要对ORACLE中成千上万的数据文件进行管理和分类，从而简化了DBA的工作量，可以使得工作效率大大提高。

 
## 二、 什么是ASM

 ASM它提供了以平台无关的文件系统、逻辑卷管理以及软RAID服务。ASM可以支持条带化和磁盘镜像，从而实现了在数据库被加载的情况下添加或移除磁盘以及自动平衡I/O以删除“热点”。它还支持直接和异步的I/O并使用Oracle9i中引入的Oracle数据管理器API（简化的I/O系统调用接口）。

 ASM是做为单独的Oracle实例实施和部署，并且它只需要有参数文件，不需要其它的任何物理文件，就可以启动ASM实例，只有它在运行的时候，才能被其它数据访问。在Linux平台上，只有运行了OCSSD服务（Oracle安装程序默认安装）了才能和访问ASM。

 
## 三、 使用ASM的好处：

 1、 将I/O平均分部到所有可用磁盘驱动器上以防止产生热点，并且最大化性能。

 2、 配置更简单，并且最大化推动数据库合并的存储资源利用。

 3、 内在的支持大文件

 4、 在增量增加或删除存储容量后执行自动联系重分配

 5、 维护数据的冗余副本以提高可用性。

 6、 支持10g，11g的数据存储及RAC的共享存储管理

 7、 支持第三方的多路径软件

 8、 使用OMF方式来管理文件

 
## 四、 ASM冗余：

 ASM使用独特的镜像算法：不镜像磁盘，而是镜像盘区。作为结果，为了在产生故障时提供连续的保护，只需要磁盘组中的空间容量，而不需要预备一个热备(hot spare)磁盘。不建议用户创建不同尺寸的故障组，因为这将会导致在分配辅助盘区时产生问题。ASM将文件的主盘区分配给磁盘组中的一个磁盘时，它会将该盘区的镜像副本分配给磁盘组中的另一个磁盘。给定磁盘上的主盘区将在磁盘组中的某个伙伴磁盘上具有各自的镜像盘区。ASM确保主盘区和其镜像副本不会驻留在相同的故障组中。**磁盘组的冗余可以有如下的形式：双向镜像文件(至少需要两个故障组)的普通冗余(默认冗余)和使用三向镜像(至少需要3个故障组)提供较高保护程度的高冗余。**一旦创建磁盘组，就不可以改变它的冗余级别。为了改变磁盘组的冗余，必须创建具有适当冗余的另一个磁盘组，然后必须使用RMAN还原或DBMS_FILE_TRANSFER将数据文件移动到这个新创建的磁盘组。

 
### **三种不同的冗余方式如下：**

 1、 **外部冗余（external redundancy）：**表示Oracle不帮你管理镜像，功能由外部存储系统实现，比如通过RAID技术；有效磁盘空间是所有磁盘设备空间的大小之和。

 2、 **默认冗余（normal redundancy）：**表示Oracle提供2份镜像来保护数据，有效磁盘空间是所有磁盘设备大小之和的1/2 （使用最多）

 3、 **高度冗余（high redundancy）：**表示Oracle提供3份镜像来保护数据，以提高性能和数据的安全，最少需要三块磁盘（三个failure group）；有效磁盘空间是所有磁盘设备大小之和的1/3，虽然冗余级别高了，但是硬件的代价也最高。

 
## 五、 ASM进程

 **ASM****实例除了传统的DBWR,LGWR,CKPT,SMON,PMON等进程还包含如下四个新后台进程：**

 **RBAL：**负责协调磁盘组的重新平衡活动(负责磁盘组均衡)

 **ARB0-ARBn：**在同一时刻可以存在许多此类进程，它们分别名为ARB0、ARB1，以此类推，执行实际的重新平衡分配单元移动进程。

 **GMON：**用于ASM磁盘组监控

 **O0nn 01-10：**这组进程建立到ASM实例的连接,某些长时间操作比如创建数据文件,RDBMS会通过这些进程向ASM发送信息

 ASMB与ASM 实例的前台进程连接，周期性的检查两个instance的健康状况。每个数据库实例同时只能与一个ASM实例连接，因此数据库只会有一个ASMB后台进程。如一个节点上有多个数据库实例，它们只能共享一个ASM实例。

 RBAL用来进行全局调用，以打开某个磁盘组内的磁盘。ASMB进程与该节点的CSS守护进程进行通信，并接收来自ASM实例的文件区间映射信息。ASMB还负责为ASM实例提供I/O统计数据

 CSS集群同步服务。要使用ASM，必须确保已经运行了CSS集群同步服务，CSS负责ASM实例和数据库实例之间的同步。

 **注意：**ASM实例必须要先于数据库实例启动，和数据库实例同步运行，迟于数据库实例关闭。ASM 实例和数据库实例的关系可以是1：1，也可以是1：n。如果是1：n，最好为ASM 安装单独的ASM_HOME。

 
## **六、 ASM支持datafile，logfiles，control files，archivelogs，RMAN backup sets等自动的数据库文件管理**

 
## **七、** ASM实例和数据库实例对应关系

 

 ![](https://files.jb51.net/file_images/article/201311/201311202207222.jpg)

 八、 Cluster ASM 架构

 ![](https://files.jb51.net/file_images/article/201311/201311202207223.jpg)

 **如需了解更详细信息请参见Oracle数据库管理员指南（Oracle首次放出）**：

 http://docs.oracle.com/cd/B28359_01/server.111/b31107/toc.htm

 好了，现在开始谈谈有关于ASM安装的相关内容，ASM的安装必须建立在操作系统和数据库软件已经安装完成的及实例未创建之前来进行安装，之后再进行选择ASM方式建库。ASM不仅可以应用于单实例的数据库，同时更适用于RAC集群方式的数据库，并且ASM只被Oracle所认，同时也是ORACLE最佳的存储解决方案，可以有效的替代RAID技术和卷管理技术，比裸设备的管理更加方便；所以现在大部分企业都在迅速的向ASM技术迁移。

 在上面我们已经探讨过了ASM的三种模式，及其的一些应用，在这里我们就不对其进行过多的累述。正式进入这篇的主题，如何安装ASM软件，安装ASM需要具备哪些条件。

 
## **ASM安装步骤：**

 
### **一、基础环境准备**

 1、 检查操作系统和数据库软件是否安装完成：

 Installation in progress (Mon Apr 09 19:12:44 CST 2012)

 ............................................................... 18% Done.

 ............................................................... 36% Done.

 ............................................................... 54% Done.

 ............................................................... 73% Done.

 ............ 76% Done.

 Install successful

 Linking in progress (Mon Apr 09 19:19:34 CST 2012)

 Link successful

 Setup in progress (Mon Apr 09 19:23:13 CST 2012)

 .............. 100% Done.

 Setup successful

 End of install phases.(Mon Apr 09 19:23:26 CST 2012)

 WARNING:A new inventory has been created in this session. However, it has not yet been registered as the central inventory of this system.

 To register the new inventory please run the script '/oracle/oraInventory/orainstRoot.sh' with root privileges.

 If you do not register the inventory, you may not be able to update or patch the products you installed.

 The following configuration scripts

 /oracle/orahome/10.2.0/db_1/root.sh

 need to be executed as root for configuring the system. If you skip the execution of the configuration tools, the configuration will not be complete and the product wont function properly. In order to get the product to function properly, you will be required to execute the scripts and the configuration tools after exiting the OUI.

 The installation of Oracle Database 10g was successful.

 

 从如上信息我们可以看到数据库已经安装完成，操作系统肯定也是没有问题的。

 
### 2、 检查数据库和操作系统版本：

 [oracle@ jb51.net db_1]$ lsb_release -a

 LSB Version: :core-3.1-ia32:core-3.1-noarch:graphics-3.1-ia32:graphics-3.1-noarch

 Distributor ID: EnterpriseEnterpriseServer

 Description: Enterprise Linux Enterprise Linux Server release 5.4 (Carthage)

 Release: 5.4

 Codename: Carthage

 [oracle@ jb51.net db_1]$

 

 [oracle@ jb51.net db_1]$ uname -a

 Linux wwl 2.6.18-164.el5 #1 SMP Thu Sep 3 02:16:47 EDT 2009 i686 i686 i386 GNU/Linux

 

 操作系统版本为5.4 X86，内核版本为2.6.18-164.el5，后面下载ASM包必须要对应

 

 [oracle@ jb51.net db_1]$ sqlplus / as sysdba

 SQL*Plus: Release 10.2.0.1.0 - Production on Mon Apr 9 19:41:54 2012

 Copyright (c) 1982, 2005, Oracle. All rights reserved.

 Connected to an idle instance.

 SQL>

 数据库版本是10.2.0.1.

 
### 3、 我们已经知道了这些信息后，我们就可以有针对性的下载ASM了:

 ASM下载地址，版本不一样，用的ASM包也不一样：

 http://www.oracle.com/technetwork/server-storage/linux/downloads/rhel5-084877.html

 找到Intel IA32 (x86) Architecture系列中的这个包下载下来：

 Drivers for kernel 2.6.18-164.el5

 · oracleasm-2.6.18-164.el5-2.0.5-1.el5.i686.rpm

 · 以及如下两个包下载下来就可以了：

 Library and Tools

 · oracleasm-support-2.1.7-1.el5.i386.rpm

 · oracleasmlib-2.0.4-1.el5.i386.rpm

 · 

 
### 4、 下载完了之后开始安装asm的rpm包，**用root用户安装，注意安装顺序，如下:**

 

 [root@wwl asmpark]# ls

 oracleasm-2.6.18-164.el5-2.0.5-1.el5.i686.rpm

 oracleasmlib-2.0.4-1.el5.i386.rpm

 oracleasm-support-2.1.7-1.el5.i386.rpm

 [root@wwl asmpark]# rpm -ivh oracleasm-support-2.1.7-1.el5.i386.rpm

 warning: oracleasm-support-2.1.7-1.el5.i386.rpm: Header V3 DSA signature: NOKEY, key ID 1e5e0159

 Preparing... ########################################### [100%]

 1:oracleasm-support ########################################### [100%]

 [root@wwl asmpark]# rpm -ivh oracleasm-2.6.18-164.el5-2.0.5-1.el5.i686.rpm

 warning: oracleasm-2.6.18-164.el5-2.0.5-1.el5.i686.rpm: Header V3 DSA signature: NOKEY, key ID 1e5e0159

 Preparing... ########################################### [100%]

 1:oracleasm-2.6.18-164.el########################################### [100%]

 [root@wwl asmpark]# rpm -ivh oracleasmlib-2.0.4-1.el5.i386.rpm

 warning: oracleasmlib-2.0.4-1.el5.i386.rpm: Header V3 DSA signature: NOKEY, key ID 1e5e0159

 Preparing... ########################################### [100%]

 1:oracleasmlib ########################################### [100%]

 [root@wwl asmpark]#

 
### 5、 好了，现在ASM相关包已经安装完成，现在来开始创建用于ASM的磁盘分区（不是一定要做，裸盘也可以做ASM）

 

 [root@wwl asmpark]# fdisk -l

 Disk /dev/sda: 16.1 GB, 16106127360 bytes

 255 heads, 63 sectors/track, 1958 cylinders

 Units = cylinders of 16065 * 512 = 8225280 bytes

 Device Boot Start End Blocks Id System

 /dev/sda1 * 1 13 104391 83 Linux

 /dev/sda2 14 1958 15623212+ 8e Linux LVM

 Disk /dev/sdb: 10.7 GB, 10737418240 bytes

 255 heads, 63 sectors/track, 1305 cylinders

 Units = cylinders of 16065 * 512 = 8225280 bytes

 Disk /dev/sdb doesn't contain a valid partition table

 Disk /dev/sdc: 10.7 GB, 10737418240 bytes

 255 heads, 63 sectors/track, 1305 cylinders

 Units = cylinders of 16065 * 512 = 8225280 bytes

 Disk /dev/sdc doesn't contain a valid partition table

 [root@wwl asmpark]#

 **我们从上图可以看出系统中有两块空闲的磁盘没有使用，我们首先需要对磁盘创建分区，但不能格式化，命令如下:**

 fdisk /dev/sdb /n/p/1/回车/回车/w

 fdisk /dev/sdc /n/p/1/回车/回车/w

 如下就已经创建好了分区：

 

 [root@wwl asmpark]# fdisk -l

 Disk /dev/sda: 16.1 GB, 16106127360 bytes

 255 heads, 63 sectors/track, 1958 cylinders

 Units = cylinders of 16065 * 512 = 8225280 bytes

 Device Boot Start End Blocks Id System

 /dev/sda1 * 1 13 104391 83 Linux

 /dev/sda2 14 1958 15623212+ 8e Linux LVM

 Disk /dev/sdb: 10.7 GB, 10737418240 bytes

 255 heads, 63 sectors/track, 1305 cylinders

 Units = cylinders of 16065 * 512 = 8225280 bytes

 Device Boot Start End Blocks Id System

 /dev/sdb1 1 1305 10482381 83 Linux

 Disk /dev/sdc: 10.7 GB, 10737418240 bytes

 255 heads, 63 sectors/track, 1305 cylinders

 Units = cylinders of 16065 * 512 = 8225280 bytes

 Device Boot Start End Blocks Id System

 /dev/sdc1 1 1305 10482381 83 Linux

 
## **二、ASM配置**

 以上已将准备环境准备好，下一步骤就是开始配置了，这里面配置包括如下几个步骤

 开始创建ASM实例，创建ASM实例的方式有两种，一种是通过命令行，还有一种是通过图形界面，执行DBCA后有一步是创建ASM实例，图形界面实在太简单了，跟创建DB是一样的，在这里就不累赘了。

 **在使用ASM之前首先要配置ASMLib驱动程序，如下：**

 我们首先可以看下asm的配置工具 oracleasm的语法和功能，如下：

 [root@wwl asmpark]# /etc/init.d/oracleasm --help

 Usage: /etc/init.d/oracleasm {start|stop|restart|enable|disable|configure|createdisk|deletedisk|querydisk|listdisks|scandisks|status}

 [root@wwl asmpark]#

 
### 1、开始配置ASMLib：

 [root@wwl asmpark]# /etc/init.d/oracleasm configure

 Configuring the Oracle ASM library driver.

 This will configure the on-boot properties of the Oracle ASM library

 driver. The following questions will determine whether the driver is

 loaded on boot and what permissions it will have. The current values

 will be shown in brackets ('[]'). Hitting <ENTER> without typing an

 answer will keep that current value. Ctrl-C will abort.

 Default user to own the driver interface []: oracle

 Default group to own the driver interface []: dba

 Start Oracle ASM library driver on boot (y/n) [n]: y

 Scan for Oracle ASM disks on boot (y/n) [y]: y

 Writing Oracle ASM library driver configuration: done

 Initializing the Oracle ASMLib driver: [ OK ]

 Scanning the system for Oracle ASMLib disks: [ OK ]

 
### 2、启用ASMLib驱动程序：

 [root@wwl asmpark]# /etc/init.d/oracleasm enable

 Writing Oracle ASM library driver configuration: done

 Initializing the Oracle ASMLib driver: [ OK ]

 Scanning the system for Oracle ASMLib disks: [ OK ]

 
### 3、通过以root用户身份运行以下命令来标记由 ASMLib 使用的磁盘：

 

 [root@wwl asmpark]# /etc/init.d/oracleasm createdisk VOL1 /dev/sdb1

 Marking disk "VOL1" as an ASM disk: [ OK ]

 [root@wwl asmpark]# /etc/init.d/oracleasm createdisk VOL2 /dev/sdc1

 Marking disk "VOL2" as an ASM disk: [ OK ]

 [root@wwl asmpark]#

 
### 4、通过如下命令查看ASM所能使用的磁盘及状态，一切显示都是正常的。

 [root@wwl asmpark]# oracleasm querydisk VOL1

 Disk "VOL1" is a valid ASM disk

 [root@wwl asmpark]# oracleasm querydisk /dev/sdb1

 Device "/dev/sdb1" is marked an ASM disk with the label "VOL1"

 [root@wwl asmpark]# oracleasm querydisk VOL2

 Disk "VOL2" is a valid ASM disk

 [root@wwl ~]# oracleasm querydisk /dev/sdc1

 Device "/dev/sdc1" is marked an ASM disk with the label "VOL2"

 [root@wwl asmpark]# oracleasm listdisks

 VOL1

 VOL2

 [root@wwl asmpark]# ls -l /dev/oracleasm/disks/*

 brw-rw---- 1 oracle dba 8, 17 Apr 10 00:25 /dev/oracleasm/disks/VOL1

 brw-rw---- 1 oracle dba 8, 33 Apr 10 00:25 /dev/oracleasm/disks/VOL2

 [root@wwl asmpark]# oracleasm status

 Checking if ASM is loaded: yes

 Checking if /dev/oracleasm is mounted: yes

 
### 5、如上ASMLib已经安装完成，并且也将磁盘标记为可用，接下来要做的就是创建ASM实例，并构建一个使用ASM磁盘来存储数据的数据库，可以使用DBCA，当然也可以使用手工的方式来创，在这里，我就采用手工方式创建ASM实例，步骤如下：

 
### 6、创建初始化参数文件,信息如下：

 [oracle@ jb51.net dbs]$ vi $ORACLE_HOME/dbs/init$ORACLE_SID.ora

 asm_diskstring='WWL:VOL*'

 background_dump_dest='/oracle/admin/+ASM/bdump'

 core_dump_dest='/oracle/admin/+ASM/cdump'

 user_dump_dest='/oracle/admin/+ASM/udump'

 instance_type='asm'

 large_pool_size=12M

 remote_login_passwordfile='SHARED'

 
### 7、增加实例信息到/etc/oratab

 $ vi /etc/oratab

 +ASM:/u01/app/oracle/product/10.2.0/db_1:Y

 
### 8、创建ASM实例密码文件：

 [oracle@ jb51.net dbs]$ $ORACLE_HOME/bin/orapwd file=$ORACLE_HOME/dbs/orapw$ORACLE_SID password='oracle' force=y;

 [oracle@ jb51.net dbs]$

 
### 9、创建ASM实例相应的目录:

 [oracle@ jb51.net dbs]$ mkdir -p $ORACLE_BASE/admin/$ORACLE_SID/bdump

 [oracle@ jb51.net dbs]$ mkdir -p $ORACLE_BASE/admin/$ORACLE_SID/cdump

 [oracle@ jb51.net dbs]$ mkdir -p $ORACLE_BASE/admin/$ORACLE_SID/udump

 [oracle@ jb51.net dbs]$

 
### 10、开启CSS服务

 $ su - root

 [root@wwl ~]# /oracle/orahome/10.2.0/db_1/bin/localconfig add

 /etc/oracle does not exist. Creating it now.

 Successfully accumulated necessary OCR keys.

 Creating OCR keys for user 'root', privgrp 'root'..

 Operation successful.

 Configuration for local CSS has been initialized

 Adding to inittab

 Startup will be queued to init within 90 seconds.

 Checking the status of new Oracle init process...

 Expecting the CRS daemons to be up within 600 seconds.

 CSS is active on these nodes.

 wwl

 CSS is active on all nodes.

 Oracle CSS service is installed and running under init(1M)

 
### 11、启动ASM实例，并创建ASM磁盘组：

 [oracle@ jb51.net dbs]$ sqlplus / as sysdba

 SQL*Plus: Release 10.2.0.1.0 - Production on Tue Apr 10 01:23:44 2012

 Copyright (c) 1982, 2005, Oracle. All rights reserved.

 Connected to:

 Oracle Database 10g Enterprise Edition Release 10.2.0.1.0 - Production

 With the Partitioning, OLAP and Data Mining options

 SQL> startup nomount;

 ASM instance started

 Total System Global Area 83886080 bytes

 Fixed Size 1217836 bytes

 Variable Size 57502420 bytes

 ASM Cache 25165824 bytes

 SQL> select instance_name,status from v$instance;

 INSTANCE_NAME STATUS

 ---------------- ------------

 +ASM STARTED

 SQL>

 现在实例我已经将其启动到nomount状态，下一步开始创建ASM磁盘组。

 
### 12、创建ASM组并将其启动到MOUNT状态，

 SQL> create diskgroup ASMGROUP1 normal redundancy disk '/dev/oracleasm/disks/VOL1','/dev/oracleasm/disks/VOL2';

 Diskgroup created.

 好了，磁盘组已经创建好了，并且也已经挂载了

 SQL> select name,state from v$asm_diskgroup;

 NAME STATE

 ------------------------------ -----------

 ASMGROUP1 MOUNTED

 SQL>

 可以看到如下，参数文件也随着更新了：

 SQL> show parameter asm_diskgroups;

 NAME TYPE VALUE

 ------------------------------------ ----------- ------------------------------

 asm_diskgroups string ASMGROUP1

 
### 13、检查ASM进程是否都正常启动了,我们之前提到的几个进程名称，这里面都有了，说明现在ASM已经是正常运行状态。

 [oracle@ jb51.net ~]$ ps -ef | grep asm

 oracle 3887 1 0 02:58 ? 00:00:00 asm_pmon_+ASM

 oracle 3889 1 0 02:58 ? 00:00:00 asm_psp0_+ASM

 oracle 3891 1 0 02:58 ? 00:00:00 asm_mman_+ASM

 oracle 3893 1 0 02:58 ? 00:00:00 asm_dbw0_+ASM

 oracle 3895 1 0 02:58 ? 00:00:00 asm_lgwr_+ASM

 oracle 3897 1 0 02:58 ? 00:00:00 asm_ckpt_+ASM

 oracle 3899 1 0 02:58 ? 00:00:00 asm_smon_+ASM

 oracle 3901 1 0 02:58 ? 00:00:00 asm_rbal_+ASM

 oracle 3903 1 0 02:58 ? 00:00:00 asm_gmon_+ASM

 ![linux](https://files.jb51.net/file_images/article/201311/201311202207224.gif)

 一、通过ASM方式建立单实例库

 ![](https://files.jb51.net/file_images/article/201311/201311202207225.png)

 ![](https://files.jb51.net/file_images/article/201311/201311202207226.png)

 ![](https://files.jb51.net/file_images/article/201311/201311202207227.png)

 二、检查通过ASM建库后，文件存储的状态：

 SQL>select file_name,tablespace_name from dba_data_files;

 FILE_NAME TABLESPACE_NAME

 -----------------------------------------------------------------

 +ASMGROUP1/wwl/datafile/users.259.780215953 USERS

 +ASMGROUP1/wwl/datafile/sysaux.257.780215951 SYSAUX

 +ASMGROUP1/wwl/datafile/undotbs1.258.780215953 UNDOTBS1

 +ASMGROUP1/wwl/datafile/system.256.780215951 SYSTEM

 SQL>

 我们由如上可以看出，现在数据都是存储在ASM新建的+ASMGROUP1的组里面，并且文件名后面跟了一大串的数字，这是因为我们在新建表空间的时候直接采用就是Oracle OMF规范来进行创建的（OMF实际上是9i里面就已经推出来的功能了），在ASM中创建表空间和添加数据文件我们就没有必要指定数据文件的存放路径了，当然他跟db_create_file_dest这个参数是相关联的。

 如下通过OMF方式创建表空间和添加数据文件的方式，可以看到很方便，默认大小就是100M，会自动扩展：

 **1、我们通过查看db_create_file_dest参数，发现了数据文件默认创建路径是在+ASMGROUP1**

 SQL> show parameter db_create_file_dest

 NAME TYPE VALUE

 ----------------------------------------------- ------------------------------

 db_create_file_dest string +ASMGROUP1

 **2、使用OMF特性来进行表空间的创建**

 SQL> create tablespace asm;

 Tablespace created.

 SQL> alter tablespace asm add datafile;

 Tablespace altered.

 **3、检查表空间是否已创建好**

 通过如下，我们可以看到，表空间已经创建成功，并且已经开启了数据文件自动扩展功能。

 SQL> selectFILE_NAME,tablespace_name,bytes/1024/1024,AUTOEXTENSIBLE,MAXBYTES/1024/1024from dba_data_files where TABLESPACE_NAME='ASM';

 FILE_NAME TABLESPACE_NAMEBYTES/1024/1024 AUT MAXBYTES/1024/1024

 ----------------------------------------------------------------- --------------- --- ------------------

 +ASMGROUP1/wwl/datafile/asm.270.780300769 ASM 100 YES 32767.9844

 +ASMGROUP1/wwl/datafile/asm.271.780300809 ASM 100 YES 32767.9844

 **4、可以动态的修改数据库创建文件的位置**

 SQL> alter system set db_create_file_dest='/oracle/oradata/wwl';

 System altered.

 ![linux](https://files.jb51.net/file_images/article/201311/201311202207224.gif)

 **一、 ASM实例相关操作：**

 **ASM实例的管理，启动，关闭**

 ASM实例的启动和数据库实例的启动有严格的先后关系，ASM启动一定早于数据库实例，关闭一定晚于ASM实例，因为它是数据库数据文件存储位置。如果ASM没有起来，起数据库将会报ORA-17503；ORA-15077的错误，错误信息如下:

 SQL>startup

 ORA-01078:failure in processing system parameters

 ORA-01565:error in identifying file '+ASMGROUP1/WWL/spfileWWL.ora'

 ORA-17503:ksfdopn:2 Failed to open file +ASMGROUP1/WWL/spfileWWL.ora

 ORA-15077:could not locate ASM instance serving a required diskgroup

 SQL>

 1.1 ASM启动的方法:

 SQL>startup

 ASMinstance started

 TotalSystem Global Area 83886080 bytes

 FixedSize 1217836 bytes

 VariableSize 57502420 bytes

 ASMCache 25165824 bytes

 ASMdiskgroups mounted

 SQL>select instance_name,status from v$instance;

 INSTANCE_NAME STATUS

 ----------------------------

 +ASM STARTED

 SQL>

 1.2 ASM关闭的方法 （必须先关闭数据库）

 **没有关闭RDBMS实例关闭ASM将报错ORA-15097，提示已连接RDBMS实例，无法关闭ASM实例**，

 **$ export Oracle_SID=+ASM**

 **$ sqlplus / as sysdba**

 **SQL> shutdown immediate**

 **ORA-15097: cannot SHUTDOWNASM instance with connected RDBMS instance**

 **关闭RDBMS实例状态ASM是可以正常关闭的。**

 **$export ORACLE_SID=WWL ---**先关闭在ASM上运行的RDBMS实例

 **$sqlplus / as sysdba**

 SQL> shutdown immediate

 Database closed.

 Database dismounted.

 ORACLE instance shut down.

 **SQL>**

 **$export ORACLE_SID=+ASM ---**再关闭ASM实例

 **$ sqlplus / as sysdba**

 SQL> shutdown immediate

 ASM diskgroups dismounted

 ASM instance shutdown

 **SQL>**

 

 **二、 ASM三种磁盘组及磁盘的添加和维护**

 **1、****ASM磁盘的添加及删除**

 1.1 添加这个步骤所需的磁盘（/dev/sdd -- /dev/sdm 共10块10G的盘）

 1.2 通过root用户查看下当前有几个ASM磁盘，磁盘状态，实例状态

 **# oracleasm listdisks**

 VOL1

 VOL2

 **# oracleasm querydisk VOL1**

 Disk"VOL1" is a valid ASM disk

 **# oracleasm querydisk VOL2**

 Disk"VOL2" is a valid ASM disk

 **# ls -l /dev/oracleasm/disks/***

 brw-rw---- 1oracle dba 8, 17 Apr 12 05:30 /dev/oracleasm/disks/VOL1

 brw-rw---- 1oracle dba 8, 33 Apr 12 05:30 /dev/oracleasm/disks/VOL2

 **# oracleasm status**

 Checking if ASMis loaded: yes

 Checking if /dev/oracleasm is mounted: yes

 我们已知数据库当有两块通过ASMLiB已经标记了的磁盘，并且状态是正常的

 1.3 开始通过ASMLib来标记新的磁盘,用于后面的实验:

 l 报错了，很经典，是由于没有创建分区导致：

 **# /etc/init.d/oracleasm createdisk VOL3 /dev/sdd**

 Marking disk"VOL3" as an ASM disk: [FAILED]

 l 先创建分区方法:fdisk /dev/sdd /n/p/1/回车/回车/w，将所有磁盘都创建分区。

 **# /etc/init.d/oracleasmcreatedisk VOL3 /dev/sdd1**

 Marking disk "VOL3" as anASM disk: [ OK ] ---可以看到，能正常创建

 **# sh oracleasm** 通过执行脚本命令，新建10个磁盘已全部完成标记

 Marking disk "VOL4" as an ASM disk: [ OK ]

 Marking disk "VOL5" as an ASM disk: [ OK ]

 Marking disk "VOL6" as an ASM disk: [ OK ]

 Marking disk "VOL7" as an ASM disk: [ OK ]

 Marking disk "VOL8" as an ASM disk: [ OK ]

 Marking disk "VOL9" as an ASM disk: [ OK ]

 Marking disk "VOL10" as an ASM disk: [ OK ]

 Marking disk "VOL11" as an ASM disk: [ OK ]

 Marking disk "VOL12" as an ASM disk: [ OK ]

 1.4 为ASMGROUP1磁盘组添加删除磁盘

 l 查看磁盘组的状态

 SQL> selectGROUP_NUMBER,NAME,STATE,TYPE from v$asm_diskgroup;

 GROUP_NUMBER NAME STATE TYPE

 ------------------------ ----------------- -------------- --------

 1 ASMGROUP1 CONNECTED NORMAL

 SQL> SELECT a.name GRPNAME,b.group_number GR_NUMBER,b.disk_numberDK_NUMBER,b.name ASMFILE,b.path,b.mount_status,b.state FROM v$asm_diskgroupa,v$asm_disk b;

 GRPNAME GR_NUMBER DK_NUMBER ASMFILE PATH MOUNT_S STATE

 ---------- ---------- ---------------------------------------- ------------------------- ------- --------

 ASMGROUP1 1 0ASMGROUP1_0000 /dev/oracleasm/disks/VOL1OPENED NORMAL

 ASMGROUP1 1 1ASMGROUP1_0001 /dev/oracleasm/disks/VOL2 OPENED NORMAL

 l 查看磁盘组ASMGROUP1中的成员

 SQL> selectgroup_number,disk_number, failgroup,name,path from v$asm_disk where FAILGROUPlike 'ASMGROUP1%';

 GROUP_NUMBERDISK_NUMBER FAILGROUP NAME PATH

 ----------------------- ------------------------------ ----------------------------------------------------------------------

 2 1 ASMGROUP1_0001 ASMGROUP1_0001 /dev/oracleasm/disks/VOL2

 2 0 ASMGROUP1_0000 ASMGROUP1_0000 /dev/oracleasm/disks/VOL1

 SQL>

 

 l 添加为ASMGROUP1添加磁盘

 SQL> alterdiskgroup ASMGROUP1 add disk '/dev/oracleasm/disks/VOL10';

 Diskgroupaltered.

 

 l 我们可以看到已经添加成功了

 SQL> select group_number,disk_number,failgroup,name,path from v$asm_disk where FAILGROUP like 'ASMGROUP1%';

 GROUP_NUMBERDISK_NUMBER FAILGROUP NAME PATH

 ----------------------- ------------------------------ ----------------------------------------------------------------------

 2 2 ASMGROUP1_0002 ASMGROUP1_0002 /dev/oracleasm/disks/VOL10

 2 1 ASMGROUP1_0001 ASMGROUP1_0001 /dev/oracleasm/disks/VOL2

 2 0 ASMGROUP1_0000 ASMGROUP1_0000 /dev/oracleasm/disks/VOL1

 

 **2、****ASM三种磁盘组的创建及删除（High Normal Extermal**）

 2.1 创建High级别的ASM磁盘组，最少需要三块磁盘来创建。

 SQL> create diskgroup asmhigh high redundancy disk'/dev/oracleasm/disks/VOL3','/dev/oracleasm/disks/VOL4','/dev/oracleasm/disks/VOL5';

 Diskgroupcreated.

 

 2.2 创建Normal级别的ASM磁盘，最少需要两个磁盘来创建。

 SQL> creatediskgroup asmnormal normal redundancy disk'/dev/oracleasm/disks/VOL6','/dev/oracleasm/disks/VOL7';

 Diskgroupcreated.

 

 2.3 创建Extermal级别的ASM磁盘，最少需要一个磁盘来创建。

 SQL> creatediskgroup asmexternal external redundancy disk '/dev/oracleasm/disks/VOL8';

 Diskgroupcreated.

 

 

 2.4 查看刚才创建的磁盘状态

 SQL> select name,state,type fromv$asm_diskgroup;

 NAME STATE TYPE

 --------------- ----------- ------

 ASMGROUP1 MOUNTED NORMAL

 ASMHIGH MOUNTED HIGH

 ASMNORMAL MOUNTED NORMAL

 ASMEXTERNAL MOUNTED EXTERN

 

 2.5 为ASM磁盘组添加成员，在这里我们就以Normal磁盘组来进行成员添加的例子：

 SQL> alter diskgroup ASMNORMAL add disk'/dev/oracleasm/disks/VOL9';

 Diskgroup altered.

 SQL> select group_number,disk_number,failgroup,name,path from v$asm_disk where FAILGROUP like 'ASMNORMAL%';

 GROUP_NUMBER DISK_NUMBER FAILGROUP NAME PATH

 ------------ ----------------------------------------- ------------------------------ ----------------------------------------

 4 2 ASMNORMAL_0002 ASMNORMAL_0002 /dev/oracleasm/disks/VOL9

 4 1 ASMNORMAL_0001 ASMNORMAL_0001 /dev/oracleasm/disks/VOL7

 4 0 ASMNORMAL_0000 ASMNORMAL_0000 /dev/oracleasm/disks/VOL6

 

 SQL>

 2.6 删除磁盘组成员，在这里我们同样以NORMAL磁盘组来进行成员删除的例子：

 SQL> alter diskgroup ASMNORMAL drop disk ASMNORMAL_0002;

 Diskgroup altered.

 SQL> select group_number,disk_number, failgroup,name,path fromv$asm_disk where FAILGROUP like 'ASMNORMAL%';

 GROUP_NUMBER DISK_NUMBER FAILGROUP NAME PATH

 ------------ ----------- ------------------------------ ----------------------------------------------------------------------

 4 1 ASMNORMAL_0001 ASMNORMAL_0001 /dev/oracleasm/disks/VOL7

 4 0 ASMNORMAL_0000 ASMNORMAL_0000 /dev/oracleasm/disks/VOL6

 SQL>

 

 **三、 模拟磁盘故障 **

 3.1 **在AMSGROUP1(NORMAL类型)磁盘组中写数据**

 

 SQL> selecttablespace_name,file_name,bytes/1024/1024 M from dba_data_files;

 TABLESPACE_NAMEFILE_NAME M

 ------------------------------------------------------------ ----------

 USERS +ASMGROUP1/wwl/datafile/users.259.780215953 5

 SYSAUX +ASMGROUP1/wwl/datafile/sysaux.257.780215951 230

 UNDOTBS1 +ASMGROUP1/wwl/datafile/undotbs1.258.78021595 25

 3

 SYSTEM +ASMGROUP1/wwl/datafile/system.256.780215951 480

 ASM +ASMGROUP1/wwl/datafile/asm.270.780300769 100

 ASM +ASMGROUP1/wwl/datafile/asm.271.780300809 100

 6 rowsselected.

 如上我们可以看到，我们所有的表空间均是放在ASMGROUP1中的，一会儿我们将对表空间写如数据，并删除一磁盘。

 3.2 **我们查看下该表空间的默认用户**

 SQL> selectusername,default_tablespace from dba_users where DEFAULT_TABLESPACE='ASM';

 USERNAME DEFAULT_TABLESPACE

 ------------------------------------------------------------

 WWL ASM

 

 3.3 **在ASM表空间写入数据。**

 通过WWL用户登录到系统创建一张表，用来测试.

 SQL> connwwl/wwl

 Connected.

 SQL> createtable wwl (id varchar(5),name varchar(10));

 Table created.

 SQL> begin

 2 fori in 1..1000 loop

 3 insert into wwl values (15,'wwl15');

 4 endloop;

 5 end;

 6 /

 PL/SQLprocedure successfully completed.

 

 我们创建了一张wwl的表，并且插入了1000行数据

 SQL> selectcount(*) from wwl;

 COUNT(*)

 ----------

 1000

 3.4 **模拟磁盘突然损坏**

 [root@wwl ~]#oracleasm deletedisk VOL2;

 Clearing diskheader: done

 Dropping disk:done

 [root@wwl ~]#

 仔细看下面，我们通过如上的命令删除了VOL2后，现在只认到一个磁盘了。

 SQL> selectgroup_number,disk_number, failgroup,name,path from v$asm_disk where FAILGROUPlike 'ASMGROUP%';

 GROUP_NUMBERDISK_NUMBER FAILGROUP NAME PATH

 ----------------------- ------------------------------ ----------------------------------------------------------------------

 2 0 ASMGROUP1_0000 ASMGROUP1_0000 /dev/oracleasm/disks/VOL1

 SQL>

 

 但是我们的实例和我们刚才创建的表数据都没有丢失，这就是冗余的好处，NORMAL模式它是用牺牲一块磁盘的空间来保障数据的安全性的，hight模式是至少牺牲一块硬盘来保障数据的安全性。

 SQL> selectcount(*) from wwl;

 COUNT(*)

 ----------

 1000

 3.5 **而且业务是不会中断的，但是在日志和硬盘指示灯上会有告警：**

 ASM日志信息如下：

 WARNING:offlining disk 2.3916240783 (ASMGROUP1_0002) with mask 0x1

 NOTE: PSTupdate: grp = 2, dsk = 2, mode = 0x6

 NOTE: cacheclosing disk 2 of grp 2: ASMGROUP1_0002

 NOTE: PSTupdate: grp = 2

 NOTE: erasingheader on grp 2 disk ASMGROUP1_0002

 3.6 **这个时候我们需要尽快更换新的硬盘，因为发生这问题之后如果另外一个磁盘再损坏的话那将是不可弥补的数据丢失，更换新硬盘后，数据将会再次进行同步。**

 3.7 ** **

 **四、 ASM别名管理**

 别名就是外号，比如说当系统自动产生的名称太过复杂不怎么好记，DBA可以通过别名，为它创建一个简单化的名称，而又不会对其现有名称造成任何影响。ASM中创建别名是通过alter diskgroup的alias子句实现，支持增加/修改/删除等多项操作。V$ASM_ALIAS视图中可以查询到当前实例中创建的别名。

 4.1 添加别名

 SQL> alter diskgroup ASMGROUP1 add alias'+ASMGROUP1/wwl/datafile/asm01.dbf' for'+ASMGROUP1/wwl/datafile/asm.270.780300769';

 Diskgroup altered.

 4.2 修改别名

 SQL> alter diskgroup ASMGROUP1 renamealias '+ASMGROUP1/wwl/datafile/asm01.dbf' for'+ASMGROUP1/wwl/datafile/asm.270.780300769';

 Diskgroup altered.

 4.3 删除别名

 SQL> alter diskgroup ASMGROUP1 dropalias '+ASMGROUP1/wwl/datafile/asm01.dbf' for'+ASMGROUP1/wwl/datafile/asm.270.780300769';

 Diskgroup altered.

 ** 无论是添加、删除或是修改别名，对原文件路径均不会有影响。**

 

 **五、 目录及目录文件管理**

 5.1 创建目录

 SQL> alter diskgroup ASMGROUP1 add directory '+ASMGROUP1/WWL1';

 Diskgroupaltered.

 5.2 修改目录

 SQL> alterdiskgroup ASMGROUP1 rename directory '+ASMGROUP1/WWL1' to '+ASMGROUP1/WWL2';

 Diskgroupaltered.

 5.3 删除目录

 SQL> alter diskgroup ASMGROUP1 drop directory '+ASMGROUP1/WWL2';

 Diskgroupaltered.

 **六、 手动平衡磁盘组**

 **一般情况下ASM都会自动对其下的磁盘组进行平衡，不过ORACLE也提供了手动平衡磁盘组的方式，通过alter diskgroup ... power 语句。前面提到过磁盘组的平衡度有0到11多个级别，默认是按照ASM_POWER_LIMIT初始化参数中设置的值，手动平衡的话，设置的平衡度可以与初始化参数中并不相同，例如，设置磁盘组平衡度为5，语句如下：**

 SQL>alter diskgroup asmgroup1 rebalance power 5;

 Diskgroup altered.

 **七、 通过ASMCMD工具管理ASM**

 [oracle@wwl ~]$ which asmcmd

 /oracle/orahome/10.2.0/db_1/bin/asmcmd

 [oracle@wwl ~]$ cd/oracle/orahome/10.2.0/db_1/bin/

 ASMCMD> ls 

 ASMEXTERNAL/

 ASMGROUP1/

 ASMHIGH/

 ASMNORMAL/

 ASMCMD>

 ASMCMD> help

 asmcmd [-p] [command]

 The environment variables ORACLE_HOME and ORACLE_SID determine the

 instance to which the program connects, and ASMCMD establishes a

 bequeath connection to it, in the same manner as a SQLPLUS / AS

 SYSDBA. The user must be a memberof the SYSDBA group.

 Specifying the -p option allows the current directory to be displayed

 in the command prompt, like so:

 ASMCMD [+DATAFILE/ORCL/CONTROLFILE] >

 [command] specifies one of the following commands, along with its

 parameters.

 Type "help [command]" to get help on a specific ASMCMDcommand.

 commands:

 --------

 cd：------------------------------------------进入下级目录或进入所需要的目录

 du：------------------------------------------显示指定的ASM目录下ASM文件占用的所有磁盘空间

 find：-----------------------------------------查找所需的文件

 help：-----------------------------------------显示帮助信息

 ls：---------------------------------------------列出ASM目录下的内容及其属性

 lsct：-------------------------------------------列出当前ASM客户端的信息

 lsdg：-------------------------------------------列出所有磁盘组及其属性

 mkalias：--------------------------------------为系统生成的文件名创建别名

 mkdir：----------------------------------------创建新目录

 pwd：------------------------------------------显示当前目录路径

 rm：--------------------------------------------删除ASM目录下的某个文件或文件夹

 rmalias：--------------------------------------删除别名

 ASMCMD>

 要查看某个命令的相信通过在命令前添加help来查看，如下:

 ASMCMD> help cd

 cd <dir>

 Change the current directory to <dir>.

 ASMCMD> help du

 du [-H] [dir]

 Display total space used for files located recursively under [dir],

 similar to "du -s" under UNIX; default is the currentdirectory. Two

 values are returned, both in units of megabytes. The first value does

 not take into account mirroring of the diskgroup while the second does.

 For instance, if a file occupies 100 MB of space, then it actually

 takes up 200 MB of space on a normal redundancy diskgroup and 300 MB

 of space on a high redundancy diskgroup. 

 [dir] can also contain wildcards.

 The -H flag suppresses the column headers from the output.

 ASMCMD> help find

 find [-t <type>] <dir> <pattern>

 Find the absolute paths of all occurrences of <pattern> under<dir>.

 <pattern> can be a directory and may include wildcards. <dir> may also

 include wildcards. Note thatdirectory names in the results have the

 "/" suffix to clarify their identity.

 The -t option allows searching by file type. For instance, one can

 search for all the control files at once. <type> must be one of the

 valid values in V$ASM_FILE.TYPE.

 ASMCMD>

 

 **八、 oracleasm工具的使用和语法介绍**

 [root@wwl ~]# oracleasm --help

 Usage: oracleasm[--exec-path=<exec_path>] <command> [ <args> ]

 oracleasm --exec-path

 oracleasm -h

 oracleasm -V

 The basic oracleasm commands are:

 configure Configure the OracleLinux ASMLib driver

 init Load andinitialize the ASMLib driver

 exit Stop the ASMLibdriver

 scandisks Scan the systemfor Oracle ASMLib disks

 status Display thestatus of the Oracle ASMLib driver

 listdisks List known OracleASMLib disks

 querydisk Determine if adisk belongs to Oracle ASMlib

 createdisk Allocate a devicefor Oracle ASMLib use

 deletedisk Return a deviceto the operating system

 renamedisk Change the labelof an Oracle ASMlib disk

 update-driver Download thelatest ASMLib driver

 [root@wwl ~]#

 **九、 ASM相关视图（V$）和数据字典（X$）**

 ASM由于其高度的封装性，使得我们很难知道窥探其内部的原理。可以通过一下视图和数据字典来来查看ASM 的信息。

 **相关视图和数据字典**

 View Name

 X$ Table name

 Description

 V$ASM_DISKGROUP

 X$KFGRP

 performs disk discovery and lists diskgroups

 V$ASM_DISKGROUP_STAT

 X$KFGRP_STAT

 diskgroup stats without disk discovery

 V$ASM_DISK

 X$KFDSK, X$KFKID

 performs disk discovery, lists disks and their usage metrics

 V$ASM_DISK_STAT

 X$KFDSK_STAT, X$KFKID

 lists disks and their usage metrics

 V$ASM_FILE

 X$KFFIL

 lists ASM files, including metadata/asmdisk files

 V$ASM_ALIAS

 X$KFALS

 lists ASM aliases, files and directories

 V$ASM_TEMPLATE

 X$KFTMTA

 lists the available templates and their properties

 V$ASM_CLIENT

 X$KFNCL

 lists DB instances connected to ASM

 V$ASM_OPERATION

 X$KFGMG

 lists rebalancing operations

 N.A.

 X$KFKLIB

 available libraries, includes asmlib path

 N.A.

 X$KFDPARTNER

 lists disk-to-partner relationships

 N.A.

 X$KFFXP

 extent map table for all ASM files

 N.A.

 X$KFDAT

 extent list for all ASM disks

 N.A.

 X$KFBH

 describes the ASM cache (buffer cache of ASM in blocks of 4K (_asm_blksize)

 N.A.

 X$KFCCE

 a linked list of ASM blocks. to be further investigated

 This list isobtained querying v$fixed_view_definitionwhere view_name like '%ASM%' whichexposes all the v$ and gv$ views with theirdefinition. Fixed tables are exposedby querying v$fixed_table where name like'x$kf%' (ASM fixed tables use the'X$KF' prefix).

 SQL>select* fromv$fixed_view_definition whereview_name like '%ASM%';

 SQL>select* from sys.v$fixed_tablewhere name like 'X$KF%' ;

 **十、 ASM常见的错误处理**

 **错误一、**

 ORA-15097：cannotSHUTDOWN ASM instance with connected RDBMS instance

 解决办法：

 发生这个问题，唯一的一个原因就是Oracle实例没有关闭，或ORACLE实例正在关闭或处于挂起状态，导致ASM实例无法关闭，解决办法，关闭RDBMS实例后再关闭ASM实例。

 **错误二、**

 [root@wwl ~]# /etc/init.d/oracleasm createdisk VOL3 /dev/sdd

 Marking disk "VOL3" as an ASM disk: [FAILED]

 报这个错的原因在于磁盘为分区导致。在创建ASM的之前必须线将磁盘分区，但不能格式化，后执行创建就不会有问题了。

 **十一、 ASM 扩展性**

 最多支持63个磁盘组； 最多支持10000个磁盘； 最大支持4pb/磁盘； 最大支持40 exabyte/ASM存储； 最大支持1百W个文件/磁盘组； 外部冗余时单个文件最大35tb，标准冗余时单个文件最大5.8tb，高冗余度时单个文件最大3.9tb

 **十二、 ASM其它信息请参考如下连接：**

 [http://docs.oracle.com/cd/E11882_01/server.112/e16102/asmfiles.htm](http://docs.oracle.com/cd/E11882_01/server.112/e16102/asmfiles.htm)

   
 
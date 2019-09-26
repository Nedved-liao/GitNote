# Oracle 19C 单机部署

本文介绍Oracle数据库19c中的64位的在Oracle的Linux 7（OL7）64位安装。

这篇文章是基于服务器安装最少2G Swap和disable SELinux设置

[TOC]



- ## 下载数据库

  Download the Oracle software from OTN or MOS depending on your support status.

  - [OTN: Oracle Database 19c (19.3) Software (64-bit)](https://www.oracle.com/technetwork/database/enterprise-edition/downloads/oracle19c-linux-5462157.html)

  - [edelivery: Oracle Database 19c (19.3) Software (64-bit)](http://edelivery.oracle.com/)

    

## 1.0 安装准备



### 1.1 硬件检查

#### 1.1.1 硬盘空间检查

> /tmp目录大小至少：1GB
>
> 安装Oracle Database所需空间：8GB

#### 1.1.2 内存检查

> 内存最小: 2GB
>
> Swap最小: 2GB



### 1.2 写入Hosts 解析

在 “/etc/hosts”  写入包含服务器的名称。

格式:

> <IP-address>  <fully-qualified-machine-name>  <machine-name>

```bash
echo '172.10.128.20 Ora19C.com Ora19C' >> /etc/hosts
```



### 1.3 安装依赖

```bash
yum install -y https://yum.oracle.com/repo/OracleLinux/OL7/latest/x86_64/getPackage/oracle-database-preinstall-18c-1.0-1.el7.x86_64.rpm

yum install bc binutils compat-libcap* compat-libstdc* glibc glibc-devel ksh libaio libaio-devel libX11 libXau libXi libXtst libgcc libstdc++ libstdc++-devel libxcb make smartmontools sysstat
```



### 1.4 写入内核参数

```bash
cat > /etc/sysctl.d/98-oracle.conf <<EOF
fs.file-max = 6815744
kernel.sem = 250 32000 100 128
kernel.shmmni = 4096
kernel.shmall = 1073741824
kernel.shmmax = 4398046511104
kernel.panic_on_oops = 1
net.core.rmem_default = 262144
net.core.rmem_max = 4194304
net.core.wmem_default = 262144
net.core.wmem_max = 1048576
net.ipv4.conf.all.rp_filter = 2
net.ipv4.conf.default.rp_filter = 2
fs.aio-max-nr = 1048576
net.ipv4.ip_local_port_range = 9000 65500
EOF

#加载内核参数
sysctl -p /etc/sysctl.d/98-oracle.conf
```



### 1.5 写入用户limit

```bash
cat > /etc/security/limits.d/oracle-database-preinstall-18c.conf <<EOF
oracle   soft   nofile    1024
oracle   hard   nofile    65536
oracle   soft   nproc    16384
oracle   hard   nproc    16384
oracle   soft   stack    10240
oracle   hard   stack    32768
oracle   hard   memlock    134217728
oracle   soft   memlock    134217728
EOF
```



### 1.6 关闭SELinux 和 防火墙

```bash
sed -i  "s/SELINUX=enforcing/SELINUX=disabled/"  /etc/selinux/config
systemctl  stop firewalld
systemctl disable firewalld
```



### 1.7 创建用户和组

常见用户组说明

| 组       | 角色    | 权限                                                         |
| -------- | ------- | ------------------------------------------------------------ |
| oinstall |         | 安装和升级oracle软件                                         |
| dba      | sysdba  | 创建、删除、修改、启动、关闭数据库，切换日志归档模式，备份恢复数据库 |
| oper     | sysoper | 启动、关闭、修改、备份、恢复数据库，修改归档模式             |

```bash
groupadd -g 54321 oinstall
groupadd -g 54322 dba
groupadd -g 54323 oper
useradd -u 54321 -g oinstall -G dba,oper oracle

#修改密码
echo 'oracle' | passwd --stdin oracle
```



### 1.8 创建安装目录

```bash
mkdir -p /data/oracle/product/19.0.0/dbhome_1
mkdir -p /data/oradata
chown -R oracle:oinstall /data
chmod -R 775 /data
```



### 1.9 配置环境变量

```bash 
cat >> /home/oracle/.bash_profile <<EOF

#Oracle Settings
export TMP=/tmp
export TMPDIR=\$TMP

export ORACLE_HOSTNAME=Ora19C
export ORACLE_BASE=/data/oracle
export ORACLE_HOME=\$ORACLE_BASE/product/19.0.0/dbhome_1

export ORACLE_UNQNAME=orcl
export ORACLE_SID=orcl

#Oracle data dir 
export DATA_DIR=/data/oradata
export ORA_INVENTORY=/data/oraInventory

export PATH=/usr/sbin:/usr/local/bin:\$PATH
export PATH=\$ORACLE_HOME/bin:\$PATH

export LD_LIBRARY_PATH=\$ORACLE_HOME/lib:/lib:/usr/lib
export CLASSPATH=\$ORACLE_HOME/jlib:\$ORACLE_HOME/rdbms/jlib
EOF
```



## 2.0 静默安装数据库

### 2.1 预配应答命令

不对db_install.rsp 文件修改,直接在命令行进行配置

```bash
cd /data/oracle/product/19.0.0/dbhome_1

# 静默安装
./runInstaller -ignorePrereq -waitforcompletion -silent                        \
    -responseFile ${ORACLE_HOME}/install/response/db_install.rsp               \
    oracle.install.option=INSTALL_DB_SWONLY                                    \
    ORACLE_HOSTNAME=${ORACLE_HOSTNAME}                                         \
    UNIX_GROUP_NAME=oinstall                                                   \
    INVENTORY_LOCATION=${ORA_INVENTORY}                                        \
    SELECTED_LANGUAGES=en,en_GB                                                \
    ORACLE_HOME=${ORACLE_HOME}                                                 \
    ORACLE_BASE=${ORACLE_BASE}                                                 \
    oracle.install.db.InstallEdition=EE                                        \
    oracle.install.db.OSDBA_GROUP=dba                                          \
    oracle.install.db.OSBACKUPDBA_GROUP=dba                                    \
    oracle.install.db.OSDGDBA_GROUP=dba                                        \
    oracle.install.db.OSKMDBA_GROUP=dba                                        \
    oracle.install.db.OSRACDBA_GROUP=dba                                       \
    SECURITY_UPDATES_VIA_MYORACLESUPPORT=false                                 \
    DECLINE_SECURITY_UPDATES=true

#安装完后根据提示
 /data/oraInventory/orainstRoot.sh	
 /data/oracle/product/19.0.0/dbhome_1/root.sh
```



### 2.2 静默建库

```bash
#普通数据库
dbca -silent -createDatabase                                                   \
     -templateName General_Purpose.dbc                                         \
     -gdbname ${ORACLE_SID} -sid  ${ORACLE_SID} -responseFile NO_VALUE         \
     -characterSet AL32UTF8                                                    \
     -sysPassword Oracle19c                                                    \
     -systemPassword Oracle19c                                                 \
     -createAsContainerDatabase false                                          \
	 -databaseType MULTIPURPOSE                                                \
     -automaticMemoryManagement false                                          \
     -totalMemory 2048                                                         \
     -storageType FS                                                           \
     -datafileDestination "${DATA_DIR}"                                        \
     -redoLogFileSize 50                                                       \
     -emConfiguration NONE                                                     \
     -ignorePreReqs
     

#可插拔数据库
dbca -silent -createDatabase 													\
 -templateName General_Purpose.dbc 												\
 -gdbname emrep -responseFile NO_VALUE 											\
 -characterSet AL32UTF8 														\
 -sysPassword Oracle19c 														\
 -systemPassword Oracle19c 														\
 -createAsContainerDatabase true 												\
 -numberOfPDBs 1 																\
 -pdbName orapdb 																\
 -pdbAdminPassword Oracle19c 													\
 -databaseType MULTIPURPOSE 													\
 -automaticMemoryManagement false 												\
 -totalMemory 1024 																\
 -redoLogFileSize 50 															\
 -emConfiguration NONE 															\
 -ignorePreReqs
```

删库命令:

>dbca -silent -deleteDatabase -sourcedb orcl -sid orcl -sysDBAUserName orcl -sysDBAPassword Oracle19c



### 2.3 在/etc/oratab 标记实例为 Y

```bash
sed -i 's/dbhome_1:N/dbhome_1:Y/' /etc/oratab
```



### 2.4 启用Oracle管理文件（OMF），并确保该实例启动时PDB开始。	 

```bash
sqlplus / as sysdba <<EOF
alter system set db_create_file_dest='${DATA_DIR}';
alter pluggable database ${PDB_NAME} save state;
exit;
EOF
```



参考文档:

[Oracle Database 19c Installation On Oracle Linux 7 (OL7)](https://oracle-base.com/articles/19c/oracle-db-19c-installation-on-oracle-linux-7#automatic_setup)


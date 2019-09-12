  # []()前言

 最近在部署一套环境，里面的数据库用的是Oracle。就顺带整理了一下笔记  
 最无语的是到2019了，还是有人分不清什么是服务端什么是客户端。  
 先让我装个客户端，特喵的又说你这个不是客户端怎么缺那么多头文件。。。  
 真被气死

 
# []()正文

 适用于Centos6+  
 oracle11.*

 
# []()环境检查

 检查 swap分区、内存、磁盘大小、系统版本

 
```
df
free
uname -a

```
 
# []()关闭selinux和防火墙

 
```
#临时关闭
setenforce 0
#永久关闭
sed -i '/^SELINUX=/s/SELINUX=.*/SELINUX=disabled/' /etc/selinux/config 
#关闭防火墙
service iptables stop
systemctl stop firewalld
systemctl disable firewalld

```
 
# []()修改主机名和hosts

 
```
#设置主机名
echo "oracle-prod" > /etc/hostname
#增加对应的hosts记录
echo "127.0.0.1 oracle-prod" >> /etc/hosts

```
 **Note**

 
> **"** 可以使用hostname来查看是否已经设置了主机名  
>  ORA-00130: invalid listener address  
>  添加与主机名与IP对应记录，不然在安装数据库时会报错
> 
>  
 
# []()修改内核参数

 
```
cat >> /etc/sysctl.conf <<EOF
fs.aio-max-nr = 1048576
fs.file-max = 6815744
kernel.shmall = 2097152
kernel.shmmax = 32903162
kernel.shmmni = 4096
kernel.sem = 250 32000 100 128
net.ipv4.ip_local_port_range = 9000 65500
net.core.rmem_default = 262144
net.core.rmem_max = 4194304
net.core.wmem_default = 262144
net.core.wmem_max = 1048576
EOF

sysctl -p 

```
 **Note**

 
> **"** kernel.shmmax = 32903162（byte）为本机物理内存的一半,可以使用free查看。  
>  或者直接使用如下命令:
> 
>  
> > echo $(( `free | awk '/Mem/ {print $2}'` /2))
> > 
> >  
>  
 
# []()修改pam验证登录

 
```
echo "session required pam_limits.so" >> /etc/pam.d/login

```
 
# []()修改用户配置文件

 
```
cat >> /etc/profile <<EOF
export LIBXCB_ALLOW_SLOPPY_LOCK=true
if [ $USER = "oracle" ]; then
if [ $SHELL = "/bin/ksh" ]; then
ulimit -p 16384
ulimit -n 65536
else
ulimit -u 16384 -n 65536
fi
umask 022
fi
EOF

source /etc/profile

```
 
# []()修改系统资源限制

 
```
cat >> /etc/security/limits.conf <<EOF
oracle soft nproc 2047
oracle hard nproc 16384
oracle soft nofile 1024
oracle hard nofile 65536    
oracle soft stack 10240
EOF

```
 
# []()准备用户与组

 
```
groupadd oinstall
groupadd dba
useradd -g oinstall -G dba oracle
echo "rs_ora2019" | passwd --stdin oracle

```
 
# []()准备安装目录

 
```
mkdir -p /u01/app/oracle/product/11.2.0.2.0/db_1
mkdir/u01/app/oraInventory
chown -R oracle:oinstall /u01
chmod -R 755 /u01

```
 
# []()配置oracle用户环境变量

 
```
cat >> /home/oracle/.bash_profile   <<EOF
#Oracle Settings umask 022;
export ORACLE_SID=orcl-prod;
export ORACLE_BASE=/u01/app/oracle;
export ORACLE_HOME=\${ORACLE_BASE}/product/11.2.0.2.0/db_1;
export PATH=\$ORACLE_HOME/bin:/usr/sbin:\$PATH;
export LD_LIBRARY_PATH=\$ORACLE_HOME/lib:/usr/lib;
export TNS_ADMIN=\$ORACLE_HOME/network/admin;
export NLS_LANG='SIMPLIFIED CHINESE_CHINA.ZHS16GBK'
EOF
source /home/oracle/.bash_profile

```
 
# []()安装依赖

 
```
yum install gcc gcc-c++ libaio-devel compat-gcc-34 compat-gcc-34-c++

```
 
# []()下载安装包

 [官网下载](https://www.oracle.com/technetwork/database/enterprise-edition/downloads/index.html)  


 需要下载如下两个文件  
 linux.x64_11gR2_database_1of2.zip  
 linux.x64_11gR2_database_2of2.zip

 若没有在IBM注册有账号可以在此下载  
  
 [Oracle11.2.0.2 from Nedved OD](http://sg.fcbnedved.cn/Linux/Oracle/Oracle11.2.0.2%20for%20Linux-x86-64%2864bit%29.zip)

 
# []()上传解压安装包

 把从官网下载得到的两个zip上传至/home/oracle/

 
```
chown -R oracle:oracle /home/oracle/l*.zip
su - oracle
unzip ./linux.x64_11gR2_database_1of2.zip && unzip ./linux.x64_11gR2_database_2of2.zip
mkdir ora_install 
mv database ora_install

```
 **Note**

 
> **"** 必须先解压1of2 再解压2of2  
>   
>  response下的=应答文件：
> 
>  
> > db_install.rsp：安装应答  
> >   
> >  dbca.rsp：创建数据库应答  
> >   
> >  netca.rsp：建立监听、本地服务名等网络设置的应答  
> > 
> > 
> >  
>  
 
# []()配置安装应答文件

 
```
cd /home/oracle/ora_install/database/response &&
cp db_install.rsp db_install.rsp.default
cat > db_install.rsp <<EOF
oracle.install.responseFileVersion=/oracle/install/rspfmt_dbinstall_response_schema_v11_2_0
oracle.install.option=INSTALL_DB_SWONLY
ORACLE_HOSTNAME=oracle-prod
UNIX_GROUP_NAME=oinstall
INVENTORY_LOCATION=/u01/app/oraInventory
SELECTED_LANGUAGES=en,zh_CN
ORACLE_HOME=/u01/app/oracle/product/11.2.0.2.0/db_1
ORACLE_BASE=/u01/app/oracle
oracle.install.db.InstallEdition=EE
oracle.install.db.isCustomInstall=false
oracle.install.db.optionalComponents=oracle.rdbms.partitioning:11.2.0.2.0,oracle.oraolap:11.2.0.2.0,oracle.rdbms.dm:11.2.0.2.0,oracle.rdbms.dv:11.2.0.2.0,orcle.rdbms.lbac:11.2.0.2.0,oracle.rdbms.rat:11.2.0.2.0
oracle.install.db.DBA_GROUP=dba
oracle.install.db.OPER_GROUP=dba
oracle.install.db.config.starterdb.characterSet=AL32UTF8
oracle.install.db.config.starterdb.memoryOption=true
oracle.install.db.config.starterdb.installExampleSchemas=false
oracle.install.db.config.starterdb.enableSecuritySettings=true
oracle.install.db.config.starterdb.control=DB_CONTROL
oracle.install.db.config.starterdb.dbcontrol.enableEmailNotification=false
oracle.install.db.config.starterdb.automatedBackup.enable=false
DECLINE_SECURITY_UPDATES=true
EOF

```
 **部分参数含义**

 
     参数                                 | 说明                                               
     ---------------------------------- | ------------------------------------------------- 
     -silent                            | 表示以静默方式安装,不会有任何提示                                
     -force                             | 允许安装到一个非空目录                                      
     -noconfig                          | 表示不运行配置助手netca                                   
     -responseFile                      | 表示使用哪个响应文件,必需使用绝对路径                              
     oracle.install.responseFileVersion | 响应文件模板的版本,该参数不要更改                                
     oracle.install.option              | 安装选项,本例只安装oracle软件,该参数不要更改                       
     DECLINE_SECURITY_UPDATES           | 是否需要在线安全更新,设置为false,该参数不要更改                      
     ORACLE_HOSTNAME                    | 安装主机名                                            
     UNIX_GROUP_NAME                    | oracle用户用于安装软件的组名                                
     INVENTORY_LOCATION                 | oracle产品清单目录                                     
     SELECTED_LANGUAGES                 | oracle运行语言环境,一般包括引文和简繁体中文                        
     ORACLE_HOME                        | Oracle安装目录                                       
     ORACLE_BASE                        | oracle基础目录                                       
     oracle.install.db.InstallEdition   | 安装版本类型,一般是企业版                                    
     oracle.install.db.isCustomInstall  | 是否定制安装,默认Partitioning,OLAP,RAT都选上了               
     oracle.install.db.customComponents | 定制安装组件列表:除了以上默认的,可加上Label Security和Database Vault
     oracle.install.db.DBA_GROUP        | oracle用户用于授予OSDBA权限的组名                           
     oracle.install.db.OPER_GROUP       | oracle用户用于授予OSOPER权限的组名                          


# []()进行安装

 
```
cd /home/oracle/ora_install/database/
./runInstaller -silent -responseFile /home/oracle/ora_install/database/response/db_install.rsp -ignorePrereq

···

#Root scripts to run

/u01/app/oracle/oraInventory/orainstRoot.sh
/u01/app/oracle/product/11.2.0.2.0/db_1/root.sh
To execute the configuration scripts:
        1. Open a terminal window
        2. Log in as "root"
        3. Run the scripts
        4. Return to this window and hit "Enter" key to continue

Successfully Setup Software.


```
 **Note**

 
> **"** 安装过程可以再开一个窗口按照提示查看日志运行。当安装成功后会在执行安装命令的那个窗口跳出#Root scripts to run ··· 的内容
> 
>  
 切换root执行

 
```
su - root
/u01/app/oracle/product/11.2.0.2.0/db_1/root.sh
/u01/app/oracle/oraInventory/orainstRoot.sh

```
 
# []()静默配置监听程序

 **使用 oracle 用户**

 
```
netca /silent /responsefile /home/oracle/ora_install/database/response/netca.rsp

```
 查看输出信息，若没有报错会在$TNS_ADMIN 下生成 listener.ora 和 sqlnet.ora

 
> ll $TNS_ADMIN
> 
>  
 通过netstat命令可以查看1521端口正在监听

 
> ss -antpul | grep 1521
> 
>  
 
# []()静默方式建库

 
```
dbca -silent -createDatabase \
-templateName General_Purpose.dbc \
-gdbname orcl-prod \
-sid orcl-prod \
-responseFile NO_VALUE \
-characterSet ZHS16GBK \
-memoryPercentage 30 -emConfiguration LOCAL

.
.
.

85% complete
96% complete
100% complete
Look at the log file "/opt/app/oracle/cfgtoollogs/dbca/orcl-prod/orcl-prod.log" for further details.

```
 **Note**

 
     参数                | 说明                                                
     ----------------- | -------------------------------------------------- 
     -silent           | 指以静默方式执行dbca命令                                    
     -createDatabase   | 指使用dbca                                           
     -templateName     | 指定用来创建数据库的模板名称，这里指定为General_Purposedbc，即一般用途的数据库模板
     -gdbname          | 指定创建的全局数据库名称，这里指定名称为 orcl-prod                    
     -sid              | 指定数据库系统标识符，这里指定为 orcl-prod，与数据库同名                 
     -responseFile     | 指定安装响应文件，NO_VALUE表示没有指定响应文件                       
     -characterSet     | 指定数据库使用的字符集，这里指定为AL32UTF8                         
     -memoryPercentage | 指定用于oracle的物理内存的百分比，这里指定为30%                      
     -emConfiguration  | 指定Enterprise Management的管理选项。                     
     LOCAL             | 表示数据库由Enterprise Manager本地管理                      

数据库成功安装之后默认是启动状态

 
> 检查  
> 
> 
>  
> > 1、进行实例进程检查  
> >   
> >  ps -ef | grep ora_ | grep -v grep
> > 
> >  
>  
 
> > > 2、查看监听状态  
> >   
> >  lsnrctl status
> > 
> >  
>  
 
> > > 3、登录查看实例状态  
> >   
> >  sqlplus / as sysdba
> > 
> >  
>  
 
# []()设置Linux开机自启动

 修改ORACLE_HOME_LISTNER  
  
 将$ORACLE_HOME/bin下的dbstart和dbshut  
 里面的ORACLE_HOME_LISTNER=$1 修改为ORACLE_HOME_LISTNER=$ORACLE_HOME

 
```
sed -i '/ORACLE_HOME_LISTNER/s/\$1/\$ORACLE_HOME/' $ORACLE_HOME/bin/dbstart
sed -i '/ORACLE_HOME_LISTNER/s/\$1/\$ORACLE_HOME/' $ORACLE_HOME/bin/dbshut

```
 
# []()配置oratab

 将N改为Y

 
```
sed -i '/orcl-prod/s/N/Y/' /etc/oratab

```
 
# []()配置rc.local

 
```
cat >> /etc/rc.d/rc.local <<EOF
su oracle -lc "/u01/app/oracle/product/11.2.0.2.0/db_1/bin/lsnrctl start"
su oracle -lc /u01/app/oracle/product/11.2.0.2.0/db_1/dbstart
EOF

chmod +x /etc/rc.d/rc.local

```
 以上就oracle11g的静默安装了。  
 2019.3.29 V1

   
  
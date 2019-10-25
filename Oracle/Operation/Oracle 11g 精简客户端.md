  # []()前言

 通常开发人员会装上一个 oracle客户端，但一般不会在自己的机器上安装Oracle database  
 Oracle 客户端安装体积很大，但是装上去了基本上就用2个功能：TNS配置服务名和sqlplus。根本没必要装那么大的空间，这很不值得。

 
# []()环境

 Centos6+  
 x86-64

 
## []()下载

 [Oracle官方下载](https://www.oracle.com/technetwork/cn/database/features/instant-client/index-092699-zhs.html)

 如果没有注册IBM账号可以在此下载  
 [Client 11.2.0.4 for rpm](http://sg.fcbnedved.cn/Linux/Oracle/Oracle_Client%20for%20rpm%2811.2.0.4%29.zip)  
 [Client 11.2.0.4 for zip](http://sg.fcbnedved.cn/Linux/Oracle/Oracle_Client%20for%20zip%2811.2.0.4%29.zip)

 下载得到(以rpm示列)

 
> oracle-instantclient11.2-basic-11.2.0.4.0-1.x86_64.rpm  
>  oracle-instantclient11.2-sqlplus-11.2.0.4.0-1.x86_64.rpm
> 
>  
 
# []()配置用户和安装oracle客户端

 
```
useradd oracle
yum -y install oracle-instantclient11.2-* 

```
 **Note**

 
> " 安装后的路径是/usr/lib/oracle,可以使用rpm -ql去查看详细信息
> 
>  
 
# []()为了方便管理，建立软连接

 
```
chown -R oracle:oracle /usr/lib/oracle
ln -s /usr/lib/oracle /home/oracle

```
 
# []()为Oracle用户设置环境变量

 
```
cat >> /home/oracle/.bash_profile <<EE
#sqlplus 变量
export ORACLE_VERSION=11.2.0.4.0
export ORACLE_HOME=/home/oracle/\${ORACLE_VERSION}/client64
export TNS_ADMIN=\$ORACLE_HOME/network/admin
export LD_LIBRARY_PATH=\$ORACLE_HOME/lib
export PATH=\$PATH:\$ORACLE_HOME/bin
EE
source  /home/oracle/.bash_profile 

```
 
# []()创建监听文件

 
```
su - oracle
mkdir -p ${ORACLE_HOME}/network/admin

```
 
# []()写入远程数据库tns信息

 
```
cat > ${TNS_ADMIN}/tnsnames.ora <<EE
##测试库   
ORCL11G =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = 127.0.0.1)(PORT = 1521))
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = orcl11g)
    )
  )
 EE

```
 
# []()测试sqlplus

 
```
sqlplus 用户/密码@//IP/SID
sqlplus 用户/密码@SID

```
 **Note**

 
> 1.每个版本的安装过程大致一样哦  
>  2.如需要使用sqlldr，可以从服务端上copy相对应的文件到对应的路径就可以啦  
>  3.需要使用simple Chinese也要从服务端copy语言文件 nls
> 
>  
 **最后跟新于2019/3/27 V2**

   
  
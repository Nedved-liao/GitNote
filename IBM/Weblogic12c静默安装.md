**Note：**

 
> weblogic在安装的时候会对系统进行严格的检查，  
>  包括jdk版本,cpu性能，swap空间，磁盘空间,tmp临时空间软件在安装时会产生大约1G的日志以及其他必须的文件等
> 
>  
 
# []()环境准备

 
## []()写入host解析

 
```
#查看当前主机名
root@nedved#hostname
nedved
#写入host
echo "127.0.0.1 nedved" >>/etc/hosts

```
 
## []()使用本地文件开启swap

 
```
sudo dd if=/dev/null of=/etc/swap bs=1M count=4096
sudo chmod 600 /etc/swap
sudo mkswap /etc/swap
sudo swapon /etc/swap
sudo echo "/etc/swap none swap default 0 0" >> /etc/fstab

```
 
## []()准备用户与组和安装目录

 
```
sudo groupadd web-data
sudo useradd -c "Weblogic User" -g web-data weblogic 
sudo passwd weblogic
sudo mkdir -p /bea/install
sudo chown -R weblogic:web-data /bea

```
 
## []()下载对应的weblogic安装包和jdk

 [weblogic12C](https://www.oracle.com/technetwork/middleware/weblogic/downloads/index.html)  
 [jdk1.8](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)

 **Note：**

 
> openJDK和oracle jdk是有所区别的，所以安装weblogic时用的的是oracle jdk。
> 
>  
 
# []()静默安装

 
## []()安装jdk1.8

 
```
sudo rpm -ivh jdk-8u201-linux-x64.rpm
sudo tee >> /etc/profile <<EOF
export JAVA_HOME=/usr/java/jdk1.8.0_201-amd64
export JRE_HOME=/usr/java/jdk1.8.0_201-amd64/jre
export CLASSPATH=$CLASSPATH:$JAVA_HOME/lib:$JAVA_HOME/jre/lib
export PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$PATH
EOF

source /etc/profile
java -version

```
 
## []()关闭随机数获取

 静默建域，启动服务时候往往会挂起或花费超过一分钟才获得响应，  
 因为linux上的SecureRandom generateSeed（）。SecureRandom 使用/dev/random生成随机数。但是/dev/random是一个阻塞数字生成器，如果它没有足够的随机数据提供，它就一直等，这迫使JVM等待。所以选择关闭  
 **方法1：**

 
```
sed -i 's#=file:/dev/urandom/#=file:/dev/./urandom#' $JAVA_HOME/jre/lib/security/java.security

```
 **方法2：**  
 [在weblogic启动脚本里setDomainEnv.sh](http://xn--weblogicsetDomainEnv-pt47a0zu3nszv4goq1g9yvd.sh): 加入以下内容

 
```
   JAVA_OPTIONS="${JAVA_OPTIONS}" -Djava.security.egd="file:/dev/./urandom"
   export JAVA_OPTIONS

```
 
## []()安装weblogic12C

 
### []()创建环境应答文件 oraInst.loc

 
```
tee > /bea/install/oraInst.loc <<EOF
inventory_loc=/bea/oraInventory
inst_group=weblogic
EOF

```
 
### []()创建响应文件 wls.rsp

 
```
tee > /bea/install/wls.rsp <<EOF
[ENGINE]
Response File Version=1.0.0.0.0
[GENERIC]
ORACLE_HOME=/bea/weblogic12C
INSTALL_TYPE=WebLogic Server
MYORACLESUPPORT_USERNAME=weblogic
MYORACLESUPPORT_PASSWORD=123
DECLINE_SECURITY_UPDATES=true
SECURITY_UPDATES_VIA_MYORACLESUPPORT=false
PROXY_HOST=
PROXY_PORT=
PROXY_USER=
PROXY_PWD=
COLLECTOR_SUPPORTHUB_URL=
EOF
chmod 660 wls.rsp

```
 
### []()进行安装

 
```
$JAVA_HOME/bin/java -jar fmw_12.1.3.0.0_wls.jar -silent -responseFile /bea/install/wls.rsp -invPtrLoc /bea/install/oraInst.loc

安装过程输出如下：
/home/weblogic/oraInst.loc 
Launcher log file is /tmp/OraInstall2019-04-08_02-08-58AM/launcher2019-04-08_02-08-58AM.log.
Extracting the installer . . . . Done
Checking if CPU speed is above 300 MHz.   Actual 2806.094 MHz    Passed
Checking swap space: must be greater than 512 MB.   Actual 1024 MB    Passed
Checking if this platform requires a 64-bit JVM.   Actual 64    Passed (64-bit not required)
Checking temp space: must be greater than 300 MB.   Actual 34095 MB    Passed
Preparing to launch the Oracle Universal Installer from /tmp/OraInstall2019-04-08_02-08-58AM
Log: /tmp/OraInstall2019-04-08_02-08-58AM/install2019-04-08_02-08-58AM.log
·
·
·
Percent Complete : 10
Percent Complete : 20
Percent Complete : 30
Percent Complete : 40
Percent Complete : 50
Percent Complete : 60
Percent Complete : 70
Percent Complete : 80
Percent Complete : 90
Percent Complete : 100

The installation of Oracle Fusion Middleware 12c WebLogic and Coherence Developer 12.2.1.3.0 completed successfully.
Logs successfully copied to /bea/weblogic/weblogic_install_dir/wls12213/cfgtoollogs/oui.

```
 
## []()设置环境变量

 
```
exec /bea/weblogic/wlserver/server/bin/setWLSEnv.sh

```
 
# []()静默建域

 
## []()生成通过py应答脚本

 
```
tee > /bea/weblogic/wlserver/server/bin/create_domains.py <<EOF
readTemplate('/bea/weblogic/wlserver/common/templates/wls/wls.jar')
cd('Servers/AdminServer')
set('ListenAddress','10.201.15.181')
set('ListenPort', 7010)
cd('../..')
cd('/Security/base_domain/User/weblogic')
cmo.setPassword('weblogic123')
setOption('OverwriteDomain', 'true')
setOption('ServerStartMode', 'prod')
writeDomain('/bea/weblogic/user_projects/domains/Ppord_domain')
closeTemplate()
exit()
EOF
#进行安装
cd  /bea/weblogic/wlserver/server/bin/ &&
./wlst.sh  ./create_domains.py

```
 **Note：**

 
> 需修改 ListenAddress、ListenPort、setPassword、writeDomain和weblogic安装路径
> 
>  
 
## []()weblogic免密启动，增加boot.properties文件

 启动WebLogic时需要输入该Domain的用户名和密码  
 在生产环境中，一般会要求不要在每次启动时都输入用户名密码，  
 解决：  
 增加boot.properties文件，保存用户名和密码，  
 启动AdminServer后boot.properties文件中的信息会自动加密。

 
```
mkdir /bea/weblogic/user_projects/domains/Ppord_domain/servers/AdminServer/security
tee > /bea/weblogic/user_projects/domains/Ppord_domain/servers/AdminServer/security/boot.properties <<EOF
username=weblogic
password=weblogic123
EOF

```
 
# []()启动weblogic

 
```
cd /bea/weblogic/user_projects/domains/Ppord_domain/bin/
nohup ./startWebLogic.sh > ./Ppord_domain.log &

```
 
## []()验证

 [http://10.201.15.181:7010/console](http://10.201.15.181:7010/console)

 4/8/2019 v1.2

   
  
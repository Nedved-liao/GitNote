 
# 一 安装补丁步骤

 1、登录linux的weblogic用户，切换到/home/weblogic/Oracle/Middleware/utils/bsu/目录下。

 2、确认当前weblogic版本,并确认所有域的进程全部关闭

 查看版本  
 ./bsu.sh -prod_dir={MW_HOME}/{WL_HOME} -status=applied -verbose -view

 输出如下:

 -bash-4.1$ ./bsu.sh -prod_dir=/app/weblogic/wlserver_10.3/ -status=applied -verbose -view  
 ProductName: WebLogic Server  
 ProductVersion: 10.3 MP6  
 Components: WebLogic Server/Core Application Server,WebLogic Server/Admi  
 nistration Console,WebLogic Server/Configuration Wizard and  
 Upgrade Framework,WebLogic Server/Web 2.0 HTTP Pub-Sub Serve  
 r,WebLogic Server/WebLogic SCA,WebLogic Server/WebLogic JDBC  
 Drivers,WebLogic Server/Third Party JDBC Drivers,WebLogic S  
 erver/WebLogic Server Clients,WebLogic Server/WebLogic Web S  
 erver Plugins,WebLogic Server/UDDI and Xquery Support,WebLog  
 ic Server/Evaluation Database,WebLogic Server/Workshop Code  
 Completion Support  
 BEAHome: /app/weblogic  
 ProductHome: /app/weblogic/wlserver_10.3  
 PatchSystemDir: /app/weblogic/utils/bsu  
 PatchDir: /app/weblogic/patch_wls1036  
 Profile: Default  
 DownloadDir: /app/weblogic/utils/bsu/cache_dir  
 JavaVersion: 1.6.0_29  
 JavaVendor: Sun

 Patch ID: B47X  
 PatchContainer: B47X.jar  
 Checksum: -345780037  
 Severity: optional  
 Category: General  
CR/BUG: 27919965  
 Restart: true  
 Description: WLS PATCH SET UPDATE 10.3.6.0.180717  
 WLS PATCH SET UPDATE 10  
 .3.6.0.180717

 

 可以看到 Patch ID 和 补丁版本号

 

 3、删除旧补丁  
 bsu.sh -remove -patchlist={PATCH_ID} -prod_dir={MW_HOME}/{WL_HOME}

 这里的 PATCH_ID 为上一次打补丁的ID

 4、查看是否存在/home/weblogic/Oracle/Middleware/utils/bsu/cache_dir 目录，没有的需要手工创建。

 5、将补丁包上传到/home/weblogic/Oracle/Middleware/utils/bsu/cache_dir目录下

 6、解压p27395085_1036_Generic.zip   
 unzip p27395085_1036_Generic.zip 

 得到如下文件

 GFWX.jar patch-catalog.xml README.txt

 详细打补丁步骤都在 README.txt

 新的PATCH_ID 可以在README.txt 其中找出,(GFWX.jar 前面的GFWX其实就是新补丁ID)

 7、执行补丁安装命令。  
bsu.sh -install -patch_download_dir={MW_HOME}/utils/bsu/cache_dir -patchlist={PATCH_ID} -prod_dir={MW_HOME}/{WL_HOME}

 ./bsu.sh -install -patch_download_dir=/home/weblogic/Oracle/Middleware/utils/bsu/cache_dir -patchlist=新补丁ID -prod_dir=/home/weblogic/Oracle/Middleware/wlserver_10.3 –verbose

 

 8、在打补丁包时，如果遇到了内存溢出的问题。需要修改bsu.sh脚本，将内存调大。

 9、启动weblogic的域，查看输出日志。确定版本是否生效。

 

 
# 二 添加weblogic 防火墙规则(仅是在对优化2018.4.18补丁更新中利用)

 
## 背景

 Oracle官方发布了7月份的关键补丁更新CPU（Critical Patch Update），修复了WebLogic Server多个远程代码执行漏洞。但是，由于官方补丁存在问题，导致其中一个远程代码执行漏洞未被完全修复。  
 中国联通信息安全部将持续关注该漏洞进展，并更新该漏洞信息。

 关键字: Weblogic远程代码执行漏洞

 Oracle Fusion Middleware（Oracle融合中间件）是美国甲骨文（Oracle）公司的一套面向企业和云环境的业务创新平台。该平台提供了中间件、软件集合等功能。Oracle WebLogic Server是其中的一个适用于云环境和传统环境的应用服务器组件。  
 Oracle官方在2018年4月18日发布了4月关键补丁更新，其中包含了Oracle WebLogic Server的一个高危漏洞（CVE-2018-2628）。  
 由于此漏洞产生于Weblogic T3服务，当开放Weblogic控制台端口（默认为7001端口）时，T3服务会默认开启，因此会造成较大影响，结合曾经爆出的Weblogic WLS 组件漏洞（ CVE-2017-10271 ），不排除会有攻击者利用挖矿的可能，因此，建议尽快部署防护措施。

 CVE-2018-2628  
 攻击者可以在未授权的情况下，通过T3协议在WebLogic Server中执行反序列化操作，最终造成远程代码执行漏洞。  
 Oracle官方将该漏洞等级定义为“高危”

 漏洞属性  
 【漏洞评级】高危  
 【CVE编号】  
 CVE-2018-2628  
 【影响范围】  
 Weblogic 10.3.6.0  
 Weblogic 12.1.3.0  
 Weblogic 12.2.1.2  
 Weblogic 12.2.1.3

 登录weblogic控制台,

 ![](https://img-blog.csdn.net/20180720014855779?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L05lZHZlZF9M/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

 第4步文本框中填写的内容为：  
 weblogic.security.net.ConnectionFilterImpl  
 第5步文本框中填写的内容为：  
 127.0.0.1 * * allow t3 t3s  
 0.0.0.0/0 * * deny t3 t3s  
 （每个字段中间有空格）  
 修改完后请重启 Weblogic 使配置生效。

 关于t3/t3s 和weblogic 防火墙规则,

 详细参考[http://www.enmotech.com/web/detail/1/482/2.html](http://www.enmotech.com/web/detail/1/482/2.html)

 ![](https://img-blog.csdn.net/20180720014953762?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L05lZHZlZF9M/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

 升级补丁详见 README.txt

 BSU命令详细参考[ http://docs.oracle.com/cd/E14759_01/doc.32/e14143/commands.htm](http://docs.oracle.com/cd/E14759_01/doc.32/e14143/commands.htm)

 t3/t3s详细参考 [http://www.enmotech.com/web/detail/1/482/2.html](http://www.enmotech.com/web/detail/1/482/2.html)

 

 

 

   
 
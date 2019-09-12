# 端口扫描 

 
> >  （1） -sS ：半开放扫描  
>  （2） sT：3次握手方式tcp的扫描  
>  （3）sU：udp端口的扫描  
>  （4）sF：也是tcp的扫描一种，发送一个FIN标志的数据包  
>  （5）sW：窗口扫描  
>  （6） sV：版本检测(sV)
> 
>  
 
# 探测主机存活常用方式

 
## （1）-sP ：进行ping扫描 

 打印出对ping扫描做出响应的主机,不做进一步测试(如端口扫描或者操作系统探测)：  
 下面去扫描192.168.4/24这个网段的的主机

 
> nmap -sP 192.168.4/24
> 
>  
 
## （2） -sn：

 -sn: Ping Scan - disable port scan #ping探测扫描主机， 不进行端口扫描 （测试过对方主机把icmp包都丢弃掉，依然能检测到对方开机状态）

 
> nmap -sn 192.168.4.1-166
> 
>  
 
## （3）-sA

 nmap 192.168.4.1 -sA （发送tcp的ack包进行探测，可以探测主机是否存活）

 
# 端口扫描的高级用法

 
## （1） -sS ：半开放扫描（非3次握手的tcp扫描）

 使用频率最高的扫描选项：SYN扫描,又称为半开放扫描，它不打开一个完全的TCP连接，执行得很快，效率高（一个完整的tcp连接需要3次握手，而-sS选项不需要3次握手）

 Tcp SYN Scan (sS) 它被称为半开放扫描  
 优点：Nmap发送SYN包到远程主机，但是它不会产生任何会话，目标主机几乎不会把连接记入系统日志。（防止对方判断为扫描攻击），扫描速度快，效率高，在工作中使用频率最高  
 缺点：它需要root/administrator权限执行

 
> [root@78778e06dc0a /]# nmap -sS 192.168.4.1   
>    
>  Starting Nmap 5.51 ( http://nmap.org ) at 2016-12-29 11:38 CST  
>  Nmap scan report for 192.168.4.1  
>  Host is up (0.00028s latency).  
>  Not shown: 995 closed ports  
>  PORT STATE SERVICE  
>  22/tcp open ssh  
>  111/tcp open rpcbind  
>  873/tcp open rsync  
>  7777/tcp open cbt  
>  8888/tcp open sun-answerbook  
>  MAC Address: 00:0C:29:56:DE:46 (VMware)
> 
>  Nmap done: 1 IP address (1 host up) scanned in 1.31 seconds
> 
>  
 
## （2） sT：3次握手方式tcp的扫描

 Tcp connect() scan (sT)和上面的Tcp SYN 对应，TCP connect()扫描就是默认的扫描模式.  
 不同于Tcp SYN扫描,Tcp connect()扫描需要完成三次握手,并且要求调用系统的connect().

 优点：不需root权限。普通用户也可以使用。  
 缺点：这种扫描很容易被检测到，在目标主机的日志中会记录大批的连接请求以及错误信息，由于它要完成3次握手，效率低，速度慢，建议使用-sS

 
> nmap -sT 192.168.4.1等同于 nmap 192.168.4.1  
>  [root@78778e06dc0a /]# nmap -sT 192.168.4.1  
>    
>  Starting Nmap 5.51 ( http://nmap.org ) at 2016-12-29 11:40 CST  
>  Nmap scan report for 192.168.4.1  
>  Host is up (0.00048s latency).  
>  Not shown: 995 closed ports  
>  PORT STATE SERVICE  
>  22/tcp open ssh  
>  111/tcp open rpcbind  
>  873/tcp open rsync  
>  7777/tcp open cbt  
>  8888/tcp open sun-answerbook  
>  MAC Address: 00:0C:29:56:DE:46 (VMware)  
>    
>  Nmap done: 1 IP address (1 host up) scanned in 0.27 seconds
> 
>  
 
## （3）sU：udp端口的扫描

 Udp scan(sU) 顾名思义,这种扫描技术用来寻找目标主机打开的UDP端口.它不需要发送任何的SYN包，因为这种技术是针对UDP端口的。UDP扫描发送UDP数据包到目标主机，并等待响应,  
 如果返回ICMP不可达的错误消息，说明端口是关闭的，如果得到正确的适当的回应，说明端口是开放的.udp端口扫描速度比较慢

 
> nmap -sU 192.168.4.1
> 
>  
 
## （4）sF：也是tcp的扫描一种，发送一个FIN标志的数据包

 FIN scan(sF)  
 有时候TcpSYN扫描不是最佳的扫描模式,因为有防火墙的存在.目标主机有时候可能有IDS和IPS系统的存在,防火墙会阻止掉SYN数据包。发送一个设置了FIN标志的数据包并不需要完成TCP的握手.  
 和sS扫描效果差不多，比sT速度快

 
> [root@78778e06dc0a /]# nmap -sF 192.168.4.1  
>    
>  Starting Nmap 5.51 ( http://nmap.org ) at 2016-12-29 11:46 CST  
>  Nmap scan report for 192.168.4.1  
>  Host is up (0.00050s latency).  
>  Not shown: 997 closed ports  
>  PORT STATE SERVICE  
>  22/tcp open|filtered ssh  
>  111/tcp open|filtered rpcbind  
>  873/tcp open|filtered rsync  
>  MAC Address: 00:0C:29:56:DE:46 (VMware)  
>    
>  Nmap done: 1 IP address (1 host up) scanned in 2.59 seconds
> 
>  
 -sF、-sX、-sN  
 秘密FIN数据包扫描、圣诞树(XmasTree)、空(Null)扫描模式  
 有的防火墙可能专门阻止-sS扫描。使用这些扫描可以发送特殊标记位的数据包  
 比如，-sF发送一个设置了FIN标志的数据包  
 它们和-sS一样也需要完成TCP的握手.  
 和sS扫描效果差不多，都比sT速度快  
 除了探测报文的标志位不同，三种扫描在行为上一致  
 优势：能躲过一些无状态防火墙和报文过滤路由器，比SYN还要隐秘  
 劣势：现代的IDS产品可以发现，并非所有的系统严格遵循RFC 793

 即使SYN扫描都无法确定的情况下使用：一些防火墙和包过滤软件能够对发送到被限制端口的SYN数据包进行监视，  
 而且有些程序比如synlogger和courtney能够检测那些扫描。使用-sF、-sX、-sN可以逃过这些干扰。  
 这些扫描方式的理论依据是：关闭的端口需要对你的探测包回应RST包，而打开的端口必需忽略有问题的包。  
 FIN扫描使用暴露的FIN数据包来探测，而圣诞树扫描打开数据包的FIN、URG和PUSH标志。  
 由于微软决定完全忽略这个标准，另起炉灶。所以这种扫描方式对Windows无效。  
 不过，从另外的角度讲，可以使用这种方式来分别两种不同的平台。  
 如果使用这种扫描方式可以发现打开的端口，你就可以确定目标注意运行的不是Windows系统。  
 如果使用-sF、-sX或者-sN扫描显示所有的端口都是关闭的，而使用-sS（SYN）扫描显示有打开的端口，你可以确定目标主机可能运行的是Windwos系统。  
 现在这种方式没有什么太大的用处，因为nmap有内嵌的操作系统检测功能。还有其它几个系统使用和windows同样的处理方式，包括Cisco、BSDI、HP/UX、MYS、IRIX。  
 在应该抛弃数据包时，以上这些系统都会从打开的端口发出复位数据包。

 

 
## （5）sW：窗口扫描

 Window扫描，即窗口扫描  
 当然也可以利用Window扫描方式，得出一些端口信息，可以与之前扫描分析的结果相互补充。Window扫描方式只对某些TCPIP协议栈才有效。  
 它也是基于tcp的扫描，个人感觉用处不大  
 另外我尝试使用它对A机器的22端口扫描，发现对方22端口状态居然是错误的。

 
> >  [root@78778e06dc0a /]# nmap -sW 192.168.4.1 -p22  
>    
>  Starting Nmap 5.51 ( http://nmap.org ) at 2016-12-29 13:17 CST  
>  Nmap scan report for 192.168.4.1  
>  Host is up (0.0027s latency).  
>  PORT STATE SERVICE  
>  22/tcp closed ssh  
>  MAC Address: 00:0C:29:56:DE:46 (VMware)  
>    
>  Nmap done: 1 IP address (1 host up) scanned in 0.34 seconds
> 
>  
 
## （6） sV：版本检测(sV)

 版本检测是用来扫描目标主机和端口上运行的软件的版本，如下扫描，多出了ssh的版本信息

 
> [root@78778e06dc0a /]# nmap -sV 192.168.4.1  
>    
>  Starting Nmap 5.51 ( http://nmap.org ) at 2016-12-29 13:18 CST  
>  Nmap scan report for 192.168.4.1  
>  Host is up (0.00017s latency).  
>  Not shown: 997 closed ports  
>  PORT STATE SERVICE VERSION  
>  22/tcp open ssh OpenSSH 5.3 (protocol 2.0)  
>  111/tcp open rpcbind  
>  873/tcp open rsync (protocol version 30)  
>  MAC Address: 00:0C:29:56:DE:46 (VMware)  
>    
>  Service detection performed. Please report any incorrect results at http://nmap.org/submit/ .  
>  Nmap done: 1 IP address (1 host up) scanned in 6.60 seconds
> 
>  
 
# nmap及其少用的

 （1）iR Choose random targets，它会随机找几个ip或者主机名进行扫描  
 nmap -iR 2 -Pn -p22

 （2）--top-ports <number>: Scan <number> most common ports  
 #扫描常用的端口，number如果写成10，那就是扫描最常用的10个端口。比如，ssh，http，ftp等热门端口  
 nmap --top-ports 5 192.168.4.1

 （3）--port-ratio <ratio>: Scan ports more common than <ratio> #扫描常用端口里，占的比重在0.x 之上的端口  
 比如ratio=0.2 那么就是常用端口中占的分量超过0.2的端口，比如http的80端口  
 nmap --port-ratio 0.1 192.168.4.1

 4）-sO：探测对方，TCP/IP协议簇中有哪些协议，类型号分别是多少  
 nmap -sO 192.168.4.1

 没什么用，就是探测对方，TCP/IP协议簇中有哪些协议，类型号分别是多少  
 icmp即是 1 Internet控制消息  
 6 传输控制 协议  
 udp即是 17 用户数据报文  
 47 通用路由封装  
 103 协议独立多播

 （5）--allports  
 --allports (不为版本探测排除任何端口)经过我的测试，发现对于一些大的端口号，它没能检测出来 默认情况下,Nmap版本探测会跳过9100 TCP端口,因为一些打印机简单地打印送到该端口的任何数据,这回导致数十页HTTP get请求,二进制SSL会话请求等等被打印出来.这一行为可以通过修改或删除nmap-service-probes中的Exclude指示符改变,您也可以不理会任何Exclude指示符,指定--allports扫描所有端口

 -S：可以伪装源地址进行扫描。这样好处在于不会被对方发现自己的真实IP  
 [root@78778e06dc0a /]# nmap -e eth0 192.168.4.1 -S 192.168.4.10  
 WARNING: If -S is being used to fake your source address, you may also have to use -e <interface> and -Pn . If you are using it to specify your real source address, you can ignore this warning.  
 上面提示如果你使用-S伪装自己源地址进行扫描的话，你必须另外使用-e 指定网卡和-Pn参数才能伪装  
 把自己源地址伪装成192.168.4.10扫描A机器  
 nmap -e eth0 192.168.4.1 -S 192.168.4.10 -Pn

 nmap -iflist：查看本地路由与接口  
 Nmap中提供了–iflist选项来查看本地主机的接口信息与路由信息。当遇到无法达到目标主机或想选择从多块网卡中某一特定网卡访问目标主机时，可以查看nmap –iflist中提供的网络接口信息。  
 和route -n功能一样

 

 原文: [https://www.cnblogs.com/nmap/p/6232969.html](https://www.cnblogs.com/nmap/p/6232969.html)

   
   
   
   
 
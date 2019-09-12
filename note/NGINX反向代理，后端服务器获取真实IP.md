---
title: NGINX反向代理，后端服务器获取真实IP
date: 2018-01-21 16:09:04
tags: CSDN迁移
---
 [ ](http://creativecommons.org/licenses/by-sa/4.0/) 版权声明：本文为博主原创文章，遵循[ CC 4.0 BY-SA ](http://creativecommons.org/licenses/by-sa/4.0/)版权协议，转载请附上原文出处链接和本声明。  本文链接：[https://blog.csdn.net/Nedved_L/article/details/79120869](https://blog.csdn.net/Nedved_L/article/details/79120869)   
    
  一般使用中间件做一个反向代理后，后端的web服务器是无法获取到真实的IP地址。

 但是生产上，这又是不允许的，那么怎么解决?

 1.在NGINX反向代理服务器上进行修改   
 2.修改后端web服务器配置文件

 NGINX：

 需要添加一个模块来处理包头   
 1.可通过添加http_realip_module模块来获取真实客户端IP地址   
 2.修改NGINX配置文件启动模块作用

 
```
vim  /usr/local/nginx/conf/nginx.conf
  location / {
... 
proxy_set_header    Host                        $host;
                proxy_set_header    X-Real-IP                    $remote_addr;
                proxy_set_header    X-Forwarded-For              $proxy_add_x_forwarded_for;
                proxy_set_header    HTTP_X_FORWARDED_FOR      $remote_addr;
        }
```
 3.重启NGINX服务器

 
## 后端web服务器修改

 后端服务器   
 粟子：Apche，NGINX，Tomcat

 NGINX：   
 修改配置文件

 
```
vim  /usr/local/nginx/conf/nginx.conf

http {
    include       mime.types;
    default_type  application/octet-stream;
        set_real_ip_from 192.168.4.1;     //添加nginx代理服务器真实IP
}
```
 重启服务

 Apache：   
 说明：Apache获取真实IP地址有2个模块：   
 mod_rpaf：Apache-2.2支持；Apache-2.4不支持；   
 mod_remoteip：Apache-2.4自带模块；Apache-2.2 支持

 示范为2.4   
 1修改配   
  `vim /usr/local/apache/conf/httpd.conf`    
 如下代码去掉注释，

 
```
#LoadModule remoteip_module modules/mod_remoteip.so
```
 3添加如下代码

 
```
RemoteIPHeader X-Forwarded-For
RemoteIPInternalProxy 192.168.4.1     //IP为nginx代理服务器的地址
```
 4修改日志显示格式   
 添加%a，日志中第二例显示真实IP

 
```
LogFormat "%h %a %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" combined
LogFormat "%h %a %l %u %t \"%r\" %>s %b" common
```
 5重启服务

 Tomcat：

 修改配置文件

 
```
vim /usr/local/tomcat/conf/server.xml
```
 修改pattern格式

 
```
<Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"  
 prefix="localhost_access_log." suffix=".txt"  
 pattern="%h  %{X-FORWARDED-FOR}i %l %u %t %r %s %b %D %q %{User-Agent}i  %T" resolveHosts="false" />
```
 重启服务

   
  

 创建目录：本文档中所有内容安装到/fdfs目录

 
> [fdfs@5861be93b5b0 /]$mkdir -p /fdfs/fastdfs/data /fdfs/nginx/nginx_temp /fdfs/soft && ln -s /fdfs/fastdfs/data /fdfs/fastdfs/data/M00
> 
>  [fdfs@5861be93b5b0 /]$sudo yum install gcc make gcc-c++ -y
> 
>  
 上传文件：  
 上传【Nginx+Fastdfs.zip】包到安装目录/fdfs/soft，执行解压到soft目录

 
> [fdfs@5861be93b5b0 /]$cd /fdfs/soft && unzip Nginx+Fastdfs.zip
> 
>  
 

 
## 1.2. libfastcommon , fastdfs  
 

 
### 1.2.1安装libfastcommon 

 
> [fdfs@5861be93b5b0 /]$cd /fdfs/soft && tar -xvf libfastcommon.tar&&cd libfastcommon
> 
>  [fdfs@5861be93b5b0 /]$./make.sh && ./make.sh install
> 
>  
 

 
### 1.2.2安装 fastdfs

 
> >  [fdfs@5861be93b5b0 /]$cd /fdfs/soft && tar -xvf FastDFS_v5.08.tar.gz && cd FastDFS
> 
>  
 

 
### 1.2.3拷贝配置文件到/fdfs/fastdfs/config  
 

 
> [fdfs@5861be93b5b0 /]$TARGET_CONF_PATH=/fdfs/fastdfs/config  
>  
> 
>  [fdfs@5861be93b5b0 /]$
> 
>  cat > ./1.sh <<EOF  
>  TARGET_CONF_PATH=/fdfs/fastdfs/config  
>  if [ ! -d \$TARGET_CONF_PATH ]; then  
>  mkdir -p \$TARGET_CONF_PATH
> 
>  fi  
>  cp -f conf/tracker.conf \$TARGET_CONF_PATH  
>  cp -f conf/storage.conf \$TARGET_CONF_PATH  
>  cp -f conf/client.conf \$TARGET_CONF_PATH  
>  cp -f conf/http.conf \$TARGET_CONF_PATH  
>  cp -f conf/mime.types \$TARGET_CONF_PATH
> 
>  >  EOF  
>  [fdfs@5861be93b5b0 /]$sh 1.sh&&rm -f 1.sh
> 
>  
 
### 1.2.4修改make文件配置路径为/fdfs/fastdfs

 
> [fdfs@5861be93b5b0 /]$sed -i 's#$DESTDIR/usr$#/fdfs/fastdfs#' make.sh  
>  [fdfs@5861be93b5b0 /]$sed -i 's#$DESTDIR/etc/fdfs#/fdfs/fastdfs/config#' make.sh  
>  [fdfs@5861be93b5b0 /]$sed -i 's#$DESTDIR/etc/init.d#/fdfs/fastdfs/init.d#' make.sh
> 
>  
 
### 1.2.5安装

 
> [fdfs@5861be93b5b0 /]$./make.sh && ./make.sh install
> 
>  
 

 
## 1.2.2. 验证

 进入/fdfs/fastdfs，出现以下5个目录bin、config、data（手工创建）、include、lib、init.d

 
> [fdfs@5861be93b5b0 /]$ll /fdfs/fastdfs  
>  total 312  
>  drwxr-xr-x. 2 root root 4096 Nov 5 06:20 bin  
>  drwxr-xr-x. 2 root root 204 Nov 5 06:46 config  
>  drwxr-xr-x. 2 root root 17 Nov 5 06:17 data  
>  drwxr-xr-x. 3 root root 21 Nov 5 06:20 include  
>  drwxr-xr-x. 2 root root 48 Nov 5 06:20 init.d  
>  drwxr-xr-x. 2 root root 54 Nov 5 06:56 lib64
> 
>  
> 
>  
 把libfastcommon 的动态库关联到fdfs

 
> ln -s /usr/lib64/libfastcommon.so /fdfs/fastdfs/lib64/libfastcommon.so  
>  ln -s /usr/lib64/libfastcommon.so /usr/lib/libfastcommon.so  
>  ln -s /fdfs/fastdfs/lib64/libfdfsclient.so /usr/lib64/libfdfsclient.so  
>  ln -s /fdfs/fastdfs/lib64/libfdfsclient.so /usr/lib/libfdfsclient.so  
>  ln -s /usr/include/fastcommon/* /fdfs/fastdfs/include/fastdfs/
> 
>  
 
## 1.3. nginx

 
### 1.3.1. 解压依赖库

 
> [fdfs@5861be93b5b0 /]$cd /fdfs/soft && tar -zxvf pcre-8.42.tar.gz && tar -zxvf zlib-1.2.11.tar.gz && tar -zxvf openssl-1.0.2n.tar.gz
> 
>  
 
### 1.3.2. 配置fastdfs-nginx-module模块

 
> >  [fdfs@5861be93b5b0 /]$cd /fdfs/soft && tar -zxvf fastdfs-nginx-module_v1.16.tar.gz && cd fastdfs-nginx-module/src && cp mod_fastdfs.conf /fdfs/fastdfs/config
> 
>  修改对应的库路径  
>  sed -i 's#/usr/local/#/fdfs/fastdfs/#g' config  
>  sed -i 's#/etc/fdfs/#/fdfs/fastdfs/config/#' config  
>  sed -i 's#lib#lib64#' config
> 
>  
 1.3.3. 安装nginx

 
> [fdfs@5861be93b5b0 /]$cd /fdfs/soft && tar -xvf nginx-1.15.5.tar.gz && cd nginx-1.15.5 && ./configure --prefix=/fdfs/nginx --with-pcre=/fdfs/soft/pcre-8.42 --with-zlib=/fdfs/soft/zlib-1.2.11 --with-openssl=/fdfs/soft/openssl-1.0.2n --add-module=/fdfs/soft/fastdfs-nginx-module/src && make && make install
> 
>  
 1.3.4. 验证  
 进入/nginx/nginx，出现以下5个目录conf、html、logs、nginx_temp（手工创建）、sbin

 
> [root@72fcbfa4c397 libfastcommon-master]$ ll /fdfs/nginx/  
>  total 4  
>  drwxr-xr-x. 2 root root 4096 Dec 20 08:54 conf  
>  drwxr-xr-x. 2 root root 40 Dec 20 08:54 html  
>  drwxr-xr-x. 2 root root 6 Dec 20 08:54 logs  
>  drwxr-xr-x. 2 root root 6 Dec 20 07:33 nginx_temp  
>  drwxr-xr-x. 2 root root 19 Dec 20 08:54 sbin
> 
>  
 

 
# 2. 配置

 
### 2.1 dfs的storage

 
> [fdfs@5861be93b5b0 /]$cd /fdfs/fastdfs/config  
>  [fdfs@5861be93b5b0 /]$vi storage.conf
> 
>  port=23000 /* storage的端口，默认为23000*/
> 
>  group_name=group1 /*分组，默认为group1*/
> 
>  base_path=/fdfs/fastdfs /*放置data和log的目录*/
> 
>  store_path0=/fdfs/fastdfs /*放置文件的目录*/
> 
>  tracker_server= 192.168.1.1:22122 /*tracker server的ip和端口，  
>  tracker_server= 192.168.1.2:22122 可以写多个tracker server，每行一个*/
> 
>  http.server_port=8080 /*web server的端口改成8080*/
> 
>  
 

 

 
## 2.2. dfs的tracker

 
### 2.2.1. 修改配置tracker.conf文件

 
> [fdfs@5861be93b5b0 /]$cd /fdfs/fastdfs/config  
>  [fdfs@5861be93b5b0 /]$vi tracker.conf
> 
>  >  port=22122 /* tracker的端口，默认为22122*/
> 
>  base_path=/fdfs/fastdfs /*放置data和log的目录*/
> 
>  store_group=group1 /*分组，改为group1*/
> 
>  reserved_storage_space = 10% /*磁盘小于10%不允许上传*/
> 
>  http.server_port=8080 /*web server的端口改成8080*/
> 
>  
 

 
## 2.3. 配置nginx的插件

 
### 2.3.1. 修改配置mod_fastdfs.conf文件

 
> >  [fdfs@5861be93b5b0 /]$cp /fdfs/soft/fastdfs-nginx-module/src/mod_fastdfs.conf /fdfs/fastdfs/config  
>  [fdfs@5861be93b5b0 /]$cd /fdfs/fastdfs/config  
>  [fdfs@5861be93b5b0 /]$vi mod_fastdfs.conf  
>   
>  connect_timeout=30
> 
>  network_timeout=60
> 
>  base_path=/fdfs/fastdfs /*放置log的目录*/
> 
>  >  tracker_server= 192.168.1.1:22122 /*tracker server的ip和端口，  
>  tracker_server= 192.168.1.2:22122 可以写多个tracker server，每行一个*/
> 
>  storage_server_port=23000 /* storage的端口*/
> 
>  group_name=group1 /*分组，默认为group1*/
> 
>  url_have_group_name=true /*是否在URL中包含group名称*/
> 
>  store_path0=/fdfs/fastdfs /*放置文件的目录*/
> 
>  http.need_find_content_type=true
> 
>  /*增加内容*/  
>  http.mime_types_filename=/fdfs/fastdfs/config/mime.types  
>  http.default_content_type=application/octet-stream  
>  include /fdfs/fastdfs/config/http.conf
> 
>  
 

 
## 2.3.2. 修改配置http.conf文件

 
> [fdfs@5861be93b5b0 /]$cd /fdfs/fastdfs/config  
>  [fdfs@5861be93b5b0 /]$vi http.conf
> 
>  http.mime_types_filename=/fdfs/fastdfs/config/mime.types
> 
>  http.anti_steal.token_check_fail=/fdfs/fastdfs/config/anti-steal.jpg
> 
>  
 

 
## 2.3.3. 修改mime.types

 
> [fdfs@5861be93b5b0 /]$cd /fdfs/fastdfs/config  
>  [fdfs@5861be93b5b0 /]$vi mime.types
> 
>  /*增加内容*/  
>  #--add next contents  
>  application/vnd.ms-word.document.macroEnabled.12 docm  
>  application/vnd.openxmlformats docx pptx xlsx  
>  application/vnd.ms-word.template.macroEnabled.12 dotm  
>  application/vnd.openxmlformats-officedocument.wordprocessingml.template dotx  
>  application/vnd.ms-powerpoint.template.macroEnabled.12 potm  
>  application/vnd.openxmlformats-officedocument.presentationml.template potx  
>  application/vnd.ms-powerpoint.addin.macroEnabled.12 ppam  
>  application/vnd.ms-powerpoint.slideshow.macroEnabled.12 ppsm  
>  application/vnd.openxmlformats-officedocument.presentationml.slideshow ppsx  
>  application/vnd.ms-powerpoint.presentation.macroEnabled.12 pptm  
>  application/vnd.ms-excel.addin.macroEnabled.12 xlam  
>  application/vnd.ms-excel.sheet.binary.macroEnabled.12 xlsb  
>  application/vnd.ms-excel.sheet.macroEnabled.12 xlsm  
>  application/vnd.openxmlformats-officedocument.spreadsheetml.template xltx  
>  application/vnd.ms-excel.template.macroEnabled.12 xltm
> 
>  
 

 
## 2.3.4. 修改client.conf

 
> [fdfs@5861be93b5b0 /]$cd /fdfs/fastdfs/config  
>  [fdfs@5861be93b5b0 /]$vi client.conf
> 
>  base_path=/fdfs/fastdfs /*放置文件的目录*/
> 
>  tracker_server=192.168.1.1:22122 /*tracker server的ip和端口，  
>  tracker_server=192.168.1.2:22122 可以写多个tracker server，每行一个*/
> 
>  http.tracker_server_port=8080
> 
>  
 

 
## 2.4. 配置nginx

 
### 2.4.1. 修改配置nginx.conf文件

 
> [fdfs@5861be93b5b0 /]$cd /fdfs/nginx/conf  
>  [fdfs@5861be93b5b0 /]$vi nginx.conf
> 
>  >  使用以下内容：  
>  worker_processes 4;  
>  error_log logs/error.log;  
>  events {  
>  worker_connections 1024;  
>  use epoll;  
>  }  
>  http {  
>  include mime.types;  
>  default_type application/octet-stream;  
>  sendfile on;  
>  keepalive_timeout 65;  
>  server {  
>  listen 8080;  
>  server_name localhost;  
>  access_log logs/access.log;
> 
>  location ~ /group1/M00/* {  
>  root /fdfs/fastdfs/data;  
>  ngx_fastdfs_module;  
>  client_max_body_size 10m;  
>  client_body_temp_path /fdfs/nginx/nginx_temp;  
>  }  
>  }
> 
>  upstream group1 {  
>  server 192.168.1.1:8080 ;  
>  server 192.168.1.2:8080 ;  
>  }
> 
>  server {  
>  listen 8081;  
>  server_name localhost;  
>  location /group1/M00 {  
>  proxy_pass http://group1;  
>  proxy_redirect off;  
>  proxy_set_header Host $host;  
>  proxy_set_header X-Real-IP $remote_addr;  
>  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;  
>  }  
>  }  
>  }
> 
>  
 2.5. Nginx 启动

 
> 启动：/fdfs/nginx/sbin/nginx -c /fdfs/nginx/conf/nginx.conf  
>  停止：/fdfs/nginx/sbin/nginx -s stop  
>  重载：/fdfs/nginx/sbin/nginx -s reload  
>  日志：/fdfs/nginx/logs/error.log  
>  /fdfs/nginx/logs/access.log
> 
>  
 首次启动可能出现以下错误  
 /fdfs/nginx/sbin/nginx: error while loading shared libraries: libfastcommon.so: cannot open shared object file: No such file or directory  
 解决办法：

 
> echo "export LD_LIBRARY_PATH=/fdfs/fastdfs/lib:$LD_LIBRARY_PATH" >> ~/.bash_profile  
>  source ~/.bash_profile
> 
>  
 
## 2.6. Tracker命令

 
> 启动：/fdfs/fastdfs/bin/fdfs_trackerd /fdfs/fastdfs/config/tracker.conf  
>  停止：killall fdfs_trackerd  
>  日志：/fdfs/fastdfs/logs/trackerd.log
> 
>  
 
## 2.7. Storage命令

 
> 启动：
> 
>  /fdfs/fastdfs/bin/fdfs_storaged /fdfs/fastdfs/config/storage.conf
> 
>  data path: /home/yuqing/fastdfs/data, mkdir sub dir...  
>  mkdir data path: 00 ...  
>  mkdir data path: 01 ...  
>  mkdir data path: 02 ...
> 
>  
 停止：killall fdfs_storaged  
 日志：/fdfs/fastdfs/logs/storaged.log

 
## 2.8. 测试

 集群测试首次需要两台都启动，否则会报错

 
## 2.8.1. 测试分布式文件系统上传图片

 上传命令格式：  
 /fdfs/fastdfs/bin/fdfs_upload_file /fdfs/fastdfs/config/storage.conf 目标文件

 
> [root@72fcbfa4c397 /]# /fdfs/fastdfs/bin/fdfs_upload_file /fdfs/fastdfs/config/storage.conf skr.jpg  
>  group1/M00/00/00/rBEAAlwbcieANq7nAAL_SObQPXc730.jpg
> 
>  
 在两台服务器的/fdfs/fastdfs/data/00/00目录可以找到此文件

 2.8.2. 测试负载均衡系统访问

 以下地址均可访问到内容  
 http://192.168.4.1:8080/group1/M00/00/00/rBEAAlwbcieANq7nAAL_SObQPXc730.jpg  
 http://192.168.1.2:8080/group1/M00/00/00/rBEAAlwbcieANq7nAAL_SObQPXc730.jpg  
 http://192.168.1.1:8081/group1/M00/00/00/rBEAAlwbcieANq7nAAL_SObQPXc730.jpg  
 http://192.168.1.2:8081/group1/M00/00/00/rBEAAlwbcieANq7nAAL_SObQPXc730.jpg

 

 ![](https://img-blog.csdnimg.cn/20181220184705616.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L05lZHZlZF9M,size_16,color_FFFFFF,t_70)

 

 

 

 

 

 

 

 

 

 

   
   
 
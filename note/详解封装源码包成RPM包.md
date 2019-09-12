 
## 1.安装rpm-build软件包

 rpm-bulid 打包所用的工具

 
```
[root@W1 root]# yum install rpm-build
```
 
## 2.生成rpmbuild目录结构

 
```
[root@W1 root]# rpmbuild -ba nginx.spec
错误：stat /root/nginx.spec 失败：没有那个文件或目录
```
 会报错，不过没问题 需要的只是生成的目录rombuild

 
```
[root@W1 rpmbuild]# pwd
/root/rpmbuild

[root@W1 rpmbuild]# ls
BUILD  BUILDROOT  RPMS  SOURCES  SPECS  SRPMS
```
 RPMS（做好后的成品放置区）   
 SOURCES(放置源码包)   
 SPECS（配置文件）

  -  
## 3.将源码软件复制到SOURCES目录

 
```
[root@W1 rpmbuild]# cp nginx-1.8.0.tar.gz /root/rpmbuild/SOURCES/
```
 记得是源码包

 
## 4.创建并修改spec配置文件

 [root@W1 rpmbuild]# vim /root/rpmbuild/SPECS/nginx.spec   
 文件后缀必须是spec，格式嘛   
 如何修改参考内德给的图

 ![这里写图片描述](https://img-blog.csdn.net/20171116102455373?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvTmVkdmVkX0w=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)   
 ！！标注的地方不能随便该   
 这是nginx修改的参考

 
```
[root@Web rpmbuild]# cat SPECS/nginx.spec 
Name:nginx  
Version:1.8.0   
Release:1.rhel7
Summary:The is a Web Server,to nginx

#Group: 
License:GPL 
URL:www.Nedved.cn   
Source0:nginx-1.8.0.tar.gz  

BuildRequires:  gcc pcre openssl-devel
#Requires:  

%description
This is a Web server nginx 

%prep
%setup -q


%build
./configure --with-http_ssl_module --with-http_stub_status_module
make %{?_smp_mflags}


%install
make install DESTDIR=%{buildroot}


%files
%doc
/usr/local/nginx/*


%changelog
```
 到这里就基本上打包完成了

 
## 5.使用配置文件创建RPM包

 1）安装依赖软件包

 
```
[root@W1 rpmbuild]# yum install gcc pcre openssl-devel -y
```
 2）rpmbuild创建RPM软件包

 
```
 [root@W1 rpmbuild]# rpmbuild -ba SPECS/nginx.spec
```
 
```
[root@W1 rpmbuild]# ls RPMS/x86_64/
nginx-1.8.0-1.rhel7.x86_64.rpm  nginx-debuginfo-1.8.0-1.rhel7.x86_64.rpm
```
 创建RPM软件包后在查看RPMS就能看到封装好的包了

 
## 6.测试RPM包是否可使用

 rpm -qpi RPMS/x86_64/nginx-1.8.0-1.rhel7.x86_64.rpm //查看封装信息   
 rpm -qpl RPMS/x86_64/nginx-1.8.0-1.rhel7.x86_64.rpm //查看安装路径   
 rpm -ivh RPMS/x86_64/nginx-1.8.0-1.rhel7.x86_64.rpm //装包   
 rpm -qa |grep nginx //查看是否安装

 OK到这里就大功告成了。   
 最后就总结下：   
 1.准备封装工具   
 2.改安装配置文件，在里面添加所需要的模块，附加一些安装信息。   
 3.然后就是测试了

   
  
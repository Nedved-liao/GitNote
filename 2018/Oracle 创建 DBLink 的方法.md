   # Oracle 创建第一个DBLink 的方法

 当用户要跨本地数据库，访问另外一个数据库表中的数据时，本地数据库中必须创建了远程数据库的dblink,通过dblink本地数据库可以像访问本地数据库一样访问远程数据库表中的数据。

 
## 创建

 
> create database link FistDBlink  
>  connect to dbName identified by dbPassword  
>  using '(DESCRIPTION =(ADDRESS_LIST =(ADDRESS =(PROTOCOL = TCP)(HOST = 192.168.4.1)(PORT = 1521)))(CONNECT_DATA =(SERVICE_NAME = orcl)))';
> 
>  
 
## 查询

 
### 1.查看所有的数据库链接，登录管理员查看 

 
> select owner,object_name from dba_objects where object_type='DATABASE LINK';
> 
>  
 
### 2.查询、删除和插入数据和操作本地的数据库是一样,不过表名需要写成“表名@dblink服务器”

 
> select * from db.tb_test@FistDBlink;
> 
>  
 
### 3.删除数据库连接

 
> drop database link FistDBlink;
> 
>  
   
   
   
 
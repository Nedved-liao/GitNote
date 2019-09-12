  # []()事件背景

 最近数据库全有oracle11g升级到oracle12C后，遇到了不小的问题  
 升级后应用的报错ORA-28040: No matching authentication protocol

 
# []()ORA-28040: No matching authentication protocol

 服务端是Oracle12C  
 客户端低于服务端版本

 官方给出的错误

 
> ORA-28040: No matching authentication protocol  
>   
>  Cause: No acceptible authentication protocol for both client and server  
>   
>  Action: Administrator should set SQLNET_ALLOWED_LOGON_VERSION parameter on both client and servers to values that matches the minimum version supported in the system.
> 
>  
 给出的解决办法是：

 
> 操作：管理员应将客户端和服务器上的SQLNET_ALLOWED_LOGON_VERSION参数设置为与系统中支持的最低版本匹配的值
> 
>  
 
## []()解决过程

 1.在$TNS_ADMIN下新建sqlnet.ora(若已存在，直接写入或更改)

 
```
SQLNET.ALLOWED_LOGON_VERSION_SERVER=8
SQLNET.ALLOWED_LOGON_VERSION_CLIENT=8


```
 2.重启监听

 
```
lsnrctl stop && lsnrctl start
lsnrctl status

```
 
> **Note "**  
>   
>  需要注意的是低版本的JDK也不无法加载Oracle12C的JDBC，所以要升级一下  
>  $TNS_ADMIN 即tnsnames.ora所在目录  
>   
>  通常同时还会出现**ORA-01017**
> 
>  
 
# []()ORA-01017: invalid username/password; logon denied

 使用PL/SQL或者在JDBC连接报错中遇到ORA-01017

 字面就是错误的用户名或密码  
 这个就逐一排查用户名和密码就行了

 第三个原因，是因为password version

 
## []()查看password version，可以看到只有12C的

 
```
SQL> select username,password_versions from dba_users where username='TESTUSER01';

USERNAME
--------------------------------------------------------------------------------
PASSWORD_VERSIONS
-----------------
TESTUSER01
12C

```
 
## []()解决方法

 重置一下密码

 
```
alter user testuser01 identified by testuser01;
SQL> select username,password_versions from dba_users where username='TESTUSER01';
USERNAME
--------------------------------------------------------------------------------
PASSWORD_VERSIONS
-----------------
TESTUSER01
10g 11G 12C

```
 重新使用PL/SQL连接没有问题

 21.5.2019/V1

   
  
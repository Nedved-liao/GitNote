# oracle 闪回概述

## 概念

![闪回flashback](C:\Users\Nedved\Desktop\闪回flashback.png)



闪回查询、闪回版本查询、闪回事务查询属于行级闪回。这三种闪回技术全部依赖于undo表空间中的undo数据。
闪回表、闪回删除属于表级闪回。闪回表也是从undo中读取数据，闪回删除是依赖recyclebin 
闪回数据库属于数据库级闪回。

## 场景应用

| 闪回级别 | 闪回场景                           | 闪回技术              | 对象依赖           | 影响数据 |
| -------- | ---------------------------------- | --------------------- | ------------------ | -------- |
| 数据库   | 表截断、逻辑错误、其他多表意外事件 | 闪回DATABASE          | 闪回日志、undo     | 是       |
| DROP     | 删除表                             | 闪回DROP              | 回收站(recyclebin) | 是       |
| 表       | 更新、删除、插入记录               | 闪回TABLE             | 还原数据,undo      | 是       |
| 查询     | 当前数据和历史数据对比             | 闪回QUERY             | 还原数据,undo      | 否       |
| 版本查询 | 比较行版本                         | 闪回Version Query     | 还原数据,undo      | 否       |
| 事务查询 | 比较                               | 闪回Transaction Query | 还原数据,undo      | 否       |
| 归档     | DDL、DML                           | 闪回Archive           | 归档日志           | 是       |



## 设置闪回

### 查看是否开启闪回

```sql
SQL> select flashback_on from V$database;
FLASHBACK_ON
------------------
NO
```



在开启闪回功能之前，必须先开启数据库归档

### 查看是否开启归档日志

```sql
SQL> archive log list;
Database log mode Archive Mode
Automatic archival Enabled
Archive destination /home/U01/app/oracle/oradata/testdb/arch
Oldest online log sequence 844
Next log sequence to archive 846
Current log sequence 846
```

如果没开启,在mount 状态开启

```sql
alter database archivelog;
```

### 设置合理的闪回区

> db_recovery_file_dest：指定闪回恢复区的位置
> db_recovery_file_dest_size：指定闪回恢复区的可用空间大小
> db_flashback_retention_target：指定数据库可以回退的时间，单位为分钟，默认1440分钟(1天),实际取决于闪回区大小



设置闪回区

```sql
SQL> alter system set db_recovery_file_dest_size=60G scope=both;
System altered.

SQL> alter system set db_recovery_file_dest='/home/U01/app/oracle/fast_recovery_area' scope=both;
System altered.

SQL> alter system set db_flashback_retention_target=4320 scope=both;
System altered.   

SQL> show parameter db_recovery

NAME TYPE VALUE

------------------------------------ ----------- ------------------------------

db_recovery_file_dest string /home/U01/app/oracle/fast_recovery_area

db_recovery_file_dest_size big integer 60G
```

### 开启闪回

开启闪回功能

需要注意的一点是

1. 在10G中，如果要开启数据库级别的闪回，需要设置相关的参数，并且使数据库处于归档模式，然后在MOUNT状态(mount状态：alter database archivelog;)下开启闪回。
2. 在11G中，如果设置了相关的参数及其开启了归档，那么就可以直接在OPEN状态下打开闪回。

```sql
SQL> select status from v$instance;
STATUS
------------
OPEN

SQL> alter database flashback on;
Database altered.

SQL> select flashback_on from v$database;
FLASHBACK_ON
------------------
YES   
```

### 关闭闪回

```sql
SQL> alter database flashback off;
Database altered.
```



## 闪回使用

**闪回查询**

Flashback query是最基本的闪回功能，直接利用回滚段中的旧数据构造某一个时刻的一致性数据版本。

只适合单个表数据恢复。对事务中相关多表数据恢复不适合，无法确保相关数据的参照完整性。

Flashback query不需要使用resetlogs打开数据库。闪回查询让你能够看到过去某个时间的的数据。能够让你查看和重构以为意外被删除或者该表的数据。

> 闪回查询主要是根据Undo表空间数据进行多版本查询，针对v$和x$动态性能视图无效，但对DBA_、ALL_、USER_是有效的
>
> 允许用户查询过去某个时间点的数据，用以重构由于意外删除或更改的数据，数据不会变化。
>
> 可以根据SCN号和具体时间进行数据库查询



### 根据SCN和时间点的闪回查询

```sql
--初始数据
SQL> select * from scott.dept;
DEPTNO DNAME LOC
---------- -------------- -------------
10 ACCOUNTING NEW YORK
20 RESEARCH DALLAS
30 SALES CHICAGO
40 OPERATIONS BOSTON

--修改时间显示格式
SQL> alter session set nls_date_format='yyyymmdd hh24:mi:ss';
Session altered.

--获取当前时间
SQL> select sysdate from dual;
SYSDATE
-----------------
20190916 14:54:28

--获取当前SCN
SQL> select dbms_flashback.get_system_change_number from dual;
GET_SYSTEM_CHANGE_NUMBER
------------------------
              1136165895

--删除部分数据
SQL> delete from scott.dept where deptno=40;
1 row deleted.
SQL> commit;
Commit complete.

--查看当前数据
SQL> select * from scott.dept;

DEPTNO DNAME LOC
---------- -------------- -------------
10 ACCOUNTING NEW YORK
20 RESEARCH DALLAS
30 SALES CHICAGO

--根据刚才的SCN号查询
SQL> select * from scott.dept as of scn 1136165895;

DEPTNO DNAME LOC
---------- -------------- -------------
10 ACCOUNTING NEW YORK
20 RESEARCH DALLAS
30 SALES CHICAGO
40 OPERATIONS BOSTON

--根据具体时间点查询
select * from t as of timestamp to_timestamp('20190916 14:54:28','yyyymmdd hh24:mi:ss');

DEPTNO DNAME LOC
---------- -------------- -------------
10 ACCOUNTING NEW YORK
20 RESEARCH DALLAS
30 SALES CHICAGO
40 OPERATIONS BOSTON

--根据具体时间点查询,这里使用减一天时间
SQL> select * from scott.dept as of timestamp sysdate-10/1440;

DEPTNO DNAME LOC
---------- -------------- -------------
10 ACCOUNTING NEW YORK
20 RESEARCH DALLAS
30 SALES CHICAGO
40 OPERATIONS BOSTON
```

### 闪回版本查询 

**Flashback Version Query**

用于查询行级数据库随时间变化的方法

一次commit命令就会创建一个版本,闪回版本查询返回在指定时间间隔或SCN间隔内的所有版本

语法:

>SELECT * FROM tablename VERSIONS {BETWEEN {SCN | TIMESTAMP} start AND end} 
>
>--start,end可以是时间也可以是scn



| 参数                          | 说明                                                         |
| :---------------------------- | ------------------------------------------------------------ |
| **versions_start{scn\|time}** | 版本开始的scn或时间戳                                        |
| **versions_end{scn\|time}**   | 版本结束scn或时间戳，如果有值表明此行后面被更改过是旧版本，如果为null，则说明行版本是当前版本或行被删除（即versions_operation值为D） |
| **versions_xid**              | 创建行版本的事务ID                                           |
| **versions_operation**        | 在行上执行的操作（I=插入，D=删除，U=更新）                   |



**示例**

```sql
--查询时间作为timestamp开始时间
SQL> select to_date(sysdate,'yyyy-mm-dd hh24:mi:ss') from dual;

TO_DATE(SYSDATE,'YY
-------------------
2019-09-16 15:59:27

--新建一张测试数据表dept_ts
SQL> create table scott.dept_ts as select * from scott.dept where 1 = 2;
Table created.
        
--插入DEPTNO=40的数据到测试表
SQL> insert into scott.dept_ts select * from scott.dept where DEPTNO = 40;   

1 row created.
        
--插入一行提交作为一个版本
SQL> commit;                                                                               
Commit complete.

--插入两行提交作为一个版本        
SQL> insert into scott.dept_ts select * from scott.dept where DEPTNO = 30;

1 row created.

SQL> insert into scott.dept_ts select * from scott.dept where DEPTNO = 20;

1 row created.

SQL> commit;                                                                             
Commit complete.
        
--再次更改DEPTNO=40的行提交，使这行有旧版本
SQL> update scott.dept_ts set LOC = 'New data' where DEPTNO = 40;

1 row updated.

SQL> commit;                                                                         

Commit complete.

--查询时间作为timestamp结束时间
SQL> select to_date(sysdate,'YYYY-MM-DD HH24:MI:SS') from dual;

TO_DATE(SYSDATE,'YY
-------------------
2019-09-16 16:01:10
        
--查询闪回版本      
SQL> SELECT VERSIONS_STARTSCN,
       VERSIONS_STARTTIME,
       VERSIONS_ENDSCN,
       VERSIONS_ENDTIME,
       VERSIONS_XID,
       VERSIONS_OPERATION,
       DEPTNO
  FROM SCOTT.DEPT VERSIONS BETWEEN TIMESTAMP TO_TIMESTAMP('2019-09-16 15:59:27', 'YYYY-MM-DD HH24:MI:SS') AND TO_TIMESTAMP('2019-09-16 16:01:10', 'YYYY-MM-DD HH24:MI:SS');
        
VERSIONS_STARTSCN VERSIONS_STARTTIME             VERSIONS_ENDSCN VERSIONS_ENDTIME               VERSIONS_XID     VERSIONS_OPERATION        DEPTNO
----------------- ------------------------------ --------------- ------------------------------ ---------------- -------------------- ----------
1032654              16-SEP-19 16:00:13 AM                                                                                            08000E0016030000        U                                    40
1032637              16-SEP-19 16:00:10 AM                                                                                            0600180017030000        I                                    20
1032637              16-SEP-19 16:00:08 AM                        						  											  0600180017030000        I                                    30
1032628              16-SEP-19 16:00:05 AM              1032654                16-SEP-19 16:00:13 AM				                  090014002C030000        I                                    40
```

一次commit是一个版本，当前版本的versions_endscn和versions_endtime值为空，旧版本则有值



### 闪回事务查询

Flashback Transaction Query

实际上是查询的数据字典flashback_transaction_query。

可以根据flashback_transaction_query 的undo_sql列值返回数据以前版本。

```sql
--flashback_transaction_query 列说明
SQL> desc flashback_transaction_query;
Name             Type           Nullable Default Comments                                  
---------------- -------------- -------- ------- ----------------------------------------- 
XID              RAW(8)         Y                Transaction identifier                    
START_SCN        NUMBER         Y                Transaction start SCN                     
START_TIMESTAMP  DATE           Y                Transaction start timestamp               
COMMIT_SCN       NUMBER         Y                Transaction commit SCN                    
COMMIT_TIMESTAMP DATE           Y                Transaction commit timestamp              
LOGON_USER       VARCHAR2(30)   Y                Logon user for transaction                
UNDO_CHANGE#     NUMBER         Y                1-based undo change number                
OPERATION        VARCHAR2(32)   Y                forward operation for this undo           
TABLE_NAME       VARCHAR2(256)  Y                table name to which this undo applies     
TABLE_OWNER      VARCHAR2(32)   Y                owner of table to which this undo applies 
ROW_ID           VARCHAR2(19)   Y                rowid to which this undo applies          
UNDO_SQL         VARCHAR2(4000) Y                SQL corresponding to this undo
```

使用闪回事务查询前，必须启用重做日志流的其他日志记录

重做日志流与Log Miner使用的数据相同，只是接口不同。




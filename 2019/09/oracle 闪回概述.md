# oracle 闪回概述

场景应用

| 闪回级别 | 闪回场景                           | 闪回技术              | 对象依赖           | 影响数据 |
| -------- | ---------------------------------- | --------------------- | ------------------ | -------- |
| 数据库   | 表截断、逻辑错误、其他多表意外事件 | 闪回DATABASE          | 闪回日志、undo     | 是       |
| DROP     | 删除表                             | 闪回DROP              | 回收站(recyclebin) | 是       |
| 表       | 更新、删除、插入记录               | 闪回TABLE             | 还原数据,undo      | 是       |
| 查询     | 当前数据和历史数据对比             | 闪回QUERY             | 还原数据,undo      | 否       |
| 版本查询 | 比较行版本                         | 闪回Version Query     | 还原数据,undo      | 否       |
| 事务查询 | 比较                               | 闪回Transaction Query | 还原数据,undo      | 否       |
| 归档     | DDL、DML                           | 闪回Archive           | 归档日志           | 是       |



查看是否开启归档日志

```sql
SQL> archive log list;
Database log mode Archive Mode
Automatic archival Enabled
Archive destination /home/U01/app/oracle/oradata/testdb/arch
Oldest online log sequence 844
Next log sequence to archive 846
Current log sequence 846
```

如果没开启,在mount

```sql
alter database archivelog;
```






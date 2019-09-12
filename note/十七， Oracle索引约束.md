  ## 管理索引-原理介绍

 索引是用于加速数据存取的数据对象。合理的使用索引可以大大降低i/o次数，从而提高数据访问性能。

 
## 单列索引

 适当的索引对于大型数据库的性能有不错的提升， 但在创建索引时要小心。选择字段取决于使用的是什么SQL查询。

 **单列索引是基于单个列所建立的索引**

 
```
create index 索引名 on 表名(列名);
create index nameIndex on custor(name);
```
 
## 复合索引

 复合索引是基于两列或是多列的索引。在同一张表上可以有多个索引，但是要求列的组合必须不同

 
```
create index emp_idx1 on emp(ename, job);
```
 使用原则   
 1)、在大表上建立索引才有意义   
 2)、在where子句或是连接条件上经常引用的列上建立索引   
 3)、索引的层次不要超过4层

 **索引的缺点**   
 索引有一些先天不足：   
 1)、建立索引，系统要占用大约为表1.2倍的硬盘和内存空间来保存索引。   
 2)、更新数据的时候，系统必须要有额外的时间来同时对索引进行更新，以维持数据和索引的一致性。   
 实践表明，不恰当的索引不但于事无补，反而会降低系统性能。因为大量的索引在进行插入、修改和删除操作时比没有索引花费更多的系统时间。

 **显示索引信息**   
 1)、在同一张表上可以有多个索引，通过查询数据字典视图dba_indexs和user_indexs，可以显示索引信息。其中dba_indexs用于显示数据库所有的索引信息，而user_indexs用于显示当前用户的索引信息：

 
```
select index_name, index_type from user_indexes where table_name = '表名';
```
 **显示索引列**   
 通过查询数据字典视图user_ind_columns,可以显示索引对应的列的信息

 
```
select table_name, column_name from user_ind_columns where index_name ='IND_ENAME';
```
 也可以通过pl/sql developer工具查看索引信息

 **栗子：**   
 下面的SQL创建一个新的表名为CUSTOMERS，并增加了五列：

 
```
CREATE TABLE CUSTOMERS(
       ID   INT              NOT NULL,
       NAME VARCHAR (20)     NOT NULL,
       AGE  INT              NOT NULL,
       ADDRESS  CHAR (25) ,
       SALARY   DECIMAL (18, 2),       
       PRIMARY KEY (ID)
);
```
 现在，您可以创建单个或多个列索引使用以下语法：

 
```
CREATE INDEX index_name
    ON table_name ( column1, column2.....);
```
 要在AGE列上创建一个索引， 来优化客户搜索一个特定的年龄，以下是SQL语法：

 
```
CREATE INDEX idx_age
    ON CUSTOMERS ( AGE );
```
 删除索引约束：

 要删除索引的约束，使用下面的SQL：

 
```
ALTER TABLE CUSTOMERS
   DROP INDEX idx_age;
```
   
  
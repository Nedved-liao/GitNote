  ## 前言

 数据的完整性用于确保数据库数据遵从一定的商业和逻辑规则，在oracle中，数据完整性可以使用约束、触发器、应用程序（过程、函数）三种方法来实现，在这三种方法中，因为约束易于维护，并且具有最好的性能，所以作为维护数据完整性的首选

 但是约束会一定程度上较低数据库性能，有些规则直接在程序逻辑中处理就可以了，同时，也有可能在面对业务变更或是系统扩展时，数据库约束会使得处理不够方便

 总之，对于约束的选择无所谓合不合理，需要根据业务系统对于准确性和性能要求的侧重度来决定。

 
## 约束

 **数据库约束有五种：**   
 主键约束（PRIMARY KEY）   
 唯一性约束（UNIQUE)   
 非空约束（NOT NULL)   
 外键约束（FOREIGN KEY)   
 检查约束（CHECK)

 **not null(非空)**   
 如果在列上定义了not null，那么当插入数据时，必须为列提供数据

 **unique(唯一)**   
 当定义了唯一约束后，该列值是不能重复的，但是可以为null

 **primary key(主键)**   
 用于唯一的标示表行的数据，当定义主键约束后，该列不但不能重复而且不能为null

 需要说明的是：一张表最多只能有一个主键，但是可以有多个unqiue约束

 **foreign key(外键)**   
 用于定义主表和从表之间的关系。外键约束要定义在从表上，主表则必须具有主键约束或是unique 约束，当定义外键约束后，要求外键列数据必须在主表的主键列存在或是为null

 **check**   
 用于强制行数据必须满足的条件，假定在sal列上定义了check约束，并要求sal列值在1000-2000之间如果不在1000-2000之间就会提示出错。

 
```
SQL> create table goods(
   goodsId char(8) primary key,             //主键
   goodsName varchar2(30),
   unitprice number(10,2) check(unitprice>0),
   category varchar2(8),
   provider varchar2(30)
);
```
 
```
SQL> create table customer( 
   customerId char(8) primary key,              //主键
   name varchar2(50) not null,              //不为空
   address varchar2(50),
   email varchar2(50) ,                     //唯一
   sex char(2) default '男' check(sex in ('男','女')),     //一个char能存半个汉字，两位char能存一个汉字
   cardId char(18)
);
```
 
```
SQL> create table purchase( 
   customerId char(8) references customer(customerId),
   goodsId char(8) references goods(goodsId),
   nums number(10) check (nums between 1 and 30)
);
```
 表是默认建在SYSTEM表空间的

 如果在建表时忘记建立必要的约束，则可以在建表后使用alter table命令为表增加约束。但是要注意：增加not null约束时，需要使用modify选项，而增加其它四种约束使用add选项。

 SQL> alter table goods modify xxx not null;   
 SQL> alter table customer add constraint xxxunique();   
 SQL> alter table customer add constraint xxx check (address in ());

 **删除约束**   
 当不再需要某个约束时，可以删除。

 
```
alter table 表名 drop constraint 约束名称；
```
 特别说明一下：在删除主键约束的时候，可能有错误，比如：alter table 表名 drop primary key；这是因为如果在两张表存在主从关系，那么在删除主表的主键约束时，必须带上cascade选项 如像：alter table 表名 drop primary key cascade;

 **显示约束信息**   
 通过查询数据字典视图user_constraints，可以显示当前用户所有的约束的信息。   
 select constraint_name, constraint_type, status, validated from user_constraints where table_name = ‘表名’;   
 **显示约束列**   
 通过查询数据字典视图user_cons_columns，可以显示约束所对应的表列信息。   
 select column_name, position from user_cons_columns where constraint_name = ‘约束名’;

 
## 约束状态

 **数据库约束有两类状态**

 启用/禁用（enable/disable）：是否对新变更的数据启用约束验证

 验证/非验证 (validate/novalidate) ：是否对表中已客观存在的数据进行约束验证这两类四种状态从语法角度讲可以随意组合，默认是 enable validate

 **四类组合**

 enable validate :   
 默认的约束组合状态，无法添加违反约束的数据行，数据表中也不能存在违反约束的数据行；

 enable novalidate :   
 无法添加违反约束的数据行，但对已存在的违反约束的数据行不做验证；

 disable validate :   
 可以添加违反约束的数据行，但对已存在的违反约束的数据行会做约束验证（从描述中可以看出来，这本来就是一种相互矛盾的约束组合，只不过是语法上支持这种组合罢了，造成的结果就是会导致DML失败）

 disable novalidate :   
 可以添加违法约束的数据行，对已存在的违反约束的数据行也不做验证。

 我们需要上传大量违反非空约束的历史数据，可以临时将约束状态转为 disable novalidate，以保证这些不合要求的数据导入表中

 
```
SQL> alter table emp modify constraint emp_ename_nn disable novalidate;
```
 在数据导入完成之后，再将约束状态转为enable novalidate 以确保之后添加的数据不会再违反约束

 
```
SQL> alter table emp modify constraint emp_ename_nn enable novalidate;
```
   
  
默认情况下，oracle是启用数据库统计收集这项功能（AWR）

通过show parameter statistics_level来判断是否启用

```sql
SQL> show parameter statistics_level

NAME_COL_PLUS_SHOW_PARAM
------------------------------------------------------------------------------
TYPE
-----------
VALUE_COL_PLUS_SHOW_PARAM
------------------------------------------------------------------------------
statistics_level
string
TYPICAL

```



值为TYPICAL或者ALL表示启用AWR
值为BASIC，表示禁用AWR



当前连接实例的AWR报告提取：@?/rdbms/admin/awrrpt

以sysdba身份登录。

> 输入@?/rdbms/admin/awrrpt按提示,下一步

```bash
[oracle@pro-bas-dev-db ~]$ sqlplus / as sysdba

SQL*Plus: Release 11.2.0.4.0 Production on Fri Oct 25 16:06:38 2019

Copyright (c) 1982, 2013, Oracle.  All rights reserved.


Connected to:
Oracle Database 11g Enterprise Edition Release 11.2.0.4.0 - 64bit Production
With the Partitioning, OLAP, Data Mining and Real Application Testing options
#这里注意@?/rdbms/admin/awrrpt没有空格
SQL> @?/rdbms/admin/awrrpt

Current Instance
~~~~~~~~~~~~~~~~

   DB Id    DB Name	 Inst Num Instance
----------- ------------ -------- ------------
 1486159865 ORCL		1 orcl


Specify the Report Type
~~~~~~~~~~~~~~~~~~~~~~~
Would you like an HTML report, or a plain text report?
Enter 'html' for an HTML report, or 'text' for plain text
Defaults to 'html'
#注意这里直接回车即可，默认就是html格式的
Enter value for report_type: 

Type Specified:  html


Instances in this Workload Repository schema
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

   DB Id     Inst Num DB Name	   Instance	Host
------------ -------- ------------ ------------ ------------
* 1486159865	    1 ORCL	   orcl 	pro-bas-dev-
						db

Using 1486159865 for database Id
Using	       1 for instance number


Specify the number of days of snapshots to choose from
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Entering the number of days (n) will result in the most recent
(n) days of snapshots being listed.  Pressing <return> without
specifying a number lists all completed snapshots.

#注意这里根据实际需要选择几天的AWR报告，一般取最近的AWR报告选择1天即可
Enter value for num_days: 1

Listing the last day's Completed Snapshots

							Snap
Instance     DB Name	    Snap Id    Snap Started    Level
------------ ------------ --------- ------------------ -----
orcl	     ORCL	      17421 25 Oct 2019 00:00	   1
			      17422 25 Oct 2019 01:00	   1
			      17423 25 Oct 2019 02:00	   1
			      17424 25 Oct 2019 03:00	   1
			      17425 25 Oct 2019 04:00	   1
			      17426 25 Oct 2019 05:00	   1
			      17427 25 Oct 2019 06:00	   1
			      17428 25 Oct 2019 07:00	   1
			      17429 25 Oct 2019 08:00	   1
			      17430 25 Oct 2019 09:00	   1
			      17431 25 Oct 2019 10:00	   1
			      17432 25 Oct 2019 11:00	   1
			      17433 25 Oct 2019 12:00	   1
			      17434 25 Oct 2019 13:00	   1
			      17435 25 Oct 2019 14:00	   1
			      17436 25 Oct 2019 15:00	   1
			      17437 25 Oct 2019 16:00	   1

#这里选择时间,我要取14:00到15:00 就输入begin_snap为17435,end_snap为17436

Specify the Begin and End Snapshot Ids
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
#输入begin_snap为17435
Enter value for begin_snap: 17435
Begin Snapshot Id specified: 17435
#输入end_snap为17436
Enter value for end_snap: 17436
End   Snapshot Id specified: 17436



Specify the Report Name
~~~~~~~~~~~~~~~~~~~~~~~
The default report file name is awrrpt_1_17435_17436.html.  To use this name,
press <return> to continue, otherwise enter an alternative.
#报告名称选择默认
Enter value for report_name: 

Using the report name awrrpt_1_17435_17436.html

...

<p />
<br /><a class="awr" href="#top">Back to Top</a><p />
<p />
End of Report
</body></html>
Report written to awrrpt_1_17435_17436.html

#最后看到输出到awrrpt_1_17435_17436.html
```



在当前路径即可看到awrrpt_1_17435_17436.html

![1571991357851](C:\Users\Nedved\AppData\Roaming\Typora\typora-user-images\1571991357851.png)


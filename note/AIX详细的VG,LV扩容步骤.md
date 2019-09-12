# 解决方案

 1.先做重要数据备份

 2.进行文件系统扩容

 
# 步骤

 
## 1. df -g 查找出/u01 对应的VG卷 VOLUME GROUP: 

 
> # df -g  
>  Filesystem GB blocks Free %Used Iused %Iused Mounted on  
>  .
> 
>  .  
>  /dev/fslv00  60.00 3.95 94% 550567 36% /u01  
>  
> 
>  # lslv fslv00  
>  LOGICAL VOLUME: fslv00 VOLUME GROUP:  rootvg  
>  LV IDENTIFIER: 00f8d67e00004c000000014545274d99.13 PERMISSION: read/write  
>  VG STATE: active/complete LV STATE: opened/syncd  
>  TYPE: jfs2 WRITE VERIFY: off  
>  MAX LPs: 1024 PP SIZE: 128 megabyte(s)  
>  COPIES: 1 SCHED POLICY: parallel  
>  LPs: 480 PPs: 480  
>  STALE PPs: 0 BB POLICY: relocatable  
>  INTER-POLICY: minimum RELOCATABLE: yes  
>  INTRA-POLICY: middle UPPER BOUND: 32  
>  MOUNT POINT: /u01 LABEL: /u01  
>  MIRROR WRITE CONSISTENCY: on/ACTIVE   
>  EACH LP COPY ON A SEPARATE PV ?: yes   
>  Serialize IO ?: NO   
>  INFINITE RETRY: no 
> 
>  
 
## 2.查看rootvg的FREE PPs:

 
> # lsvg rootvg  
>  VOLUME GROUP: rootvg VG IDENTIFIER: 00f8d67e00004c000000014545274d99  
>  VG STATE: active  PP SIZE: 128 megabyte(s)  
>  VG PERMISSION: read/write TOTAL PPs: 799 (102272 megabytes)  
>  MAX LVs: 256 FREE PPs: 10 (1280 megabytes)  
>  LVs: 13 USED PPs: 789 (100992 megabytes)  
>  OPEN LVs: 12 QUORUM: 2 (Enabled)  
>  TOTAL PVs: 1 VG DESCRIPTORS: 2  
>  STALE PVs: 0 STALE PPs: 0  
>  ACTIVE PVs: 1 AUTO ON: yes  
>  MAX PPs per VG: 32512   
>  MAX PPs per PV: 1016 MAX PVs: 32  
>  LTG size (Dynamic): 256 kilobyte(s) AUTO SYNC: no  
>  HOT SPARE: no BB POLICY: relocatable  
>  PV RESTRICTION: none INFINITE RETRY: no
> 
>  
 PP SIZE(PP单位)

 FREE PPs(PP块数)

 FREE PPs: 10 (1280 megabytes) 可以看到只剩下1个G了

 
## 3.发现空间不足,进行磁盘扩展

 
### 3.1刷新磁盘(若果添加了物理磁盘,需要刷新一下)

 
> # cfgmgr
> 
>  
 cfgmgr命令详见

 [http://blog.sina.com.cn/s/blog_5a2405d10100lbxy.html](http://blog.sina.com.cn/s/blog_5a2405d10100lbxy.html) 

 
### 3.2查看磁盘使用情况

 
> # lspv  
>  hdisk0 00f8d67e45274d2c rootvg active   
>  hdisk1 none None   
>  hdisk2 none None   
>  hdisk3 none None   
>  hdisk4 none None   
>  hdisk5 none None   
>  hdisk6 none None   
>  hdisk7 none None   
>  hdisk8 none None   
>  hdisk9 none None   
>  hdisk10 none None   
>  hdisk11 none None   
>  hdisk12 none None   
>  hdisk13 none None   
>  hdisk14 none None   
>  hdisk15 none None   
>  hdisk16 none None   
>  hdisk17 none None   
>  hdisk18 00f8d67efa51a04f oggvg active   
>  hdisk19 00f8d67f77eb5875 None   
>  hdisk20 00f8d67e3106babe datavg active   
>  hdisk21 00f8d67ee9fbaac8 oggvg active   
>  hdisk22 00f8d67fa805f48a None   
>  hdisk23 none None   
>  .  
>  .
> 
>  
 注:查看磁盘空间大小 

 
> #bootinfo -s hdisk1
> 
>  
 
### 3.3进行扩展,发现报错

 
> >  #extendvg rootvg hdisk2 hdisk3  
>  0516-1254 extendvg: Changing the PVID in the ODM.  
>  0516-1254 extendvg: Changing the PVID in the ODM.  
>  0516-1162 extendvg: Warning, The Physical Partition Size of 128 requires the  
>  creation of 1093 partitions for hdisk2. The limitation for volume group  
>  rootvg is 1016 physical partitions per physical volume. Use chvg command  
>  with -t option to attempt to change the maximum Physical Partitions per  
>  Physical volume for this volume group.  
>  0516-1162 extendvg: Warning, The Physical Partition Size of 128 requires the  
>  creation of 1093 partitions for hdisk3. The limitation for volume group  
>  rootvg is 1016 physical partitions per physical volume. Use chvg command  
>  with -t option to attempt to change the maximum Physical Partitions per  
>  Physical volume for this volume group.  
>  0516-792 extendvg: Unable to extend volume group.
> 
>  #chdev -l hdisk2 -a pv=yes  
>  #chdev -l hdisk3 -a pv=yes
> 
>  >  #chvg -t rootvg  
>  0516-1187 chvg: Volume group not changed. Either the volume group rootvg  
>  has the specified factor value or is at proper maximum physical partitions  
>  per physical volume limit already.   
>  0516-732 chvg: Unable to change volume group rootvg.
> 
>  >  //因为PP数目已经达到上限
> 
>  
 
### 3.4刷新pp 

 
> #chvg -t 2 rootvg   
>  0516-1164 chvg: Volume group rootvg changed. With given characteristics rootvg  
>  can include upto 16 physical volumes with 2032 physical partitions each.
> 
>  //数字2表示2×1016个PP
> 
>  
 
### 3.5再次进行扩展

 
> #extendvg rootvg hdisk2 hdisk3
> 
>  
 
### 3.6查看磁盘使用情况,发现已经在使用

 
> #lspv  
>  hdisk0 0007df58fb3e71d4 rootvg active  
>  hdisk1 00009d03638bf367 rootvg active  
>  hdisk2 00009d034caa708d rootvg active  
>  hdisk3 00009d034caa7219 rootvg active
> 
>  
 
### 3.7查看vg卷的情况,确认已经扩容

 
> #lsvg rootvg  
>  VOLUME GROUP: rootvg VG IDENTIFIER: 0007df580000d70000000113fb3e72ce  
>  VG STATE: active PP SIZE: 128 megabyte(s)  
>  VG PERMISSION: read/write TOTAL PPs: 3278 (419584 megabytes)  
>  MAX LVs: 256  FREE PPs: 2208 (282624 megabytes)  
>  LVs: 60 USED PPs: 1070 (136960 megabytes)  
>  OPEN LVs: 27 QUORUM: 1  
>  TOTAL PVs: 4 VG DESCRIPTORS: 4  
>  STALE PVs: 0 STALE PPs: 0  
>  ACTIVE PVs: 4 AUTO ON: yes  
>  MAX PPs per VG: 32512   
>  MAX PPs per PV: 2032 MAX PVs: 16  
>  LTG size (Dynamic): 256 kilobyte(s) AUTO SYNC: no  
>  HOT SPARE: no BB POLICY: relocatable
> 
>  
 
## 4.扩容完毕,进行文件系统扩容

 
> #chfs -a size=+500G /u01
> 
>  0516-787 extendlv: Maximum allocation for logical volume /u01 is 1145.
> 
>  
 注：+代表在原来基础上增加500G大小  
   
 从这个提示可以看到逻辑卷lv2已经到达最大的pp扩展数

 
> # lslv fslv00  
>  LOGICAL VOLUME: fslv00 VOLUME GROUP:  rootvg  
>  LV IDENTIFIER: 00f8d67e00004c000000014545274d99.13 PERMISSION: read/write  
>  VG STATE: active/complete LV STATE: opened/syncd  
>  TYPE: jfs2 WRITE VERIFY: off  
> MAX LPs: 1024  PP SIZE: 128 megabyte(s)  
>  COPIES: 1 SCHED POLICY: parallel  
>  LPs: 480 PPs: 480  
>  STALE PPs: 0 BB POLICY: relocatable  
>  INTER-POLICY: minimum RELOCATABLE: yes  
>  INTRA-POLICY: middle UPPER BOUND: 32  
>  MOUNT POINT: /u01 LABEL: /u01  
>  MIRROR WRITE CONSISTENCY: on/ACTIVE   
>  EACH LP COPY ON A SEPARATE PV ?: yes   
>  Serialize IO ?: NO   
>  INFINITE RETRY: no 
> 
>  
 已经超过max值了

 重新调整pp的max值

 
> # smit chlv  
>  Change a Logical Volume
> 
>  Move cursor to desired item and press Enter.
> 
>  Change a Logical Volume  
>  Rename a Logical Volume 
> 
>  
> 
>  >  
> 
>  Change a Logical Volume
> 
>  Type or select a value for the entry field.  
>  Press Enter AFTER making all desired changes.
> 
>  [Entry Fields]  
>  * LOGICAL VOLUME name [fslv00(填写正确的lv名字)]
> 
>  
> 
>  
> 
>  
> 
>  Change a Logical Volume
> 
>  Type or select values in entry fields.  
>  Press Enter AFTER making all desired changes.  
>    
>  [Entry Fields]  
>  * Logical volume NAME fslv00  
>  Logical volume TYPE [jfs2] +  
>  POSITION on physical volume middle +  
>  RANGE of physical volumes minimum +  
>  MAXIMUM NUMBER of PHYSICAL VOLUMES [32] #  
>  to use for allocation  
>  Allocate each logical partition copy yes +  
>  on a SEPARATE physical volume?  
>  RELOCATE the logical volume during yes +  
>  reorganization?  
>  Logical volume LABEL [/u01]  
>  MAXIMUM NUMBER of LOGICAL PARTITIONS [1024(更改)]  #  
>  SCHEDULING POLICY for writing/reading parallel +  
>  logical partition copies  
>  PERMISSIONS read/write +  
>  Enable BAD BLOCK relocation? yes +  
>  Enable WRITE VERIFY? no +  
>  Mirror Write Consistency? active +  
>  Serialize IO? no +  
>  Mirror Pool for First Copy +  
>  Mirror Pool for Second Copy +  
>  Mirror Pool for Third Copy +  
>  Infinite Retry Option no 
> 
>  
 回车保存,退出

 再次进行扩展文件系统

 
> #chfs -a size=+500G /u01
> 
>  
 没有报错,扩容完毕

 df -g 再次检查就行了

 

 

   
   
   
   
   
 
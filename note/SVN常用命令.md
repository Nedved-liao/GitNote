---
title: SVN常用命令
date: 2018-10-23 18:47:37
tags: CSDN迁移
---
   常用的svn命令

 

 
# 1、add 

 往版本库中添加新的文件

 
> [root@78778e06dc0a test]# ls  
>  t1 test.1 test.2 test.3  
>  [root@78778e06dc0a test]# svn add t1  
>  A t1  
>  [root@78778e06dc0a test]# svn add test.*  
>  A test.1  
>  A test.2  
>  A test.3
> 
>  
 
## 2、cat 

 输出指定文件或URL的内容。  
   
 svn cat 目标[@版本]...如果指定了版本，将从指定的版本开始查找。 (PREV 是上一版本,也可以写具体版本号,这样输出结果是可以提交的)

 
> >  [root@78778e06dc0a test]# svn cat t1  
>  Hello World  
>  [root@78778e06dc0a test]# svn cat t1 > t2  
>  [root@78778e06dc0a test]# cat t2  
>  Hello World  
>  [root@78778e06dc0a test]# svn cat -r PREV t1  
>  Hello World
> 
>  
 

 
# 3、checkout (co) 

 将文件checkout到本地目录  
 svn checkout path（path是服务器上的目录）

 
> >  [root@78778e06dc0a ~]# svn co svn://127.0.0.1:99/No-1  
>  A No-1/test  
>  A No-1/test/t1  
>  A No-1/test/test.1  
>  A No-1/test/test.2  
>  A No-1/test/test.3  
>  Checked out revision 10.  
>  [root@78778e06dc0a ~]# ls  
>  anaconda-ks.cfg No-1
> 
>  
 
# 4、commit (ci)

 将改动的文件提交到版本库  
   
 svn commit -m "LogMessage" [-N] [--no-unlock] PATH(如果选择了保持锁，就使用--no-unlock开关)

 
> >  [root@78778e06dc0a test]# touch ci  
>  [root@78778e06dc0a test]# svn add ci  
>  A ci  
>  [root@78778e06dc0a test]# svn ci -m 'add the file ci'  
>  Adding ci  
>  Transmitting file data .  
>  Committed revision 11.
> 
>  
 
# 5、delete (del, remove, rm)

 删除文件  
   
 svn delete path -m "delete test fle"

 
> >  [root@78778e06dc0a test]# svn del ci  
>  D ci  
>  [root@78778e06dc0a test]# svn ci -m 'delete the ci file'  
>  Deleting ci
> 
>  Committed revision 12.
> 
>  
 
# 6、diff (di)

 比较差异  
   
 svn diff path(将修改的文件与基础版本比较)  
 svn diff -r m:n path(对版本m和版本n比较差异)

 
> >  [root@78778e06dc0a test]# svn di -r 9:10 t1  
>  Index: t1  
>  ===================================================================  
>  --- t1 (revision 9)  
>  +++ t1 (revision 10)  
>  @@ -1 +1,2 @@  
>  Hello World  
>  +V2
> 
>  
 
# 7、info

 查看文件详细信息  
   
 svn info path

 
> >  [root@78778e06dc0a test]# svn info t1  
>  Path: t1  
>  Name: t1  
>  Working Copy Root Path: /root/No-1  
>  URL: svn://127.0.0.1:99/No-1/test/t1  
>  Repository Root: svn://127.0.0.1:99/No-1  
>  Repository UUID: 38f02c43-b084-41d5-9250-07068e50e63f  
>  Revision: 11  
>  Node Kind: file  
>  Schedule: normal  
>  Last Changed Author: First  
>  Last Changed Rev: 10  
>  Last Changed Date: 2018-10-23 09:58:47 +0000 (Tue, 23 Oct 2018)  
>  Text Last Updated: 2018-10-23 10:13:01 +0000 (Tue, 23 Oct 2018)  
>  Checksum: fc10817f046ecaad956e264d5ed35424d52c9b55
> 
>  
 
# 8、list (ls)

 版本库下的文件和目录列表  
   
 svn list path

 
> >  [root@78778e06dc0a test]# svn ls  
>  t1  
>  test.1  
>  test.2  
>  test.3
> 
>  
 
# 9、lock/unlock

 加锁/解锁  
   
 svn lock -m "LockMessage" [--force] PATH

 
> >  [root@78778e06dc0a test]# svn lock -m 'test' t1  
>  't1' locked by user 'First'.  
>  [root@78778e06dc0a test]# svn unlock t1  
>  't1' unlocked.
> 
>  
 
# 10、log

 查看日志  
   
 svn log path

 
> >  [root@78778e06dc0a test]# svn log t1  
>  ------------------------------------------------------------------------  
>  r10 | First | 2018-10-23 09:58:47 +0000 (Tue, 23 Oct 2018) | 1 line
> 
>  v2  
>  ------------------------------------------------------------------------  
>  r9 | First | 2018-10-23 09:57:30 +0000 (Tue, 23 Oct 2018) | 1 line
> 
>  123  
>  ------------------------------------------------------------------------
> 
>  
 
# 11、merge

 将两个版本之间的差异合并到当前文件  
   
 svn merge -r m:n path

 
> >  [root@78778e06dc0a test]# svn merge -r 9:10 t1
> 
>  
 （差异合并到当前文件，但是一般都会产生冲突，需要处理一下）  
 

 
# 12、mkdir

 创建纳入版本控制下的新目录  
   
 svn mkdir: 创建纳入版本控制下的新目录。  
 用法: 1、mkdir PATH...  
 2、mkdir URL...

 
> >  [root@78778e06dc0a test]# svn mkdir tmp  
>  A tmp  
>  [root@78778e06dc0a test]# svn ls  
>  t1  
>  test.1  
>  test.2  
>  test.3  
>  [root@78778e06dc0a test]# svn ci -m 'mkdir the tmp file'  
>  Adding tmp
> 
>  Committed revision 13.  
>  [root@78778e06dc0a test]# svn up  
>  Updating '.':  
>  At revision 13.  
>  [root@78778e06dc0a test]# ls  
>  t1 test.1 test.2 test.3 tmp
> 
>  
 13、更新到某个版本  
   
 svn update -r m path  
 例如：  
 svn update如果后面没有目录，默认将当前目录以及子目录下的所有文件都更新到最新版本。  
 svn update -r 200 test.php(将版本库中的文件test.php还原到版本200)  
 svn update test.php(更新，于版本库同步。如果在提交的时候提示过期的话，是因为冲突，需要先update，修改文件，然后清除svn resolved，最后再提交commit)

 [root@78778e06dc0a test]# svn up  
 Updating '.':  
 At revision 13.

 14、resolved  
 解决冲突  
   
 svn resolved: 移除工作副本的目录或文件的“冲突”状态。  
 用法: resolved PATH...  
 注意: 本子命令不会依语法来解决冲突或是移除冲突标记；它只是移除冲突的  
 相关文件，然后让 PATH 可以再次提交。

 15、status  
 查看文件或者目录状态  
   
 1）svn status path（目录下的文件和子目录的状态，正常状态不显示）  
 【?：不在svn的控制中；M：内容被修改；C：发生冲突；A：预定加入到版本库；K：被锁定】  
 2）svn status -v path(显示文件和子目录状态)  
 第一列保持相同，第二列显示工作版本号，第三和第四列显示最后一次修改的版本号和修改人。  
 注：svn status、svn diff和 svn revert这三条命令在没有网络的情况下也可以执行的，原因是svn在本地的.svn中保留了本地版本的原始拷贝。  
 简写：svn st

 16、switch  
 代码库URL变更  
   
 svn switch (sw): 更新工作副本至不同的URL。  
 用法: 1、switch URL [PATH]  
 2、switch --relocate FROM TO [PATH...]  
   
 1、更新你的工作副本，映射到一个新的URL，其行为跟“svn update”很像，也会将  
 服务器上文件与本地文件合并。这是将工作副本对应到同一仓库中某个分支或者标记的  
 方法。  
 2、改写工作副本的URL元数据，以反映单纯的URL上的改变。当仓库的根URL变动  
 (比如方案名或是主机名称变动)，但是工作副本仍旧对映到同一仓库的同一目录时使用  
 这个命令更新工作副本与仓库的对应关系。  
 

 [参考 https://blog.csdn.net/yangzhongxuan/article/details/7018168?utm_source=blogxgwz0](https://blog.csdn.net/yangzhongxuan/article/details/7018168?utm_source=blogxgwz0)

   
   
   
   
   
   
   
 
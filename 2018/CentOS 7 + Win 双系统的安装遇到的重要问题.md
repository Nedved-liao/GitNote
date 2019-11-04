  前言：对于刚学linux的朋友们，多多小小因为各种原因需要装双系统，亦或者爱好使然。多数是问题解决,第一次装系统者不推荐看….   
 那么现在内德在此就说说在本本上装双系统会遇到的问题及其解决方法。

 
## 环境准备

 1.镜像（没镜像？官网呗）   
 2.U盘启动 用软碟通吧（没插件）   
 3.引导（双引导）   
 4.分区划分

 
## go go go

 环境:   
 在为window把你的硬盘给格式化,根据你要给centos多小空间格式.   
 window自带有管理磁盘工具无需额外下载,   
 此电脑—->管理—>磁盘划分(记得要备份好数据)

 1.建议是在装好window下再装centos，当然反过来也OK就是分区较复杂（所以不推荐centos—>window）

 制作启动盘百度上有就不说了

 上重点

 
## 引导

 1.BIOS引导   
 首先进入BIOS，老规矩开机前一顿狂按Esc，F8，F11……等等就看你的机子了，一般都是Esc，HP是F8。   
 要是不知道如何进入的就按自己的机子品牌百度哈

 进入BOS后就找到Boot一项，   
 ![这里写图片描述](https://img-blog.csdn.net/20171114195140442?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvTmVkdmVkX0w=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

 华硕好像是直接跳出来的。。

 U盘启动了 就可以看到centos引导菜单了   
 ![这里写图片描述](https://img-blog.csdn.net/20171114201017069?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvTmVkdmVkX0w=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

 
## 重点来了 重点来了

 别太高兴一股劲选择安装啊！

 接下来我问要做的是进行第二次引导   
 因为centos不会自动指向U盘，所以需要我们手动指向，不然找不到安装文件。

 将光标移到第一行，然后这里不是直接点Install CentOS7，要按Tab键或按e先配置CentOS镜像位置。   
 按下Tab或e之后可以看到一下三行英文：

 找到   
 limuze /image/vmlinuz inst.stage2=hd:LABEL=CentOS\x207\x20x86_64 quiet   
 **_initrdefi /image/pxeboot/initrd.img_**   
 把内德加粗的这一段改成：   
 limuze /image/vmlinuz initrd=initrd.img linux dd quiet

 然后就 ctrl + x执行就可以看到所有盘符和编号了   
 然后就找到你的U盘名称   
 ![这里写图片描述](https://img-blog.csdn.net/20171114203052828?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvTmVkdmVkX0w=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)   
 找到后要记住哦，内德的是sdb4

 记住这个sdb4，然后重新再来进入到引导菜单。   
 这次就还是将光标移到第一行，然后按e或tab   
 这次改成这样就可以指向U盘了   
 limuze /image/vmlinuz inst.stage2=hd:/dev/sdb4 quiet

 后面就是熟悉的图形界面了   
 ![这里写图片描述](https://img-blog.csdn.net/20171114203649091?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvTmVkdmVkX0w=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

 这里就按照在为window上划分的分区进行选择了

 
## 第二个重点来了

 分区划分   
 ![这里写图片描述](https://img-blog.csdn.net/20171114203942156?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvTmVkdmVkX0w=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)   
 不推荐用标准分区划分，因为有些机子会检测错误。   
 所以就LVM格式，所以呢我们就可以点击自动创建分区了。   
 创建分区后在根据你所需要的更改下大小和名称就OK了

 
## 必看提示：

 1.划分分区不要window上的 数据给干掉哦，window的分区可能会检测为未知的分区，所以不要看到一有未知的东西就像把Ta给干掉。   
 2.centos安装后回到window有些机子不会把centos的硬盘显示出来，但有些会显示哦，所以还是那句话不要随便干掉。   
 3.centos 一般会有两个boot，一个是centos自身的引导，另一个是启动引导。这个一般会在window显示为没有格式化的2G磁盘，还是那句话不要干掉Ta

 这些问题解决后，装过系统的人都OK了。第一次装系统的或许会看不明白，可以百度详细了解过程，内德仅提供新问题解决方法和注意事项.

   
  
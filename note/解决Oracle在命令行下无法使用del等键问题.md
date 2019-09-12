  前言：   
 Oracle使用Linux命令行进行编辑？   
 有PL/SQL development，SQL development等工具，为何用Linux命令行？   
 但也免不了有用的的时候

 以下是解决在Linux命令行无法使用快捷键的方法

 rlwrap 可用来支持oracle下sqlplus历史命令的回调功能，提高效率。

 1.安装以下软件   
 readline   
 ncurses-devel   
 readline-devel   
 compat-libtermcap   
 compat-readline   
 rlwrap

 使用yum或者在网上搜索相对应的RPM包安装

 
```
yum install readline readline-devel  ncurses-devel  compat-libtermcap compat-readline rlwrap -y
```
 2.修改 /home/u11/.bash_profile 

 在最后添加如下代码，以换用rlwrap

 
```
alias sqlplus="rlwrap sqlplus"
alias rman="rlwrap rman"
```
 再次登录sqlplus就可以使用快捷键了

   
  
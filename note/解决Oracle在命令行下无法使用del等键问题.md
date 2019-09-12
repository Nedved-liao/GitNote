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

   
  
   # 事件背景

 服务应用weblogic通过nohup启动.

 nohup的使用全部都在weblogic域中的bin目录下

 但是没有做定期nohup.out的清理  
 导致核心服务的日志过大,在出现问题时候难以打开日志进行问题定位

 
# 处理方法

 crontab 加脚本每天凌晨清理备份一次nohup日志

 脚本如下

 
> #!/bin/bash
> 
>  #ENV  
>  DATE=$(date +"%m-%d")  
>  BASE=$(awk -F"'" '/gobin/{print $2}' ~/.bashrc|awk '{print $2}')  
>  FILE=$BASE'log'  
>  LOG=$BASE'nohup.out'
> 
>  >  #Check if the log file exist  
>  if [ ! -d $FILE ];then  
>  mkdir $FILE  
>  fi
> 
>  #Backup the log to the FILE  
>  echo "$DATE"  
>  echo '#Back up the log file#'  
>  cp $LOG $FILE/$DATE.log
> 
>  #Clean today's log  
>  echo 'Start to clear..'  
>  cp /dev/null $LOG
> 
>  #Delete files from seven days ago  
>  find $FILE -type f -mtime +7 -name "*.log" -exec rm {} \;  
>  echo '#Finished.#'
> 
>  
 脚本ENV取当前变量说明

 目的脚本要适应多台服务器中的weblogic域路径,

 利用weblogic用户下的alias(里面在部署项目时候设置有域的bin路径)

 
> cat ~/.bashrc  
>  # .bashrc
> 
>  # Source global definitions  
>  if [ -f /etc/bashrc ]; then  
>  . /etc/bashrc  
>  fi
> 
>  **...**
> 
>  >  export PS1="<\! `hostname` [`whoami`] :"'$PWD'">"  
>  alias gobin='cd /home/weblogic/bea/user_projects/domains/pro_domain/bin/'
> 
>  > **...**
> 
>  
> 
>  # User specific aliases and functions
> 
>  
 截取里面域的路径

 
> BASE=$(awk -F"'" '/gobin/{print $2}' ~/.bashrc|awk '{print $2}')
> 
>  
 

 通过自动获取域的路径,同时通过ansible批发到核心服务当中.

 再配合crontab实现核心服务器日志的定期清理

 

 

  
   
   
 
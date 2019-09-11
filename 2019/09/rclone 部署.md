# 在Linux上使用rclone挂载Google Drive 和 Onedrive

> rclone可以帮助我们在Linux上挂载一些储存服务,包括Google drive, onedrive, box, AWS S3等等.同时不会占用硬盘空间
>简直就是小容量vps的福音

## 1.0 安装 Rclone
>参考[rclone Linux 安装描述](https://rclone.org/install/)

### 1.1 稳定版安装
Script installation
To install rclone on Linux/macOS/BSD systems, run:
```bash
curl https://rclone.org/install.sh | sudo bash
```
### 1.2 Beta安装
For beta installation, run:
```bash
curl https://rclone.org/install.sh | sudo bash -s beta
```

### 1.3 源码安装

从预编译二进制文件安装Linux 获取并解压缩
Linux installation from precompiled binary
Fetch and unpack
```bash
curl -O https://downloads.rclone.org/rclone-current-linux-amd64.zip
unzip rclone-current-linux-amd64.zip
cd rclone-*-linux-amd64
```
复制bin文件到/usr/bin/
Copy binary file
```bash
sudo cp rclone /usr/bin/
sudo chown root:root /usr/bin/rclone
sudo chmod 755 /usr/bin/rclone
```
安装对应的man帮助
Install manpage
```bash
sudo mkdir -p /usr/local/share/man/man1
sudo cp rclone.1 /usr/local/share/man/man1/
sudo mandb 
```

Note:
>“ 官方同时给出了docker方式部署,利用docker -v 挂载到OS上。个人感觉意义不大就介绍了


## 2.0 本地PC 上下载rclone (可选)

为什么要在本地win10 上下载rclone ?
若果你的VPS 可以开启图形化的可以忽略这个步骤

主要问题是客户端授权,在运行rclone需要先授权。授权其中会跳转到浏览器，google drive 授权是给出网址,可以直接复制到chrome上进行授权获取认证,但是onedrive 通常是给出的网址是http://127.0.0.1:53682/auth 这个很操蛋vps没有图形化就不无法解决了

所以直接利用本地pc  win10上获取auth就省事多了.

### 2.1 在本地window下载rclone
https://downloads.rclone.org/rclone-current-windows-amd64.zip

解压出来后,进入cmd，输入
```bash
rclone authorize "onedrive"
```

之后会弹出窗口认证，然后复制token 记录好,后面在linux 上会用到
```bash
Paste the following into your remote machine --->
{"access_token":"xxxx"}  #请复制{xx}整个内容(包括花括号)
<---End paste
```


## 3.0 初始化配置rclone
回到vps上

执行rclone config 配置google drive

```bash
[root@lab-test ~]# rclone config
2019/09/11 10:18:28 NOTICE: Config file "/root/.config/rclone/rclone.conf" not found - using defaults
No remotes found - make a new one
n) New remote
s) Set configuration password
q) Quit config
n/s/q> n  #选择新建一个配置
name> test #配置名称
Type of storage to configure.
Enter a string value. Press Enter for the default ("").
Choose a number from below, or type in your own value
 1 / 1Fichier
   \ "fichier"
 2 / Alias for an existing remote
   \ "alias"
 3 / Amazon Drive
   \ "amazon cloud drive"
 4 / Amazon S3 Compliant Storage Provider (AWS, Alibaba, Ceph, Digital Ocean, Dreamhost, IBM COS, Minio, etc)
   \ "s3"
 5 / Backblaze B2
   \ "b2"
 6 / Box
   \ "box"
 7 / Cache a remote
   \ "cache"
 8 / Dropbox
   \ "dropbox"
 9 / Encrypt/Decrypt a remote
   \ "crypt"
10 / FTP Connection
   \ "ftp"
11 / Google Cloud Storage (this is not Google Drive)
   \ "google cloud storage"
12 / Google Drive
   \ "drive"
13 / Google Photos
   \ "google photos"
14 / Hubic
   \ "hubic"
15 / JottaCloud
   \ "jottacloud"
16 / Koofr
   \ "koofr"
17 / Local Disk
   \ "local"
18 / Mega
   \ "mega"
19 / Microsoft Azure Blob Storage
   \ "azureblob"
20 / Microsoft OneDrive
   \ "onedrive"
21 / OpenDrive
   \ "opendrive"
22 / Openstack Swift (Rackspace Cloud Files, Memset Memstore, OVH)
   \ "swift"
23 / Pcloud
   \ "pcloud"
24 / Put.io
   \ "putio"
25 / QingCloud Object Storage
   \ "qingstor"
26 / SSH/SFTP Connection
   \ "sftp"
27 / Union merges the contents of several remotes
   \ "union"
28 / Webdav
   \ "webdav"
29 / Yandex Disk
   \ "yandex"
30 / http Connection
   \ "http"
31 / premiumize.me
   \ "premiumizeme"
Storage> 12 #选择对应的标号 这里选择的是Google Drive
** See help for drive backend at: https://rclone.org/drive/ **

Google Application Client Id
Setting your own is recommended.
See https://rclone.org/drive/#making-your-own-client-id for how to create your own.
If you leave this blank, it will use an internal key which is low performance.
Enter a string value. Press Enter for the default ("").
client_id> #一般留空
Google Application Client Secret
Setting your own is recommended.
Enter a string value. Press Enter for the default ("").
client_secret> #一般留空
Scope that rclone should use when requesting access from drive.
Enter a string value. Press Enter for the default ("").
Choose a number from below, or type in your own value
 1 / Full access all files, excluding Application Data Folder.
   \ "drive"
 2 / Read-only access to file metadata and file contents.
   \ "drive.readonly"
   / Access to files created by rclone only.
 3 | These are visible in the drive website.
   | File authorization is revoked when the user deauthorizes the app.
   \ "drive.file"
   / Allows read and write access to the Application Data folder.
 4 | This is not visible in the drive website.
   \ "drive.appfolder"
   / Allows read-only access to file metadata but
 5 | does not allow any access to read or download file content.
   \ "drive.metadata.readonly"
scope> 1 #选择完全访问所有文件
ID of the root folder
Leave blank normally.
Fill in to access "Computers" folders. (see docs).
Enter a string value. Press Enter for the default ("").
root_folder_id> #一般留空
Service Account Credentials JSON file path
Leave blank normally.
Needed only if you want use SA instead of interactive login.
Enter a string value. Press Enter for the default ("").
service_account_file> #一般留空
Edit advanced config? (y/n)
y) Yes
n) No
y/n> n #是否修改客户端配置,没有特殊就不改了
Remote config
Use auto config?
 * Say Y if not sure
 * Say N if you are working on a remote or headless machine
y) Yes
n) No
y/n> n #是否使用自动配置,这里选择n
If your browser doesn't open automatically go to the following link: https://accounts.google.com/o/oauth2/auth?access_type=offline&client_id=202264815644.apps.googleusercontent.com&redirect_uri=urn%3Aietf%3Awg%3Aoauth%3A2.0%3Aoob&response_type=code&scope=https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fdrive&state=cdadc76b05c67ecf139a03d81e3c8e2c
Log in and authorize rclone for access 
Enter verification code> #复制上面给出的认证地址到chrome,得到的认证码粘贴到这里

Configure this as a team drive?
y) Yes
n) No
y/n> n #不用team drive 选择n
--------------------
[remote]
client_id = 
client_secret = 
scope = drive
root_folder_id = 
service_account_file =
token = {"access_token":"XXX","token_type":"Bearer","refresh_token":"XXX","expiry":"2014-03-16T13:57:58.955387075Z"}
--------------------
y) Yes this is OK
e) Edit this remote
d) Delete this remote
y/e/d> y #确认配置

```

完成后选择退出

这时候可以使用再次使用rclone config就可以看到刚新建的配置名称
```bash
[root@lab-test ~]# rclone config
Current remotes:

Name                 Type
====                 ====
test                 drive

e) Edit existing remote
n) New remote
d) Delete remote
r) Rename remote
c) Copy remote
s) Set configuration password
q) Quit config
e/n/d/r/c/s/q>

```

> onedrive 是基本一样的,就是在给出认证链接时候给的是http://127.0.0.1:53682/auth。
> 这时候就直接把在window 上获得的认证粘贴到Enter verification code> 步骤就行了

## 4.0 挂载
```bash
# 先新建一个目录,用于挂载
mkdir -p /mnt/google-drive-test
# 挂载命令
rclone mount test: /mnt/google-drive-test --allow-other --allow-non-empty --vfs-cache-mode writes &
# df 查看
df -h
Filesystem      Size  Used Avail Use% Mounted on
udev            483M     0  483M   0% /dev
tmpfs            99M   13M   87M  13% /run
/dev/sda1       9.8G  7.6G  1.8G  82% /
tmpfs           495M     0  495M   0% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           495M     0  495M   0% /sys/fs/cgroup
test:            7EB     0   7EB   0% /mnt/google-drive-test
# 使用dd 测速一下
dd if=/dev/zero of=/mnt/google-drive-test/1  bs=2048 count=1k
记录了1024+0 的读入
记录了1024+0 的写出
2097152字节(2.1 MB)已复制，1.88323 秒，1.1 MB/秒
```
1.1MB/s 速度还是不错的

Note:
> " 这里需要注意的挂载时候命令默认是前台展示,退出就卸载了.
使用 rclone mount --help 可以看到
When the program ends, either via Ctrl+C or receiving a SIGINT or SIGTERM signal,
the mount is automatically stopped.
所以使用使用还得加个& 掉到后台

## 5.0 卸载
卸载很简单就是直接umount 掉就行了
```bash
#卸载
umount /mnt/google-drive-test
#确认是否已卸载
df -h
Filesystem      Size  Used Avail Use% Mounted on
udev            483M     0  483M   0% /dev
tmpfs            99M   13M   87M  13% /run
/dev/sda1       9.8G  7.6G  1.8G  82% /
tmpfs           495M     0  495M   0% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           495M     0  495M   0% /sys/fs/cgroup
```

## 6.0 设置启动脚本

这里提供一个类似server 启动的脚本
```bash
#!/bin/bash

PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
export PATH
NAME_BIN="rclone"
### BEGIN INIT INFO
# Provides:          rclone
# Required-Start:    $all
# Required-Stop:     $all
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: Start rclone at boot time
# Description:       Enable rclone by daemon.
### END INIT INFO

NAME="OneDrive" #rclone name名
REMOTE='Download' #远程文件夹
LOCAL='/home/OneDrive' #挂载地址

Green_font_prefix="\033[32m" && Red_font_prefix="\033[31m" && Green_background_prefix="\033[42;37m" && Red_background_prefix="\033[41;37m" && Font_color_suffix="\033[0m"
Info="${Green_font_prefix}[信息]${Font_color_suffix}"
Error="${Red_font_prefix}[错误]${Font_color_suffix}"
RETVAL=0

check_running(){
        PID="$(ps -C $NAME_BIN -o pid= |head -n1 |grep -o '[0-9]\{1,\}')"
        if [[ ! -z ${PID} ]]; then
                return 0
        else
                return 1
        fi
}
do_start(){
        check_running
        if [[ $? -eq 0 ]]; then
                echo -e "${Info} $NAME_BIN (PID ${PID}) 正在运行..." && exit 0
        else
                fusermount -zuq $LOCAL >/dev/null 2>&1
                mkdir -p $LOCAL
                sudo /usr/bin/rclone mount $NAME:$REMOTE $LOCAL --copy-links --no-gzip-encoding --no-check-certificate --allow-other --allow-non-empty --umask 000 >/dev/null 2>&1 &
                sleep 2s
                check_running
                if [[ $? -eq 0 ]]; then
                        echo -e "${Info} $NAME_BIN 启动成功 !"
                else
                        echo -e "${Error} $NAME_BIN 启动失败 !"
                fi
        fi
}
do_stop(){
        check_running
        if [[ $? -eq 0 ]]; then
                kill -9 ${PID}
                RETVAL=$?
                if [[ $RETVAL -eq 0 ]]; then
                        echo -e "${Info} $NAME_BIN 停止成功 !"
                else
                        echo -e "${Error} $NAME_BIN 停止失败 !"
                fi
        else
                echo -e "${Info} $NAME_BIN 未运行"
                RETVAL=1
        fi
        fusermount -zuq $LOCAL >/dev/null 2>&1
}
do_status(){
        check_running
        if [[ $? -eq 0 ]]; then
                echo -e "${Info} $NAME_BIN (PID $(echo ${PID})) 正在运行..."
        else
                echo -e "${Info} $NAME_BIN 未运行 !"
                RETVAL=1
        fi
}
do_restart(){
        do_stop
        do_start
}
case "$1" in
        start|stop|restart|status)
        do_$1
        ;;
        *)
        echo "使用方法: $0 { start | stop | restart | status }"
        RETVAL=1
        ;;
esac
exit $RETVAL

```
此脚本出自
https://raw.githubusercontent.com/x91270/Centos/master/rcloned

>使用时候先改好里面的变量
NAME="OneDrive" #rclone name名
REMOTE='Download' #远程文件夹
LOCAL='/home/OneDrive' #挂载地址

但是只能对应挂载一个盘,当我需要挂载几个google drive 或者同时挂载onedrive 时候就麻烦极了
尽是些花里胡哨的玩意
当对应多盘的时候直接编写一个脚本到/usr/bin下方便多了
```bash
cat /usr/bin/rclone-server
rclone mount test: /mnt/google-drive-test --allow-other --allow-non-empty --vfs-cache-mode writes &
rclone mount test1: /mnt/google-drive-test1 --allow-other --allow-non-empty --vfs-cache-mode writes &
```
添加时候直接往里面加就完事,不用的时候就直接umount.


## 总结
如果手里有大容量的Google Drive或者OneDrive可以使用rclone挂载到VPS上,将不常用的文件丢进去可以很大程度上节约空间。当然要注意的是在保存重要文件的的盘不要翻车哦
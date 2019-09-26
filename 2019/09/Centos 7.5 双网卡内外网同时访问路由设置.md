# [Centos 7.5 双网卡内外网同时访问路由设置](https://www.cnblogs.com/xuefy/p/11344086.html)



说明：服务器有两张网卡分别是eth0、eth1，eth0配置内网IP：192.168.1.1/24，eth1配置外网IP：10.1.1.1/24；要求192.168.0.0/16网段走网卡eth0，网关是192.168.1.254，其余网段走网卡eth1，网关是10.1.1.254;
一、配置网卡IP
vi /etc/sysconfig/network-scripts/ifcfg-eth0 //配置网卡1

NAME="eth0"
ONBOOT="yes"
IPADDR="192.168.1.1"
PREFIX="24"
\#GATEWAY="192.168.1.254" 只能有一个默认路由，网卡2做默认路由啦，这里就注释掉

vi /etc/sysconfig/network-scripts/ifcfg-eth1 //配置网卡2

NAME="eth1"
ONBOOT="yes"
IPADDR="10.1.1.1"
PREFIX="24"
GATEWAY0="10.1.1.254"

二、添加内网访问路由
vi /etc/sysconfig/network-scripts/route-eth0 //配置网卡1路由

ADDRESS0=192.168.0.0
NETMASK0=255.255.0.0
GATEWAY0=192.168.1.254

下面这样配置是一样的

192.168.0.0/16 via 192.168.1.254 dev eth0
1
三、重启网路服务，立即生效
systemctl restart network.service
四、查看路由表
ip route list
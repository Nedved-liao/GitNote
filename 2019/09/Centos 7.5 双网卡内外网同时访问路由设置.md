# Centos 双网卡内外网同时访问路由设置



## 说明

| 网卡 | 网段           | 网关          | 备注 |
| ---- | -------------- | ------------- | ---- |
| eth0 | 192.168.1.1/24 | 192.168.1.254 | 内网 |
| eth1 | 10.1.1.1/24    | 10.1.1.254    | 外网 |



## 1.0 配置网卡



### 一、配置eth0网卡

```bash
cat > /etc/sysconfig/network-scripts/ifcfg-eth0 <<EOF
DEVICE=eth0
ONBOOT=yes
BOOTPROTO=none
IPADDR=192.168.1.1
NETMASK=255.255.0.0
#GATEWAY=192.168.1.254
EOF
```

> 只能有一个默认路由，网卡eth1 做默认路由,所以eth0 的GATEWAY 就注释了



### 二、配置eth1网卡

 ```bash
cat > /etc/sysconfig/network-scripts/ifcfg-eth1 <<EOF
DEVICE=eth0
ONBOOT=yes
BOOTPROTO=none
IPADDR=10.1.1.1
NETMASK=255.255.0.0
GATEWAY=10.1.1.254
EOF
 ```



## 2.0 添加内网访问路由



### 一、配置eth0 的路由

```bash
cat > /etc/sysconfig/network-scripts/route-eth0
ADDRESS0=192.168.0.0
NETMASK0=255.255.0.0
GATEWAY0=192.168.1.254
EOF
```




## 3.0 重启网路服务

```bash
systemctl restart network
```



查看路由

```bash
route -n
ip route list
```


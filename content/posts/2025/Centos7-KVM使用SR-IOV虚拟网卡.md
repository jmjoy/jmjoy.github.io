+++
date = '2025-02-04T12:51:16+08:00'
title = 'Centos7 KVM使用SR-IOV虚拟网卡'
categories = ["运维"]
tags = ["KVM", "SR-IOV"]
+++

首先得搭建好`libvirtd`，和`SR-IOV`相关的BIOS和内核支持，并非本文重点，省略。

为了使用SR-IOV虚拟网卡，并且KVM支持网络高可用，需要使用`hostdev`挂载两张SR-IOV虚拟网卡，并且在KVM内部做bond。

## 宿主机的配置

假设母机需要给到虚拟机使用的物理网卡是p2p1，p2p3。

为了让`libvirtd`自动管理虚拟网卡的创建，可以定义两个network的xml，目录是`/etc/libvirt/qemu/networks/`：

passthrough-1.xml

```xml
<network> 
  <name>passthrough-1</name>
  <forward mode='hostdev' managed='yes'>
    <pf dev='p2p1'/>
  </forward>
</network>
```

passthrough-2.xml

```xml
<network> 
  <name>passthrough-2</name>
  <forward mode='hostdev' managed='yes'>
    <pf dev='p2p3'/>
  </forward>
</network>
```

然后分别执行：

```shell
virsh net-define passthrough-1.xml
virsh net-start passthrough-1
virsh net-autostart passthrough-1
 
virsh net-define passthrough-2.xml
virsh net-start passthrough-2
virsh net-autostart passthrough-2
```

## KVM的XML配置

定义net-device的xml，尖括号`<>`包含的变量替换成适合的：

*两张网卡的MAC_ADDRESS必须一致，否则在KVM内部做bond的时候会遇到`operation not permitted`的错误。*

kvm-net-device-1.xml

```xml
<interface type='network' managed='yes'>
  <mac address='<MAC_ADDRESS>'/>
  <driver name='vfio'/>
  <source network="passthrough-1"/>
  <vlan>
    <tag id='<VLAN_ID>'/>
  </vlan>
  <address type='pci' domain='0x0000' bus='0x00' slot='0x10' function='0x0'/>
</interface>

```

kvm-net-device-2.xml

```xml
<interface type='network' managed='yes'>
  <mac address='<MAC_ADDRESS>'/>
  <driver name='vfio'/>
  <source network="passthrough-2"/>
  <vlan>
    <tag id='<VLAN_ID>'/>
  </vlan>
  <address type='pci' domain='0x0000' bus='0x00' slot='0x11' function='0x0'/>
</interface>
```

热插网卡操作：

```shell
virsh attach-device <DOMAIN> kvm-net-device-1.xml
virsh attach-device <DOMAIN> kvm-net-device-2.xml
```

## KVM内网卡配置

`/etc/sysconfig/network-scripts`目录：

ifcfg-bond0

```ini
DEVICE=bond0
BONDING_OPTS=mode=balance-rr
TYPE=Bond
BONDING_MASTER=yes
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=none
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
NAME=bond0
ONBOOT=yes
ZONE=public
IPADDR="<IPADDR>"
GATEWAY="<GATEWAY>"
NETMASK="<NETMASK>"
DNS1="<DNS1>"
DNS2="<DNS2>"
BOOTPROTO="static"
```

ifcfg-ens16

```ini
DEVICE=ens16
BOOTPROTO=none
MASTER=bond0
SLAVE=yes
ONBOOT=yes
```

ifcfg-ens17

```ini
DEVICE=ens17
BOOTPROTO=none
MASTER=bond0
SLAVE=yes
ONBOOT=yes
```

然而，Centos7下，`ifcfg-bond0`在设置IP的时候会默认添加一条vlan的路由，将导致KVM内访问同vlan段的ip不通。

而我没有找到ifcfg脚本在调用`ip addr`的时候添加`noprefixroute`标记的配置项。

为了解决这个问题，添加钩子：

/sbin/ifup-local

```shell
#!/bin/bash
#
# ifup-local: 本地 ifup 钩子脚本
# 该脚本在所有接口启用时被调用，可以根据接口名称执行特定操作
 
/sbin/ip route del <FIRSTIP>/24
```

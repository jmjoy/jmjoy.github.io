+++
date = "2017-02-25T22:32:47+08:00"
title = "Linux高性能服务器学习笔记"
showonlyimage = false
image = "/images/Linux高性能服务器编程.jpg"
categories = ["其他"]
+++

# 基础部分 #

1. 查看DNS `cat /etc/resolv.conf`
*nameserver开头的就是DNS服务器的IP地址*

2. 查看IP地址 `host -t A www.baidu.com`

3. 查看MTU分片长度 `ifcofnig`

4. 查看路由表 `route`

5. 是否允许IP转发 `cat /proc/sys/net/ipv4/ip_forward`

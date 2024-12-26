---
title: "支持正则判定域名的DNS服务器"
date: 2020-04-21T12:42:26+08:00
---

公司的测试环境有一个需求，在测试域名加上分支信息，比如：

|          | 对应                                |
| -------- | ----------------------------------- |
| 正式域名 | [PROJECT].domain.com                |
| 测试域名 | [PROJECT]--[BRANCH].domain-test.com |

类似这种形式，其中`[BRANCH]`是git分支名字，用于区分业务测试联调的feature或者bugfix等。测试域名限制只能有三级，方便域名和https证书的管理。

那么问题就来了，不同`[PROJECT]`的测试域名可能指向的是不同的ip，常规的DNS服务器支持配置泛解析`*.domain-test.com`到某个ip，也支持针对配置特定三级域名`a.domain-test.com`解析到另一个ip，但却没有配置`a-*.domain-test.com`到某个ip的，需要枚举所有域名配置。这对于少量的域名来说问题不大，但是对于上述的需求则是不合理的。想想看，每次业务增加一个分支，运维就需求配置`[PROJECT]`×`[BRANCH]`笛卡尔积个域名，运维肯定会抗议的。

本来想着这个应该是蛮常见的需求，应该会有现成的方案吧，可是google了好多关键字，研究了几种DNS服务器，也没发现有支持类似需求的方案，也有可能是自己的搜索能力不行吧，于是决定撸起袖子自己搞了。

一开始是自己照着DNS的UDP协议自己实现了一遍的，用了一段时间也没啥问题。但是发现DNS协议并不只有UDP还有基于TCP协议的，而且DNS服务器还有很多的功能，自己不想重新造轮子，于是就决定二次开发一下某个DNS服务器，挑了一下就决定基于看起来比较简单的`dnsmasq`。

改动很简单，就是基于`address`配置加上一个正则的功能而已，代码开源出来了，仓库地址：[dnsmasq-plus](https://github.com/jmjoy/dnsmasq-plus)。

——

编辑于2020年7月1日
突然发现coredns其实是支持正则匹配域名的，详细请见：[https://coredns.io/plugins/template/](https://coredns.io/plugins/template/)，打脸了。

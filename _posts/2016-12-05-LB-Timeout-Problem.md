---
layout: post
title: 负载均衡超时问题
description: "负载均衡超时问题"
category: Web
avatarimg:
tags: [LB, LVS, Timeout]
duoshuo: true
---


# 引言

今天有一个电话面试，里面问到这样一个问题，连接 LVS,Nginx 负载均衡后的后端 Memcahced 服务器超时，分析一下可能是什么原因。
还问了下我们设置的连接超时时间是多少？  
因为最近 2 年没有实际负责过线上 LVS 相关业务维护经验，所以当时只依据对 LVS 和 Nginx 的负载均衡理论部分掌握，分析不出来是什么原因，
这个问题确实应该是有 LVS 和 Nginx 相关线上维护经验才能知道。
下面我来查找一些资料，看看上面问题的可能原因。


# 负载均衡超时问题

以下来自阿里云负载均衡相关文档，我们来学习下：

<pre>

为什么会碰到 VIP 连接访问超时

注：访问超时场景很多，本文档主要是从服务端入手分析

CASE 1 VIP 被安全防护

如流量黑洞和清洗，WAF 防护（waf 的特点是为建连后向 client 和 lvs 双向发送 rst 报文）

CASE 2 客户端端口不足

尤其容易发生在压测的时候，客户端端口不足会导致建立连接失败，负载均衡默认会抹除 tcp 连接的 timestamp 属性，linux协议栈的 tw_reuse(time_wait 状态连接复用)无法生效，time_wait 状态连接堆积导致客户端端口不足

解决方法：
客户端端使用长连接代替短连接。
使用 RST 报文断开连接（socket 设置 SO_LINGER 属性） ，而不是发 FIN 包这种方式断开。

CASE 3 后端服务器 accept 队列满

后端服务器 accept 队列满，导致后端服务器不回复 syn_ack 报文，客户端超时。

解决方法：默认的 net.core.somaxconn 参数为128，执行 sysctl -w net.core.somaxconn=1024 或者其它更大的值，并重启后端服务器上的应用。

CASE 4 从 4 层负载均衡后端服务器访问该 4 层负载均衡 VIP

4 层负载均衡，在该负载均衡的后端服务器上再去访问该负载均衡 VIP，这个目前是无法支持的，会导致连接失败，常见的场景是用户后端应用使用 URL 拼接的方式跳转访问

CASE 5 对连接超时的 rst 处理不当

负载均衡上建立 TCP 连接后如果 900s 未活动，则会向 client 和 rs 双向发送 rst 断开连接，有的应用对 rst 异常处理不当，可能会对已关闭的连接再次发送数据导致应用超时

</pre>


连接 LVS,Nginx 负载均衡后的后端 Memcahced 服务器超时，分析一下可能是什么原因。
经过下面一些资料查找学习，我分析可能的原因还有可能是 LVS 超时时间设置过小。
 

<pre>
LVS 的持久时间有 2 个

1. 把同一个 CIP 发来的请求转发到同一台 RS 的持久超时时间。

2.一个连接创建后空闲时的超时时间，这个超时时间分为 3 种。
01. TCP 的空闲超时时间
02. LVS 收到客户端 TCP FIN 的超时时间
03. UDP 的超时时间

</pre>

下面我来看看默认的 LVS 持久时间：

```bash
[root@VM_15_187_centos ~]# rpm -qa | grep keepalived
keepalived-1.2.13-5.el6_6.x86_64
[root@VM_15_187_centos ~]# grep persis /etc/keepalived/keepalived.conf 
    persistence_timeout 50
    persistence_timeout 50
    persistence_timeout 50
[root@VM_15_187_centos ~]# rpm -qa | grep ipvsadm   
ipvsadm-1.26-4.el6.x86_64
[root@VM_15_187_centos ~]# ipvsadm -Ln --timeout
Timeout (tcp tcpfin udp): 900 120 300
```    

因为最近 2 年没有实际负责过线上 LVS 相关业务维护经验，所以这个时间具体设置成多少我也不知道，  
但推测这个也应该来根据业务的实际情况来设置。


# Ref
[负载均衡超时问题](https://help.aliyun.com/document_detail/27659.html)  
[LVS连接的持久时间](http://lavafree.iteye.com/blog/1125906)  
[lvs & keepalived的tcp 长连接的问题解决办法](http://tangay.iteye.com/blog/1135586)  
[lvs持久性工作原理和配置](http://xstarcd.github.io/wiki/sysadmin/lvs_persistence.html)  
[LVS持久性工作原理和配置方法](http://www.3mu.me/lvs持久性工作原理和配置方法/)     
[lvs配置persistence_timeout 参数导致数据库负载不均](http://www.yalasao.com/43/lvs-persistence-timeout)  





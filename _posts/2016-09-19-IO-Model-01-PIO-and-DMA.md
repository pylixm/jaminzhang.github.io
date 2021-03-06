---
layout: post
title: I/O 模型 01 - PIO 与 DMA
description: "I/O 模型 01 - PIO 与 DMA"
category: OS
avatarimg:
tags: [IO, PIO, DMA]
duoshuo: true
---

I/O 模型 01 - PIO 与 DMA


我们所关注的 I/O 操作主要是网络数据的连接和发送，以及磁盘文件的访问，
我们将其归纳为多种模型，称为 I/O 模型，它们的本质便在于 CPU 的参与方式。

# PIO 与 DMA

在介绍 I/O 模型之前，有必要简单地说说慢速 I/O 设备和内存之间的数据传输方式。

我们拿磁盘来说，很早以前，磁盘和内存之间的数据传输是需要 CPU 控制的，也就是如果我们读取磁盘文件到内存中，
数据要经过 CPU 存储转发，这种方式称为 PIO。显然这种方式非常不合理，需要占用大量的 CPU 时间来读取文件，
造成文件访问时系统几乎停止响应。

后来， DMA(Direct Memory Access, 直接内存访问) 取代了 PIO，它可以不经过 CPU 而直接进行磁盘和内存的数据交换。
在 DMA 模式下，CPU 只需要向 DMA 控制器下达指令，让 DMA 控制器来处理数据的传送即可，
DMA 控制器通过系统总线来传输数据，传送完毕再通知 CPU，这样就在很大程度上降低了 CPU 占用率，
大大节省了系统资源，而它的传输速度与 PIO 的差异其实并不十分明显，因为这主要取决于慢速设备的速度。
可以肯定的是，PIO 模式的计算机我们现在已经很少见到了。

# Ref
[摘自《构建高性能Web站点》第3章 服务器并发处理能力 3.6 I/O 模型](https://book.douban.com/subject/3924175/)  

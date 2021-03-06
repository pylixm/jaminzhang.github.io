---
layout: post
title: 中小企业自动化部署 01 - 目标及流程
description: "中小企业自动化部署 01 - 目标及流程"
category: CI/CD
avatarimg:
tags: [CI/CD, Deploy, Rollback]
duoshuo: true
---

# 软件程序（代码）传统部署方式及缺点

1. 纯手工 scp
2. 纯手工登录服务器 git pull/svn update
3. 纯手工 ftp 上传
4. 开发提供压缩包，rz 上传，解压

上述传统部署方式缺点：

1. 全程运维参与，占用大量时间
2. 上传速度慢
3. 人为失误多，管理混乱
4. 回滚慢，不及时


# 环境规划

* 开发环境：开发人员本地有自己的环境，运维需要配置公用服务给开发的测试环境，例如开发数据库、redis/memcached 等。
* 测试环境：功能测试环境和性能测试环境。
* 预生产环境：生产环境集群中的某一个节点担任。
* 生产环境：直接对用户提供服务的环境。

>
测试环境和生产环境数据库是不一样的。

预生产环境产生的原因：

* 数据库不一致：测试环境和生产环境数据库是不一样的。
* 生产环境的联调接口并没有测试和生产环境之分，如第三方的支付接口。

# 如何设计一套生产自动化部署系统

前提：  
已经有一个可以上线的代码在代码仓库  

1. 规划
2. 实现
3. 总结和扩展（PDCA）
4. 在生产环境中应用

最重要的是第四步。

## 环境

一个集群中有 10 个节点

## 目标

* 实现一键部署程序到这 10 个节点
* 一键回滚到任意版本
* 一键回滚到上个版本

>
总体来说分为两块：部署和回滚。

## 部署

1. 代码在哪里？（git/svn）
2. 获取什么版本的代码？（直接拉取某个 Git 分支，指定 tag，指定 SVN 版本号）
3. 差异解决（1、各个节点之间的差异 2、代码仓库和实际的差异（配置文件是否放在代码仓库中？配置文件只在部署机上有，单独的项目单独的配置））
4. 如何更新（是否需要重启服务？如 Tomcat 需要重启）
5. 测试（部署完成后，测试是否正常）
6. 串行和并行（根据需求：一个一个或同时部署、分组部署）
7. 如何执行（1、Shell 命令行 2、Web 界面）

## 部署流程

1. 获取代码（直接拉取）
2. 编译（可选）
3. 放置配置文件（节点配置文件可能不一样）
4. 打包（加快传输速度）
5. SCP 到目标服务器（可配置 SSH 免密钥）
6. 将目标服务器移除集群（不同集群方法不一样）
7. 解压
8. 放置到 Webroot
9. SCP 差异文件（打包的是基础包，然后再加上差异文件）
10. 重启（可选）
11. 测试
12. 加入集群

![deploy-procedure](http://jaminzhang.github.io/images/CI-CD/deploy-procdure.png)  


## 回滚流程

回滚可以分成三种级别：

1. 正常回滚
2. 紧急回滚
3. 非常紧急回滚

### 1 正常回滚

1. 列出回滚版本
2. 目标服务器移除集群
3. 执行回滚
4. 重启和测试
5. 加入集群

### 2 紧急回滚

1. 列出回滚版本
2. 执行回滚
3. 重启和测试

### 3 非常紧急回滚

1. 直接回滚到上个版本（需要在某处记录上一个版本）
2. 重启和测试

![rollback-procedure](http://jaminzhang.github.io/images/CI-CD/rollback-procdure.png)  


# Ref
[中小企业自动化部署实践](https://www.unixhot.com/article/31)  


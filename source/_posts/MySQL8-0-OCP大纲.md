---
title: MySQL8.0 OCP大纲
tags:
  - MySQL
  - OCP
category: MySQL
cover: 1603562403.jpg
top_img: 1603562403.jpg
date: 2021-03-06 11:14:15
description: 本文主要列举MySQL8.0 OCP考点
---

本文主要列举MySQL8.0 OCP考点

### 体系结构

- 配置客户端到服务器的连接
- 了解MySQL如何存储数据
- 了解InnoDB如何存储数据和日志
- 配置缓冲区和缓存
- 了解和使用数据字典

### 服务器安装和配置

安装和使用MySQL服务器和客户端程序
识别在安装过程中创建的文件和文件夹
启动和停止MySQL
升级MySQL
使用选项和选项文件配置MySQL
配置MySQL变量
在同一主机上启动多个MySQL服务器

### 安全

- 创建用户帐户和角色
- 使用身份验证插件
- 控制用户和角色权限
- 认识常见的安全风险
- 安全的MySQL服务器连接
- 提供密码和登录安全性
- 保护MySQL主机环境
- 防止SQL注入攻击
- 加密MySQL资料
- 配置MySQL企业防火墙

### 监控与维护

- 配置和查看MySQL日志文件
- 监控MySQL进程和状态
- 配置MySQL企业审核
- 使用MySQL Enterprise Monitor查看MySQL中的活动
- 监控数据库增长并解释容量计划
- 解决资源锁定问题

### 查询优化

- 检查MySQL如何优化查询
- 使用MySQL Enterprise Monitor分析查询
- 创建索引以提高服务器性能
- 监视和了解索引统计信息

### 备份与恢复

区分不同类型的备份
实施备份策略
使用MySQL Enterprise Backup备份和还原数据
使用mysqldump和mysqlpump执行逻辑备份
说明何时以及如何使用原始文件备份
备份二进制日志

### 高可用性技术

说明复制如何提供高可用性和可伸缩性
配置复制
解释二进制日志在复制中的作用
配置多源复制
解释复制线程的作用
监控复制并排除故障
描述MySQL InnoDB集群和组复制
配置一个MySQL InnoDB集群
执行InnoDB集群恢复
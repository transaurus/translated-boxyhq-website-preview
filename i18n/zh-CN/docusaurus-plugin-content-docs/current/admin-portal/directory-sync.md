---
title: Manage Directory Sync using Admin Portal
sidebar_label: Directory Sync
---

# 管理目录同步

在管理控制台的**目录同步**菜单中，您可以查看目录连接详情。

## 创建目录同步连接

1. 进入管理控制台的**目录同步**模块
2. 点击**新建连接**
3. 填写以下信息：
   - 输入**目录名称**
   - 从[支持的提供商列表](../directory-sync/providers/)中选择**目录服务提供商**
   - 填写**租户标识**
   - 填写**产品标识**
   - 输入**Webhook接收地址**
   - 输入**Webhook安全密钥**

![创建目录](/images/docs/admin-portal/dsync/create-directory.png)

---

## 更新目录同步连接

1. 进入管理控制台的**目录同步**模块
2. 在目录连接列表中点击编辑图标
3. 可修改以下配置：
   - 更新**目录名称**
   - 修改**Webhook接收地址**
   - 重置**Webhook安全密钥**
   - 切换**启用Webhook事件日志**开关

![更新目录](/images/docs/admin-portal/dsync/update-directory.png)

---

## 查看目录用户

1. 进入管理控制台的**目录同步**模块
2. 在目录连接列表中点击数据库图标
3. 选择**用户**标签页
4. 点击眼睛图标查看用户原始JSON数据

![用户数据](/images/docs/admin-portal/dsync/users.png)

---

## 查看目录群组

1. 进入管理控制台的**目录同步**模块
2. 在目录连接列表中点击数据库图标
3. 选择**群组**标签页
4. 点击眼睛图标查看群组原始JSON数据

![群组数据](/images/docs/admin-portal/dsync/groups.png)

---

## 查看Webhook事件

1. 进入管理控制台的**目录同步**模块
2. 在目录连接列表中点击数据库图标
3. 选择**Webhook事件**标签页
4. 点击眼睛图标查看事件原始JSON数据

![事件日志](/images/docs/admin-portal/dsync/logs.png)

点击**清除事件**按钮可删除数据库中的所有事件记录

---

## 启用Webhook事件日志

1. 进入管理控制台的**目录同步**模块
2. 在目录连接列表中点击编辑图标
3. 切换**启用Webhook事件日志**开关

默认情况下Webhook事件不会被记录
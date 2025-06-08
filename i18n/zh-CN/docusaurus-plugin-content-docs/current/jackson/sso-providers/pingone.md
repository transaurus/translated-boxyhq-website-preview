---
title: Set up your own custom SAML application for PingOne SAML
sidebar_label: PingOne SAML
---

# PingOne SAML 单点登录

本节将指导您如何为PingOne SAML设置自定义SAML应用程序。

## 创建应用

在PingOne账户中，从左导航菜单选择**连接** > **应用程序**。

若应用已存在，请从列表中选择并跳转至[配置应用](#configure-application)章节。

若需新建SAML应用，点击加号按钮创建新应用。

![PingOne SAML 步骤1](/images/docs/jackson/sso-providers/pingone/1.png)

填写**应用名称**，从**应用类型**中选择**SAML应用**，点击**配置**。

## 配置应用

![PingOne SAML 步骤2](/images/docs/jackson/sso-providers/pingone/2.png)

在**SAML配置**部分填写以下字段值：

- **ACS URLs**
- **Entity ID**

请替换为从SAML Jackson获取的对应值。

点击**保存**完成配置。

![PingOne SAML 步骤3](/images/docs/jackson/sso-providers/pingone/3.png)

点击顶部**配置**标签页，选择**下载元数据**获取XML元数据文件。

![PingOne SAML 步骤4](/images/docs/jackson/sso-providers/pingone/4.png)

## 属性映射

点击顶部**属性映射**标签页，配置以下属性：

| SAML App  | PingOne       |
| --------- | ------------- |
| id        | User ID       |
| email     | Email Address |
| firstName | Given Name    |
| lastName  | Family Name   |

参考下方截图示例。

![PingOne SAML 步骤5](/images/docs/jackson/sso-providers/pingone/5.png)

确保通过应用旁的**切换**按钮启用应用，以供用户使用。

![PingOne SAML 步骤6](/images/docs/jackson/sso-providers/pingone/6.png)

## 后续步骤

您已成功完成PingOne SAML自定义应用的配置。现在可以为应用分配用户并开始使用。

## 相关资源

- [Ping Identity官网](https://www.pingidentity.com/en.html)
- [Ping Identity文档](https://docs.pingidentity.com/)

:::tip
遇到问题？[在此提问](https://discord.gg/uyb7pYt4Pa)
:::
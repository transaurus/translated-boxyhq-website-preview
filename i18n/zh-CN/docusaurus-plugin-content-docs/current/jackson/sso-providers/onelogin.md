---
title: Set up your own custom SAML application for OneLogin SAML
sidebar_label: OneLogin SAML
---

# OneLogin SAML 单点登录

本节将指导您如何为OneLogin SAML设置自定义SAML应用程序。

## 创建应用程序

登录OneLogin账户后，从顶部导航菜单点击**应用程序**。

若应用程序已创建，请从列表中选择并跳转至[配置应用程序](#configure-application)章节。

若需新建SAML应用程序，请点击**添加应用**按钮。

![OneLogin SAML 步骤1](/images/docs/jackson/sso-providers/onelogin/1.png)

在**查找应用程序**区域搜索**SAML测试连接器**，并从结果中选择**SAML自定义连接器(高级版)**。

![OneLogin SAML 步骤2](/images/docs/jackson/sso-providers/onelogin/2.png)

为应用程序输入**显示名称**后点击**保存**。

![OneLogin SAML 步骤3](/images/docs/jackson/sso-providers/onelogin/3.png)

## 配置应用程序

进入应用程序后，点击左侧**配置**标签页。

您需要在**应用程序详情**区域填写以下字段：

- **受众(实体ID)**
- **ACS(消费者)URL验证器**
- **ACS(消费者)URL**
- 从**SAML发起方**下拉菜单中选择**服务提供商**

请将字段值替换为从SAML Jackson获取的对应参数。

点击**保存**完成配置。

![OneLogin SAML 步骤4](/images/docs/jackson/sso-providers/onelogin/4.png)

![OneLogin SAML 步骤4.1](/images/docs/jackson/sso-providers/onelogin/4_1.png)

点击右上角**更多操作**下拉菜单，选择**SAML元数据**以下载元数据XML文件。

![OneLogin SAML 步骤5](/images/docs/jackson/sso-providers/onelogin/5.png)

## 属性映射

进入应用程序后点击左侧**参数**标签页。

需要配置以下属性：

| SAML Custom Connector Field | Value      |
| --------------------------- | ---------- |
| id                          | UUID       |
| email                       | Email      |
| firstName                   | First Name |
| lastName                    | Last Name  |

如下图所示。

![OneLogin SAML 步骤6](/images/docs/jackson/sso-providers/onelogin/6.png)

参考下图将**id**属性映射至**UUID**。

在**字段名**输入框填写"id"并勾选**包含在SAML断言中**复选框，点击**保存**继续。

![OneLogin SAML 步骤7](/images/docs/jackson/sso-providers/onelogin/7.png)

在后续界面中，从**值**下拉列表选择**UUID**并点击**保存**。

![OneLogin SAML 步骤8](/images/docs/jackson/sso-providers/onelogin/8.png)

对其他属性(email, firstName, lastName)重复相同操作。

## 后续步骤

您已成功完成OneLogin SAML自定义应用程序配置。现在可以为应用程序分配用户并开始使用。

## 相关资源

- [OneLogin](https://www.onelogin.com/) 官方网站
- 访问 [OneLogin 开发者文档](https://developers.onelogin.com/)

:::tip
遇到问题？[在此提问](https://discord.gg/uyb7pYt4Pa)
:::
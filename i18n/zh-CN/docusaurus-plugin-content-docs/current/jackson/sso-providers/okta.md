---
title: Set up your own custom SAML application for Okta SAML
sidebar_label: Okta SAML
---

# Okta SAML 单点登录配置

本节将指导您如何为Okta SAML设置自定义SAML应用程序。

## 创建应用程序

登录Okta账户后，从左导航菜单点击**应用程序**。

若应用程序已创建，请从列表中选择并跳转至[配置应用程序](#configure-application)章节。

若需新建SAML应用，点击**创建应用集成**按钮。

![Okta SAML 步骤1](/images/docs/jackson/sso-providers/okta/1.png)

在后续界面选择**SAML 2.0**并点击**下一步**。

![Okta SAML 步骤2](/images/docs/jackson/sso-providers/okta/2.png)

输入**应用名称**后点击**下一步**。

![Okta SAML 步骤3](/images/docs/jackson/sso-providers/okta/3.png)

## 配置应用程序

在**SAML设置**部分填写以下参数：

- **单点登录URL**
- **受众URI (SP实体ID)**
- 从**Name ID格式**下拉菜单中选择**EmailAddress**

请将参数值替换为从SAML Jackson获取的对应值。

![Okta SAML 步骤4](/images/docs/jackson/sso-providers/okta/4.png)

## 属性映射

在**属性声明**部分配置以下属性：

| Name      | Value          |
| --------- | -------------- |
| id        | user.id        |
| email     | user.email     |
| firstName | user.firstName |
| lastName  | user.lastName  |

参见下方截图示例。

![Okta SAML 步骤5](/images/docs/jackson/sso-providers/okta/5.png)

选择**我是添加内部应用的Okta客户**后点击**完成**。

![Okta SAML 步骤6](/images/docs/jackson/sso-providers/okta/6.png)

进入应用**登录**标签页，定位至**SAML签名证书**区域

点击对应证书的**操作**下拉菜单，选择**查看IdP元数据**。元数据XML文件将在新窗口打开，可复制至剪贴板。

![Okta SAML 步骤7](/images/docs/jackson/sso-providers/okta/7.png)

## 后续步骤

您已成功完成Okta SAML自定义应用的配置。现在可以为应用分配用户并开始使用。

## 相关资源

- [Okta](https://www.okta.com/) 官方网站
- 查阅[Okta文档](https://developer.okta.com/docs/concepts/saml/)

:::tip
遇到问题？[在此提问](https://discord.gg/uyb7pYt4Pa)
:::
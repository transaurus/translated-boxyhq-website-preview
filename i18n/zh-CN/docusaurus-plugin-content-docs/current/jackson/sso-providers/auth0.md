---
title: Set up your own custom SAML application for Auth0 SAML
sidebar_label: Auth0 SAML
---

# Auth0 SAML 单点登录配置

本节将指导您如何为Auth0 SAML配置自定义应用程序。

## 创建应用程序

在Auth0仪表盘中，从左导航菜单点击**Applications**。

若应用程序已创建，请从列表中选择并跳转至[配置应用程序](#configure-application)章节。

![Auth0 SAML 步骤1](/images/docs/jackson/sso-providers/auth0/1.png)

若需新建SAML应用程序，点击**Create Application**按钮。

输入应用程序**名称**后点击**Create**。

![Auth0 SAML 步骤1.1](/images/docs/jackson/sso-providers/auth0/1_1.png)

## 配置应用程序

点击**Addons**标签页，启用**SAML2 WEB APP**插件。

![Auth0 SAML 步骤2](/images/docs/jackson/sso-providers/auth0/2.png)

在后续界面输入**应用程序回调URL**并点击**Enable**。

请替换为从SAML Jackson获取的对应值。

![Auth0 SAML 步骤3](/images/docs/jackson/sso-providers/auth0/3.png)

切换至**Usage**标签页，下载**身份提供商元数据文件**。

![Auth0 SAML 步骤4](/images/docs/jackson/sso-providers/auth0/4.png)

## 属性映射

Auth0 SAML与SAML Jackson集成无需额外配置属性映射。

## 后续步骤

您已成功完成Auth0 SAML应用程序配置。现在可为应用程序分配用户并开始使用。

## 相关资源

- [Auth0官方文档](https://auth0.com/docs)

:::tip
如有疑问？[在此提问](https://discord.gg/uyb7pYt4Pa)
:::
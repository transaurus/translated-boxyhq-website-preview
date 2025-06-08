---
title: Set up your own custom SAML application for Microsoft Entra ID SAML
sidebar_label: Microsoft Entra ID SAML
---

# Microsoft Entra ID（原Azure AD）SAML单点登录配置

本节将指导您如何为Microsoft Entra ID SAML设置自定义应用程序。

## 创建应用程序

在Azure管理控制台中，从左导航菜单选择**企业应用程序**。

![Microsoft Entra ID SAML步骤1](/images/docs/jackson/sso-providers/azure/1.png)

若已创建应用程序，请从列表中选择并跳转至[配置应用程序](#configure-application)章节

若需新建SAML应用，点击顶部**新建应用程序**按钮。

![Microsoft Entra ID SAML步骤2](/images/docs/jackson/sso-providers/azure/2.png)

在后续界面点击**创建自己的应用程序**，输入应用名称后点击**创建**。

![Microsoft Entra ID SAML步骤3](/images/docs/jackson/sso-providers/azure/3.png)

## 配置应用程序

在应用的**管理**部分选择**单点登录**，然后选择**SAML**。

![Microsoft Entra ID SAML步骤4](/images/docs/jackson/sso-providers/azure/4.png)

点击**基本SAML配置**部分的**编辑**按钮。

![Microsoft Entra ID SAML步骤5](/images/docs/jackson/sso-providers/azure/5.png)

在**基本SAML配置**部分填写以下值：

- **标识符(实体ID)**
- **回复URL(断言消费者服务URL)**

请使用从SAML Jackson获取的对应值替换。

点击**保存**应用更改。

![Microsoft Entra ID SAML步骤6](/images/docs/jackson/sso-providers/azure/6.png)

## 属性映射

点击**属性与声明**部分的**编辑**按钮。

![Microsoft Entra ID SAML步骤7](/images/docs/jackson/sso-providers/azure/7.png)

您需要在**属性与声明**部分配置以下属性：

| Name                                                                 | Value                  |
| -------------------------------------------------------------------- | ---------------------- |
| `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress` | user.mail              |
| `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname`    | user.givenname         |
| `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name`         | user.userprincipalname |
| `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname`      | user.surname           |

请参考下方截图。

![Microsoft Entra ID SAML步骤8](/images/docs/jackson/sso-providers/azure/8.png)

进入**SAML签名证书**部分，下载**联合元数据XML**文件。

![Microsoft Entra ID SAML步骤9](/images/docs/jackson/sso-providers/azure/9.png)

## 后续步骤

您已成功完成Microsoft Entra ID SAML自定义应用程序配置。现在可以为应用分配用户并开始使用。

## 相关资源

- [Azure管理控制台](https://portal.azure.com/)

:::tip
遇到问题？[在此提问](https://discord.gg/uyb7pYt4Pa)
:::
---
title: Set up your own custom SAML application for Google SAML
sidebar_label: Google SAML
---

# Google SAML 单点登录配置

本节将指导您如何为Google SAML设置自定义应用程序。

## 创建应用程序

在Google管理控制台中，点击侧边栏的**应用**，然后从列表中选择**网页和移动应用**。

若已创建应用程序，请直接从列表中选择并跳转至[配置应用程序](#configure-application)章节。

若未创建SAML应用，请点击菜单中的**添加自定义SAML应用**。

![Google SAML 步骤1](/images/docs/jackson/sso-providers/google/1.png)

为应用程序输入**应用名称**后点击**继续**。

![Google SAML 步骤2](/images/docs/jackson/sso-providers/google/2.png)

## 配置应用程序

在下一界面点击**下载元数据**获取XML文件，随后点击**继续**。

![Google SAML 步骤3](/images/docs/jackson/sso-providers/google/3.png)

在**服务提供商详情**部分填写以下值：

- **ACS URL**
- **实体ID**

将值替换为从SAML Jackson获取的对应参数，点击**继续**保存配置。

![Google SAML 步骤4](/images/docs/jackson/sso-providers/google/4.png)

## 属性映射

在**属性**部分需配置以下属性：

| App attributes | Google directory attributes |
| -------------- | --------------------------- |
| email          | Primary email               |
| firstName      | First name                  |
| lastName       | Last name                   |

参考下方截图。

完成属性配置后点击**完成**保存设置。

![Google SAML 步骤5](/images/docs/jackson/sso-providers/google/5.png)

在下一界面点击**用户访问**以配置应用程序登录权限。

![Google SAML 步骤6](/images/docs/jackson/sso-providers/google/6.png)

勾选**为所有人启用**复选框并点击**保存**。

![Google SAML 步骤7](/images/docs/jackson/sso-providers/google/7.png)

## 后续步骤

您已成功完成Google SAML自定义应用程序配置。现在可为用户分配应用权限并开始使用。

## 相关资源

- [Google管理控制台](https://admin.google.com/)

:::tip
遇到问题？[在此提问](https://discord.gg/uyb7pYt4Pa)
:::
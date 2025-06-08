---
title: Set up your own custom SAML application for JumpCloud SAML
sidebar_label: JumpCloud SAML
---

# JumpCloud SAML 单点登录配置

本节将指导您如何为 JumpCloud SAML 设置自定义应用程序。

## 创建应用程序

在 JumpCloud 管理员控制台中，点击左侧导航菜单的 **SSO**。

若应用程序已创建，请从列表中选择并跳转至[配置应用程序](#configure-application)章节。

若需新建 SAML 应用，点击加号图标后选择 **Custom SAML App** 创建新应用。

![JumpCloud SAML 步骤1](/images/docs/jackson/sso-providers/jumpcloud/1.png)

为应用程序填写 **显示标签**。

![JumpCloud SAML 步骤2](/images/docs/jackson/sso-providers/jumpcloud/2.png)

## 配置应用程序

点击 **SSO** 标签页并填写以下字段：

- **IdP 实体 ID**
- **SP 实体 ID**
- **ACS URL**

请将上述值替换为从 SAML Jackson 获取的对应参数。

![JumpCloud SAML 步骤3](/images/docs/jackson/sso-providers/jumpcloud/3.png)

## 属性映射

在 **属性** 章节中配置以下属性：

参考下方截图。

| Service Provider Attribute Name | JumpCloud Attribute Name |
| ------------------------------- | ------------------------ |
| email                           | email                    |
| firstName                       | firstname                |
| lastName                        | lastname                 |

确保勾选 **声明重定向端点** 复选框。

最后点击 **激活** 保存应用配置。

![JumpCloud SAML 步骤4](/images/docs/jackson/sso-providers/jumpcloud/4.png)

返回已创建的 SAML 应用，点击 **SSO** 标签页中的 **导出元数据** 按钮下载 XML 元数据文件。

![JumpCloud SAML 步骤5](/images/docs/jackson/sso-providers/jumpcloud/5.png)

## 后续步骤

您已成功完成 JumpCloud 自定义 SAML 应用的配置。现在可为应用分配用户并开始使用。

## 相关资源

- [JumpCloud](https://jumpcloud.com/) 官方网站

:::tip
遇到问题？[点击提问](https://discord.gg/uyb7pYt4Pa)
:::
---
title: Set up your own custom SAML application for Rippling SAML
sidebar_label: Rippling SAML
---

# Rippling SAML 单点登录配置

本节将指导您如何为 Rippling SAML 设置自定义 SAML 应用程序。

## 创建应用程序

若 Rippling 应用目录中尚未存在该应用，您需在 Rippling 账户中新建一个**自定义应用**。

通过左侧导航菜单进入 **IT管理** > **自定义应用**。

![Rippling SAML 步骤1](/images/docs/jackson/sso-providers/rippling/1.png)

点击**创建新应用**按钮以新建应用程序。

在后续页面填写以下字段：

- **应用名称**
- **选择分类**
- **上传图标**
- **您想创建哪种类型的应用？** - 请确保从列表中选择**单点登录(SAML)**。

![Rippling SAML 步骤2](/images/docs/jackson/sso-providers/rippling/2.png)

## 配置应用程序

从后续页面复制**IdP元数据URL**或**IdP元数据XML**。该值将用于在 SAML Jackson 中配置您的SAML连接。

![Rippling SAML 步骤3](/images/docs/jackson/sso-providers/rippling/3.png)

您需要输入**ACS URL**和**实体ID**。这些值可从 SAML Jackson 获取。

![Rippling SAML 步骤4](/images/docs/jackson/sso-providers/rippling/4.png)

确保**服务提供商实体ID**与 SAML Jackson 中的**SAML受众(SP实体ID)**值匹配。

在您刚创建的自定义应用的**设置**选项卡中，进入**高级SAML设置**部分，勾选**禁用IdP发起的SSO断言中的InResponseTo字段**。

由于 Rippling 会为**InResponseTo**字段发送虚拟值，而 SAML Jackson 将尝试验证该字段（若存在）。

![Rippling SAML 步骤5](/images/docs/jackson/sso-providers/rippling/5.png)

## 后续步骤

您已成功为 Rippling SAML 配置自定义应用。此时可为应用分配用户并开始使用。

## 相关资源

- [Rippling](https://www.rippling.com/) 官方网站
- 访问 [Rippling 开发者文档](https://developer.rippling.com/)

:::tip
遇到问题？[在此提问](https://discord.gg/uyb7pYt4Pa)
:::
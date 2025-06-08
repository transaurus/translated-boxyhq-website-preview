---
title: JumpCloud SCIM v2.0
sidebar_label: JumpCloud SCIM v2.0
---

# JumpCloud SCIM v2.0

本指南将引导您完成配置 SAML Jackson 使用 JumpCloud SCIM v2.0 目录服务提供商的流程。

---

### 创建 JumpCloud SAML 应用

JumpCloud 仅支持在现有 SAML 应用中配置 SCIM 供应功能。

选择现有 SAML 应用或新建一个。从左导航菜单点击 **SSO** 并选择您的自定义 SAML 应用。

![JumpCloud SCIM 步骤 1](/images/docs/jackson/dsync-providers/jumpcloud/1.png)

在 SAML 应用内点击 **身份管理** 标签页。

在 **SCIM 版本** 下选择 **SCIM 2.0** 并填写以下信息：

- 基础 URL
- 令牌密钥
- 测试用户邮箱

点击 **测试连接** 验证凭证是否正确，然后点击 **激活**。

接着点击 **保存** 存储设置。

![JumpCloud SCIM 步骤 2](/images/docs/jackson/dsync-providers/jumpcloud/2.png)

至此，您已成功配置 JumpCloud SCIM 应用。

---

### 分配用户与推送群组

![JumpCloud SCIM 步骤 3](/images/docs/jackson/dsync-providers/jumpcloud/3.png)

在 SAML 应用内点击 **用户群组** 标签页。

可查看所有可用群组，选择需要同步的群组后点击 **保存**。

若无现有群组，请从左导航菜单点击 **用户群组** 并点击加号图标创建新群组。

为群组命名。

![JumpCloud SCIM 步骤 4](/images/docs/jackson/dsync-providers/jumpcloud/4.png)

选择 **用户** 标签页并勾选需要分配到该群组的用户。

![JumpCloud SCIM 步骤 5](/images/docs/jackson/dsync-providers/jumpcloud/5.png)

接着选择 **应用** 标签页并选择需要分配该群组的应用。

![JumpCloud SCIM 步骤 6](/images/docs/jackson/dsync-providers/jumpcloud/6.png)

至此，您已成功分配用户并推送群组。
---
title: Microsoft Entra ID SCIM v2.0
sidebar_label: Microsoft Entra ID SCIM v2.0
---

# Microsoft Entra ID SCIM v2.0

本指南将引导您完成配置SAML Jackson以使用Microsoft Entra ID SCIM v2.0目录提供商的流程。

---

### 创建Azure应用

在Microsoft Azure门户中点击**Azure Active Directory**。

![Microsoft Entra ID SCIM 步骤1](/images/docs/jackson/dsync-providers/azure/1.png)

从左导航菜单点击**企业应用程序**。

![Microsoft Entra ID SCIM 步骤2](/images/docs/jackson/dsync-providers/azure/2.png)

如果应用已创建，从列表中选择并跳转至章节[启用SCIM配置](#enable-azure-scim-provisioning)

若尚未创建应用，点击**新建应用程序**按钮。

![Microsoft Entra ID SCIM 步骤2.1](/images/docs/jackson/dsync-providers/azure/2_1.png)

在下一界面点击**创建自己的应用**，输入应用**名称**后点击**创建**按钮。

![Microsoft Entra ID SCIM 步骤3](/images/docs/jackson/dsync-providers/azure/3.png)

---

### 启用Azure SCIM配置

从左导航菜单点击**配置**，然后点击**开始使用**。

![Microsoft Entra ID SCIM 步骤4](/images/docs/jackson/dsync-providers/azure/4.png)

在**配置模式**下拉框选择**自动**，并在**管理员凭据**部分输入以下信息：

- 租户URL
- 密钥令牌

点击**测试连接**按钮验证凭据是否正确，点击**保存**存储凭据。

![Microsoft Entra ID SCIM 步骤5](/images/docs/jackson/dsync-providers/azure/5.png)

展开**映射**部分，确保为应用启用了用户和组的属性映射。默认映射应可正常工作。

![Microsoft Entra ID SCIM 步骤6](/images/docs/jackson/dsync-providers/azure/6.png)

展开**设置**部分并进行以下调整：

- 在**范围**下拉框选择**仅同步分配的用户和组**
- 确认**配置状态**设为**开启**

![Microsoft Entra ID SCIM 步骤7](/images/docs/jackson/dsync-providers/azure/7.png)

至此您已成功完成Microsoft Entra ID SCIM API集成配置。

---

### 分配用户

在应用中点击左导航菜单的**用户和组**，点击**添加用户/组**。

![Microsoft Entra ID SCIM 步骤8](/images/docs/jackson/dsync-providers/azure/8.png)

在**用户**下方选择**未选择**。

在界面右侧选择需分配至应用的用户，点击**选择**按钮后点击**分配**完成用户关联。

![Microsoft Entra ID SCIM 步骤9](/images/docs/jackson/dsync-providers/azure/9.png)

### 常见问题

#### Microsoft Entra ID SCIM默认同步频率是多少？

Microsoft Entra ID 会根据用户和组分配等情况，自动在应用程序中配置和更新用户账户。同步操作按固定时间间隔进行，通常每20-40分钟执行一次。

#### 为什么 Microsoft Entra ID SCIM 中的 PATCH 请求会失败？

这是 Entra ID SCIM 实现的一个已知问题。您需要在 SCIM 端点 URL 中添加查询参数 `?aadOptscim062020` 才能使 PATCH 请求正常工作。该查询参数会更新 PATCH 行为并确保 SCIM 合规性。[了解更多](https://learn.microsoft.com/en-us/entra/identity/app-provisioning/application-provisioning-config-problem-scim-compatibility#flags-to-alter-the-scim-behavior)
---
title: Okta SCIM v2.0
sidebar_label: Okta SCIM v2.0
---

# Okta SCIM v2.0

本指南将引导您完成配置SAML Jackson以使用Okta作为目录同步提供商的流程。

---

### 创建Okta应用程序

从左导航菜单点击**Applications**，然后点击**Browse App Catalog**按钮。

![Okta SCIM Step 1](/images/docs/jackson/dsync-providers/okta/1.png)

在搜索框中输入**SCIM 2.0 Test App (Oauth Bearer Token)**，选择名为**SCIM 2.0 Test App (Oauth Bearer Token)**的应用。

![Okta SCIM Step 2](/images/docs/jackson/dsync-providers/okta/2.png)

在应用页面点击**Add Integration**按钮。

![Okta SCIM Step 3](/images/docs/jackson/dsync-providers/okta/3.png)

为应用命名后点击**Next**按钮，点击**Done**完成应用创建。

![Okta SCIM Step 4](/images/docs/jackson/dsync-providers/okta/4.png)

---

### 启用Okta SCIM配置

在您的应用中点击**Provisioning**标签页，然后点击**Configure API Integration**。

![Okta SCIM Step 5](/images/docs/jackson/dsync-providers/okta/5.png)

勾选**Enable API Integration**复选框并输入以下信息：

- SCIM 2.0基础URL
- OAuth持有者令牌

这些信息将通过Jackson在通过API或管理门户创建目录同步连接时提供。

点击**Test API Credentials**测试连接凭证是否正确，然后点击**Save**保存凭证。

![Okta SCIM Step 6](/images/docs/jackson/dsync-providers/okta/6.png)

您将看到新设置页面，点击**To App**并启用以下设置：

![Okta SCIM Step 7](/images/docs/jackson/dsync-providers/okta/7.png)

至此，您已成功配置Okta SCIM API集成。

---

### 分配用户

在您的应用中点击**Assignments**标签页，从**Assign**下拉菜单中选择**Assign to People**。

![Okta SCIM Step 8](/images/docs/jackson/dsync-providers/okta/8.png)

选择要分配给应用的用户并点击**Assign**按钮。

![Okta SCIM Step 9](/images/docs/jackson/dsync-providers/okta/9.png)

点击**Assign**按钮后，将弹出包含多个字段的新窗口。确认字段值后点击**Save and Go Back**按钮。

![Okta SCIM Step 10](/images/docs/jackson/dsync-providers/okta/10.png)

---

### 推送群组

导航至**Push Groups**标签页，从**Push Groups**下拉菜单中选择**Find group by name**。

![Okta SCIM Step 11](/images/docs/jackson/dsync-providers/okta/11.png)

找到您想要分配给应用程序的群组，点击**保存**按钮。

![Okta SCIM 步骤 12](/images/docs/jackson/dsync-providers/okta/12.png)

### 常见问题

#### 为什么我没有看到Okta中被移除用户的DELETE事件？

Okta不会执行DELETE操作，而是向SCIM应用程序发送请求，将`active`属性设置为`false`。因此，对于从Okta应用程序中移除的用户，您会观察到`user.updated`事件，其中`active`属性被更改为`false`。

#### 为什么Okta中停用的用户没有触发事件？

Okta不会为被暂停的用户发送特定的取消配置事件。这种缺少停用事件的情况是Okta的一个已知问题。
---
title: OneLogin SCIM v2.0
sidebar_label: OneLogin SCIM v2.0
---

# OneLogin SCIM v2.0 配置指南

本指南将逐步介绍如何配置 SAML Jackson 以使用 OneLogin SCIM v2.0 目录服务提供商。

---

### 创建 OneLogin 应用

点击顶部导航菜单中的 **Applications**，然后点击 **Add App** 按钮。

![OneLogin SCIM 步骤1](/images/docs/jackson/dsync-providers/onelogin/1.png)

在搜索框中输入 **SCIM**，从应用列表中选择名为 **SCIM Provisioner with SAML (SCIM V2 Enterprise)** 的应用。

![OneLogin SCIM 步骤2](/images/docs/jackson/dsync-providers/onelogin/2.png)

为应用命名后点击 **Save** 按钮。

![OneLogin SCIM 步骤3](/images/docs/jackson/dsync-providers/onelogin/3.png)

至此，您已成功创建 OneLogin SCIM 应用。

---

### 启用 OneLogin SCIM 配置

在您的应用中，点击左侧的 **Configuration** 选项卡。

输入以下信息：

- SCIM 基础URL
- SCIM Bearer Token

这些信息将在您通过 API 或管理门户创建目录同步连接时由 Jackson 提供。

点击 **Enable** 按钮测试连接以验证凭证是否正确，然后点击 **Save** 按钮保存凭证。

![OneLogin SCIM 步骤4](/images/docs/jackson/dsync-providers/onelogin/4.png)

接着点击 **Provisioning** 选项卡，勾选 **Enable provisioning** 复选框。点击 **Save** 按钮保存更改。

![OneLogin SCIM 步骤5](/images/docs/jackson/dsync-providers/onelogin/5.png)

然后点击 **Parameters** 选项卡，从表格中选择 **Groups** 行。

![OneLogin SCIM 步骤6](/images/docs/jackson/dsync-providers/onelogin/6.png)

在弹出窗口中勾选 **Include in User Provisioning** 选项，点击 **Save** 按钮。

![OneLogin SCIM 步骤7](/images/docs/jackson/dsync-providers/onelogin/7.png)

---

### 分配用户

要为应用分配用户，请从顶部导航菜单中选择 **Users**，然后选择要分配的用户。

在用户页面中，点击左侧的 **Application** 选项卡，然后点击加号图标。

![OneLogin SCIM 步骤8](/images/docs/jackson/dsync-providers/onelogin/8.png)

弹出窗口将显示可分配的应用列表。选择您之前创建的应用，点击 **Continue** 按钮。

![OneLogin SCIM 步骤9](/images/docs/jackson/dsync-providers/onelogin/9.png)

在下一个模态窗口中点击 **Save** 确认分配。

![OneLogin SCIM 步骤10](/images/docs/jackson/dsync-providers/onelogin/10.png)

根据您的配置，可能需要审批此次分配。

如果在表格中看到显示为**待处理**的文本，请点击该文本。这将弹出一个模态窗口，点击**批准**按钮以确认分配。

![OneLogin SCIM 步骤11](/images/docs/jackson/dsync-providers/onelogin/11.png)

至此，该用户将被成功分配至应用程序。

---

### 推送群组

若要将群组推送至您的应用程序，需先在应用中创建新**角色**。

在顶部导航栏中选择**用户**，然后从下拉菜单中选择**角色**。

点击**新建角色**以创建新角色。

![OneLogin SCIM 步骤12](/images/docs/jackson/dsync-providers/onelogin/12.png)

输入角色名称，选择先前创建的应用程序，点击**保存**按钮。

![OneLogin SCIM 步骤13](/images/docs/jackson/dsync-providers/onelogin/13.png)

点击角色的**用户**标签页，搜索需分配该角色的用户。

点击**添加至角色**按钮分配用户，随后点击**保存**按钮。

![OneLogin SCIM 步骤14](/images/docs/jackson/dsync-providers/onelogin/14.png)

在后续模态窗口中点击**保存**以确认分配。

![OneLogin SCIM 步骤15](/images/docs/jackson/dsync-providers/onelogin/15.png)

返回您的应用程序，点击左侧**规则**标签页，点击**添加规则**按钮。

为规则命名。在**操作**部分，从下拉菜单中选择**在您的应用名称中设置群组**，然后为每个**角色**选择与**您的应用名称**匹配的值。点击**保存**按钮。

![OneLogin SCIM 步骤16](/images/docs/jackson/dsync-providers/onelogin/16.png)

点击左侧**用户**标签页，可能会在表格中看到**待处理**的配置项。点击该文本以批准分配。

![OneLogin SCIM 步骤17](/images/docs/jackson/dsync-providers/onelogin/17.png)

在后续模态窗口中点击**批准**以确认分配。

![OneLogin SCIM 步骤18](/images/docs/jackson/dsync-providers/onelogin/18.png)

### 常见问题

#### 为什么OneLogin中删除群组时没有触发任何事件？

这是OneLogin SCIM的已知问题。OneLogin不会为已删除的群组发送特定的取消配置事件，因此当群组被移除时，您不会收到诸如`group.deleted`或`group.user_removed`之类的事件。建议在从SCIM应用程序中删除群组之前，先移除群组中的所有用户。
---
title: Manage Enterprise SSO using Admin Portal
sidebar_label: Enterprise SSO
---

# 管理SSO连接

在管理控制台的**企业SSO**菜单中，您可以查看SSO连接的详细信息。

## 添加SAML连接

当您在身份提供商处设置好SAML应用后，身份提供商会生成一个SAML元数据文件。

SAML元数据文件包含有关SAML 2.0协议消息交换中可使用的各种SAML机构的信息。

从您的IdP下载SAML元数据文件，并按照以下步骤添加SAML连接。

- 点击**新建连接**按钮。
- 选择**SAML**作为类型。
- 为您的连接命名。
- 添加您的**租户**（例如：boxyhq.com）和**产品**（例如：flex）。
- 添加**允许的重定向URL**。
- 添加**默认重定向URL**。
- 粘贴**原始IdP XML**。
- 如果勾选了**ForceAuthn**，即使用户已有活跃会话，也将强制在SAML IdP重新认证。
- 点击**保存更改**按钮。

现在您已成功添加了第一个SAML连接。

## 添加OIDC连接

当您在身份提供商处注册好OIDC应用后，身份提供商会生成clientId和clientSecret。

`clientSecret`将用于认证OIDC客户端（此处为jackson）并获取`access_token`。

同时获取OpenID提供商的[发现](https://openid.net/specs/openid-connect-discovery-1_0.html#ProviderConfig)端点（通常以`/.well-known/openid-configuration`结尾），并按照以下步骤添加OIDC连接。

- 点击**新建连接**按钮。
- 选择**OIDC**作为类型。
- 为您的连接命名。
- 添加您的**租户**（例如：boxyhq.com）和**产品**（例如：flex）。
- 添加**允许的重定向URL**。
- 添加**默认重定向URL**。
- 输入暴露提供商元数据的OpenID提供商发现URL。
- 将**clientId**和**clientSecret**值粘贴到UI中的客户端ID[OIDC提供商]/客户端密钥[OIDC提供商]字段。
- 点击**保存更改**按钮。

现在您已成功添加了第一个OIDC连接。

## SAML联盟

SAML联盟是一项企业功能，您需要拥有企业许可证才能使用此功能。

在**管理控制台**中，进入**企业SSO** > **SAML联盟**。

### 创建SAML联盟应用

- 点击**新建应用**。
- 输入以下信息：
  - **名称**
  - **租户**
  - **产品**
  - **ACS URL**（服务提供商的ACS URL）
  - **实体ID**（服务提供商的实体ID）

![创建SAML联盟应用](/images/docs/admin-portal/saml-federation/create-saml-federation-app.png)

---

### 更新SAML联盟应用

- 在SAML联盟应用列表中，点击您要更新的应用的**编辑**图标。
- 您可以更新以下信息：
  - **名称**
  - **ACS URL**（服务提供商的ACS URL）
  - **实体ID**（服务提供商的实体ID）

![更新SAML联盟应用](/images/docs/admin-portal/saml-federation/update-saml-federation-app.png)

---

### 查看IdP配置

点击**查看IdP配置**按钮以查看SAML联盟应用的IdP配置。您需要向服务提供商提供以下信息：

- **SSO 登录地址**
- **实体ID / 受众URI / 受众限制**
- **X.509证书**

![查看IdP配置](/images/docs/admin-portal/saml-federation/view-idp-configuration.png)
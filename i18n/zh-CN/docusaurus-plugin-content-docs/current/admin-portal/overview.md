---
title: Get Started with Admin Portal
sidebar_label: Get Started
---

# 管理门户入门指南

SAML Jackson 配备了一个管理门户，通过易用的网页界面，您可以管理 **企业级单点登录(SSO)**、**目录同步** 和 **设置链接** 产品。

我们未来的产品（如适用）也将在管理门户中提供。

要启用管理门户，请确保您以服务形式部署 [Jackson](../jackson/deploy/service)。

## 认证方式

BoxyHQ SAML Jackson 支持多种认证方式。您可以选择以下任意一种方式对管理门户的用户进行身份验证。

我们使用 [NextAuth](https://next-auth.js.org) 来处理认证。设置以下环境变量以配置认证。

- `NEXTAUTH_URL`: 您的应用URL
- `NEXTAUTH_SECRET`: 用于加密会话cookie的密钥。可通过 `openssl rand -base64 32` 生成

### 1. 魔法链接

使用魔法链接，您无需输入密码即可登录管理门户。系统会向您发送包含登录链接的电子邮件。您需要配置 **SMTP服务器凭证** 来发送邮件。

配置以下环境变量以启用魔法链接：

- `SMTP_HOST`: SMTP服务器主机
- `SMTP_PORT`: SMTP服务器端口  
- `SMTP_USER`: SMTP服务器用户名  
- `SMTP_PASSWORD`: SMTP服务器密码  
- `SMTP_FROM`: 发送魔法链接邮件时使用的发件人邮箱  
- `NEXTAUTH_ACL`: 允许访问管理门户的邮箱或通配符数组。例如：`NEXTAUTH_ACL=tonystark@gmail.com,*@marvel.com`

按以下步骤登录管理门户：

- 访问您应用中的 `/api/auth/signin` 路径  
- 输入符合 `NEXTAUTH_ACL` 的邮箱地址，点击 **发送魔法链接**  
- 查收收件箱，您将收到来自 SAML Jackson 的邮件  
- 打开邮件并点击 **登录** 按钮

### 2. 邮箱密码登录

通过邮箱密码登录，您可以使用邮箱和密码组合登录管理门户。这可以作为魔法链接或SAML单点登录的替代方案，是最快捷的管理门户入门方式。

配置环境变量以启用邮箱密码认证：

- `NEXTAUTH_ADMIN_CREDENTIALS`: 逗号分隔的邮箱密码对。例如：`david@acme.com:secretpassword,john@acme.com:anotherpassword`

当前密码需以明文形式设置，我们将很快增加加密密码支持。

按以下步骤登录管理门户：

- 访问您应用中的 `/api/auth/signin` 路径  
- 输入与 `NEXTAUTH_ADMIN_CREDENTIALS` 匹配的邮箱和密码，点击 **登录**

### 3. SAML/OIDC单点登录

通过SAML/OIDC单点登录，您可以使用身份提供商(Identity Provider)登录管理门户。系统将重定向至您的身份提供商进行登录，认证成功后返回管理门户。

配置以下环境变量以启用SAML/OIDC单点登录：

- `ADMIN_PORTAL_SSO_TENANT`: 作为SSO连接的租户标识，建议设置不易与企业SSO主连接冲突的值  
- `ADMIN_PORTAL_SSO_PRODUCT`: 作为SSO连接的产品标识，建议设置不易与企业SSO主连接冲突的值

若未设置，将使用默认值 `_jackson_boxyhq` 和 `_jackson_admin_portal`。

要启用SAML/OIDC单点登录，您需要先通过[魔法链接](#1-magic-links)或[邮箱密码登录](#2-email-and-password)访问管理门户。登录后，即可在管理门户中配置SAML/OIDC单点登录。

前往**设置** > **管理门户SSO**，添加新的SSO连接。我们同时支持SAML和OIDC协议。

按照以下步骤登录管理门户：

- 访问您应用中的路径 `/api/auth/signin`
- 点击**使用SSO登录**按钮
- 系统将重定向至您的SAML身份提供商进行认证，完成后跳转回管理门户
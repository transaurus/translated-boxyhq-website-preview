---
title: UI Best Practices for Configuring SAML Single Sign-On
description: UI Best Practices for Configuring SAML Single Sign-On
sidebar_label: Configuring SAML SSO
---

# 配置SAML单点登录

关于SAML单点登录的概述，请阅读[SSO基础构建模块](/blog/sso-building-blocks)。

本指南旨在为您提供配置租户SAML单点登录的思路，以及在实施过程中可遵循的最佳实践。

## SSO连接标识符

多租户企业应用必须将SSO连接关联到**租户标识符**。例如，应用可以使用以下任意一项作为租户标识符：

- 邮箱域名
- 组织ID或组织别名
- 团队ID或团队别名
- 其他唯一标识符

通常，应用会要求用户输入租户标识符以启动认证流程。随后应用会根据该标识符查找对应的SSO连接。若找到有效连接，则将用户重定向至其身份提供商（IdP）；若未找到，则显示相应错误信息。

如果应用要求输入工作邮箱地址，则需提取域名部分作为租户标识符。

以下是其他应用采用的SSO连接标识方式：

- Airtable使用邮箱域名
- Vercel使用团队别名
- Netlify使用团队ID
- Sentry使用组织ID

了解更多关于[SAML单点登录的UI最佳实践](/guides/jackson/login-with-saml-sso)。

## SSO设置界面

应用应允许每个租户为其用户配置SAML单点登录。

根据应用的租户架构设计，可将SAML SSO配置页面置于组织或团队的设置页面下。

由于SAML SSO是企业专属的安全功能，最佳实践是将其配置页面放在**安全**选项卡下。

以下是Sentry为其客户配置SAML SSO的示例：

<p>
  <img
    src="/images/guides/sso/sentry-metadata.png"
    style={{ border: '1px solid black' }}
    title="The SAML SSO settings"
  />
</p>

要创建[通过SAML Jackson的SSO连接](/docs/jackson/sso-flow/#2-sso-connection-api)，必须传递租户标识符、产品名称和XML元数据。因此设置页面需提供表单以获取XML元数据。

通常由具备更高权限的管理员用户为其租户配置SAML SSO。

## 附加SSO设置

在应用中实施SAML SSO时，可考虑以下附加设置：

**强制SAML认证**

为增强安全性，可对租户强制执行SAML SSO，要求所有成员必须通过身份提供商（IdP）进行认证。

**默认角色**

允许租户设置默认角色。所有通过SSO创建新账户的成员都将被分配预设的默认角色。
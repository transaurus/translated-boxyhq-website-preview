---
title: Add SAML SSO to Django App with BoxyHQ
description: Add SAML SSO to Django App with BoxyHQ
sidebar_label: Django
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# 为Django应用添加SAML单点登录

本文档假设您已有一个Django应用，并希望为企业客户启用SAML单点登录认证。通过本指南，您将构建一个支持SAML单点登录用户认证的应用。

访问[GitHub仓库](https://github.com/boxyhq/jackson-examples/tree/main/apps/adonisjs)查看Django SAML单点登录集成的源代码。

为应用集成SAML单点登录需要完成以下步骤。

- 配置SAML单点登录
- 通过SAML单点登录进行认证

## 配置SAML单点登录

此步骤允许您的租户为其用户配置SAML连接。阅读以下指南以了解更多信息。

- [配置SAML单点登录的UI最佳实践](/guides/jackson/configuring-saml-sso)
- [SSO连接API](/docs/jackson/sso-flow/)

## 通过SAML单点登录进行认证

添加SAML连接后，应用可使用该连接通过SAML Jackson发起SSO认证流程。以下章节将重点介绍SSO认证部分。

[开发中 - 即将推出]
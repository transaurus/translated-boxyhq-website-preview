---
title: Set up app in your OpenID Connect Identity Provider
sidebar_label: OpenID Connect Provider
---

# OIDC 提供商设置

要允许应用或客户端通过OIDC身份提供商登录，我们需要在IdP中注册一个应用程序。具体流程可能因不同IdP而略有差异（请参考您的IdP文档），但存在几个通用配置项。

- **回调URL**：这是Jackson接收OIDC授权响应的端点。请确保设置为完整的HTTPS URL，即Jackson服务端点与[oidcPath](../deploy/env-variables.md#oidcpath)环境变量的拼接结果。Jackson服务中`oidcPath`的默认值为`/api/oauth/oidc`。
- **客户端/应用ID**：OIDC身份提供商通常会为注册的应用生成唯一标识符。请记录此ID，后续在Jackson中创建SSO连接时需要用到。
- **客户端密钥**：IdP在生成客户端ID的同时会生成一个用于令牌签发时验证客户端身份的密钥。请记录此密钥，后续在Jackson中创建SSO连接时需要用到。

:::info
由于Jackson在应用与OIDC身份提供商之间扮演代理角色，此处的应用程序/客户端即指Jackson服务本身。
:::
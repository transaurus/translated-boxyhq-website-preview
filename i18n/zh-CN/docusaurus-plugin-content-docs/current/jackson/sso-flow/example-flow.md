---
title: Example flow
sidebar_label: Example flow
---

## 场景示例

假设您有一个客户`acme.com`正在使用您的两项SaaS服务。暂将其命名为：

- `CRM（客户关系管理）` - 需与面向客户（acme.com的客户）的SAML身份提供商（Azure AD）集成
- `HRM（人力资源管理系统）` - 需与`acme.com`员工的Google Workspace账户集成

以下展示该场景下的SSO工作原理

## 配置SSO连接

首先作为应用开发者，您需要为CRM和HRM[添加](./index.md#21-add-connection)SSO连接。这些配置（下图标注为SAML元数据和OIDC元数据）将保存在Jackson中，后续用于协调身份提供商的登录流程。

## 登录流程

Jackson采用授权码流程，具体步骤如下：

1. 客户端（浏览器应用）通过重定向至Jackson的[`authorize`](./index.md#31-authorize)端点发起登录。Jackson将解析请求中的租户/产品信息，并据此将用户重定向至配置的身份提供商。
2. 第二步根据身份提供商类型有所不同：对于SAML身份提供商，Jackson会构建SAML请求并签名后发送至IdP，IdP验证请求后将用户重定向至登录页面；对于OIDC身份提供商，Jackson会构建OpenID Connect请求并重定向用户至OIDC提供商的授权端点。
3. 用户成功登录后，身份提供商将重定向回Jackson。SAML响应包含用户档案，OIDC响应则包含授权码（Jackson用其从OIDC身份提供商获取令牌和用户档案）。Jackson生成短期有效的代码并将用户档案与之关联存储。
4. 上一步生成的授权码发送至客户端应用。
5. 客户端通过[兑换](./index.md#32-code-exchange)代码获取令牌，并调用Jackson的[`userInfo`](./index.md#33-profile-request)端点获取完整用户档案。

![SSO流程](/images/docs/jackson/sso-flow.png)
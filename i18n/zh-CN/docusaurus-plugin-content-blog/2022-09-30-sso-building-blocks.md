---
slug: sso-building-blocks
title: SSO Building blocks - SAML, OAuth 2.0 and OpenID Connect
author: Aswin V
author_title: Senior Developer @BoxyHQ
author_url: https://twitter.com/av__2021
author_image_url: /images/authors/aswin.jpg
tags_disabled: [sso, engineering, saml-jackson, oauth-2.0]
image: /images/docs/jackson/sso-flow.png
---

我们已从[用户](2022-06-30-understanding-saml-sso-the-basics-from-the-user-side.md)和[应用提供商](2022-06-30-understanding-saml-sso-the-basics-from-the-solution-providers-side.md)两个角度对SAML进行了宏观层面的解析。

本文将深入探讨SAML、OAuth 2.0和OpenID Connect的技术细节，以及这些技术如何共同构建Jackson SSO的基石。

## SAML

SAML专为2000年代初期的传统Web应用设计，其目标是通过将认证委托给身份提供商（IdP）来为应用提供无缝用户体验。应用不再需要维护用户身份，只需将浏览器重定向至IdP——后者对用户进行认证后，返回关于已登录用户的断言。该断言本质上是一个令牌，向应用声明用户已在IdP完成认证，且断言在有效期内持续有效。

虽然SAML解决了SSO（单点登录）和联邦身份问题，但并未涉及授权机制。

随着单页应用（SPA）和移动端等原生平台的兴起，授权需求日益凸显。越来越多的逻辑从后端迁移至前端，应用间通过HTTP(S)相互通信的生态系统逐渐形成。例如：用户可通过应用预订电影票并向Facebook好友发送邀请。这意味着应用开始代表终端用户向其他服务（如Facebook）请求数据（好友列表）。传统解决方案是向应用共享服务凭证，但这极其危险——应用将获得等同于用户的所有权限。这催生了OAuth等标准协议，通过机制化授权实现受控的有限资源访问。

## OAuth 2.0

OAuth 2.0授权框架使第三方应用/客户端能够代表资源所有者（用户）获取对HTTP服务的受限访问权限。流程最终阶段，客户端将获得短期有效的访问令牌，用于访问受保护资源。

流程始于应用将用户代理重定向至授权服务器（AS）。AS对用户进行认证并获取资源访问许可后，携带授权代码重定向回客户端。该代码作为用户授权的凭证，最终被客户端用于换取访问令牌。此流程称为授权码许可（Authorization Code grant），是OAuth 2.0支持的四种许可类型之一，其余类型暂不赘述。

该流程具备以下优势：

- 用户仅需向授权服务器认证，凭证绝不暴露给应用
- 访问令牌不通过用户代理传输<sup>*</sup>，而是通过HTTP请求直达客户端
- 授权服务器可通过客户端密钥验证客户端身份

**\*** _需特别说明：另一种"隐式许可"类型确实会通过重定向URL片段返回访问令牌_

#### 将OAuth 2.0用于认证

由于认证通常发生在颁发访问令牌之前，通常认为持有访问令牌即代表认证已完成。随后该令牌被用于查询身份API以获取用户详情。

但这并不能完全作为认证凭证，且存在若干缺陷：

- 访问令牌对客户端是不透明的，其目标受众是受保护的资源服务器。
- 由于认证事件的信息未被暴露，无法判断用户是否仍处于登录状态。
- 在客户端直接从返回URL获取访问令牌（隐式授权）的情况下，攻击者有可能注入恶意令牌。通过使用授权码流程可缓解此风险，因为令牌是直接从令牌端点获取的。
- OAuth未对身份API的字段集进行标准化。例如：某个身份提供商可能使用`user_id`作为用户标识符，而另一个可能使用`subject`。这要求客户端针对不同身份提供商进行差异化处理。

基于OAuth 2.0构建的新标准"OpenID Connect"引入了ID令牌等新机制，可作为可靠的认证凭证，同时标准化了`scope`和`claims`等要素。

## OpenID Connect (OIDC)

OpenID Connect 1.0是在OAuth 2.0协议基础上构建的简单身份层。它支持三种流程，其中授权码流程和隐式流程与OAuth 2.0定义类似。关键区别在于会返回JWT格式（JSON Web Token）的ID令牌。ID令牌中的声明(claims)可帮助客户端可靠确认用户身份。受众(`aud`)声明会被设置为客户端ID，这意味着ID令牌专供该客户端使用。遵循OAuth 2.0规范，仍会返回访问令牌用于从"userinfo"端点获取用户信息。OIDC还标准化了ID令牌和"userinfo"响应中的声明。

## 构建SSO拼图

现在我们已掌握SSO的所有组件，在Jackson中整合后的流程如下所示。

![SSO流程](/images/docs/jackson/sso-flow.png)

请允许我进行说明。

我们需要支持两类SSO身份提供商——SAML和OIDC。假设有两个应用场景：一个CRM系统需要通过SAML登录Azure AD，另一个HRM系统需要通过OIDC登录Google Workspace。

#### 配置SSO连接

初步步骤（上图中绿色箭头标示）是向Jackson添加SSO连接。对于SAML身份提供商，这意味着保存来自IdP的XML元数据。该元数据包含发送SAML请求的SSO URL以及验证SAML断言签名的公钥。对于OIDC身份提供商，我们需要发现URL和客户端凭证（注册应用的clientID和clientSecret）。发现URL可用于查询IdP的元数据（如授权和令牌端点），而客户端凭证将用于认证执行OIDC流程的Jackson客户端。

#### 登录流程

Jackson同时支持OAuth 2.0和OpenID流程。通过支持带PKCE的授权码流程，OAuth 2.0流程变得更为安全。当客户端请求中包含`openid`范围时，Jackson将切换至OpenID流程。

登录流程遵循以下步骤：

1. 客户端（浏览器应用）通过重定向至Jackson的`authorize`端点发起登录。Jackson将解析请求中的租户/产品信息，并据此将用户重定向至配置的身份提供商（IdP）。
2. 第二步根据身份提供商类型有所不同。对于SAML IdP，Jackson会构建SAML请求、进行签名后发送至IdP。IdP验证请求并对用户进行认证。对于OIDC IdP，Jackson会构建OpenID Connect请求并将用户重定向至OIDC提供商的授权端点。
3. 用户成功登录后，IdP将重定向回Jackson。对于SAML，响应中包含用户档案。对于OIDC，响应包含授权码，Jackson使用该授权码从OIDC IdP获取令牌和用户档案。Jackson生成一个短期有效的`authorization code`并将用户档案与之关联存储。
4. 上一步生成的`authorization code`会被发送至客户端应用。
5. 客户端用该授权码交换令牌，并通过查询Jackson的userInfo端点获取完整用户档案。在OpenID流程中，Jackson会返回包含基本用户档案的ID Token。

## 最终思考

简而言之，Jackson充当了客户端应用与IdP之间的代理，承担了与配置的IdP进行SAML/OIDC流程协调的重任。通过这种方式，您可以快速扩展应用认证至任意数量的提供商，从而专注于核心产品开发。
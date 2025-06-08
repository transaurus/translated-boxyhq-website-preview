---
title: BoxyHQ SAML as Auth0 Social provider
sidebar_label: Auth0
---

# Auth0

[Auth0](https://auth0.com)无需过多介绍，它是最受欢迎的身份验证产品之一。然而在被Okta收购后，Auth0被迫将重心转向企业级市场，这导致初创公司只能免费获得三个企业级SSO连接（SAML或OIDC），之后就需要升级到昂贵的付费计划。

对于早期初创企业而言，这使得企业级SSO集成变得难以负担。为此我们开发了无缝方案：继续使用Auth0进行基础身份验证，同时通过BoxyHQ直接从Auth0接入企业级SSO。用户管理仍完全保留在Auth0中，但借助BoxyHQ的OAuth 2.0封装层，您可以为客户提供企业级SAML或OIDC登录功能。

访问我们的[GitHub代码库](https://github.com/boxyhq/jackson-examples/tree/main/apps/auth0-user-store)查看Auth0集成的源代码。

## 后续步骤

- 有问题需要咨询？[点击这里提问](https://discord.gg/uyb7pYt4Pa)
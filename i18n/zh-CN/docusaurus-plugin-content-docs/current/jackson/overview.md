---
title: Enterprise SSO Docs
---

# 企业级单点登录(SSO)

Jackson是我们为SAML/OIDC身份提供商提供的单点登录(SSO)服务[BoxyHQ开箱即用的SSO解决方案]。

Jackson通过OAuth 2.0流程实现SSO，抽象了底层SAML/OIDC协议的所有复杂性。查看我们的[GitHub仓库](https://github.com/boxyhq/jackson)。

## 架构设计

![企业级SSO架构图](/images/docs/jackson/enterprise-sso-arch.png)

我们采用模块化架构从零开发了Jackson系统。业务逻辑被分离到各个控制器中，这有助于我们逐步采用新功能。

![实现架构图](/images/docs/jackson/jackson-architecture.png)

## 核心特性

### 数据自主掌控​

- 开源解决方案让您完全掌控数据
- 支持自带数据库(BYOD)，兼容任意数据库
- 内置支持MySQL、MariaDB、Postgres、MongoDB、Redis、PlanetScale
- 完美适配主流云服务商的数据库服务

### 灵活易用​

- 专为OAuth 2.0/OpenID Connect 1.0流程设计
- 可与现有OAuth 2.0/OpenID Connect 1.0库集成

### 技术支持

通过以下渠道联系维护团队：

- [GitHub讨论区](https://github.com/boxyhq/jackson/discussions)
- [GitHub问题追踪](https://github.com/boxyhq/jackson/issues)
- [GitHub个人资料](https://github.com/deepakprabhakara)中的联系邮箱

### 社区支持

- [Discord社区](https://discord.gg/uyb7pYt4Pa)
- [Twitter账号](https://twitter.com/boxyhq)

### 开源许可

[Apache 2.0许可证](https://github.com/boxyhq/jackson/blob/main/LICENSE)
---
slug: changelog-saml-jackson-2023
title: 'BoxyHQ Changelog: Transforming Enterprise SSO and Directory Sync in 2023 with SAML Jackson'
author: Schalk Neethling
author_title: Customer Success, Community, DevRel - @BoxyHQ
author_url: https://github.com/schalkneethling
author_image_url: /images/authors/schalk.jpg
tags_disabled: [changelog]
image: /img/blog/saml-jackson-2023-changelog.jpg
---

2023年是[BoxyHQ的SAML Jackson项目](https://github.com/boxyhq/jackson)具有里程碑意义的一年，我们通过[一系列创新性更新](https://github.com/boxyhq/jackson/releases?page=1)重新定义了单点登录(SSO)和目录同步的实现方式。本文不仅回顾我们的成就，更致敬活跃社区的无价贡献。从推出首个企业级功能SAML联邦认证，到通过改进SAML配置界面提升用户体验，我们的每一步都致力于赋能全球开发者和组织。

这一年的旅程中，我们实现了诸多重要里程碑：支持OIDC连接的[RFC9207](https://datatracker.ietf.org/doc/html/rfc9207)规范、可配置的设置链接有效期、通过`DO_NOT_TRACK`和`BOXYHQ_NO_ANALYTICS`赋予隐私控制权。我们还通过SAML追踪器等功能强化安全性，简化了SAML配置错误的排查流程。

响应社区需求，我们[推出了软件即服务(SaaS)方案](https://app.eu.boxyhq.com/auth/join)，为偏好托管服务的用户提供选择。作为开源核心公司，我们很高兴能通过这项专有SaaS解决方案满足用户的多样化需求。

回顾这些关键时刻，我们始终铭记驱动开源创新的集体精神。每个版本都凝聚着我们对构建稳健、易用、可扩展解决方案的承诺，这离不开像您这样的用户反馈与贡献。

让我们共同细数2023年的亮点与2024年的新篇章，见证BoxyHQ Jackson如何持续进化，为所有人提供更安全高效的身份管理方案。

## 2023年第一季度重要版本

### SAML Jackson 1.6.0

- 管理门户支持企业SSO登录
- 新增匿名分析功能以了解产品使用情况，同时通过`DO_NOT_TRACK`或`BOXYHQ_NO_ANALYTICS`参数将控制权完全交给用户（即使启用也不会收集任何用户或设备识别信息）
- 管理门户集成审计日志服务([Retraced](https://github.com/retracedhq/retraced))
- 推出首个企业功能SAML联邦认证，实现单SAML连接多联邦连接复用
- 设置链接功能支持生成专属链接供客户配置SSO和目录同步连接

### SAML Jackson 1.7.0

- 支持每个租户和产品创建多个目录同步连接

### SAML Jackson 1.8.0

- 为知名SSO提供商显示友好名称（替代域名显示）
- 更新MySQL/MariaDB/MS-SQL表结构防止`ER_TOO_LONG_KEY`错误

### SAML Jackson 1.9.0

- OIDC连接配置时增加发现URL备用方案
- 数据隐私保险库产品UI上线（封闭测试阶段）

## 2023年第二季度重要版本

### SAML Jackson 1.10.0

- 新增简化SAML配置错误调试的功能。您可通过管理门户中的SAML追踪器选项卡访问该功能，查看SAML登录流程中的所有错误，并获取解决问题所需的完整上下文。
- 新增对DynamoDB作为存储引擎的支持
- 现已全面支持Azure AD目录同步的批量操作
- 新增SSO和目录同步连接操作的Webhook事件通知
- 支持启用/禁用SSO和目录同步连接
- 新增目录同步供应商支持：Google Workspace。由于Google的SCIM 2.0支持较为基础，我们直接对接其API以提供完整的目录同步功能

## 2023年第三季度重要版本更新

### SAML Jackson 1.11.0

- 新增按产品获取SSO和目录同步连接的API
- [重大变更] 根据规范要求，DELETE API在next.js中不再接受请求体，改为使用查询参数
- 新增配置链接(Setup Links)API
- 新增按产品过滤SAML追踪记录的API

### SAML Jackson 1.12.0

- SAML配置界面用户体验优化
- 新增Windows本地构建支持
- 为目录同步添加[OpenTelemetry](https://opentelemetry.io/)指标监控

## 2023年第四季度重要版本更新

### SAML Jackson 1.13.0

- OIDC颁发者(Issuer)现正确设置为外部URL并符合RFC规范
- 新增管理SAML联邦连接的API

### SAML Jackson 1.14.0

- 数据库架构更新，新增namespace列以加速扫描
- 目录同步：新增连接更新API
- 健康检查失败时记录错误日志
- 支持元数据中的多证书配置及SAML响应无证书时的验证

### SAML Jackson 1.15.0

- 新增目录同步事件批处理功能（通过DSYNC_WEBHOOK_BATCH_SIZE标志启用），以应对高流量场景
- 新增`Strict-Transport-Security`、`X-Frame-Options`和`X-Content-Type-Options`安全头
- 为OIDC连接附加友好供应商名称
- 支持回退到不含签名属性的证书
- 修复嵌入式npm的配置链接URL生成问题
- 优化SCIM实现以通过所有Okta Runscope测试

## 2024年前两周更新

新年伊始，我们已实现多项重要功能，并有更多计划正在推进。

- 配置链接过期时间现可配置（通过SETUP_LINK_EXPIRY_DAYS环境变量或API调用中的expiryDays属性）
- 新增对OIDC连接的RFC9207规范支持
- 在控制器和数据库接口中添加close函数，用于终止数据库连接
- 支持产品品牌定制（企业版功能）

## 总结

回顾BoxyHQ的Jackson项目在2023年至2024年初的卓越历程，我们充满自豪与感激。通过突破性功能和持续改进的旅程，充分展现了协作精神和开源理念的力量。衷心感谢社区每位成员的宝贵贡献、反馈以及对SAML Jackson的信任。

展望未来，我们期待继续与您同行，通过持续创新来应对身份管理领域的演进挑战。若认可我们的项目价值，欢迎在[GitHub上为我们加星](https://github.com/boxyhq/jackson)——这是表达支持并帮助更多人发现我们工作的好方式。若您认识可能受益于我们工具的人，也欢迎推荐。

请持续关注更多更新，我们将继续致力于为开发者和组织提供最优秀的SSO与目录同步技术解决方案。
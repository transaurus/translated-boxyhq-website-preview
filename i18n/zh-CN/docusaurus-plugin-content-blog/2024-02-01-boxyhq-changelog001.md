---
slug: changelog001
title: 'BoxyHQ Changelog, January 2024: SAML Jackson, Retraced, and More'
author: Schalk Neethling
author_title: Customer Success, Community, DevRel - @BoxyHQ
author_url: https://github.com/schalkneethling
author_image_url: /images/authors/schalk.jpg
tags_disabled: [changelog]
image: /images/blog/changelog001.jpg
---

欢迎阅读首期BoxyHQ更新日志！我们很高兴与您分享过去两周的重要更新，包括[SAML Jackson](https://github.com/boxyhq/jackson)（SSO与目录同步）、[Retraced](https://github.com/retracedhq/retraced)（审计日志）以及我们的[开源SaaS入门套件](https://github.com/boxyhq/saas-starter-kit)的显著改进。

## SAML Jackson

在SAML Jackson（我们的单点登录与目录同步解决方案）方面，我们致力于提供强大、易用且可扩展的解决方案。过去两周我们取得了重要进展，以下是v1.16.1至v1.18.2版本的主要亮点：

- 新增了尊重隐私的连接数统计功能
- 改进了自定义品牌支持（增值功能）
- SAML联盟现在支持桥接至OIDC协议
- 为OIDC连接添加错误追踪及相关改进
- 新增统计API端点用于查询SSO和目录同步连接数
- 原生集成[Ory](https://www.ory.sh/)平台
- 支持将多个租户关联至SAML联盟应用（特别适合大型企业）
- 对SAML联盟功能的整体优化
- 多项依赖项更新以保障安全性和性能

您可以在[GitHub查看完整更新日志](https://github.com/boxyhq/jackson/releases)。

## Retraced与SaaS入门套件

Retraced（我们的审计日志解决方案）是BoxyHQ与[Replicated](https://www.replicated.com/)的合作项目。虽然其更新频率不及SAML Jackson，但它是我们产品体系的关键组成部分。即将推出的新功能包括面向高级用户的SIEM（安全信息与事件管理）集成，支持将审计日志发送至[Splunk](https://www.splunk.com/)等SIEM平台。

您可以通过[GitHub获取Retraced完整更新记录](https://github.com/retracedhq/retraced/releases)。

> **注意：** 审计日志目前仅支持自托管模式。

[开源SaaS入门套件](https://github.com/boxyhq/saas-starter-kit)也有多项改进：团队可生成邀请链接简化新成员加入流程，增强了验证与错误处理机制，并更新了多个依赖项。如果您想深入了解BoxyHQ及我们的开源解决方案，[参与SaaS入门套件开发](https://github.com/boxyhq/saas-starter-kit/issues)将是最佳入门途径——我们在此诚挚邀请您加入。

本期更新日志到此结束。如果您有任何疑问，欢迎通过[Mastodon](https://hachyderm.io/@boxyhq)、[GitHub](https://github.com/boxyhq)、[Discord](https://discord.boxyhq.com/)或[Twitter/X](https://twitter.com/boxyhq)联系我们。
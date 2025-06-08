---
title: Directory Sync Docs
---

# 目录同步

目录同步帮助组织自动化管理用户的创建与删除流程。该系统基于跨域身份管理协议（SCIM）构建。

通过建立统一的用户身份数据源，该功能可节省大量组织管理时间，简化用户生命周期管理流程，同时确保数据安全。

目录同步功能自v1.2.1版本起集成于我们的SAML Jackson服务（及npm库）。部署详情请参阅[SAML Jackson](../jackson/deploy/index.md)文档，或访问[GitHub代码库](https://github.com/boxyhq/jackson#directory-sync)。

当前仅支持SCIM 2.0协议，因SCIM 1.1版本已弃用并将于2022年12月2日终止支持。若您仍需使用SCIM 1.1且无法迁移，请联系我们获取帮助。

![目录同步流程](/images/docs/jackson/directory-sync-flow.png)

## 概览

- [API参考](api-reference)
- [管理门户](../admin-portal/directory-sync)
- [Webhook](webhooks)
- [事件与类型](events)
- [目录同步服务商](providers)
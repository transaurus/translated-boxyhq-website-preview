---
title: Deploy Enterprise SSO on mogenius
sidebar_label: mogenius
---

# mogenius

本指南将引导您在mogenius平台上完成SAML Jackson的配置与部署。mogenius是开发者运行云端应用的最便捷方式，平台已提供SAML Jackson模板，您可在数分钟内将其部署至全自动化的云基础设施上。

若您尚无mogenius账户，请在此[注册](https://studio.mogenius.com/user/registration)。平台提供详尽的[入门指南](https://docs.mogenius.com/overview/quickstart)助您快速上手。

## 部署SAML Jackson

- 前往[mogenius](https://docs.mogenius.com/templates-and-services/various/saml-jackson)并按照指引将SAML Jackson作为Docker容器部署。
- 请从[Docker Hub](https://hub.docker.com/r/boxyhq/jackson/tags)部署最新版本，并保持镜像更新以获得额外安全保障。

### 验证部署

访问应用路径`/api/hello`可验证Jackson是否成功启动。若部署成功，您将在屏幕上看到如下输出内容。

```javascript
{
  name: 'Jules Winnfield';
}
```

## 后续步骤

至此您已在服务器上成功运行可用的SAML Jackson实例！

- 遇到问题？[在此提问](https://discord.gg/uyb7pYt4Pa)
- [登录管理门户](/docs/admin-portal/overview#authentication-methods)
- [添加SAML配置](/docs/admin-portal/enterprise-sso)
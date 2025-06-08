---
title: SAML Federation
sidebar_label: SAML Federation
---

# SAML 联合认证

SAML联合认证是企业级功能，需持有企业许可证方可使用。

该功能允许您无需额外插件或代码修改即可实现多身份提供商（IdP）联合认证。在SAML支持仅限于单一提供商而您需要对接多IdP的场景中尤为实用。请联系我们获取更多信息。

## SAML联合认证流程

若您使用Jackson作为SAML身份提供商（IdP）并需与其他IdP（如Okta）联合认证，流程如下：

- 用户访问服务提供商（SP）的登录页面
- 用户点击"使用SAML登录"按钮
- SP向Jackson的SSO端点发送SAML请求
- Jackson根据请求的租户和产品组合显示可选IdP列表（若存在多个IdP）
- Jackson将用户重定向至所选IdP进行认证
- 认证成功后，IdP向Jackson的ACS端点提交（POST方式）SAML响应
- Jackson处理来自IdP的SAML响应，生成新的SAML响应并提交（POST方式）至SP的ACS端点
- SP处理来自Jackson的SAML响应并为用户创建新会话（具体取决于SP的实现方式）

请参阅[创建SAML联合认证应用](/docs/admin-portal/enterprise-sso#saml-federation)了解配置方法。
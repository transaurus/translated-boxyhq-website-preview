---
title: Set up your own custom SAML application for a Generic IdP
sidebar_label: Generic SAML 2.0 Provider
---

# 通用SAML配置

本指南说明与身份提供商配置SAML所需的参数设置。完成配置后，您将获得一个XML元数据文件，需通过我们的[API（或调用NPM包中的API控制器连接方法）](../sso-flow/index.md#21-add-connection)进行配置。

> 请注意：所有URL末尾请勿添加斜杠。

请严格按照以下格式创建：

- 断言消费者服务URL/单点登录URL/目标URL：`http://localhost:5225/api/oauth/saml`

- 实体ID/标识符/受众URI/受众限制：`https://saml.boxyhq.com`

- 响应：`已签名`

- 断言签名：`已签名`

- 签名算法：`RSA-SHA256`

- 断言加密：`未加密`

:::info
已部署的Jackson服务会通过服务提供商(SP)端点公开上述元数据，访问地址为：`<jackson_endpoint>/.well-known/saml-configuration`
:::

## SAML属性/声明映射

如上所述，我们支持SAML声明中的4个标准属性——`id`、`email`、`firstName`、`lastName`。大多数提供商采用通用属性映射，但部分提供商使用自定义映射规则，具体请参考身份提供商的官方文档。

| SAML Attribute                                                         | Jackson mapping |
| ---------------------------------------------------------------------- | --------------- |
| `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier` | id              |
| `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress`   | email           |
| `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname`      | firstName       |
| `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname`        | lastName        |
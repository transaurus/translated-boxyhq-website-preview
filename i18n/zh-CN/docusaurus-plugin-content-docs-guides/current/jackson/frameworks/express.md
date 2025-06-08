---
title: Add SAML SSO to Express.js App with BoxyHQ
description: Add SAML SSO to Express.js App with BoxyHQ
sidebar_label: Express.js
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# 为Express.js应用添加SAML单点登录

本指南假设您已拥有一个Express.js应用，并希望为企业客户启用SAML单点登录认证。完成本指南后，您将获得一个支持通过SAML单点登录认证用户的应用。

访问[GitHub代码库](https://github.com/boxyhq/jackson-examples/tree/main/apps/express)查看Express.js集成SAML单点登录的源代码。

为应用集成SAML单点登录需要完成以下步骤。

- 配置SAML单点登录
- 通过SAML单点登录进行认证

## 配置SAML单点登录

此步骤允许您的租户为其用户配置SAML连接。阅读以下指南以了解更多信息。

- [配置SAML单点登录的UI最佳实践](/guides/jackson/configuring-saml-sso)
- [SSO连接API](/docs/jackson/sso-flow/)

## 通过SAML单点登录进行认证

添加SAML连接后，应用可使用该连接通过SAML Jackson发起SSO认证流程。以下部分将重点介绍SSO认证环节。

### 安装SAML Jackson

要开始使用SAML Jackson，请通过Node包管理器将该包添加至项目依赖。

```bash
npm i --save @boxyhq/saml-jackson
```

### 配置SAML Jackson

配置SAML Jackson以适配Express.js应用。

```js title="jackson.js"
const baseUrl = 'https://your-app.com';
const samlAudience = 'https://saml.boxyhq.com';
const product = 'saml-demo.boxyhq.com';
const samlPath = '/sso/acs';
const redirectUrl = `${baseUrl}/sso/callback`;

// SAML Jackson options
const options = {
  externalUrl: baseUrl,
  samlAudience,
  samlPath,
  db: {
    engine: 'sql',
    type: 'postgres',
    url: 'postgres://postgres:postgres@localhost:5432/postgres',
  },
};

module.exports = {
  baseUrl,
  product,
  samlPath,
  redirectUrl,
  samlAudience,
  options,
};
```

`samlPath`是身份提供商认证用户后提交SAML响应的路径，`redirectUrl`是SAML Jackson认证完成后重定向用户的地址。

按以下方式初始化SAML Jackson：

请注意`@boxyhq/saml-jackson`的初始化是异步操作，因此不能在顶层直接调用。应在初始化express服务器的函数中执行。

```js title="routes/index.js"
const { options, product, redirectUrl } = require('../jackson');

let apiController;
let oauthController;

(async function init() {
  const jackson = await require('@boxyhq/saml-jackson').controllers(options);

  apiController = jackson.connectionAPIController;
  oauthController = jackson.oauthController;
})();
```

### 发起认证请求

添加路由以启动认证流程：该路由通过将用户重定向至其配置的身份提供商来发起SAML单点登录流程。

<Tabs>
<TabItem value="01" label="With Tenant and Product" default>

```js title="/routes/index.js"
router.post('/sso', async (req, res, next) => {
  const tenant = 'boxyhq.com'; // The user's tenant

  const { redirect_url } = await oauthController.authorize({
    tenant,
    product,
    state: 'a-random-state-value',
    redirect_uri: redirectUrl,
  });

  res.redirect(redirect_url);
});
```

</TabItem>

<TabItem value="02" label="With Client ID">

```js title="/routes/index.js"
router.post('/sso', async (req, res, next) => {
  const clientId = '123456789'; // The tenant's client ID

  const { redirect_url } = await oauthController.authorize({
    client_id: clientId,
    state: 'a-random-state-value', // You can use the `state` parameter to restore application state between redirects.
    redirect_uri: redirectUrl,
  });

  res.redirect(redirect_url);
});
```

</TabItem>
</Tabs>

### 接收SAML响应

认证成功后，身份提供商将SAML响应以POST方式提交至断言消费者服务(ACS) URL。

添加路由来处理SAML响应。确保该路由与初始化SAML Jackson库时配置的`samlPath`值匹配，且能接收POST请求。

```js title="/routes/index.js"
router.post('/sso/acs', async (req, res, next) => {
  const { RelayState, SAMLResponse } = req.body;

  const { redirect_url } = await oauthController.samlResponse({
    RelayState,
    SAMLResponse,
  });

  res.redirect(redirect_url);
});
```

### 请求访问令牌

添加另一个路由用于接收认证后的回调。确保该路由与先前配置的`redirectUrl`值匹配。

应用通过传递授权`code`及认证信息（包括`client_id`、`client_secret`和`redirect_uri`）来请求`access_token`。

<Tabs>
<TabItem value="01" label="With Tenant and Product" default>

```js title="/routes/index.js"
router.get('/sso/callback', async (req, res, next) => {
  const { code, state } = req.query;

  const tenant = 'boxyhq.com'; // The user's tenant
  const product = 'saml-demo.boxyhq.com'; // Your app or product name

  const clientId = `tenant=${tenant}&product=${product}`;
  const clientSecret = 'dummy';

  // Exchange the `code` for `access_token`
  const { access_token } = await oauthController.token({
    code,
    client_id: clientId,
    client_secret: clientSecret,
    redirect_uri: redirectUrl,
  });
});
```

</TabItem>

<TabItem value="02" label="With Client ID">

```js title="/routes/index.js"
router.get('/sso/callback', async (req, res, next) => {
  const { code, state } = req.query;

  const clientId = '123456789'; // The tenant's client ID
  const clientSecret = 'dUdSOmGoxr'; // The tenant's client Secret

  // Exchange the `code` for `access_token`
  const { access_token } = await oauthController.token({
    code,
    client_id: clientId,
    client_secret: clientSecret,
    redirect_uri: redirectUrl,
  });
});
```

</TabItem>
</Tabs>

### 获取用户资料

获取`access_token`后，可用其从身份提供商检索用户资料。若授权有效，`userInfo`方法将返回包含用户资料的响应。

```js
const user = await oauthController.userInfo(access_token);
```

完整响应示例如下：

```json
{
  "id":"<id from the Identity Provider>",
  "email": "jackson@coolstartup.com",
  "firstName": "SAML",
  "lastName": "Jackson",
  "requested": {
    "tenant": "<tenant>",
    "product": "<product>",
    "client_id": "<client_id>",
    "state": "<state>"
  },
  "raw": {
    ...
  }
}
```

### 用户认证

当从身份提供商处获取到用户信息后，您需要判断该用户是否已存在于应用程序中并进行认证。若用户不存在，通常需要在数据库中创建新记录来存储该用户信息。
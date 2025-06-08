---
title: Add SAML SSO to AdonisJS App with BoxyHQ
description: Add SAML SSO to AdonisJS App with BoxyHQ
sidebar_label: AdonisJS
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# 为AdonisJS应用集成SAML单点登录

本指南假设您已拥有一个AdonisJS应用，并希望为企业客户启用SAML单点登录认证。完成本指南后，您的应用将能够通过SAML单点登录对用户进行身份验证。

访问[GitHub代码库](https://github.com/boxyhq/jackson-examples/tree/main/apps/adonisjs)查看AdonisJS集成SAML单点登录的源代码。

为应用集成SAML单点登录包含以下步骤：

- 配置SAML单点登录
- 通过SAML单点登录进行认证

## 配置SAML单点登录

此步骤允许您的租户为其用户配置SAML连接。阅读以下指南以了解更多：

- [配置SAML单点登录的UI最佳实践](/guides/jackson/configuring-saml-sso)
- [SSO连接API文档](/docs/jackson/sso-flow/)

## 通过SAML单点登录进行认证

添加SAML连接后，应用可使用该连接通过SAML Jackson发起SSO认证流程。以下部分将重点介绍SSO认证环节。

### 安装SAML Jackson

要开始使用SAML Jackson，请通过Node包管理器将该包添加至项目依赖。

```bash
npm i --save @boxyhq/saml-jackson
```

### 配置SAML Jackson

配置SAML Jackson以适配AdonisJS应用。

```js title="lib/jackson.ts"
import { type JacksonOption } from '@boxyhq/saml-jackson';

export const appUrl = 'https://your-app.com';
export const samlAudience = 'https://saml.boxyhq.com';
export const redirectUrl = `${appUrl}/sso/callback`;

export const options: JacksonOption = {
  externalUrl: appUrl,
  samlAudience,
  samlPath: '/sso/acs',
  db: {
    engine: 'sql',
    type: 'postgres',
    url: 'postgres://postgres:postgres@localhost:5432/postgres',
  },
};
```

`samlPath`是身份提供商在用户认证后提交SAML响应的端点，`redirectUrl`是SAML Jackson在认证完成后重定向用户的地址。

创建一个名为`JacksonProvider`的自定义Provider，该Provider依赖于`@boxyhq/saml-jackson`。其`boot`方法将初始化SAML Jackson并返回单例实例。

```js title="providers/JacksonProvider.ts"
import type { ApplicationContract } from '@ioc:Adonis/Core/Application';

import { options } from '../lib/jackson';

export default class JacksonProvider {
  constructor(protected app: ApplicationContract) {}

  public async boot() {
    const jackson = await require('@boxyhq/saml-jackson').default(options);

    this.app.container.singleton('BoxyHQ/Jackson', () => {
      const { connectionAPIController, oauthController } = jackson;

      return {
        connectionAPIController,
        oauthController,
      };
    });
  }

  public register() {
    // Register your own bindings
  }

  public async ready() {
    // App is ready
  }

  public async shutdown() {
    // Cleanup, since app is going down
  }
}
```

若使用TypeScript开发，需创建声明文件。

```js title="contracts/jackson.ts"
declare module '@ioc:BoxyHQ/Jackson' {
  import { type IOAuthController, type IConnectionAPIController } from '@boxyhq/saml-jackson';

  export const connectionAPIController: IConnectionAPIController;
  export const oauthController: IOAuthController;
}
```

### 发起认证请求

添加路由以启动认证流程：该路由通过将用户重定向至其配置的身份提供商来发起SAML SSO流程。

```js title="start/routes.ts"
import LoginController from 'App/Controllers/Http/LoginController';

Route.post('/login', async (ctx) => {
  return new LoginController().store(ctx);
});
```

`LoginController`的`store`方法负责将用户重定向至身份提供商。

<Tabs>
<TabItem value="01" label="With Tenant and Product" default>

```js title="app/Controllers/Http/LoginController.ts"
import type { HttpContextContract } from '@ioc:Adonis/Core/HttpContext';

import { oauthController } from '@ioc:BoxyHQ/Jackson';
import { type OAuthReq } from '@boxyhq/saml-jackson';
import { redirectUrl } from '../../../lib/jackson';

export default class LoginController {
  public async store({ request, response }: HttpContextContract) {
    const tenant = 'boxyhq.com'; // The user's tenant
    const product = 'saml-demo.boxyhq.com'; // Your app or product name
    const state = 'a-random-state-value'; // You can use the `state` parameter to restore application state between redirects.

    const { redirect_url } = await oauthController.authorize({
      tenant,
      product,
      state,
      redirect_uri: redirectUrl,
    } as OAuthReq);

    return response.redirect(redirect_url as string);
  }
}
```

</TabItem>

<TabItem value="02" label="With Client ID">

```js title="app/Controllers/Http/LoginController.ts"
import type { HttpContextContract } from '@ioc:Adonis/Core/HttpContext';

import { oauthController } from '@ioc:BoxyHQ/Jackson';
import { type OAuthReq } from '@boxyhq/saml-jackson';
import { redirectUrl } from '../../../lib/jackson';

export default class LoginController {
  public async store({ request, response }: HttpContextContract) {
    const clientId = '123456789'; // The tenant's client ID

    const { redirect_url } = await oauthController.authorize({
      client_id: clientId,
      state: 'a-random-state-value', // You can use the `state` parameter to restore application state between redirects.
      redirect_uri: redirectUrl,
    } as OAuthReq);

    return response.redirect(redirect_url as string);
  }
}
```

</TabItem>
</Tabs>

### 接收SAML响应

认证成功后，身份提供商将SAML响应POST至断言消费者服务(ACS)URL。

添加路由以处理SAML响应。确保该路由与初始化SAML Jackson库时配置的`samlPath`值匹配，且能够接收POST请求。

```js title="start/routes.ts"
import SSOController from 'App/Controllers/Http/SSOController';

Route.post('/sso/acs', async (ctx) => {
  return new SSOController().acs(ctx);
});
```

`SSOController`的`acs`方法负责处理来自身份提供商的SAML响应，并将用户重定向至`redirectUrl`。

```js title="app/Controllers/Http/SSOController.ts"
import type { HttpContextContract } from '@ioc:Adonis/Core/HttpContext';

import { oauthController } from '@ioc:BoxyHQ/Jackson';

export default class SSOController {
  public async acs({ request, response }: HttpContextContract) {
    const relayState = request.input('RelayState');
    const samlResponse = request.input('SAMLResponse');

    const { redirect_url } = await oauthController.samlResponse({
      RelayState: relayState,
      SAMLResponse: samlResponse,
    });

    return response.redirect(redirect_url as string);
  }
}
```

### 请求访问令牌

添加另一个路由用于接收认证完成后的回调。确保该路由与先前配置的`redirectUrl`值匹配。

```js title="apps/adonisjs/start/routes.ts"
import SSOController from 'App/Controllers/Http/SSOController';

Route.get('/sso/callback', async (ctx) => {
  return new SSOController().callback(ctx);
});
```

应用通过传递授权`code`及包含`client_id`、`client_secret`和`redirect_uri`的认证详情来请求`access_token`。

`SSOController`的`callback`方法负责此流程。

<Tabs>
<TabItem value="01" label="With Tenant and Product" default>

```js title="app/Controllers/Http/SSOController.ts"
import type { HttpContextContract } from '@ioc:Adonis/Core/HttpContext';

import { oauthController } from '@ioc:BoxyHQ/Jackson';
import { type OAuthTokenReqWithCredentials } from '@boxyhq/saml-jackson';
import { redirectUrl } from '../../../lib/jackson';

export default class SSOController {
  public async callback({ request, response, auth }: HttpContextContract) {
    const { code, state } = request.qs();

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
    } as OAuthTokenReqWithCredentials);
  }
}
```

</TabItem>

<TabItem value="02" label="With Client ID">

```js title="app/Controllers/Http/SSOController.ts"
import type { HttpContextContract } from '@ioc:Adonis/Core/HttpContext';

import { oauthController } from '@ioc:BoxyHQ/Jackson';
import { type OAuthTokenReqWithCredentials } from '@boxyhq/saml-jackson';
import { redirectUrl } from '../../../lib/jackson';

export default class SSOController {
  public async callback({ request, response, auth }: HttpContextContract) {
    const { code, state } = request.qs();

    const clientId = '123456789'; // The tenant's client ID
    const clientSecret = 'dUdSOmGoxr'; // The tenant's client Secret

    // Exchange the `code` for `access_token`
    const { access_token } = await oauthController.token({
      code,
      client_id: clientId,
      client_secret: clientSecret,
      redirect_uri: redirectUrl,
    } as OAuthTokenReqWithCredentials);
  }
}
```

</TabItem>
</Tabs>

### 获取用户资料

获取到 `access_token` 后，您可以用它从身份提供商处检索用户资料。若授权有效，`userInfo` 方法将返回包含用户资料的响应。

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

从身份提供商获取用户信息后，您可判断该用户是否存在于您的应用中并进行认证。若用户不存在，通常需要在数据库中创建新记录来存储该用户信息。
# NPM 库

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

SAML Jackson 提供 [npm 包](https://www.npmjs.com/package/@boxyhq/saml-jackson)形式，可集成至任何基于 **Node.js** 的 Web 应用框架。

使用以下命令安装 SDK：

```bash
npm install @boxyhq/saml-jackson
```

:::info

ESM 库与动态导入

在捆绑服务端代码中处理 `jose` 和 `openid-client`：

我们的 npm 库会动态导入 `jose` 和 `openid-client` 包，因为它们已转为 ESM（ECMAScript 模块）格式。

若您正在捆绑服务端代码（例如 Next.js 项目），这些依赖可能在构建过程中被排除。请按以下步骤确保包含它们：

- 在代码中添加对 jose 和 openid-client 的虚拟导入

- 在 Next.js 配置中设置 `outputFileTracingIncludes` 以显式包含这些依赖

其他捆绑服务端代码的框架也需要类似调整。

:::

---

## 配置选项

请注意 `@boxyhq/saml-jackson` 的初始化是异步操作，不能在顶层直接调用。

```js
import jackson, {
  type IConnectionAPIController,
  type IOAuthController,
} from '@boxyhq/saml-jackson';

let oauth: IOAuthController;
let connection: IConnectionAPIController;

(async function init() {
  const jackson = await require('@boxyhq/saml-jackson').controllers({
    externalUrl: 'https://your-app.com',
    samlAudience: 'https://saml.boxyhq.com',
    oidcPath: '/api/oauth/oidc',
    samlPath: '/api/oauth/saml',
    db: {
      engine: 'sql',
      type: 'postgres',
      url: 'postgres://postgres:postgres@localhost:5432/postgres',
    },
  });

  oauth = jackson.oauthController;
  connection = jackson.connectionAPIController;
})();
```

---

## 单点登录连接

### 创建 SAML 连接

创建新的 SAML 单点登录连接。

<Tabs>
<TabItem value="01" label="Request" default>

```js
await connection.createSAMLConnection({
  tenant: 'boxyhq',
  product: 'your-app',
  rawMetadata: '<raw-saml-metadata>', // Visit https://mocksaml.com to download Metadata
  redirectUrl: ['https://your-app.com/*'],
  defaultRedirectUrl: 'https://your-app.com/sso/callback',
});
```

</TabItem>

<TabItem value="02" label="Response">

```json
{
  "defaultRedirectUrl": "https://your-app.com/sso/callback",
  "redirectUrl": ["https://your-app.com/*"],
  "tenant": "boxyhq",
  "product": "your-app",
  "clientID": "f7c909a5c72a5535847acf32558b2429a5172dd6",
  "clientSecret": "cc6ba07bc42c2f449c9b0a3cc41c256dea08f705e1b44fdc",
  "forceAuthn": false,
  "idpMetadata": {
    "sso": {
      "postUrl": "https://mocksaml.com/api/saml/sso",
      "redirectUrl": "https://mocksaml.com/api/saml/sso"
    },
    "slo": {},
    "entityID": "https://saml.example.com/entityid",
    "thumbprint": "d797f3829882233d3f01e49643f6a1195f242c94",
    "validTo": "Jul  1 21:46:38 3021 GMT",
    "loginType": "idp",
    "provider": "saml.example.com"
  },
  "certs": {
    "publicKey": "-----BEGIN CERTIFICATE----- ... -----END CERTIFICATE-----\r\n",
    "privateKey": "-----BEGIN RSA PRIVATE KEY----- ... -----END RSA PRIVATE KEY-----\r\n"
  }
}
```

</TabItem>
</Tabs>

### 更新 SAML 连接

更新现有 SAML 单点登录连接。

```js
await connection.updateSAMLConnection({
  tenant: 'boxyhq',
  product: 'your-app',
  rawMetadata: '<raw-saml-metadata>',
  redirectUrl: ['https://your-app.com/*'],
  defaultRedirectUrl: 'https://your-app.com/sso/callback-updated',
  clientID: '<clientID of the SAML SSO Connection>',
  clientSecret: '<clientSecret of the SAML SSO Connection>',
});
```

### 创建 OIDC 连接

创建新的 OIDC 单点登录连接。

<Tabs>
<TabItem value="01" label="Request" default>

```js
await connection.createOIDCConnection({
  tenant: 'boxyhq',
  product: 'your-app',
  redirectUrl: ['https://your-app.com/*'],
  defaultRedirectUrl: 'https://your-app.com/sso/callback',
  oidcDiscoveryUrl:
    'https://accounts.google.com/.well-known/openid-configuration',
  oidcClientId: '<OpenID Client ID>',
  oidcClientSecret: '<OpenID Client Secret>',
});
```

</TabItem>

<TabItem value="02" label="Response">

```json
{
  "defaultRedirectUrl": "https://your-app.com/sso/callback",
  "redirectUrl": ["https://your-app.com/*"],
  "tenant": "boxyhq",
  "product": "your-app",
  "clientID": "749f95c4bd02b4adb6c0633249e70d5ad45b75e2",
  "clientSecret": "2d730ac71c74e7d49dccf362c9a61005b6246cc65d6d0fa4",
  "oidcProvider": {
    "discoveryUrl": "https://accounts.google.com/.well-known/openid-configuration",
    "clientId": "<OpenID Client ID>",
    "clientSecret": "<OpenID Client Secret>",
    "provider": "accounts.google.com"
  }
}
```

</TabItem>
</Tabs>

### 更新 OIDC 连接

更新现有 OIDC 单点登录连接。

```js
await connection.updateOIDCConnection({
  tenant: 'boxyhq',
  product: 'your-app',
  redirectUrl: ['https://your-app.com/*'],
  defaultRedirectUrl: 'https://your-app.com/sso/callback',
  oidcDiscoveryUrl:
    'https://accounts.google.com/.well-known/openid-configuration',
  oidcClientId: '<OpenID Client ID>',
  oidcClientSecret: '<OpenID Client Secret>',
  clientID: '<clientID of the OIDC SSO Connection>',
  clientSecret: '<clientSecret of the OIDC SSO Connection>',
});
```

### 获取 SAML/OIDC 连接

获取现有 SAML 或 OIDC 单点登录连接的详细信息。

<Tabs>
<TabItem value="01" label="Request" default>

```js
// Using tenant and product
await connection.getConnections({
  tenant: 'boxyhq',
  product: 'your-app',
});

// Using the client ID
await connection.getConnections({
  clientID: '<clientID of the SSO Connection to be retrieved>.',
});
```

</TabItem>
<TabItem value="02" label="Response">

```json
[
  {
    "defaultRedirectUrl": "https://your-app.com/sso/callback",
    "redirectUrl": ["https://your-app.com/*"],
    "tenant": "boxyhq",
    "product": "your-app",
    "clientID": "...",
    "clientSecret": "...",
    "forceAuthn": false,
    "idpMetadata": {
      "sso": {
        "postUrl": "https://mocksaml.com/api/saml/sso",
        "redirectUrl": "https://mocksaml.com/api/saml/sso"
      },
      "slo": {},
      "entityID": "https://saml.example.com/entityid",
      "thumbprint": "d797f3829882233d3f01e49643f6a1195f242c94",
      "validTo": "Jul  1 21:46:38 3021 GMT",
      "loginType": "idp",
      "provider": "saml.example.com"
    },
    "certs": {
      "publicKey": "-----BEGIN CERTIFICATE----- ... -----END CERTIFICATE-----\r\n",
      "privateKey": "-----BEGIN RSA PRIVATE KEY----- ... -----END RSA PRIVATE KEY-----\r\n"
    }
  }
]
```

</TabItem>
</Tabs>

### 删除 SAML/OIDC 连接

删除现有 SAML 或 OIDC 单点登录连接。

```js
// Using tenant and product
await connection.deleteConnections({
  tenant: 'boxyhq',
  product: 'your-app',
});

// Using client ID and client secret
await connection.deleteConnections({
  clientID: '<clientID of the SSO Connection>',
  clientSecret: '<clientSecret of the SSO Connection>',
});
```

---

## 单点登录认证

### 处理 OAuth 2.0（或 OIDC）授权请求

要启动流程，应用必须触发 OAuth 2.0（或 OIDC）重定向至您应用的授权端点。您需要在授权处理器中使用 `authorize` 方法。

`authorize` 方法会根据租户/产品配置的连接解析 SSO URL（`redirect_url`）。应用需将用户重定向至该 URL。请注意：SSO URL 结构因连接类型而异——SAML 连接会包含 `SAMLRequest`，而 OIDC 连接的 SSO URL 将是附带 OIDC 请求参数（scope、response_type 等）的授权端点。

<Tabs>
<TabItem value="01" label="Request" default>

```ts
await oauth.authorize({
  tenant: 'boxyhq',
  product: 'your-app',
  redirect_uri:
    '<app redirect URI to which Jackson sents back the authorization code after authentication>',
  state:
    '<opaque value from the app which will be returned back from Jackson, this is need to prevent CSRF attacks>',
  response_type: 'code',
  code_challenge:
    '<transformed value of code_verifier used to prevent interception of authorization_code in PKCE flow>',
  code_challenge_method:
    '<transformation method applied on code_verifier to generate code_challenge>',
  scope:
    '<can contain space separated values such as openid or even encoded tenant/product>',
  nonce:
    '<string value used to associate a client session with an ID Token in openid flow, and to mitigate replay attacks>',
  idp_hint:
    '<this will contain the clientID of the SSO connection that the user selects in the case of multiple ones configured for a tenant/product>',
  prompt: '<pass "login" to force authentication at the SAML IdP>',
});
```

</TabItem>

<TabItem value="02" label="Response">

```json
{
  "redirect_url": "https://mocksaml.com/api/saml/sso?RelayState=boxyhq_jackson_...&SAMLRequest=nVbZkqs4En33V1T4...",
  "authorize_form": ""
}
```

</TabItem>
</Tabs>

### 处理身份提供商响应

身份提供商完成认证后会将响应返回至您的应用。处理该响应后，系统会提取认证用户档案并关联至短期有效的代码返回给应用。请根据以下说明使用对应方法处理响应：

#### SAML 响应

处理来自SAML身份提供商的响应。认证成功后，IdP会通过浏览器POST方式将`SAMLResponse`和`RelayState`发送至应用的断言消费者服务(ACS) URL（即`samlPath`）。您需要在ACS端点内使用`samlResponse`方法。该方法将解析并验证SAML响应，随后提取用户资料。

<Tabs>
<TabItem value="01" label="Request" default>

```js
await oauth.samlResponse({
  SAMLResponse: '<SAML Response from the SAML IdP>',
  RelayState: '<Relaystate from the original SAML request to the IdP>',
});
```

</TabItem>

<TabItem value="02" label="Response">

```json
{
  "redirect_url": "https://your-app.com/sso/callback?code=5db7257fde94e062f6243572e31818d6e64c3097&state=c38ee339-6b82-43d3-838f-4036820acce9"
}
```

</TabItem>
</Tabs>

#### OIDC响应

处理来自OIDC身份提供商的响应。认证成功后，IdP会通过浏览器重定向方式将`code`和`state`发送至处理OIDC响应的重定向URL（即`oidcPath`）。您需要在`oidcPath`处理器内使用`oidcAuthzResponse`方法。该方法将通过OIDC提供商将`code`兑换为tokenSet（包含id_token和access_token），并调用OIDC提供商的"userinfo"端点。最终结合`id_token`声明和`userinfo`响应生成用户资料。

<Tabs>
<TabItem value="01" label="Request" default>

```js
await oauth.oidcAuthzResponse({
  code: '<code received from OIDC IdP after authentication>',
  state: '<state from the original OIDC request to the IdP>',
});
```

</TabItem>

<TabItem value="02" label="Response">

```json
{
  "redirect_url": "https://your-app.com/sso/callback?code=5db7257fde94e062f6243572e31818d6e64c3097&state=c38ee339-6b82-43d3-838f-4036820acce9"
}
```

</TabItem>
</Tabs>

### 请求访问令牌

通过传递上一步获取的授权`code`及其他认证信息，请求获取`access_token`。

<Tabs>
<TabItem value="01" label="Request" default>

```js
const tenant = 'boxyhq';
const product = 'your-app';

await oauth.token({
  code: '<Authorization code received from Jackson at redirect_url after login at IdP>',
  redirect_uri:
    '<redirect_uri used in original authorization request to Jackson>',
  client_id: `tenant=${tenant}&product=${product}`,
  client_secret: 'dummy',
  grant_type: 'authorization_code',
});
```

</TabItem>

<TabItem value="02" label="Response">

```json
{
  "access_token": "6b81f03b60c34e46e740d96c7e6242923736a2d1",
  "token_type": "bearer",
  "expires_in": 300
}
```

</TabItem>
</Tabs>

### 获取用户资料

获取`access_token`后，可用其从身份提供商处检索用户资料。

<Tabs>
<TabItem value="01" label="Request" default>

```js
const accessToken = '<Access token from code exchange step above>';

await oauth.userInfo(accessToken);
```

</TabItem>

<TabItem value="02" label="Response">

```json
{
  "raw": {
    "id": "1dda9fb491dc01bd24d2423ba2f22ae561f56ddf2376b29a11c80281d21201f9",
    "email": "samuel@example.com",
    "firstName": "Samuel",
    "lastName": "Jackson",
    "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier": "samuel@example.com"
  },
  "id": "1dda9fb491dc01bd24d2423ba2f22ae561f56ddf2376b29a11c80281d21201f9",
  "email": "samuel@example.com",
  "firstName": "Samuel",
  "lastName": "Jackson",
  "requested": {
    "client_id": "f7c909a5c72a5535847acf32558b2429a5172dd6",
    "state": "c38ee339-6b82-43d3-838f-4036820acce9",
    "redirect_uri": "https://your-app.com/sso/callback",
    "tenant": "boxyhq",
    "product": "your-app",
    "scope": []
  }
}
```

</TabItem>
</Tabs>
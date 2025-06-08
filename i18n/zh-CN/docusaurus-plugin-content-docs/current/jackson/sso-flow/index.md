import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# 单点登录（SSO）

Jackson采用多租户方式实现SSO，抽象了底层SAML/OIDC协议的所有复杂性。这意味着您可以通过单个Jackson实例为所有（您的）客户跨产品启用SSO，并且同时支持SAML和OIDC身份提供商（IdP）。

> **注意：** 以下所有API均支持`application/x-www-form-urlencoded`和`application/json`内容类型。下文示例使用`application/x-www-form-urlencoded`。

> **注意：** OAuth 2.0协议使用下划线命名参数，而我们的API采用驼峰命名。例如OAuth 2.0流程中使用`client_id`，而我们的API中使用`clientID`。

## 1. 配置SSO提供商

请按照[此文档](./sso-providers)指导您的客户为产品正确配置SAML/OIDC。您应复制该文档并根据您的自定义设置进行修改，我们使用了适用于演示应用的默认值。

## 2. SSO连接API

您需要在UI中为您的客户提供配置入口（通常账户设置页面是理想位置），然后调用以下API。

### 2.1 添加连接

<Tabs>
<TabItem value="saml" label="SAML" default>

Once your customer has set up the SAML app on their Identity Provider, the Identity Provider will generate an IdP or SP metadata file. Some Identity Providers only generate an IdP metadata file but it usually works for the SP login flow as well. It is an XML file that contains various attributes Jackson needs to validate incoming SAML login requests. This step is the equivalent of setting an OAuth 2.0 app and generating a client ID and client secret that will be used in the login flow.

The following API call sets up the connection in Jackson:

```bash
curl --location --request POST 'http://localhost:5225/api/v1/connections' \
--header 'Authorization: Api-Key <Jackson API Key>' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'encodedRawMetadata=Base64(<IdP/SP metadata XML>)' \
--data-urlencode 'defaultRedirectUrl=http://localhost:3366/login/saml' \
--data-urlencode 'redirectUrl=http://localhost:3366/*' \
--data-urlencode 'redirectUrl=http://localhost:3000/*' \
--data-urlencode 'tenant=boxyhq.com' \
--data-urlencode 'product=demo' \
--data-urlencode 'name=demo-connection' \
--data-urlencode 'description=Demo SAML connection'
```

- `metadataUrl`: URL containing the SAML metadata contents. Either this or `encodedRawMetadata` needs to be specified
- `encodedRawMetadata`: Base64 encoding of the XML metadata your customer gets from their Identity Provider. Either this or `metadataUrl` needs to be specified
- `defaultRedirectUrl`: The redirect URL to use in the IdP login flow. Jackson will call this URL after completing an IdP login flow
- `redirectUrl`: Allowed redirect URL. Repeat this field multiple times to allow multiple redirect URLs. Jackson will disallow any redirects not on this list (or not the default URL above).
- `tenant`: Jackson supports a multi-tenant architecture, this is a unique identifier you set from your side that relates back to your customer's tenant. This is normally an email, domain, an account id, or user-id. **Should not contain the : character since we use it as a delimiter internally**
- `product`: Jackson support multiple products, this is a unique identifier you set from your side that relates back to the product your customer is using. **Should not contain the : character since we use it as a delimiter internally**
- `name`: A friendly name to identify the SAML connection
- `description`: A short description with some information of the connection

The response returns a JSON with `clientID` and `clientSecret` that can be stored against your tenant and product for a more secure OAuth 2.0 flow. If you do not want to store the `clientID` and `clientSecret` you can alternatively use `client_id=tenant=<tenantID>&product=<productID>` and use `dummy` (or the value you set for the [secret verifier](../deploy/env-variables.md#client_secret_verifier) env) as the value for `client_secret` when setting up the OAuth 2.0 flow. Additionally a `idpMetadata.provider` attribute is also returned which indicates the domain of your Identity Provider.
</TabItem>
<TabItem value="oidc" label="OIDC">

Once your customer has set up the [OIDC app](../sso-providers/generic-oidc.md) on their Identity Provider, the Identity Provider will generate a clientId and clientSecret.

The following API call sets up the connection in Jackson:

```bash
curl --location --request POST 'http://localhost:5225/api/v1/connections' \
--header 'Authorization: Api-Key <Jackson API Key>' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'defaultRedirectUrl=http://localhost:3366/login/oidc' \
--data-urlencode 'oidcDiscoveryUrl=<well-known url of OIDC Provider>' \
--data-urlencode 'oidcClientId=<clientId of IdP Registered App>' \
--data-urlencode 'oidcClientSecret=<clientSecret of IdP Registered App>' \
--data-urlencode 'redirectUrl=http://localhost:3366/*' \
--data-urlencode 'redirectUrl=http://localhost:3000/*' \
--data-urlencode 'tenant=boxyhq.com' \
--data-urlencode 'product=demo' \
--data-urlencode 'name=demo-connection' \
--data-urlencode 'description=Demo OIDC connection'
```

- `oidcDiscoveryUrl`: OpenID Providers supporting discovery make the metadata available at the endpoint obtained by concatenating issuer and `/.well-known/openid-configuration`
- `oidcClientId`: The client identifier issued to the client during the IdP registration process.
- `oidcClientSecret`: The client secret issued to the client during the IdP registration process.
- `defaultRedirectUrl`: The redirect URL to use in the IdP login flow. Jackson will call this URL after completing an IdP login flow
- `redirectUrl`: Allowed redirect URL. Repeat this field multiple times to allow multiple redirect URLs. Jackson will disallow any redirects not on this list (or not the default URL above).
- `tenant`: Jackson supports a multi-tenant architecture, this is a unique identifier you set from your side that relates back to your customer's tenant. This is normally an email, domain, an account id, or user-id. **Should not contain the : character since we use it as a delimiter internally**
- `product`: Jackson support multiple products, this is a unique identifier you set from your side that relates back to the product your customer is using. **Should not contain the : character since we use it as a delimiter internally**
- `name`: A friendly name to identify the OIDC connection
- `description`: A short description with some information of the connection

The response returns a JSON with `clientID` and `clientSecret` that can be stored against your tenant and product for a more secure OAuth 2.0 flow. If you do not want to store the `clientID` and `clientSecret` you can alternatively use `client_id=tenant=<tenantID>&product=<productID>` and use `dummy` (or the value you set for the [secret verifier](../deploy/env-variables.md#client_secret_verifier) env) as the value for `client_secret` when setting up the OAuth 2.0 flow. Additionally a `idpMetadata.provider` attribute is also returned which indicates the domain of your Identity Provider.
</TabItem>
</Tabs>

### 2.2 获取连接

此端点可用于检索为租户/产品配置的SAML/OIDC连接，用于向客户展示配置详情。您可以使用`clientID`或`tenant`与`product`组合进行查询。

<Tabs>
<TabItem value="saml" label="SAML" default>

```bash
curl -G --location 'http://localhost:5225/api/v1/connections' \
--header 'Authorization: Api-Key <Jackson API Key>' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'tenant=boxyhq.com' \
--data-urlencode 'product=demo'
```

```bash
curl -G --location 'http://localhost:5225/api/v1/connections' \
--header 'Authorization: Api-Key <Jackson API Key>' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'clientID=<Client ID>'
```

The response returns a JSON with `idpMetadata.provider`indicating the domain of your Identity Provider. If an empty JSON payload is returned then we do not have any connection stored for the attributes you requested.
</TabItem>
<TabItem value="oidc" label="OIDC">

```bash
curl -G --location 'http://localhost:5225/api/v1/connections' \
--header 'Authorization: Api-Key <Jackson API Key>' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'tenant=boxyhq.com' \
--data-urlencode 'product=demo'
```

```bash
curl -G --location 'http://localhost:5225/api/v1/connections' \
--header 'Authorization: Api-Key <Jackson API Key>' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'clientID=<Client ID>'
```

The response returns a JSON with `oidcProvider.provider` indicating the domain of your Identity Provider. If an empty JSON payload is returned then we do not have any connection stored for the attributes you requested.
</TabItem>
</Tabs>

### 2.3 更新连接

此端点用于更新现有SSO连接配置。

<Tabs>
<TabItem value="saml" label="SAML" default>

```bash
curl --location --request PATCH 'http://localhost:5225/api/v1/connections' \
--header 'Authorization: Api-Key <Jackson API Key>' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'clientID=<Client ID>' \ /* Required */
--data-urlencode 'clientSecret=<Client Secret>' \ /* Required */
--data-urlencode 'encodedRawMetadata=Base64(<IdP/SP metadata XML>)' \
--data-urlencode 'defaultRedirectUrl=http://localhost:3366/login/saml' \
--data-urlencode 'redirectUrl=http://localhost:3366/*' \
--data-urlencode 'redirectUrl=http://localhost:3000/*' \
--data-urlencode 'tenant=boxyhq.com' \ /* Required */
--data-urlencode 'product=demo' \ /* Required */
--data-urlencode 'name=demo-connection' \
--data-urlencode 'description=Demo SAML connection'
```

</TabItem>
<TabItem value="oidc" label="OIDC" default>

```bash
curl --location --request PATCH 'http://localhost:5225/api/v1/connections' \
--header 'Authorization: Api-Key <Jackson API Key>' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'clientID=<Client ID>' \ /* Required */
--data-urlencode 'clientSecret=<Client Secret>' \ /* Required */
--data-urlencode 'oidcDiscoveryUrl=<well-known url of OIDC Provider>' \
--data-urlencode 'oidcClientId=<clientId of IdP Registered App>' \
--data-urlencode 'oidcClientSecret=<clientSecret of IdP Registered App>' \
--data-urlencode 'defaultRedirectUrl=http://localhost:3366/login/oidc' \
--data-urlencode 'redirectUrl=http://localhost:3366/*' \
--data-urlencode 'redirectUrl=http://localhost:3000/*' \
--data-urlencode 'tenant=boxyhq.com' \ /* Required */
--data-urlencode 'product=demo' \ /* Required */
--data-urlencode 'name=demo-connection' \
--data-urlencode 'description=Demo OIDC connection'
```

</TabItem>
</Tabs>

### 2.4 删除连接

此端点可通过租户/产品组合或clientID删除现有连接。

```bash
curl -X "DELETE" --location 'http://localhost:5225/api/v1/connections' \
--header 'Authorization: Api-Key <Jackson API Key>' \
--url-query 'tenant=boxyhq.com' \
--url-query 'product=demo'
```

```bash
curl -X "DELETE" --location 'http://localhost:5225/api/v1/connections' \
--header 'Authorization: Api-Key <Jackson API Key>' \
--url-query 'clientID=<Client ID>'
--url-query 'clientSecret=<Client Secret>'
```

## 3. OAuth 2.0流程

:::info

#### OpenID Connect流程

Jackson同时支持[OIDC流程](https://openid.net/specs/openid-connect-core-1_0.html)。当`scope`参数包含`openid`时，令牌端点将额外返回包含用户声明（`id, email, firstName, lastName`）的`id_token`。要启用此流程，请确保在[OpenID配置](../deploy/env-variables.md#openid-configuration)中配置密钥和算法。若认证请求包含`nonce`参数，该值将原样传递至ID Token供客户端验证，防止重放攻击。
:::

Jackson的设计理念是将底层SAML/OIDC登录流程抽象为纯OAuth 2.0流程，这意味着其兼容所有标准OAuth 2.0库（包括客户端和服务器端）。需特别注意：SSO连接是按客户配置的，这与OAuth 2.0中单个应用可支持所有客户登录的模式不同。

Jackson还支持[PKCE授权流程](https://oauth.net/2/pkce/)，可有效保护单页应用（SPA）。

如需自行实现该流程，步骤如下：

### 3.1 授权

OAuth流程始于将用户重定向至`authorize` URL：

```bash
https://localhost:5225/api/oauth/authorize
  ?response_type=code&provider=saml
  &client_id=<clientID or tenant and product query params as described in the SSO connection API section above>
  &redirect_uri=<redirect URL>
  &state=<randomly generated state id>
```

- `response_type`：目前仅支持`code`类型，未来可能会扩展
- `client_id`：使用SSO连接API返回的client_id，或通过`tenant=<tenantID>&product=<productID>`直接使用租户和产品ID。**注意：**请勿忘记对包括`client_id`在内的查询参数进行URL编码。
- `tenant`：可选方案，若OAuth 2.0库支持，可将`client_id`设为`dummy`并通过此参数指定自定义租户属性
- `product`：若指定了`tenant`则必须同时指定此参数
- `idp_hint`：当同一`tenant/product`匹配多个连接时，用于选择身份提供商。应指向Jackson中SSO连接的绝对"clientID"
- `redirect_uri`：授权流程完成后用户将被重定向至此URI
- `state`：使用随机生成的字符串作为状态值，该值将作为查询参数回传至`redirect_uri`。应验证此值以防止XSRF攻击
- `nonce`（OIDC流程专用）：用于关联客户端会话与ID令牌的字符串值，可防范重放攻击。该值从认证请求原样传递至ID令牌，必须包含足够的随机性防止攻击者猜测
- `forceAuthn`（SAML SSO连接专用）：当值为`true`时，发往IdP的SAML请求将强制设置`ForceAuthn`参数，即使用户已有活跃会话也需重新认证
- `login_hint`（适用于Google等OIDC IdP）：该参数将被转发至OIDC IdP的授权请求，IdP可据此值（通常为邮箱等账户标识）跳过账户选择页面

> **注意**：您也可将编码后的tenant/product通过`scope`、`access_type`或`resource`参数传递（需将`client_id`设为`dummy`）。这在某些无法动态设置client_id的场景中特别实用。

用户将被引导至基于配置的SAML/OIDC元数据的IdP。若出现错误，系统会将`error`、`error_description`及原始请求中的`state`参数（参见[错误响应](https://www.oauth.com/oauth2-servers/authorization/the-authorization-response/)）返回至`redirect_uri`（为防止开放重定向，`redirect_uri`会与saml连接进行匹配验证）。

### 3.2 授权码兑换

当用户在IdP成功登录后，IdP会向Jackson返回SAML/OIDC响应。SAML响应包含用户档案，OIDC响应则包含授权码（Jackson用此码从OIDC IdP获取令牌和用户档案）。Jackson会生成短期有效的`code`并关联用户档案，授权成功后用户被重定向至`redirect_uri`，其查询参数将包含`code`和`state`。您需验证该state是否与授权请求中的初始值匹配。

随后可通过以下请求将授权码兑换为令牌：

```bash
curl --request POST \
  --url 'http://localhost:5225/api/oauth/token' \
  --header 'content-type: application/x-www-form-urlencoded' \
  --data 'grant_type=authorization_code' \
  --data 'client_id=<clientID or tenant and product query params as described in the SAML connection API section above>' \
  --data 'client_secret=<clientSecret or clientSecretVerifier if set (which otherwise defaults to "dummy")' \
  --data 'redirect_uri=<redirect URL>' \
  --data 'code=<code from the query parameter above>'
```

- `grant_type`：目前仅支持`authorization_code`流程，未来可能扩展
- `client_id`：使用SSO连接API返回的clientID，或通过`tenant=<tenantID>&product=<productID>`使用租户和产品ID组合。**注意：**请勿忘记对包括`client_id`在内的查询参数进行URL编码
- `client_secret`：使用SAML连接API返回的clientSecret，若通过tenant和product指定clientID则使用clientSecretVerifier（默认为"dummy"）
- `redirect_uri`：授权流程完成后用户将被重定向至此URI，需与先前请求保持一致

若一切正常，您将收到包含访问令牌的JSON响应。此令牌是下一步获取用户档案的必要凭证。

```json
{
  "access_token": <access token>,
  "token_type": "bearer",
  "expires_in": 300
}
```

### 3.3 用户资料请求

此时，短期有效的访问令牌可用于请求用户资料。您需要发起以下请求：

```bash
curl --request GET \
  --url 'https://localhost:5225/api/oauth/userinfo' \
  --header 'authorization: Bearer <access token>'
```

如果一切顺利，您将收到包含用户资料的JSON响应：

```json
{
  "id": <id from the Identity Provider>,
  "email": "sjackson@coolstartup.com",
  "firstName": "SAML",
  "lastName": "Jackson"
  "raw": {
    ...
  },
  "requested": {
    ...
  }
}
```

- `id`: 由身份提供商提供的用户唯一标识（SAML单点登出请求要求将`id`作为`nameId`传递。若未来计划实现SLO功能，请保存此`id`值）
- `email`: 身份提供商提供的用户邮箱地址
- `firstName`: 身份提供商提供的用户名字
- `lastName`: 身份提供商提供的用户姓氏
- `raw`: 包含SAML提供商返回的所有声明属性
- `requested`: 包含授权请求中的`tenant`、`product`、`client_id`和`state`参数，可用于在客户端按需重建上下文

## 4. SAML单点登出(SLO)

单点登出(SLO)是一种SAML流程，允许终端用户登出单个会话时，自动注销SSO过程中创建的所有关联会话。

SLO流程始于将用户重定向至`/api/logout`端点：

```bash
https://localhost:5225/api/logout
  ?nameId=google-auth0-f92cc1834efc0f73e9c09
  &tenant=boxyhq.com
  &product=demo
  &redirectUrl=http://www.example.com/logout
```

- `nameId`: 标识SAML断言的主体（通常为已认证的用户）
- `tenant`: 租户标识
- `product`: 产品标识
- `redirectUrl`: 登出后重定向URL

身份提供商(IdP)会将登出请求的响应（通过POST操作）发送至特定URL（单点登出URL）。您需要在IdP上注册该URL `/api/logout/callback` 以处理响应。

登出成功后，Jackson会将用户重定向至`redirectUrl`。

:::info
建议保存`userinfo`接口返回的用户`id`，以便在SLO请求中将其作为`nameId`参数使用。
:::
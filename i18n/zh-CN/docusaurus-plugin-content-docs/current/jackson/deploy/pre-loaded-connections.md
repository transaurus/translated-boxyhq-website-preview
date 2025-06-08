# 预加载SSO连接

#### SAML

若设置了PRE_LOADED_CONNECTION环境变量，其应指向具有以下结构的目录（示例如下）：-

```bash
boxyhq.js
boxyhq.xml
anothertenant.js
anothertenant.xml
```

JS文件需遵循以下结构：-

```javascript
module.exports = {
  defaultRedirectUrl: 'http://localhost:3366/login/saml',
  redirectUrl: '["http://localhost:3366/*"]',
  tenant: 'boxyhq.com',
  product: 'demo',
  name: 'testConnection',
  description: 'Just a test connection',
};
```

XML文件（需与.js文件同名）是从身份提供商获取的原始元数据文件，请确保以utf-8编码保存。

上述配置及XML文件对应[添加连接API-SAML标签页](../sso-flow/index.md#21-add-connection)。

#### OIDC

对于基于OpenID身份提供商的连接，请在PRE_LOADED_CONNECTION路径下创建如下所示的*.js文件：-

参见[添加连接API-OIDC标签页](../sso-flow/index.md#21-add-connection)

```javascript
module.exports = {
  defaultRedirectUrl: 'http://localhost:3366/sso/oauth/oidc',
  redirectUrl: '["http://localhost:3366"]',
  tenant: 'oidc.example.com',
  product: 'crm',
  name: 'OIDC connection for oidc.example.com',
  description: 'OIDC connection for oidc.example.com',
  oidcDiscoveryUrl:
    'https://accounts.google.com/.well-known/openid-configuration',
  oidcClientId: '<ClientId from the OpenID IdP>',
  oidcClientSecret: '<ClientSecret from the OpenID IdP>',
};
```
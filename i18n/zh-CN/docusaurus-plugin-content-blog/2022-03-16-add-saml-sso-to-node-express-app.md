---
slug: add-saml-sso-to-node-express-app
title: How to add SAML Single Sign On to an Express app
author: Kiran K
author_title: Senior Developer @BoxyHQ
author_url: https://twitter.com/tokirankrishnan
author_image_url: /images/authors/kiran.jpg
tags_disabled:
  [enterprise-readiness, engineering, saml, saml-jackson, integrations, sso]
---

本文将指导您如何为Express.js应用添加SAML单点登录功能。您将使用[SAML Jackson](https://boxyhq.com/docs/jackson/overview)与[Auth0](https://auth0.com/single-sign-on)实现用户认证和路由保护。

完整代码可访问[GitHub仓库](https://github.com/boxyhq/express-jackson-auth0-demo)获取。

让我们开始吧！

## 先决条件

跟随本文操作需要满足以下条件：

- 已安装Node.js运行环境
- 具备Node.js和Express.js基础知识

## 数据库配置

本文将使用免费的[Heroku Postgres数据库](https://devcenter.heroku.com/articles/heroku-postgresql)替代本地Postgres服务。

- 访问[Heroku注册页面](https://signup.heroku.com/login)创建账户
- 进入[应用控制台](https://dashboard.heroku.com/apps)点击**创建新应用**
- 输入应用名称后点击**创建应用**按钮
- 切换到**资源**标签页
- 在插件搜索框选择**Heroku Postgres**并提交订单
- 点击**Heroku Postgres**进入**设置**标签页
- 点击**查看凭证**按钮复制**URI**连接字符串

至此您已创建免费PostgreSQL数据库并获取连接URI，后续步骤将使用该连接字符串。

## 身份提供商配置

我们将使用Auth0作为身份提供商(IdP)，该服务负责管理应用的用户账户体系。

- 首先访问[Auth0注册页面](https://auth0.com/signup)创建账户
- 进入[仪表盘 > 应用 > 应用程序](https://manage.auth0.com/dashboard/)
- 点击**创建应用程序**按钮
- 输入新应用名称
- 选择**常规Web应用**类型后点击**创建**
- 进入创建的应用，点击**插件**标签页
- 在**SAML2 Web应用**模块启用插件开关
- 切换到**使用**标签页下载**身份提供商元数据**
- 进入**设置**标签页进行以下配置：
- 添加`http://localhost:3000/sso/acs`作为接收SAML响应的**应用回调URL**
- 粘贴以下JSON配置后点击**启用**按钮

```json
{
  "audience": "https://saml.boxyhq.com",
  "mappings": {
    "id": "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier",
    "email": "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress",
    "firstName": "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname",
    "lastName": "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname"
  }
}
```

`audience`参数用于验证SAML受众标识符。[了解更多](https://boxyhq.com/docs/jackson/deploy/env-variables#saml_audience)

Auth0提供数据库连接方式，支持通过邮箱/用户名和密码认证用户，这些凭证会安全存储在Auth0用户库中。

现在让我们创建用户库以便用户注册登录：

- 进入[Auth0仪表盘 > 认证 > 数据库](https://manage.auth0.com/dashboard/)
- 点击**创建数据库连接** - [Auth0文档](https://auth0.com/docs/authenticate/database-connections/custom-db/create-db-connection)
- 输入连接名称后点击**创建**
- 切换到**应用**标签页启用刚创建的应用

完成以上准备后，我们进入下一步操作。

## 快速开始

打开终端克隆GitHub仓库：

```bash
git clone https://github.com/devkiran/express-saml.git
```

```bash
cd express-saml
```

安装项目依赖：

```bash
npm install
```

配置环境变量：

```bash
cp .env.example .env
```

将 `DATABASE_URL` 变量更新为您的 Heroku Postgres 数据库连接 URI。

在数据库连接 URI 后追加 `?sslmode=no-verify`，否则 Heroku 不允许连接到数据库。这是 Heroku 的特定配置。

例如 `postgres://hcydrtasctfyth:fe001b264322d6cf794@ec2-1-2-3-4.compute-1.amazonaws.com:5432/demo?sslmode=no-verify`

## 关于 Express 应用

这是一个使用 `express-generator` 创建的简单 express.js 应用。您也可以使用任何其他 express.js 应用。

我们的 express.js 应用仅包含 2 个路由。

- `GET /` 渲染首页
- `GET /dashboard` 渲染仪表盘

那么计划是什么？我们将为 express.js 应用添加 SAML SSO 登录（通过 Auth0），以便只有经过身份验证的用户才能访问 `/dashboard`。

## 安装 SAML Jackson

运行以下命令安装最新版本的 SAML Jackson。

```bash
npm i --save @boxyhq/saml-jackson
```

安装 Jackson 后，让我们初始化它。

将以下代码添加到 `routes/index.js`。

```javascript
// routes/index.js

...

let apiController;
let oauthController;

const jacksonOptions = {
  externalUrl: process.env.APP_URL,
  samlAudience: process.env.SAML_AUDIENCE,
  samlPath: '/sso/acs',
  db: {
    engine: 'sql',
    type: 'postgres',
    url: process.env.DATABASE_URL,
  },
};

(async function init() {
  const jackson = await require('@boxyhq/saml-jackson').controllers(jacksonOptions);

  apiController = jackson.apiController;
  oauthController = jackson.oauthController;
})();
```

## 设置 Express.js 路由

### 添加 SAML 元数据

您将创建的第一个路由是 `GET /config`。此路由将显示一个包含以下字段的表单：

- `Metadata`：输入您从 IdP 下载的 XML 元数据内容。
- `Tenant`：Jackson 支持多租户架构，这是您设置的唯一标识符，与您的客户租户相关联。通常是电子邮件、域名、账户 ID 或用户 ID。
- `Product`：Jackson 支持多产品，这是您设置的唯一标识符，与您的客户正在使用的产品相关联。

```javascript
// routes/index.js

router.get('/config', async (req, res) => {
  res.render('config');
});
```

添加一个视图来显示表单。

```html
<!-- views/config.ejs -->

<!DOCTYPE html>
<html>
  <head>
    <title>SAML Config</title>
    <link
      rel="stylesheet"
      href="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0/css/bootstrap.min.css"
      crossorigin="anonymous"
    />
    <link rel="stylesheet" href="/stylesheets/style.css" />
  </head>
  <body>
    <h1>SAML Config</h1>
    <p>Add SAML Metadata.</p>
    <form action="/config" method="POST">
      <div class="form-group">
        <label for="tenant">Tenant</label>
        <input
          type="text"
          name="tenant"
          id="tenant"
          class="form-control col-md-6"
          required="required"
        />
      </div>
      <div class="form-group">
        <label for="product">Product</label>
        <input
          type="text"
          name="product"
          id="product"
          class="form-control col-md-6"
          required="required"
        />
      </div>
      <div class="form-group">
        <label for="rawMetadata">Metadata (Raw XML)</label>
        <textarea
          name="rawMetadata"
          id="rawMetadata"
          cols="30"
          rows="10"
          class="form-control col-md-6"
          required="required"
        ></textarea>
      </div>
      <button type="submit" class="btn btn-primary">Submit</button>
    </form>
  </body>
</html>
```

现在让我们添加另一个路由 `POST /config`，它将通过调用 SAML Jackson 配置 API 来存储表单数据。

此步骤相当于设置 OAuth 2.0 应用并生成将在登录流程中使用的客户端 ID 和客户端密钥。

```javascript
// routes/index.js

router.post('/config', async (req, res, next) => {
  const { rawMetadata, tenant, product } = req.body;

  const defaultRedirectUrl = 'http://localhost:3000/sso/callback';
  const redirectUrl = '["http://localhost:3000/*"]';

  try {
    await apiController.config({
      rawMetadata,
      tenant,
      product,
      defaultRedirectUrl,
      redirectUrl,
    });

    res.redirect('/config');
  } catch (err) {
    next(err);
  }
});
```

上面的代码中有几点需要注意。

`defaultRedirectUrl` 保存了在 IdP 登录流程中使用的重定向 URL。Jackson 将在完成 IdP 登录流程后调用此 URL。

`redirectUrl` 保存了一个包含允许重定向 URL 列表的数组。Jackson 将禁止任何不在此列表中的重定向。

接下来，让我们启动 express 应用。应用默认启动一个服务器并监听端口 3000 的连接。

```bash
npm start
```

现在，访问 [http://localhost:3000/config](http://localhost:3000/config)，您应该会看到一个表单页面。

![SAML 配置](/img/blog/add-saml-sso-to-node-express-app/add-saml-config-form.png)

在这里，您可以添加从 Auth0 下载的元数据。填写表单中的 Tenant、Product，并原样粘贴元数据 XML 内容。

我将使用 'boxyhq.com' 作为租户，'crm' 作为产品。

响应返回一个包含 `client_id` 和 `client_secret` 的 JSON，可以针对您的租户和产品存储，以实现更安全的 OAuth 2.0 流程。

若您不希望存储 `client_id` 和 `client_secret`，可在配置OAuth 2.0流程时使用 `client_id=tenant=<tenantID>&product=<productID>` 格式，并为 `client_secret` 填写任意值作为替代方案。

### 将用户重定向至身份提供商(IdP)

添加SAML元数据后，需创建路由将用户重定向至IdP以启动SAML认证流程。

现在添加新路由 `GET /sso/authorize`。

请务必修改代码中的租户(tenant)和产品(product)值。

```javascript
// routes/index.js

router.get('/sso/authorize', async (req, res, next) => {
  try {
    const tenant = 'boxyhq.com';
    const product = 'crm';

    const body = {
      response_type: 'code',
      client_id: `tenant=${tenant}&product=${product}`,
      redirect_uri: 'http://localhost:3000/sso/callback',
      state: 'a-random-state-value',
    };

    const { redirect_url } = await oauthController.authorize(body);

    res.redirect(redirect_url);
  } catch (err) {
    next(err);
  }
});
```

`oauthController.authorize()` 将返回一个 `redirect_url`。您应将用户重定向至此URL以启动IdP认证流程。

### 处理来自IdP的SAML响应

此路由将成为应用的断言消费者服务(ACS) URL。ACS URL用于告知IdP在用户认证后向何处POST其SAML响应。

SAML响应包含两个字段：`SAMLResponse` 和 `RelayState`。

```javascript
// routes/index.js

router.post('/sso/acs', async (req, res, next) => {
  try {
    const { SAMLResponse, RelayState } = req.body;

    const body = {
      SAMLResponse,
      RelayState,
    };

    const { redirect_url } = await oauthController.samlResponse(body);

    res.redirect(redirect_url);
  } catch (err) {
    next(err);
  }
});
```

调用方法 `oauthController.samlResponse()` 将返回一个 `redirect_url`。您应将用户重定向至此URL，其查询参数将包含 `code` 和 `state`。

### 授权码交换

现在将 `code` 交换为 `token`，该令牌用于获取用户资料。

创建新路由 `GET /sso/callback` 来处理回调。

```javascript
// routes/index.js

router.get('/sso/callback', async (req, res, next) => {
  const { code } = req.query;

  const tenant = 'boxyhq.com';
  const product = 'crm';

  const body = {
    code,
    client_id: `tenant=${tenant}&product=${product}`,
    client_secret: 'client_secret',
  };

  try {
    // Get the access token
    const { access_token } = await oauthController.token(body);

    // Get the user information
    const profile = await oauthController.userInfo(access_token);

    // Add the profile to the express session
    req.session.profile = profile;

    res.redirect('/dashboard');
  } catch (err) {
    next(err);
  }
});
```

在上述代码中，请替换 `tenant` 和 `product` 的值为您自己的配置。

### 保护仪表板路由

现在需要修改 `GET /dashboard` 路由，确保仅认证用户可访问。

通过添加条件检查会话中是否存在 `profile` 来实现。

若 `profile` 为 `undefined`，则将用户重定向至 `/`，否则在仪表板显示用户资料。

用以下代码替换原 `GET /dashboard` 路由。

```javascript
// routes/index.js

router.get('/dashboard', function (req, res, next) {
  const { profile } = req.session;

  if (profile === undefined) {
    return res.redirect('/');
  }

  // Pass the profile to the view
  res.render('dashboard', {
    profile,
  });
});
```

用以下代码替换 `views/dashboard.ejs` 视图文件。

```html
<!-- views/dashboard.ejs -->

<!DOCTYPE html>
<html>
  <head>
    <title>Dashboard</title>
    <link rel="stylesheet" href="/stylesheets/style.css" />
  </head>
  <body>
    <h1>Dashboard</h1>
    <p>Only authenticated users should access this page.</p>

    <p>Id - <%= profile.id %></p>
    <p>Email - <%= profile.email %></p>
  </body>
</html>
```

从命令行重启Express应用，然后访问授权URL [http://localhost:3000/sso/authorize](http://localhost:3000/sso/authorize)。

若配置正确，系统将重定向至Auth0认证页面，点击注册链接完成注册。

认证成功后，应用将重定向至仪表板并显示用户的 `id` 和 `email`。

![仪表板](/img/blog/add-saml-sso-to-node-express-app/express-dashboard.png)

## 结语

恭喜！您已成功通过SAML Jackson和Auth0为Express.js应用集成SAML单点登录功能。

## 参考资源

了解更多SAML Jackson相关信息，请查阅以下资源：

- [SAML Jackson官方文档](https://boxyhq.com/docs/jackson/overview)
- [SAML Jackson GitHub仓库](https://github.com/boxyhq/jackson)
- [演示SAML Jackson用法的Express示例应用](https://github.com/boxyhq/jackson-examples/tree/main/apps/express)

欢迎提交反馈与贡献！
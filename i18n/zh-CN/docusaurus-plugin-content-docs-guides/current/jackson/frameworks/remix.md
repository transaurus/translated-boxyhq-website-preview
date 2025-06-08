---
title: Add SAML SSO to Remix App with BoxyHQ
description: Add SAML SSO to Remix App with BoxyHQ
sidebar_label: Remix
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# 为Remix应用集成SAML单点登录

本文将介绍如何使用企业级单点登录(SSO)在Remix应用中实现用户认证。

如需直接查看源码，请访问：https://github.com/boxyhq/jackson-remix-auth

:::info
本指南演示如何通过嵌入Jackson实现单点登录(SSO)功能，因此无需单独部署Jackson服务。
:::

## 开始准备

### 创建Remix应用

```bash
npx create-remix@latest
```

可选择Remix应用服务器作为部署目标。根据偏好选择TypeScript或JavaScript，本指南所有代码示例均采用TypeScript。

### 安装依赖项

```bash
npm i remix-auth @boxyhq/remix-auth-sso
```

[`remix-auth`](https://github.com/sergiodxa/remix-auth)是Remix应用的全功能开源认证方案。[`@boxyhq/remix-auth-sso`](https://github.com/boxyhq/remix-auth-sso)提供了与SSO服务提供商交互的remix-auth策略。

## 配置remix-auth

首先需要从`remix-auth`获取`Authenticator`实例，该实例提供登录/注销的API接口。

### 为`Authenticator`创建会话存储

app/sessions.server.ts: https://github.com/boxyhq/jackson-remix-auth/blob/main/app/sessions.server.ts

> **注意：我们将采用基于Cookie的会话机制，参见remix的[createCookieSessionStorage](https://remix.run/docs/en/v1/api/remix#createcookiesessionstorage)方法。**

```typescript
import { createCookieSessionStorage } from 'remix';

const sessionStorage = createCookieSessionStorage({
  cookie: {
    name: '__session',
    httpOnly: true,
    path: '/',
    sameSite: 'lax',
    secrets: process.env.COOKIE_SECRETS!.split(','),
    secure: process.env.NODE_ENV === 'production',
  },
});

const { getSession, commitSession, destroySession } = sessionStorage;
const JACKSON_ERROR_COOKIE_KEY = 'jackson_error';

export default sessionStorage;
export { getSession, commitSession, destroySession, JACKSON_ERROR_COOKIE_KEY };
```

### 创建`Authenticator`实例

app/auth.server.ts: https://github.com/boxyhq/jackson-remix-auth/blob/main/app/auth.server.ts

> **注意：策略初始化将在后续章节完成**

```typescript
import { Authenticator } from 'remix-auth';
import {
  BoxyHQSSOStrategy,
  type BoxyHQSSOProfile,
} from '@boxyhq/remix-auth-sso';
import invariant from 'tiny-invariant';
import sessionStorage from './sessions.server';

let auth: Authenticator;
declare global {
  var __auth: Authenticator | undefined;
}

function createAuthenticator() {
  const auth = new Authenticator<BoxyHQSSOProfile>(sessionStorage);

  // Strategy use for the hosted saml service provider goes here

  // Strategy use for the embedded saml service provider goes here

  return auth;
}

if (process.env.NODE_ENV === 'production') {
  auth = createAuthenticator();
} else {
  //  In development we don't want to recreate the Authenticator for every change
  if (!global.__auth) {
    global.__auth = createAuthenticator();
  }
  auth = global.__auth;
}

export { auth };
```

## 配置Jackson SSO功能

我们将使用SAML Jackson npm包来建立API路由（在Remix中称为[资源路由](https://remix.run/docs/en/main/guides/resource-routes)）。

### 安装jackson

首先安装`@boxyhq/saml-jackson`：

```bash
npm i @boxyhq/saml-jackson
```

继续操作前，请为jackson配置[数据库](../../../docs/jackson/deploy/service#database)，参考[npm库的数据库环境变量配置](../../../docs/jackson/deploy/env-variables#database-configuration)。

### 配置`JacksonProvider`

app/auth.jackson.server.ts: https://github.com/boxyhq/jackson-remix-auth/blob/main/app/auth.jackson.server.ts

> **注意：下方设置的[clientSecretVerifier](../../../docs/jackson/deploy/env-variables#client_secret_verifier)将与Authenticator传入的client_secret进行匹配 &nbsp;**

```typescript
 const opts =  {
  ...
   db: {
    engine: "sql",
    url: "postgresql://postgres:postgres@localhost:5432/postgres",
    type: "postgres",
  },
  clientSecretVerifier: process.env.CLIENT_SECRET_VERIFIER,
  ...
 }

 ...

 async function JacksonProvider({
   appBaseUrl,
 }: {
   appBaseUrl: string;
 }): Promise<{
   connectionAPIController: IConnectionAPIController;
   oauthController: IOAuthController;
 }> {
   const _opts = { ...opts, externalUrl: appBaseUrl, samlAudience: appBaseUrl };
   // this is needed because in development we don't want to restart
   // the server with every change, but we want to make sure we don't
   // create a new connection to the DB with every change either.
   if (process.env.NODE_ENV === "production") {
     const controllers = await jackson(_opts);
     connectionAPIController = controllers.connectionAPIController;
     oauthController = controllers.oauthController;
   } else {
     if (!global.__connectionAPIController && !global.__oauthController) {
       const controllers = await jackson(_opts);
       global.__connectionAPIController = controllers.connectionAPIController;
       global.__oauthController = controllers.oauthController;
     }
     connectionAPIController = global.__connectionAPIController;
     oauthController = global.__oauthController;
   }

   return { connectionAPIController, oauthController };
 }

 ...

```

### 处理OAuth 2.0和SSO连接的资源路由

接下来为[OAuth 2.0流程](../../../docs/jackson/sso-flow/#3-oauth-20-flow)和[SSO连接API](../../../docs/jackson/sso-flow/#2-sso-connection-api)创建接口文件：

```bash
app/routes $ mkdir api && cd api
routes/api $ touch oauth.\$slug.ts v1.connections.ts
```

oauth.$slug.ts: https://github.com/boxyhq/jackson-remix-auth/blob/main/app/routes/api/oauth.%24slug.ts

```typescript
...

// Handles GET /api/oauth/authorize, GET /api/oauth/userinfo
export const loader: LoaderFunction = async ({ params, request }) => {

  ... // Some validation logic
  const operation = params.slug;
  const url = new URL(request.url);

  const { oauthController } = await JacksonProvider({
    appBaseUrl: url.origin,
  });

  // rightmost query param will win in case of multiple ones with same name
  const queryParams = Object.fromEntries(url.searchParams.entries());

  switch (operation) {
    case "authorize": {
      ...
      try {
        const { redirect_url, authorize_form } =
          await oauthController.authorize(
            queryParams as unknown as OAuthReqBody
          );
        ...
      } catch (err: any) {
        ... // error handling, redirect to /error page
      }
    }
    case "oidc": {
      try {
        const { redirect_url } = await oauthController.oidcAuthzResponse(
          queryParams as unknown as OIDCAuthzResponsePayload
        );
        if (redirect_url) {
          return redirect(redirect_url, 302);
        }
      } catch (err: any) {
        ... // error handling, redirect to /error page
      }
    }
    case "userinfo": {
     ... // token validation
      try {
        const profile = await oauthController.userInfo(token);
        return json(profile);
      } catch (error: any) {
        ... // error handling
      }
    }
  }
};

// Handles POST /api/oauth/saml, POST /api/oauth/token
export const action: ActionFunction = async ({ params, request }) => {

  ... // Some validation logic
  const operation = params.slug;
  const url = new URL(request.url);

  const { oauthController } = await JacksonProvider({
    appBaseUrl: url.origin,
  });
  switch (operation) {
    case "saml": {
      try {
        const { redirect_url } = await oauthController.samlResponse(body);
        return redirect(redirect_url, 302);
      } catch (err: any) {
        ... // error handling
      }
    }
    case "token": {
      try {
        const tokenRes = await oauthController.token(body);
        return json(tokenRes);
      } catch (error: any) {
        ... // error handling
      }
    }
  }
};
```

v1.connections.ts: https://github.com/boxyhq/jackson-remix-auth/blob/main/app/routes/api/v1.connections.ts

```typescript
  export const loader: LoaderFunction = async ({ request }) => {
   const url = new URL(request.url);
   const queryParams = Object.fromEntries(
     url.searchParams.entries()
   ) as unknown as { clientID?: string; tenant?: string; product?: string };
   ...// Validate apiKey
   const { connectionAPIController } = await JacksonProvider({ appBaseUrl: url.origin });

   try {
     return json(await connectionAPIController.getConnections(queryParams));
   } catch (error: any) {
     ... // error handling
   }
 };

 export const action: ActionFunction = async ({ request }) => {
   const url = new URL(request.url);
   const contentType = request.headers.get("Content-Type");
   ... // Validate body,apiKey
   const { connectionAPIController } = await JacksonProvider({ appBaseUrl: url.origin });

   try {
     switch (request.method) {
       case "POST":
         return json(await connectionAPIController.createSAMLConnection(body));
       case "PATCH":
         await connectionAPIController.updateSAMLConnection(body);
         return new Response(null, { status: 204 });
       case "DELETE":
         await connectionAPIController.deleteConnections(body);
         return new Response(null, { status: 204 });
     }
   } catch (error: any) {
     ... // error handling
   }
 };
```

### 初始化 BoxyHQSSOStrategy

如下所示将策略与 `Authenticator` 结合使用。`clientID/Secret` 值预期由客户端动态设置，目前可暂设为 `dummy`。

将 `issuer` 指向应用URL。

auth.server.ts: https://github.com/boxyhq/jackson-remix-auth/blob/main/app/auth.server.ts

> **环境变量 BOXYHQSSO_ISSUER 需设置为应用URL：http://localhost:3366 或实际托管地址。参考 [.env.example](https://github.com/boxyhq/jackson-remix-auth/blob/main/.env.example) 文件 &nbsp;**

```typescript
invariant(process.env.BASE_URL, 'Expected BASE_URL to be set in env');
invariant(
  process.env.BOXYHQSSO_ISSUER,
  'Expected BOXYHQSSO_ISSUER to be set in env'
);

const BASE_URL = process.env.BASE_URL;
const BOXYHQSSO_ISSUER = process.env.BOXYHQSSO_ISSUER;
// Strategy use for the embedded sso service provider goes here
auth.use(
  new BoxyHQSSOStrategy(
    {
      issuer: BOXYHQSSO_ISSUER, //same as the APP URL
      clientID: 'dummy',
      clientSecret: process.env.CLIENT_SECRET_VERIFIER || 'dummy',
      callbackURL: new URL('/auth/sso/embed/callback', BASE_URL).toString(),
    },
    async ({ profile }) => {
      return profile;
    }
  ),
  'boxyhq-sso-embed'
);
```

### 处理登录和身份提供商回调的路由

需要创建两个路由：

~> [/auth/sso/embed](https://github.com/boxyhq/jackson-remix-auth/blob/main/app/routes/auth.sso.embed.tsx) - 登录操作处理器  
~> [/auth/sso/embed/callback](https://github.com/boxyhq/jackson-remix-auth/blob/main/app/routes/auth.sso.embed.callback.tsx) - 授权成功后携带授权码重定向至此。通过 `code` 交换获取 `token` 及用户档案。

在 `app/routes` 目录下创建以下文件：

app/routes/auth.sso.embed.tsx: https://github.com/boxyhq/jackson-remix-auth/blob/main/app/routes/auth.sso.embed.tsx

```tsx
...
export const action: ActionFunction = async ({ request }) => {
  const formData = await request.formData();
  const email = formData.get("email");
  const product = formData.get("product");

  ... // Add some validation logic

  // extracting the tenant from email is one way to set it
  const tenant = email.split("@")[1];

  return await auth.authenticate("boxyhq-sso-embed", request, {
    successRedirect: "/private",
    failureRedirect: "/login",
    context: {
      clientID: `tenant=${tenant}&product=${product}`,
      clientSecret: process.env.CLIENT_SECRET_VERIFIER || "dummy",
    },
  });
};
```

app/routes/auth.sso.embed.callback.tsx: https://github.com/boxyhq/jackson-remix-auth/blob/main/app/routes/auth.sso.embed.callback.tsx

```tsx
...
export const loader: LoaderFunction = async ({ request, params }) => {
  return auth.authenticate("boxyhq-sso-embed", request, {
    successRedirect: "/private",
    failureRedirect: "/login",
  });
};
```

## 添加SSO连接

为 [mocksaml.com](https://mocksaml.com) [添加SAML SSO连接](../../../docs/jackson/sso-flow/#21-add-connection)。启动应用后可按如下方式调用连接API：

<details>
<summary>Below adds a SAML SSO connection for https://mocksaml.com</summary>
<pre>
curl --location --request POST 'http://localhost:3366/api/v1/connections'
--header 'Authorization: Api-Key __JACKSON_API_KEY__'
--header 'Content-Type: application/x-www-form-urlencoded'
--data-urlencode 'encodedRawMetadata=PD94bWwgdmVyc2lvbj0iMS4wIiBlbmNvZGluZz0iVVRGLTgiIHN0YW5kYWxvbmU9Im5vIj8+CjxFbnRpdHlEZXNjcmlwdG9yIHhtbG5zOm1kPSJ1cm46b2FzaXM6bmFtZXM6dGM6U0FNTDoyLjA6bWV0YWRhdGEiIGVudGl0eUlEPSJodHRwczovL3NhbWwuZXhhbXBsZS5jb20vZW50aXR5aWQiIHZhbGlkVW50aWw9IjIwMjYtMDYtMjJUMTg6Mzk6NTMuMDAwWiI+CiAgPElEUFNTT0Rlc2NyaXB0b3IgV2FudEF1dGhuUmVxdWVzdHNTaWduZWQ9ImZhbHNlIiBwcm90b2NvbFN1cHBvcnRFbnVtZXJhdGlvbj0idXJuOm9hc2lzOm5hbWVzOnRjOlNBTUw6Mi4wOnByb3RvY29sIj4KICAgIDxLZXlEZXNjcmlwdG9yIHVzZT0ic2lnbmluZyI+CiAgICAgIDxLZXlJbmZvIHhtbG5zOmRzPSJodHRwOi8vd3d3LnczLm9yZy8yMDAwLzA5L3htbGRzaWcjIj4KICAgICAgICA8WDUwOURhdGE+CiAgICAgICAgICA8WDUwOUNlcnRpZmljYXRlPk1JSUM0akNDQWNvQ0NRQzMzd255YlQ1UVpEQU5CZ2txaGtpRzl3MEJBUXNGQURBeU1Rc3dDUVlEVlFRR0V3SlYKU3pFUE1BMEdBMVVFQ2d3R1FtOTRlVWhSTVJJd0VBWURWUVFEREFsTmIyTnJJRk5CVFV3d0lCY05Nakl3TWpJNApNakUwTmpNNFdoZ1BNekF5TVRBM01ERXlNVFEyTXpoYU1ESXhDekFKQmdOVkJBWVRBbFZMTVE4d0RRWURWUVFLCkRBWkNiM2g1U0ZFeEVqQVFCZ05WQkFNTUNVMXZZMnNnVTBGTlREQ0NBU0l3RFFZSktvWklodmNOQVFFQkJRQUQKZ2dFUEFEQ0NBUW9DZ2dFQkFMR2ZZZXR0TXNjdDFUNnRWVXdUdWROSkg1UG5iOUdHbmtYaTlady9lNng0NUREMApSdVJPTmJGbEoyVDRSakFFL3VHK0FqWHhYUThvMlNaZmI5K0dnbUNIdVRKRk5nSG9aMW5GVlhDbWIvSGc4SHBkCjR2T0FHWG5kaXhhUmVPaXEzRUg1WHZwTWpNa0ozKzgrOVZZTXpNWk9qa2dRdEFxTzM2ZUFGRmZOS1g3ZFRqM1YKcHdMa3Z6Ni9LRkNxOE9Bd1krQVVpNGVabTVKNTdEMzFHempId2ZqSDlXVGVYME15bmRtbk5CMXFWNzVxUVIzYgoyL1c1c0dIUnYrOUFhcmdnSmtGK3B0VWtYb0x0VkE1MXdjZlltNmhJTHB0cGRlNUZRQzhSV1kxWXJzd0JXQUVaCk5meXJSNEplU3dlRWxOSGc0TlZPczRUd0dqT1B3V0dxelRmZ1RsRUNBd0VBQVRBTkJna3Foa2lHOXcwQkFRc0YKQUFPQ0FRRUFBWVJsWWZsU1hBV29acEZmd05pQ1FWRTVkOXpaMERQek5kV2hBeWJYY1R5TWYwejVtRGY2RldCVwo1R3lvaTl1M0VNRURuekxjSk5rd0pBQWMzOUFwYTRJMi90bWwrSnkyOWRrOGJUeVg2bTkzbmdtQ2dkTGg1WmE0CmtodVUzQU0zTDYzZzdWZXhDdU83a3dramgvK0xxZGNJWHNWR082WERmdTJRT3MxWHBlOXpJekxwd20vUk5ZZVgKVWpiU2o1Y2UvamVrcEF3N3F5VlZMNHhPeWg4QXRVVzFlazN3SXcxTUp2RWdFUHQwZDE2b3NoV0pwb1MxT1Q4TApyLzIyU3ZZRW8zRW1TR2RUVkdnazN4M3MrQTBxV0FxVGN5anI3UTRzL0dLWVJGZm9tR3d6MFRaNEl3MVpOOTlNCm0wZW8yVVNsU1JUVmw3UUhSVHVpdVNUaEhwTEtRUT09CjwvWDUwOUNlcnRpZmljYXRlPgogICAgICAgIDwvWDUwOURhdGE+CiAgICAgIDwvS2V5SW5mbz4KICAgIDwvS2V5RGVzY3JpcHRvcj4KICAgIDxOYW1lSURGb3JtYXQ+dXJuOm9hc2lzOm5hbWVzOnRjOlNBTUw6MS4xOm5hbWVpZC1mb3JtYXQ6ZW1haWxBZGRyZXNzPC9OYW1lSURGb3JtYXQ+CiAgICA8U2luZ2xlU2lnbk9uU2VydmljZSBCaW5kaW5nPSJ1cm46b2FzaXM6bmFtZXM6dGM6U0FNTDoyLjA6YmluZGluZ3M6SFRUUC1SZWRpcmVjdCIgTG9jYXRpb249Imh0dHBzOi8vbW9ja3NhbWwuY29tL2FwaS9zYW1sL3NzbyIvPgogICAgPFNpbmdsZVNpZ25PblNlcnZpY2UgQmluZGluZz0idXJuOm9hc2lzOm5hbWVzOnRjOlNBTUw6Mi4wOmJpbmRpbmdzOkhUVFAtUE9TVCIgTG9jYXRpb249Imh0dHBzOi8vbW9ja3NhbWwuY29tL2FwaS9zYW1sL3NzbyIvPgogIDwvSURQU1NPRGVzY3JpcHRvcj4KPC9FbnRpdHlEZXNjcmlwdG9yPg=='
--data-urlencode 'defaultRedirectUrl=http://localhost:3366'
--data-urlencode 'redirectUrl=["http://localhost:3366/*"]'
--data-urlencode 'tenant=boxyhq.com'
--data-urlencode 'product=saml-demo.boxyhq.com'
--data-urlencode 'name=demo-config'
--data-urlencode 'description=Demo SAML config'
</pre>
</details>

## 应用路由

### 登录/登出

`登录`页面需包含可接收邮箱（用于租户识别）和产品的表单。同时需修改`嵌入式SAML提供商`按钮的表单动作。

**login.tsx: https://github.com/boxyhq/jackson-remix-auth/blob/main/app/routes/login.tsx &nbsp;**

```tsx
  export const loader: LoaderFunction = async ({ request }) => {
    // check if authenticated then redirect to /private
    await auth.isAuthenticated(request, { successRedirect: "/private" });
    // return form error data from session
  }

   export default function Login() {
     return (
       ...
       <Form
        method="post"
        action="/auth/sso"
        reloadDocument
        >
        ... // input fields for email and product
        <button type="submit">Continue with SAML SSO (Hosted SAML Provider)</button>
        <button type="submit" formAction="/auth/sso/embed">Continue with SAML SSO (Embedded SAML Provider)</button>
       </Form>
       ...
     )
```

**logout.ts: https://github.com/boxyhq/jackson-remix-auth/blob/main/app/routes/logout.ts &nbsp;**

```typescript
export const loader: LoaderFunction = async ({ request }) => {
  await auth.logout(request, { redirectTo: '/login' });
};
```

### 私有页面

此页面呈现已登录用户的个人资料。

**private.tsx: https://github.com/boxyhq/jackson-remix-auth/blob/main/app/routes/private.tsx &nbsp;**

```tsx
export const loader: LoaderFunction = async ({ request }) => {
  const profile = await auth.isAuthenticated(request, {
    failureRedirect: '/login',
  });

  return json<LoaderData>({ profile });
};

export default function Private() {
  const { profile } = useLoaderData<LoaderData>();
  return (
    <>
      <h1 className="text-primary mb-4 font-bold md:text-3xl">Raw profile</h1>
      <pre>
        <code>{JSON.stringify(profile, null, 2)}</code>
      </pre>
    </>
  );
}
```

### 错误页面

当SAML流程（/api/oauth/authorize 和 /api/oauth/saml）中出现错误时，用户将被[重定向](https://github.com/boxyhq/jackson-remix-auth/blob/e75cb4a9c340b088749ec63d6932f2f4b206a07d/app/routes/api/oauth.%24slug.ts#L63)至错误页面。

在 `/error` 路径下创建：

**error.tsx: https://github.com/boxyhq/jackson-remix-auth/blob/main/app/routes/error.tsx &nbsp;**

```tsx
export const loader: LoaderFunction = async ({ request }) => {
  const session = await getSession(request.headers.get('Cookie'));
  const { statusCode, message } = session.get(JACKSON_ERROR_COOKIE_KEY) || {
    statusCode: null,
    message: '',
  };
  return json({ statusCode, message });
};

export default function Error() {
  const { statusCode, message } = useLoaderData<LoaderData>();

  let statusText = '';
  if (typeof statusCode === 'number') {
    if (statusCode >= 400 && statusCode <= 499) {
      statusText = 'client-side error';
    }
    if (statusCode >= 500 && statusCode <= 599) {
      statusText = 'server error';
    }
  }

  if (statusCode === null) {
    return null;
  }

  return (
    <div className="h-full">
      <div className="h-[20%] translate-y-[100%] px-[20%] text-[hsl(152,56%,40%)]">
        <svg
          className="mb-5 h-10 w-10"
          fill="none"
          viewBox="0 0 24 24"
          stroke="currentColor"
          strokeWidth={2}
        >
          <path
            strokeLinecap="round"
            strokeLinejoin="round"
            d="M12 8v4m0 4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z"
          />
        </svg>
        <h1 className="text-xl font-extrabold md:text-6xl">{statusCode}</h1>
        <h2 className="uppercase">{statusText}</h2>
        <p className="mt-6 inline-block">SAML error: </p>
        <p className="mr-2 text-xl font-bold">{message}</p>
      </div>
    </div>
  );
}
```

## 准备就绪

至此，您的Remix应用已具备单点登录功能。🎉

## 后续步骤

- 有问题需要咨询？[请在此提问](https://discord.gg/uyb7pYt4Pa)
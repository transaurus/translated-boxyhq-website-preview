---
title: Add SAML SSO to Next.js App with BoxyHQ
description: Add SAML SSO to Next.js App with BoxyHQ
sidebar_label: Next.js
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# 为Next.js应用添加SAML单点登录

本指南假设您已拥有一个Next.js应用，并希望为企业客户启用SAML单点登录认证。完成本指南后，您的应用将能够通过SAML单点登录对用户进行身份验证。

访问[GitHub仓库](https://github.com/boxyhq/jackson)查看**BoxyHQ的SAML单点登录方案**。

访问[GitHub仓库](https://github.com/boxyhq/saas-starter-kit)查看**Next.js SAML单点登录集成**的源代码。

为应用集成SAML单点登录需要完成以下步骤。

- 配置SAML单点登录
- 通过SAML单点登录进行认证

## 配置SAML单点登录

此步骤允许您的租户为其用户配置SAML连接。阅读以下指南以了解更多信息。

- [SAML单点登录配置的UI最佳实践](/guides/jackson/configuring-saml-sso)
- [SSO连接API](/docs/jackson/sso-flow/)

## 通过SAML单点登录进行认证

添加SAML连接后，应用可使用该连接通过SAML Jackson发起SSO认证流程。以下章节将重点介绍SSO认证环节。

### 安装SAML Jackson

要开始使用SAML Jackson，请通过Node包管理器将该包添加至项目依赖。

```bash
npm i --save @boxyhq/saml-jackson
```

### 配置SAML Jackson

配置SAML Jackson以适配Next.js应用。

```bash title=".env"
NEXTAUTH_URL=https://your-app.com
NEXTAUTH_SECRET= #A random string is used to hash tokens, sign/encrypt cookies and generate cryptographic keys.
```

```js title="lib/jackson.ts"
import jackson, {
  type IOAuthController,
  type JacksonOption,
} from "@boxyhq/saml-jackson";

const samlAudience = "https://saml.boxyhq.com";
const samlPath = "/api/oauth/saml";

const opts: JacksonOption = {
  externalUrl: `${process.env.NEXTAUTH_URL}`,
  samlAudience,
  samlPath,
  db: {
    engine: "sql",
    type: "postgres",
    url: "postgres://postgres:postgres@localhost:5432/postgres",
  },
};

let oauthController: IOAuthController;

const g = global as any;

export default async function init() {
  if (!g.oauthController) {
    const ret = await jackson(opts);

    oauthController = ret.oauthController;
    g.oauthController = oauthController;
  } else {
    oauthController = g.oauthController;
  }

  return {
    oauthController,
  };
}
```

`samlPath`是身份认证提供商在用户认证后提交SAML响应的端点。

我们将使用[NextAuth.js](https://next-auth.js.org/)进行认证。NextAuth是为Next.js应用提供的完整开源认证解决方案。

```bash
npm i --save next-auth
```

NextAuth内置了BoxyHQ SAML提供商`boxyhq-saml`。我们将使用该提供商对用户进行认证。

```js title="pages/api/auth/[...nextauth].ts"
import NextAuth, { type NextAuthOptions } from 'next-auth';
import BoxyHQSAMLProvider from 'next-auth/providers/boxyhq-saml';

export const authOptions: NextAuthOptions = {
  providers: [
    BoxyHQSAMLProvider({
      authorization: { params: { scope: '' } },
      issuer: `${process.env.NEXTAUTH_URL}`,
      clientId: 'dummy',
      clientSecret: 'dummy',
      httpOptions: {
        timeout: 30000,
      },
    }),
  ],
  session: {
    strategy: 'jwt',
  },
};

export default NextAuth(authOptions);
```

### 发起认证请求

添加路由以启动认证流程，该路由通过将用户重定向至其配置的身份提供商来发起SAML SSO流程。

```js title="pages/api/oauth/authorize.ts"
import type { NextApiRequest, NextApiResponse } from "next";
import type { OAuthReq } from "@boxyhq/saml-jackson";

import jackson from "../../../../lib/jackson";

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const { oauthController } = await jackson();

  const { redirect_url } = await oauthController.authorize(
    req.query as unknown as OAuthReq
  );

  return res.redirect(302, redirect_url as string);
}
```

### 接收SAML响应

认证成功后，身份提供商将SAML响应POST至断言消费者服务(ACS)URL。

添加路由以处理SAML响应。确保该路由与初始化SAML Jackson库时配置的`samlPath`值匹配，且能够接收POST请求。

```js title="pages/api/oauth/saml.ts"
import type { NextApiRequest, NextApiResponse } from "next";

import jackson from "../../../../lib/jackson";

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const { oauthController } = await jackson();

  const { RelayState, SAMLResponse } = req.body;

  const { redirect_url } = await oauthController.samlResponse({
    RelayState,
    SAMLResponse,
  });

  return res.redirect(302, redirect_url as string);
}
```

### 请求访问令牌

添加另一个路由以接收认证后的回调。

NextAuth通过传递授权`code`及包含`grant_type`、`redirect_uri`和`code_verifier`的认证详情来请求`access_token`。

```js title="pages/api/oauth/token.ts"
import type { NextApiRequest, NextApiResponse } from 'next';

import jackson from '../../../../lib/jackson';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const { oauthController } = await jackson();

  const response = await oauthController.token(req.body);

  return res.json(response);
}
```

### 获取用户资料

获取`access_token`后，NextAuth可用其从身份提供商检索用户资料。若授权有效，`userInfo`方法将返回包含用户资料的响应。

```js title="pages/api/oauth/userinfo.ts"
import type { NextApiRequest, NextApiResponse } from 'next';

import jackson from '../../../../lib/jackson';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const { oauthController } = await jackson();

  const authHeader = req.headers['authorization'];

  if (!authHeader) {
    throw new Error('Unauthorized');
  }

  const token = authHeader.split(' ')[1];

  const user = await oauthController.userInfo(token);

  return res.json(user);
}
```

完整的响应内容大致如下：

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

当从身份提供商处获取到用户信息后，您需要判断该用户是否已存在于您的应用中并进行认证。若用户不存在，通常需要在数据库中创建新记录来存储该用户信息。

### 启动OAuth登录流程

您可以使用NextAuth的`signIn`方法，通过`boxyhq-saml`提供商发起认证。

可通过`signIn()`方法的第三个参数，向`/api/oauth/authorize`端点传递`tenant`和`product`等附加参数。

:::info
请确保已为租户和产品组合配置有效的SAML连接，否则认证将失败。了解如何创建SAML连接请参阅[此处](/docs/jackson/deploy/npm-library#create-saml-connection)
:::

要使本示例应用正常运行，您需要先为租户`boxyhq.com`和产品`saml-demo.boxyhq.com`添加SAML连接，才能对用户进行认证。

```js title="pages/login.tsx"
import type { NextPage } from 'next';
import { useSession, signIn } from 'next-auth/react';

const Login: NextPage = () => {
  const { data: session, status } = useSession();

  if (status === 'loading') {
    return <>Loading...</>;
  }

  if (status === 'authenticated') {
    return <>Authenticated</>;
  }

  // Starts OAuth sign-in flow
  signIn('boxyhq-saml', undefined, {
    tenant: 'boxyhq.com',
    product: 'saml-demo.boxyhq.com',
  });

  return <>Unauthenticated</>;
};

export default Login;
```
---
title: Add SAML SSO to React App with BoxyHQ
description: Add SAML SSO to React App with BoxyHQ
sidebar_label: React
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# 为React应用添加SAML单点登录

本指南假设您已有一个React应用，并希望为企业客户启用SAML单点登录认证。通过本指南，您将获得一个支持使用SAML单点登录进行用户认证的应用。

若希望直接查看源码，请访问：https://github.com/boxyhq/jackson-examples/tree/main/apps/react-example

:::info
我们使用Express后端配合React单页应用。相关代码可在 https://github.com/boxyhq/jackson-examples/tree/main/apps/express-api 找到
:::

为应用集成SAML单点登录需要以下步骤。

- 配置SAML单点登录
- 通过SAML单点登录进行认证

## 在React上配置企业级单点登录

此步骤允许您的租户为其用户配置SAML连接。阅读以下指南以了解更多信息。

- [配置SAML单点登录的UI最佳实践](/guides/jackson/configuring-saml-sso)
- [单点登录连接API](/docs/jackson/sso-flow/)

## 通过SAML单点登录进行认证

添加SAML连接后，应用可使用该连接通过SAML Jackson发起单点登录认证流程。以下章节主要关注单点登录认证部分。

### 部署SAML Jackson服务

第一步是部署SAML Jackson服务。请按照[部署文档](/docs/jackson/deploy/service)安装并配置SAML Jackson。

### 设置SAML Jackson集成

我们将使用客户端库`@bity/oauth2-auth-code-pkce`来实现认证流程。这是一个零依赖的OAuth 2.0客户端，实现了带PKCE的授权码模式以提供客户端保护。

```bash
npm i --save @bity/oauth2-auth-code-pkce
```

让我们配置`OAuth2AuthCodePKCE`客户端以使用SAML Jackson服务进行认证。这里我们使用自定义钩子，以便`oauthClient`可在应用其他位置使用。

```ts title="src/hooks/useOAuthClient.ts"
import { OAuth2AuthCodePKCE } from '@bity/oauth2-auth-code-pkce';
import { useEffect, useState } from 'react';

const JACKSON_URL = process.env.REACT_APP_JACKSON_URL;

interface OauthClientOptions {
  redirectUrl: string;
}
export default function useOAuthClient({
  redirectUrl,
}: OauthClientOptions): OAuth2AuthCodePKCE | null {
  const [oauthClient, setOauthClient] = useState<OAuth2AuthCodePKCE | null>(
    null
  );

  useEffect(() => {
    setOauthClient(
      new OAuth2AuthCodePKCE({
        authorizationUrl: `${JACKSON_URL}/api/oauth/authorize`,
        tokenUrl: `${JACKSON_URL}/api/oauth/token`,
        // Setting the clientId dummy here. We pass additional query params for
        // tenant and product in the authorize request.
        clientId: 'dummy',
        redirectUrl,
        scopes: [],
        onAccessTokenExpiry(refreshAccessToken) {
          console.log('Expired! Access token needs to be renewed.');
          alert(
            'We will try to get a new access token via grant code or refresh token.'
          );
          return refreshAccessToken();
        },
        onInvalidGrant(refreshAuthCodeOrRefreshToken) {
          console.log(
            'Expired! Auth code or refresh token needs to be renewed.'
          );
          alert('Redirecting to auth server to obtain a new auth grant code.');
          //return refreshAuthCodeOrRefreshToken();
        },
      })
    );
  }, [redirectUrl]);

  return oauthClient;
}
```

### 设置全局认证基础组件

#### 认证上下文(AuthContext)

我们需要一种方式使已登录的`user`以及`signIn`、`signOut`方法全局可访问。这些方法连同`setTenant`（用于选择单点登录流程的租户）和`authStatus`（布尔值，帮助我们根据认证状态是否完全已知或正在加载来条件渲染内容）通过使用`AuthContext`在整个应用中可用。

```tsx title="src/lib/AuthProvider.tsx"
import React, { useState, useEffect, ReactNode, createContext } from 'react';
import { useLocation } from 'react-router-dom';
import useOAuthClient from '../hooks/useOAuthClient';
import { authenticate, getProfileByJWT } from './backend';

interface ProviderProps {
  children: ReactNode;
}

interface AuthContextInterface {
  setTenant?: React.Dispatch<React.SetStateAction<string>>;
  authStatus: 'UNKNOWN' | 'FETCHING' | 'LOADED';
  user: any;
  signIn: () => void;
  signOut: (callback: VoidFunction) => void;
}

// localstorage key to store from url
const APP_FROM_URL = 'appFromUrl';

export const AuthContext = createContext<AuthContextInterface>(null!);
```

我们还创建了一个返回`AuthContext`句柄的自定义钩子。

```ts title="src/hooks/useAuth.ts"
import { useContext } from 'react';
import { AuthContext } from '../lib/AuthProvider';

const useAuth = () => {
  return useContext(AuthContext);
};

export default useAuth;
```

#### 认证提供者(AuthProvider)

我们将在AuthProvider中实现整个流程。

1. 应用外壳渲染完成后，我们会执行一个副作用钩子，利用从`useOAuthClient`获取的`authClient`来处理认证流程。这里需要处理两种场景：第一种是已从SSO提供商(Jackson)获取access_token的情况，此时可通过传入cookie来获取已登录用户资料；第二种是用户在身份提供商(IdP)完成登录后浏览器重定向回应用的场景，此时需将重定向中的授权码交换为访问令牌，再传递给应用后端完成登录。

    ```tsx title="src/lib/AuthProvider.tsx"
      const AuthProvider = ({ children }: ProviderProps) => {
        const [user, setUser] = useState<any>(null);
        const [authStatus, setAuthStatus] = useState<AuthContextInterface['authStatus']>('UNKNOWN');

        ...

        const redirectUrl = process.env.REACT_APP_APP_URL + from;

        const authClient = useOAuthClient({ redirectUrl });

        useEffect(() => {
          let didCancel = false;

          const loadUser = async () => {
            if (!authClient) {
              return;
            }
            setAuthStatus('FETCHING');
            if (authClient.isAuthorized()) {
              const { data, error } = await getProfileByJWT();
              if (!didCancel && !error) {
                setUser(data);
                setAuthStatus('LOADED');
              }
            } else {
              try {
                const hasAuthCode = await authClient?.isReturningFromAuthServer();
                if (!hasAuthCode) {
                  devLogger('no auth code detected...');
                } else {
                  const token = !didCancel
                    ? await authClient?.getAccessToken()
                    : null;
                  token && localStorage.removeItem(APP_FROM_URL);
                  // 认证操作在后端完成，使用上述令牌获取用户资料
                  const profile = await authenticate(token?.token?.value);
                  if (!didCancel && profile) {
                    setUser(profile);
                  }
                }
              } catch (err) {
                console.error(err);
              } finally {
                setAuthStatus('LOADED');
              }
            }
          };

          loadUser();
          return () => {
            didCancel = true;
          };
        }, [authClient]);

        ...

         const value = {
          authStatus,
          user,
        };

        return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>;
      };

      export { AuthContext, AuthProvider };
    ```

2. 当用户尝试访问受保护/私有路由时，将被重定向至登录页。在此之前我们会将其尝试访问的当前路径保存在历史状态中。该逻辑封装在`RequireAuth`包装组件中，可用于保护需要认证的路由。

    ```tsx title="src/components/RequireAuth.tsx"
    const RequireAuth = ({ children }: { children: JSX.Element }) => {
      let { user, authStatus } = useAuth();
      let location = useLocation();

      if (authStatus !== 'LOADED') {
        return null;
      }

      if (!user) {
        // 重定向至/login页面，但保存触发重定向的原始路径
        // 这使得用户登录后能跳转至目标页面，相比直接返回首页能提供更好的用户体验
        return <Navigate to="/login" state={{ from: location }} replace />;
      }

      return children;
    };

    export default RequireAuth;
    ```

    随后我们在`AuthProvider`内部使用`from`状态构建`redirectUrl`来初始化`oAuthClient`。

    ```ts title="src/lib/AuthProvider.tsx"
    let location = useLocation();
    let from =
      location.state?.from?.pathname ||
      localStorage.getItem(APP_FROM_URL) ||
      '/profile';

    const redirectUrl = process.env.REACT_APP_APP_URL + from;

    const authClient = useOAuthClient({ redirectUrl });
    ```

3. `signIn`和`signOut`方法可按如下方式实现：

    ```tsx title="src/lib/AuthProvider.tsx"
    const signIn = async () => {
      // 在重定向前存储'from'路径，用于从SSO提供商重定向回来后正确初始化oauthClient
      localStorage.setItem(APP_FROM_URL, from);
      // 发起登录流程
      await authClient?.fetchAuthorizationCode({
        tenant,
        product: 'saml-demo.boxyhq.com',
      });
    };

    const signOut = async (callback: VoidFunction) => {
      authClient?.reset();
      setUser(null);
      callback();
    };

    const value = {
      signIn,
      signOut,
    };

    return (
      <AuthContext.Provider value={value}>{children}</AuthContext.Provider>
    );
    ```

### 发起认证请求

让我们添加一个页面来启动认证流程。该页面通过调用 `AuthContext` 中的 `signIn` 方法，将用户重定向至其配置的身份提供商（通过 Jackson），从而发起 SAML SSO 流程。

当用户点击"使用 SAML SSO 继续"按钮时，将被重定向至身份提供商。

```js title="src/pages/Login.tsx"
import React from 'react';
import { Navigate, useLocation } from 'react-router-dom';
import useAuth from '../hooks/useAuth';

const Login = () => {
  let location = useLocation();

  let from = location.state?.from?.pathname || '/profile';

  const { signIn, setTenant, authStatus, user } = useAuth();

  if (authStatus !== 'LOADED') {
    return null;
  }

  if (authStatus === 'LOADED' && user) {
    return <Navigate to={from} replace />;
  }

  return (
    <div className="mx-auto h-screen max-w-7xl">
      <div className="flex h-full flex-col justify-center space-y-5">
        <h2 className="text-center text-3xl">Log in to App</h2>
        <div className="mx-auto w-full max-w-md px-3 md:px-0">
          <div className="rounded border border-gray-200 bg-white py-5 px-5">
            <form className="space-y-3" method="POST" onSubmit={signIn}>
              <label htmlFor="tenant" className="block text-sm">
                Tenant ID
              </label>
              <input
                type="text"
                name="tenant"
                placeholder="boxyhq"
                defaultValue="boxyhq.com"
                className="block w-full appearance-none rounded border border-gray-300 text-sm placeholder-gray-400 focus:outline-none focus:ring-indigo-500"
                required
                onChange={(e) =>
                  typeof setTenant === 'function' && setTenant(e.target.value)
                }
              />
              <button
                type="submit"
                className="w-full rounded border border-transparent bg-indigo-600 px-4 py-2 text-sm font-medium text-white focus:outline-none"
              >
                Continue with SAML SSO
              </button>
            </form>
          </div>
        </div>
      </div>
    </div>
  );
};

export default Login;
```

### 获取用户资料

获取 `accessToken` 后，React 应用可借此从身份提供商处检索用户资料。

通常您会使用后端服务（例如 Express.js）调用 SAML Jackson API，通过 `accessToken` 获取用户资料。

以下是 Express.js 路由示例，它们会在登录时或通过解析客户端 cookie 中的 JWT 来返回用户资料。

```js
app.get('/api/authenticate', async function (req, res, next) {
  const accessToken = req.query.access_token;

  if (!accessToken) {
    throw new Error('Access token not found.');
  }

  const response = await fetch(
    `${jacksonUrl}/api/oauth/userinfo?access_token=${accessToken}`,
    {
      method: 'GET',
    }
  );

  const profile = await response.json();

  // Once the user has been retrieved from the Identity Provider,
  // you may determine if the user exists in your application and authenticate the user.
  // If the user does not exist in your application, you will typically create a new record in your database to represent the user.

  const token = jsonwebtoken.sign(
    {
      id: profile.id,
      email: profile.email,
      firstName: profile.firstName,
      lastName: profile.lastName,
    },
    jwtSecret
  );

  res.cookie('sso-token', token, { httpOnly: true });
  res.json(profile);
});

app.get('/api/profile', async function (req, res, next) {
  const token = req.cookies['sso-token'];

  if (!token) {
    return res
      .status(401)
      .json({ data: null, error: { message: 'Missing JWT' } });
  }

  // You may fetch the user profile from your database using the user id.

  const payload = jsonwebtoken.verify(token, jwtSecret);

  return res.json({ data: payload, error: null });
});
```

用户资料示例如下：

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

在 React 应用中，若已持有 access_token 则调用 `getProfileByJWT`；若携带授权码从 SSO 提供商返回则调用 `authenticate`。

```ts title="src/lib/backend.ts"
const apiUrl = process.env.REACT_APP_API_URL;

export const authenticate = async (token: string | undefined) => {
  if (!token) {
    throw new Error('Access token not found.');
  }

  const response = await fetch(
    `${apiUrl}/api/authenticate?access_token=${token}`,
    {
      method: 'GET',
      credentials: 'include',
    }
  );
  if (response.ok) {
    return await response.json();
  }
  return null;
};

export const getProfileByJWT = async () => {
  const response = await fetch(`${apiUrl}/api/profile`, {
    method: 'GET',
    credentials: 'include',
  });

  return await response.json();
};
```

## 准备就绪

至此，您的 React 应用已实现单点登录功能。🎉

## 后续步骤

- 遇到问题？[在此提问](https://discord.gg/uyb7pYt4Pa)
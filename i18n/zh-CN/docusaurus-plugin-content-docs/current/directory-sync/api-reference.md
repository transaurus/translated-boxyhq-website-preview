---
title: Directory Sync API Reference
sidebar_label: API Reference
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# API 参考

以下指南提供有关目录同步服务可用API和SDK的信息。

### 目录

以下是目录连接的属性。

#### 属性

- `name`: 目录名称
- `tenant`: 要为其创建目录的租户ID
- `product`: 要为其创建目录的产品ID
- `type`: 目录提供商类型。详见[目录提供商](./providers)获取更多信息
- `webhook.endpoint`: 目录连接将POST事件发送到的webhook URL
- `webhook.secret`: 用于签名webhook负载的webhook密钥
- `log_webhook_events`: 指示是否应记录webhook事件
- `deactivated`: 指示目录连接是否已停用

:::info
`tenant`和`product`不应包含`:`字符，因为我们在内部将其用作分隔符。
:::

### 初始化目录同步

```javascript
const opts = {
  externalUrl: 'https://my-cool-app.com',
  db: {
    engine: 'mongo',
    url: 'mongodb://localhost:27017/my-cool-app',
  },
};

let directorySyncController;

// Please note that the initialization of @boxyhq/saml-jackson is async, you cannot run it at the top level.
async function init() {
  const jackson = await require('@boxyhq/saml-jackson').controllers(opts);

  directorySyncController = jackson.directorySyncController;
}
```

### 创建新目录

创建新的目录连接。

#### 请求

<Tabs>
<TabItem value="01" label="Node.js" default>

```javascript showLineNumbers
const { data, error } = await directorySyncController.directories.create({
  name: 'App',
  tenant: 'boxyhq',
  product: 'jackson',
  type: 'onelogin-scim-v2',
  webhook_url: 'https://my-cool-app.com/webhook',
  webhook_secret: 'my-secret',
});
```

</TabItem>
<TabItem value="02" label="Shell">

```bash
curl --request POST \
  --url http://localhost:5225/api/v1/directory-sync \
  --header 'Authorization: Api-Key secret' \
  --header 'Content-Type: application/json' \
  --data '{
 "name": "App",
 "tenant": "boxyhq",
 "product": "jackson",
 "type": "onelogin-scim-v2",
 "webhook_url": "https://my-cool-app.com/webhook",
 "webhook_secret": "my-secret"
}'
```

</TabItem>
</Tabs>

#### 响应

```json
{
  "data": {
    "id": "58b5cd9dfaa39d47eb8f5f88631f9a629a232016",
    "name": "App",
    "tenant": "boxyhq",
    "product": "jackson",
    "type": "onelogin-scim-v2",
    "log_webhook_events": false,
    "scim": {
      "path": "/api/scim/v2.0/58b5cd9dfaa39d47eb8f5f88631f9a629a232016",
      "secret": "IJzAoevjD_liiiy-VkDtXg",
      "endpoint": "http://localhost:5225/api/scim/v2.0/58b5cd9dfaa39d47eb8f5f88631f9a629a232016"
    },
    "webhook": {
      "endpoint": "https://my-cool-app.com/webhook",
      "secret": "my-secret"
    }
  },
  "error": null
}
```

---

### 获取目录列表

获取给定租户和产品的目录列表。一个租户可以为相同或不同产品配置多个目录。

#### 请求

<Tabs>
<TabItem value="01" label="Node.js" default>

```javascript showLineNumbers
const tenant = 'boxyhq';
const product = 'jackson';

const { data, error } =
  await directorySyncController.directories.getByTenantAndProduct(
    tenant,
    product
  );
```

</TabItem>
<TabItem value="02" label="Shell">

```bash
curl --request GET \
  --url 'http://localhost:5225/api/v1/directory-sync?tenant=boxyhq&product=jackson' \
  --header 'Authorization: Api-Key secret' \
  --header 'Content-Type: application/json'
```

</TabItem>
</Tabs>

#### 响应

```json
{
  "data": [
    {
      "id": "58b5cd9dfaa39d47eb8f5f88631f9a629a232016",
      "name": "App",
      "tenant": "boxyhq",
      "product": "jackson",
      "type": "onelogin-scim-v2",
      "log_webhook_events": false,
      "scim": {
        "path": "/api/scim/v2.0/58b5cd9dfaa39d47eb8f5f88631f9a629a232016",
        "secret": "IJzAoevjD_liiiy-VkDtXg",
        "endpoint": "http://localhost:5225/api/scim/v2.0/58b5cd9dfaa39d47eb8f5f88631f9a629a232016"
      },
      "webhook": {
        "endpoint": "https://my-cool-app.com/webhook",
        "secret": "my-secret"
      }
    }
  ],
  "error": null
}
```

---

### 获取目录详情

通过唯一ID获取目录详细信息。

#### 请求

<Tabs>
<TabItem value="01" label="Node.js" default>

```javascript showLineNumbers
const directoryId = '58b5cd9dfaa39d47eb8f5f88631f9a629a232016';

const { data, error } = await directorySyncController.directories.get(
  directoryId
);
```

</TabItem>
<TabItem value="02" label="Shell">

```bash
curl --request GET \
  --url 'http://localhost:5225/api/v1/directory-sync/58b5cd9dfaa39d47eb8f5f88631f9a629a232016' \
  --header 'Authorization: Api-Key secret' \
  --header 'Content-Type: application/json'
```

</TabItem>
</Tabs>

#### 响应

```json
{
  "data": {
    "id": "58b5cd9dfaa39d47eb8f5f88631f9a629a232016",
    "name": "App",
    "tenant": "boxyhq",
    "product": "jackson",
    "type": "onelogin-scim-v2",
    "log_webhook_events": false,
    "scim": {
      "path": "/api/scim/v2.0/58b5cd9dfaa39d47eb8f5f88631f9a629a232016",
      "secret": "IJzAoevjD_liiiy-VkDtXg",
      "endpoint": "http://localhost:5225/api/scim/v2.0/58b5cd9dfaa39d47eb8f5f88631f9a629a232016"
    },
    "webhook": {
      "endpoint": "https://my-cool-app.com/webhook",
      "secret": "my-secret"
    }
  },
  "error": null
}
```

---

### 列出目录用户

列出目录中的所有用户。

#### 请求

<Tabs>
<TabItem value="01" label="Node.js" default>

```javascript showLineNumbers
const tenant = 'boxyhq';
const product = 'jackson';

const { data, error } = await directorySyncController.users
  .setTenantAndProduct(tenant, product)
  .getAll();
```

</TabItem>
<TabItem value="02" label="Shell">

```bash
curl --request GET \
  --url 'http://localhost:5225/api/v1/directory-sync/users?tenant=boxyhq&product=jackson' \
  --header 'Authorization: Api-Key secret' \
  --header 'Content-Type: application/json'
```

</TabItem>
</Tabs>

#### 响应

```json
{
  "data": [
    {
      "id": "6296f71e-15fd-4af4-86ee-d6623b3ef1a4",
      "first_name": "Aswin",
      "last_name": "Venugopal",
      "email": "aswin@boxyhq.com",
      "active": true,
      "raw": {
        "schemas": ["urn:ietf:params:scim:schemas:core:2.0:User"],
        "userName": "aswin@boxyhq.com",
        "name": {
          "givenName": "Aswin",
          "familyName": "Venugopal",
          "honorificPrefix": "Sir"
        },
        "emails": [
          {
            "primary": true,
            "value": "aswin@boxyhq.com",
            "type": "work"
          }
        ],
        "displayName": "Aswin Venugopal",
        "addresses": [
          {
            "primary": true,
            "postalCode": "123"
          }
        ],
        "locale": "en-US",
        "externalId": "00u34iw1hm16RmjS95d7",
        "groups": [],
        "active": true,
        "id": "6296f71e-15fd-4af4-86ee-d6623b3ef1a4"
      }
    },
    {
      "id": "ebc31d6e-7d62-4f81-b9e5-eb5f1a04ee92",
      "first_name": "Kiran",
      "last_name": "Krishnan",
      "email": "kiran@boxyhq.com",
      "active": true,
      "raw": {
        "schemas": ["urn:ietf:params:scim:schemas:core:2.0:User"],
        "userName": "kiran@boxyhq.com",
        "name": {
          "givenName": "Kiran",
          "familyName": "Krishnan"
        },
        "emails": [
          {
            "primary": true,
            "value": "kiran@boxyhq.com",
            "type": "work"
          }
        ],
        "displayName": "Kiran Krishnan",
        "addresses": [
          {
            "primary": true,
            "region": "Kerala"
          }
        ],
        "locale": "en-US",
        "externalId": "00u3e3cmpdDydXdzV5d7",
        "groups": [],
        "active": true,
        "id": "ebc31d6e-7d62-4f81-b9e5-eb5f1a04ee92"
      }
    }
  ],
  "error": null
}
```

---

### 获取目录用户详情

获取目录用户的详细信息。

#### 请求

<Tabs>
<TabItem value="01" label="Node.js" default>

```javascript showLineNumbers
const tenant = 'boxyhq';
const product = 'flex';
const userId = 'ebc31d6e-7d62-4f81-b9e5-eb5f1a04ee92';

const users = await directorySyncController.users
  .setTenantAndProduct(tenant, product)
  .get(userId);
```

</TabItem>
<TabItem value="02" label="Shell">

```bash
curl --request GET \
  --url 'http://localhost:5225/api/v1/directory-sync/users/ebc31d6e-7d62-4f81-b9e5-eb5f1a04ee92?tenant=boxyhq&product=jackson' \
  --header 'Authorization: Api-Key secret' \
  --header 'Content-Type: application/json'
```

</TabItem>
</Tabs>

#### 响应

```json
{
  "data": {
    "id": "ebc31d6e-7d62-4f81-b9e5-eb5f1a04ee92",
    "first_name": "Kiran",
    "last_name": "Krishnan",
    "email": "kiran@boxyhq.com",
    "active": true,
    "raw": {
      "schemas": ["urn:ietf:params:scim:schemas:core:2.0:User"],
      "userName": "kiran@boxyhq.com",
      "name": {
        "givenName": "Kiran",
        "familyName": "Krishnan"
      },
      "emails": [
        {
          "primary": true,
          "value": "kiran@boxyhq.com",
          "type": "work"
        }
      ],
      "displayName": "Kiran Krishnan",
      "addresses": [
        {
          "primary": true,
          "region": "Kerala"
        }
      ],
      "locale": "en-US",
      "externalId": "00u3e3cmpdDydXdzV5d7",
      "groups": [],
      "active": true,
      "id": "ebc31d6e-7d62-4f81-b9e5-eb5f1a04ee92"
    }
  },
  "error": null
}
```

---

### 列出目录群组

列出目录中的所有群组。

#### 请求

<Tabs>
<TabItem value="01" label="Node.js" default>

```javascript showLineNumbers
const tenant = 'boxyhq';
const product = 'jackson';

const users = await directorySyncController.groups
  .setTenantAndProduct(tenant, product)
  .getAll();
```

</TabItem>
<TabItem value="02" label="Shell">

```bash
curl --request GET \
  --url 'http://localhost:5225/api/v1/directory-sync/groups?tenant=boxyhq&product=jackson' \
  --header 'Authorization: Api-Key secret' \
  --header 'Content-Type: application/json'
```

</TabItem>
</Tabs>

#### 响应

```json
{
  "data": [
    {
      "id": "44d08c0e-d185-4a5e-80a6-b47a717ffaa5",
      "name": "Developers",
      "raw": {
        "schemas": ["urn:ietf:params:scim:schemas:core:2.0:Group"],
        "displayName": "Developers",
        "members": [
          {
            "value": "6296f71e-15fd-4af4-86ee-d6623b3ef1a4",
            "display": "aswin@boxyhq.com"
          },
          {
            "value": "ebc31d6e-7d62-4f81-b9e5-eb5f1a04ee92",
            "display": "kiran@boxyhq.com"
          }
        ],
        "id": "44d08c0e-d185-4a5e-80a6-b47a717ffaa5"
      }
    }
  ],
  "error": null
}
```

---

### 获取目录群组详情

获取目录群组的详细信息。

#### 请求

<Tabs>
<TabItem value="01" label="Node.js" default>

```javascript showLineNumbers
const tenant = 'boxyhq';
const product = 'jackson';
const groupId = '44d08c0e-d185-4a5e-80a6-b47a717ffaa5';

const users = await directorySyncController.groups
  .setTenantAndProduct(tenant, product)
  .get(groupId);
```

</TabItem>
<TabItem value="02" label="Shell">

```bash
curl --request GET \
  --url 'http://localhost:5225/api/v1/directory-sync/groups/44d08c0e-d185-4a5e-80a6-b47a717ffaa5?tenant=boxyhq&product=jackson' \
  --header 'Authorization: Api-Key secret' \
  --header 'Content-Type: application/json'
```

</TabItem>
</Tabs>

#### 响应

```json
{
  "data": {
    "id": "44d08c0e-d185-4a5e-80a6-b47a717ffaa5",
    "name": "Developers",
    "raw": {
      "schemas": ["urn:ietf:params:scim:schemas:core:2.0:Group"],
      "displayName": "Developers",
      "members": [
        {
          "value": "6296f71e-15fd-4af4-86ee-d6623b3ef1a4",
          "display": "aswin@boxyhq.com"
        },
        {
          "value": "ebc31d6e-7d62-4f81-b9e5-eb5f1a04ee92",
          "display": "kiran@boxyhq.com"
        }
      ],
      "id": "44d08c0e-d185-4a5e-80a6-b47a717ffaa5"
    },
    "members": [
      {
        "group_id": "44d08c0e-d185-4a5e-80a6-b47a717ffaa5",
        "user_id": "6296f71e-15fd-4af4-86ee-d6623b3ef1a4"
      },
      {
        "group_id": "44d08c0e-d185-4a5e-80a6-b47a717ffaa5",
        "user_id": "ebc31d6e-7d62-4f81-b9e5-eb5f1a04ee92"
      }
    ]
  },
  "error": null
}
```

---

### 列出群组成员

获取目录群组的成员。仅返回用户ID。

(v1.20.6版本引入)

#### 请求

<Tabs>
<TabItem value="01" label="Node.js" default>

```javascript showLineNumbers
await directorySyncController.groups
  .setTenantAndProduct('boxyhq', 'jackson')
  .getGroupMembers({
    groupId: '44d08c0e-d185-4a5e-80a6-b47a717ffaa5',
    pageOffset: 0,
    pageLimit: 10,
  });
```

</TabItem>
<TabItem value="02" label="Shell">

```bash
curl --request GET \
  --url 'http://localhost:5225/api/v1/directory-sync/groups/44d08c0e-d185-4a5e-80a6-b47a717ffaa5/members?tenant=boxyhq&product=jackson&pageOffset=0&pageLimit=10' \
  --header 'Authorization: Api-Key secret' \
  --header 'Content-Type: application/json'
```

</TabItem>
</Tabs>

#### 响应

```json
{
  "data": [
    {
      "user_id": "107130779649255553459"
    },
    {
      "user_id": "107471187046551199726"
    }
  ]
}
```

---

### Google目录同步

Google 目录同步特有属性：

- `google_domain`: Google Workspace 域名。
- `google_access_token`: Google 管理员访问令牌。
- `google_refresh_token`: Google 管理员刷新令牌。

Jackson 需要以下 API 权限范围：

- `https://www.googleapis.com/auth/admin.directory.group.member.readonly`
- `https://www.googleapis.com/auth/admin.directory.group.readonly`

#### 1. 获取认证 URL

获取用于管理员凭据认证的 URL。Google 将提示用户使用管理员凭据进行认证，并授权应用访问 Google Workspace 目录。

#### 请求

<Tabs>
<TabItem value="01" label="Node.js" default>

```javascript showLineNumbers
await directorySyncController.google.generateAuthorizationUrl({
  directoryId: '58b5cd9dfaa39d47eb8f5f88631f9a629a232016',
});
```

</TabItem>
</Tabs>

#### 响应

```json
{
  "data": {
    "authorizationUrl": "https://accounts.google.com/..."
  },
  "error": null
}
```

#### 2. 获取访问令牌

从授权码获取访问令牌。

#### 请求

<Tabs>
<TabItem value="01" label="Node.js" default>

```javascript showLineNumbers
await directorySyncController.google.getAccessToken({
  directoryId: '58b5cd9dfaa39d47eb8f5f88631f9a629a232016',
  code: 'authorization-code',
});
```

</TabItem>
</Tabs>

#### 响应

```json
{
  "data": {
    "access_token": "ya29.a0AWY7CknlXtELCHzdKuS...",
    "refresh_token": "1//03MJpa-zseicbCgYIARAAGAMSNwF-...",
    "scope": "https://www.googleapis.com/auth/admin.directory.group.member.readonly https://www.googleapis.com/auth/admin.directory.group.readonly",
    "token_type": "Bearer",
    "expiry_date": 1684343934305
  },
  "error": null
}
```

#### 3. 将访问令牌设置到目录

将访问令牌和刷新令牌设置到目录。我们将使用访问令牌向 Google 目录 API 发起请求。

#### 请求

<Tabs>
<TabItem value="01" label="Node.js" default>

```javascript showLineNumbers
await directorySyncController.google.setToken({
  directoryId: '58b5cd9dfaa39d47eb8f5f88631f9a629a232016',
  accessToken: 'ya29.a0AWY7CknlXtELCHzdKuS...',
  refreshToken: '1//03MJpa-zseicbCgYIARAAGAMSNwF-...',
});
```

</TabItem>
</Tabs>

#### 响应

```json
{
  "data": {
    "id": "58b5cd9dfaa39d47eb8f5f88631f9a629a232016",
    "name": "App",
    "tenant": "boxyhq",
    "product": "jackson",
    "type": "google",
    "google_domain": "boxyhq.com",
    "google_access_token": "ya29.a0AWY7CknlXtELCHzdKuS...",
    "google_refresh_token": "1//03MJpa-zseicbCgYIARAAGAMSNwF-..."
  },
  "error": null
}
```

#### 4. 同步目录

Jackson 可配置为同步您的 Google Workspace 目录。

<Tabs>
<TabItem value="01" label="Node.js" default>

```javascript showLineNumbers
const callback = (event: DirectorySyncEvent) => {
  console.log(event);
};

await directorySyncController.sync(callback);
```

</TabItem>
<TabItem value="02" label="Shell">

```bash
curl -X POST \
  -H "Authorization: Api-Key YOUR_API_KEY" \
  http://localhost:5225/api/v1/dsync/cron/sync-google
```

</TabItem>
</Tabs>

理想情况下，您会希望按计划（例如每 2 小时）运行同步。您可以使用 cron 作业按计划调用此 URL。

有关[目录同步事件](#handle-the-requests-from-identity-providers)的更多信息，请参阅下文。

---

### 处理来自身份提供商的请求

确保您的应用能够处理来自身份提供商的请求。

#### 路由

通常，您需要向应用添加以下路由以处理请求。但某些身份提供商的 `Methods` 可能有所不同。

| Route       | Methods    | Event                                          |
| ----------- | ---------- | ---------------------------------------------- |
| /Users      | POST       | A new user has been assigned to a SCIM app     |
| /Users/:id  | PUT, PATCH | A user information has been updated            |
| /Users/:id  | DELETE     | A user has been removed from a SCIM app        |
| /Groups     | POST       | A new group has been pushed                    |
| /Groups/:id | PUT        | Group name has been changed                    |
| /Groups/:id | PATCH      | User has been added to or removed from a group |
| /Groups/:id | DELETE     | Group has been removed                         |

#### 用户请求

```javascript showLineNumbers
const { data, status } = await directorySyncController.requests.handle(request);
```

`request` 的结构应如下：

```javascript
{
  method: HTTPMethod;
  body?: any;
  directoryId: string;
  resourceId: string;
  resourceType: "users",
  apiSecret: string;
  query: {
    count?: number;
    startIndex?: number;
    filter?: string;
  };
};
```

#### 群组请求

处理群组请求与处理用户请求类似。

```javascript showLineNumbers
const { data, status } = await directorySyncController.requests.handle(request);
```

`request` 的结构应如下：

```javascript
{
  method: HTTPMethod;
  body?: any;
  directoryId: string;
  resourceId: string;
  resourceType: "groups",
  apiSecret: string;
  query: {
    count?: number;
    startIndex?: number;
    filter?: string;
  };
};
```

#### 回调函数

您可以选择将回调函数作为第二个参数传递给处理方法 `requests.handle`。Jackson 在处理完请求后，会以事件对象作为第一个参数调用回调。您可以使用事件对象来决定要采取的操作。

```javascript showLineNumbers
const callback = (event: DirectorySyncEvent) => {
  console.log(event);
};

const { data, status } = await directorySyncController.requests.handle(
  request,
  callback
);
```

### 批量处理事件

您可以启用目录同步事件的批量处理，而不是实时接收事件到您的 webhook 端点。启用后，Jackson 会将事件排队并按批次处理。批次大小可通过 [DSYNC_WEBHOOK_BATCH_SIZE](/docs/jackson/deploy/env-variables#dsync_webhook_batch_size) 环境变量配置。

理想情况下，您会希望按计划运行事件处理端点。您可以使用 cron 作业来实现。

<Tabs>
<TabItem value="02" label="Shell">

```bash
curl -X POST \
  -H "Authorization: Api-Key YOUR_API_KEY" \
  http://localhost:5225/api/v1/dsync/cron/process-events
```

</TabItem>
</Tabs>

或者，您可以设置 `DSYNC_WEBHOOK_BATCH_CRON_INTERVAL` 环境变量。
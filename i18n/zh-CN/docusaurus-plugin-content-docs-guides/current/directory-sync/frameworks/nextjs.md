---
title: Implement Directory Sync (SCIM) to your Next.js App using Jackson
sidebar_label: Next.js
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# Next.js

本指南将教你如何在Next.js应用中实现SCIM用户配置功能。

## 快速开始

目录同步功能可帮助企业自动化管理其SaaS应用中的用户配置与注销流程。通过建立统一的用户身份数据源，该功能能显著节省组织管理时间，简化用户生命周期管理。

### 安装SAML Jackson

首先在Next.js应用中安装SAML Jackson组件。

```bash
npm i --save @boxyhq/saml-jackson
```

### 初始化SAML Jackson

新建`jackson.ts`文件用于存放Jackson初始化配置。

```javascript title="lib/jackson.ts"
import type { JacksonOption, IDirectorySyncController } from '@boxyhq/saml-jackson';
import jackson from '@boxyhq/saml-jackson';

let directorySyncController: IDirectorySyncController;

const g = global as any;

const opts = {
  externalUrl: `http://localhost:3000/`,
  samlPath: '/',
  scimPath: '/api/scim',
  db: {
    engine: 'sql',
    type: 'postgres',
    url: "postgres://username:password@localhost:5432/your-database-name",
  },
} as JacksonOption;

export default async function init() {
  if (!g.directorySyncController) {
    const ret = await jackson(opts);

    directorySyncController = ret.directorySyncController;
    g.directorySyncController = directorySyncController;
  } else {
    directorySyncController = g.directorySyncController;
  }

  return {
    directorySyncController,
  };
}
```

### 创建目录

集成第一步是为租户创建目录。

目录同步服务商（身份提供商）要求提供**SCIM基础URL**和**SCIM认证令牌**，这两个参数对每个用户创建的目录都是唯一的。

```javascript
const { data, error } = await directorySyncController.directories.create({
  name: 'any-name',
  type: 'okta-scim-v2',
  tenant: "tenant-identifier"
  product: "product-identifier"
});
```

响应示例如下：

```javascript
{
  "data": {
    "id": "58b5cd9dfaa39d47eb8f5f88631f9a629a232016",
    "name": "any-name",
    "tenant": "tenant-identifier",
    "product": "product-identifier",
    "type": "okta-scim-v2",
    "log_webhook_events": false,
    "scim": {
      "path": "/api/scim/58b5cd9dfaa39d47eb8f5f88631f9a629a232016",
      "secret": "IJzAoevjD_liiiy-VkDtXg",
      "endpoint": "http://localhost:5225/api/scim/58b5cd9dfaa39d47eb8f5f88631f9a629a232016"
    },
    "webhook": {
      "endpoint": "",
      "secret": ""
    }
  },
  "error": null
}
```

请注意JSON中的`scim.endpoint`和`scim.secret`字段值，用户在身份提供商处配置SCIM应用时需要这些参数。

通常需要提供用户界面让用户为其服务商创建目录，并展示所创建目录的`SCIM基础URL`和`SCIM认证令牌`。这类UI通常位于组织设置或团队设置模块。

参考下方演示示例：

![创建目录](/videos/create-directory.gif)

可通过调用`directorySyncController.providers()`方法获取支持的目录同步服务商列表。

### 理解SCIM API请求

实现SCIM的核心是构建RESTful API，用于接收身份提供商的SCIM配置请求，将用户和群组同步至您的应用。所有请求都将发送至`SCIM基础URL(scim.endpoint)`。

以下是您的API需要能够处理的来自身份提供商SCIM配置的请求类型（针对给定的SCIM基础URL）：

#### 用户配置

| Route      | Methods    |
| ---------- | ---------- |
| /Users     | POST       |
| /Users/:id | GET        |
| /Users/:id | PUT, PATCH |
| /Users/:id | DELETE     |

#### 推送群组及成员关系

| Route       | Methods    |
| ----------- | ---------- |
| /Groups     | POST       |
| /Groups/:id | GET        |
| /Groups/:id | PUT, PATCH |
| /Groups/:id | DELETE     |

### 处理SCIM API请求

现在添加路由来处理来自目录同步服务商的请求。

```javascript title="pages/api/scim/[...directory].ts"
import type { NextApiRequest, NextApiResponse } from 'next';
import type { DirectorySyncRequest, HTTPMethod, DirectorySyncEvent } from '@boxyhq/saml-jackson';

import jackson from '../../../lib/jackson';

export default async function handler(req: NextApiRequest, res: NextApiResponse) {
  const { directorySyncController } = await jackson();

  const { method, query, body } = req;

  const directory = query.directory as string[];
  const [directoryId, path, resourceId] = directory; // Extract the params

  // Construct the event
  const request: DirectorySyncRequest = {
    method: method as HTTPMethod,
    body: body ? JSON.parse(body) : undefined,
    directoryId: directoryId,
    resourceId: resourceId,
    resourceType: path === 'Users' ? 'users' : 'groups',
    apiSecret: extractAuthToken(req),
    query: {
      count: req.query.count ? parseInt(req.query.count as string) : undefined,
      startIndex: req.query.startIndex ? parseInt(req.query.startIndex as string) : undefined,
      filter: req.query.filter as string,
    },
  };

  // Handle the requests
  // highlight-start
  const { status, data } = await directorySyncController.requests.handle(request, async (event: DirectorySyncEvent) => {
    console.log(event); // Do something with the event
  });
  // highlight-end

  // Send the response back
  return res.status(status).json(data);
}

// Fetch the auth token from the request headers
const extractAuthToken = (req: NextApiRequest): string | null => {
  const authHeader = req.headers.authorization || null;

  return authHeader ? authHeader.split(' ')[1] : null;
};
```

`pages/api/scim/[...directory].ts`是通配路径路由，匹配参数将以查询参数形式传递给页面，且始终以数组形式传递。

重点注意：可向`directorySyncController.requests.handle`传递异步回调方法作为第二个参数。该方法的首个参数将接收SCIM事件对象。

查阅[SCIM事件与类型文档](/docs/directory-sync/events)以深入了解事件机制。

利用这些事件触发应用内操作，例如根据目录变更创建新用户或更新现有用户信息。

### 配置身份提供商

此步骤通常由终端用户自行完成。我们为每个目录同步服务商提供了[详细配置文档](/docs/directory-sync/providers/)。
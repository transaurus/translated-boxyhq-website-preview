---
title: Implement Directory Sync (SCIM) to your Express.js App using Jackson
sidebar_label: Express.js
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# Express.js

本指南将教您在Express.js应用中实现SCIM配置。

## 快速开始

目录同步帮助企业在SaaS应用中自动化管理用户的创建与删除。通过建立用户身份数据的单一真实来源，它简化了用户生命周期管理流程，为企业节省宝贵时间。

### 安装SAML Jackson

首先将SAML Jackson安装到您的Express.js应用中。

```bash
npm i --save @boxyhq/saml-jackson
```

### 初始化SAML Jackson

请注意`@boxyhq/saml-jackson`的初始化是异步操作，不能在顶层执行。请在初始化express服务器的函数中运行。

```javascript
let directorySyncController;

const opts = {
  externalUrl: `http://localhost:3000/`,
  samlPath: '/',
  scimPath: '/api/scim',
  db: {
    engine: 'sql',
    type: 'postgres',
    url: 'postgres://username:password@localhost:5432/your-database-name',
  },
};

async function init() {
  const ret = await require('@boxyhq/saml-jackson').controllers(opts);

  directorySyncController = ret.directorySyncController;
}
```

### 创建目录

集成第一步是为租户创建目录。

目录同步提供商（身份提供商）要求您提供**SCIM基础URL**和**SCIM认证令牌**。这两个值对应用用户创建的每个目录都是唯一的。

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

请注意上述JSON中的`scim.endpoint`和`scim.secret`键值。用户在身份提供商配置SCIM应用时需要这些值。

通常您需要提供UI界面让用户为其提供商创建目录，并显示用户所创建目录的`SCIM基础URL`和`SCIM认证令牌`。这类UI通常位于组织设置或团队设置中。

例如，参考下方演示。

![创建目录](/videos/create-directory.gif)

可通过调用`directorySyncController.providers()`方法获取支持的目录同步提供商列表。

### 理解SCIM API请求

实现SCIM的关键是构建RESTful API，使身份提供商的SCIM配置能调用该API将用户和群组配置到您的应用。请求将发送至`SCIM基础URL(scim.endpoint)`。

以下是您的API需要能接收的、针对给定`SCIM基础URL(scim.endpoint)`的身份提供商SCIM配置调用。

#### 用户配置

| Route      | Methods    |
| ---------- | ---------- |
| /Users     | POST       |
| /Users/:id | GET        |
| /Users/:id | PUT, PATCH |
| /Users/:id | DELETE     |

#### 推送群组及群组成员关系

| Route       | Methods    |
| ----------- | ---------- |
| /Groups     | POST       |
| /Groups/:id | GET        |
| /Groups/:id | PUT, PATCH |
| /Groups/:id | DELETE     |

### 处理SCIM API请求

确保初始化express的body解析器中间件以处理`application/scim+json`等SCIM内容类型标头。

```javascript
app.use(express.json({ type: ["application/*+json", "application/json"] }));
```

现在添加路由来处理目录同步提供商的入站请求。

```javascript
router.all(
  '/api/scim/:directoryId/:resourceType/:resourceId?',
  async (req, res, next) => {
    const { params, method, body, headers, query } = req;
    const { directoryId, resourceType, resourceId } = params;

    const authToken = headers.authorization.split(' ')[1];

    // Construct the event
    const request = {
      method: method,
      body,
      directoryId: directoryId,
      resourceId: resourceId,
      resourceType: resourceType.toLowerCase(),
      apiSecret: authToken,
      query: {
        count: query.count ? parseInt(query.count) : undefined,
        startIndex: query.startIndex ? parseInt(query.startIndex) : undefined,
        filter: query.filter,
      },
    };

    // Handle the requests
    // highlight-start
    const { status, data } = await directorySyncController.requests.handle(
      request,
      async (event) => {
        console.log(event); // Do something with the event
      }
    );
    // highlight-end

    // Send the response back
    return res.status(status).json(data);
  }
);
```

`router.all('/api/scim/:directoryId/:resourceType/:resourceId?', async (req, res, next) => {...})`是通配路径路由，匹配参数将作为参数传递给路由。

请注意高亮行，您可以将异步回调方法作为第二个参数传递给`directorySyncController.requests.handle`。该方法将以SCIM事件作为第一个参数被调用。

查看[SCIM事件与类型](/docs/directory-sync/events)文档以深入了解事件机制。

利用这些事件触发应用中的操作，例如根据目录变更在应用中创建新用户或更新现有用户。

### 配置身份提供商

您的用户通常需要在其终端完成此步骤。我们为每个目录同步提供商提供了[详细文档](/docs/directory-sync/providers/)。
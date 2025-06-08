---
title: Directory Sync Webhooks
sidebar_label: Webhooks
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# Webhook 网络钩子

Webhook 是一种 HTTP 请求，由源系统中的事件触发并发送到目标系统（例如您的应用程序），通常携带数据负载。

SAML Jackson 使用 webhook 在目录用户和组发生变更时通知您的应用程序。您可以通过 webhook 触发应用内的操作，例如根据目录中的变更在您的应用中创建新用户或更新用户信息。

## 配置 Webhook

SAML Jackson 允许您在创建目录同步连接时（通过管理门户或API）配置 webhook 地址和密钥。您可以随时修改这些配置。

- [管理门户](../admin-portal/directory-sync)
- [API参考](api-reference#create-a-new-directory)

## 接收事件

当目录用户或组发生变更时，SAML Jackson 会向您的 webhook 端点发送 POST 请求。您的 webhook 端点应能接收请求并返回 200 状态码。

通过以下步骤开始接收事件：

- 在服务器上创建 HTTP 端点（URL）作为 webhook 接收地址
- 创建目录同步连接并配置 webhook 端点
- 在身份提供商处配置 SCIM 应用
- 处理 webhook 请求并返回 200 状态码

完整事件列表请参阅[事件与类型](events)页面。

以下是典型的 webhook 请求示例：

```json
POST /your-webhook-endpoint
Content-Type: application/json
BoxyHQ-Signature: t=xxx,s=xxxx

{
  "directory_id": "58b5cd9dfaa39d47eb8f5f88631f9a629a232016",
  "event": "user.created",
  "tenant": "boxyhq",
  "product": "jackson",
  "data": {
    "id": "038e767b-9bc6-4dbd-975e-fbc38a8e7d82",
    "first_name": "Deepak",
    "last_name": "Prabhakara",
    "email": "deepak@boxyhq.com",
    "active": true,
    "raw": {...}
  }
}
```

:::caution
若 webhook 请求失败，SAML Jackson 不会自动重试。如已启用 webhook 事件日志，管理门户中将生成相关日志记录。
:::

## Webhook 安全验证

使用请求头中的签名（**BoxyHQ-Signature**）验证 webhook 请求是否来自 SAML Jackson。该签名通过目录同步连接中配置的**webhook密钥**生成。

签名包含以下逗号分隔的值：

- **t** 请求时间戳
- **s** 请求签名

签名示例如下：

```
BoxyHQ-Signature: t=1545010989801,s=xxxx
```

## 验证 Webhook 签名

按以下步骤验证签名有效性：

- 从签名中提取 **t** 和 **s** 值
- 拼接签名字符串：
  - 时间戳 (t)
  - 字符 `.`
  - 原始 JSON 负载
- 使用 webhook 密钥计算签名字符串的 `HMAC-SHA256` 哈希值
- 对比请求头中的签名与计算得到的签名

验证通过后，即可使用 webhook 负载数据执行应用内操作。

```javascript showLineNumbers
// Your webhook secret
const secret = 'your-secret-here';

// The signature header from the webhook request.
const signatureHeader =
  't=1657016083300,s=9ece981128a8f2a96f95e3d0be68c37b825d5c15f5b8982e0b7e198a76621866';

// JSON body from the webhook request.
const body = {...}

const [t, s] = signatureHeader.split(',');

const timestamp = t.split('=')[1];
const signature = s.split('=')[1];

const signedPayload = `${timestamp}.${JSON.stringify(body)}`;

const expectedSignature = crypto
  .createHmac('sha256', secret)
  .update(signedPayload)
  .digest('hex');

// Compare the expectedSignature to the signature
if(signature === expectedSignature) {
  // The webhook request is valid
} else {
  // The webhook request is invalid
}
```
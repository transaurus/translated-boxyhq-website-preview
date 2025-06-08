---
title: Enterprise SSO Webhook Events
sidebar_label: Events
---

# Webhook 事件

SAML Jackson 通过 Webhook 在连接发生变更时通知您的应用程序。我们将针对每个连接发送以下4种事件通知。

## 事件类型

<details>
<summary>
  sso.created - New connection has been created.
</summary>
<p>

```json
{
  "event": "sso.created",
  "tenant": "boxyhq",
  "product": "demo",
  "data": {
    "name": "SSO Connection",
    "description": "SSO Connection for BoxyHQ",
    "clientID": "326991705d478f0178fc2b49e35cd166dc771061",
    "clientSecret": "15b2db91b2ba4c848b68148f108035e7138d69104d99de89",
    "provider": "saml.example.com",
    "friendlyProviderName": null
  }
}
```

</p>
</details>

<details>
<summary>
  sso.deactivated - A connection has been deactivated.
</summary>
<p>

```json
{
  "event": "sso.deactivated",
  "tenant": "boxyhq",
  "product": "demo",
  "data": {
    "name": "SSO Connection",
    "description": "SSO Connection for BoxyHQ",
    "clientID": "326991705d478f0178fc2b49e35cd166dc771061",
    "clientSecret": "15b2db91b2ba4c848b68148f108035e7138d69104d99de89",
    "provider": "saml.example.com",
    "friendlyProviderName": null
  }
}
```

</p>
</details>

<details>
<summary>
  sso.activated - A connection has been activated.
</summary>
<p>

```json
{
  "event": "sso.activated",
  "tenant": "boxyhq",
  "product": "demo",
  "data": {
    "name": "SSO Connection",
    "description": "SSO Connection for BoxyHQ",
    "clientID": "326991705d478f0178fc2b49e35cd166dc771061",
    "clientSecret": "15b2db91b2ba4c848b68148f108035e7138d69104d99de89",
    "provider": "saml.example.com",
    "friendlyProviderName": null
  }
}
```

</p>
</details>

<details>
<summary>
  sso.deleted - A connection has been deleted.
</summary>
<p>

```json
{
  "event": "sso.deleted",
  "tenant": "boxyhq",
  "product": "demo",
  "data": {
    "name": "SSO Connection",
    "description": "SSO Connection for BoxyHQ",
    "clientID": "326991705d478f0178fc2b49e35cd166dc771061",
    "clientSecret": "15b2db91b2ba4c848b68148f108035e7138d69104d99de89",
    "provider": "saml.example.com",
    "friendlyProviderName": null
  }
}
```

</p>
</details>

## 配置 Webhook

配置 Webhook 需要设置以下环境变量：

- `WEBHOOK_URL` - 接收 Webhook 事件的目标 URL
- `WEBHOOK_SECRET` - 用于签名 Webhook 事件的密钥
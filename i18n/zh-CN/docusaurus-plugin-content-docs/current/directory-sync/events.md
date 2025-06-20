---
title: Directory Sync Webhook Events
sidebar_label: Events
---

# 事件

SAML Jackson 通过 Webhook 在目录、用户、群组及成员关系发生变更时通知您的应用程序。

## 用户事件

我们将通知您以下3类与用户相关的事件。每个事件都将发送至您在目录同步应用中配置的Webhook URL。

<details>
  <summary>user.created - New user has been assigned to the app.</summary>
  <p>

```json
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
    "raw": {
      "schemas": ["urn:ietf:params:scim:schemas:core:2.0:User"],
      "userName": "deepak@boxyhq.com",
      "name": {
        "givenName": "Deepak",
        "familyName": "Prabhakara"
      },
      "emails": [
        {
          "primary": true,
          "value": "deepak@boxyhq.com",
          "type": "work"
        }
      ],
      "title": "CEO",
      "displayName": "Deepak Prabhakara",
      "locale": "en-US",
      "externalId": "00u1ldzzogFkXFmvT5d7",
      "groups": [],
      "active": true,
      "id": "038e767b-9bc6-4dbd-975e-fbc38a8e7d82"
    }
  }
}
```

  </p>

</details>

<details>
  <summary>user.updated - A user's properties has been updated.</summary>
  <p>

```json
{
  "directory_id": "58b5cd9dfaa39d47eb8f5f88631f9a629a232016",
  "event": "user.updated",
  "tenant": "boxyhq",
  "product": "jackson",
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
      "active": true,
      "id": "ebc31d6e-7d62-4f81-b9e5-eb5f1a04ee92",
      "groups": []
    }
  }
}
```

  </p>
</details>

<details>
  <summary>user.deleted - A user has been removed from the Directory Provider.</summary>
  <p>

```json
{
  "directory_id": "58b5cd9dfaa39d47eb8f5f88631f9a629a232016",
  "event": "user.deleted",
  "tenant": "boxyhq",
  "product": "jackson",
  "data": {
    "id": "ebc31d6e-7d62-4f81-b9e5-eb5f1a04ee92",
    "first_name": "Kiran",
    "last_name": "Krishnan",
    "email": "kiran@boxyhq.com",
    "active": false,
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
      "active": false,
      "id": "ebc31d6e-7d62-4f81-b9e5-eb5f1a04ee92",
      "groups": []
    }
  }
}
```

  </p>
</details>

## 群组事件

我们将通知您以下5类与群组及成员关系相关的事件。每个事件都将发送至您在目录同步应用中配置的Webhook URL。

<details>
  <summary>group.created - New group has been added to the app.</summary>
  <p>

```json
{
  "directory_id": "58b5cd9dfaa39d47eb8f5f88631f9a629a232016",
  "event": "group.created",
  "tenant": "boxyhq",
  "product": "jackson",
  "data": {
    "id": "29e3adde-b4bb-45fc-bf65-2b44f29fd6f6",
    "name": "dev",
    "raw": {
      "schemas": ["urn:ietf:params:scim:schemas:core:2.0:Group"],
      "displayName": "dev",
      "members": [],
      "id": "29e3adde-b4bb-45fc-bf65-2b44f29fd6f6"
    }
  }
}
```

  </p>
</details>

<details>
  <summary>group.updated - A group's properties has been updated.</summary>
  <p>

```json
{
  "directory_id": "58b5cd9dfaa39d47eb8f5f88631f9a629a232016",
  "event": "group.updated",
  "tenant": "boxyhq",
  "product": "jackson",
  "data": {
    "id": "29e3adde-b4bb-45fc-bf65-2b44f29fd6f6",
    "name": "developers",
    "raw": {
      "schemas": ["urn:ietf:params:scim:schemas:core:2.0:Group"],
      "displayName": "developers",
      "members": [],
      "id": "29e3adde-b4bb-45fc-bf65-2b44f29fd6f6"
    }
  }
}
```

  </p>
</details>

<details>
  <summary>group.deleted - A group has been removed from the app.</summary>
  <p>

```json
{
  "directory_id": "58b5cd9dfaa39d47eb8f5f88631f9a629a232016",
  "event": "group.deleted",
  "tenant": "boxyhq",
  "product": "jackson",
  "data": {
    "id": "29e3adde-b4bb-45fc-bf65-2b44f29fd6f6",
    "name": "developers",
    "raw": {
      "schemas": ["urn:ietf:params:scim:schemas:core:2.0:Group"],
      "displayName": "developers",
      "members": [],
      "id": "29e3adde-b4bb-45fc-bf65-2b44f29fd6f6"
    }
  }
}
```

  </p>
</details>

<details>
  <summary>group.user_added - A user has been added to a directory group.</summary>
  <p>

```json
{
  "directory_id": "58b5cd9dfaa39d47eb8f5f88631f9a629a232016",
  "event": "group.user_added",
  "tenant": "boxyhq",
  "product": "jackson",
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
      "active": true,
      "id": "ebc31d6e-7d62-4f81-b9e5-eb5f1a04ee92",
      "title": "Developer",
      "groups": []
    },
    "group": {
      "id": "29e3adde-b4bb-45fc-bf65-2b44f29fd6f6",
      "name": "developers",
      "raw": {
        "schemas": ["urn:ietf:params:scim:schemas:core:2.0:Group"],
        "displayName": "developers",
        "members": [],
        "id": "29e3adde-b4bb-45fc-bf65-2b44f29fd6f6"
      }
    }
  }
}
```

  </p>
</details>

<details>
  <summary>group.user_removed - A user has been removed from a directory group.</summary>
  <p>

```json
{
  "directory_id": "58b5cd9dfaa39d47eb8f5f88631f9a629a232016",
  "event": "group.user_removed",
  "tenant": "boxyhq",
  "product": "jackson",
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
      "active": true,
      "id": "ebc31d6e-7d62-4f81-b9e5-eb5f1a04ee92",
      "title": "Developer",
      "groups": []
    },
    "group": {
      "id": "29e3adde-b4bb-45fc-bf65-2b44f29fd6f6",
      "name": "developers",
      "raw": {
        "schemas": ["urn:ietf:params:scim:schemas:core:2.0:Group"],
        "displayName": "developers",
        "members": [],
        "id": "29e3adde-b4bb-45fc-bf65-2b44f29fd6f6"
      }
    }
  }
}
```

  </p>
</details>

## 目录事件

配置Webhook需设置以下环境变量：

- `WEBHOOK_URL` - 接收Webhook事件的目标URL
- `WEBHOOK_SECRET` - 用于签名Webhook事件的密钥

我们将通知您以下4类与目录连接相关的事件：

<details>
<summary>
  dsync.created - New connection has been created.
</summary>
<p>

```json
{
  "event": "dsync.created",
  "tenant": "boxyhq",
  "product": "demo",
  "data": {
    "id": "d8aa6c93-c960-4925-9b31-4a4d2ad3bb44",
    "name": "Okta Directory",
    "type": "okta-scim-v2"
  }
}
```

</p>
</details>

<details>
<summary>
  dsync.deactivated - A connection has been deactivated.
</summary>
<p>

```json
{
  "event": "dsync.deactivated",
  "tenant": "boxyhq",
  "product": "demo",
  "data": {
    "id": "d8aa6c93-c960-4925-9b31-4a4d2ad3bb44",
    "name": "Okta Directory",
    "type": "okta-scim-v2"
  }
}
```

</p>
</details>

<details>
<summary>
  dsync.activated - A connection has been activated.
</summary>
<p>

```json
{
  "event": "dsync.activated",
  "tenant": "boxyhq",
  "product": "demo",
  "data": {
    "id": "d8aa6c93-c960-4925-9b31-4a4d2ad3bb44",
    "name": "Okta Directory",
    "type": "okta-scim-v2"
  }
}
```

</p>
</details>

<details>
<summary>
  dsync.deleted - A connection has been deleted.
</summary>
<p>

```json
{
  "event": "dsync.deleted",
  "tenant": "boxyhq",
  "product": "demo",
  "data": {
    "id": "d8aa6c93-c960-4925-9b31-4a4d2ad3bb44",
    "name": "Okta Directory",
    "type": "okta-scim-v2"
  }
}
```

</p>
</details>

## 常见问题

### 哪些身份提供商不会发送用户删除事件？

以下身份提供商不会发送`user.deleted`事件：

- Okta
- Microsoft Entra ID（原Azure AD）

### 当身份提供商未发送用户删除的Webhook事件时，如何判断用户是否被删除？

请注意，并非所有身份提供商都会发送用户删除请求。您可以通过监听SAML Jackson的`user.updated`事件，检查`active`属性值来判断用户是否被删除。当用户被删除时，该属性值将变为`false`。

### SAML Jackson支持哪个SCIM版本？

当前SAML Jackson支持SCIM 2.0版本。
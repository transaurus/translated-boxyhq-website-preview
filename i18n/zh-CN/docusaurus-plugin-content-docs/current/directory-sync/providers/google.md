---
title: Google Workspace Directory
sidebar_label: Google Workspace
---

# Google Workspace

Google Workspace 对 SCIM 的支持较为有限，因此我们直接调用其 API 来提供完整的类 SCIM 功能。

启用此功能需通过 Google 认证以获取 Admin SDK API 访问权限。创建新目录同步连接时，系统会显示 Google 认证 URL，您可将该 URL 发送给客户以完成其 Google 租户的认证。此过程无需传统 SCIM 配置。

## 自托管配置指南

若采用自托管部署，以下指南将帮助您配置 SAML Jackson 以使用 Google Workspace 作为目录同步供应商。

---

Jackson 需配置 Google OAuth 应用来访问 Admin SDK API。您可使用现有 OAuth 应用或新建一个。

### 创建 OAuth 应用

访问 [Google Cloud Console](https://console.cloud.google.com/) 并从列表中选择您的项目。

![Google Workspace DSync Step 1](/images/docs/jackson/dsync-providers/google/oauth/1.png)

从左侧菜单选择 **APIs & Services**，然后选择 **Credentials**。

从 **CREATE CREDENTIALS** 下拉菜单中选择 **OAuth client ID**。

![Google Workspace DSync Step 2](/images/docs/jackson/dsync-providers/google/oauth/2.png)

为凭证命名，并选择 **Web application** 作为 **Application type**。

添加以下 **Authorized redirect URIs** 后点击 **Create**。

`https://<your-domain>/api/scim/oauth/callback`

![Google Workspace DSync Step 3](/images/docs/jackson/dsync-providers/google/oauth/3.png)

:::info
注意：上述回调 URL 适用于将 Jackson 作为服务使用的情况。

若以 NPM 包形式使用 Jackson，需在您的应用中配置重定向 URI。详见 [Google 目录同步 API](/docs/directory-sync/api-reference#google-directory-sync)。
:::

复制 **Client ID** 和 **Client secret** 并妥善保存。

![Google Workspace DSync Step 4](/images/docs/jackson/dsync-providers/google/oauth/4.png)

:::info
请务必配置并发布 `OAuth consent screen`，否则客户将无法访问。
:::

参阅[环境变量](/docs/jackson/deploy/env-variables#directory-sync)章节了解如何配置 Jackson 使用这些参数。

完成 Jackson 配置后，即可通过 Google OAuth 认证租户并同步其 Workspace 目录。

### 启用 Admin SDK API

需按以下步骤启用 Admin SDK API 访问权限：

进入控制台的 `Enabled APIs and services` 版块。

![Google Workspace DSync Enable API Step 1](/images/docs/jackson/dsync-providers/google/oauth/enable-api-1.png)

搜索 `admin sdk api`。

![Google Workspace DSync Enable API Step 2](/images/docs/jackson/dsync-providers/google/oauth/enable-api-2.png)

选择 `Admin SDK API` 并点击 `ENABLE` 按钮。

![Google Workspace DSync Enable API Step 3](/images/docs/jackson/dsync-providers/google/oauth/enable-api-3.png)

### 配置同步计划

Jackson 可配置为按计划（例如每 2 小时）同步 Google Workspace 目录。

Jackson 服务暴露了以下 API URL 用于触发同步。您可以通过定时任务（cron job）按计划调用该 URL。

根据 Google 目录的数量，同步过程可能需要几分钟才能完成。

```bash
curl -X POST \
  -H "Authorization: Api-Key YOUR_API_KEY" \
  http://localhost:5225/api/v1/dsync/cron/sync-google
```

或者，您可以通过设置环境变量 `DSYNC_GOOGLE_CRON_INTERVAL` 来启用 Jackson 服务中的定时同步功能。

了解更多关于 [Google 目录同步 API](/docs/directory-sync/api-reference#4-sync-directory) 的信息。

### 常见问题

#### Google Workspace 是否支持实时同步？

不支持。Google Workspace 的同步并非实时进行。Jackson 会按计划（例如每 2 小时）同步 Google Workspace 目录。在自托管部署中，您可以配置定时任务按自定义计划同步目录。

#### 能否仅同步 Google Workspace 中的特定群组？

目前无法实现。Jackson 会同步 Google Workspace 中的所有群组，您需要在应用端自行筛选所需群组。
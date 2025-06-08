---
title: Getting Started
sidebar_label: Getting Started
---

有两种方式可将目录同步功能集成到您的应用程序中。

## 作为独立服务运行

运行独立服务的优势在于您能获得我们所有的UI工具（如管理门户），缺点是您需要额外管理该服务的运维工作。

**若您未使用Node.js开发，则这是唯一可选的集成方案。**

了解更多关于[如何运行Jackson服务](/docs/jackson/deploy/service)

Jackson通过Webhook机制在目录用户或群组发生变更时通知您的应用程序。您可利用Webhook触发业务逻辑，例如根据目录变更在应用中创建新用户或更新现有用户。

## 作为NPM库集成

此方案仅适用于使用Node.js开发的应用程序。

嵌入Jackson作为NPM库的优势在于无需维护独立服务，但缺点是无法直接使用我们的UI工具（如管理门户）。

```bash
npm i --save @boxyhq/saml-jackson
```

查阅[API参考文档](/docs/directory-sync/api-reference)了解可用方法详情。
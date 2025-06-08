---
title: Getting Started with SAML Jackson
description: Getting Started with SAML Single Sign-On authentication
sidebar_label: Overview
---

SAML Jackson 可与任何网络应用程序集成，实现**单点登录（SSO）认证**功能。

SAML Jackson 提供两种集成方式，您可根据实际使用场景任选其一。

## 作为[独立服务](./service.md)运行

独立服务部署的优势在于可直接获得我们全套UI工具（Next.js管理门户），开箱即用地处理SSO和目录同步连接。缺点是需额外维护独立服务的运维工作。若您未使用Node.js技术栈，此方式将成为唯一选择。

## 作为[NPM库](./npm-library.md)嵌入

此方案适用于基于Node.js开发的应用程序。

以NPM库形式嵌入Jackson的优势在于无需维护独立服务，但缺点是无法直接获得我们的UI工具套件（如管理门户）。
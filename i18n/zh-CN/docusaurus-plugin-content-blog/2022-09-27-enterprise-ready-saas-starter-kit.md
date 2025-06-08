---
slug: enterprise-ready-saas-starter-kit
title: Enterprise-ready SaaS Starter Kit
author: Kiran K
author_title: Senior Developer @BoxyHQ
author_url: https://twitter.com/tokirankrishnan
author_image_url: /images/authors/kiran.jpg
tags_disabled: [enterprise-readiness, nextjs-template, saas-starter-kit]
---

企业级SaaS入门套件是基于**Next.js**的SaaS开发框架，可帮助您在构建[企业级SaaS应用](/blog/enterprise-readiness-made-simple)时节省数百小时的开发时间。

[使用Next.js SaaS入门套件快速启动企业应用开发](https://github.com/boxyhq/saas-starter-kit)

观看[视频教程](https://youtu.be/oF8QIwQIhyo)，了解如何配置本地环境并运行该SaaS套件提供的企业级功能。

## SAML单点登录

SAML（安全断言标记语言）是一种基于XML的开放标准，用于在身份提供商（IdP）和服务提供商（SP）之间传输身份数据。

[单点登录(SSO)](/blog/understanding-saml-sso-the-basics-from-the-solution-providers-side)允许客户在外部管理团队成员，而无需依赖您内置的用户表。

SAML单点登录通过[SAML Jackson](https://github.com/boxyhq/jackson)实现集成

## 目录同步(SCIM)

[目录同步](/blog/understanding-scim-and-directory-sync)可帮助组织自动化实现用户配置与撤销配置。

它能优化用户生命周期管理流程，通过节省宝贵时间、创建统一的用户身份数据源、保障数据安全来提升效率。

目录同步功能通过[SAML Jackson](https://github.com/boxyhq/jackson#directory-sync)实现集成

## 审计日志

为您的企业客户提供记录和检索应用内事件的能力。通过我们的简易集成，您可以快速满足合规要求，避免团队从零构建复杂解决方案。

组织能够监控数据流动并及时获知安全漏洞。审计日志有助于精确定位信息滥用行为，确保数据策略得到遵循。

## Webhook与事件

Webhook是一种系统通知机制，当您的SaaS应用中发生特定事件时，无需接收请求即可通知外部应用程序。

当客户端无法预知事件发生时间并需要实时通知时，Webhook是最佳解决方案。

Webhook功能通过[Svix](https://github.com/svix/svix-webhooks)实现集成

## 团队管理

团队功能描述了现代软件中实现协作与管理的能力。

通过团队功能，SaaS应用用户可以创建账户并邀请他人共同使用应用程序。

## 其他功能

现在让我们看看该SaaS套件提供的其他常规功能 🥇

- 创建账户
- 邮箱密码登录
- 魔法链接登录
- SAML单点登录
- Google登录[[配置Google OAuth](https://support.google.com/cloud/answer/6158849?hl=en)]
- GitHub登录[[创建Github OAuth应用](https://docs.github.com/en/developers/apps/building-oauth-apps/creating-an-oauth-app)]
- 目录同步(SCIM)
- 更新账户
- 创建团队
- 邀请团队成员
- 管理团队成员
- 更新团队设置
- Webhook与事件
- 国际化支持
- 审计日志
- 角色与权限
- 深色模式

## 快速开始

请按照以下简单步骤在本地运行项目。

### 1. 环境准备

克隆或fork该GitHub仓库

```bash
git clone https://github.com/boxyhq/saas-starter-kit.git
```

### 2. 进入项目目录

```bash
cd saas-starter-kit
```

### 3. 安装依赖项

```bash
npm install
```

### 4. 配置.env环境文件

将`.env.example`复制为`.env`。

```bash
cp .env.example .env
```

### 5. 设置数据库结构

```bash
npx prisma db push
```

### 6. 启动服务端

在开发环境中运行：

```bash
npm run dev
```

### 7. 启动Prisma Studio

Prisma Studio是数据库数据的可视化编辑器。

```bash
npx prisma studio
```

## 贡献指南

贡献使开源社区成为学习、启发和创造的绝佳场所。任何贡献都将受到整个社区的高度赞赏。

我们诚邀用户和社区成员为[saas-starter-kit](https://github.com/boxyhq/saas-starter-kit)代码库贡献力量。

如需更多帮助或有关构建SaaS应用的疑问，请加入我们的[社区Discord](https://discord.boxyhq.com)。
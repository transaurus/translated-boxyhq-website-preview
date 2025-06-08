---
slug: understanding-scim-and-directory-sync
title: Understanding SCIM and Directory Sync
author: Kiran K
author_title: Senior Developer @BoxyHQ
author_url: https://twitter.com/tokirankrishnan
author_image_url: /images/authors/kiran.jpg
tags_disabled:
  [enterprise-readiness, engineering, saml-jackson, directory-sync, scim]
image: /images/docs/jackson/directory-sync-flow.png
---

本文将带您全面了解SCIM与目录同步的所有关键知识。

## 什么是SCIM？

跨域身份管理系统（SCIM）是一项开放标准，用于实现用户配置（目录同步）的自动化。

SCIM促进了身份提供商（如Okta、OneLogin等）与服务提供商（企业级SaaS应用）之间的用户身份数据通信。

## 为什么需要关注SCIM？

许多公司仍采用人工方式管理用户入职和离职，这消耗了大量IT资源和时间。

用户生命周期管理（ULM）从员工入职开始持续到离职，核心在于根据人员变动动态配置和撤销用户权限。

对IT部门而言，这意味着需要在新员工入职时为其创建所需云应用账户，并在员工离职时确保及时撤销这些账户的访问权限。

小型组织尚可手动处理这些流程，但在大规模场景下，这种方式容易出错且效率低下。此时SCIM与目录同步技术便成为企业的救星。

## 什么是目录同步？

目录同步帮助组织自动化实现用户的配置与权限撤销。

通过建立用户身份数据的单一可信源，该技术不仅能节省大量工时，还能确保数据安全，从而优化用户生命周期管理流程。

## 目录同步如何运作？

目录同步通过标准协议（SCIM）实现对用户身份数据的程序化访问。

典型实现包含客户端（身份提供商IDP，维护用户身份目录）与服务端（企业SaaS应用，需要获取部分身份数据）。

当用户身份信息变更时，这些变更会根据SCIM协议自动同步至服务提供商，后者可据此进行相应数据处理。

常见变更包括：新增组织成员、更新现有用户、移除离职人员、调整用户组归属等。

从技术角度看，SCIM是一组API接口，支持通过REST端点以JSON格式操作用户和群组对象数据。

## 如何为SaaS应用添加目录同步？

自主构建目录同步实现对于任何工程团队都将是繁琐且耗时的过程。

工程团队在启动开发前需明确以下问题：

- 如何支持多身份提供商？
- 需要投入多少资源进行支持维护？
- 团队是否具备SCIM及安全领域的专业知识？
- 自主开发目录同步是否值得？

简言之，自主构建目录同步并非明智之选，这需要大量工程投入和持续维护。

最佳替代方案是采用BoxyHQ旗下SAML Jackson等开源的专业目录同步实现方案。

## BoxyHQ目录同步方案

BoxyHQ可帮助企业仅用几行代码即可为SaaS应用添加目录同步功能。

企业SaaS应用只需向SAML Jackson提供Webhook端点，当IDP中的用户身份数据发生变更时，Jackson就会实时通知该Webhook。

![目录同步流程](/images/docs/jackson/directory-sync-flow.png)

简而言之，Jackson封装了目录同步实现的所有复杂细节，使您的集成工作变得简单高效。

BoxyHQ的目录同步实现支持Okta、OneLogin、Azure AD和JumpCloud等身份提供商。我们期待在未来的版本中添加更多身份提供商。

以下是使[BoxyHQ的目录同步实现](/docs/directory-sync/overview)脱颖而出的几个关键方面。

- 开源解决方案。
- 允许您保持对数据的控制。
- 支持多个身份提供商。
- 允许您监听六个关键的SCIM事件。

## 最终思考

目录同步对于每个寻求用户生命周期管理系统效率和安全的组织来说都是一项有价值的投资。

如果您有兴趣无麻烦地实现企业级准备，那么让我们聊聊吧！您可以预约一次免费咨询电话，与我们的CEO讨论我们如何提供帮助。让我们一起开始这段旅程。
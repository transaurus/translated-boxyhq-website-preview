---
description: 'What is SAML? What role does it play in enterprise single sign-on (SSO)? Learn what these technologies are and how BoxyHQ simplifies the implementation process for developers.'
slug: what-is-saml-and-why-you-should-care
title: 'What is SAML and why you should care'
tags_disabled: [saml, boxyhq, single-sign-on, sso, open-source, developerfirst]
image: /images/blog/what-is-saml.png
hide_table_of_contents: false
author: Schalk Neethling
author_title: Customer Success, Community, DevRel - @BoxyHQ
author_url: https://github.com/schalkneethling
author_image_url: /images/authors/schalk.jpg
---

import LearnMoreCTA from "../src/components/ctas/LearnMoreCTA";
import Picture from '../src/components/Picture';
import SignupCTA from "../src/components/ctas/SignupCTA";

在当今数字世界中，安全认证不仅是必需品，更是服务与用户之间信任的基石。对于许多组织而言，驾驭SAML的复杂性并实施稳健的单点登录(SSO)解决方案可能令人望而生畏。本文探讨了SAML在增强网络安全中的关键作用、企业在采用过程中面临的挑战，并介绍了BoxyHQ简化这一流程的解决方案，使先进安全技术变得触手可及。

## 从宏观视角看SAML

SAML（安全断言标记语言）是一种开放标准，利用XML定义认证和授权数据交换框架。其最新版本SAML 2.0主要用于Web浏览器单点登录，允许用户使用同一组凭证访问多个服务。这一功能对我们尤为重要，因为[Web浏览器SSO配置文件](https://en.wikipedia.org/wiki/SAML_2.0#Web_browser_SSO_profile)正是我们关注的核心领域，它能实现跨Web应用的无缝安全访问。

## 为什么选择SAML？

在相同安全域内实现单点登录较为简单，例如可通过cookie实现。但当需要跨多个安全域实施单点登录时，该过程会迅速变得复杂且易出错。尽管跨域单点登录存在实际需求，但过去解决此问题的尝试导致了大量无法互操作的自定义实现方案泛滥。

这促使了Web浏览器SSO配置文件的定义和标准化以提升互操作性。此外，SAML明确规定了三种角色：主体(principal)、身份提供商(IdP)和服务提供商(SP)。主体通常也被称为主题(subject)，一般是人类用户，但也可能是其他应用程序。身份提供商是关于主体的集中式权威数据源，常见的有Keycloak、Aerobase、Gluu和Okta等。而服务提供商？如果您正在构建SaaS应用，那么您就是服务提供商！

最后，正如其名称所示，SAML用于生成断言(assertions)——这个术语在编写测试代码时可能较为熟悉。在SAML语境中，断言是针对主体作出的声明，主要是使服务提供商能够做出访问控制决策的断言。

综上所述，SAML是一种通过开放、可互操作的标准实现跨安全域单点登录的方法，该标准定义了标记语言、协议、绑定和配置文件。我们利用所有这些要素，通过查询身份提供商来生成关于主体的断言，从而使服务提供商能够做出明智且可信的访问控制决策。

## SAML认证流程

SAML在认证流程中的作用比其各个组成部分的总和更为简单，让我们通过示例来解析。在具体讨论SAML之前，我们需要先退一步理解单点登录的本质。

### 什么是单点登录？

单点登录正如其字面含义所示，是通过单一凭证登录多个应用的方法。让我们逐步解析这个概念，随后再叠加SAML层。当加入公司时，您通常会获得多个应用的访问权限。公司授予这些应用访问权限的一种方式是让您在每个提供商处使用公司邮箱和自设密码创建账户。这种情况如下图所示：

<Picture alt="llustration of a person facing three separate authentication processes leading to different services highlighting the complexity without Single Sign-On." pictureSrc="/images/blog/no-single-sign-on" height="613" width="1229" />

即使使用密码管理器，您仍需管理三个独立账户。当您离职时，公司还面临确保您在每个服务提供商的访问权限被终止的问题。因此，出于安全性和用户体验改善的考虑，企业选择使用身份提供商。

通过这个身份提供商，您只需认证一次（单点登录），随后即可使用同一组凭证访问每个应用程序。您无需在服务提供商处进行认证，而是在身份提供商处完成一次认证后，由身份提供商将您重定向至服务提供商。当您离职时，公司只需在身份提供商处撤销您的账户即可终止所有访问权限。整个流程大致如下：

<Picture alt="Graphic showing streamlined authentication with Single Sign-On where a figure is connected to a single IdP, which leads to different services symbolizing simplified access to multiple services." pictureSrc="/images/blog/single-sign-on" height="613" width="1229" />

简而言之，这就是单点登录的核心机制。相信您已经理解为何这对企业和用户而言都是重大进步。

<LearnMoreCTA label="Learn more about Enterprise SSO" url="/enterprise-sso" />

### SAML的作用

那么SAML在其中扮演什么角色？SAML是实现身份提供商与服务提供商之间认证与通信的桥梁。它使得服务提供商能够确认主体（用户）身份的真实性，从而做出访问控制决策。

SAML认证流程有两种触发方式。最常用的是IdP发起的流程（IdP-initiated flow），即用户从身份提供商处开始认证，随后携带SAML响应被定向至服务提供商。您可能在工作中见过这种场景：登录公司仪表盘(sso.mycompany.com)后，所有可访问的应用程序会集中展示，点击任意应用都无需重复登录。

这是因为服务提供商会配置断言消费服务(ACS)，用于解析和验证来自IdP的SAML响应。此时IdP需要生成SAML响应，而服务提供商需要具备解析验证能力。值得注意的是，此流程中IdP和服务提供商都未发起SAML请求。

另一种SP发起的流程(SP-initiated flow)则会用到SAML请求。用户从服务提供商处开始访问，服务提供商生成SAML请求并将用户重定向至预配置的IdP端点。IdP完成用户认证后，会像之前一样携带SAML响应将用户返回服务提供商。虽然支持的流程通常由服务提供商决定，但两者兼存的情况很常见，这意味着IdP和服务提供商都需要具备请求生成与响应处理能力。

## 关于SAML的现状

具体技术细节将另文详述，但必须指出整个体系包含多个复杂组件。如果您是面向企业或政府机构提供SaaS服务的厂商，极有可能需要支持SAML单点登录。

### 为何特别强调这点？

因为SAML有个关键特性尚未提及：作为较旧的标准（2.0版发布于2005年），它正逐步被WebAuthn、OAuth2及基于OAuth2的OpenID Connect等现代标准取代。

因此现代工具、框架和企业更倾向采用新标准。但如同所有技术演进，迁移过程需要时间成本且伴随风险。虽然没有精确统计数据，但在医疗、金融和政府机构等领域，SAML仍是主流解决方案并将持续存在。

需要说明的是，这些行业绝非仅存的SAML用户。即便在科技领域，微软等企业仍在使用基于SAML的系统和流程。因此支持SAML对避免错失商业机会至关重要。

那么该如何应对这一挑战？一种方案是立即实施支持，或在需求出现时及时响应。除非这是您的核心业务，否则这样做会给现有及未来的资源带来巨大压力——因为与产品其他模块一样，SAML实现需要持续维护，并处理随时可能出现的边界情况。尽管SAML作为开放标准已存在20余年，但不同实现之间仍存在细微差异。

## BoxyHQ如何解决这个问题？

与其耗费宝贵的脑力、资源和资金自行解决所有问题，不如让BoxyHQ充当您与基于SAML的身份提供商之间的代理桥梁。这意味着您的应用程序只需兼容OAuth2或OpenID Connect协议。BoxyHQ的SAML Jackson组件将负责这些协议与SAML之间的转换，同时处理不同实现和身份提供商之间的兼容性问题。

<Picture alt="An image depicting a diagram illustrating single sign-on integration using BoxyHQ, with multiple Identity Providers (IdPs) connected through a single API for simplified authentication." pictureSrc="/images/blog/with-boxyhq" height="613" width="1229" />

通过BoxyHQ作为代理服务提供商居中协调，您可以在开发应用时完全无需考虑SAML协议或认证流程的发起方式。此外，您还将获得一个管理门户，用于配置产品与连接关系——包括可生成共享链接的设置功能，使您能在不暴露敏感信息的前提下快速建立SSO连接。

立即免费试用我们的SaaS产品，当您准备正式使用时，既可继续作为SaaS客户由我们托管基础设施及处理SAML事务，也可基于我们的开源特性选择自主部署方案，后续还能为自托管实例启用更多企业级功能和高级支持。产品详情请访问定价页面，或立即联系我们，为未来增长开启新机遇。

<SignupCTA campaign="what-is-saml" />
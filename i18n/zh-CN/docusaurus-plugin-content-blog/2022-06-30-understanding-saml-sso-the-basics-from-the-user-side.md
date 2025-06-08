---
slug: understanding-saml-sso-the-basics-from-the-user-side
title: Understanding SAML SSO, the basics from the user side
author: Jay Singh
author_title: Marketing Consultant
author_url: https://www.linkedin.com/in/jaydsingh/
author_image_url: /images/authors/jay.jpg
tags_disabled: [enterprise-readiness, saml, saml-jackson, sso, single-sign on]
image: /img/blog/sso/with-saml.png
---

作为一名长期从事技术工作的人，我一直需要理解我们构建产品的技术本质。这个过程对我来说总是过于复杂，因此我现在总是尝试为像我这样的非技术人员编写指南。事实证明，一旦你理解了它，就能更容易地向其他非技术人员解释！那么，让我尝试解释SAML（安全断言标记语言）单点登录（SSO），以及为什么BoxyHQ能如此轻松地实现它。首先，你可能不仅听说过SAML，还听说过OAuth 2.0和OIDC，这些都是实现单点登录的协议。它们之间有一些细微差别，但为了简单起见，本文不深入讨论这些细节。

让我们从SAML SSO是什么以及它的作用开始。SAML SSO的一个实际例子是：你公司的用户登录到一个统一的仪表盘，里面包含了他们使用的所有外部服务的图标，比如CRM（如Hubspot）和会计软件（如Xero）。用户现在只需点击这些图标即可登录任何服务，而无需单独登录每一个。

但这是如何实现的呢？SAML SSO的核心思想是通过将访问集中到一个外部系统，可以更好地管理访问权限并提升安全性。在我们的例子中，那个允许用户点击图标直接登录的仪表盘就是SAML的实际应用。因为公司已经通过SAML连接到其外部服务，现在可以让用户从一个统一的入口访问所有服务。这个统一的入口被称为身份提供商（IdP），它通过SAML认证对所有其他服务的访问。

下面的图表展示了使用和不使用SAML时的访问流程：
![不使用SAML SSO](/img/blog/sso/without-saml.png)

在上图中可以看到，公司没有使用SAML，因此用户必须为每个服务单独输入用户名和密码登录。用户名和密码由服务提供商管理，访问权限也由服务提供商的管理员控制。用户需要从每个服务获取访问权限，并记住每个服务的登录信息。

![使用SAML SSO](/img/blog/sso/with-saml.png)

在上图中可以看到，公司使用了Okta等身份提供商（IdP），用户只需登录一次，就可以从一个统一的仪表盘访问所有外部服务。这也意味着公司管理员可以直接通过IdP管理对不同服务的访问权限。

请注意，这只是对SAML的高层次概述，其背后的技术细节可能会复杂得多。

我们一直从公司用户的角度来看SAML，但同样重要的是，服务提供商也需要构建SAML集成，以便连接到客户的IdP。这对服务提供商来说可能是一个非常耗时且漫长的过程，而这正是BoxyHQ的用武之地。服务提供商无需为每个客户使用的IdP构建定制集成（这可能需要数月时间），而是可以使用BoxyHQ，通过一次集成即可连接到所有需要的IdP！你可以在短短8天内实现SAML就绪！想了解具体实现方式，请查看我的另一篇博客[这里](./2022-06-30-understanding-saml-sso-the-basics-from-the-solution-providers-side.md)。

那么，SAML的主要优势是什么？以下是我总结的三个最重要的方面。

#### 增强安全性

SAML本质上是一种安全标准，它通过在安全环境中提供单一认证点，为你的服务增加了一层额外的安全性，这是大多数企业客户会要求的。

#### 提升用户体验

作为用户，使用SAML非常简单且体验愉悦，因为只需登录一次，之后就能通过仪表盘一键访问所有外部服务。这为用户节省了时间，并显著提升了产品使用体验。

#### 降低成本

在没有SAML的情况下，您需要跨多个服务维护账户信息；而使用SAML后，这些全部由身份提供商（IdP）集中管理。

BoxyHQ是开源平台，我们的SAML SSO产品SAML Jackson只是我们帮助企业实现企业级就绪的首个解决方案。如果您想探讨认证策略或部署SAML SSO，可以[在此](https://cal.com/deepak-boxyhq/demo)预约与CEO的会议，了解我们如何提供支持。

希望这篇关于SAML及其用例的高阶解读对您有所帮助。如有任何疑问，欢迎通过我们[官网](https://boxyhq.com/)的在线聊天随时联系。
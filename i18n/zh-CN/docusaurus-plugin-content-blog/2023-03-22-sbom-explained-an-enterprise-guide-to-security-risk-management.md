---
slug: sbom-explained-an-enterprise-guide-to-security-risk-management
title: 'SBOM Explained: An enterprise guide to security risk management'
author: Nathan Tarbert
author_title: Community Success Engineer @BoxyHQ
author_url: https://github.com/NathanTarbert
author_image_url: /images/authors/nathan.jpg
tags_disabled: [open-source]
image: /img/blog/security-lock.jpg
---

在这个技术时代，软件公司正迅速转向严格的合规姿态。你可能会问，为什么会出现这种趋势？过去几年间发生了什么变化？这背后有多重因素，但主要可归结为以下四大类别。

- 安全性
- 产品开发
- 合规性
- 供应链与风险管理

## 什么是SBOM（软件物料清单）？

你可以将现代应用程序想象成正在烘焙的蛋糕。用于制作蛋糕的原料清单就是SBOM（软件物料清单）。对于软件而言，这些原料就是开源或私有依赖项以及第三方组件。这份清单可能非常庞大，从数百到数千不等，尤其是考虑到传递性依赖（组件所依赖的间接依赖项）时。这份清单会变得极其复杂，在企业层面几乎无法规模化管理。大多数企业或中小型组织从头编写应用程序的时代已经过去。如今，大多数应用程序都是使用预构建的软件包（即开源组件）组装而成。过去十年间，这导致公开可用的软件组件和敏捷开发的采用率大幅提升。

## 安全性

由于针对企业基础设施的网络攻击几乎成为日常事件，越来越多的组织将安全性置于首位。公司面临着影响成本、效率和生产力的重大决策。代码库存储在云端（无论是私有还是公有）存在巨大的风险和责任。关于风险的黄金法则是：问题不在于"是否"会被入侵，而在于"何时"会被入侵。因此，必须采取预防措施，为软件基础设施添加多层防护。

SBOM能为企业提供更好的可见性，帮助识别和跟踪安全漏洞，从而及时打补丁。代码更新时应触发生成新的报告（SBOM）。

我们可以深入探讨各种预防措施，例如扫描应用程序中的漏洞或进行渗透测试。对于所有面向企业的组织，推荐采用[最小可行安全产品](https://mvsp.dev/)。简单的措施可能包括部署[审计日志](https://github.com/retracedhq/retraced)来记录和搜索应用程序内部事件。

## 产品开发

SBOM通过提供产品中所有开源依赖项和组件的完整清单，能够改进并助力软件开发流程。从组织角度来看，确保所有软件及相关组件保持最新且正确授权至关重要。清晰了解这些内容是降低风险的关键，有助于理解产品的构成和构建方式。
因此，开发团队可以快速提升透明度、沟通效率和协作决策能力，从而缩短上市时间，同时确保符合行业合规标准和法规。

![matrix finger](/img/blog/matrix-finger.jpg)

## 合规性

2021年，美国政府发布了一项[行政命令](https://www.gsa.gov/technology/it-contract-vehicles-and-purchasing-programs/information-technology-category/it-security/executive-order-14028)，要求所有与美国政府有业务往来的软件公司必须提供详细的清单（软件物料清单或SBOM），列出其生产并销售给任何联邦机构的所有软件相关组件。该立法旨在提高透明度，以加强供应链基础设施的安全性。这迅速促使其他国家考虑类似的立法。

## 供应链与风险管理

采取主动措施并自上而下地了解软件生态系统中所有组件的构成至关重要。有鉴于此，SBOM正迅速成为软件供应链风险管理的标准。随着复杂攻击的增多，在组织安全方面采取系统化方法比以往任何时候都更加重要。

生态系统正在不断发展，涌现出许多工具，可帮助你验证、扫描和监控所有组件及清单文件。

如果您有一款面向企业或公共部门销售的开源产品，就必须预先考虑合规性要求，并将SBOM（软件物料清单）集成到产品中。尤其是在面向高度监管行业的实体时更需如此。基于这一考量，BoxyHQ为我们旗下四大产品（企业级单点登录、目录同步、审计日志和数据隐私保险库）均采取了这一策略，并提供了配套指南以协助您做出决策。我们选择将[Cosign](https://www.sigstore.dev/)直接构建到工具中——当然市面上还有更多可选方案，这里有一份[精选的SBOM相关工具列表](https://github.com/awesomeSBOM/awesome-sbom)。

欢迎访问BoxyHQ官网了解更多关于为产品生成SBOM的信息。例如我们的企业级SSO的SBOM文档：[BoxyHQ](https://boxyhq.com/docs/jackson/sbom)
---
slug: the-ikea-effect-in-software-engineering
title: The IKEA effect in Software Engineering
author: Deepak Prabhakara
author_title: Co-founder & CEO @BoxyHQ
author_url: https://github.com/deepakprabhakara
author_image_url: /images/authors/deepak.jpg
tags_disabled: [enterprise-readiness, ikea-effect]
image: /img/blog/jay-wennington-BdeMttZx6Fs-unsplash.jpg
---

最近我不得不重新布置家庭办公室，于是决定去最近的宜家逛逛。Micke、Malm、Brusali、Alex和Bekant系列琳琅满目的书桌只是这段旅程的开始。我知道我得把书桌运回家，找个合适的地方拆箱，翻出螺丝刀、锤子、内六角扳手，最后花上几个小时组装。虽然享受这个过程，但也不乏挫败感。最终，由于倾注了劳动，我对这张书桌格外珍视。

![宜家组装](/img/blog/jay-wennington-BdeMttZx6Fs-unsplash.jpg)

<div style={{fontSize: "10px", marginTop: "-10px", paddingBottom: "20px"}}>照片由<a href="https://unsplash.com/@jaywennington?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Jay Wennington</a>拍摄，发布于<a href="https://unsplash.com/?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a></div>

这就是"宜家效应"——一种认知偏差，我们会对自己参与创造的产品赋予不成比例的高价值。这使得人们更珍视自己参与构建的事物。作为开发者，我们每天都在面对这种挑战，即经典的"自建还是采购"决策。在理想世界中我们当然希望亲手打造一切，但现实中必须权衡业务需求、有限资源和交付期限。真正的挑战在于：如何在构建核心业务与外包非核心任务之间找到平衡点。

就像半成品宜家书桌毫无用处一样，如果我们陷入功能实现的细节泥潭而无法完成，那么开发中的功能同样失去价值。在创建BoxyHQ过程中，我们不断听到初创企业关于"企业级就绪"的痛点——他们需要满足企业客户的各种合规、安全和治理要求，开发者30-40%的时间都耗费在实现细节上。这些初创公司本应组建团队专注于核心业务创新，但现实是非核心功能开始争夺资源，最终这些功能往往只是勉强达标，永远无法达到应有的创新高度。

我们见证了软件吞噬世界的过程。这导致安全漏洞的攻击面扩大，进而推动企业界对安全议题的空前重视。安全已成为所有公司的首要任务，不再仅是CISO团队的职责。这意味着开发者面临合规与数据安全的"左移"趋势，而BoxyHQ正以创新方式解决这个问题。我们提供即插即用的非核心功能模块：简单的自托管服务和API，开发者可以轻松集成到现有技术栈中。从此不必再为SAML登录流程的正确实现、健壮审计日志服务的构建扩展，或是保护PII数据的最佳加密方案而困扰。

BoxyHQ将为您打造所有"宜家书桌"，让您无需亲自动手。只需坐在预装好的书桌前，继续专注最初想要构建的酷炫产品。

附：隆重推出SAML Jackson（谁不爱《低俗小说》呢），这项SAML单点登录服务可无缝对接OAuth 2.0流程，并抽象掉SAML协议中繁琐的XML部分。查看[我们的代码库](https://github.com/boxyhq/jackson)和[演示示例](https://github.com/boxyhq/jackson-examples/tree/main/apps/next-auth)。若SAML SSO目前与您无关，别忘了收藏我们以便回访。我们正在构建**"DevSecMesh"**安全网格，未来数月将推出更多激动人心的功能。
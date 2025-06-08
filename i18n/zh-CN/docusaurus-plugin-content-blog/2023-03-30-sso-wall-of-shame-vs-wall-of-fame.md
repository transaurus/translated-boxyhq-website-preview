---
slug: sso-wall-of-shame-vs-wall-of-fame
title: 'SSO "Wall of Shame" vs "Wall of Fame"'
tags_disabled:
  [developer, security, cybersecurity, devsecops, appsec, sso, open-source]
image: /img/blog/shame-vs-fame.png
authors:
  - name: Sama - Carlos Samame
    title: Co-founder & COO @BoxyHQ
    url: https://www.linkedin.com/in/samame
    image_url: /images/authors/sama.jpg
  - name: Deepak Prabhakara
    title: Co-founder & CEO @BoxyHQ
    url: https://github.com/deepakprabhakara
    image_url: /images/authors/deepak.jpg
---

除非你与世隔绝，否则很可能听说过"SSO耻辱墙"。这份名单记录了那些将单点登录(SSO)视为奢侈品而非核心安全需求的供应商。上榜企业招致诸多非议实属必然——在经济下行、安全隐私至关重要的当下，许多机构却将此视为创收良机。

以下按字母顺序列举了部分登上"耻辱墙"的知名企业（参见下方截图）。完整名单详见[sso.tax](https://sso.tax)。显然，对企业SSO和审计日志（追踪关键事件）等安全功能加价，不过是其盈利模式的冰山一角。

![sso税单列表](/img/blog/sso-tax-list.png)

但此举是否正当？外界难以评判，尤其对需要盈利增长的风投支持企业而言。然而在BoxyHQ（同为风投背景），我们坚信能做得更好。网络安全税应当终止，我们更应聚焦提升安全防护、创造积极影响。以Hubspot为例，其SSO功能溢价高达6300%！这充分暴露了某些企业的荒谬与贪婪。

那么"荣誉墙"呢？这份独立名单记录了以身作则、不剥削客户的典范企业。简单搜索就能发现Grafana、Cal.com、Sumo Logic等值得玩味的名字。

理解初创企业的定价逻辑，需考量企业级交易流程：RFP、安全问卷等合规程序会让初创公司疲于应付，这似乎能合理化分级定价。但随产品成熟，企业级功能应超越SSO这类基础配置——毕竟构建维护SAML协议本就耗时费力，还要应对大型企业团队入驻的客服压力。

理想状态下，我们期待看到完整的SSO荣誉墙，以此支持那些未被贪婪吞噬的诚信企业。虽无力主导此倡议，我们仍可提供实用建议：

- 对企业核心功能而非SSO收费（若产品尚未成熟请继续阅读）
- 将企业级安全合规流程本身设为付费项，而非SSO功能
- 按用户席位分级定价（SMB席位通常较少）
- 若企业版非席位制，可采用用量计费模式

我们正通过开源企业级SSO解决方案SAML Jackson贡献力量，开发者仅需几行代码即可集成。欢迎访问[GitHub仓库](https://github.com/boxyhq/jackson)，您的反馈与星标将助力提升安全意识🙂

谨记：守护安全，践行善举，远离SSO税的黑暗面！
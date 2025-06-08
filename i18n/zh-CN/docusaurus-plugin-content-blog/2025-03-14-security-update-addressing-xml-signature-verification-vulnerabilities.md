---
slug: security-update-addressing-xml-signature-verification-vulnerabilities
title: 'Security Update: Addressing XML Signature Verification Vulnerabilities'
tags_disabled: [developer, security, cybersecurity, saml]
image: /img/blog/matthew-henry-fPxOowbR6ls-unsplash.jpg
author: Deepak Prabhakara
author_title: Co-founder & CEO @BoxyHQ
author_url: https://github.com/deepakprabhakara
author_image_url: /images/authors/deepak.jpg
---

在BoxyHQ，安全是我们的首要任务。最近，我们使用的一个库中披露了一个漏洞。得益于我们团队、[Alexander Tan](https://hackerone.com/ahacker1)、库维护者Chris Barth、WorkOS以及其他供应商之间的协调努力，我们高效且负责任地修复了该问题。

没有客户受到影响，我们的多租户安全措施为此类风险提供了额外防护。

## 漏洞详情

通过DigestValue注释绕过XML签名验证 [GHSA-x3m8-899r-f7c3](https://github.com/node-saml/xml-crypto/security/advisories/GHSA-x3m8-899r-f7c3)

- 该漏洞允许攻击者通过在`<DigestValue>`元素内注入XML注释来绕过XML签名验证。
- 可能导致篡改或恶意的XML文档被误认为有效，从而引发数据操纵或身份验证绕过等安全风险。
- 已通过严格解析和验证`<DigestValue>`修复该问题。

通过多个SignedInfo引用绕过XML签名验证 [GHSA-9p8x-f768-wp2g](https://github.com/node-saml/xml-crypto/security/advisories/GHSA-9p8x-f768-wp2g)

- 当XML签名包含多个`<SignedInfo>`引用时，会导致验证过程出现歧义。
- 攻击者可利用此漏洞诱使验证机制对文档的非目标部分进行验证。
- 修复方案通过强制严格处理`<SignedInfo>`引用来防止此类绕过技术。

这些漏洞凸显了强健的XML签名验证对防止篡改和未授权访问的重要性。

## 协同披露实践

软件中的安全漏洞不可避免，但处理方式至关重要。我们的流程优先考虑透明度、协作和快速修复：

- 验证问题并评估其影响
- 与WorkOS及其他供应商协调降低风险
- 在开发完整修复方案期间部署临时补丁
- 与Alexander Tan及库维护者合作解决问题
- 发布最终补丁并向受影响用户提供明确指南

这种结构化方法确保了快速响应，同时将风险降至最低。

## 安全是社区的共同事业

网络安全依赖于协作。Alexander负责任的披露、维护者的迅速响应以及供应商的合作对有效解决此问题至关重要。开放的协作强化了软件生态系统，使我们能够主动应对威胁。

## 展望未来

我们感谢促成此次修复的集体努力——特别感谢Alexander Tan和WorkOS的贡献。我们对安全的承诺坚定不移，并鼓励开发者和研究人员就潜在风险展开开放对话。

如有安全问题，请联系security@boxyhq.com。关于该漏洞利用和缓解措施的详细博文即将发布。

感谢所有为解决问题做出贡献的人！
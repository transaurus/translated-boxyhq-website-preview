---
slug: changelog-retraced-2023
title: 'BoxyHQ 2023 Changelog: Enhancing Security with Retraced-Powered Audit Logs'
author: Schalk Neethling
author_title: Customer Success, Community, DevRel - @BoxyHQ
author_url: https://github.com/schalkneethling
author_image_url: /images/authors/schalk.jpg
tags_disabled: [changelog]
image: /img/blog/retraced-2023-changelog.jpg
---

BoxyHQ的审计日志功能由[Retraced](https://github.com/retracedhq/retraced)提供支持，秉承开源精神，该项目是我们与[Replicated](https://www.replicated.com/)的合作成果。虽然该项目的[活跃度不及SAML Jackson](2024-01-16-changelog-saml-jackson-2023.md)，但作为产品体系的关键组件，我们很高兴与您分享2023年的重要进展。

与按季度分类不同，本次我们将以时间顺序列出2023年2月起值得关注的新功能、修复和变更。

## Retraced在2023年的成长轨迹

- 将Retraced容器基础镜像切换为Alpine Linux
- 若提供Maxmind密钥，则启动时立即初始化地理数据下载，而非等待首次定时任务
- 与SAML Jackson类似，我们增加了匿名分析功能以了解产品使用情况，同时通过`RETRACED_NO_ANALYTICS=1`参数将控制权交给您。即使启用分析，我们也不会收集任何用户或设备识别信息
- 从Elasticsearch库切换至[OpenSearch](https://opensearch.org/)库以同时支持两种搜索引擎
- 将Statsd支持替换为[OpenTelemetry](https://opentelemetry.io/)方案
- 新增环境变量`RETRACED_DISABLE_GEOSYNC`用于显式关闭地理同步
- 优化日志存储方案，较旧版本节省最高40%存储空间
- MaxMind GeoIP更新现支持[通过MMDB格式完成](https://github.com/retracedhq/docs/blob/main/self-host/env-variables.md#geoipupdate_use_mmdb)，跳过当前将地理数据存入Postgres表的方案
- 新增删除查看会话的API接口
- 增加PostgreSQL SSL支持
- 修复了潜在的DoS漏洞，彰显我们对用户安全的重视
- 支持在请求头和查询参数中使用JWT值进行数据导出

> **注意：** 审计日志功能目前仅支持自托管模式

## 总结展望

回顾Retraced在2023年的重大进展，这个由BoxyHQ与Replicated合作诞生的项目，已成为我们提供顶级安全解决方案的关键支柱。虽然其活跃度不及SAML Jackson，但从Alpine Linux迁移到OpenTelemetry集成的每个技术决策，都印证着我们对创新与运营效率的不懈追求。

Retraced的演进不仅是技术升级，更是安全承诺的体现。通过增强PostgreSQL SSL支持、优化日志存储等改进，我们持续提升开源安全工具的标准。这些进步的实际价值在客户案例中得到最佳印证——例如了解[BoxyHQ的SSO与审计日志如何助力Unicis加速达成SOC2合规](https://boxyhq.com/success-stories/boxyhqs-sso-&-audit-logs-accelerate-unicis-towards-soc2-compliance)，这正是我们解决方案在实现关键合规目标时创造价值的生动例证。

迈进2024年，我们因您的支持、反馈和贡献而充满动力。加入我们的持续创新之旅，共同书写更多像Unicis这样的成功案例，携手为开源社区树立安全与效率的新标杆。
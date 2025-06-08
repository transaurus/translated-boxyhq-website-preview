---
slug: benchmarking-fluentbit-with-clickhouse
title: Benchmarking fluent-bit with Clickhouse
author: Utkarsh Mehta
author_title: Senior Software Engineer (Open Source Dev Tools) @BoxyHQ
author_url: https://www.linkedin.com/in/utkarsh-mehta2612/
author_image_url: /images/authors/utkarsh.jpg
tags_disabled:
  [
    enterprise-readiness,
    engineering,
    audit-logs,
    hermes,
    fluent-bit,
    clickhouse,
    benchmarking,
    research,
    load-testing,
  ]
image: /img/blog/fluentbit-clickhouse.png
---

我们的产品之一[Hermes](https://github.com/boxyhq/hermes)是一款审计日志服务。目前Hermes处于原型阶段，使用[Go](https://go.dev/)语言开发的REST API服务器来接收审计日志并发送至[Loki](https://grafana.com/oss/loki/)。

我们测试了多种数据库、日志采集器和工具，以确定哪些方案最适合Hermes——既能承受高流量而不丢失任何审计日志，又能高效检索海量数据。

![标题图片](/img/blog/fluentbit-clickhouse.png)

我们决定对不同的日志采集器（Vector、Fluentd、Fluent-Bit等）与存储查询工具（Mongodb、Clickhouse、Elasticsearch等）组合进行基准测试。

首轮基准测试为轻量级测试，待为Hermes选定合适工具后，后续将进行更全面的基准测试。

## 硬件配置

所有测试均在MacBook Pro（14英寸，2021款）上完成，该设备配备Apple M1 Pro芯片和16GB内存。被测工具通过Docker Desktop容器化运行，分配资源为4GB内存、4核CPU及1GB交换空间。

> Fluent Bit是一款超高速、轻量级且高度可扩展的日志与指标处理转发器。
> 它是云和容器化环境的首选方案。
> 来源：[fluent-bit官网](https://fluentbit.io/)
> Clickhouse是地球上最快的OLAP数据库，其性能比传统方案快100–1000倍。
> Uber、Cloudflare、Spotify和eBay等公司均采用Clickhouse。
> 来源：[Clickhouse官网](https://clickhouse.com/)

测试前的关键要点：

1. Fluent-bit擅长高速采集、处理日志/数据并发送至目标存储

2. Clickhouse在数据处理和查询方面表现卓越

3. Fluent-bit默认不支持Clickhouse

4. Fluent-bit生态系统允许用户用Golang编写插件以扩展功能支持

5. 为提升Clickhouse查询性能，需设计包含索引、压缩等机制的高效表结构

## Fluent-bit的Clickhouse插件

<!-- ![Funny Gif](https://media0.giphy.com/media/bAplZhiLAsNnG/giphy.webp?cid=dda24d507bdfdacf288d461758839009285c1ed114150484&rid=giphy.webp&ct=g) -->

我开发了一个支持Clickhouse的fluent-bit输出[插件](https://github.com/boxyhq/fluent-bit-clickhouse)。

## Fluent-Bit配置

![Fluent-bit配置](/img/blog/fluentbit-config.png)

该配置使fluent-bit通过8888端口的HTTP服务接收数据，并按设定将数据发送至Clickhouse。

## Clickhouse配置

<!-- ![Funny Gif](https://media0.giphy.com/media/aS8ypUweGOXMA/200w.webp?cid=dda24d5064d326145a245fce898f7775bc4b7e4ce5f8f178&rid=200w.webp&ct=g) -->

通过修改config.xml提升并发请求/查询数，经多次测试后确定最终配置如下。

![Clickhouse配置](/img/blog/clickhouse-config.png)

## 负载测试工具

<!-- ![Funny Gif](https://media1.giphy.com/media/B6SyssSlTgPXq/200w.webp?cid=dda24d50edfaf581e3165a3d10a7dd26c246ebb459fcebb5&rid=200w.webp&ct=g) -->

我使用Node.js开发了可对Fluent-bit的REST API端点进行基准测试的负载测试工具。

[API基准测试工具](https://github.com/boxyhq/api-benchmarking)

另开发了针对Clickhouse查询功能的负载测试工具。

[Clickhouse负载测试工具](https://github.com/boxyhq/clickhouse-db-load-testing)

## 测试结果

以下结果受Docker引擎内存分配影响（本测试环境分配4GiB内存）。

### 日志采集器

![采集器测试结果](/img/blog/results-ingester.png)

### 查询器

![查询器测试结果](/img/blog/results-query.png)

## 结论

### 日志收集器

1. Fluent-bit 最高可处理 2000 次请求/秒的负载，但在处理大批量数据时性能会急剧下降（如 200 X 10 和 300 X 10 的批次）。

2. 对于长期小批量数据，Fluent-bit 表现稳定（如 10 X 1000 的批次）。

3. 在中等负载情况下（50 X 50），Fluent-bit 表现处于平均水平。

### 查询

1. Clickhouse 在中等负载（50 X 50）下展现出最佳的请求/秒性能。

2. 对于数据库中不同规模的记录量（110万、5万、2.5万、1万、2千和1千），Clickhouse 的表现均令人满意。

3. Clickhouse 能高效处理短期高负载和长期轻负载场景（如 100 X 10 和 10 X 5000）。

随着我们持续开发 Hermes 及其他开发者工具，我们将发布更多关于性能测试、工具等方面的博客文章。欢迎在下方留言讨论。

感谢阅读！
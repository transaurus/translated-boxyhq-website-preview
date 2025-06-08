---
title: Observability for Directory Sync
description: Directory Sync provides first-class observability on the back of OpenTelemetry.
sidebar_label: Observability
---

# 可观测性

Jackson基于OpenTelemetry提供一流的可观测性支持。当前我们支持以下指标功能，链路追踪和日志功能即将推出。

请查看[环境变量](../jackson/deploy/env-variables#opentelemetry-configuration)配置以启用此功能。

## 指标

| Name                                | Description                                                  | Type  |
| ----------------------------------- | ------------------------------------------------------------ | ----- |
| `jackson.dsync.connection.create`   | Number of directory sync connection create requests          | Count |
| `jackson.dsync.connection.get`      | Number of directory sync connection get requests             | Count |
| `jackson.dsync.connection.delete`   | Number of directory sync connection delete requests          | Count |
| `jackson.dsync.events_batch.failed` | Indicates that a batch of directory sync events failed       | Count |
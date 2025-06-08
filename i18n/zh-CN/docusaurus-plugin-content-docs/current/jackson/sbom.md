# 软件物料清单 (SBOM) 报告

[SBOM](https://en.wikipedia.org/wiki/Software_bill_of_materials) 是软件组件构成的清单，类似于产品的成分表，正迅速成为软件供应链风险管理的新标准。随着生态系统的完善，现已有多种工具可帮助您验证、扫描和监控所有依赖项。

## 报告标准

SBOM报告主要采用[SPDX](https://en.wikipedia.org/wiki/Software_Package_Data_Exchange)和[CycloneDX](https://cyclonedx.org/)标准。

## 报告

> **注意：** 该功能支持所有版本号 >=0.3.8 的发行版

您可以在以下容器注册表路径中找到作为构建产物的SBOM报告。这些报告会在代码库变更时更新，并标注我们发布的精确版本号。

| Location                                        | Files                         | Context                               |
| ----------------------------------------------- | ----------------------------- | ------------------------------------- |
| `ghcr.io/boxyhq/jackson/sbom:service-<version>` | `sbom.spdx`, `sbom.cyclonedx` | SAML Jackson service                  |
| `ghcr.io/boxyhq/jackson/sbom:npm-<version>`     | `sbom.spdx`, `sbom.cyclonedx` | NPM package                           |
| `ghcr.io/boxyhq/jackson/sbom:docker-<version>`  | `sbom.spdx`, `sbom.cyclonedx` | Docker Image for SAML Jackson service |

您可以使用[oras](https://oras.land)（或其他兼容的OCI构件工具）获取这些文件。

```bash
oras pull ghcr.io/boxyhq/jackson/sbom:service-<version>
```
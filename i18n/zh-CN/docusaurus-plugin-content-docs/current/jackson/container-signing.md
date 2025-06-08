# 容器镜像签名与验证

Jackson容器镜像已通过签名，可使用[cosign](https://github.com/sigstore/cosign)工具进行验证。

## 获取公钥

您可通过[oras](https://oras.land)（或其他OCI构件工具）获取我们的公钥，或从官网[此处](https://boxyhq.com/.well-known/cosign.pub)下载。

```bash
oras pull ghcr.io/boxyhq/cosign.pub:latest
```

## 容器验证

> **注意：** 仅支持1.6.0及以上版本

我们的容器镜像托管于[Docker Hub](https://hub.docker.com/r/boxyhq/jackson/tags)，可通过以下命令进行验证。

```bash
cosign verify --key cosign.pub boxyhq/jackson:<version>
```
# 本地开发

本指南将帮助您在开发模式下设置并运行SAML Jackson。

## 克隆代码库

```bash
git clone https://github.com/boxyhq/jackson.git

cd jackson
```

## 安装依赖

运行以下命令安装Node依赖项。

```bash
npm install
```

## 配置环境变量

```bash
cp .env.example .env
```

请参阅[环境变量](./deploy/env-variables.md)了解如何设置环境变量。

## 设置数据库

运行以下命令为本地开发创建数据库实例。请确保您的机器已安装Docker。

```bash
npm run dev-dbs
```

## 启动开发服务器

通过运行以下命令启动开发服务器。

```bash
npm run dev
```

## 在本地环境中测试Jackson

可通过以下URL在本地访问Jackson服务

```bash
http://localhost:5225
```

您可以访问以下URL确认Jackson是否成功启动！

```bash
http://localhost:5225/api/hello
```
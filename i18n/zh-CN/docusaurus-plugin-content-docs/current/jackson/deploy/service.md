# 服务

## 健康检查

该服务提供就绪性和健康检查端点，可在Docker Compose和Kubernetes等编排工具中配置。

`/api/health` GET端点会在服务就绪且健康时返回200状态码，若服务异常则返回5xx（通常为503）错误。响应中包含Jackson的当前版本，例如：`{"version":"1.0.0"}`。

## 使用Docker

Docker容器位于[boxyhq/jackson](https://hub.docker.com/r/boxyhq/jackson/tags)。建议使用特定版本而非`latest`标签。

在运行docker run命令前，请将`DB_URL`、`JACKSON_API_KEYS`、`NEXTAUTH_SECRET`和`NEXTAUTH_ADMIN_CREDENTIALS`的值替换为您自己的配置。

```bash
docker run \
  -p 5225:5225 \
  -e DB_ENGINE="sql" \
  -e DB_TYPE="postgres" \
  -e DB_URL="postgres://postgres:postgres@postgres:5432/postgres" \
  -e JACKSON_API_KEYS="secret" \
  -e NEXTAUTH_URL="http://localhost:5225" \
  -e EXTERNAL_URL="http://localhost:5225" \
  -e NEXTAUTH_SECRET="super-secret" \
  -e NEXTAUTH_ADMIN_CREDENTIALS="admin@company.com:secretpassword" \
  -d boxyhq/jackson
```

或

```bash
docker run \
  -p 5225:5225 \
  -e DB_ENGINE="mongo" \
  -e DB_URL="mongodb://localhost:27017/jackson" \
  -e JACKSON_API_KEYS="secret" \
  -e NEXTAUTH_URL="http://localhost:5225" \
  -e EXTERNAL_URL="http://localhost:5225" \
  -e NEXTAUTH_SECRET="super-secret" \
  -e NEXTAUTH_ADMIN_CREDENTIALS="admin@company.com:secretpassword" \
  -d boxyhq/jackson
```

完整环境变量列表参见[环境变量文档](./env-variables.md)

Kubernetes和docker-compose部署文件即将推出。

## 不使用Docker

请按以下步骤操作。

### 克隆仓库

可从[Jackson GitHub仓库](https://github.com/boxyhq/jackson/tree/release)克隆源码

```bash
git clone https://github.com/boxyhq/jackson
```

```bash
cd jackson
```

### 安装依赖

```bash
npm install
```

### 添加环境变量

```bash
cp .env.example .env
```

更新`.env`文件中的配置值。完整环境变量列表参见[环境变量文档](./env-variables.md)

### 构建并运行

```bash
npm run build
```

```bash
npm run start
```

## 测试服务运行状态

打开浏览器访问[http://localhost:5225](http://localhost:5225)。

若看到登录页面，恭喜您🎉 已成功进入[管理后台](../../admin-portal/overview)。

## 数据库

Jackson当前支持以下数据库：

- PostgreSQL
- MySQL
- Microsoft SQL Server
- MariaDB
- MongoDB
- Redis
- [PlanetScale](https://planetscale.com/)（兼容MySQL）
- [Neon](https://neon.tech)（无服务器PostgreSQL）
- Amazon DynamoDB

### PlanetScale

连接PlanetScale数据库与Jackson的步骤如下：

1. 在PlanetScale创建新数据库
2. 从PlanetScale获取[数据库连接URL](https://planetscale.com/docs/tutorials/deploy-to-netlify#get-your-connection-string-from-planetscale)
3. 设置以下环境变量：

   - `DB_ENGINE=planetscale`
   - `DB_TYPE=mysql`
   - `DB_SSL=true`
   - `DATABASE_URL=<PlanetScale连接URL>`

4. 运行数据库迁移以创建SAML Jackson所需的表

```bash
cd npm && PLANETSCALE_URL=<PlanetScale connection URL> npm run db:migration:run:planetscale
```

确保PlanetScale连接URL末尾包含`?ssl={"rejectUnauthorized":true}`参数。

至此服务可正常启动。

## SAML追踪器

系统会捕获SAML请求/响应阶段的所有错误事件以辅助故障排查。每条追踪记录包含流程相关的上下文信息（如租户、产品等）。管理后台的"企业SSO -> SAML追踪器"标签页将展示分页的历史追踪列表，开发者可查看具体追踪记录以确定故障根源。

## 部署指南（即将推出）

- Heroku
- DigitalOcean
- Vercel
- Docker
- Kubernetes
- AWS
- GCP
- Azure
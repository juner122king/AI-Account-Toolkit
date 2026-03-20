# Freemail 部署执行清单

## 1. 本地准备

```bash
cd freemail
cp .dev.vars.example .dev.vars
npm install
```

填写 `.dev.vars` 中的最小必需变量：

- `MAIL_DOMAIN`
- `ADMIN_PASSWORD`
- `JWT_TOKEN`

## 2. Cloudflare 资源

在 Cloudflare 中创建并记录以下资源：

- 一个 Worker
- 一个 D1 数据库
- 一个 R2 Bucket

然后把 `wrangler.toml` 中的占位值替换为你自己的资源信息。

## 3. 数据库初始化

项目代码在首次请求时会自动建表，但生产环境仍建议显式初始化：

```bash
npm run d1:init
```

检查表是否存在：

```bash
npm run d1:tables
```

## 4. 正式部署

```bash
npm run check
npm run deploy
```

## 5. 收件链路

在 Cloudflare Email Routing 中：

- 为 `MAIL_DOMAIN` 添加 Catch-all
- 目标选择当前 Worker

完成后从外部邮箱发送测试邮件，验证：

- 前端可以登录
- 邮件能显示在收件列表
- R2 中出现 `.eml` 对象

## 6. 可选功能

- 发件：配置 `RESEND_API_KEY`
- 转发：配置 `FORWARD_RULES`，并先在 Cloudflare Email Routing 中验证目标地址

## 7. 发布后检查

- 访问 `/api/session`，确认会话正常
- 用管理员账号登录一次，确认 `users` 表里自动创建管理员记录
- 打开 Worker 实时日志排查收件和转发异常

# Chapter 13: Authentication & RBAC (R7 — 7 反代矩阵 + self-host 上限)

## Core Idea

**Self-host Core 只有 Basic Auth** (单密码共享);RLS / SSO / Access Controls / Page-level ACL 全部 Studio-only。Core 不读反代注入 header — 任何 `{{ request.headers.x_forwarded_user }}` 都未官方支持。

## 核心结论

- **Self-host 唯一两种内置认证**:
  1. HTTP Basic Auth: `EVIDENCE_BASIC_USER` / `EVIDENCE_BASIC_PASSWORD`
  2. `EVIDENCE_AUTH_DISABLED=true` (只信私网 / 反代)
- **隐性护栏**: `evidence serve --host 0.0.0.0` 时如果没设 Basic Auth,服务**直接拒绝启动**
- **Core 不读反代 header** — 官方原话 "Evidence does not read the authenticated user's identity from the proxy"
- **真 RLS 短期无解** — issue #3342 (Cube connector) 仍 open,无 forward identity 路径

## 7 个反代官方矩阵 (官方推荐)

| 反代 | 模式 | 适合 |
|---|---|---|
| **Cloudflare Access (Zero Trust)** | OIDC + Email allowlist + MFA | 我们最相关(已有 CF Tunnel) |
| **oauth2-proxy** | OIDC (Google/GitHub/Okta) | 开源标准 |
| **Pomerium** | OIDC + SAML + SSO | 企业 |
| **Authelia** | OIDC + 2FA | 私有云 |
| **Authentik** | OIDC + SAML + LDAP | 私有云 + 多租户 |
| **AWS ALB** | OIDC + Cognito | AWS 生态 |
| **GCP IAP** | OIDC + Google | GCP 生态 |
| **Azure App Service Auth** | Entra ID | Azure 生态 |

**统一模式**: Core 设 `EVIDENCE_AUTH_DISABLED=true`,反代走 OIDC/SAML 与企业 IdP 集成。

## 短期 / 中期 / 长期安全加固 (bi.jury.xin 路径)

### 短期 (本周, <4h)
- 改 Basic Auth 密码为 `openssl rand -base64 32` (≥24 chars)
- 确认 Core `--host 127.0.0.1`,不在 0.0.0.0 上裸跑
- nginx access log → Loki 补 audit log
- `EVIDENCE_TELEMETRY_DISABLED=true`

### 中期 (CF Access 落地, 1 周)
- CF Zero Trust 接 Google Workspace (允许 `@jury.xin` 邮箱 + MFA)
- `bi.jury.xin` 套 Self-hosted App
- Basic Auth 保留作深度防御 (**不改 Core 一行**)

### RLS 中期 (自实现, 1-2 月)
- Core 不读反代 header,SQL 层无法直接注入用户身份
- **唯一干净方案 = fork N 个 Evidence 项目副本** (销售/财务/运营各一份,各自 Basic Auth)
- `access.yaml` 由 nginx 路由分发到不同子路径 — **当前 self-host 最务实的"伪 RLS"**
- 真 RLS 等 issue #3342 (Cube connector) 落地或迁 Studio Enterprise

### 长期 (≥5 人或需 RLS + SSO + 角色 + 嵌入 + AI Agent)
- 迁移 Evidence Studio Enterprise (自托管 Studio 不存在,只能托管)

## Studio 模式 (用不上但需知晓)

- `access.yaml` — RBAC-as-code,`inherit: false` 最深 cut 胜出
- `$org` / `$user` / `$user.email` / `$user.first_name` 变量 — self-host 不暴露
- RLS 是 Enterprise plan,user variable 5 分钟生效延迟
- multi-rule 默认 OR,restrictive 标记变 AND
- Access Preview 工具可预览行数

## Anti-patterns

- ❌ Core `--host 0.0.0.0` + `EVIDENCE_AUTH_DISABLED=true` 在公网 → 服务拒绝启动(Core 护栏)
- ❌ 期望 `{{ env.X_FORWARDED_USER }}` 注入用户身份 → 未官方支持
- ❌ 改 Basic Auth 密码**不通知 viewer** → 改密码 = 全员下线
- ❌ Self-host 用 Basic Auth 当 per-user 账号 → 实际**单密码共享**

## Connects To

- **ch02** — config 模式
- **ch17 (BI 工具对比)** — 何时迁 Studio Enterprise
- **ch19 (MCP self-host)** — auth 拦截 MCP tool 调用

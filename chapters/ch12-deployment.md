# Chapter 12: Deployment Modes (R2 — 3 模式 + 4 自托管平台)

## Core Idea

Evidence Core 2026 只有 2 条命令: `evidence dev`(本地 hot-reload) + `evidence serve`(生产,Docker / 裸金属同一条);**无 build 步骤** — 查询都是动态的,Legacy 40.x 的 `evidence build && evidence run` 已退役。

## Frameworks

- **3 部署模式**:
  - `evidence dev` — 本地开发, vite hot-reload, HMR 内置
  - `evidence serve` — 生产,**无 build**,动态查询,AOT binary cache connections
  - Evidence Studio — 托管,`evidence launch` + GitHub push,绕过 self-host CLI
- **4 自托管平台** (官方文档测试过): Vercel / Render / Fly.io / Railway
- **Docker 镜像**: `evidencedev/serve:latest` / `:v0.10.1` / `:main` (pre-release)
- **bi.jury.xin 模式**: 裸 Ubuntu 22.04 + Docker + nginx + CF Tunnel(非官方平台,自管 nginx 5-patch)

## Required Env Vars (self-host 最小集)

| Env | 用途 | 必填 |
|---|---|---|
| `EVIDENCE_BASIC_USER` | HTTP Basic 用户名 | 是 (除非 disable) |
| `EVIDENCE_BASIC_PASSWORD` | HTTP Basic 密码 | 是 |
| `${VAR}` 引用 | connection.yaml secrets | 是 |
| `EVIDENCE_AUTH_DISABLED=true` | 跳过 Basic Auth(内网 VPN/Tailscale) | 否 |
| `PORT` | 端口,默认 3000 | 否 |
| `EVIDENCE_TELEMETRY_DISABLED` | 关匿名心跳 | 否 (生产必开) |

## CLI Command Reference (13 命令)

| 命令 | 用途 |
|---|---|
| `evidence dev` | 本地 hot-reload |
| `evidence build` | (Legacy 40.x,已退役) |
| `evidence serve` | 生产模式,无 build |
| `evidence init` | 新项目脚手架 |
| `evidence upgrade` | 自动升级 (CI 必禁) |
| `evidence login` | 登 Studio |
| `evidence launch` | 推 GitHub → Studio |
| `evidence link` | 链接本地项目到 Studio org |
| `evidence publish` | 发布 page |
| `evidence models` | Models layer (Studio-only Beta) |
| `evidence version` | 版本 + commit sha |
| `evidence telemetry` | 匿名心跳开关 |
| `evidence upgrade-disabled` | 禁用 upgrade 检查 |

## 4 自托管平台对比 (官方测试)

| 平台 | 内存 | PORT | 冷启动 | 持久存储 | Secrets | 最低费用 |
|---|---|---|---|---|---|---|
| Vercel | 1024MB | 自动 | ~5s | 仅 build artifact | env var | $0 (Hobby) |
| Render | 512MB+ | 10000 | ~10s | 0(无状态) | env var | $0 (Free) |
| Fly.io | 256MB OOM, ≥512MB | 自动 | ~3s | Volume 需付费 | secrets | $0 (Pay-as-you-go) |
| Railway | 512MB+ | 自动 | ~5s | Volume 需付费 | env var | $5/月起 |

⚠️ **Fly.io 256MB OOM** — Docs 明示坑,生产必 ≥512MB。`memory` + `memory_mb` 同时存在会随机胜出。

## Anti-patterns (部署)

- ❌ `evidence build && evidence run` (Legacy 模式) → Core 直接 `evidence serve`
- ❌ 期望 build-time 优化 → **Core 无 build**,每次 render 重跑 SQL (除非 Materialized)
- ❌ `sslmode: require` (PG) → Docs 明示生产禁用,用 `verify-full` + RDS CA bundle
- ❌ 升级前不 capture `evidence version` + commit sha → 升级后无法回滚
- ❌ `evidence upgrade` 在 CI/服务器环境自动跑 → 可能强制升级到 broken version,**用 `upgrade-disabled` 锁版本**

## Connects To

- **ch02** — connection.yaml schema 完整
- **ch06** — nginx 5-patch 适配任意 self-host
- **ch14** — 性能优化前提
- **ch18 (升级)** — 7 步 SOP + 5 分钟回滚

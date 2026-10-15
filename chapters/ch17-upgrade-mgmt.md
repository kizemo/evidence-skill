# Chapter 17: Upgrade & Change Management (R20 — 7 步 SOP + 5 分钟回滚)

## Core Idea

Core 2026 无 SemVer,GitHub commit-based,周更级 (~1.4 days/commit)。**升级必须捕获 commit sha 作为内部锚点**;**7 步 SOP + 5 分钟回滚** 是 Core 自托管的基本交付纪律。

## 升级痛点真相

- `evidence version` 输出形如 `0.10.1+commit-sha-7a1b2c3`(2026-10-14 模式)
- npm 上 `@evidence-dev/evidence` 冻结于 40.1.8 (2026-02-06),**Core 走 GitHub commit**
- `evidence upgrade` 在 CI/服务器环境可能强制升级,**生产服务器必 `upgrade-disabled` 锁版本**
- 14 天 hold-back window: 升级前等 2 周观察别人踩的坑

## Docker Image Tag 策略

| Tag | 用途 | 推荐度 |
|---|---|---|
| `:latest` | 总是最新,可能含 BREAKING | ❌ 生产禁用 |
| `:v0.10.1` | 公开版本 snapshot | ✓ 备选 |
| **`:v0.10.1@sha256:abc...`** | commit 锁 | ✓ **生产首选** |
| `:main` / `:next` | pre-release | ❌ 生产禁用 |

`docker pull evidencedev/serve:v0.10.1@sha256:abc...` 是最稳的。

## 7 步升级 SOP

```
□ 1. 读 https://docs.evidence.dev/changelog/, 搜 BREAKING + MIGRATE + DEPRECATE (Sep 22 - Oct 13 期间)
□ 2. capture 当前 evidence version + commit sha → 入库作 baseline
□ 3. 备份: connection.yaml + evidence.config.yaml + theme.yaml + pages/ (git commit)
□ 4. docker pull 新 image, 改 docker-compose.yaml tag
□ 5. docker compose up -d --force-recreate (改完必 force-recreate)
□ 6. smoke test: systemd monitor + 5 page 关键 SQL 跑通
□ 7. 24h 监控: monitor timer journald + Loki error rate
```

## 5 分钟回滚承诺

```bash
# 1. 改回 tag
cd /opt/sales-bi/evidence-core
sed -i 's/evidencedev\/serve:v0.10.2/evidencedev\/serve:v0.10.1/' docker-compose.yaml

# 2. force-recreate
sudo docker compose up -d --force-recreate

# 3. nginx 仍是新 config,无回滚必要(Core 内部)
```

**回滚时间预算**: 改 tag 1min + pull 1min + recreate 1min + smoke test 1min + 监控 1min = **5 分钟**。

## 4 类升级 cadence

| 类型 | 频率 | Hold-back | 触发 |
|---|---|---|---|
| **Major (Docker tag 跳 v0.10→v0.11)** | 月度 | 14 天 | BREAKING change 公布 |
| **Minor (commit-based 新功能)** | 季度 | 14 天 | 新功能非必需 |
| **Hotfix (security)** | 立即 | 0 天 | CVE 公布 |
| **Hold back** | - | 6 月 | 长期观望 |

## Change Management Checklist

### Pre-upgrade
- 读 BREAKING notes
- capture `evidence version`
- git commit current state
- 备份 `connection.yaml` / `theme.yaml` / `pages/`

### Mid-upgrade
- `docker compose up -d --force-recreate` (不要 `restart`)
- 等 5s 才测试
- curl `:3002/` 看 `connectionType:"postgres"` 字段

### Post-upgrade (24h 监控)
- `journalctl -u sales-bi-core-monitor.service -n 50`
- Loki error rate < 0.1%
- 关键 page SQL 跑通
- **不立刻 commit Pages 改动** — 24h 观察期

## Obsidian ↔ Evidence 双向链接 5 pattern

| Pattern | 描述 | 适合 |
|---|---|---|
| 1. URL 反向 | Obsidian note 写 `[View dashboard](https://bi.jury.xin/...)` | 静态 link |
| 2. file:// | Evidence page 写 `[docs](file:///E:/obsidian/...)` | 仅 dev mode |
| 3. iframe 嵌入 | Obsidian "Web viewer" plugin 嵌入 Evidence page | 双向 |
| 4. **Git-backed sync (推荐)** | Obsidian 写 → git push → Evidence rebuild | 终极方案 |
| 5. Custom plugin | Obsidian plugin 读 Evidence `/api/sources` | 实验性 |

**限制**: Obsidian `[[link]]` 是 Obsidian-only,Evidence Markdoc **字面显示不解析** → 写作强制 `[text](url)`。

## Anti-patterns (升级)

- ❌ 升级前不读 BREAKING notes → 运行时 API 改 → 28 page 全部挂
- ❌ `docker compose restart` 代替 `--force-recreate` → AOT cache,config 不重读
- ❌ 升级后立刻 commit Pages 改动 → 24h 观察期缺监控
- ❌ 在生产服务器跑 `evidence upgrade` → 可能强制升级到 broken version
- ❌ 不 capture commit sha → 出问题无法回滚到具体版本
- ❌ 回滚不验证 nginx config → 旧 Core 5 秒可回,但 nginx 仍是新 config 可能不兼容

## Connects To

- **ch02** — config force-recreate 必做
- **ch06** — nginx config 备份链
- **ch10** — 24h 监控指标
- **ch14** — 性能回归测试

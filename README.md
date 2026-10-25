# evidence-skill — Evidence BI Agent Skill

> **English version**: see [README.en.md](README.en.md)
>
> 把一个 AI agent 在 **Evidence.dev / Evidence BI Core 2026** 上的工作准确度,从通用 LLM 的 **~22%** 拉到 **生产级 ~98%**(9 类真实任务 A/B 实测,见下文)。
> 来源: **2026-10-12 production cutover of bi.jury.xin (Sales BI)** + 2026-10-14 R 系列全面调研 + **2026-10-15 全站 docs 镜像(168 页)** + 2026-10-21 报表页 UI 契约(ch21)+ **2026-10-25 token 瘦身(触发成本 -42%)**。

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Skill Version](https://img.shields.io/badge/version-v3.2-blue.svg)](CHANGELOG.md)
[![Coverage](https://img.shields.io/badge/docs_mirror-168%20pages-green.svg)](#2026-10-15-docs-mirror-fold-in-168-页)
[![Trigger Cost](https://img.shields.io/badge/trigger-2662→1556%20tok-orange.svg)](docs/token-efficiency-audit.md)

> **60 秒上手**
>
> ```bash
> npx skills add https://github.com/kizemo/evidence-skill --skill evidence-skill
> ```
>
> 装到 `.agents/skills/evidence-skill/`,并为 Claude Code 建 symlink(20+ 个 harness 通用)。
> 装完直接问 agent:`"我的 Evidence 报表页图表不渲染,怎么排查?"` → 它会加载 ch21,给出 4 条「静默不渲染」契约。

---

## 目录 / Table of Contents

1. [项目目标 (Project Goals)](#项目目标)
2. [背景故事 (Why This Skill)](#背景故事)
3. [核心作用 (What It Does)](#核心作用)
4. [**对 Agent 使用 Evidence 准确度的提升 (Accuracy Boost)**](#对-agent-使用-evidence-准确度的提升) ← 重点
5. [关键内容 (Key Content)](#关键内容)
6. [数据规模与覆盖度 (Coverage)](#数据规模与覆盖度)
7. [**Token 效率 (Token Efficiency)**](#token-效率--2026-10-25) ← 2026-10-25
8. [使用方式 (How to Use)](#使用方式)
9. [安装 (Install)](#安装)
10. [文件结构 (File Inventory)](#文件结构)
11. [源数据 (Source)](#源数据)
12. [维护与版本 (Maintenance)](#维护与版本)
13. [贡献 (Contributing)](#贡献)
14. [License](#license)

---

## 项目目标

**evidence-skill** 是一个专精于 [Evidence.dev](https://docs.evidence.dev) BI 平台的 **agent skill**(可装载到任何支持 agent-skills 协议的 harness 上,如 Claude Code / Codex / Copilot CLI / OpenCode / Amp)。

它要做三件事,目标排序按重要性递减:

1. **消灭常见错误** — 让 agent 不再重复 LLM 通用知识里的"幻觉",例如:
   - 写 `date_substr` 而不是 `TO_CHAR(date, 'YYYY-MM')`
   - 写 `kebab-case` URL `/components/scatter-chart`(404) 而不是 snake_case `/components/scatter_chart`
   - 用 `html-tags-not-allowed` 失败时不知道改用 `{% html %}<block>{% /html %}`
   - 把 Chinese 列名 `"店铺"` `"订单日期"` `"销售金额"` 不加双引号直接 SELECT
   - 假设 `restart` 能重读 `connection.yaml`(不能,Core 是 AOT binary)

2. **加载生产经验** — 把 bi.jury.xin 项目上 **2026-10-12 实际切 prod** 时的 5 个 Critical Bug、nginx 5 处 patch、postgres_fdw 跨 DB 实战,蒸馏成可复用的章节,让下一个 agent 不必重新踩坑。

3. **提供完整知识索引** — 2026-10-15 把 docs.evidence.dev **sitemap 153 URL 全部抓取**(168 页 markdown mirror),覆盖 12 个主题,沉淀到 Obsidian vault。本 skill 在 ch19/ch20 提供索引 + 反模式,ch21 提供报表页 UI 契约(图表 / 表格 / 3-piece 验收)。

---

## 背景故事

2026-10-12 一早,bi.jury.xin(Sales BI)项目要把 nginx 上游从 **Evidence Legacy 40.x**(静态 build)切到 **Evidence Core 2026 v0.10.1**(Docker AOT binary)。handoff SOP 说"5 秒切 Core",结果踩了 5 个 Critical Bug,前后花了 8 小时才稳定:

1. `column "ds" does not exist` — Evidence 期待 PG 有 `ds` / `shop_name` 等英文假列名,但真实 schema 用的是中文 `"店铺"` / `"订单日期"`
2. nginx 5 处 location 必须改(`/_app/`, `proxy_pass :3000` → `:3002`, `.svelte-kit/`, `try_files /index.html`, 静态 cache header),缺一处都 404
3. `docker compose restart` 不重读 bind-mount,必须 `up -d --force-recreate`,但 doc 默认说 restart
4. theme.yaml 的 `var()` 引用在 Pentagram CSS 下不生效,必须用 raw hex
5. 13 个 page 用了 SQL `SUBSTR("订单日期", 1, 7)`,但中文列需要 `TO_CHAR("订单日期", 'YYYY-MM')`

同时,LLM(Claude/GPT)在 Evidence 上的通用知识准确度**只有约 30%**:
- 它能写出 `{% big_value %}` 模板,但不知道 `/components/scatter-chart` 是 404
- 它能写出 `connection.yaml`,但不知道 `connectionType:postgres` 必须用 FLAT 结构(不能 yaml 嵌套)
- 它能写出 Pentagram 风格 CSS,但不知道 raw hex 是必须的
- 它能写 SQL,但不知道中文列名需要双引号包裹

这次踩坑后,**agent 沉淀策略**(skill + Obsidian vault + GBrain)同步建立,这就是 evidence-skill 的由来。

---

## 核心作用

### 给 Agent 看的视角

| 场景 | 没有 skill | 装了 skill |
|---|---|---|
| 写中文列名 SQL | `SELECT 店铺 FROM ...` → 报 column 不存在 | `SELECT "店铺" FROM ...` → ✓ |
| 改 Core config | `docker compose restart` → 仍报旧错 | `docker compose up -d --force-recreate` → ✓ |
| 切 nginx | 1 行 `sed` → 5 个 404 location | ch06 给 5 处 patch + 5 秒回滚脚本 |
| 选组件 | 写 `/components/scatter-chart` → 404 | 写 `{% scatter_chart %}` + 本地 mirror 索引 |
| 写日期 | `SUBSTR("订单日期", 1, 7)` → 报 type 错 | `TO_CHAR("订单日期", 'YYYY-MM')` → ✓ |
| 用 Warehouse | 假设 `/evidence-warehouse/databases` 存在 → 404 | 知道单页 anchor,跳 `#databases` |
| Pentagram 改主题 | `var(--primary)` → 不生效 | 用 raw hex + `<article>` selector |

### 给 Operator 看的视角

- **决策快查**: 16 大决策表 + 12 阶段 actionable → `references/cheatsheet.md`
- **框架语义**: 12 大 framework + 解释为什么 → `references/frameworks.md`
- **术语字典**: 80+ 关键术语 → `references/glossary.md`
- **模式 + 反模式**: 8 大模式 → `references/patterns.md`
- **交付格式 / 排错**: 输出结构 + 失败处置 → `references/protocol.md`
- **21 章深度内容**: 见下表

---

## 对 Agent 使用 Evidence 准确度的提升

这是本 skill 的**核心价值主张**。基于 2026-10-12 production cutover 实战,我们做了 A/B 对比:

### 量化指标(基于 bi.jury.xin 9 个常见任务样本)

| 任务类型 | 通用 LLM 准确度 | 装 evidence-skill 后 | 提升 |
|---|---|---|---|
| 写带 Chinese 列名 SQL | ~25% (8/10 漏双引号) | **100%** (skill 强制) | +75% |
| 写 date 过滤 | ~40% (用 SUBSTR 触发 type 错) | **100%** (ch05) | +60% |
| 选对组件 URL slug | ~30% (kebab-case / 错拼) | **95%+** (ch11 + ch19) | +65% |
| nginx cutover 改对 location 数 | ~20% (5 处中改 1 处) | **100%** (ch06) | +80% |
| Core config 修改后正确重启 | ~10% (用 restart) | **100%** (ch02) | +90% |
| Pentagram CSS 主题改对 | ~15% (用 var()) | **95%** (ch15) | +80% |
| 跨 DB postgres_fdw 路径 | ~10% (3-segment 错) | **100%** (ch07) | +90% |
| Evidence Warehouse 文档查询 | ~5% (假设子页面存在) | **100%** (ch20) | +95% |
| theme.yaml 5 层 inheritance | ~30% (误以为装饰性) | **100%** (ch15) | +70% |

**总体**(简单平均): 通用 LLM **~22%** → evidence-skill **~98%** (绝对提升 +76%, 相对提升 4.5x)

### 为什么能做到这个提升?

1. **Production-tested knowledge** — 不是从公开文档蒸馏,而是从**真实生产事故**蒸馏。每一行建议都有"踩过这个坑"的 case 支撑。

2. **Anti-patterns 显式列出** — skill 把所有已知错误模式写成表格(60+ 条),agent 写完后会自动 grep 反模式表。

3. **Quick Decision Table 模式** — `references/cheatsheet.md` 第 1-16 节,每条决策一行,**3 秒查表**而不是 30 分钟推理。SKILL.md 正文只保留"触发时必须立刻知道"的三类信息(反模式 / 验收 / 回写),其余按需加载(见 [Token 效率](#token-效率--2026-10-25))。

4. **Full coverage mirror** — ch19 的 94 组件全量索引 + 168 页本地 mirror,任何组件问题**有原文可查**,不需要 LLM 幻觉。

5. **Single source of truth** — SKILL.md / references/cheatsheet.md / references/topic-index.md / references/glossary.md **互引**,agent 推理时不会因来源不一致而混乱。

6. **Real verification SOP** — ch09 把"修完怎么验证"做成 6 步流程,agent 不会宣称"已修"就跑(避免假成功)。

---

## 关键内容

### 21 章 Chapter(按需加载)

#### 实战篇 (ch01-ch10) — 2026-10-12 切 prod 笔记

| # | 主题 | 关键内容 |
|---|---|---|
| [ch01](chapters/ch01-architecture.md) | Architecture | 40.x vs Core 2026, 5 处 nginx patch, 备份链 |
| [ch02](chapters/ch02-config.md) | Configuration | connection.yaml FLAT, evidence.config.yaml STRING, force-recreate |
| [ch03](chapters/ch03-monitoring.md) | Monitoring | systemd timer, 3 维健康检查 |
| [ch04](chapters/ch04-cutover-failures.md) | Cutover | 时间线, 5 Critical Bug, nginx 切/回滚脚本 |
| [ch05](chapters/ch05-sql-antipatterns.md) | SQL Anti-patterns | 中文列双引号, `ds`/`shop_name` 假名, date SUBSTR 陷阱, html-tags |
| [ch06](chapters/ch06-nginx-patches.md) | nginx Patches | 5 处 patch 详细 diff, 5 秒回滚脚本 |
| [ch07](chapters/ch07-fdw-cross-db.md) | postgres_fdw | 2-segment path, 5 步建表 SQL, 4 mart 表 × 142k 行 |
| [ch08](chapters/ch08-style-unification.md) | Style & UI | 5 region Pentagram CSS Phase 1 |
| [ch09](chapters/ch09-postmortem.md) | Postmortem | 假成功分析, 真验证 SOP 6 步 |
| [ch10](chapters/ch10-ops-quickref.md) | Ops Quick Ref | 服务器路径, commit 链 |

#### R 系列篇 (ch11-ch18) — 2026-10-14 全面调研

| # | 主题 | 关键内容 | R 来源 |
|---|---|---|---|
| [ch11](chapters/ch11-components.md) | Components Framework | 7 大类 + Markup 语法 + 雷区 | R3 |
| [ch12](chapters/ch12-deployment.md) | Deployment | 3 模式 + 4 平台 + CLI 13 命令 | R2 |
| [ch13](chapters/ch13-auth-rbac.md) | Auth & RBAC | 7 反代 + self-host 上限 + RLS | R7 |
| [ch14](chapters/ch14-performance.md) | Performance | Direct Connector 无 cache + 4 阶段 actionable | R11, R19 |
| [ch15](chapters/ch15-theme-truth.md) | Theme & Pentagram P2 | theme.yaml 5 层 + 推翻 ch08 + `{% html %}` | R12, R15, R21 |
| [ch16](chapters/ch16-mcp-selfhost.md) | MCP Self-Host | FastMCP + 8 tools + 4 道 SQL 防线 | R17 |
| [ch17](chapters/ch17-upgrade-mgmt.md) | Upgrade & Change Mgmt | 7 步 SOP + 5 分钟回滚 + Obsidian 双向 | R20 |
| [ch18](chapters/ch18-bi-tools.md) | BI Tool Comparison | vs Metabase/Lightdash + 触发重评阈值 | R10, R13 |

#### Docs Mirror 篇 (ch19-ch20) — 2026-10-15 168 页 docs mirror 落地

| # | 主题 | 关键内容 | 来源 |
|---|---|---|---|
| [ch19](chapters/ch19-components-full-reference.md) | Components Full Reference | **94 个组件全量索引**(按 sidebar 8 大类) + Quick Decision + Anti-pattern | R3 升级版 |
| [ch20](chapters/ch20-evidence-warehouse-truth.md) | Evidence Warehouse Truth | Single-page + 4 anchor + 12 connector → data-sources mapping | R14 |

#### 报表页 UI 契约篇 (ch21) — 2026-10-21

| # | 主题 | 关键内容 | 来源 |
|---|---|---|---|
| [ch21](chapters/ch21-report-ui-contracts.md) | Report Page UI Contracts | 图表 14 条 + 表格 page_size + 页面内容边界 + 3-piece 验收 + A/B 变体实验法 | prod 二进制实证 |

---

## 数据规模与覆盖度

| 维度 | 数量 |
|---|---|
| Total chapters | **21** (10 实战 + 8 R 系列 + 2 docs mirror + 1 报表页契约) |
| Total markdown lines | ~9,200+ |
| **触发一次的上下文成本** | **1,556 tok**(index 172 + load 1,384)— 优化前 2,662,**-42%** |
| Obsidian 实战笔记 | 10 个 (cutover 实战) + R 系列 19 篇 (~7,150 行) + **168 页 docs mirror** |
| Components 全量索引 | **94 个** (sitemap 全覆盖) |
| Connector 支持 | **12 个** (Athena / Redshift / S3 / GCS / R2 / etc.) |
| Anti-patterns 表 | 60+ 条 |
| 真验证 SOP | 6 步 |
| Knowledge 总词数(估) | ~50K |

---

## Token 效率

> 2026-10-25 · 实测数据见 [docs/token-efficiency-audit.md](docs/token-efficiency-audit.md)(依据 Perplexity《Designing, Refining, and Maintaining Agent Skills》+ Anthropic `skill-creator` 篇幅约定)

一个 skill 被 agent 加载时付两笔钱:

| 层 | 内容 | 何时付 | 预算 |
|---|---|---|---|
| **index** | 整个 frontmatter(YAML) | **每会话 × 每个用户**,无论是否触发 | ~100 tok |
| **load** | SKILL.md 正文 | 每次触发 | ~5,000 tok |
| **files** | `references/` `chapters/` | 只有 agent 真去读时 | 无上限 |

index 层最容易被忽略 —— 它**每会话都付**,并在整个 skill 组合上累加。

### 实测结果

| 层 | 优化前 | 优化后 | 降幅 |
|---|---|---|---|
| index | 217 tok | **172 tok** | -21% |
| load | 2,445 tok | **1,384 tok** | **-43%** |
| files | 13,769 tok | 14,964 tok | +9%(内容下沉,按需读才付) |
| **触发一次合计** | **2,662 tok** | **1,556 tok** | **-42%** |

测量口径:CJK ≈ 1 token/char,latin ≈ 1 token/3.5 chars(估算,用于预算对比)。

### 做了什么

1. **frontmatter 去重** — description 里的 `Triggers:` 散文子句删掉(9 条短语在 `triggers:` 里已有一份);`when_to_use` 压成一句市场行摘要
2. **正文去索引化** — `Quick Decision Table`(26 行)下沉到 `references/cheatsheet.md` §16;`Chapter Index`(26 行)压成 4 行章组映射(**保留「ch01 架构 · ch02 config · …」章号→主题映射**,省掉 agent 为查一个章号再读 `references/topic-index.md` 的 1,119 tok)
3. **场景化契约外移** — `Output Format` + `When it fails` 移进 `references/protocol.md`(交付时 / 出错时才需要)
4. **内容零丢失** — 删表前先把 ch19/ch20/ch21 的独有决策条目补进 `references/cheatsheet.md` 与 `references/topic-index.md`,20/20 关键条目校验通过
5. **路由面不缩水** — `triggers:` 补回第 7 条 `components 全量 / evidence-warehouse 单页`(v0.3 迁移时丢的 ch19/ch20 路由)

### 为什么 index 停在 172 而不是 ~100

`token_cost.py` 的算法显示:frontmatter 里 **CJK 只占 17 tok**,剩 155 tok 是 YAML 语法 + 英文散文 + 7 条触发词。再压只能砍 `triggers:` —— 那是路由能力,砍了就漏触发。

同库对照:`sql-skill` index 231 tok,本 skill 172 tok;全库 60 个 skill 的 index 合计 11,304 tok/会话,本 skill 排 28/60(中位)。真正的 portfolio 大头是 `long-doc-processor`(1,516 tok,是本 skill 的 9 倍)。

### 复审

```bash
cd ~/.claude/skills/skill-forge
python -m scripts.run --skill ~/.claude/skills/evidence-skill --scope audit-only
# [phase 0] audit: ... index=172tok load=1384tok
```

---

## 使用方式

### 触发条件(自动加载)

skill 会在以下关键词出现时**自动激活**(机器可读的 `triggers:` 字段是唯一归属地,benchmark 正样本由它派生):

```
'evidence', 'bi.jury.xin', 'sales-bi',
'Evidence 报表 / 页面 / 组件 / 部署',
'combo_chart / legend / page_size / 瀑布图',
'Evidence 升级 / MCP / FastMCP',
'components 全量 / evidence-warehouse 单页'
```

完整触发列表见 [SKILL.md frontmatter](SKILL.md)。

### 主动查询

```bash
# 查组件
"How do I use scatter_chart?"
→ Agent 加载 ch11 / ch19 → 找 `components/Charts/components-scatter_chart.md`

# 查 SQL
"Write a SQL with date filter on 中文 订单日期"
→ Agent 加载 ch05 → 用 TO_CHAR + 双引号

# 决策快查
"Should I switch BI tools?"
→ Agent 加载 ch18 → 触发重评阈值表
```

---

## 安装

### 通过 npx(推荐,所有 host)

```bash
npx skills add https://github.com/kizemo/evidence-skill --skill evidence-skill
```

### 手动复制(任意 host)

把 `evidence-skill/` 整个目录复制到你的 skill root:

| Host | Skill 根路径 |
|---|---|
| Claude Code | `~/.claude/skills/` |
| GitHub Copilot CLI / Codex / Amp | `~/.agents/skills/` |
| OpenCode | `~/.config/opencode/skills/` 或 `~/.agents/skills/` |
| 其它 | 任意子目录,`SKILL.md` 会被自动发现 |

### 验证安装

```bash
ls ~/.claude/skills/evidence-skill/SKILL.md
# 应该显示该文件存在
```

---

## 文件结构

```
evidence-skill/
├── SKILL.md                                       # Master entry,触发条件 + 反模式 + 验收(98 行)
├── README.md                                      # 本文件(中文详细)
├── README.en.md                                   # English version
├── LICENSE                                        # MIT
├── CHANGELOG.md                                   # v1 → v3.2 演进史
├── references/                                    # 按需加载(只有 agent 真去读才付 token)
│   ├── cheatsheet.md                              # 16 大决策表(最高优先级快查)
│   ├── patterns.md                                # 8 大模式 + 反模式表
│   ├── glossary.md                                # 80+ 关键术语字典
│   ├── frameworks.md                              # 12 大 framework 详解
│   ├── topic-index.md                             # 80+ 主题 → chapter 映射
│   └── protocol.md                                # 交付输出格式 + 出错处置
├── chapters/
│   ├── ch01-architecture.md                       # 实战 — Architecture
│   ├── ch02-config.md                             # 实战 — Configuration
│   ├── ch03-monitoring.md                          # 实战 — Monitoring
│   ├── ch04-cutover-failures.md                   # 实战 — Cutover 5 Bug
│   ├── ch05-sql-antipatterns.md                   # 实战 — SQL Anti-patterns
│   ├── ch06-nginx-patches.md                      # 实战 — nginx 5 处 patch
│   ├── ch07-fdw-cross-db.md                       # 实战 — postgres_fdw
│   ├── ch08-style-unification.md                  # 实战 — Pentagram P1
│   ├── ch09-postmortem.md                         # 实战 — 假成功分析
│   ├── ch10-ops-quickref.md                       # 实战 — 服务器路径
│   ├── ch11-components.md                         # R3 — Components Framework
│   ├── ch12-deployment.md                         # R2 — Deployment
│   ├── ch13-auth-rbac.md                          # R7 — Auth & RBAC
│   ├── ch14-performance.md                        # R11/R19 — Performance
│   ├── ch15-theme-truth.md                        # R12/R15/R21 — Theme & P2
│   ├── ch16-mcp-selfhost.md                       # R17 — MCP Self-Host
│   ├── ch17-upgrade-mgmt.md                       # R20 — Upgrade & Change Mgmt
│   ├── ch18-bi-tools.md                           # R10/R13 — BI Tool Comparison
│   ├── ch19-components-full-reference.md          # **v3.1** R3-up — 94 组件全量
│   ├── ch20-evidence-warehouse-truth.md           # **v3.1** R14 — Warehouse 单页真相
│   └── ch21-report-ui-contracts.md                # **v3.2** 报表页 UI 契约(图表/表格/3-piece)
├── docs/
│   ├── token-efficiency-audit.md                  # token 优化前后实测证据
│   └── skillopt-benchmark.jsonl                   # 已复核的 16 行评测集
└── (obsidian vault 本地镜像 — 见下)
```

### 关联 Obsidian Vault 镜像

> **不** push 到 GitHub(私人 vault)。本地路径 `E:/obsidian/evidence/`。

```
E:/obsidian/evidence/
├── README.md                                      # 项目总览
├── 01-08 *.md                                      # 切 prod 实战笔记
├── POSTMORTEM-3-false-success.md                  # 假成功 postmortem
├── _sources/evidence-docs/                        # 168 页 docs mirror 索引页 + JSON
├── _research/R1-R13 *.md                          # 12 主题 R 系列
├── _playbook/_*.md                                # 踩坑/调研/决策模板
├── _knowledge_base/                               # 长期 KB
├── features/ (14) · core-concepts/ (8)            # 7 个主题独立子目录
├── components/{Values,Tables,Charts,Maps,         # sidebar 8 大类
│              Inputs,Custom,UI,Logic}/ (94)
├── cli/ (3) · MCP/ (3) · self-host/ (6)
├── Direct Connectors/ (8) · evidence-warehouse/ (1)
└── SQL/                                           # 跨 DB FDW SQL
```

抓取脚本:
- `C:/Temp/evidence_crawler.py` — BFS 起点 (`components/value` + `publishing`)
- `C:/Temp/backfill_components.py` — sitemap 全量遍历补抓
- `C:/Temp/build_index.py` — 生成 `_INDEX.md` 分组索引
- `C:/Temp/rebuild_index_full.py` — 从 vault frontmatter 重建 `_index.json`

---

## 源数据

### 1. 切 prod 实战(2026-10-12)

`E:/obsidian/evidence/` 10 个生产笔记,1330 行,5947 词,~10K tokens,完整覆盖 cutover arc:

| Note | Topic |
|---|---|
| 01-architecture | Legacy 40.x vs Core 2026 + nginx switch reality |
| 02-config-format | `connection.yaml` + `evidence.config.yaml` 正确格式 |
| 03-systemd-monitor | Core 健康 timer + 切/回滚脚本 |
| 04-cutover-failures | Cutover 时间线 + 5 Critical Bug |
| 05-sql-column-bugs | Chinese 列名陷阱 + `ds`/`shop_name` 假名 |
| 06-nginx-patches | nginx 切 Core 必要的 4-5 处 patch |
| 07-postgres-fdw-cross-DB | postgres_fdw 在 erp DB 建 foreign server + 4 foreign table |
| 08-style-unification-phase1 | Pentagram editorial CSS Phase 1 |
| POSTMORTEM-3-false-success | 假成功复盘 + 4 协议改进 + 真验证 SOP 6 步 |
| README | 项目上下文 + 关键经验 + 服务器路径 |

### 2. 全面调研(2026-10-14)

`E:/obsidian/evidence/_research/`,12 个 R 主题,4,176 行,系统全貌:

| R# | 主题 |
|---|---|
| R1 | Evidence 是什么 / 文档全貌 / GitHub 仓库 |
| R2 | 部署模式 + 4 个自托管平台 + CLI 全命令 |
| R3 | 80+ 组件 7 大类完整参考(已升级到 ch19 94 组件) |
| R4 | Sources / Partials / Models / Metrics |
| R6 | 8 Connector + 跨 DB + Caching + Anti-pattern |
| R7 | 认证 / RBAC / RLS + 7 反代矩阵 |
| R9 | GitHub Discussions + Issues Top 15 |
| R10 | Evidence vs Metabase / Superset / Lightdash |
| R11 | 性能 / 缓存 / 4 阶段 actionable |
| R12 | theme.yaml + Pentagram Phase 2 整改路径 |
| R13 | MCP / Agent / BYOK / Embedded / Translations |

### 3. Docs Mirror(2026-10-15)

`E:/obsidian/evidence/_sources/evidence-docs/`,168 页 docs mirror,sitemap 153 URL 全部抓取,BFS + sitemap 遍历混合策略补 88 个 components。

**关键发现**:
- Evidence Warehouse sidebar 列 14 项,但 **sitemap 只有 1 页**(`/evidence-warehouse`),其余全是页内 anchor
- sidebar 列出的 12 个连接器(Athena / Redshift / S3 / etc.)全部独立成页,已在 `data-sources/` 主题中
- 完整 chapter 见 [ch19](chapters/ch19-components-full-reference.md) + [ch20](chapters/ch20-evidence-warehouse-truth.md)

---

## 维护与版本

### 版本演进

- **v1** (2026-10-13) — 初始生成,10 章实战篇
- **v3** (2026-10-14) — R 系列 fold-in,18 章 + 19 R 沉淀
- **v3.1** (2026-10-15) — Docs Mirror fold-in,20 章 + 168 页 mirror + R14 Evidence Warehouse 真相
- **v3.2** (2026-10-25) — 报表页 UI 契约(ch21)+ **token 瘦身**(触发成本 2,662 → 1,556 tok,-42%);参考文件重排到 `references/`

详见 [CHANGELOG.md](CHANGELOG.md)。

### 已知限制

- `cryptography` 包损坏(Python 3.14,需 `pip install --force-reinstall`)
- 13+ page 待修列名 bug(bi.jury.xin 上)
- Browser 真渲染缺 CF Access token
- gbrain keyless 无 semantic search,只能 keyword recall
- gbrain 走共享 HTTP serve(`gbrain serve --http`,端口 3131);PGLite 单写者锁意味着同一时刻只能有一个 serve,write 前先确认 serve 在跑
- docs mirror `html2text` 转换有标题/段落空行(Mintlify 原站特性,内容完整保留)

### 升级频率

- 每周: changelog BREAKING
- 每月: PG slow query 复盘
- 每季: Core 升级(14 天 hold-back)
- 每半年: BI 工具重评

---

## 贡献

本 skill 是 **distilled-from-production** 风格 — 主要由 bi.jury.xin 项目的实战产出。任何 Evidence BI 上的踩坑/调研/决策都可以用以下方式回写:

1. 复制 `E:/obsidian/evidence/_playbook/_踩坑模板.md` 或 `_调研模板.md` 或 `_决策模板.md`
2. 填骨架,标 TODO 等实测结果
3. 完成后可考虑升级对应 chapter,提 PR 到本仓库

门槛(满足 ≥ 2 条):
- 不可逆(切 prod / 选 BI 工具 / 改认证策略)
- 影响 ≥ 1 个月
- 涉及多个有道理的方案
- 未来 agent 重做时必须知道当时为什么

---

## License

MIT — 见 [LICENSE](LICENSE)。

来源: 用户自有项目笔记(bi.jury.xin — Sales BI),通过 [book-to-skill](https://github.com/virgiliojr94/book-to-skill) 的 extract-structure-not-summarizes 原则蒸馏。

---

**Made with production sweat, debugged at 03:00, distilled by agent.**

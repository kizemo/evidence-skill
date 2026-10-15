# Chapter 18: BI Tool Comparison (R10 — Evidence vs Metabase/Superset/Lightdash)

## Core Idea

Evidence 的**直接对手不是 GUI BI 阵营 (Metabase/Superset)**,而是 **Lightdash** (代码化 + dbt-native + Agentic + MCP)。bi.jury.xin 当前无可替代,但**触发阈值**清楚。

## 7 候选工具对比矩阵

| 工具 | License | Stars | 部署 | SQL-first | AI | RBAC/RLS | 适合 |
|---|---|---|---|---|---|---|---|
| **Evidence Core 2026** | MIT | 7.0k | self-host | ✓ | Studio only | Studio only | 工程师团队,小规模 |
| **Metabase** | AGPL | 49.6k | self-host/cloud | 拖拽 + SQL | 商业 | ✓ (完整) | 业务用户,中等规模 |
| **Superset** | Apache-2.0 | 75.1k | self-host | SQL + 拖拽 | 商业 | ✓ (开源) | 数据分析师 |
| **Streamlit** | Apache-2.0 | 46.0k | self-host | Python data app | BYOK | ✗ | 数据科学 app |
| **Observable Framework** | ISC | 3.7k | self-host | JS notebook | ✗ | ✗ | 数据新闻 |
| **Lightdash** | NOASSERTION | 6.2k | self-host | dbt YAML | ✓ (MCP) | ✓ | dbt 项目 |
| **Tableau Cloud** | 商业 | - | SaaS | 拖拽 | ✓ | ✓ | 大企业 |

## 关键决策矩阵

| 评估维度 | Evidence 优势 | Metabase 优势 | Lightdash 优势 |
|---|---|---|---|
| 工程师上手 | ★★★★★ (markdown) | ★★ (拖拽) | ★★★★ (YAML) |
| 业务用户上手 | ★ (代码) | ★★★★★ (拖拽) | ★★ (需懂 dbt) |
| 版本控制 | ★★★★★ (git-native) | ★ (DB) | ★★★★★ (git YAML) |
| AI Agent | Studio only | 商业 | ★★★★★ (MCP) |
| 认证/SSO | self-host 弱 (Basic only) | ★★★★★ (完整) | ★★★★ |
| RLS | Studio only | ★★★★★ (sandboxing) | ★★★★ |
| 自托管成本 | 服务器费 | 服务器费 | 服务器费 |
| 7 候选中 MIT | ✓ 唯一 | ✗ (AGPL) | ✗ (NOASSERTION) |
| 社区 star | 7.0k | 49.6k | 6.2k |

## Evidence 独门优势 (vs 全部)

1. **代码即源真**: `.md` 文件 git-versioned,code review 友好
2. **静态 + 动态混合**: 一份 markdown,build-time + run-time 兼顾
3. **AI agent 原生集成** (Studio + BYOK)
4. **组件即数据**: chart 用 markdown 模板,无需写 JS
5. **第一个 MIT + 工程师团队友好的 BI**

## Evidence 短板 (vs 替代)

1. **无 mobile native app** (Metabase / Tableau 有)
2. **Self-host 缺 RLS** (Metabase / Superset 开源就有)
3. **无 drag-and-drop for 业务用户** (Metabase 胜出)
4. **社区小于 Metabase** (49.6k vs 7.0k)
5. **中文教程 zero-content** (R15 调研结论)

## 何时选 Evidence

- ✓ 小团队, 1-3 名 SQL dev
- ✓ SQL-first,no-code 避免
- ✓ Git-native,code-review 友好
- ✓ AI agent 集成 (Studio / 自实现 MCP)
- ✓ 服务器自托管可行

## 何时不选 Evidence

- ✗ 大型非技术业务用户群 (>10 人)
- ✗ 重监管 RLS 需求 (医疗 / 金融)
- ✗ 需要 native mobile / tablet
- ✗ 需要 scheduled email reports (Studio only)
- ✗ 业务用户不愿写 SQL

## 触发 BI 工具重评的硬阈值 (3 个)

| 阈值 | 触发 | 迁移目标 |
|---|---|---|
| **报表 > 30 个** | 维护成本指数级增长 | Lightdash (若已上 dbt) |
| **业务用户 > 10 人 + 任一 RLS/SSO/邮件订阅** | 团队从工程师扩张到非技术 | Metabase Cloud |
| **需要 LLM 玩 BI** | AI 报表助手是硬需求 | Studio Enterprise + BYOK |

## 迁移成本估算 (Lightdash 切换)

| 项目 | 状态 | 切换工作量 |
|---|---|---|
| SQL queries (28 pages) | 已存在 | 0 (复用) |
| dbt YAML (Lightdash 强需) | **未上 dbt** | 1-2 月 (从 0 建 dbt 项目) |
| Charts 模板 (Evidence .md) | 已存在 | 全部重写 (YAML) |
| 认证 (Basic Auth) | 弱 | 重做 (Lightdash SSO) |
| 部署 | Docker | Docker (相似) |
| **总人力** | - | **3-6 人月** |

**当前决策**: bi.jury.xin 维持 Evidence Core 即可,**当前最佳无对手**;触发重评才动。

## 1 个反直觉发现 (Lightdash 真相)

**Lightdash = Evidence + dbt 治理层** — 同样 git-native,同样代码化,**但有 dbt 项目作依赖**。bi.jury.xin 当前 SQL-first 但无 dbt,迁 Lightdash 成本 > 收益。

**dbt v2 2026 默认** (Rust 重写, Apache-2.0): 引入 dbt = 6 月项目(从 0 起步)。**未达触发阈值前不建议**。

## 6 个反直觉发现汇总

1. Metabase 三档协议 (AGPL + Commercial + Starter),OSS 用户改 UI/插 plugin 都需重发协议
2. Streamlit 不属于 BI,当 BI 用是 hack
3. Lightdash = Evidence + dbt 治理,二者未来正面竞争
4. Evidence Self-host = 服务器费 vs Metabase Cloud ≈ $85/月 vs Superset Preset ≈ $3k/月,**小型部署下 Evidence 最便宜**
5. AGPL 不构成 onboarding blocker (我们不改 UI 即不受),但仍是 vs Metabase 的"概念风险"
6. SvelteKit 静态资源 nginx `Cache-Control: public, max-age=31536000, immutable` 后,Evidence 端无独立优化空间

## Connects To

- **ch11** — 组件对比 (Evidence 80+ vs Metabase drag-drop)
- **ch13** — RLS 限制 (Metabase 开源就有 sandboxing)
- **ch17** — 升级路径 (Evidence 无 SemVer, vs Metabase SemVer 严格)
- **ch16 (MCP)** — Lightdash 2026 主推 MCP,Evidence 需自实现

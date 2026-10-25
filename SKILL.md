---
name: evidence-skill
description: >-
  Use when configuring, debugging, or designing Evidence BI Core 2026 pages on
  bi.jury.xin — charts, tables, filters, Chinese-schema SQL, nginx, themes, MCP,
  upgrades, and report-page UI contracts.
triggers:
  - "evidence"
  - "bi.jury.xin"
  - "sales-bi"
  - "Evidence 报表 / 页面 / 组件 / 部署"
  - "combo_chart / legend / page_size / 瀑布图"
  - "Evidence 升级 / MCP / FastMCP"
  - "components 全量 / evidence-warehouse 单页"
mutating: true
brain_first: exempt
when_to_use: "Evidence BI Core 2026 pages on bi.jury.xin."
version: 0.3.2
---

# Evidence Core 2026 — Sales BI

> **改完必验收 3-piece(SQL 直查 + Playwright 真渲染 + 像素采样)——「配置对了」不等于「渲染对了」。**

**Source**: `mirror/`(docs.evidence.dev 公开文档镜像,137 页)+ `templates/` + GBrain;契约以 **prod 二进制实证**为准,不采信文档推测。作者本机原始 vault = `E:/obsidian/evidence/`。
**Generated**: 2026-10-13 初始 → 2026-10-14 fold-in(R1-R21)→ 2026-10-21 v0.3(补 ch21 报表页 UI 契约)→ 2026-10-25 v0.3.1(token 瘦身)→ **v0.3.2(公开文档镜像 mirror/ 随 skill 分发,远程安装自包含)**

## When to use / NOT

| 用本 skill | 不用本 skill |
|---|---|
| Evidence Core 0.10.1 页面/图表/表格/SQL/部署/升级/主题/MCP/报表页 UI 契约 | Legacy 40.x 新功能 · Studio 专属 · 非 PG · 纯 CSS 调优(用 `impeccable`) |

## Chapter Index

21 章按需加载;主题 → 章的完整映射见 `references/topic-index.md`。

| 章组 | 覆盖(章号 → 主题) | 何时读 |
|---|---|---|
| ch01-ch10 | ch01 架构 · ch02 config · ch03 监控 · ch04 cutover 失败 · ch05 SQL 反模式 · ch06 nginx patch · ch07 postgres_fdw · ch08 样式 · ch09 复盘 · ch10 运维速查 | 动 prod 前 |
| ch11-ch18 | ch11 组件 · ch12 部署 · ch13 鉴权 · ch14 性能 · ch15 主题真相 · ch16 FastMCP · ch17 升级 SOP · ch18 BI 工具选型 | 系统性设计 |
| ch19-ch20 | ch19 94 组件全量索引 · ch20 Evidence Warehouse 单页真相 | 找组件 / 架构 |
| **ch21** | **报表页 UI 契约(图表 14 条 · 表格 · 页面内容 · 3-piece · A/B 变体法)** | **改报表页** |

## References(按需加载)

| 文件 | 何时读 |
|---|---|
| `references/cheatsheet.md` | 16 决策表,决策快查(最高优先级) |
| `references/patterns.md` | 8 大模式 + 反模式表,复用现成方案 |
| `references/glossary.md` | 术语不熟 |
| `references/frameworks.md` | 需要解释"为什么" |
| `references/topic-index.md` | 80+ 主题 → 章映射,grep 不到时 |
| `references/protocol.md` | 交付答案格式 · 出错先查 |
| `mirror/` | 94 组件原文 + 8 类公开文档镜像(随 skill 分发,离线可查) |
| `templates/` | 踩坑/调研/决策 3 个写回模板(复制到你的 vault `_playbook/`) |

## Scope & Limits

**覆盖**: Core 0.10.1 配置/部署/排错/升级/性能/主题/MCP/报表页 UI 契约;中文 PG schema;nginx patch;vs Metabase/Lightdash。
**不覆盖**: Legacy 40.x 新功能;Studio 专属;非 PG;多 Core;Pentagram 深度设计。
**已知限制**: `cryptography` 包损坏 (Python 3.14);13+ page 待修列名 bug;gbrain keyless 无 semantic search。
**升级频率**: 每周 changelog BREAKING;每月 PG slow query;每季 Core 升级 (14 天 hold-back)。

## Anti-Patterns(踩坑飞轮)

> 真实踩坑 = 已验证 ≥1 次;不收未验证断言。

| 看到 | 别做 | 真做 |
|---|---|---|
| 中文 `"店铺"` 直接 SQL | 不加双引号 | 必须 `"店铺"` 包裹 — ch05 |
| nginx proxy_pass 改完 | 1 行 sed 重启 | 5 处 patch + 5 秒回滚 — ch06 |
| 改 prod 页面 | 直接 scp | 备份 + diff;**备份必须对应要回到的状态** — ch21 §4 |
| 图表改完 | 看配置就说好了 | 3-piece 验收 — ch21 §5 |
| 想写「怎么改报表」的说明 | 写进报表页 | 放 json / memory / handoff — ch21 §3 |
| 同步脚本加正则 | 用位置组 | 具名组 + 注入漂移往返测试 — ch21 §5 |
| agent 查 Evidence | 暴露 SQL 不拦截 | FastMCP 4 道防线 — ch16 |
| sidebar 找不到组件 | 推断 sidebar=sitemap | BFS+sitemap 混合 — ch20 |

## Verification

- [ ] `wc -w SKILL.md` ≤ 1000(重内容在 chapters/ 与 references/)
- [ ] frontmatter 可被 YAML 解析(description 用块标量)
- [ ] 新契约落在 chapter,没堆在 SKILL.md
- [ ] 改 prod 前后都跑 3-piece
- [ ] 沉淀已回写 Obsidian / memory

## 沉淀回写 SOP(答完/踩坑后主动做)

| 触发 | 立即做 | 模板 |
|---|---|---|
| 切 prod / 升级失败 / ≥2 次重试未修 | 写 `<vault>/YYYY-MM-DD-bug-<short>.md` | `templates/踩坑模板.md` |
| 新技术 / 新工具 / 重大变更前调研 | 写 `<vault>/_research/R<NN>-<topic>.md` | `templates/调研模板.md` |
| 不可逆 / ≥1 月影响决策 | 写 `<vault>/_decisions/YYYY-MM-DD-<short>.md` | `templates/决策模板.md` |
| 组件契约新发现 | 补 `chapters/ch21-*.md`(图表/表格契约) | 契约表 + 踩坑表 |
| 完成后(可选) | `gbrain remember fact="..." entity=projects/sales-bi` | — |

**模板位置**:`templates/`(随 skill 分发;复制到你的 vault `_playbook/` 即可用,模板里的 `../_research/` 是相对**你的 vault** 的路径)。作者本机 vault = `E:/obsidian/evidence/`。
**门槛**(满足 ≥2 条才写):不可逆 / 影响 ≥1 月 / 多个合理方案 / 未来 agent 重做必须知道原因。
**周回写**:每周五或下次 session 开头自审 3 问 —— 这周有什么值得沉淀?属踩坑/调研/决策哪类?有 → 填模板。

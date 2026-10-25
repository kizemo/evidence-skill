# Changelog

Skill evolution history. Format: [Keep a Changelog](https://keepachangelog.com/)

## [v3.2] — 2026-10-25 — Token 效率优化 + 报表页 UI 契约 (ch21)

> 本次同时把仓库追平到安装目录的实际状态 —— v3.1 之后 skill 又经历了 v0.3(2026-10-21,ch21 + 新 frontmatter)与 v0.3.1(本次,token 瘦身)两次演进,仓库此前未同步。

### Added
- **ch21 — Report Page UI Contracts**(1,058 行):图表 14 条契约 · 表格 `page_size` · 页面内容边界 · 3-piece 验收(SQL 直查 + Playwright 真渲染 + 像素采样)· A/B 变体实验法
- **`references/protocol.md`**(36 行):交付输出格式(4 段)+ 出错先查表(7 症状),从 SKILL.md 正文外移
- **`references/cheatsheet.md` §16**:报表页 UI 契约决策 14 条(ch19/ch20/ch21 独有条目下沉,删表前先搬)
- **`references/topic-index.md`**:补 ch19/ch20/ch21 主题映射(原先只覆盖 ch01-ch18)
- **`docs/token-efficiency-audit.md`** + **`docs/skillopt-benchmark.jsonl`**:优化前后实测证据 + 已复核的 16 行评测集

### Changed
- **SKILL.md 瘦身**:156 → 98 行 / 891 → 461 words
  - frontmatter:description 删掉 `Triggers:` 散文子句(9 条短语在 `triggers:` 里已有一份);`when_to_use` 压成一句市场行摘要
  - 正文:`Quick Decision Table`(26 行)→ 下沉到 `references/cheatsheet.md` §16;`Chapter Index`(26 行)→ 压成 4 行章组映射;`Output Format` + `When it fails` → 移进 `references/protocol.md`
- **`triggers:` 补第 7 条** `components 全量 / evidence-warehouse 单页` —— v0.3 迁移时丢掉的 ch19/ch20 路由,补回
- **参考文件重排**:根目录平铺的 `cheatsheet.md` / `patterns.md` / `glossary.md` / `frameworks.md` / `topic-index.md` → `references/`(用 `git mv` 保留历史,与安装目录结构一致)
- **章节内容追平安装目录**:ch05 159→295 行、ch08 120→161 行、ch11 54→283 行等 10 个 chapter 更新
- **Anti-Patterns 飞轮**:5 行 → 8 行(新增 ch21 的 3 条:改 prod 页直接 scp / 图表改完看配置就说好了 / 说明写进报表页)
- **Chapter Index 保留章号映射**:压成 4 行章组时保留「ch01 架构 · ch02 config · …」,agent 不用为查一个章号再读 `references/topic-index.md`(1,119 tok)—— 代价是 load +36 tok,值
- README.md / README.en.md:新增「Token 效率」章节 + 21 章目录 + 触发词同步 + 文件结构树更新 + **60 秒上手 quick start** + 首屏准确率数字对齐实测表(**~22% → ~98%**,原文写 ~30% → ~95% 与表格自相矛盾)

### Token 实测

| 层 | 优化前 | 优化后 | 降幅 |
|---|---|---|---|
| index | 217 tok | **172 tok** | -21% |
| load | 2,445 tok | **1,384 tok** | **-43%** |
| files | 13,769 tok | 14,964 tok | +9%(内容下沉,按需读才付) |
| **触发一次合计** | **2,662 tok** | **1,556 tok** | **-42%** |

口径:CJK ≈ 1 token/char,latin ≈ 1 token/3.5 chars(skill-forge `token_cost.py`,用于预算对比而非计费)。

### Verification
- `skill-forge --scope audit-only` → `index=172tok load=1384tok`,deterministic judge `det=True`,verdict `accept`
- 内容保全 **20/20** 关键条目校验通过(TO_CHAR / force-recreate / taobao.mart / html-tags / theme.yaml / combo_chart / itemStyle / page_size / date_filter.between / legend.data / icon=path:// / WITH CTE / 3-piece / sidebar / Basic Auth / FastMCP 等)
- `wc -w SKILL.md` = 461(≤ 1000 闸门)
- frontmatter YAML 可解析,`description` 用块标量(RULING-5)
- 仓库与安装目录 `~/.claude/skills/evidence-skill/` 逐文件校验一致(SKILL.md + 21 chapters + 6 references)
- 组合对照:全库 60 个 skill 的 index 合计 11,304 tok/会话,本 skill 排 28/60(中位),低于同类 `sql-skill` 的 231 tok

### Known limits
- **index 停在 172,未达 handover 预估的 ~120**。原因:frontmatter 里 CJK 只占 17 tok,剩 155 tok 是 YAML 语法 + 英文散文 + 7 条触发词;再压必须砍 `triggers:` = 砍路由能力。判定为地板,不再继续压。
- **Phase 3 forward 未跑**(exit=2):skill-forge 的 bootstrap benchmark 是通用占位探针(带 `# BOOTSTRAP_PENDING_REVIEW` 标记),设计上就拒绝在未复核探针上做优化。已另存一份**已复核**的 16 行评测集到 `docs/skillopt-benchmark.jsonl`,未来跑 forward 需显式 `--benchmark` 传入(否则每次 run 都会被覆盖)。
- **Anti-Patterns 飞轮仍在观察期**:新增的 ch21 三条尚未经 ≥1 次真实生产验证,1-2 周后复审命中率。

---

## [v3.1.1] — 2026-10-15 — gbrain 共享 HTTP serve 迁移 (patch)

### Changed
- 从 stdio-local `gbrain serve` 迁移到共享 `gbrain serve --http`(端口 3131,绑 127.0.0.1)
- 关闭 2 个 stdio serve(PID 501368 + 真锁主 **PID 528536**),新 HTTP serve PID **771148** 在后台持续运行
- v3.1 注释中引用的"PID 864412"是当时真实值,启动之间已被 MCP gbrain 重启覆盖 — 该值不再代表当前锁主

### Wired
- **Codex**:`~/.codex/config.toml` 写入 `[mcp_servers.gbrain]` inline bearer token(file 收紧到 0600);SessionEnd hook 落盘
- **Claude Code**:`~/.claude/settings.json` 写入 5 个 lifecycle hooks(SessionStart / UserPromptSubmit / Stop / SessionEnd / PreCompact)
- **plugin gbrain@gbrain**:enabled (user-scope)

### Pending (manual)
- Claude Code MCP server registration 未通过 bootstrap 完成 — MSYS2 PATH 中无 `claude.exe`。在 PowerShell 跑一次:
  `claude mcp add gbrain --scope user --transport http http://127.0.0.1:3131/mcp`
  之后 `gbrain doctor` 会将 Claude Code 列入 harness 列表

### Brain snapshot (at migration)
- pages: **18** / chunks: **110**
- engine: pglite @ version **0.60.214.0**
- UPGRADE_AVAILABLE: 0.60.243.0(非阻塞,每次启动刷此行)

### Verification
- `gbrain doctor --only harness_wiring --json` → `{"status":"healthy","health_score":100,"top_issues":[]}`

---

## [v3.1] — 2026-10-15 — Docs Mirror Fold-in

### Added
- **168 页 docs mirror**: `E:/obsidian/evidence/_sources/evidence-docs/` + 7 个主题独立子目录 (components/features/core-concepts/cli/MCP/self-host/Direct Connectors/evidence-warehouse)
- **ch19 — Components Full Reference**: 94 个组件全量索引(按 sidebar 8 大类 V/T/C/M/I/C-UI/L),每组件关键属性 + 本地 wikilink 跳转
- **ch20 — Evidence Warehouse Truth**: 单页文档 + 4 anchor + 12 connector → data-sources 完整映射表
- **CHANGELOG.md**: 本文件

### Changed
- SKILL.md description 加新触发词:`components 全量`, `evidence-warehouse single page`, `docs mirror`, `sidebar 不等于 sitemap`
- SKILL.md Quick Decision Table 加 2 行(ch19 + ch20)
- SKILL.md 章节索引: 18 章 → 20 章,新增 "Docs Mirror 篇 (ch19-ch20)"
- SKILL.md 调研入口: 加 168 页 docs mirror 路径 + 抓取脚本引用
- SKILL.md 已知限制: 加 gbrain lock_held + docs mirror html2text 空行说明
- README.md File inventory: 加 ch19 + ch20 + CHANGELOG.md
- README.md Source: 加 "2026-10-15 docs mirror fold-in" 小节(168 页统计 + R14 关键发现)

### Validation
- sitemap 全覆盖 153 URL,BFS + sitemap 遍历混合策略补 88 个 components
- 0 失败,0 内容丢失
- gbrain ingest 待另一 session (PID 864412) 关闭后进行

---

## [v3] — 2026-10-14 — R-series Fold-in

### Added
- ch11-ch18 (R1-R21): Components / Deployment / Auth-RBAC / Performance / Theme-Pentagram P2 / MCP Self-host / Upgrade Mgmt / BI Tools
- R 系列沉淀到 `E:/obsidian/evidence/_research/` (19 R, 7,150+ 行)

### Changed
- SKILL.md 从 10 章扩到 18 章
- SKILL.md description 加 R 系列触发词
- 加入 "Skill 反馈" 章节(踩坑/调研/决策回写 SOP)

---

## [v1] — 2026-10-13 — 初始生成

### Added
- SKILL.md (master entry)
- ch01-ch10: 实战篇 (2026-10-12 切 prod 笔记)
- 配套文件: glossary.md / patterns.md / cheatsheet.md / topic-index.md / frameworks.md

### Source
- `E:/obsidian/evidence/` 10 production markdown notes
- book-to-skill extract-structure-not-summarizes

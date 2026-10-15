# Changelog

Skill evolution history. Format: [Keep a Changelog](https://keepachangelog.com/)

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

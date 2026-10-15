# evidence-skill 维护任务

> 用途:跟进项目下一阶段 + 给新会话的快速 context。
> 上次更新:2026-10-15(共享 HTTP serve 迁移完成,脑锁释放)

## 当前阶段 / Active Phase

**v3.1 fold-in 已合并**(`81eeeed`)— chapters 19/20 + 168 页 docs mirror 落盘,产物齐备。
最后一步 "gbrain ingest" 因脑锁被另一 agent session 占用而**显式暂停**(原 CHANGELOG v3.1 行)。

2026-10-15 本日 session 完成脑锁释放 + 共享 HTTP serve 迁移:

- ✅ 释放脑锁(杀 PID 501368 + 真锁主 528536)
- ✅ 起 `gbrain serve --http` PID **771148** → `http://127.0.0.1:3131/mcp`,健康持续
- ✅ **Codex** 完整接入:`~/.codex/config.toml` 写 inline bearer token + SessionEnd hook
- ✅ **Claude Code** 5 个 lifecycle hooks 写入 `~/.claude/settings.json`(SessionStart / UserPromptSubmit / Stop / SessionEnd / PreCompact)
- ✅ `gbrain doctor --only harness_wiring --json` → `{status:"healthy", health_score:100, top_issues:[]}`

## 下一步 / Next

- [ ] **gbrain ingest 168 页 docs mirror**(`E:/obsidian/evidence/_sources/evidence-docs/`)
- [ ] **gbrain ingest R1-R21**(19 R, 7150+ 行,位于 `E:/obsidian/evidence/_research/`)
- [ ] **补 Claude Code MCP server 注册**(MSYS2 PATH 找不到 `claude.exe`,需 PowerShell 跑
      `claude mcp add gbrain --scope user --transport http http://127.0.0.1:3131/mcp`)
- [ ] ingest 完后:`gbrain stats --json` 记录新 page/chunk 计数 → CHANGELOG 加 v3.2 条

## 历史里程碑(已 commit,不再回看)

- [x] 2026-10-12 **调研阶段** — agent-skills + skills-from-materials + book-to-skill 三份调研 + 部署报告
- [x] 2026-10-13 **v1** — SKILL.md + ch01-ch10 实战篇 + glossary/patterns/cheatsheet/topic-index/frameworks
- [x] 2026-10-14 **v3** — R-series chapters ch11-ch18(R1-R21 7,150+ 行)
- [x] 2026-10-15 **v3.1** — ch19 components 全量(94) + ch20 evidence-warehouse 真相 + 168 页 docs mirror
- [x] 2026-10-15 **patch** — 共享 HTTP serve 迁移(本 task 单列条目)

## 关键路径索引

| 路径 | 内容 |
|---|---|
| `SKILL.md` | master entry |
| `chapters/ch01-ch20.md` | 20 章核心 |
| `frameworks.md` / `patterns.md` / `glossary.md` / `cheatsheet.md` / `topic-index.md` | 配套速查 |
| `E:/obsidian/evidence/_sources/evidence-docs/` | 168 页 docs mirror(待 ingest) |
| `E:/obsidian/evidence/_research/` | R1-R21 调研系列(待 ingest) |
| `C:/Temp/gbrain-bootstrap-premint-*.log` | bootstrap-harness token 备份 |

## 不要做

- 不要 reset / clean / stash 任何 git 操作(v3 / v3.1 已稳定)
- 不要启动第二个 `gbrain serve --http`(端口 3131 必须独占)
- 不要删除 `C:/Temp/gbrain-bootstrap-premint-*.log`(token 备份)

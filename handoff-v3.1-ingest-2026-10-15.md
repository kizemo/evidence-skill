# Handoff — v3.1 gbrain ingest — 2026-10-15

> 承接对象:下一会话(任意 harness 启 Claude Code / Codex / Cursor)。本文件是 gbrain ingest 工作的全部 context 真相,**不需要重读历史对话**。

## 1. 上次会话交付

- v3.1 fold-in(`81eeeed`):ch19 + ch20 + 168 页 docs mirror 已合并到 main
- 共享 **gbrain HTTP serve** 迁移完成(`bpaq0hcge` 失败的是旧进程的中间产物,正常)
- Codex 完整接入(无遗漏);Claude Code 5 hooks 写入 settings.json(无遗漏);Claude Code MCP server 注册 **未完成**(MSYS2 PATH 限制,见 §5)
- `gbrain doctor` status:healthy,score 100/100,top_issues 空
- `task.md` 重写;`CHANGELOG.md` 加 [v3.1.1] patch 条

## 2. gbrain 接入信息

| 项 | 值 |
|---|---|
| HTTP serve URL | `http://127.0.0.1:3131/mcp` |
| 健康检查 | `curl http://127.0.0.1:3131/health` → `{"status":"ok","engine":"pglite"}` |
| 后台进程 | PID **771148** (bun),task ID `b3uw9rsdw`(background supervisor) |
| 当前 brain 状态 | pages=18, chunks=110 |
| bootstrap-harness token | `gbrain_04d889fd44654e17f8fc052c765c6a5c9f7f74c64a6c925877f35ea15d6c30d4` |
| token 备份 | `C:/Temp/gbrain-bootstrap-premint-*.log` |

⚠️ PGLite 模式下端口 3131 **只能有一个 serve 占用**。启动第二个会拒绝绑定。

## 3. 待 ingest 数据源(本任务的工作)

### 3.1 168 页 docs mirror

```
E:/obsidian/evidence/_sources/evidence-docs/
```

子目录:
- `components/`(88 个 component markdown)
- `features/`(prod 功能页)
- `core-concepts/`(架构/连接/查询/部署基础)
- `cli/`
- `MCP/`
- `self-host/`
- `Direct Connectors/`
- `evidence-warehouse/`(12 connector → 36 data-source 完整映射)

### 3.2 R1-R21 调研系列

```
E:/obsidian/evidence/_research/   (19 R, 7150+ 行)
```

> 跟 ch11-ch18 内容有重叠。ingest 时去重,优先用 chapter 内的版本(ch 是已编辑/压缩过的)。

## 4. ingest 工作流(参考路径)

```bash
# 4.1 确认 serve 健康
curl http://127.0.0.1:3131/health

# 4.2 用 brain 内置 CLI 把目录喂进去
# (具体命令取决于 gbrain 0.60.214 的 ingest 接口;
#  跑 `gbrain ingest --help` 或 `gbrain brain --help` 探)
gbrain ingest --source "obsidian-evidence-docs" \
              --path "E:/obsidian/evidence/_sources/evidence-docs" \
              --strategy bfs+sitemap

# 4.3 R-series 单独 ingest(目录更小,可批处理)
gbrain ingest --source "evidence-research-r" \
              --path "E:/obsidian/evidence/_research" \
              --strategy recursive-md
```

**必做验证**:
```bash
gbrain stats --json             # pages 增长 / chunks 增长
gbrain doctor --only source_sync --json   # source 状态
curl http://127.0.0.1:3131/health  # serve 仍然健康
```

## 5. Manual fallback — Claude Code MCP server 注册

本次没自动跑成。下次会话开头若想补上:

```powershell
# PowerShell(不是 bash)
claude mcp add gbrain --scope user --transport http http://127.0.0.1:3131/mcp
```

跑完后:
```bash
gbrain doctor --only harness_wiring --json   # 应在 harness 列表里看到 "claude-code"
```

## 6. 收尾动作(ingest 完成后)

- [ ] 跑 `gbrain stats --json` 取新 page/chunk 数
- [ ] 写 CHANGELOG 一条 `[v3.2]`(标题建议:"docs mirror + R-series ingested into brain")
- [ ] 更新 `task.md`:这次 ingest 的所有 checkbox 打勾;写下一阶段
- [ ] 写新的 handoff 文档(或在新一轮开局更新本文件 "Resolved" 段)

## 7. 不要做

- 🚫 不要启动第二个 `gbrain serve --http`(端口冲突)
- 🚫 不要 `rm` / 删 `C:/Temp/gbrain-bootstrap-premint-*.log`(token 备份)
- 🚫 不要 git reset --hard / clean -fd 回退 v3 / v3.1
- 🚫 不要改 `~/.claude/settings.json` 的 `_gbrain: bootstrap-harness-v1` 段(那是 hook 锚)
- 🚫 不要写第二条 [v3.1.1] patch 条(本条已记录全部现状)

## 8. 参考

- `task.md`(项目根) — todo 现状
- `CHANGELOG.md` [v3.1.1] — patch 条细节 + brain snapshot
- 本仓 `gbrain doctor --only harness_wiring --json` 输出 — 当下健康证据
- `E:/Users/Duanyi/.gbrain/brain.pglite` — 实际脑数据(别动)

# Handoff — gbrain shared HTTP serve migration + evidence-skill optimize — 2026-10-16

> 承接对象:下一会话(任意 harness 启 Claude Code / Codex / Cursor)。本文件是本会话全部工作的 context 真相,**不需要重读历史对话**。
> 上一轮 handoff: `handoff-v3.1-ingest-2026-10-15.md` — 提供 v3.1 fold-in 上下文,与本文互补。

## 1. 上次会话交付摘要

两件大事:

1. **gbrain 共享 HTTP serve 迁移** — 从 stdio-local 模式迁到 `gbrain serve --http`,Codex 完整接入,Claude Code 5 lifecycle hooks 落盘
2. **evidence-skill user-scope 升级** — skill-forge 6 阶段流水线跑过(audit-only → forward 因 PGLite 锁失败 → 手写 best.md 候选),按 (b) 合并策略:ch19/ch20 文件同步 + 4 处 SKILL.md Edit + 5 行 Anti-Patterns

## 2. 文件清单(本会话所有变更)

### 项目 repo: `F:\soft\00selfmade\evidence-skill\`
- **committed 5a8f707** `docs: add v3.1.1 gbrain HTTP migration patch + handoff`
  - `CHANGELOG.md` +27 行(顶部 [v3.1.1] patch 条)
  - `task.md` 重写为 v3.1 ingest active phase
  - `handoff-v3.1-ingest-2026-10-15.md` 新建

### 用户-scope skill: `E:\Users\Duanyi\.claude\skills\evidence-skill\`
- `SKILL.md` 850 → **995 words**
  - +v3.1 路由词(description 触发词)
  - +Quick Decision 1 行(Evidence Warehouse → ch20)
  - +`## Docs Mirror 篇 (ch19-ch20)` 章节
  - +`## Anti-Patterns (踩坑飞轮)` 段 5 行表(中文列 / nginx 5 patch / sidebar≠sitemap / Self-host RLS / agent SQL)
- `chapters/ch19-components-full-reference.md`(从项目侧 cp)
- `chapters/ch20-evidence-warehouse-truth.md`(从项目侧 cp)

### Claude Code 全局 config: `E:\Users\Duanyi\.claude\`
- `settings.json` +1 段
  - `mcpServers.gbrain` = `{ type: "http", url: "http://127.0.0.1:3131/mcp" }`
  - 5 lifecycle hooks(SessionStart / UserPromptSubmit / Stop / SessionEnd / PreCompact)由 bootstrap harness 写入
  - plugin `gbrain@gbrain` = true(原有)
- 其它 hooks(无 delete-gate / guard-secret-files / cbm-code-discovery-gate / session-start-handoff-bootstrap)**未碰**

### Codex 全局 config: `C:\Users\Duanyi\.codex\`
- `config.toml` +`[mcp_servers.gbrain]` 段(inline bearer token,mode 0600)
- `hooks.json` +SessionEnd hook

### gbrain: `C:\Users\Duanyi\.gbrain\`
- `brain.pglite` — 18 pages / 110 chunks(PGLite 数据库,持久化)
- `config.json` — 引用 brain.pglite 路径
- **install.json 不存在**(skill-forge fail-open = 视为非 bundled)

### Skill-forge 产物(全部 scratch): `C:\Temp\skill-forge-scratch\evidence-skill\`
- `best.md` 997 words(手写候选,留着作对照)
- `audit-report.md` 5 行(skill-forge strict 输出)
- `skillopt-benchmark.jsonl` 16 行(bootstrap 产物)

### 临时日志: `C:\Temp\`
- `gbrain-bootstrap-premint-*.log` — **token 备份,不能删**
- `gbrain-bootstrap-final-*.log` — bootstrap 全流程
- `gbrain-doctor-*.log` — health 审计

## 3. 系统状态(本会话结束时刻,2026-10-16 11:48)

| 系统 | 状态 | 备注 |
|---|---|---|
| **共享 HTTP serve** | 🟡 **DEAD** | PID 771148 死(未知原因),834416 重启后 lock_held,已杀 |
| port 3131 owner | 无 | 空 |
| Codex MCP gbrain | ✅ 完整 wired | 配置文件持久化 |
| Claude Code 5 hooks | ✅ 持久化 | settings.json 段存在 |
| Claude Code MCP server | 🟡 mcpServers 段已加 | **下次 session 启动 + HTTP serve 健康才生效** |
| brain.pglite | ✅ 18 pages / 110 chunks | 持久数据无损 |
| stdio serve PID 742340 | 🟢 running | 锁脑,本会话 MCP gbrain 仍靠它,无 client |
| stdio serve PID 953392 | 🟢 running | 我杀 834416 后 gbrain auto-spawn,lock_holder_blocked,无害 |
| gbrain v0.60.214.0 | 落后 | UPGRADE_AVAILABLE → 0.60.243.0 / 0.60.255.0(每次启动刷此行,非阻塞) |
| evidence-skill user-scope | ✅ 995 words / 20 chapters / 同步 ch19-20 | 路由 + Anti-Patterns 修补落地 |

## 4. 已知问题 / 必须告知的回归

1. **HTTP serve 死了** — 原因未查清(可能是 bun runtime GC / 内存压力 / 别的);session 中途可能再发生,需观察
2. **stdio serve auto-spawn** — gbrain 看到 HTTP serve 死后自动起了 953392 顶(但 stdio 模式),无害
3. **mcpServers.gbrain 段未验证** — 只 add 了 JSON 段,Claude Code 没机会实际加载过(因 serve 一直不健康)
4. **grain 版本落后** — 0.60.214.0 → 0.60.243.0 跨 29 个 minor,无 critical fix 必装的证据,但每次启动刷 UPGRADE 行挺烦
5. **skill-forge strict YAML validator fail** — ORIGINAL SKILL.md 的 description 末尾 `Triggers: '...'` 段在 skill-forge strict yaml.safe_load 解析失败(pre-existing contract drift,与本轮无关)
6. **Anti-Patterns 未经验证** — 5 行表是本会话手写,声称"已踩坑",实际**只有部分经过真实生产使用**;飞轮运转 1-2 周后再审计

## 5. 下次 session 启动序列(必做 4 步)

```powershell
# 1. 清光所有 stdio serve(解除脑锁,腾出 3131)
taskkill /F /IM gbrain.exe /T

# 2. 起共享 HTTP serve(端口 3131,background 保持运行)
gbrain serve --http
# 期望输出: surface=full;健康 curl http://127.0.0.1:3131/health 返回 {"status":"ok","engine":"pglite"}

# 3. (可选)验证 mcpServers wiring
gbrain doctor --only harness_wiring --json
# 期望: { harness:"claude-code" + "codex", status:"healthy", score:100 }

# 4. 启动 Claude Desktop
# mcpServers.gbrain 段自动接管,MCP gbrain 工具自动出现
```

## 6. 之后的下阶段任务(按优先级)

1. **真正 ingest docs mirror + R1-R21 到 brain** — `handoff-v3.1-ingest-2026-10-15.md` 已铺好背景,但本会话未跑(因 PGLite 锁冲突,HTTP serve 死了)
2. **gbrain v0.60.214.0 → 0.60.243.0 升级**(可选) — 跨 29 个 minor,执行前先看 changelog
3. **补 Claude Code MCP server pre-approval** — 当前 settings.json 没有 `permissions.allow` 段,导致每次用 mcp__gbrain__* 工具要手动确认。批量 pre-approve 需要先确认是用户的偏好(避免违反"auto-approve 禁用"红线)
4. **Anti-Patterns 飞轮运转** — 接下来 1-2 周观察 5 行表命中率,不触发的降级,新触发的加进去
5. **下一批 v3.2 内容** — 等 ingest 完成 + 飞轮数据积累再做

## 7. 不要做(红线)

- 🚫 启动第二个 `gbrain serve --http`(端口冲突)
- 🚫 删 `C:/Temp/gbrain-bootstrap-premint-*.log`(token 备份)
- 🚫 evidence-skill 项目 repo `git reset --hard` / `clean -fd`(回退 v3 / v3.1)
- 🚫 改 `~/.claude/settings.json` 的 `_gbrain: bootstrap-harness-v1` 段(那是 hook 锚)
- 🚫 修改 `gbrain@gbrain` plugin 启用状态(0 维 = 关)
- 🚫 跑 `gbrain self-upgrade` 不告诉用户(版本升级要 plan)
- 🚫 跑 `gbrain skillopt` 不先 taskkill 腾脑(PGLite 锁阻塞)

## 8. 阅读顺序(新会话第一分钟)

1. `F:\soft\00selfmade\evidence-skill\handoff-gbrain-shared-http-2026-10-16.md`(本文件,本轮 handoff)
2. `F:\soft\00selfmade\evidence-skill\handoff-v3.1-ingest-2026-10-15.md`(上轮 handoff,提供 v3.1 ingest 上下文)
3. `F:\soft\00selfmade\evidence-skill\CHANGELOG.md` 顶部 [v3.1.1] 条(本轮关键变更摘要)
4. `F:\soft\00selfmade\evidence-skill\task.md`(当前 active phase 状态)
5. `E:\Users\Duanyi\.claude\skills\evidence-skill\SKILL.md`(核心 995-word skill)

## 9. 关键路径索引

| 路径 | 内容 |
|---|---|
| `E:\Users\Duanyi\.claude\settings.json` | Claude Code 全局配置,含 mcpServers + 5 hooks |
| `E:\Users\Duanyi\.claude\skills\evidence-skill\SKILL.md` | 995-word skill,本轮升级的产物 |
| `E:\Users\Duanyi\.claude\skills\evidence-skill\chapters\ch{01-20}-*.md` | 20 章 support |
| `C:\Users\Duanyi\.codex\config.toml` | Codex wired 状态 |
| `C:\Users\Duanyi\.gbrain\brain.pglite` | 实际脑数据,18 pages |
| `C:\Temp\skill-forge-scratch\evidence-skill\best.md` | 997-word 候选,留着对照 |
| `C:\Temp\gbrain-bootstrap-premint-*.log` | **token 备份** |
| `E:/obsidian/evidence/_sources/evidence-docs/` | 168 页 docs mirror(待 ingest) |
| `E:/obsidian/evidence/_research/` | R1-R21 调研系列(待 ingest) |

## 10. 环境基线

- **工作分支**:main(已 commit 5a8f707,working tree clean)
- **Python**: 3.14.6 + pyyaml 6.0.2 + skill-forge 可 import
- **gbrain**: 0.60.214.0 → 0.60.243.0 / 0.60.255.0 可升级
- **PGLite 单写者锁** 必须先 taskkill 才能起 serve
- **会话健康基准**: 单 session <2h,单回合 diff <500 行

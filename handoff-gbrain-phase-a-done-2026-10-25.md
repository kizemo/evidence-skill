# Handoff — gbrain Phase A 完成 + evidence-skill v3.2 已推送 — 2026-10-25(晚)

> 承接对象:下一会话。本文件是本会话全部工作的 context 真相,**不需要重读历史对话**。
> 上一份 handoff:`handoff-gbrain-serve-and-skill-v32-2026-10-25.md`(白天版,已被本文件取代)

## 1. 本会话完成

### ① Phase A —— gbrain 共享 HTTP serve 复活 + 双 harness 收敛 ✅

四步全部达成并验证:

| 步骤 | 结果 |
|---|---|
| `taskkill /F /IM gbrain.exe /T` | 杀掉 6 个 PID(含 3 个 stdio auto-spawn 顶回来的) |
| `gbrain serve --http` | ✅ 端口 3131,`surface=full` |
| `curl /health` | ✅ `{"status":"ok","version":"0.60.287.0","engine":"pglite"}` |
| `doctor --only harness_wiring` | ✅ **healthy / 100 / top_issues []** |

收敛后(`gbrain bootstrap harness --status`):

```
serve: OK at http://127.0.0.1:3131/mcp (engine pglite, v0.60.287.0)
token: OK ('bootstrap-harness' via claude registration — page_count 18, chunk_count 110)
  claude-code/mcp (user): confirmed
  claude-code/permission (user): confirmed
  claude-code/hooks (user): confirmed
  codex/mcp (user): confirmed
  hook entries in settings.json: 5 gbrain (0 without the _gbrain marker)
```

完整 doctor 里 `bootstrap_harness_health`: **fail → ok**。

### ② evidence-skill v3.2 —— token 瘦身 + README 完善 ✅(已推 GitHub)

| 层 | 前 | 后 | 降幅 |
|---|---|---|---|
| index | 217 | **172** | -21% |
| load | 2,445 | **1,384** | -43% |
| **触发一次** | **2,662** | **1,556** | **-42%** |

GitHub `kizemo/evidence-skill`(PUBLIC)已更新 description + 10 个 topics;README 中英双语加了「Token 效率」章节 + 60 秒上手 quick start + 准确率数字对齐实测表。

## 2. 开工第一步

```powershell
# ① 释放脑锁(必须先做!stdio auto-spawn 会立刻顶回来)
powershell.exe -NoProfile -Command "taskkill /F /IM gbrain.exe /T"

# ② 起共享 serve
gbrain serve --http
# ③ 健康检查
curl http://127.0.0.1:3131/health
```

## 3. 下阶段必交付(按优先级)

1. **Phase B · ingest 168 页 docs mirror + R1-R21 到 brain** —— **锁已释放,从未跑过**
   - `E:/obsidian/evidence/_sources/evidence-docs/`(168 页)
   - `E:/obsidian/evidence/_research/`(19 R;与 ch11-ch18 重叠时以 chapter 内版本为准)
   - 验证 `gbrain stats --json`(pages 应 18 → ~200)
   - ⚠️ `doctor` 的 `connection` / `pglite_data_dir` 两条 warn 在共享 serve 下是**常态**(serve 正当其职持锁),不是故障
2. **Phase C · Anti-Patterns 飞轮观察 1-2 周**(ch21 新增 3 条尚未真实生产验证)
3. **(可选)plugin_lane_collision 去重** —— 见 §6,当前判定保留 warn

## 4. 关键决策记录(为什么这么做)

| 决策 | 理由 |
|---|---|
| MCP 注册写**内层** `~/.claude/.claude.json` | 外层 `E:\Users\Duanyi\.claude.json` 是 doctor 从 bash 跑时读的;**活着的 Haha harness 读内层**(它设了 `CLAUDE_CONFIG_DIR=E:\Users\Duanyi\.claude`)。上轮加到外层 → doctor 满意但真 harness 看不见 |
| 用官方 `claude mcp add` 而不是手改 JSON | 它知道写哪个文件(schema 实测:`{"type":"http","url":...,"headers":{...}}`) |
| 自己编 `claude.exe` 转发器放 `C:\Temp\claude-shim\` | gbrain 的解析器**不认 `.cmd`**;`claude-haha.cmd` 转 sidecar 但 sidecar 强制要 mode 参数,复制改名不可行。用 `csc.exe` 编 4.6KB 转发器,**不污染用户 bin 目录** |
| token 轮换用 `--harness all` | 只导 claude-code 的话,旧 token 被 revoke 后 **codex 会挂** |
| 吊销旧 token `30863edf` | 全盘确认零引用后手工吊销(gbrain 的 remove flow 不吊销 supplied token)。吊销前查过 plugin launcher —— 它**无任何 token 逻辑**,走本地 IPC |
| `plugin_lane_collision` 保留不动 | 去掉 HTTP → agent 只剩降级的 status-only 工具集;关 plugin → 丢掉整套 `gbrain:*` skills。两害相权,保留 warn |

## 5. 本轮改动的文件

**用户环境(全部有备份)**:

- `E:\Users\Duanyi\.claude\.claude.json` — 内层,新增 `gbrain` HTTP MCP 条目
- `E:\Users\Duanyi\.claude.json` — 外层,同样新增(doctor 读这个)
- `E:\Users\Duanyi\.claude\settings.json` — 新增 `permissions.allow: ["mcp__gbrain"]`,5 个 gbrain hooks 重确认
- `C:\Users\Duanyi\.codex\config.toml` — `[mcp_servers.gbrain]` inline token(模式 666 → **0600** 收紧)+ trust entry
- `C:\Users\Duanyi\.codex\hooks.json` — Codex SessionEnd hook
- gbrain auth store — 新 token `ee90ac09`(read+write+skills_member_self,115 ops);旧 `30863edf` **REVOKED**
- `C:\Temp\claude-shim\claude.exe` — 给 gbrain 用的 PATH shim(源码 `ClaudeShim.cs` 同目录)
- 备份:`C:\Temp\BK-settings.json.outer` / `BK-claude.json.outer` / `BK-claude.json.inner` / `BK-codex-config.toml` / `BK-old-token.txt`

**repo `F:/soft/00selfmade/evidence-skill/`(已 push,origin/main = `f8fb03d`)**

- `9d67476` feat(skill)!: v3.2 token 瘦身 + ch21 —— 触发成本 -43%
- `f8fb03d` docs: README 完善 + Chapter Index 加回章号映射
- `58882af` docs: handoff + task.md

## 6. 已知遗留(都不阻塞)

1. **`plugin_lane_collision` warn** —— gbrain plugin(stdio)+ 手接 HTTP 两个 owner。理由见 §4。
2. **`shared skills (claude-code / codex): pending`** —— "a supplied token cannot enroll multiple harness identities"。用 supplied token 收敛的固有限制,需每个 harness 独立 explicit grant 才解。
3. **`doctor` 完整跑 status=warnings / score 65** —— 其中 `connection` + `pglite_data_dir` 是共享 serve 持锁的常态;`agent_contract` / `resolver_health` / `skills_manifest_integrity` / `sync_failures` 是先前就有的。
4. **gbrain 0.60.287.0 → 0.60.400.0 可升** —— 跨 113 minor,**未评估,别自动升**。
5. **`C:\Temp\gbrain-bootstrap-premint-*.log` 里的旧 token 已失效**(已吊销),但日志按红线保留不删。

## 7. 禁止事项

- 🚫 启动第二个 `gbrain serve --http`
- 🚫 跑 `gbrain skillopt` 不先 taskkill 腾脑(PGLite 单写者锁)
- 🚫 `gbrain self-upgrade`(跨 113 minor 未评估)
- 🚫 删除 `C:/Temp/gbrain-bootstrap-premint-*.log`(token 备份)
- 🚫 手工改 `~/.claude/.claude.json` 的 gbrain 条目 —— 用 `claude mcp add gbrain --scope user --transport http <url> --header "Authorization: Bearer <token>"`
- 🚫 在 MSYS2 bash 直接跑 `taskkill /F /IM`(路径转换吃标志位,必须套 PowerShell)
- 🚫 伪造 skillopt benchmark 行(bootstrap 是占位探针,设计上拒绝在未复核探针上优化)
- 🚫 evidence-skill repo `git reset --hard` / `clean -fd`

## 8. 必读顺序

1. `F:/soft/00selfmade/evidence-skill/task.md`(当前 active phase + 本轮新发现表)
2. `F:/soft/00selfmade/evidence-skill/docs/token-efficiency-audit.md`(若还要继续压 token)
3. `E:/Users/Duanyi/.claude/skills/evidence-skill/SKILL.md`(98 行的成品)
4. `F:/soft/00selfmade/evidence-skill/CHANGELOG.md` 顶部 `[v3.2]`

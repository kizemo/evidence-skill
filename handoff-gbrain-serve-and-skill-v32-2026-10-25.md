# Handoff — gbrain HTTP serve 复活 + evidence-skill v3.2 token 瘦身 — 2026-10-25

> 承接对象:下一会话(任意 harness 启 Claude Code / Codex / Cursor)。
> 本文件是本会话全部工作的 context 真相,**不需要重读历史对话**。
> 上一份 handoff:`handoff-gbrain-shared-http-2026-10-16.md` —— gbrain 迁移的原始背景,与本文互补。

## 1. 本会话完成的两件事

### ① evidence-skill v3.2 token 瘦身(**已完成并推送** `9d67476`)

| 层 | 优化前 | 优化后 | 降幅 |
|---|---|---|---|
| index(每会话都付) | 217 tok | **172 tok** | -21% |
| load(每次触发付) | 2,445 tok | **1,348 tok** | **-45%** |
| **触发一次合计** | **2,662 tok** | **1,520 tok** | **-43%** |

`SKILL.md` 156 → 98 行 / 891 → 461 words。做法与证据全部在 `docs/token-efficiency-audit.md`。

### ② gbrain 共享 HTTP serve —— **仍是死的,未做**

本轮只做了 taskkill 腾脑,HTTP serve **没有重新起来**。Phase A/B/C 原封不动。

## 2. 开工前必做(2 条命令)

```powershell
# ① 必须先用 PowerShell —— MSYS2 bash 会把 /F /IM 路径转换吃掉
powershell.exe -NoProfile -Command "taskkill /F /IM gbrain.exe /T"

# ② 起共享 HTTP serve(端口 3131,background 保持运行)
gbrain serve --http
# 期望:curl http://127.0.0.1:3131/health → {"status":"ok","engine":"pglite"}
```

**⚠️ 本轮新发现的坑 —— stdio auto-spawn 循环**:
脑锁一释放,gbrain 立刻起 stdio 进程顶上来(本轮观察到 3 个:249632 / 263644 / 413556)。
**taskkill 之后要马上起 serve**,否则锁又被占,PGLite 单写者冲突。
本轮实测:taskkill 后进程数 3 → 0,但一跑 gbrain 命令它就自己长回来。

## 3. 下阶段必交付(按优先级)

1. **Phase A · serve 复活 + 双 harness 验证**
   - `gbrain serve --http`(3131)
   - `curl http://127.0.0.1:3131/health`
   - `gbrain doctor --only harness_wiring --json` → 期望 claude-code + codex 都 healthy
   - (可选)PowerShell `claude mcp add gbrain --scope user --transport http http://127.0.0.1:3131/mcp`
2. **Phase B · ingest 168 页 docs mirror + R1-R21**(因锁阻塞**从未跑过**)
   - `E:/obsidian/evidence/_sources/evidence-docs/`(168 页)
   - `E:/obsidian/evidence/_research/`(19 R;与 ch11-ch18 重叠时以 chapter 内版本为准)
   - 验证 `gbrain stats --json`(pages 应 18 → ~200)+ serve 仍健康
3. **Phase C · Anti-Patterns 飞轮观察 1-2 周**(ch21 新增 3 条尚未真实生产验证)

## 4. 环境基线(本会话结束时刻)

| 项 | 值 |
|---|---|
| 分支 / HEAD | `main` / `9d67476`(已 push,origin/main 已确认) |
| working tree | clean |
| **gbrain 版本** | **0.60.287.0** —— 上一份 handoff 记的 0.60.214.0 **已过时**,期间自动升级 |
| gbrain 进程 | 3 个 stdio(249632 / 263644 / 413556),占着 PGLite 锁 |
| port 3131 | **空**,`/health` 无响应 |
| brain.pglite | 18 pages / 110 chunks(持久数据无损) |
| skill 触发成本 | index 172 / load 1,348 / 合计 1,520 tok |
| 全库 index 合计 | 11,304 tok/会话(60 个 skill) |

## 5. 本轮改动的文件(全部已 commit + push)

**repo `F:/soft/00selfmade/evidence-skill/`(commit `9d67476`)**

- `SKILL.md` — 瘦身重写(98 行)
- `references/cheatsheet.md` — 头部覆盖说明 + **新增 §16**(14 条报表页 UI 契约决策)
- `references/topic-index.md` — 补 ch19/ch20/ch21 的 12 条主题映射
- `references/protocol.md` — **新建**(交付输出格式 + 出错先查表)
- `chapters/ch21-report-ui-contracts.md` — **新建**(1,058 行)
- `chapters/ch05, ch08, ch11-ch18` — 追平安装目录(10 个 chapter)
- `cheatsheet/patterns/glossary/frameworks/topic-index.md` → `references/`(`git mv`,保留历史)
- `docs/token-efficiency-audit.md` — **新建**(前后实测证据 + 地板分析)
- `docs/skillopt-benchmark.jsonl` — **新建**(已复核的 16 行评测集)
- `README.md` / `README.en.md` — 新增「Token 效率」章节 + 21 章目录 + 触发词同步 + 文件结构树
- `CHANGELOG.md` — `[v3.2]` 条
- `task.md` — 重写为当前 active phase

**安装副本 `E:/Users/Duanyi/.claude/skills/evidence-skill/`** —— 与 repo **逐文件校验一致**
(SKILL.md + 6 references + 21 chapters)。另含 skill-forge 自动产物 `audit-report.md` / `best.md` /
`skillopt-benchmark.jsonl`(bootstrap 占位探针,带 `# BOOTSTRAP_PENDING_REVIEW` 标记)。

## 6. 关键决策记录(为什么这么做)

| 决策 | 理由 |
|---|---|
| `triggers:` 保留 7 条而不是砍到 3 条 | `triggers:` 是触发词的**唯一机器可读归属地**,skill-forge 的 benchmark 正样本由它派生(`author.py`)。砍它 = 砍路由能力 |
| 补回第 7 条 `components 全量 / evidence-warehouse 单页` | v0.3 迁移时丢的 ch19/ch20 路由,会导致 ch19/ch20 漏触发 |
| index 停在 172,不压到 handover 预估的 120 | `token_cost.py` 实测:frontmatter 里 CJK 只占 17 tok,剩 155 tok 是 YAML 语法 + 英文散文。再压只能砍 triggers。**地板,不是没做到** |
| 不伪造 skillopt benchmark 行 | bootstrap 是通用占位探针,设计上就拒绝在未复核探针上做优化(forward exit=2 是正确行为)。另存一份**真**的已复核评测集到 `docs/skillopt-benchmark.jsonl` |
| 先搬内容再删表 | `cheatsheet.md` 原本只覆盖 ch01-ch18,ch21 的 7 条决策在 SKILL.md 里是**孤本**。删表前先下沉,20/20 校验通过 |
| Phase 3 forward 不跑 | 本次是**确定性结构重构**,不是需要 LLM 判优的语义改写;且 gbrain PGLite 锁需要先释放 |

## 7. 禁止事项

- 🚫 启动第二个 `gbrain serve --http`(端口冲突)
- 🚫 删 `C:/Temp/gbrain-bootstrap-premint-*.log`(token 备份)
- 🚫 跑 `gbrain skillopt` 不先 taskkill(PGLite 单写者锁)
- 🚫 `gbrain self-upgrade` 不告诉用户(现已是 0.60.287.0,别乱动)
- 🚫 在 MSYS2 bash 直接跑 `taskkill /F /IM`(路径转换吃标志位,必须套 PowerShell)
- 🚫 evidence-skill repo `git reset --hard` / `clean -fd`(v3 / v3.1 / v3.2 已稳定)
- 🚫 改 `~/.claude/settings.json` 的 `_gbrain: bootstrap-harness-v1` 段
- 🚫 伪造 benchmark 行骗过 skill-forge 的 gate

## 8. 必读顺序

1. `F:/soft/00selfmade/evidence-skill/handoff-gbrain-serve-and-skill-v32-2026-10-25.md`(本文件)
2. `F:/soft/00selfmade/evidence-skill/task.md`(当前 active phase + 本轮新发现)
3. `F:/soft/00selfmade/evidence-skill/docs/token-efficiency-audit.md`(若还要继续压 token)
4. `F:/soft/00selfmade/evidence-skill/handoff-gbrain-shared-http-2026-10-16.md`(gbrain 迁移原始背景)
5. `E:/Users/Duanyi/.claude/skills/evidence-skill/SKILL.md`(995 → 461 words 的成品)

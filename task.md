# evidence-skill 维护任务

> 用途:跟进项目下一阶段 + 给新会话的快速 context。
> 上次更新:2026-10-25(v3.2 token 瘦身落地并推送 `9d67476`)

## 当前阶段 / Active Phase

**v3.2 已合并并推送**(`9d67476`,origin/main 已确认)——

- SKILL.md 触发成本 **2,662 → 1,520 tok(-43%)**;156 → 98 行 / 891 → 461 words
- 补齐 **ch21 报表页 UI 契约**(1,058 行)
- 参考文件重排到 `references/`(带历史 `git mv`),新增 `references/protocol.md`
- README 中英双语新增「Token 效率」章节 + 21 章目录
- 内容零丢失已验证(**20/20** 关键条目);路由面未缩水(`triggers:` 补回第 7 条)

**gbrain 共享 HTTP serve 仍是死的** —— 见下方 Next Phase A。

## 下一步 / Next(按优先级)

- [ ] **Phase A · gbrain 共享 HTTP serve 复活**

      1. `powershell -NoProfile -Command "taskkill /F /IM gbrain.exe /T"` ← **必须先做**
      2. `gbrain serve --http`(端口 3131,background 保持运行)
      3. `curl http://127.0.0.1:3131/health` → `{"status":"ok","engine":"pglite"}`
      4. `gbrain doctor --only harness_wiring --json` → claude-code + codex 双 healthy
      5. (可选)PowerShell 跑 `claude mcp add gbrain --scope user --transport http http://127.0.0.1:3131/mcp`
         (MSYS2 PATH 里没有 `claude.exe`,必须用 PowerShell)

- [ ] **Phase B · ingest 168 页 docs mirror + R1-R21 到 brain**(此前因锁阻塞从未跑过)

      - `E:/obsidian/evidence/_sources/evidence-docs/`(168 页)
      - `E:/obsidian/evidence/_research/`(19 R,7,150+ 行;与 ch11-ch18 重叠时以 chapter 内版本为准)
      - 验证:`gbrain stats --json`(pages 应 18 → ~200)+ `curl /health` 仍 ok

- [ ] **Phase C · Anti-Patterns 飞轮观察 1-2 周**

      8 行表;ch21 新增的 3 条(改 prod 页直接 scp / 图表改完看配置就说好了 / 说明写进报表页)尚未经 ≥1 次真实生产验证,到期复审命中率

## 本轮新发现(重要)

| 事实 | 值 |
|---|---|
| **gbrain 版本** | **0.60.287.0** —— 上一份 handoff 记的 0.60.214.0 已过时,期间自动升级 |
| **stdio auto-spawn 循环** | 脑锁一释放,gbrain 立刻起 stdio 进程顶上来(本轮观察到 3 个:249632 / 263644 / 413556)。**起 HTTP serve 前必须先 taskkill**,否则端口 + PGLite 锁被占 |
| port 3131 | **空** —— HTTP serve 未运行,`/health` 无响应 |
| skill 触发成本 | index 172 / load 1,348 / 合计 **1,520 tok** |
| 全库 index 合计 | 11,304 tok/会话(60 个 skill);下一个该优化的是 `long-doc-processor`(**1,516 tok**,是本 skill 的 9 倍) |

## 历史里程碑(已 commit,不再回看)

- [x] 2026-10-12 **调研阶段** —— agent-skills + skills-from-materials + book-to-skill 三份调研 + 部署报告
- [x] 2026-10-13 **v1** —— SKILL.md + ch01-ch10 实战篇 + glossary/patterns/cheatsheet/topic-index/frameworks
- [x] 2026-10-14 **v3** —— R-series chapters ch11-ch18(R1-R21 7,150+ 行)
- [x] 2026-10-15 **v3.1** —— ch19 components 全量(94)+ ch20 evidence-warehouse 真相 + 168 页 docs mirror
- [x] 2026-10-15 **patch v3.1.1** —— 共享 HTTP serve 迁移(Codex wired + Claude Code 5 hooks 落盘)
- [x] 2026-10-25 **v3.2** —— ch21 报表页 UI 契约 + **token 瘦身(-43%)** + `references/` 重排 + README 更新

## 关键路径索引

| 路径 | 内容 |
|---|---|
| `SKILL.md` | master entry(98 行) |
| `chapters/ch01-ch21.md` | 21 章核心 |
| `references/` | 6 个按需加载参考(cheatsheet / patterns / glossary / frameworks / topic-index / protocol) |
| `docs/token-efficiency-audit.md` | token 优化前后实测证据 + 地板分析 |
| `docs/skillopt-benchmark.jsonl` | **已复核**的 16 行评测集(forward phase 需 `--benchmark` 显式传入,否则被 bootstrap 覆盖) |
| `E:/obsidian/evidence/_sources/evidence-docs/` | 168 页 docs mirror(待 ingest) |
| `E:/obsidian/evidence/_research/` | R1-R21 调研系列(待 ingest) |
| `C:/Temp/gbrain-bootstrap-premint-*.log` | bootstrap-harness token 备份 |
| `~/.claude/skills/evidence-skill/` | 安装副本(与本 repo 逐文件校验一致) |

## 不要做

- 不要 reset / clean / stash 任何 git 操作(v3 / v3.1 / v3.2 已稳定)
- 不要启动第二个 `gbrain serve --http`(端口 3131 必须独占)
- 不要删除 `C:/Temp/gbrain-bootstrap-premint-*.log`(token 备份)
- 不要跑 `gbrain skillopt` 不先 taskkill 腾脑(PGLite 单写者锁)
- 不要伪造 skillopt benchmark 行 —— bootstrap 是通用占位探针(`# BOOTSTRAP_PENDING_REVIEW`),设计上就拒绝在未复核探针上做优化
- 不要在 MSYS2 bash 里直接跑 `taskkill /F /IM` —— 路径转换会吃掉标志位,必须套 `powershell.exe -NoProfile -Command`

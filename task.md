# evidence-skill 维护任务

> 用途:跟进项目下一阶段 + 给新会话的快速 context。
> 上次更新:2026-10-25(Phase A 完成 — gbrain 共享 HTTP serve 复活 + 双 harness 收敛)

## 当前阶段 / Active Phase

**Phase A ✅ 完成(2026-10-25)** —— gbrain 共享 HTTP serve 复活,双 harness 全部收敛:

| 项 | 结果 |
|---|---|
| `gbrain serve --http` | ✅ 端口 3131,`/health` → `{"status":"ok","version":"0.60.287.0","engine":"pglite"}` |
| `gbrain doctor --only harness_wiring` | ✅ **healthy / score 100 / top_issues []** |
| `gbrain bootstrap harness --status` | ✅ claude-code/mcp · permission · hooks · codex/mcp **全部 confirmed** |
| `bootstrap_harness_health` | ✅ **ok**(此前 fail) |
| Claude Code MCP | ✅ `~/.claude/.claude.json` → `gbrain` → http://127.0.0.1:3131/mcp ✓ Connected |
| Codex MCP | ✅ `~/.codex/config.toml` `[mcp_servers.gbrain]` inline token(0600) |
| token | ✅ `ee90ac09-8f81-4278-a6c2-9b9e3c88bf7e`(read+write+skills_member_self,115 ops) |

**evidence-skill v3.2 也已推送**(`9d67476` + `f8fb03d`)—— 触发成本 2,662 → 1,556 tok(**-42%**),21 章 + 6 references,GitHub `kizemo/evidence-skill` PUBLIC。

## 下一步 / Next(按优先级)

- [x] **Phase B · ingest evidence vault 到 brain —— ✅ 完成(2026-10-25 17:29)**

      - **225 页 / 831 chunks** 导入 `evidence-vault` source,0 errors
      - 源数据实况:`E:/obsidian/evidence/` 共 **222 个 .md**(10-19 那次扁平化后,168 页 mirror 变成了
        `components/` 94 + `core-concepts/` 8 + `features/` 14 + `cli/` 3 + `MCP/` 3 + `self-host/` 6 +
        `Direct Connectors/` 8 + `evidence-warehouse/` 1 + `_research/` 21 + `_research/_mirror/` 18 +
        `_sources/evidence-docs/` 32 + 根目录实战笔记 14)
      - **修了 3 个 vault 文件的 YAML frontmatter**(否则 gbrain 拒绝猜 → 那 3 篇进不了脑):
        `2026-10-19-bug-...md`(列表项后跟 `← 注释`)、`R20` / `R12`(`mirror: [a] [b]` 两个 flow 序列挤一行)
      - 检索验证:`components/charts/components-scatter_chart` / `components-combo_chart` /
        `components-range_calendar` / `features-page-settings` 全部命中,type=external-doc
      - `doctor` 完整跑 **29 检查 = 22 ok + 7 warn / 0 fail**,warn 与 import 前完全相同(无新增)
      - ⚠️ `doctor` 的 `connection` / `pglite_data_dir` 两条 warn 在共享 serve 下是**常态**,不是故障

- [x] **facts 层 provider 已解决** —— 配了 `claude-cli` provider(免 API key):
      `facts.extraction_model=claude-cli:claude-fable-5-1` + `pricing.overrides {claude-cli:*:0}`;
      实测 provider 被调用(AI SDK Warning 出现)
- [x] **升级评估(A 方案)** —— **不可行**:`gbrain self-upgrade` 报 "is up to date",
      check-only 报 update_available=false + latest_version 空(网络查不到 GitHub release)。
      **但升级不需要** —— 见下方「facts 层真实范围」
- [ ] **facts 层补齐(只剩最后一步)** —— 阻塞在 **168 file-drift hold**:
      之前 import 时 vault 还不是 git repo,DB 与磁盘不一致。路径:
      ① `gbrain sources status evidence-vault --json` 看清单
      ② `gbrain repair file-drift --source evidence-vault --prefer <file|database>`(要用户决策)
      ③ `gbrain sync --source evidence-vault --no-pull --no-embed` 触发 backstop
      ④ `gbrain doctor --only facts_drain --json` 验证 facts_inserted > 0
      ⚠️ **facts 层只对 15 页 note 开放**(R1/R2/R4/R6/R7/R9/R11/R13/R15/R17/R19/R21 等),
      167 页 external-doc 是**设计上**归 search/query,不进 facts(ELIGIBLE_TYPES)
      详见 `handoff-gbrain-facts-extraction-2026-10-25.md`

- [ ] **Phase C · Anti-Patterns 飞轮观察 1-2 周**

      8 行表;ch21 新增的 3 条(改 prod 页直接 scp / 图表改完看配置就说好了 / 说明写进报表页)尚未经 ≥1 次真实生产验证

- [ ] **(可选)gbrain plugin 去重** —— `plugin_lane_collision` warn:gbrain marketplace plugin(stdio)+ 手接的 HTTP 注册两个 owner 并存。
      plugin 那条走 stdio,会另起进程抢 PGLite 锁 → 已是 status-only 降级;HTTP 那条全功能。
      **去掉 HTTP 会让 agent 只剩降级工具集,关 plugin 会丢掉整套 `gbrain:*` skills** → 当前判定:保留 warn 不动配置。

## 本轮新发现(重要)

| 事实 | 值 |
|---|---|
| **两个分叉的 `.claude.json`** | `E:\Users\Duanyi\.claude.json`(外层,doctor 从 bash 跑时读这个)vs `E:\Users\Duanyi\.claude\.claude.json`(**内层,活着的 Haha harness 读这个**,因为 `CLAUDE_CONFIG_DIR=E:\Users\Duanyi\.claude`)。**改 MCP 注册必须改内层**,否则 doctor 满意但真 harness 看不见 |
| **gbrain 要真 `claude.exe`** | 机器上只有 `claude-haha.cmd`(转 sidecar),gbrain 的解析器**不认 `.cmd`**。已在 `C:\Temp\claude-shim\claude.exe` 用 `Add-Type` 编了一个转发器(`csc.exe` 在 `C:\Windows\Microsoft.NET\Framework64\v4.0.30319\`),**没有污染用户 bin 目录** |
| **gbrain 版本** | **0.60.287.0** → 有 0.60.400.0 可升(跨 113 minor,未评估,**别自动升**) |
| **stdio auto-spawn 循环** | 脑锁一释放,gbrain 立刻起 stdio 进程顶上来。**起 HTTP serve 前必须先 taskkill**,否则端口 + PGLite 锁被占 |
| **铸 token 需独占脑** | `gbrain auth create/rescope/revoke` 都要 DB 直连 → 必须先停 serve。流程:停 serve → 铸/改 token → 重启 serve → `bootstrap harness --token` |
| **token 重名** | `gbrain auth create bootstrap-harness` 会和旧的同名 → rescope 要 `--id <uuid>`(从 `gbrain auth list` 拿) |
| 全库 index 合计 | 11,304 tok/会话(60 个 skill);下一个该优化的是 `long-doc-processor`(**1,516 tok**) |
| **facts_drain 上游 bug** | gbrain 0.60.287.0 的 job insert 违反自家 `minion_jobs.chk_attempts_order` 约束 → facts-absorb 任务 error。`gbrain repair` 无对应 kind,`apply-migrations` 显示全部已应用 → 只有升级能解。**与本次 import 无关**(报错发生在 import 之前的 stdio auto-spawn) |
| **`gbrain ingest` 命令不存在** | 正确命令是 **`gbrain import <dir>`**(handoff 里写错了)。相关:`sync` / `capture` / `extract links\|timeline\|all` / `embed` |
| **`gbrain list` 默认 limit=50** | 统计页数必须加 `--limit 1000`,否则看到的是被截断的 50。`sources list` 的 `page_count` 是**磁盘 .md 文件数**,不是 DB 页数 |
| **vault 已被扁平化** | 2026-10-19 把 `_sources/evidence-docs/<子目录>/*` 移到 `E:/obsidian/evidence/<子目录>/*`。老的 `_sources/evidence-docs/` 只剩 32 个 data-sources 页 |

## 历史里程碑(已 commit,不再回看)

- [x] 2026-10-12 **调研阶段** —— agent-skills + skills-from-materials + book-to-skill 三份调研 + 部署报告
- [x] 2026-10-13 **v1** —— SKILL.md + ch01-ch10 实战篇 + glossary/patterns/cheatsheet/topic-index/frameworks
- [x] 2026-10-14 **v3** —— R-series chapters ch11-ch18(R1-R21 7,150+ 行)
- [x] 2026-10-15 **v3.1** —— ch19 components 全量(94)+ ch20 evidence-warehouse 真相 + 168 页 docs mirror
- [x] 2026-10-15 **patch v3.1.1** —— 共享 HTTP serve 迁移(Codex wired + Claude Code 5 hooks 落盘)
- [x] 2026-10-25 **v3.2** —— ch21 报表页 UI 契约 + **token 瘦身(-42%)** + `references/` 重排 + README 完善 + GitHub description/topics
- [x] 2026-10-25 **Phase A** —— gbrain serve 复活 + 双 harness 收敛(token 轮换 + permission 预批准 + 5 hooks 重确认)
- [x] 2026-10-25 **Phase B** —— evidence vault 225 页 / 831 chunks ingest 进 brain(新 source `evidence-vault`)+ 修 3 个 vault frontmatter

## 关键路径索引

| 路径 | 内容 |
|---|---|
| `SKILL.md` | master entry(98 行 / 461 words) |
| `chapters/ch01-ch21.md` | 21 章核心 |
| `references/` | 6 个按需加载参考(cheatsheet / patterns / glossary / frameworks / topic-index / protocol) |
| `docs/token-efficiency-audit.md` | token 优化前后实测证据 + 地板分析 |
| `docs/skillopt-benchmark.jsonl` | **已复核**的 16 行评测集(forward phase 需 `--benchmark` 显式传入) |
| `E:/obsidian/evidence/_sources/evidence-docs/` | 168 页 docs mirror(待 ingest) |
| `E:/obsidian/evidence/_research/` | R1-R21 调研系列(待 ingest) |
| `C:/Temp/gbrain-bootstrap-premint-*.log` | bootstrap-harness token 备份(旧 token,**没删**) |
| `C:/Temp/BK-*.{json,toml}` | 本轮改写前的 4 个配置文件备份 + 旧 token 记录 |
| `C:/Temp/claude-shim/claude.exe` | 给 gbrain 用的 `claude` 转发器(PATH shim) |
| `~/.claude/skills/evidence-skill/` | 安装副本(与本 repo 逐文件校验一致) |

## 不要做

- 不要 reset / clean / stash 任何 git 操作(v3 / v3.1 / v3.2 已稳定)
- 不要启动第二个 `gbrain serve --http`(端口 3131 必须独占)
- 不要删除 `C:/Temp/gbrain-bootstrap-premint-*.log`(token 备份)
- 不要跑 `gbrain skillopt` 不先 taskkill 腾脑(PGLite 单写者锁)
- 不要伪造 skillopt benchmark 行 —— bootstrap 是通用占位探针,设计上就拒绝在未复核探针上做优化
- 不要在 MSYS2 bash 里直接跑 `taskkill /F /IM` —— 路径转换会吃掉标志位,必须套 `powershell.exe -NoProfile -Command`
- **不要手工改 `~/.claude/.claude.json` 的 gbrain 条目** —— 用 `claude mcp add gbrain --scope user --transport http <url> --header "Authorization: Bearer <token>"`(写内层文件)
- 不要 `gbrain self-upgrade`(0.60.287 → 0.60.400 跨 113 minor,未评估)

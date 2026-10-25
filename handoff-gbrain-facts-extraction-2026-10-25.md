# Handoff — gbrain facts 抽取调查 + 升级评估 — 2026-10-25(深夜)

> 承接对象:下一会话。本文件是 facts 抽取问题的**完整真相**,不需重读历史对话。
> 上游 handoff:`handoff-gbrain-phase-b-ingest-done-2026-10-25.md`(ingest 完成)。

## 1. 用户两个请求与结果

| 请求 | 结果 |
|---|---|
| **A: 升级 gbrain 修 facts 抽取**(用户明确选 A,给了备用代理 127.0.0.1:7897) | ❌ **不可行** —— `gbrain self-upgrade` 返回 "is up to date",`--check-only` 报 `update_available:false` + `latest_version:""`。install_method=bun-link(bun install -g github:garrytan/gbrain),升级要拉 GitHub,当前网络 12KB/s。marker 打印的 `0.60.400.0` 是**过时缓存**。 |
| **B: 解决 facts 层无 API key / 无 model 问题** | ✅ **已解决** —— 发现 `claude-cli` provider 免 key(靠本机 claude CLI 的 OAuth 会话),已配 `facts.extraction_model=claude-cli:claude-fable-5-1` + `pricing.overrides {claude-cli:*:0}`。 |

## 2. 核心结论:升级根本不需要

facts 抽取的真正障碍**不是** `chk_attempts_order`(那是重试才炸的**次生症状**),而是三个设计性事实:

1. **`gbrain import` 故意不触发 backstop** —— `import-file.ts:1165` 注释原文 "Wiring runFactsBackstop here would always produce a no-op envelope"
2. **managed mode 写管线** —— 给 vault 做 `git init` 后进入 managed mode,任何写操作(含 facts 回写)必须走 `persistence_request_id` 管线,手动 `jobs submit facts-absorb` 会 `skipped: missing_write_authority`
3. **facts 层只对部分类型开放** —— `ELIGIBLE_TYPES = {note, meeting, slack, email, calendar-event, source, writing}`,evidence-vault 225 页里 **167 页是 external-doc**(参考文档**设计上**归 search/query,不进 facts),合格的只有 **15 页 note**

## 3. facts 层的真实范围(不是 225 页)

15 页 note 型(这才是要抽 facts 的):
`_research/r1,r2,r4,r6,r7,r9,r11,r13,r15,r17,r19,r21` + `research-evidence-blueprint` + `task-p1-mirror-notes` + `prompt-p1-mirror-notes`

## 4. 当前阻塞:168 file-drift hold

`gbrain sync` 输出:
```
0 file(s) held this run, 168 held in source evidence-vault
Managed sync synced: 0 entries this run
```

原因:之前 `gbrain import` 时 vault **还不是 git repo**,DB 与磁盘/git 状态不一致。
增量 sync 判定 "up to date",不触发 backstop。

**下一步(按序)**:
```bash
# 1. 看 168 个 hold 的具体清单
gbrain sources status evidence-vault --json

# 2. 逐个或批量预览修复(注意:file vs database 二选一,要用户决策)
gbrain repair file-drift --source evidence-vault --slug <slug> --prefer database   # 或 --prefer file

# 3. 修完后重新 sync,触发 backstop
gbrain sync --source evidence-vault --no-pull --no-embed

# 4. 验证 facts 抽取
gbrain doctor --only facts_drain --json   # 看 facts_inserted 是否 > 0
```

## 5. 已验证的路径(部分)

- `gbrain jobs submit facts-absorb --params '{"slug":"_research/r11-performance","sourceId":"evidence-vault"}' --follow` → 跑通了,job 里出现 `AI SDK Warning (claude-cli / claude-fable-5-1)` **证明 provider 被调用**;但被 `skipped: missing_write_authority` 挡在 LLM 调用**之前**(readFactsBackstopJobPage 阶段就 return)
- 结论:provider 没问题,纯粹是 managed mode 的写权限门槛

## 6. 关键环境状态

| 项 | 值 |
|---|---|
| gbrain 版本 | 0.60.287.0(**不可升级**,self-upgrade 报 up to date) |
| facts.extraction_model | `claude-cli:claude-fable-5-1`(免 key) |
| pricing.overrides | `{"claude-cli:*":0}`(订阅制不该按 API 价计费) |
| claude.exe shim | `C:\Users\Duanyi\.local\bin\claude.exe`(4608B C# 转发器,已在 PATH) |
| vault git | `E:\obsidian\evidence` 已 git init,初始提交 `2431321` |
| managed mode | 已启用(`.gbrain-owner.json` 有 token + worktreeId) |
| serve | 会话中被 stdio auto-spawn 反复抢锁,需要时 taskkill |

## 7. 禁止事项

- 🚫 不要在 managed mode 下手动 `jobs submit facts-absorb`(会 skipped missing_write_authority)
- 🚫 不要跑 `gbrain import` 触发 facts(设计上 no-op)
- 🚫 不要试图升级 gbrain(网络不可行 + self-upgrade 报 up to date)
- 🚫 不要手动改 `~/.claude/.claude.json` 的 gbrain 条目
- 🚫 在 MSYS2 bash 直接 `taskkill /F /IM`(必须套 PowerShell)
- 🚫 `gbrain repair file-drift` 不先确认 --prefer file 还是 database(这是数据决策,要用户拍板)

## 8. 必读顺序

1. 本文件(事实真相)
2. `task.md`(active phase)
3. `reference_gbrain_ingest_mechanics.md`(ingest 机制三坑)
4. `reference_gbrain_http_serve.md`(serve + facts_drain bug)

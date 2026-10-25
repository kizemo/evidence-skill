# Handoff — gbrain Phase B 完成(evidence vault 225 页入库)— 2026-10-25(傍晚)

> 承接对象:下一会话。本文件是 Phase B 的 context 真相,**不需要重读历史对话**。
> 上一份 handoff:`handoff-gbrain-phase-a-done-2026-10-25.md`(Phase A,已被本文件取代)

## 1. Phase B 结果

| 项 | 结果 |
|---|---|
| import | ✅ **225 页 / 831 chunks / 0 errors** |
| source | ✅ 新建 `evidence-vault` → `E:\obsidian\evidence`(federated,incarnation `45db39d8`) |
| 检索 | ✅ `components/charts/components-scatter_chart`、`components-combo_chart`、`components-range_calendar`、`features-page-settings` 全部命中,type=external-doc |
| doctor | ✅ 完整跑 **29 检查 = 22 ok + 7 warn / 0 fail**,warn 与 import 前完全相同(无新增) |
| harness_wiring | ✅ healthy / 100 / top_issues [] |

命令(供复现):
```bash
gbrain sources add evidence-vault --path "E:/obsidian/evidence" --name "Evidence Obsidian Vault" --federated --force
gbrain import "E:/obsidian/evidence" --source evidence-vault --no-embed --workers 4 --json
```

## 2. 关键发现(推翻了好几条旧假设)

### ① `gbrain ingest` 这个命令不存在

handoff 写的 `gbrain ingest --source ... --strategy bfs+sitemap` **是错的**。真实命令:
- `gbrain import <dir> [--source <id>] [--no-embed] [--workers N] [--fresh] [--json]` ← 本次用的
- `gbrain sync [--repo <path>]` / `gbrain capture` / `gbrain extract links|timeline|all` / `gbrain embed`
- 没有 `--strategy` 参数(那个是旧版接口的)

### ② 源数据在 2026-10-19 被扁平化过

handoff 记的 "168 页 docs mirror,7 个子目录" 已经不成立。实际:
- `_sources/evidence-docs/` 只剩 **32 个文件**(29 个 `data-sources-*.md` + editing/publishing/index + 2 json)
- 真正的 mirror 被移到 **`E:/obsidian/evidence/` 顶层**:`components/`(94,25,956 行)/ `core-concepts/`(8)/
  `features/`(14)/ `cli/`(3)/ `MCP/`(3)/ `self-host/`(6)/ `Direct Connectors/`(8)/ `evidence-warehouse/`(1)
- 加 `_research/`(19 R + INDEX + blueprint,7,488 行)+ `_research/_mirror/`(18 个 ch 压缩版,1,184 行)
- 加根目录 14 个实战笔记(01-08 + POSTMORTEM + README + handoff/prompt/task)
- **vault 合计 222 个 .md**

### ③ `gbrain list` 默认 limit=50 —— 之前的页数统计全是错的

- `gbrain list` 默认返回 **50 行**(要 `--limit 1000`)
- `gbrain sources list` 的 `page_count` 是**磁盘 .md 文件数**,不是 DB 页数
  (rime 44=44 精确吻合;taobao-operation 报 2013 而磁盘有 2112)
- 真实 DB 页数(import 前):default 18 + rime 44 + taobao-operation ≥1000 + evidence-vault 0
- doctor 报的 "18 pages / 110 chunks" 也是 `default` 源的口径

### ④ facts 层没被填充 —— 但**不是**那个 upstream bug 造成的

`facts_inserted: 0`、`backlog: {waiting:0, delayed:0, active:0}`。
真相:**import 路径根本不给 facts-absorb 任务入队**,所以没有 drain 被触发。
那个 `minion_jobs.chk_attempts_order` 报错发生在 **17:18**(stdio auto-spawn 撞的),
**在 import(17:25-17:29)之前** —— 是独立的两个问题,先前归因错了。

→ 后果:225 页只进了 **search/query 层**,`recall`(facts)层对 evidence 内容是空的。
→ 补齐方式:手动 `gbrain capture` / `gbrain put_page` 触发抽取,或等 gbrain 升级修好约束。

### ⑤ vault 里 3 个文件的 YAML frontmatter 是坏的

gbrain 拒绝猜,所以那 3 篇进不了脑。已修(只动 frontmatter,正文零改动,备份在
`C:/Temp/vault-frontmatter-fix-2026-10-25/`):

| 文件 | 问题 | 修法 |
|---|---|---|
| `2026-10-19-bug-sales-bi-prod-core-scp-overwrite.md` | 列表项后跟 `← 中文注释`,被当成标量的一部分 | 注释移到独立 `#` 行 |
| `_research/R20-upgrade-and-obsidian.md` | `mirror: [a] [b]` 两个 flow 序列挤一行 | 改 block 样式(与其余 5 篇 R 一致) |
| `_research/R12-styling-theming.md` | 同上 | 同上 |

## 3. 本轮改动的文件

**用户 vault(`E:\obsidian\evidence\`)—— 3 个文件的 frontmatter**
- `2026-10-19-bug-sales-bi-prod-core-scp-overwrite.md`
- `_research/R20-upgrade-and-obsidian.md`
- `_research/R12-styling-theming.md`
- 备份:`C:\Temp\vault-frontmatter-fix-2026-10-25\`

**brain**
- 新 source `evidence-vault`(local_path `E:\obsidian\evidence`)
- evidence-vault 下 225 页 / 831 chunks

**repo `F:/soft/00selfmade/evidence-skill/`** —— `task.md` 更新(Phase B 标记完成 + 修正 facts 归因 + 新增 4 条口径纠正)

**备份**
- `C:\Temp\brain-backup-2026-10-25-172334\` —— import 前的脑快照(1.4G / 1973 文件)
- `C:\Temp\fix-vault-frontmatter.py` / `C:\Temp\sync-token-numbers.py` / `C:\Temp\mcp.py` —— 本轮脚本

## 4. 下一个阶段

1. **Phase C · Anti-Patterns 飞轮观察 1-2 周**(8 行表;ch21 新增 3 条尚未真实生产验证)
2. **(可选)facts 层补齐** —— 见 §2 ④
3. **(可选)plugin_lane_collision 去重** —— plugin stdio + 手接 HTTP 两个 owner,当前判定保留 warn
4. **(可选)gbrain 0.60.287 → 0.60.400 升级** —— 跨 113 minor,**未评估,别自动升**
5. **evidence-skill v3.2 已在 GitHub**(`kizemo/evidence-skill` PUBLIC),下一步可考虑把 brain 的
   `evidence-vault` source 接进 skill 的 `recall` 入口

## 5. 禁止事项

- 🚫 启动第二个 `gbrain serve --http`
- 🚫 跑 `gbrain skillopt` 不先 taskkill 腾脑(PGLite 单写者锁)
- 🚫 跑 `gbrain import` 不先 taskkill —— **stdio auto-spawn 会立刻抢锁**(本轮撞过两次)
- 🚫 `gbrain self-upgrade`(跨 113 minor 未评估)
- 🚫 删除 `C:/Temp/gbrain-bootstrap-premint-*.log`(token 备份)
- 🚫 手工改 `~/.claude/.claude.json` 的 gbrain 条目 —— 用 `claude mcp add gbrain --scope user --transport http <url> --header "Authorization: Bearer <token>"`
- 🚫 在 MSYS2 bash 直接跑 `taskkill /F /IM` —— 必须套 `powershell.exe -NoProfile -Command`
- 🚫 伪造 skillopt benchmark 行
- 🚫 evidence-skill repo `git reset --hard` / `clean -fd`

## 6. 必读顺序

1. `F:/soft/00selfmade/evidence-skill/task.md`(当前 active phase + 本轮 4 条口径纠正)
2. `F:/soft/00selfmade/evidence-skill/handoff-gbrain-phase-b-ingest-done-2026-10-25.md`(本文件)
3. `E:/Users/Duanyi/.claude/skills/evidence-skill/SKILL.md`(98 行的成品)

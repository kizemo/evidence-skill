# Token 效率审计 — evidence-skill v3.2

> **日期**: 2026-10-25
> **工具**: `skill-forge`(`python -m scripts.run --skill <dir> --scope audit-only`)
> **依据**: Perplexity《Designing, Refining, and Maintaining Agent Skills》(三层 token 预算)+ Anthropic 官方 `skill-creator` 篇幅约定
> **原始交接分析**: `F:/soft/00selfmade/skill-optimizer/handover/evidence-skill-token-efficiency.md`

---

## 1. 三层成本模型

一个 skill 被 agent 加载时付两笔(半)钱:

| 层 | 内容 | 何时付 | 预算 | 本次 |
|---|---|---|---|---|
| **index** | 整个 frontmatter(YAML) | **每会话 × 每个用户**,无论是否触发 | ~100 tok | 217 → **172** |
| **load** | SKILL.md 正文 | 每次触发 | ~5,000 tok | 2,445 → **1,384** |
| **files** | `references/` `chapters/` | 只有 agent 真去读时 | 无上限 | 13,769 → 14,964 |

index 层最容易被忽略 —— 它**每会话都付**,并在整个 skill 组合上累加。

---

## 2. 优化前后对照

| 层 | 优化前 | 优化后 | 降幅 |
|---|---|---|---|
| index | 217 tok | **172 tok** | **-21%** |
| load | 2,445 tok | **1,384 tok** | **-43%** |
| files | 13,769 tok | 14,964 tok | +9%(内容下沉,按需读才付) |
| **触发一次合计** | **2,662 tok** | **1,556 tok** | **-42%** |

其它指标:`SKILL.md` 156 → **98 行**;`wc -w` 891 → **461**。

### 测量口径

CJK ≈ 1 token/char,latin ≈ 1 token/3.5 chars。这是**估算**,用于预算对比而非计费。
实现见 `skill-forge/scripts/_lib/token_cost.py` 的 `estimate_tokens()` / `measure_text_tiers()`。

**index 层的构成真相**:index = 整个 frontmatter 原文的 token 估计(含 YAML 语法)。实测拆分:
- CJK 触发词只占 **17 tok**
- 剩 155 tok 是 YAML 语法 + 英文散文 + 7 条触发词的列表开销

→ **继续压 index 只能砍 `triggers:`,那是路由能力。** 判定为地板,不再继续压。

---

## 3. 改了什么

### 3.1 frontmatter 去重(-45 tok)

```diff
 description: >-
   Use when configuring, debugging, or designing Evidence BI Core 2026 pages on
   bi.jury.xin — charts, tables, filters, Chinese-schema SQL, nginx, themes, MCP,
   upgrades, and report-page UI contracts.
-  Triggers: "evidence", "bi.jury.xin", "sales-bi", "Evidence 报表", "combo_chart",
-  "legend", "page_size", "Evidence 部署", "FastMCP evidence".
...
-when_to_use: "Use when the user asks about Evidence BI Core 2026 pages, charts, tables, SQL, deployment, or upgrade."
+when_to_use: "Evidence BI Core 2026 pages on bi.jury.xin."
```

`triggers:` 字段是触发词的**唯一机器可读归属地**(`skill-forge/scripts/author.py`:`"the triggers: field is the single home for trigger phrases"`),benchmark 正样本由它派生。description 里的散文子句是纯重复。

### 3.2 正文去索引化(-1,097 tok)

| 原章节 | 行数 | 去向 |
|---|---|---|
| `## Quick Decision Table` | 26 | → `references/cheatsheet.md` §16(新增 14 条 ch19/ch20/ch21 决策) |
| `## Chapter Index` | 26 | → 压成 4 行章组映射,明细去 `references/topic-index.md` |
| `## Output Format` | 9 | → `references/protocol.md` |
| `## When it fails` | 8 | → `references/protocol.md` |

**保留在正文**(Perplexity 说的"最高价值 gotcha",触发时确实需要):
`When to use / NOT` · `References`(read-conditions 表)· `Scope & Limits` · `Anti-Patterns` · `Verification` · `沉淀回写 SOP`

### 3.3 内容零丢失(改表之前先搬)

`references/cheatsheet.md` 原本只覆盖 ch01-ch18 —— ch21 的 7 条决策在 SKILL.md 里是**孤本**。
`references/topic-index.md` 原本完全没有 ch19/ch20/ch21。

所以删表之前先补:

- `references/cheatsheet.md` §16 —— 14 条报表页 UI 契约决策 + 4 条原先只在 SKILL.md 出现的条目
- `references/topic-index.md` —— 补 12 条 ch19/ch20/ch21 主题映射
- `references/protocol.md` —— 新文件,36 行

**校验**:20/20 关键条目在 `references/` 命中(TO_CHAR / force-recreate / taobao.mart / html-tags-not-allowed / theme.yaml / combo_chart / itemStyle / page_size / date_filter.between / legend.data / icon=path:// / Expand table / WITH CTE / 3-piece / sidebar / Basic Auth / FastMCP / upgrade-disabled / postgres_fdw / prod 真源)

### 3.4 路由面不缩水

v0.3(2026-10-21)迁移时,旧 v3.1 description 里的 14 个触发词被压到 6 个,
`components 全量` / `evidence-warehouse single page` / `sidebar 不等于 sitemap` 丢失 → ch19/ch20 会漏触发。

补第 7 条:`components 全量 / evidence-warehouse 单页`。

---

## 4. 组合对照

全库 60 个 skill,index 层合计 **11,304 tok/会话**。

| skill | index | 备注 |
|---|---|---|
| `long-doc-processor` | 1,516 tok | **真正的 portfolio 大头**,是本 skill 的 9 倍 |
| `erp-to-pgsql-sync` | 439 tok | |
| `impeccable` | 341 tok | |
| `compliance-medical` | 327 tok | |
| … | | |
| **`evidence-skill`(优化前)** | **217 tok** | |
| **`evidence-skill`(优化后)** | **172 tok** | 排 28/60(中位) |
| `sql-skill`(同类参照) | 231 tok | |

结论:本 skill 已低于同类 `sql-skill`;**下一个该优化的是 `long-doc-processor`**。

---

## 5. 复审

```bash
cd ~/.claude/skills/skill-forge
python -m scripts.run --skill ~/.claude/skills/evidence-skill --scope audit-only
```

期望输出:

```
[phase 0] audit: root_cause=other scope=audit-only index=172tok load=1384tok
[phase 4] verify: det=True rubric-skipped spot=None accept=True
```

### Phase 3 forward 为什么没跑

`exit=2` —— skill-forge 的 bootstrap benchmark 是**通用占位探针**(3 个问题循环 5 遍,末尾带
`# BOOTSTRAP_PENDING_REVIEW` 标记),设计上就拒绝在未复核的探针上做优化。

这是**正确行为,不是 bug**。已另存一份**已复核**的 16 行评测集到 `docs/skillopt-benchmark.jsonl`
(8 条 routing 正样本对应 7 个 trigger + 4 条 contract + 4 条 quality 负向探针)。

未来跑 forward 必须显式传参,否则每次 run 都会被 bootstrap 覆盖:

```bash
python -m scripts.run --skill ~/.claude/skills/evidence-skill \
  --scope improve --benchmark docs/skillopt-benchmark.jsonl
```

---

## 6. 已知限制

1. **index 172 未达 handover 预估的 ~120** —— 地板分析见 §2。继续压 = 砍路由。
2. **Phase 3 forward 未跑** —— 见 §5,需显式 benchmark + gbrain PGLite 锁释放。
3. **Anti-Patterns 飞轮仍在观察期** —— ch21 新增的 3 条(改 prod 页直接 scp / 图表改完看配置就说好了 / 说明写进报表页)尚未经 ≥1 次真实生产验证,1-2 周后复审命中率。

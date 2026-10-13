# Chapter 4: Cutover Methodology — Timeline + 5 Critical Bugs

## Core Idea

2026-10-12 07:24 切 prod,07:42 完成 nginx 切换 + Core force-recreate,但**到 09:00 用户浏览器验证时大部分 page 仍报 connection error**。整个流程 4 次提交,3 次"自验通过"实际是 fake success,直到第 4 次才真修。**教训**:**架构问题必须提早 question architecture**,不要"修一个发现另一个"。

## Frameworks Introduced

- **5 步切 prod 流程**:修 handoff → 切 nginx upstream → 5 秒回滚验证 → Core force-recreate → 真验证 SSR HTML
  - When to use: 任何 prod 系统切新版本
  - How: 每步独立验证,不要"切完跑一下"就宣布成功

- **Critical Bug #1-#5 catalog**:
  - When to use: 切任何新系统前,先做"已知失败模式"脑图
  - How: 把已踩过的坑写成"必查清单",不重蹈覆辙

- **5 秒回滚 + 自动 backup**:
  - When to use: 高风险切换
  - How: 切前自动 backup 当前 live config,切后立刻跑 5 page 烟雾测试,失败立即回滚

- **真验证 ≠ grep SSR HTML**:
  - When to use: 任何"我以为好了"的瞬间
  - How: 直连 DB + Python urllib 真 GET + 多 pattern grep(详见 ch09)

## Key Concepts

- **handoff SOP gap**:2026-10-11 handoff 假设"5 秒切 nginx upstream"完全错,真实架构需要 4 处 patch
- **`nginx: [emerg] duplicate location "/"`**:`catch-all` 用 `insert` 而非 `replace` 触发
- **5 个 dev proxy**:vite dev 在 :3000,Core 在 :3002,需全改
- **`html-tags-not-allowed` 替换整块**:Core 编译时发现 raw `<div>` 包裹 component,标记 error,error 替换整个父块包括 component
- **`{{ date_filter.start }}` 字面 `{ }`**:Core 把 `{ }` 当字符串 literal,PG 报 syntax error

## Mental Models

- **切 prod 是连续 4 个独立 sub-task**,每个都要独立验证
- **`docker logs` 只看 connection layer**,SQL 错误要直连 PG 才看得到
- **3 次 fake success 根因**:每次只看自己改的那层是否修,没看用户实际看到的层
- **`html-tags-not-allowed` 不是 limitation,是 critical**:替换整块包括 component,导致 `<Error />` badge 阻塞渲染

## Anti-patterns

- **❌ 切 prod 假设 1 行 sed** —— 实际是 4 处 patch
- **❌ 切完跑 1 次验证就宣布成功** —— 必跑真验证(直连 DB + 多 pattern grep)
- **❌ `connection.yaml` 写 list-of-dicts** —— Core 不识别,silent fallback
- **❌ `evidence.config.yaml` 写 object** —— `project.evidence` 必须是字符串版本号
- **❌ `<div class="hall-kpi">` 包裹 component** —— 触发 `html-tags-not-allowed` 替换整块

## Code Examples

**Bug #1 错误格式**:
```yaml
# WRONG
connections:
  - name: sales_bi
    type: postgres
    database: erp
```
**Bug #1 正确格式**:
```yaml
# CORRECT
type: postgres
host: 124.223.115.201
database: erp
schema: bi
```

**Bug #3 错误 SQL**:
```sql
-- WRONG
WHERE CAST(source_year_month || '-01' AS DATE) BETWEEN DATE '{{ date_filter.start }}' AND DATE '{{ date_filter.end }}'
-- Core 把字面 { } 当字符串,PG 报 syntax error at or near "2026"
```
**Bug #3 正确(任一)**:
```sql
-- CORRECT 1: 用 Evidence 内置 date_filter
WHERE source_year_month {{ date_filter.between }}

-- CORRECT 2: hardcode(option-A trade 失动态 filter)
WHERE source_year_month BETWEEN '2025-01' AND '2026-09'
```

**Bug #4 table rows= 错用法**:
```markdown
-- WRONG
<DataTable data={sales} rows=10 pageSize=20 sort=false />

-- CORRECT (Core v0.10.1 API)
<DataTable data={sales} />
-- pageSize / rows / sort=false 全删
```

**Bug #5 raw `<div>` 包裹 big_value**:
```markdown
-- WRONG
<div class="hall-kpi">
  {% big_value data="pnl_kpis" value="actual_sales" title="实际销售额" text_size="xl" /%}
</div>

-- CORRECT
{% row %}
  {% big_value data="pnl_kpis" value="actual_sales" title="实际销售额" text_size="xl" /%}
{% /row %}
```

## Reference Tables

| Bug | 错 | 对 | commit | 状态 |
|---|---|---|---|---|
| #1 `connection.yaml` LIST | `connections: [list]` | `type: postgres` FLAT 顶层 | `a141446` | ✅ |
| #2 `evidence.config.yaml` object | `evidence: {databases: [...]}` | `evidence: "0.10.1"` STRING | `fcb6bda` | ✅ |
| #3 SQL `{{ date_filter.start }}` 错语法 | 字面 `{ }` | `{{ date_filter.between }}` 或 hardcode | `601ac00` | ✅(hardcode) |
| #4 `sort=false` / `pageSize` 不支持 | `<DataTable rows=10 sort=false />` | `<DataTable />` (空 prop) | `601ac00` | ✅(删) |
| #5 `<div class="hall-kpi">` 包裹 | 6 处 + 18 warning | `{% row %}...{% /row %}` | `f18a74d` | ✅(全删) |

## Key Takeaways

1. **5 critical bugs 中 3 个来自沉默的 silent fallback** —— Core 不报错,只给模糊错误
2. **3 次 fake success 根因**:每次只看自己改的那层,不看用户看到的层
3. **html-tags-not-allowed 不是 limitation** —— 是 critical,会替换整块包括 component
4. **date_filter 字面 `{ }` 是 Core 语法** —— 实际是 placeholder,不是 PG 字符串
5. **架构问题要提早 question architecture** —— 不要"修一个发现另一个"

## Connects To

- **ch01**:AOT binary 决定 force-recreate,影响 Bug #1/#2 修复流程
- **ch02**:Bug #1 + #2 详细配置说明
- **ch05**:Bug A(SUBSTR date)+ Bug B(html-tags-not-allowed)详细
- **ch09**:postmortem 详细分析 3 次 fake success
- **ch10**:下个会话必做 — 推广 SQL fix 到全部 28 page

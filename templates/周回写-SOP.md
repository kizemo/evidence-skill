---
type: sop
title: 周回写 SOP — 每周 Sales BI 决策/调研/踩坑回写
created: 2026-10-15T00:00:00.000Z
parent: ./_README
tags:
  - evidence
  - sales-bi
  - sop
  - weekly
  - 沉淀
---

# 周回写 SOP

> **目的**: 每周把 sales-bi 项目上的"决策 / 调研 / 踩坑"沉淀到 Obsidian,不让新会话重蹈覆辙。
> **触发**: 每周五下午 (或下次会话开头前 5 分钟)。
> **配套**: `_README.md` + `task-p1-mirror-notes-2026-10-15.md`

---

## 1. 自审 3 问 (入口)

1. **本周 sales-bi 项目上发生了什么值得沉淀的?**
2. **事件类型 = 踩坑 / 调研 / 决策?**
3. **满足门槛** (决策模板) 或 **≥2 次重试未修** (踩坑模板) 或 **重大变更前** (调研模板)?

如果 3 问都是 ✅ → 触发回写。

---

## 2. 回写流程 (5 步)

```
1. 复制模板
   踩坑 → _playbook/_踩坑模板.md
   调研 → _playbook/_调研模板.md
   决策 → _playbook/_决策模板.md

2. 新建文件(命名按模板规范)
   踩坑 → <vault>/YYYY-MM-DD-bug-<short>.md
   调研 → <vault>/_research/R<NN>-<topic>.md
   决策 → <vault>/_decisions/YYYY-MM-DD-decision-<short>.md

3. 填骨架 (强约束段)
   踩坑: #1 时序表 + #2 假成功原因 + #4 真修复
   调研: #2 方法 + #3 关键发现 + #4 阶段 actionable
   决策: Context / Decision / Consequences (6 section)

4. 标 TODO 等实测结果
   不要等完美数据再写 — 边做边写

5. (可选) gbrain 沉淀
   gbrain remember fact="<核心结论>" entity=projects/sales-bi
```

---

## 3. 阈值提示

| 信号 | 触发模板 |
|---|---|
| 切 prod / 升级 / 配置变更失败, ≥2 次未修 | **踩坑模板** (强建议) |
| silent fallback 不报错的任何"假成功" | **踩坑模板** (强建议) |
| 用户要求"想用 X 工具",未知 X 表现 | **调研模板** |
| 选 BI 工具 / 改认证策略 / 切 prod 路径 | **决策模板** (门槛 ≥2 满足) |
| R 调研超过 6 月,新事件触发 | **调研模板** 重做 §3 |

---

## 4. 与现有材料的关系

| 已有材料 | 关系 |
|---|---|
| `../README.md` (Evidence 迁移总览) | 回写时关联 |
| `../01-08 + POSTMORTEM` | 已有实战,新踩坑可交叉引用 |
| `../_research/R1-R21` (调研沉淀) | 新调研续编号 R22+ |
| `~/.../_decisions/` (待创建) | 决策 ADR 仓库 |
| GBrain `projects/sales-bi` (5 facts) | 关键事实交叉沉淀 |

---

## 5. 周回写 5 步完成判据

- [ ] 自审 3 问:本周有事件? — Yes/No
- [ ] 类型 = 踩坑/调研/决策?
- [ ] 至少 1 个新笔记?
- [ ] 至少 1 个 wikilink 关联现有材料?
- [ ] gbrain 沉淀 ≥ 1 fact?(可选)

---

## 双向链接

- 入口: `[[_README]]`
- 配套: `[[../task-p1-mirror-notes-2026-10-15]]` P2.4 段
- 触发:`[[../handoff-evidence-obsidian-optimization-2026-10-14]]` §4 P2

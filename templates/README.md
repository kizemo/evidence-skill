---
type: index
title: Playbook 索引 — 踩坑 / 调研 / 决策模板
created: 2026-10-15T00:00:00.000Z
parent: ../
tags:
  - evidence
  - navigation
  - playbook
  - sales-bi
---

# Playbook 索引

> **目的**: 把 sales-bi 项目上"决策、踩坑、调研"沉淀成模板,降低未来 session 启动成本。
> **原则**: 满足"重大决策才写",不强制每件事写一遍。
> **关联**: 配套 P1 完成 mirror 矩阵,P2 加模板机制。

## 3 模板

| 模板 | 何时用 | 真实范例 |
|---|---|---|
| [_踩坑模板.md](_踩坑模板.md) | fake success / ≥2 次重试未修 / 真实 silent fallback | [[../POSTMORTEM-3-false-success]] |
| [_调研模板.md](_调研模板.md) | 新技术 / 新工具 / 重大变更前 | [[../_research/R1-official-docs-overview]] |
| [_决策模板.md](_决策模板.md) | 不可逆 / 影响 ≥1 月 / 多方案 / 未来必知 | (暂无 ADR, Phase 3 待补) |

## 命名规范速查

| 模板 | 路径 | 文件名 |
|---|---|---|
| 踩坑 | `../YYYY-MM-DD-bug-<short>.md` | `2026-10-15-bug-fake-success.md` |
| 调研 | `../_research/R<NN>-<topic>.md` | `R22-evidence-studio-rls.md` |
| 决策 | `../_decisions/YYYY-MM-DD-decision-<short>.md` | `2026-10-15-decision-evidence-core-keep.md` |

> 注: 决策目录 `../_decisions/` 待 P2.4 创建,现决策先放进 `_playbook/` 同级。

## 周回写 SOP (P2.4 配套)

**触发**: 每周五下午 (可选周四晚)。

**自审 3 问**:
1. 本周 sales-bi 项目上发生了什么值得沉淀的?
2. 事件类型 = 踩坑 / 调研 / 决策?
3. ≥1 个 → 复制对应模板 → 填骨架 → 标 TODO 实测结果

**完整 SOP 见**: [[../task-p1-mirror-notes-2026-10-15]] (P2.4 段落)

## 模板修改门槛

- **不要轻易改骨架**: 3 个模板的 section 顺序是经过验证的。
- **可以加 section**: 在 # 6 之后添加 #7-#N(可选)。
- **必填字段不要删**: 任何标 ⚠️ 的都是不可省。
- **改完**: 在 `_README.md` 顶部加 changelog(1 行)。

## 双向链接

- 配套 mirror 笔记: `[[../_research/_mirror/_mirror-ch09-postmortem]]`
- 入口: `[[../README]]`
- handoff: `[[../handoff-evidence-obsidian-optimization-2026-10-14]]`

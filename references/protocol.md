# Protocol — 输出格式与失败处置

> SKILL.md 只保留"触发时必须立刻知道"的决策信息(路由 / 反模式 / 验收 / 回写)。
> 本文件收纳两个**特定场景**才需要的契约:交付时(输出格式)· 出错时(失败处置)。

---

## Output Format(交付答案结构)

1. **结论先行** —— 是什么 / 改哪 / 风险
2. **契约依据** —— 哪条实证,附命令或代码片段
3. **验收证据** —— 3-piece 实际输出(行数 / PASS 数 / 像素断言)
4. **沉淀去向** —— 新契约写进哪个 chapter / memory;**报表页里不写**

---

## When it fails(出错时先查这里)

| 症状 | 先查 |
|---|---|
| 契约查不到 | 组件契约在二进制里:`sudo docker exec sales-bi-core sh -c "grep -ao '.\{200\}<prop>.{400\}' /usr/local/bin/evidence"` |
| prod 与 git 不一致 | 以 prod 真源为准(ch05),别信 git-tracked |
| 页面 500 | 先查中文列名 / 跨 block 引用(ch05) |
| 假成功 | ch09 真验证 SOP(ch21 §5 是页面级细化) |
| 升级后起不来 | ch17 回滚 SOP |
| 图表静默不渲染 | ch21 §1.1 四条(name 位置 / 列名 ASCII / 自闭合 / stacked) |
| 图例整块不显示 | `legend.data` 数组**必须逗号分隔**(ch21 §1.4) |
| 日期筛选器 500 | `{{date_filter.between}}` 单端选时是空串(ch21 §8) |

---

## 关联

- **Anti-Patterns 飞轮**(SKILL.md 正文)—— 已知踩坑,改前先查
- **`references/cheatsheet.md`** —— 决策快查(每条一行)
- **`references/patterns.md`** —— 8 大模式 + 反模式表

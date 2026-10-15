# Claude Code Skills 生态调研任务

## 目标
调研 GitHub 上 Claude Code Skills 生态的开源高星项目,对比分析,输出团队分享用 md 报告。

## 产出
- `agent-skills-调研报告-2026-10-12.md` (1500-2500 字)

## 进度
- [x] 任务分类 + 方案审 (bounded research)
- [x] TaskCreate 4 项任务
- [x] 扫描 GitHub 候选项目
- [x] 深读 Top 项目 README (WebFetch 失败改用 gh API 抓 README base64)
- [x] 构建对比矩阵
- [x] 编写调研报告 md

## 产出
- `agent-skills-调研报告-2026-10-12.md` (350 行,约 2800 字) — **主题已被用户修正,本份仅作安装部署参考保留**
- `skills-from-materials-调研报告-2026-10-12.md` — **主题修正版**:聚焦「如何从书籍/资料/讲座/教程蒸馏出 Skill」的元能力
- `book-to-skill-部署报告-2026-10-12.md` — **部署** 的 virgiliojr94/book-to-skill 到 cc-haha 的完整步骤

## 用户修正(2026-10-12)
- 用户原意:研究**「如何提炼 skill 的 skill」**(meta-skill),不是 Claude 推荐安装的 skill 列表
- 修正:新建 `skills-from-materials-调研报告-2026-10-12.md`,前一份保留作为安装部署参考

## cc-haha 部署(2026-10-12)
- 用户要求:**为 cc-haha 安装部署 virgiliojr94/book-to-skill**
- 部署方式:Claude Code 单 skill 模式(直接复制到 `~/.claude/skills/book-to-skill/`)
- 验证:Claude Code 已自动识别,`python -m book_to_skill --check` 报告所有提取器 ready
- 已知限制:本机 cryptography 包有 Python 3.14 metadata 编码 bug,导致 pdfminer.six 实际 import 失败;但 book-to-skill 会 fallback 到 pypdf,不影响主流程

## 调研方法
1. `gh search repos` 多关键词扫
2. WebFetch 抓 Top 候选 README
3. 对比矩阵
4. 写报告

## 关键问题
- Anthropic 官方 skills 仓库?
- 第三方插件市场最强?
- IDE 兼容性?
- MCP 与 Skills 关系?
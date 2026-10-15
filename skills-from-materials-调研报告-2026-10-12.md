# 从书籍/资料/讲座/教程蒸馏出 Skills — 生态调研报告

> 调研日期:2026-10-12
> 调研范围:GitHub 上「把原始材料(PDF 书/讲义/视频转录/课程/内部文档)蒸馏成 Agent SKILL.md」的开源项目
> 筛选标准:主题直接命中 + 维护活跃 + 思路可借鉴
> 调研人:Claude (in `F:\soft\00selfmade\evidence-skill`)

> ⚠️ 这是对前一份 `agent-skills-调研报告-2026-10-12.md`(聚焦"安装别人写好的 skill")的**主题修正版**。
> 前一份报告保留为安装部署参考;本份专攻**「如何自己提炼 skill」**这一元能力。

---

## 一、执行摘要

1. **「材料 → Skill」不是单一技术,而是一个谱系**:从「手工摘要」到「全自动蒸馏」,到「人机协作流水线」,到「专家级采访式萃取」,选择取决于**输入质量 + 期望保真度 + 复用频率**。
2. **三大标杆项目**:
   - **`virgiliojr94/book-to-skill`**(34.6k ⭐)——**行业标准**,Python 提取器 + 规范驱动的 Skill 生成器,实测比"丢整本书进 context"省 **24-51× tokens**,跨 7 个 Agent runtime。
   - **`weitzu-com/book-to-skill`**(3 ⭐,中文"书道")——**7 阶段流水线 + 五维验证**,把"判断力"而不是"记忆"当验收标准,已用 5 本书验证。
   - **`AgriciDaniel/skill-forge`**(179 ⭐)——**最完整的 Skill 创作工作流**(plan/build/review/evolve/publish/eval/benchmark/convert 8 命令),但不是 book-specific。
3. **Anthropic 官方 `skill-creator`** 是底层方法论,但**没有材料摄取能力**——它假设你已经有结构化意图,不会帮你把 PDF 转成 skill。
4. **课程/视频 → skill** 还没出大爆款,但 `awesome-edu/video-to-skill`(4 ⭐)和 `ShazebA/course-to-skill-builder`(2 ⭐)已经探路。
5. **蒸馏出来的 Skill 是否包设计？**——决定因素是 `description` 字段的触发词命中率(渐进披露第一层)。蒸馏过程中最该优化的不是 SKILL.md 内容,而是这个 description。

---

## 二、蒸馏的本质:从「材料」到「能力」

```
原材料                  中间产物                  产物                  落地
─────────           ────────────           ─────────          ─────────
PDF/EPUB/DOCX   →   结构化 Markdown    →   SKILL.md 目录   →   Agent 工具调用
视频转录文本       JSONL 知识索引           ├─ SKILL.md
讲座讲义          多 Agent 4 层解析       ├─ chapters/
内部 Wiki          认知架构五要素          ├─ glossary.md
                                               ├─ patterns.md
                                               └─ cheatsheet.md
```

**核心公式**(从 `weitzu-com/book-to-skill` 的"第一性原理"提炼):

> **一本书的价值 = 它改变读者「如何判断与行动」的能力。**
> **因此 Skill 不以"记住了多少"为验收,以"能否在新场景做出符合原书精神的判断"为验收。**

这条原则把「蒸馏」和「摘要」根本区分开:**摘要保留原文,蒸馏提走灵魂**。

---

## 三、三大蒸馏范式

### 范式 1:抽取式蒸馏(Extract-then-Generate)

**代表**:`virgiliojr94/book-to-skill`、`zhimaAi/BookToSkill`、`muyuhill/100books-to-skills`

**核心思想**:Python/脚本先把 PDF/EPUB 转干净 Markdown + 章节结构,再让 LLM 按 SKILL.md 规范生成产物。

```
┌────────────────┐    ┌────────────────┐    ┌────────────────┐
│  PDF/EPUB/MD   │ →  │  Markdown 切分  │ →  │  LLM 按 SKILL  │
│  (200 页原文)  │    │  章节/块/批次   │    │  规范生成产物  │
└────────────────┘    └────────────────┘    └────────────────┘
   确定性,可缓存        可验证,可追溯          创造性,需评测
```

- **优点**:可重复、可批处理、可量化
- **缺点**:对扫描版 PDF / 弱结构讲义效果差;依赖 extractor 质量
- **典型产出**:`SKILL.md` (~4k tokens) + `chapters/*.md` + `glossary.md` + `patterns.md` + `cheatsheet.md`

### 范式 2:多 Agent 协作式(Distributed Parsing)

**代表**:`weitzu-com/book-to-skill`(7 阶段流水线)、`omdsh-dev/dsh-book2skill`(5 阶段 + 3 个人工 gate)、`anthropics/skills/skill-creator`(评估循环)

**核心思想**:把"蒸馏"拆成多个并行/串行子任务(预处理 / 多层解析 / 认知架构 / 编写 / 验证 / 归档),每个子任务可独立 prompt。

```
Stage 0: 预处理 + 冲突检测
Stage 1: 结构化(章节边界)
Stage 2: 多 Agent 并行 4 层解析(概念 → 框架 → 方法 → 原则)
Stage 3: 认知架构(身份 / 知识分配 / 能力边界 / 行为模式 / 交互协议)
Stage 4: SKILL.md + references 六件套
Stage 5: 五维验证(覆盖度 / 关系 / 应用 / 判断 / 边界)
Stage 6: 归档(知识图谱 + 日志 + 索引)
```

- **优点**:保真度最高,适合"硬书"(《原则》《穷查理宝典》这种需要传达"判断力"的书)
- **缺点**:慢、贵、需要人在回路(human gates)
- **典型产出**:同样格式的 SKILL.md,但内容深度显著高

### 范式 3:专家访谈式(Knowledge Extraction from Humans)

**代表**:`neosun100/expert-knowledge-extraction`(4 ⭐,中文"专家知识萃取方法论")、`bella0513/llm-wiki-extractor`(4 ⭐,Karpathy-style LLM Wiki)

**核心思想**:蒸馏的对象不是书,而是**人**——通过战略性问题(背景 → 案例 → 反思 → 启发)从行业专家口述/笔记中萃取方法论,产物是「可调用的思维框架」而非"知识摘抄"。

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│ 专家访谈记录     │ →  │ 战略性问题链     │ →  │ SKILL.md        │
│ 邮件 / 笔记     │    │ (4 层递进)      │    │ (思维框架)      │
│ 演讲录音转写    │    │                 │    │                 │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

- **优点**:能保留**作者人格**(「不是 AI 读了一本书,是那本书的作者附身 AI 帮你出主意」)
- **缺点**:极度依赖专家配合度,无法纯自动化
- **典型产出**:`yysws5566/book-to-skill` 这类「以书铸魂」skill,模拟作者人格

---

## 四、Top 工具对比矩阵

| 项目 | ⭐ | 类型 | 输入 | 范式 | 验证机制 | 强项 | 短板 |
|---|---|---|---|---|---|---|---|
| **[virgiliojr94/book-to-skill](https://github.com/virgiliojr94/book-to-skill)** | 34.6k | 工具 + Skill | PDF/EPUB/DOCX/MD/HTML/RTF/MOBI/文件夹 | 范式 1 | 实测 24-51× token 节省 | **跨 7 个 runtime**,最广;Python 提取器成熟 | 扫描版 PDF 弱;无内置评测 |
| **[weitzu-com/book-to-skill](https://github.com/weitzu-com/book-to-skill)** | 3 | 方法论流水线 | PDF/扫描 PDF/EPUB/MD/TXT | 范式 2 | **5 维验证**(覆盖/关系/应用/判断/边界) | **「判断力而非记忆」哲学**;7 阶段清晰 | 速度慢;3 个 human gate 需人工 |
| **[AgriciDaniel/skill-forge](https://github.com/AgriciDaniel/skill-forge)** | 179 | Skill 创作工作流 | 任意(无 extractor) | 范式 2 | 0-100 健康分 / benchmark variance | **最完整的 Skill 创作 CLI**:8 命令、4 复杂度 tier、benchmark、跨平台 convert | 无材料提取能力 |
| **[muyuhill/100books-to-skills](https://github.com/muyuhill/100books-to-skills)** | 14 | **30 个现成 Skill** | 商业/成长书籍 | 范式 1 手工版 | 框架可用性 | **直接复用**——已经有 30 个体现成的 Skill | 不是工具,是作品集 |
| **[zhimaAi/BookToSkill](https://github.com/zhimaAi/BookToSkill)** | 32 | 工具 + Skill 包 | TXT/MD/DOCX/PDF | 范式 1 增强 | JSONL 知识索引 + 源单元 ID 可追溯 | **工业级可追溯**;有界批次协议 | 仅 Python CLI;输出格式较固定 |
| **[anthropics/skills/skill-creator](https://github.com/anthropics/skills/tree/main/skills/skill-creator)** | 内嵌于 180k | Skill 创作元 skill | 任意(假设你已有意图) | 范式 2 评测循环 | **eval + benchmark 自动化** | **官方方法论**;描述优化器 | 无 extractor;无 PDF 处理 |
| **[yysws5566/book-to-skill](https://github.com/yysws5566/book-to-skill)** | 4 | Skill 蒸馏器 | PDF/EPUB/TXT/书名 | 范式 3 | (无明确验证) | **「以书铸魂」**——保留作者人格 | 哲学性强,工程化弱 |
| **[apple-ouyang/book-to-skill](https://github.com/apple-ouyang/book-to-skill)** | 162 | 中文 Fork | 同上 | 范式 1 | 实测 24-51× | **中文 README 友好**;决策副驾驶定位 | 与上游差异较小 |
| **[omdsh-dev/dsh-book2skill](https://github.com/omdsh-dev/dsh-book2skill)** | 3 | 工具 + 可视化 | 任意 | 范式 2 | 3 个人工 gate + 浏览器时间线 | **可视化进度面板** | 新项目,文档不全 |
| **[runyuan-wang/book-to-skill-distillation](https://github.com/runyuan-wang/book-to-skill-distillation)** | 10 | Agent Skill | 长 PDF | 范式 1 | (待确认) | 直接作为 Skill 安装 | 工具规模小 |
| **[47thtechcorner/RayCodes_BookToSkill](https://github.com/47thtechcorner/RayCodes_BookToSkill)** | 9 | 本地工具 | PDF | 范式 1 + Ollama | **完全本地** | **零 token 成本**;隐私好 | 质量取决于本地模型 |
| **[ariel-lee-1023/Books-to-Skill-Refs](https://github.com/ariel-lee-1023/Books-to-Skill-Refs)** | 2 | 多书蒸馏 | 多本 PDF | 范式 2 | **保留分歧** | 多视角、保留作者分歧 | 工程化早期 |

### 课程/视频专题

| 项目 | ⭐ | 输入 | 强项 |
|---|---|---|---|
| [awesome-edu/video-to-skill](https://github.com/awesome-edu/video-to-skill) | 4 | 视频 | "book-to-skill 的视频 cousin" |
| [ShazebA/course-to-skill-builder](https://github.com/ShazebA/course-to-skill-builder) | 2 | MP4 课程 | 本地 CLI;结构化输出 |
| [gkpacker/course-to-skill](https://github.com/gkpacker/course-to-skill) | 0 | 课程材料 | **可审计、可安装的 Codex skill** |
| [tubeagentkit/youtube-transcript-skills](https://github.com/tubeagentkit/youtube-transcript-skills) | 80 | YouTube 视频 | 转录 + 搜索 + 频道浏览 |

### 知识萃取 / Wiki 风格

| 项目 | ⭐ | 定位 |
|---|---|---|
| [neosun100/expert-knowledge-extraction](https://github.com/neosun100/expert-knowledge-extraction) | 4 | 专家访谈式方法论(中文) |
| [bella0513/llm-wiki-extractor](https://github.com/bella0513/llm-wiki-extractor) | 4 | Karpathy-style LLM Wiki |
| [tbgallinucci/alexandria](https://github.com/tbgallinucci/alexandria) | 0 | Self-hostable AI chat + PDF + SKILL(RAG) |

---

## 五、Top 5 深度解析

### 1. `virgiliojr94/book-to-skill`(34.6k ⭐)— **行业事实标准**

- **架构**:**Python 确定性提取器**(document → clean text + metadata)+ **规范驱动的生成器**(agent 按 SKILL.md 把素材变成结构化 skill)
- **支持的输入**:PDF / EPUB / DOCX / MD / HTML / RTF / MOBI / 整个文件夹 / glob
- **跨 Agent**:`~/.agents/skills/<slug>/` 是公共目录,被 **Copilot CLI、Amp、Codex、OpenCode、OpenClaw** 原生读取;Claude Code 走 `~/.claude/skills/<slug>/` 软链接;Hermes Agent 走 `/skills/<category>/<slug>/`
- **产物清单**:
  ```
  SKILL.md           # 核心心智模型 + 章节索引,~4000 tokens
  chapters/ch01.md   # 每章一个,按需加载,~1000 tokens/章
  glossary.md        # 全部术语,按字母排序,~1500 tokens
  patterns.md        # 全部技术/算法/设计模式,~2000 tokens
  cheatsheet.md      # 决策表 + 快查规则,~1000 tokens
  ```
- **核心数据**:**比"丢整本书进 context"省 24-51× tokens**(在真实书籍上原话是"the discovery loop tax")
- **谁适合**:技术书/手册/ADR/品牌指南/合规文档,**复用频率高**的所有结构化散文
- **不适合**:扫描版 PDF(需预处理 OCR)、非结构化访谈记录(走范式 3)

### 2. `weitzu-com/book-to-skill`(3 ⭐,「书道」)— **哲学最完整**

- **哲学**:**「以用为验」**——Skill 验收标准是"在新场景能否做出符合原书精神的判断",**不是"记住了多少"**
- **7 阶段流水线**:
  - Stage 0 预处理 + 冲突检测(先判断,再动手)
  - Stage 1 结构化(章节边界 + 全角→半角)
  - Stage 2 多 Agent 并行 4 层解析(概念 → 框架 → 方法 → 原则)
  - Stage 3 认知架构 5 要素(身份 / 知识分配 / 能力边界 / 行为模式 / 交互协议)
  - Stage 4 SKILL.md(<500 行)+ references 六件套
  - Stage 5 **5 维验证**(覆盖度 / 关系 / 应用 / 判断 / 边界)
  - Stage 6 归档闭环(知识图谱 + 工作日志 + 索引)
- **三省六部治理**:用中国古代政府架构比喻 stage 间的协作(可读性强)
- **已验证**:5 本真实书籍
- **谁适合**:**硬书**——《原则》《穷查理宝典》《影响力》这种需要传达判断力的
- **短板**:每个 stage 都要人工 gate,速度慢、贵

### 3. `AgriciDaniel/skill-forge`(179 ⭐)— **Skill 创作 CLI 之最**

- **不是 book-specific**,但是**最完整的 Skill 创作 CLI**
- **8 个命令**:`plan / build / repalyze / evolve / publish / eval / benchmark / convert`
- **4 复杂度 tier**:
  | Tier | 结构 | 适用 |
  |---|---|---|
  | 1 | 单 SKILL.md | 简单工作流、文档生成 |
  | 2 | SKILL.md + scripts | 需确定性验证的任务 |
  | 3 | 多 Skill(orchestrator + sub-skills) | 多工作流复杂领域 |
  | 4 | 完整系统(带 agents) | 企业级并行分析 |
- **评测**:0-100 健康分(6 个质量维度)、benchmark variance、threshold gating
- **跨平台 convert**:Codex / Gemini CLI / Antigravity / Cursor
- **谁适合**:**已经决定要建 Skill 但想做得工程化**的团队
- **短板**:没有内置 PDF extractor——它假设你已经把材料预处理成了 Markdown/代码

### 4. `muyuhill/100books-to-skills`(14 ⭐)— **30 个现成中文 Skill 样本库**

- **独特价值**:**不是工具,是作品集**——已经把 30 本商业/成长类好书蒸馏成了 Skill,每个都是"可调用的思维框架"
- **示例 Skill**:
  - 《原子习惯》4 步法 + 身份驱动 → 「每天写作的系统」
  - 《影响力》6 大原则 → 「为什么没人看我的文章」
  - 《原则》5 步流程 + 极度求真透明 → 「决策系统」
  - 《GTD》5 步工作流 → 「清空大脑」
  - 《深度工作》4 种哲学 → 「分心时代夺回专注」
- **使用方式**:把 SKILL.md 内容粘贴到对话中,说「以后遇到相关问题,请按这个框架分析」
- **谁适合**:**想直接用,不想造轮子**的运营/销售/产品
- **短板**:手工蒸馏,没有自动化 pipeline;30 个都聚焦商业/成长,**技术类缺乏**

---

## 六、不同输入类型的最优工具选择

```
你的输入是什么？
├── 一本书 (PDF/EPUB)
│   ├── 复用频率高 + 技术向 → virgiliojr94/book-to-skill
│   ├── 「硬书」/判断力向 → weitzu-com/book-to-skill
│   └── 多本书、保留分歧 → ariel-lee-1023/Books-to-Skill-Refs
│
├── 一门课程 (视频 + 讲义)
│   ├── MP4 → SKILL.md → ShazebA/course-to-skill-builder
│   ├── 已有材料 → gkpacker/course-to-skill(可审计 Codex skill)
│   └── YouTube 课程 → tubeagentkit/youtube-transcript-skills
│
├── 视频/讲座(单条)
│   └── awesome-edu/video-to-skill
│
├── 内部文档(ADR/runbook/wiki)
│   └── 直接用 virgiliojr94/book-to-skill 的"非书场景"模式
│
├── 专家访谈/邮件/笔记
│   └── neosun100/expert-knowledge-extraction(范式 3)
│
└── 已有意图(不需要材料提取)
    ├── 想要工程化工作流 → AgriciDaniel/skill-forge
    ├── 想要官方方法论 → anthropics/skills/skill-creator
    └── 想要中文化 → pingguoge001-coder/skill-creator-cn
```

---

## 七、自建蒸馏管道的 SOP(7 步)

如果你想**自己造一个 book-to-skill 工具**(而不是装现成的),按下面走:

### Step 1:选输入格式

- PDF/EPUB → `pymupdf` + `pdfplumber`(前者扫排版,后者抽表格)
- DOCX → `python-docx`
- 扫描版 → `paddleocr` 或 `docling`(IBM)
- 视频 → `yt-dlp` + `whisper.cpp`(本地转写)
- HTML/MD → 直接读

### Step 2:结构化

- 按章节切分,**每块 ≤ 5000 Unicode 字符**(参照 `zhimaAi/BookToSkill` 的有界协议)
- 保留原标题层级,**全角转半角**(参照 `weitzu-com/book-to-skill` 的预处理)
- 生成**目录树 JSON**,便于后续按需加载

### Step 3:多 Agent 4 层解析

并行 4 个 subagent:
- **概念层**:术语、定义、人物、案例(产出 glossary.md)
- **框架层**:模型、矩阵、流程图、决策树(产出 patterns.md)
- **方法层**:步骤、清单、模板、checklist(产出 cheatsheet.md)
- **原则层**:作者反复强调的判断准则(进入 SKILL.md body)

### Step 4:写 SKILL.md

按 [agentskills.io](https://agentskills.io/specification) 规范:
```yaml
---
name: book-slug-or-domain
description: |
  Use this skill when the user mentions [book keywords] or asks about
  [domain questions]. [Action verb 1], [action verb 2], [action verb 3].
  Trigger keywords: keyword1, keyword2, keyword3.
---

# [Book Title] - Distilled Judgment

## When to use
- Trigger scenarios...

## Core mental models
[From 原则层]

## Decision frameworks
[From 框架层]

## Concrete patterns
[From 方法层]

## Common anti-patterns
[What the book explicitly warns against]

## Reference
- chapters/ — loaded on demand
- glossary.md
- patterns.md
- cheatsheet.md
```

**关键**:`description` 是渐进披露第一层,决定 Skill 能否被触发。**写 trigger keywords 时要"pushy"**(参照 `anthropics/skills/skill-creator` 的"undertriggering"警告)。

### Step 5:验证(最容易被跳过)

按 `weitzu-com/book-to-skill` 的 **5 维验证**走:
1. **覆盖度**:书的 10 个核心论点,Skill 是否都体现了?
2. **关系**:作者对 X 与 Y 关系的判断,Skill 是否保留?
3. **应用**:给一个**新场景**(原书未涉及的),Skill 能给出符合原书精神的建议吗?
4. **判断**:面对道德/价值两难,Skill 能否给出一致的判断框架?
5. **边界**:作者明确说"不适用"的场景,Skill 是否也写明了?

### Step 6:写评测 script(可选)

按 `anthropics/skills/skill-creator` 的 eval 模板:
```python
# eval-viewer/generate_review.py
# 准备 5-10 个测试 prompt
prompts = [
    "用 X 框架帮我分析这个问题:...",
    "如果我面临 Y 决策,你会怎么建议?",
    ...
]
# 对每个 prompt,跑 2-3 次,统计一致性
```

### Step 7:持续更新

- 用 `weitzu-com` 的 Stage 6 归档闭环:每次蒸馏记录、存日志、记索引
- 用 `zhimaAi/BookToSkill` 的 JSONL 索引:新发现的概念可以**追加到现有 Skill**,而不是重蒸馏

---

## 八、常见坑与风险

| 坑 | 说明 | 规避 |
|---|---|---|
| **「蒸馏 = 摘要」** | 摘要保留原文,蒸馏要提走灵魂 | 用 `weitzu-com` 的"以用为验"标准 |
| **`description` 写得太短** | 渐进披露第一层 100 tokens,描述不"pushy"就被跳过 | 按 `skill-creator` 建议,关键词要丰富 |
| **每章都当文件** | 章节数 > 50 时,目录本身就成了 context 负担 | `virgiliojr94` 的 SKILL.md 只放心智模型+索引,详细按需 |
| **扫描版 PDF 直跑** | `pymupdf` 抽不出文字 | 先 OCR(`paddleocr`/`docling`),再切结构 |
| **没有源追溯** | 蒸馏结果错时,不知道出自哪章 | 用 `zhimaAi` 的 JSONL 源单元 ID,或保留 `chapters/*.md` 原文 |
| **多书观点冲突** | A 书说 X,B 书说 ~X,Skill 不交代 | 用 `ariel-lee-1023` 的"保留分歧"模式 |
| **跳过评测** | 写完 SKILL.md 立刻用,幻觉率高 | 至少跑 5 个 test prompt,看一致性 |
| **Agent runtime 不兼容** | `SKILL.md` 在 Claude Code 跑得通,在 Cursor 不一定 | 用 `skill-forge convert` 跨平台编译 |

---

## 九、选型决策表(快查)

### 我想...

| 需求 | 推荐 |
|---|---|
| **技术书/手册快速跑通** | `virgiliojr94/book-to-skill` ⭐首选 |
| **传达作者「判断力」** | `weitzu-com/book-to-skill`(书道) |
| **已经知道要建什么 Skill,要工程化** | `AgriciDaniel/skill-forge` |
| **直接复用现成的商业/成长类 Skill** | `muyuhill/100books-to-skills`(30 个现成) |
| **中文 PDF + 工业级可追溯** | `zhimaAi/BookToSkill` |
| **完全本地/隐私优先** | `47thtechcorner/RayCodes_BookToSkill`(Ollama) |
| **YouTube 课程** | `tubeagentkit/youtube-transcript-skills`(80 ⭐) |
| **MP4 → SKILL.md** | `ShazebA/course-to-skill-builder` |
| **多本书视角融合** | `ariel-lee-1023/Books-to-Skill-Refs` |
| **从专家访谈提炼** | `neosun100/expert-knowledge-extraction` |
| **Karpathy LLM Wiki 风格** | `bella0513/llm-wiki-extractor` |
| **完整 Skill 创作方法论** | `anthropics/skills/skill-creator`(官方元 Skill) |
| **官方方法论的中文化** | `pingguoge001-coder/skill-creator-cn` |
| **可视化进度面板** | `omdsh-dev/dsh-book2skill` |
| **模拟作者人格** | `yysws5566/book-to-skill`(以书铸魂) |

---

## 十、参考资源

- **SKILL.md 规范**:`https://agentskills.io/specification`
- **Anthropic 官方 skill-creator**:`https://github.com/anthropics/skills/tree/main/skills/skill-creator`
- **跨 Agent 目录标准**:`~/.agents/skills/<slug>/`(virgiliojr94 推动的开放约定)
- **视频蒸馏 cousin**:`https://github.com/awesome-edu/video-to-skill`
- **评测方法论**:skill-creator 内的 `eval-viewer/`
- **中文 skill 创作本地化**:`https://github.com/pingguoge001-coder/skill-creator-cn`

---

## 结论

蒸馏 skill 的能力,**比"装别人 skill"重要 10 倍**。原因:

1. **复利**——蒸馏一次,Skill 可被 7 个 runtime 任意调用,被无数次会话使用
2. **稀缺**——市场上高质量 Skill 仍稀缺,但你团队/你个人的内部知识是稀缺的
3. **可演化**——Skill 的最大价值不是"覆盖了原书",而是"基于原书生成新判断";每次调用都是一次小型蒸馏

**起步建议**(按 7-2-1 路径):

1. **先用** `virgiliojr94/book-to-skill` 跑通一本技术书(1 小时)
2. **再读** `weitzu-com/book-to-skill` 的"以用为验"哲学,理解蒸馏 vs 摘要的本质区别
3. **最后** `anthropics/skills/skill-creator` 的 eval/benchmark 模板,给你的 Skill 加评测循环

**最重要的:**无论用哪个工具,**写 SKILL.md 的 `description` 时要"pushy"**(参照 Anthropic 官方 skill-creator 关于 undertriggering 的警告)——这是 Skill 能否被触发的最大瓶颈。
# Claude Code Skills 生态调研报告

> 调研日期:2026-10-12
> 调研范围:GitHub 上「Claude Code Skills / Agent Skills / Plugin Marketplace」相关开源项目
> 筛选标准:星标 ≥ 1k + 与 SKILL.md 规范或 Claude Code Plugin 体系相关 + 维护活跃
> 调研人:Claude (in `F:\soft\00selfmade\evidence-skill`)

---

## 一、执行摘要(Executive Summary)

1. **生态已成型**:Claude Skills 不再是孤岛。Anthropic 官方有 [anthropics/skills](https://github.com/anthropics/skills)(180k+ ⭐,19 个官方 skill) 和 [anthropics/claude-plugins-official](https://github.com/anthropics/claude-plugins-official)(37k+ ⭐),并联合发布了统一的 `SKILL.md` 规范(见 [agentskills.io/specification](https://agentskills.io/specification))。
2. **「技能 / 插件 / 命令 / 钩子 / MCP」五件套**已经成为主流 Agent Runtime 的标准扩展模型:Claude Code / Codex / Gemini CLI / Cursor / OpenCode / Aider / Windsurf 等 13+ Agent 都已支持这套规范。
3. **三大选型梯队**(按星数 + 维护活跃度):
   - **官方**:`anthropics/skills`(标杆)+ `anthropics/claude-plugins-official`(官方插件市场)
   - **框架型**:`obra/superpowers`(298k ⭐,方法论+技能)和 `affaan-m/ECC`(277k ⭐,harness 调优系统)
   - **大而全**:`mattpocock/skills`(287k ⭐,工程实战)、`alirezarezvani/claude-skills`(28k ⭐,388 个 skill 跨 13 个 IDE)
4. **发现入口**:[davepoon/buildwithclaude](https://github.com/davepoon/buildwithclaude) 是当下最完整的 plugin 发现平台(51 plugins + 117 agents + 175 commands + 28 hooks + 26 skills),同时收录 20k+ 社区插件。
5. **MCP ≠ Skills,但协同**:MCP(Model Context Protocol)给 agent 接**外部工具**(DB / 浏览器 / GitHub API),Skills 教 agent **怎么做任务**。两者通过 Plugin 一起打包。

---

## 二、SKILL.md 规范速览

[agentskills.io](https://agentskills.io/) 是 Anthropic 联合社区发布的开放规范,核心是 **一个目录 = 一个 Skill**:

```
my-skill/
├── SKILL.md          # 必填:YAML frontmatter + Markdown 指令
├── scripts/          # 可选:Python/Bash/JS 可执行脚本
├── references/       # 可选:深度文档(REFERENCE.md / FORMS.md)
├── assets/           # 可选:模板、图片、数据
└── ...
```

### YAML 必填字段

| 字段 | 必填 | 限制 |
|---|---|---|
| `name` | ✅ | ≤64 字符,小写字母/数字/连字符,不能以 `-` 开头或结尾,不能 `--`,**必须等于父目录名** |
| `description` | ✅ | ≤1024 字符,**描述「做什么 + 何时触发」**(触发关键词要明确,否则 Agent 不调用) |
| `license` | ❌ | 推荐 Apache 2.0 / MIT |
| `compatibility` | ❌ | ≤500 字符,声明依赖环境 |
| `metadata` | ❌ | 自由 key-value,如 `author`、`version` |
| `allowed-tools` | ❌ | 预批准工具列表(实验性) |

### 渐进披露(Progressive Disclosure)

1. **元数据层**(~100 tokens):启动时所有 skill 的 `name` + `description` 被扫描
2. **指令层**(<5000 tokens):当任务命中时,完整 `SKILL.md` body 被加载
3. **资源层**:scripts / references / assets 按需读取

**关键原则**:`SKILL.md` 主体 ≤ 500 行,深内容挪到 `references/` 下,避免烧 context。

---

## 三、Top 项目深度对比矩阵

| 项目 | ⭐ | License | 最近更新 | 类型 | 核心定位 | 安装命令 |
|---|---|---|---|---|---|---|
| [obra/superpowers](https://github.com/obra/superpowers) | 298k | MIT | 2026-10-10 | 框架 + 方法论 | **Agentic 软件开发方法论**,把 brainstorming → TDD → subagent 串成 SOP,15+ runtime 装 | `/plugin install superpowers@claude-plugins-official` |
| [mattpocock/skills](https://github.com/mattpocock/skills) | 287k | MIT | 2026-10-12 | 工程技能集合 | "**Skills for Real Engineers, not vibe coding**",小而精、可改可组合 | `claude plugin install mattpocock-skills@claude-plugins-official` |
| [affaan-m/everything-claude-code](https://github.com/affaan-m/everything-claude-code) | 277k | MIT | 2026-10-12 | Harness 调优系统 | **Skills + Instincts + Memory + Security + 性能优化** 五合一 | `/plugin marketplace add affaan-m/everything-claude-code` |
| [anthropics/skills](https://github.com/anthropics/skills) | 180k | Apache 2.0(mixed) | 2026-10-09 | **官方** | 19 个示例 Skill(Docx/PDF/PPTX/Xlsx + Design + 沟通 + skill-creator) | `/plugin marketplace add anthropics/skills` |
| [ComposioHQ/awesome-claude-skills](https://github.com/ComposioHQ/awesome-claude-skills) | 76k | Apache 2.0 | 2026-09-18 | Awesome list + 商业 | **1000+ skill 策展 + Composio MCP Gateway**(1000+ SaaS 集成) | `claude --plugin-dir ./connect-apps-plugin` |
| [anthropics/claude-plugins-official](https://github.com/anthropics/claude-plugins-official) | 37k | Apache 2.0 | 2026-10-12 | **官方插件市场** | 内部插件 + 第三方收录,plugin 目录元数据 | `/plugin install <name>@claude-plugins-official` |
| [alirezarezvani/claude-skills](https://github.com/alirezarezvani/claude-skills) | 28k | MIT | 2026-08-30 | **Skill 大仓库** | 388 个 Skill + 118 Agent + 150 命令,跨 13 IDE | `/plugin marketplace add alirezarezvani/claude-skills` |
| [OthmanAdi/planning-with-files](https://github.com/OthmanAdi/planning-with-files) | 27k | MIT | 2026-10-12 | 单 Skill 杀手锏 | **持久化文件规划**,抗 context 压缩/崩溃,长任务首选 | `npx skills` 或 `/plugin marketplace add OthmanAdi/planning-with-files` |
| [travisvn/awesome-claude-skills](https://github.com/travisvn/awesome-claude-skills) | 15k | 无 | 2026-04-28 | Awesome list | 策展 + 解释 Skill 工作原理 | (浏览) |
| [Jeffallan/claude-skills](https://github.com/Jeffallan/claude-skills) | 11k | MIT | 2026-10-03 | 67 个全栈 Skill | 后端/数据库/移动/前端,端到端开发技能包 | `/plugin marketplace add Jeffallan/claude-skills` |
| [davepoon/buildwithclaude](https://github.com/davepoon/buildwithclaude) | 3.6k | MIT | 2026-10-12 | **发现平台** | 51 plugins + 117 agents + 175 commands + 28 hooks + 26 skills,Web UI 索引 | `/plugin marketplace add davepoon/buildwithclaude` |
| [obra/superpowers-marketplace](https://github.com/obra/superpowers-marketplace) | 1.3k | MIT | — | 插件市场 | obra 自己策展的市场,只装 superpowers 干净 | `/plugin marketplace add obra/superpowers-marketplace` |

> 表格排序按星数降序。星数为 2026-10-12 当日 `gh repo view` 实测,可能略高于 GitHub 列表显示。

---

## 四、Top 5 深度解析

### 1. obra/superpowers(298k ⭐)— **方法论框架** ⭐推荐起点

- **定位**:不只是 skill 集合,而是一套**完整的 agentic 软件开发方法论**——brainstorming → spec → TDD → subagent-driven-development → review → 收尾。
- **配套 Skill**:`superpowers:brainstorming`(对话澄清需求)、`superpowers:writing-plans`(写实施计划)、`superpowers:test-driven-development`(TDD 守门)、`superpowers:systematic-debugging`(5 步排错)、`superpowers:verification-before-completion`(完成前证据自查)、`superpowers:dispatching-parallel-agents`(并行 fan-out)。
- **支持 runtime**:Claude Code / Codex / Antigravity / Cursor / Devin CLI / Factory Droid / Gemini CLI / GitHub Copilot CLI / Grok / Kimi / OpenCode / Pi / Qwen Code / Hermes / Muse——**15+ 种**,最广。
- **适合谁**:团队要建立标准化 agentic 开发 SOP 的场景,任何复杂任务都可以从 `brainstorming` skill 触发。
- **风险**:方法论很重,简单问答会被强制走 brainstorming → 可能过度消耗 token。

### 2. anthropics/skills(180k ⭐)— **官方规范 + 19 个示例** ⭐必备基线

- **官方 19 个 Skill**:
  - 文档:`docx`、`pdf`、`pptx`、`xlsx`(支撑 Claude 文档创建能力,source-available)
  - 设计:`brand-guidelines`、`canvas-design`、`theme-factory`、`frontend-design`、`algorithmic-art`、`web-artifacts-builder`
  - 沟通:`internal-comms`、`slack-gif-creator`
  - 元:`skill-creator`(教你怎么写自己的 skill)、`mcp-builder`(教你怎么写 MCP 服务器)
  - 流程:`doc-coauthoring`、`webapp-testing`、`academy-guide`、`claude-api`、`discernment-nudge`
- **价值**:每个 skill 都是规范参考实现,直接读 `SKILL.md` 学设计模式。
- **安装**:通过 `/plugin marketplace add anthropics/skills`,然后在 Claude Code 里选 `document-skills` 或 `example-skills`。

### 3. mattpocock/skills(287k ⭐)— **工程实战**

- **作者**:Matt Pocock(Total TypeScript 创始人,TypeScript 领域知名布道师)。
- **哲学**:**反对 "GSD / BMAD / Spec-Kit 那种全包方法论"**,主张小、可改、可组合、模型无关。
- **技能**:每条都是从作者实际 `.agents/` 提炼,工程日常高频。
- **安装**:Claude Code 用 `claude plugin install mattpocock-skills@claude-plugins-official`;Codex/Copilot/Gemini 各自有命令。

### 4. alirezarezvani/claude-skills(28k ⭐)— **388 个 Skill 跨 13 IDE**

- **量级**:388 skills + 118 agents + 150 slash commands + 7 personas,**跨 Claude Code / Codex / Gemini CLI / Hermes / Mistral Vibe / Cursor / Aider / Windsurf / Kilo / OpenCode / Augment / Antigravity / OpenClaw 13 个平台**。
- **覆盖领域**:工程、营销(含 AEO)、合规、C 级咨询(CFO/CMO/CRO/CPO 等 21 个 `/cs:*` 命令)、生产力(weekly-review/deep-work)、学术研究(litreview/grants)、企业研究运营。
- **亮点**:706 个 Python 工具脚本,全部 stdlib-only,零 pip 依赖。
- **⚠️ Windows 坑**:必须 `git clone -c core.symlinks=true`,且设 `PYTHONUTF8=1`,否则 `.gemini/`/`.codex/` 等变 1 行指针文件。

### 5. davepoon/buildwithclaude(3.6k ⭐)— **发现平台**

- **不是 skill 集合,而是发现入口**:51 plugins + 117 agents + 175 commands + 28 hooks + 26 skills 自有,**外加索引 20k+ 社区插件 + 4,500+ MCP server + 1,100+ marketplace**。
- **Web UI**:`buildwithclaude.com` 可视化浏览。
- **适合**:不知道装啥、想先看看生态长啥样的场景。

---

## 五、MCP 与 Skills 的关系

```
┌─────────────────────┐
│    Plugin           │  ← 安装单元(/plugin install xxx)
├─────────────────────┤
│  Skills  │ Commands │  Hooks + Agents
│ (怎么做) │ (做什么) │ (什么时候触发)
└────┬────────┬───────┘
     │        │
     │        ▼
     │   ┌─────────┐
     │   │   MCP   │ ← 外部工具(GitHub / DB / Browser)
     │   └─────────┘
     ▼
   Agent(Runtime):Claude Code / Codex / Cursor / Gemini CLI ...
```

- **Skills**:教 agent **如何做某类任务**(例如 "怎么写 PRD"、"怎么审 PR"),文件型、零网络。
- **Commands**:用户主动触发的 slash 命令(例如 `/commit`),行为定义。
- **Hooks**:生命周期事件(`PreToolUse`、`PostToolUse`),用于自动通知/格式化/安全检查。
- **MCP Servers**:给 agent 接外部工具的协议,运行时占用网络/进程。
- **Agents**:Subagent 定义(如 `superpowers:code-reviewer`),独立上下文跑独立任务。

一个 Plugin 可以打包任意组合,典型如:anthropics/claude-plugins-official 收录的 plugin 同时带 skills + commands + hooks + MCP。

---

## 六、选型建议(团队)

### 场景 A:个人开发者,先入坑

**装这三个就够了**:
1. **obra/superpowers**——拿到完整的 agentic 工作流(brainstorming → TDD → subagent)
2. **anthropics/skills**——拿到官方规范参考
3. **OthmanAdi/planning-with-files**——长任务不丢上下文

### 场景 B:研发团队标准化

1. **obra/superpowers**——强制 TDD / brainstorming / verification SOP
2. **mattpocock/skills**——日常工程实战技能(TypeScript / 测试 / code review)
3. **自定义内部 skill 仓库**——用 `skill-creator` 模式写团队专属 SOP,放进内部 git,通过 `obra/superpowers-marketplace` 类似的机制发布

### 场景 C:非工程岗(产品 / 运营 / 销售)

1. **alirezarezvani/claude-skills**——388 个跨域 skill,营销 / 销售 / 财务 / 法务 / 学术覆盖最广
2. **ComposioHQ/awesome-claude-skills** + Composio MCP——1000+ SaaS 集成,可发邮件、推 Slack、建工单

### 场景 D:安全 / 渗透 / 红队

1. **trailofbits/skills**(7.5k ⭐)——Trail of Bits 出品,安全研究方法论
2. **elementalsouls/Claude-BugHunter**(5k ⭐)——bug hunting + 红队
3. **P4nda0s/reverse-skills**(2.2k ⭐)——逆向工程
4. **SnailSploit/Claude-Red**(7.4k ⭐)——offensive security skills

> ⚠️ **风险提醒**:`affaan-m/ECC` 在 README 顶部明确警告:「只从官方渠道装 ECC,第三方 mirror 可能含 malware」——这反映出 Skills/Plugin 生态目前**没有统一的签名/审计机制**,装第三方仓库前务必看 commit、读 SKILL.md、谨慎授予 `Bash(*)` 权限。

---

## 七、安装部署 SOP

### 7.1 通用前置

- 已装 Claude Code 最新版(≥1.0)或其他 Agent Runtime(Codex / Gemini CLI / Cursor 等)
- Node.js 18+(部分 marketplace 用 `npx skills` 安装)
- Git(用于 `marketplace add` 协议)

### 7.2 安装 superpowers(最快见效)

**Claude Code**(3 行):
```bash
# 1. 加官方 marketplace
/plugin marketplace add anthropics/claude-plugins-official

# 2. 装 superpowers
/plugin install superpowers@claude-plugins-official

# 3. 重启 Claude Code,下次会话会自动加载 using-superpowers
```

**Antigravity**:
```bash
agy plugin install https://github.com/obra/superpowers
```

### 7.3 安装 anthropics/skills 官方示例

```bash
/plugin marketplace add anthropics/skills
/plugin install document-skills@anthropic-agent-skills
/plugin install example-skills@anthropic-agent-skills
```

之后在 Claude Code 里说 "用 PDF skill 提取 path/to/file.pdf 的表单字段",即触发。

### 7.4 安装 alirezarezvani 跨 IDE 大套餐(Windows 注意)

```powershell
# 必须开 Developer Mode + 用 symlinks 模式 clone
git clone -c core.symlinks=true https://github.com/alirezarezvani/claude-skills.git
cd claude-skills
$env:PYTHONUTF8=1  # 防止 Unicode 报错

# Claude Code 路径
/plugin marketplace add alirezarezvani/claude-skills
/plugin install engineering-skills@claude-code-skills
```

按域选装:`engineering-skills` / `marketing-skills` / `c-level-skills` / `finance-skills` 等,不要全装。

### 7.5 写自己的 Skill(最小可工作)

```bash
mkdir my-skill && cd my-skill
```

写 `SKILL.md`:
```markdown
---
name: my-skill
description: |
  A clear description of what this skill does AND when to use it.
  Include trigger keywords: "PDF", "form", "extract", etc.
---

# My Skill

## Step 1
[指令]

## Examples
- Input: ...
- Output: ...
```

丢到任意目录,然后用 `marketplace.json` 注册(参考 anthropics/claude-plugins-official 的格式),或本地用:
```bash
claude --plugin-dir /path/to/parent-dir
```

### 7.6 验证 Skill 合法

```bash
# agentskills/agentskills 仓库提供 skills-ref CLI
npx skills-ref validate ./my-skill
```

---

## 八、风险与坑

| 风险 | 说明 | 规避 |
|---|---|---|
| **第三方插件供应链** | Plugin 可包含任意 MCP server / Bash 命令,`affaan-m/ECC` 已警告 mirror 含 malware | 只从官方渠道装,审 SKILL.md + commit history,禁 `Bash(*)` |
| **Token 消耗** | Skill 描述 `description` 越长,启动扫描耗 token 越多 | `description` ≤200 字符聚焦,触发词明确 |
| **过度方法论** | superpowers 在简单任务也会强制 brainstorming | 简单问答直接回,只在新功能 / bug fix 走 SOP |
| **Context 漂移** | 长任务中 agent 忘了早期约束 | 用 `planning-with-files` 做外部化持久计划 |
| **MCP 与 Skill 角色混淆** | 试图用 Skill 写 MCP 集成 | 工具调用走 MCP server,流程/规范走 Skill |
| **Plugin 名不可变** | anthropics/claude-plugins-official 明确:`name` 一旦发布不可改,改名会破坏已安装用户 | 内部 plugin 用 `displayName` 改 UI 标签,`renames` 映射做兼容 |
| **Windows symlink 坑** | alirezarezvani 大仓跨 IDE 镜像靠 symlink,Windows 默认关 | `git clone -c core.symlinks=true` + Developer Mode |

---

## 九、附录:候选清单(按星数降序)

> 共扫描 100+ 个项目,以下保留 ≥1k ⭐ 且主题直接命中者(节选)。

| ⭐ | 项目 | 一句话 |
|---|---|---|
| 298k | obra/superpowers | Agentic 开发方法论 + 框架 |
| 287k | mattpocock/skills | 真实工程师的 skills |
| 277k | affaan-m/ECC | Harness 调优系统(性能+安全+memory) |
| 180k | anthropics/skills | **Anthropic 官方 skills 仓库** |
| 96k | punkpeye/awesome-mcp-servers | MCP 服务器策展 |
| 87k | virgiliojr94/book-to-skill | 把 PDF 书转成 skill |
| 76k | ComposioHQ/awesome-claude-skills | 1000+ skills + Composio MCP |
| 61k | DietrichGebert/ponytail | "最懒高级工程师"风格 |
| 41k | Shubhamsaboo/awesome-llm-apps | 100+ AI Agents + RAG |
| 37k | anthropics/claude-plugins-official | **官方 plugin 目录** |
| 36k | morluto/rea | Reverse engineer anything |
| 35k | microsoft/playwright-mcp | 微软出品的 Playwright MCP |
| 33k | github/github-mcp-server | GitHub 官方 MCP |
| 28k | PrefectHQ/fastmcp | Pythonic MCP server 框架 |
| 28k | alirezarezvani/claude-skills | 388 skills × 13 IDE |
| 27k | OthmanAdi/planning-with-files | 持久化文件规划 |
| 25k | activepieces/activepieces | AI Workflow Automation(~400 MCP) |
| 20k | op7418/Humanizer-zh | 中文版 Humanizer skill |
| 19k | GoogleCloudPlatform/mcp-toolbox | 数据库 MCP |
| 16k | googleapis/mcp-toolbox | Google MCP toolbox |
| 15k | travisvn/awesome-claude-skills | 策展列表 |
| 14k | nidhinjs/prompt-master | 写 prompt 的 prompt |
| 13k | budtmo/docker-android | Android Docker + MCP + AI agent |
| 12k | Jeffallan/claude-skills | 67 个全栈技能 |
| 10k | BehiSecc/awesome-claude-skills | 策展 |
| 7.5k | trailofbits/skills | Trail of Bits 安全 |
| 7.4k | SnailSploit/Claude-Red | 红队 offensive skills |
| 6.6k | SawyerHood/dev-browser | Agent 浏览器 |
| 6.2k | joeseesun/qiaomu-anything-to-notebooklm | 多源 → NotebookLM |
| 5k | elementalsouls/Claude-BugHunter | Bug Hunting 82 skills |
| 4.6k | danyuchn/asd-ste100-skill | STE100 简化英语 |
| 3.7k | glitternetwork/pinme | 前端部署一行 |
| 3.6k | davepoon/buildwithclaude | 插件发现平台 |
| 3.5k | ElementalSouls/Claude-OSINT | OSINT 8 skills |
| 3.5k | tradermonty/claude-trading-skills | 股票交易 skills |
| 3.0k | Agentchengfeng/chengfeng-videocut-skills | 视频剪辑 Agent |
| 2.8k | elementalsouls/Claude-OSINT | OSINT 8 skills |
| 2.8k | dominikmartn/nothing-design-skill | Nothing 设计语言 |
| 2.5k | yctimlin/mcp_excalidraw | Excalidraw MCP + skill |
| 2.5k | aiwithremy/claude-skills-llm-council | 5 个 AI 顾问 + peer review |
| 2.3k | P4nda0s/reverse-skills | 逆向工程 |
| 2.2k | threerocks/hand-drawn-styles | 5 种手绘风提示词 |
| 2.0k | lemomo-ai/lemo-opuscar | 43 种影片风格 |
| 1.7k | NeoLabHQ/context-engineering-kit | Context 工程提升 agent 质量 |
| 1.4k | quemsah/awesome-claude-plugins | 自动索引的 plugin 列表 |
| 1.3k | obra/superpowers-marketplace | obra 自己策展的市场 |
| 1.2k | polyuiislab/infiAgent | 长时 agent 框架 |

---

## 十、参考资源

- **规范**:`https://agentskills.io/specification`
- **官方文档**:`https://support.claude.com/en/articles/12512176-what-are-skills`
- **官方 blog**:[Equipping agents for the real world with Agent Skills](https://anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills)
- **Plugin Marketplace 文档**:`https://code.claude.com/docs/en/plugin-marketplaces`
- **MCP 文档**:`https://modelcontextprotocol.io`
- **Skills API**:`https://docs.claude.com/en/api/skills-guide`

---

**结论**:Claude Skills 生态在 2026 H2 已进入「官方规范 + 大量社区技能 + 跨 runtime 兼容」的稳态期。**任何新团队接入 Claude Code,先装 `obra/superpowers` + `anthropics/skills` 两个就够了**,后续按需扩展。**写自己的 skill 比装别人的更重要**——`anthropics/skills` 里的 `skill-creator` skill 就是官方教学样板。
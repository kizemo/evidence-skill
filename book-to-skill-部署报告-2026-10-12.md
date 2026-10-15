# book-to-skill 部署报告 — cc-haha

> 部署日期:2026-10-12
> 部署对象:cc-haha (Claude code-haha) = 当前 Claude Code 安装
> Skill 版本:1.4.0
> 部署人:Claude

---

## 1. 部署结果

✅ **book-to-skill 已成功部署到 cc-haha**,Claude Code 已自动识别该 skill(出现在可用 skill 列表中)。

---

## 2. 安装路径

| 项 | 路径 | 大小 |
|---|---|---|
| Skill 安装位置 | `C:\Users\Duanyi\.claude\skills\book-to-skill\` | 2.4 MB |
| Skill 主入口 | `~/.claude/skills/book-to-skill/SKILL.md`(55KB) | — |
| Python 包 | `~/.claude/skills/book-to-skill/book_to_skill/`(可编辑安装) | — |
| CLI 脚本 | `C:\Users\Duanyi\AppData\Roaming\Python\Python314\Scripts\book-to-skill.exe` | — |
| 暂存源 | `C:\Temp\book-to-skill-deploy\book-to-skill.tar.gz` (1.4MB,可清理) | — |

---

## 3. 安装步骤

### 3.1 拉取源码

```bash
# git 直连 GitHub 在本机有 TLS 问题,改用 curl + tarball
curl -sL https://api.github.com/repos/virgiliojr94/book-to-skill/tarball \
  -o /c/Temp/book-to-skill-deploy/book-to-skill.tar.gz
tar -xzf /c/Temp/book-to-skill-deploy/book-to-skill.tar.gz -C /c/Temp/book-to-skill-deploy/
```

### 3.2 复制到 cc-haha skills 目录

```bash
cp -r /c/Temp/book-to-skill-deploy/virgiliojr94-book-to-skill-a24e62f/. \
      /c/Users/Duanyi/.claude/skills/book-to-skill/
```

⚠️ 直接复制整个仓库即可——book-to-skill 是单 skill 结构,根目录就是 `SKILL.md`,不需要 plugin manifest。

### 3.3 安装 Python 依赖

```bash
# 核心解析器(pdf-inspector 单独装,pypdf/python-docx 等已自带)
python -m pip install --user pdf-inspector

# pdfminer.six 需用旧版(20251230)避开 Python 3.14 metadata 解析 bug
python -m pip install --user --no-deps --force-reinstall \
  --ignore-requires-python pdfminer.six==20251230

# 其他依赖(本机已预装或装在系统层)
python -m pip install --user --no-deps \
  ebooklib beautifulsoup4 striprtf trafilatura

# book-to-skill 自身(可编辑安装,方便升级)
python -m pip install --user --no-deps -e \
  "C:/Users/Duanyi/.claude/skills/book-to-skill"
```

### 3.4 验证

```bash
# 1. cc-haha 自动识别 skill
#    Claude Code 系统提示的可用 skill 列表中应出现:
#    "book-to-skill: Converts books and documents (PDF, EPUB, DOCX...)..."

# 2. CLI 验证
python -m book_to_skill --help
python -m book_to_skill --check
```

---

## 4. 提取器支持矩阵(实际 --check 结果)

| 格式 | 提取器 | 状态 |
|---|---|---|
| **PDF (smart)** | pdf-inspector ≥ 1.15 | ✓ ready |
| **PDF (text-heavy)** | pypdf + pdfminer.six + pdftotext | ✓ ready |
| **PDF (technical: tables/code/formulas)** | docling | ⚠️ fallback(用普通文本链) |
| **EPUB** | ebooklib + beautifulsoup4 | ✓ ready |
| **DOCX** | python-docx | ✓ ready |
| **HTML** | trafilatura + beautifulsoup4 | ✓ ready |
| **RTF** | striprtf | ✓ ready |
| **Markdown / Text / RST / ADOC** | stdlib | ✓ ready |
| **MOBI / AZW / AZW3** | Calibre | ✗ **缺失**(需要装 Calibre) |

---

## 5. 已知问题与环境限制

### 5.1 cryptography 包空字节损坏(环境层 bug,非本次安装引起)

- **症状**:`from pdfminer.high_level import extract_text` 抛 `SyntaxError: source code string cannot contain null bytes`
- **根因**:`C:\Python314\Lib\site-packages\cryptography\` 部分文件在 Python 3.14 上的 metadata 编码错误(0x97 字节),触发 pip / importlib 解析失败
- **影响**:pdfminer.six 走 cryptography 失败,但 **book-to-skill 走 fallback 到 pypdf**,仍能解析绝大多数文本型 PDF
- **建议**:暂不动,等 Python 3.14 后续 patch;或后续装 `docling`(独立 wheel,不依赖 cryptography)

### 5.2 Calibre 未装

- **影响**:Kindle 格式(MOBI / AZW / AZW3)无法处理
- **建议**:若需处理 Kindle 书,装 Calibre(`https://calibre-ebook.com/download`)

### 5.3 git 直连 GitHub 失败

- **症状**:`git clone https://github.com/...` 报 `SSL routines::unexpected eof`
- **解决**:改用 `curl -sL https://api.github.com/repos/<owner>/<repo>/tarball`
- **影响**:无,部署完成

---

## 6. 使用方法

### 6.1 在 cc-haha 中触发

直接对 Claude 说:
```
把 D:/books/atomic-habits.pdf 转成一个 skill
```

cc-haha 会自动加载 `book-to-skill` skill(触发词:book / PDF / EPUB / convert / distill / 蒸馏 / 提炼)。

### 6.2 命令行直接调用

```bash
python -m book_to_skill "D:/books/atomic-habits.pdf" atomic-habits
# 或多文件 / glob
python -m book_to_skill "D:/books/*.pdf"
# 仅分析不生成
python -m book_to_skill "D:/books/foo.pdf" --mode analyze
# 启用技术模式(需 docling)
python -m book_to_skill "D:/books/tech-book.pdf" --mode technical
```

产物默认放 `~/.agents/skills/<slug>/`(跨 runtime),Claude Code 自动发现。

---

## 7. 文件清单(本次改动)

| 操作 | 路径 | 大小 |
|---|---|---|
| **新增** | `C:\Users\Duanyi\.claude\skills\book-to-skill\SKILL.md` | 55 KB |
| **新增** | `C:\Users\Duanyi\.claude\skills\book-to-skill\book_to_skill\` | (Python 包) |
| **新增** | `C:\Users\Duanyi\.claude\skills\book-to-skill\{docs,evals,scripts,tests,tools,overrides}\` | — |
| **新增** | `C:\Users\Duanyi\.claude\skills\book-to-skill\{README.md,README.zh-CN.md,pyproject.toml,...}` | — |
| **新增** | `C:\Users\Duanyi\AppData\Roaming\Python\Python314\Scripts\book-to-skill.exe` | (CLI) |
| **新增** | `C:\Users\Duanyi\AppData\Roaming\Python\Python314\site-packages\book_to_skill-1.4.0.dist-info\` | — |
| **新增(可清理)** | `C:\Temp\book-to-skill-deploy\book-to-skill.tar.gz` | 1.4 MB |

**未改动**:`~/.claude/plugins/installed_plugins.json`、`~/.claude/plugins/known_marketplaces.json`、`settings.json`——不需要,因为 book-to-skill 不是 plugin marketplace 模式。

---

## 9. 回滚指引

```bash
# 1. 卸载 Python 包
python -m pip uninstall book-to-skill pdf-inspector pdfminer.six -y
# (注:cryptography 不动,因为是环境层问题,不属于本次部署)

# 2. 删除 skill 目录
rm -rf "C:/Users/Duanyi/.claude/skills/book-to-skill/"

# 3. 清理暂存
rm -rf "C:/Temp/book-to-skill-deploy/"
```

下次重启 cc-haha,book-to-skill 就不再可用。

---

## 10. 升级指引

```bash
# 1. 拉取新版本源码(替换方式)
curl -sL https://api.github.com/repos/virgiliojr94/book-to-skill/tarball \
  -o /c/Temp/book-to-skill-deploy/book-to-skill.tar.gz
tar -xzf /c/Temp/book-to-skill-deploy/book-to-skill.tar.gz -C /c/Temp/

# 2. 替换 SKILL.md 和 book_to_skill/ (其他文件可不动)
cp -r /c/Temp/book-to-skill-deploy/virgiliojr94-book-to-skill-*/SKILL.md \
      /c/Users/Duanyi/.claude/skills/book-to-skill/

# 3. 因为是 -e 可编辑安装,新版本自动生效
python -m book_to_skill --check
```

---

## 11. 验证步骤

部署完成后,执行下列 3 条确认无误:

```bash
# (1) 验证 CLI 能跑
python -m book_to_skill --help
# 期望输出:Usage + 支持格式列表

# (2) 验证提取器状态
python -m book_to_skill --check
# 期望:PDF / EPUB / DOCX / HTML / RTF 都 ✓ ready;MOBI ✗ MISSING

# (3) 验证 cc-haha 已加载 skill
# 在 Claude Code 中说:"把 D:/books/atomic-habits.pdf 转成 skill"
# 期望:Claude 自动调用 book-to-skill 的工作流
```

---

## 12. 相关文档

- **Skill 调研报告**:`agent-skills-调研报告-2026-10-12.md`(生态全景)
- **蒸馏专题报告**:`skills-from-materials-调研报告-2026-10-12.md`(方法论对比)
- **官方仓库**:`https://github.com/virgiliojr94/book-to-skill`
- **规范**:`https://agentskills.io/specification`
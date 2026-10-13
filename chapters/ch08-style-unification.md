# Chapter 8: Style Unification — 5 Regions Pentagram CSS

## Core Idea

Sales BI Phase 1 样式统一完成:**5 个跨组统一区域**(Hero / KPI Strip / Chart Block / Table Block / Section Spacing)用 Pentagram 编辑设计语言,11 commits,3 小时 1 subagent-driven execution。**关键发现**:Core 是 AOT 编译二进制,CSS 在 image build time 烘焙,**T1+T2-T6 必须用 raw hex 值**(`var()` 优先级被覆盖),`.hall-page` class 不存在(wrap 是 `<article>`)。

## Frameworks Introduced

- **5 region 统一设计**:Hero / KPI Strip / Chart Block / Table Block / Section Spacing
  - When to use: 任何 Evidence 项目的样式统一
  - How: 每个 region 一组 CSS rule + 实际渲染验证

- **raw hex 而非 var()**:
  - When to use: Core v0.10.1 的 CSS override
  - How: `background: #FFFFFF;` 不是 `background: var(--bg-primary);`

- **`<article>` 选择器**:
  - When to use: Core HTML wrap
  - How: `article h1 { ... }` 不是 `.hall-page h1 { ... }`

- **nginx config 同步到 git**:
  - When to use: live nginx config 改完
  - How: `live/sales-bi.live-nginx-config` + `deploy/scripts/sync_nginx_config_to_git.sh`

- **CF Access 鉴权 for Playwright**:
  - When to use: 自动化截图 / 浏览器测试
  - How: `CLOUDFLARE_ACCESS_TOKEN` env var + `tests/cf_access_auth.py`

## Key Concepts

- **AOT 编译二进制 CSS 烘焙**:`evidencedev/serve` 是 Go/Rust 静态编译,CSS 在 image build time 烘焙到 binary
- **`theme.yaml` 装饰性**:不覆盖 runtime CSS vars,只作元数据
- **`:root` 块 SSR HTML 烘焙**:优先级最高,`var()` fallback 被覆盖
- **`.hall-page` class 不存在**:Core 实际 wrap 是 `<article>`,不能用 `.hall-page` 选择器
- **Pentagram 编辑设计语言**:深 navy `#0F2A52` + warm grey `#6B6B6B` + accent gold `#C8A658` + soft cream `#F5F2EC`

## Mental Models

- **CSS 烘焙顺序**:image build → binary → runtime,任何 runtime 改 CSS 都不生效
- **Pentagram 5 region 是 5 个独立子系统**:每个 region 自己的 CSS rule,不互相耦合
- **CSS 选择器必须 match 实际 HTML**:Core wrap 是 `<article>`,Legacy 是 `.hall-page`,切 Core 后 CSS 选择器要全改
- **CF Access 是必经层**:任何 prod 测试都要带 access token

## Anti-patterns

- **❌ 用 `var(--name)` 引用 CSS 变量** —— Core 烘焙了 `:root` 块,priority 最高,var() fallback 被覆盖
- **❌ 用 `.hall-page` 选择器** —— Core wrap 是 `<article>`,class 不存在
- **❌ 改完 nginx config 不同步 git** —— 后续 phase 无法 reference 当前 live config
- **❌ 自动化测试不传 CF Access token** —— Playwright 会被 CF 拦
- **❌ 想改 Core 静态 CSS** —— Core 是 AOT,改 source 后必须重建 image,不在 phase 1 范围

## Code Examples

**5 region CSS(Pentagram design tokens)**:

| 区域 | CSS 选择器 | Hex 值 |
|---|---|---|
| A. Hero | `article h1`, `article .iss-meta`, `article blockquote, article .callout` | title `#0F2A52` / meta `#6B6B6B` / callout bg `#F5F2EC` border-left `#C8A658` |
| B. KPI Strip | `article [class*="block-big-value"]` | bg `#FFFFFF` border `#E5E2DC` value `#0F2A52` |
| C. Chart Block | `article [class*="chart"], article .chart-container` | bg `#FFFFFF` border `#E5E2DC` padding `24px` |
| D. Table Block | `article table`, `article table thead th`, `article table tbody tr` | bg `#FFFFFF` thead `#F5F2EC` zebra `#FAF9F5` hover `#F0F4F8` |
| E. Section Spacing | `article h2`, `article h3` | h2 margin-top `32px` margin-bottom `16px` h3 margin-top `24px` |

**11 commits**:
```
f31accb feat(core): T8 visual gate + T9 handoff + C-2 CF Access auth helper
9e9a7b7 test(core): legacy CSS rename coverage (Phase 1 step 7)
53c8538 feat(core): Region E — Section Spacing rhythm (Phase 1 step 6)
6e36476 feat(core): Region D — Table Block sticky header + zebra (Phase 1 step 5)
2625077 feat(core): Region C — Chart Block card styling (Phase 1 step 4)
7eb527d feat(core): Region B — KPI Strip card styling (Phase 1 step 3)
4e910e4 feat(core): bring live nginx config into git (C-1)
a2d03f0 fix(T2): assert_hero full Region A checks + portable browser + rm restore_proxy_block
562005a feat(core): Region A — Hero CSS (Phase 1 step 2)
be44064 fix(plan): T2-T6 use raw hex not var()
d348be1 feat(core): Pentagram design tokens in theme.yaml
1f1ca2e docs(superpowers/plans): Phase 1 — Sales BI style unification plan
```

**总文件 4 个创建 + 1 个重命名**:
```
nginx-cutover/live/sales-bi.live-nginx-config   # C-1 同步 live nginx
deploy/scripts/sync_nginx_config_to_git.sh     # C-1 同步脚本
tests/cf_access_auth.py                        # C-2 CF Access Playwright
docs/superpowers/handoffs/2026-10-12-phase-1-style-unification-done.md  # T9 handoff
```
(已修改文件: `tests/visual_regression.py`、`evidence-core/static/sales-bi.css`、`tests/server_regression.py`、`theme.yaml`)

**CF Access Playwright**:
```python
# tests/cf_access_auth.py
import os
os.environ.setdefault("CLOUDFLARE_ACCESS_TOKEN", "<user-provided-token>")
# 然后跑 Playwright,5 page 截图
```

## Reference Tables

| 决策 | 选择 | 原因 |
|---|---|---|
| 5 region 而不是 1 全局 | 分 5 个独立 CSS rule | Phase 2/3 逐报表优化时各 region 独立演进 |
| raw hex 而不是 var() | T1+T2-T6 全用 raw hex | Core 烘焙 `:root` 优先级最高,var() fallback 失效 |
| `<article>` 而不是 `.hall-page` | article 选择器 | Core 实际 HTML wrap,Legacy class 不存在 |
| nginx config 同步 git | `live/sales-bi.live-nginx-config` | 后续 phase 可 reference 当前 live |
| CF Access token for Playwright | env var + helper | 不污染 repo,user 自行设 |

## Key Takeaways

1. **Core 是 AOT CSS 烘焙** —— runtime 改 CSS 不生效,必须重建 image
2. **T1+T2-T6 必须用 raw hex** —— `var()` 被覆盖
3. **`<article>` 是 Core wrap** —— 不是 `.hall-page`
4. **nginx config 同步 git** —— `live/` + sync script
5. **CF Access token for browser 验证** —— env var + helper script

## Connects To

- **ch01**:AOT 架构决定 CSS 烘焙
- **ch06**:nginx patches + 同步 git(C-1)
- **ch09**:browser 验证灰区(CF Access 拦 Playwright)
- **ch10**:Phase 2/3 后续工作

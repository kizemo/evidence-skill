# Chapter 15: Theme & Pentagram Phase 2 (R12 — 推翻 ch08 "theme.yaml 装饰性" 旧结论)

## Core Idea

**`theme.yaml` 在 Core 2026 不是装饰性** — 它真实消费并绑定 chart series 色 / color scale / 字体 / 圆角 / 间距 / density / depth / chart 动画 / table zebra;**5 层 inheritance** (默认 → org[Studio only] → project `theme.yaml` → page frontmatter `theme:` → component `chart_options.color_palette`) 全部 native supported。**推翻 ch08 旧结论**。

## 5 层 theme.yaml Inheritance

```
1. default           — Evidence 内置
2. org (Studio only) — 整个 org 共享
3. project theme.yaml — bi.jury.xin 根目录
4. page frontmatter  — 单页 override `theme: {...}`
5. component attr    — `chart_options.color_palette` 等
```

## theme.yaml 全字段 (Core v0.10.1)

```yaml
# theme.yaml (项目根目录, 2025-12-11 上线, 2026-06-24 扩展)
colors:
  base: "#FFF1E5"           # FT salmon 风格背景
  card: "#FFFFFF"
  cardLayoutBackground: "#F8F5EE"
  positive: "#0F5499"        # FT blue
  negative: "#990F3D"        # FT red
  info: "#0D7680"            # FT teal
fonts:
  heading: serif
  body: sans
  mono: monospace
radius: 8px                  # 0 / 4px / 8px / 16px
depth: subtle                # flat / subtle / pronounced
density: comfortable         # compact / comfortable / spacious
chart:
  barRadius: 4
  animation: subtle
table:
  rowLines: true
  hoverColor: "#F0F4F8"
  zebra: true
```

## 推翻 ch08 的关键发现

| 旧结论 (ch08) | 真实 (R12 2026-10) |
|---|---|
| `theme.yaml` 装饰性 | 5 层 inheritance,真实消费 |
| `var(--name)` 不可用 | 改用 `theme.yaml` 直接控制 colors / radius / density |
| 必须 raw hex 兜底 | `theme.yaml` 70% 视觉可原生 |
| `.hall-page` class | Core wrap 是 `<article>`(旧结论正确,保留) |

## `{% html %}` block 唯一 HTML 注入入口

2026-06-25 上线,markdown body 内 arbitrary HTML/JS/CSS 的**唯一官方入口**:

```markdown
{% html %}
<style>
  article h1 { border-bottom: 1px solid var(--color-card-border); }
  blockquote { border-left: 4px solid #C8A658; }
</style>
{% /html %}
```

**`{% html %}` 与 raw `<div>` 的区别**:
- `{% html %}...{% /html %}` → Core 编译通过,Markdown 内嵌 HTML/CSS/JS
- raw `<div>` / `<style>` 在 markdown body → 触发 `html-tags-not-allowed`,**整个父块**被替换
- `{% html %}` 是 Pentagram 品牌专属细节的兜底层

## Pentagram Phase 2 整改路径

**A. Hero** → `theme.yaml fonts.heading: serif` + `{% html %}` partials 处理 article h1 border-bottom
**B. KPI Strip** → `theme.yaml colors.card + radius:8px + depth:subtle` 全 native,Phase 1 `[class*="block-big-value"]` selector 退役
**C. Chart Block** → `theme.yaml chart.barRadius:4 + density:comfortable + colors.card`
**D. Table Block** → `theme.yaml table.rowLines:true + hoverColor + colors.cardLayoutBackground`,90% native
**E. Section Spacing** → `theme.yaml density:comfortable` + partial `article h2::before { /* 装饰 */ }`

**预估**: Phase 1 `evidence-core/static/sales-bi.css` 从 ~300 行缩到 ~80 行,只保留 Pentagram 品牌专属 toggle。

## Selectors 锚点 (Phase 1 + Phase 2 都用)

- **`<article>` / `<main>` / `:root`** / 裸 tag → Core 渲染选择器
- **ECharts 内部 `<div>` 全部 Svelte hash 化** → 不能用 class 锁定,只能用锚点
- **`article h1` 风格** → Phase 1 用 markdown-rendered 锚点继续正确

## Anti-patterns (Theme)

- ❌ 用 `var(--name)` 期待 fallback 链 → Core 烘焙 `:root` 优先级最高,var() 失效(改用 theme.yaml)
- ❌ 改 theme.yaml 后 `docker compose restart` → AOT cache,改完**必 force-recreate**
- ❌ theme.yaml 写无效 schema 字段 → 静默忽略,无警告
- ❌ `<div class="pentagram-...">` 自定义 class → Core Svelte 编译去除
- ❌ 在 markdown body 用 raw `<style>` → `html-tags-not-allowed`

## Connects To

- **ch08** (Pentagram Phase 1 实战 CSS) — Phase 1 仍可参考,Phase 2 路径升级
- **ch14** — theme 改动后必 `force-recreate` + 等 5s 才测试
- **ch05** — html-tags-not-allowed 唯一解

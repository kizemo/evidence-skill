# Chapter 20: Evidence Warehouse 真相 — Single-Page + Anchor 真相

> **R14 新增 · 2026-10-15**
> 源: `E:/obsidian/evidence/evidence-warehouse/evidence-warehouse.md` (11894 chars)
> 关键发现: sidebar 14 项是 **单页 anchor**,**所有连接器都在 `data-sources/` 主题中独立成页**

## Core Idea

docs.evidence.dev/evidence-warehouse 在 **Mintlify sidebar 中显示 14 个子项**(Overview + Databases(3) + Object Storage(6) + Data Lakes(2) + Flat Files(1)),但:

1. **sitemap 只列出 1 个 URL** — `/evidence-warehouse`
2. **所有 `/evidence-warehouse/*` 子路径返回 404**(已探测 13 个候选 URL,全 404)
3. **overview 页内 4 个 H2 anchor** 同页跳转:
   - `#databases` (position 111977)
   - `#object-storage` (position 118946)
   - `#data-lakes` (position 122692)
   - `#flat-files` (position 128402)
4. **sidebar 列出的 12 个连接器全部独立成页**,在 `data-sources/` 主题中(28 页已抓全)

## 关键启示:不要相信 sidebar 计数

Mintlify 文档站的 sidebar 是**纯展示**,可能:
- 把多个独立 page 假合并到一个 overview page 的 anchor 下(本例)
- 反过来,某些 sidebar item 实际上是独立 page
- 切勿以 sidebar 计数 = sitemap 计数

## 验证 SOP (碰到新 section 第一步)

1. 拉 `https://docs.evidence.dev/sitemap.xml` → grep section 关键词,看真实 URL 数量
2. 用 probe 脚本批量 HEAD 候选子路径(`/section/<key>`, `/section/<key>/<sub>`),全 404 即可确认单页
3. 用 anchor 探测脚本找页内 `<h2 id="...">` 或 `<a href="#...">`,确认 sidebar 是页内跳

## 12 个 Connector → 本地数据源映射

| Sidebar 名 | 本地 wikilink | URL |
|---|---|---|
| **Databases** | | |
| Athena | `data-sources/data-sources-athena.md` | `data-sources/athena` |
| MotherDuck | `data-sources/data-sources-motherduck.md` + `Direct Connectors/direct-connectors-motherduck.md` | `data-sources/motherduck` + `direct-connectors/motherduck` |
| Redshift | `data-sources/data-sources-redshift.md` | `data-sources/redshift` |
| **Object Storage** | | |
| Microsoft Azure Blob Storage | `data-sources/data-sources-abs.md` | `data-sources/abs` |
| BackBlaze | `data-sources/data-sources-backblaze.md` | `data-sources/backblaze` |
| Custom Bucket Provider | `data-sources/data-sources-custom.md` | `data-sources/custom` |
| Google Cloud Storage (GCS) | `data-sources/data-sources-gcs.md` | `data-sources/gcs` |
| Cloudflare R2 | `data-sources/data-sources-r2.md` | `data-sources/r2` |
| AWS (S3) | `data-sources/data-sources-s3.md` | `data-sources/s3` |
| **Data Lakes** | | |
| Delta Lake | `data-sources/data-sources-deltalake_s3.md` | `data-sources/deltalake_s3` |
| Iceberg | `data-sources/data-sources-iceberg.md` | `data-sources/iceberg` |
| **Flat Files** | | |
| Flat Files | `data-sources/data-sources-file.md` | `data-sources/file` |

**总计**: 12 个 sidebar 项,**全部已抓**(部分双源,如 MotherDuck 有 data-sources 和 direct-connectors 两份)。

## Evidence Warehouse 的实际内容 (来自本地 11894 chars 摘要)

- **概念**: Evidence 托管的数据仓库,Evidence Core 自动管理 source 的 schema、cache、refresh
- **核心能力**:
  - 自动从源(数据库 / 对象存储 / Data Lake / Flat Files)物化数据
  - 自动 schema 推断,SQL 查询无需显式建表
  - 多种刷新策略: Manual / Scheduled (5min/15min/1h/1d/1w)
- **与 Legacy 40.x 的差异**: 40.x 是 build-time 静态数据;Warehouse 是 runtime 物化,支持定时刷新
- **与 Studio 的差异**: Studio 提供完整 SaaS,Warehouse 是自托管的可选组件

## Anti-patterns

- ❌ 假设 `/evidence-warehouse/databases/athena` 存在 → 404,**跳 `#databases` anchor**
- ❌ 把 Evidence Warehouse 当作"另一种 data-sources" → 它是 metadata + cache 层,source 还是 data-sources 列出的
- ❌ 在 self-host 上期待 Warehouse 有完整功能 → 见 ch13 (Auth & RBAC),self-host 只有 Basic Auth,完整 Warehouse = Studio

## Reference Links

- 本地仓库主页: `E:/obsidian/evidence/evidence-warehouse/evidence-warehouse.md` (含 "Related Connectors" 章节)
- 原始页面: <https://docs.evidence.dev/evidence-warehouse>
- 验证脚本: `C:/Temp/probe_warehouse.py` + `C:/Temp/inspect_warehouse3.py`
- 连接器详情: `data-sources/_sources/evidence-docs/` (28 页)
- Direct Connector 模式: `Direct Connectors/direct-connectors-snowflake.md` 等 8 页

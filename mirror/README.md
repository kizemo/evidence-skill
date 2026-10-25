# mirror/ — Evidence 公开文档镜像

> **这是什么**:`docs.evidence.dev` 公开文档的**逐字镜像**,随 skill 一起分发。
> **为什么在 skill 里**:ch19 的 94 组件索引只给组件名 + 用途,完整 Attributes / Examples / 可用日期变量
> 在这里。远程安装者不必联网、不必另装 Obsidian vault,`ch19` 里的路径直接可达。
> **抓取时间**:2026-10-15(sitemap 153 URL 全覆盖,components 94/94)
> **上游**:<https://docs.evidence.dev>(Evidence.dev,MIT License)
> **原文标注**:每个文件头部都有 `> 原文: <https://docs.evidence.dev/...>`,可回溯逐字来源。

## 目录

| 目录 | 页数 | 内容 |
|---|---|---|
| `components/` | 94 | 全部组件文档,按 sidebar 8 大类分:`Values/` `Tables/` `Charts/` `Maps/` `Inputs/` `Custom/` `UI/` `Logic/` |
| `features/` | 14 | 页面级功能(page settings、editor、publishing 等) |
| `core-concepts/` | 8 | 架构 / 连接 / 查询 / 部署基础 |
| `cli/` | 3 | CLI 命令与连接配置 |
| `MCP/` | 3 | MCP 接入 |
| `self-host/` | 6 | 自托管形态与限制 |
| `Direct Connectors/` | 8 | 直连型连接器(bigquery / snowflake / postgres / s3 …) |
| `evidence-warehouse/` | 1 | Warehouse 单页真相(12 connector → data-sources 映射) |

## 怎么用

ch19 的「Quick Decision」表里给的路径就是这里的相对路径(相对于 skill 根目录):

```
mirror/components/Charts/components-combo_chart.md
mirror/components/Values/
mirror/evidence-warehouse/evidence-warehouse.md
```

agent 读 `chapters/ch19-components-full-reference.md` 时,直接按这些路径取全文即可 —— **离线可用**。

## 更新

镜像是一次性快照。Evidence 上游更新后重新抓取时,同步替换本目录并在
`CHANGELOG.md` 记一条;**不要**手工编辑镜像内容(它是原文副本,编辑会丢失与上游的可追溯性)。

## 不在镜像里的东西

作者的**个人 Obsidian vault**(`E:/obsidian/evidence/`)**不随 skill 分发** —— 它含生产环境细节
(服务器地址、库配置、事故复盘)。远程用户请自建 vault,写回模板见 `../templates/`。

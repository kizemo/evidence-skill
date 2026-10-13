# Chapter 7: Cross-DB with postgres_fdw

## Core Idea

Core Postgres Direct Connector 1 个 project = 1 个 PG database。要跨 DB 查询,正确做法是:**PostgreSQL `postgres_fdw` 在 erp DB 上建 foreign server + foreign table 代理 taobao DB**。Core 无需跨 connection,直接从 erp DB 看 taobao schema。**2-segment 路径 `taobao.mart_X` 正确,3-segment `taobao.bi.mart_X` 错**。

## Frameworks Introduced

- **postgres_fdw 跨 DB 模式**:
  - When to use: 1 个 Core 要读多个 PG database 的表
  - How: 在主 DB 建 extension + server + user mapping + 4 foreign tables

- **2-segment path `schema.table`**:
  - When to use: 写跨 DB foreign table 查询
  - How: `taobao.mart_shop_kpi_daily`,**不**写 `taobao.bi.mart_shop_kpi_daily`

- **架构真相**:Core connector 只能连 1 DB,但**应用层连 1 DB,DBA 层在 DB 内做 cross-database query**,跟 Core 无关
  - When to use: 设计多 DB 架构
  - How: 跨 DB 是 PG 内部的事,跟 Evidence connection.yaml 无关

- **`CREATE SERVER` port 是 quoted string**:
  - When to use: 写 `CREATE SERVER ... OPTIONS (port '5432', ...)`
  - How: `'5432'`(string)不是 `5432`(integer),否则 PG 报 syntax error

## Key Concepts

- **postgres_fdw**:PostgreSQL Foreign Data Wrapper,把另一个 PG server 的表映射成本地表
- **Foreign server**:指向远程 PG server 的连接定义(host、port、dbname)
- **User mapping**:本地 user → 远程 user 的认证映射
- **Foreign table**:本地表定义,但实际数据在远程 server,query 时 PG 自动代理
- **2-segment path**:`taobao.mart_X` —— 这里的 `taobao` 是本地 schema 名(在 erp DB 内建),不是 database 名
- **3-segment path 错**:`taobao.bi.mart_X` —— PG 看到 `taobao` 当 database 名,试图跨 DB ref,但 postgres_fdw 不支持
- **local scope CTE**:`WITH taobao.mart_X AS (...当前表...)` CTE 是 local scope,可用
- **142k 行**:4 mart 表总和,Core SQL 简单聚合可接受

## Mental Models

- **跨 DB 责任在 DBA 层,不在应用层**:Core 不用知道 taobao DB 存在,只连 erp DB
- **postgres_fdw 是 schema 桥**:在 erp DB 内建 `taobao` schema,表是 foreign 的,query 透明
- **3-segment 路径是常见误解**:PG 解析器看到第一个 `.` 前的字符串会优先当 database 名
- **2-segment 把"taobao"当 schema 名**:`taobao` 是 erp DB 内的 local schema,装 foreign tables

## Anti-patterns

- **❌ 假设 Core connector 能连多 DB** —— 错,只能 1 DB。要跨 DB 走 FDW
- **❌ 多 Core deployment** —— 不必要,1 Core + FDW 即可
- **❌ 写 3-segment path**:`taobao.bi.mart_X` → `cross-database references are not implemented`
- **❌ `CREATE SERVER ... OPTIONS (port 5432)`** —— port 必须是 quoted string `'5432'`
- **❌ 假设"不可能"未经实测** —— 我(AI)曾这么说,用户当场拒绝接受,该结论没经过实测验证

## Code Examples

**5 步建表 SQL(在 erp DB 上)**:
```sql
-- 1. 扩展启用(erp DB)
CREATE EXTENSION IF NOT EXISTS postgres_fdw;

-- 2. Foreign Server(指向 taobao DB)
CREATE SERVER taobao_pg FOREIGN DATA WRAPPER postgres_fdw OPTIONS (
    host '124.223.115.201',
    port '5432',                    -- 注意:quoted string,不是 integer
    dbname 'taobao'
);

-- 3. User mapping(kizemo@erp → kizemo@taobao)
CREATE USER MAPPING FOR kizemo SERVER taobao_pg OPTIONS (
    user 'kizemo',
    password '<password>'
);

-- 4. Taobao schema(在 erp DB 内创建空 schema,装 foreign tables 用)
CREATE SCHEMA IF NOT EXISTS taobao;

-- 5. 4 个 foreign tables(列定义从 information_schema 复制)
CREATE FOREIGN TABLE taobao.mart_shop_kpi_daily (
    "shop_name" text,
    "stat_date" date,
    "visitor_count" numeric,
    ...  -- 每个 column 必须明确定义
) SERVER taobao_pg OPTIONS (schema_name 'bi', table_name 'mart_shop_kpi_daily');
```

**正确 vs 错误路径**:
```sql
-- ✓ CORRECT (2-segment)
SELECT * FROM taobao.mart_shop_kpi_daily LIMIT 10;

-- ✗ WRONG (3-segment,误认为跨 DB ref)
SELECT * FROM taobao.bi.mart_shop_kpi_daily;
-- ERROR: cross-database references are not implemented
--        "taobao.bi.mart_shop_kpi_daily"
```

**实测验证**:
```bash
$ PYTHONIOENCODING=utf-8 python3 -c "
import psycopg2
conn = psycopg2.connect(host='124.223.115.201', dbname='erp', user='kizemo', password='...')
cur = conn.cursor()
cur.execute('SELECT COUNT(*) FROM taobao.mart_shop_kpi_daily')
print('shop_kpi_daily:', cur.fetchone())   # (1729,)
"
# ✓ 1729 rows
# ✓ ('大融医疗器械专营店', 19102, 86934.78)
```

**新增 mart 表**:
```sql
-- 在 erp DB 内查询已建表
SELECT ft.schemaname, ft.tablename, ft.ftoptions
FROM pg_foreign_table ft
JOIN pg_foreign_server fs ON ft.ftserver = fs.oid
ORDER BY 1, 2;

-- 加新表(从 information_schema 复制列定义)
CREATE FOREIGN TABLE taobao.mart_X (
    "col1" type1, "col2" type2, ...
) SERVER taobao_pg OPTIONS (schema_name 'bi', table_name 'X');

-- 重建(taobao DB schema 变化时)
DROP FOREIGN TABLE taobao.mart_X;
CREATE FOREIGN TABLE taobao.mart_X (...);
```

## Reference Tables

### 行为特征

| 查询 | 跨 DB 行为 |
|---|---|
| `SELECT * FROM taobao.mart_X` | ✓ foreign scan,代理到 taobao DB 的 `bi.mart_X` |
| `JOIN taobao.mart_X + bi.localY` | ✓ 两边都返回本地(foreign 看作 local table) |
| 三段路径 `taobao.bi.X` | ✗ 误认为跨 DB ref |
| `WITH taobao.mart_X AS (...当前表...)` | ✓ CTE local scope |

### 6 个恢复的 page(FDW 启用后 100% 数据完整)

| Page | 原始 spec | 实际查询 | 行为 |
|---|---|---|---|
| shop-overview | `taobao.bi.mart_shop_kpi_daily` | `taobao.mart_shop_kpi_daily` | 7 店 × 247 天,visitor/paid/CVR/promotion daily |
| shop-conversion | 同上 | 同上 | 5 段漏斗(visitor/cart/order/paid/repeat) |
| shop-customer | `taobao.bi.mart_customer_asset_daily` | `taobao.mart_customer_asset_daily` | 客户漏斗 + 粉丝细分 |
| shop-rating | `taobao.bi.mart_shop_kpi_daily` (DSR 字段) | 同上 | DSR 评分 + 评价数 |
| shop-traffic | `taobao.bi.mart_traffic_source_daily` | `taobao.mart_traffic_source_daily` | L1/L2/L3 流量来源 |
| promotion-traffic-light | `taobao.bi.mart_promotion_monthly` | `taobao.mart_promotion_monthly` | 红黄绿灯 4 渠道 |

## Key Takeaways

1. **Core connector 1 个 = 1 DB** —— 但**应用层连 1 DB,DBA 层做 cross-DB**,跟 Core 无关
2. **2-segment path**:`taobao.mart_X`,**不**写 `taobao.bi.mart_X`
3. **`CREATE SERVER` port quoted string**:`'5432'` 不是 `5432`
4. **跨 DB 是 PG 内部的事** —— 跟 Evidence connection.yaml 无关
5. **未经实测不要下"不可能"结论** —— 我(AI)曾这么说,被用户当场拒绝

## Connects To

- **ch05**:shop_* page 真实列名(shop_name、stat_date)与假名映射
- **ch10**:下个会话必做 — `erp` + `taobao` 多 DB 架构通过 FDW 已完成
- **ch09**:反思"我之前说'不可能'"—— 架构 limitation 措辞太重

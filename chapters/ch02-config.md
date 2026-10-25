# Chapter 2: Configuration — connection.yaml + evidence.config.yaml

## Core Idea

Core v0.10.1 配置两铁律:**`connection.yaml` FLAT top-level**(不是 list of dicts),**`evidence.config.yaml` 的 `project.evidence` 必须是版本字符串**(不是 object)。错格式 → silent fallback → "unsupported type undefined"。**改完必须 `docker compose up -d --force-recreate`**。

## Frameworks Introduced

- **FLAT top-level connection**:Core 期望 `type: postgres` 直接顶层,数据库信息也顶层
  - When to use: 写任何 `connection.yaml`
  - How: 不要写 `connections: [list of dicts]`,直接 `type:` + `host:` + `database:` + `schema:`

- **STRING version field**:`project.evidence` 是 Core 用来检查项目最初由哪个版本创建的字段,**不是数据库引用**
  - When to use: 写 `evidence.config.yaml`
  - How: 写成字符串 `"0.10.1"`,不要写成 object/databases list

- **force-recreate on config change**:AOT binary 启动时 cache connections,运行时不会重读文件
  - When to use: 改任何 Core 配置(连接、env、theme)
  - How: `docker compose up -d --force-recreate`,不是 `restart`

- **Silent fallback debugging**:Core 配错不报错,要靠外部验证(connectionType grep、PG 直连)
  - When to use: Core 报"unsupported type"或行为异常
  - How: `docker logs` 查 `Failed to load`,curl 验 `connectionType:"postgres"`

## Key Concepts

- **FLAT top-level**:`type: postgres` 写在 YAML 顶层,不嵌套
- **silent fallback**:Core 读错配置后**不抛错**,退回默认(默认 `type: undefined`)
- **`unsupported type undefined`**:Core 不知道连什么数据库,典型表现
- **`Failed to load evidence.config.yaml; using default pages directory`**:Core 日志里看到这句 → 配错了
- **force-recreate**:Compose flag,完全销毁 + 重建 container,触发 bind-mount 重挂载 + 进程重启
- **`AUTH_DISABLED=true`**:prod Core 走 CF Tunnel SSO,关基本 auth
- **`bind-mount`**:Docker `-v ./:/project` 把 host 目录挂进容器,改 host 文件 → 容器内即时可见(但**进程不会自动 reload**)

## Mental Models

- **Core 配置 = AOT binary 的输入** —— 启动时读一次,然后**冻结**
- **`project.evidence` 字段是版本指纹** —— 不是数据库引用,Database 通过 `connection.yaml` 的 `database:` 字段
- **silent fallback = 配置反模式** —— Core 应该抛错,但目前 silent → 用户体验差,排查靠外部验证

## Anti-patterns

- **❌ 写 `connections: [list of dicts]`**:
  ```yaml
  # WRONG
  connections:
    - name: sales_bi
      type: postgres
      database: erp
  ```
  Core 读到 `connections:` 但内部逻辑只识别顶层 `type` → silent fallback → `type: undefined`

- **❌ `evidence:` 写成 object**:
  ```yaml
  # WRONG
  project:
    evidence:
      databases: [sales_bi, shop_bi]
  ```
  Core 报 `project.evidence must be a non-empty string`,silent fallback

- **❌ 改完 config 跑 `docker compose restart`**:StartedAt 不变,文件不重读
- **❌ 用旧记忆**(我之前 list-of-dicts 是错的)——已纠正,Core v0.10.1 文档明确 FLAT

## Code Examples

**正确 `connection.yaml`**:

> ⚠️ 库密码已脱敏 —— 真实凭据只留在作者本机,不随 skill 分发。

```yaml
type: postgres
host: 124.223.115.201
port: 5432
user: kizemo
password: <redacted>
database: erp
schema: bi
sslmode: disable
```

**正确 `evidence.config.yaml`**:
```yaml
project:
  name: sales-bi
  evidence: "0.10.1"
```

**正确 `docker-compose.yml` 关键段**:
```yaml
volumes:
  - ./:/project              # bind-mount 让 config 文件被 Core 直接读
environment:
  - AUTH_DISABLED=true        # prod 不设基本 auth(若 CF Tunnel 已是 SSO)
  - POSTGRES_PASSWORD_SALES_BI=...   # 若用 ${...} 引用 .env 的密码
```

**改完 config 后强制 reload**:
```bash
cd /opt/sales-bi/evidence-core
sudo docker compose up -d --force-recreate   # 必须是 --force-recreate
```

**验证 Core 真的连上 PG**:
```bash
# 看 Core log
sudo docker logs sales-bi-core --tail 20

# curl SSR HTML 找 connectionType
curl -s 'http://127.0.0.1:8080/' | grep -oE 'connectionType:[^,}]*'
# 应见: connectionType:"postgres", hasLocalConnection:true, isServe:true
```

## Reference Tables

| 字段 | 必填 | 格式 | 错误示例 |
|---|---|---|---|
| `connection.yaml` 顶层 `type` | ✅ | string | `type: postgres` 不是 `connections: [list]` |
| `database` | ✅ | string | PostgreSQL database name |
| `schema` | ❌ | string (默认 `public`) | 也可写 `schemas: [bi, analytics]` |
| `host` / `port` / `user` / `password` | ✅ | string | 密码含特殊字符直接 string |
| `evidence.config.yaml` `project.evidence` | ✅ | **string** (版本号) | `"0.10.1"` 不是 object |
| `docker-compose.yml` `volumes: ./:/project` | ✅ | bind-mount | Core 启动时读这里面的 config |

## Key Takeaways

1. **FLAT + STRING** 是两铁律,违反就是 silent fallback
2. **force-recreate 是唯一可靠 reload 方式** —— restart 不够
3. **Core silent fallback 是反模式** —— 排查必须外部验证
4. **`project.evidence` 是版本指纹** —— 不是 database 引用
5. **改 config 后看 Core log + curl 验** —— 不要靠记忆

## Connects To

- **ch01**:AOT binary 架构决定 force-recreate 是必须的
- **ch04**:5 critical bugs 中 #1(connection.yaml LIST)+ #2(evidence.config.yaml STRING)都出自本章
- **ch05**:SQL 列名错位(数据库连上后才暴露的下一层)
- **ch09**:silent fallback 导致 3 次 fake success

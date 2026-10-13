# Chapter 1: Architecture — Evidence 40.x (Legacy) vs Core 2026

## Core Idea

Evidence 40.x 是 SvelteKit vite dev + 静态 build 混合架构;**Core 2026 v0.10.1 是独立 Docker container 跑的 AOT 编译二进制**。两者 nginx upstream 切法不是 1 行 sed,是 **4 处 patch**。

## Frameworks Introduced

- **AOT binary 架构认知**:`Core` 是 Go/Rust 静态编译二进制,启动时 cache connections 和 CSS,改 bind-mount 文件必须 `force-recreate` 才生效。
  - When to use: 任何 Core 配置 / 主题 / 静态资源改动后
  - How: `docker compose up -d --force-recreate`(不是 `restart`)

- **Legacy 双模式混合**:Legacy 40.x 跑 `evidence dev`(vite dev on :3000)+ nginx serve `/var/www/evidence/` 静态 build
  - When to use: 理解为什么 nginx 有这么多 `proxy_pass :3000` + 静态 `root` 块
  - How: 知道这是过渡架构,Core 切完才能退役

- **nginx patch 而非 sed**:nginx duplicate `location /` 会 fail 编译,所以 catch-all 必须**整块 replace**,不能 insert
  - When to use: 写任何 nginx 切换脚本
  - How: 用 `m.end()` 替换 `m.start()` 锚点;`nginx -t` 验证后再 reload

## Key Concepts

- **AOT ELF binary**:`/usr/local/bin/evidence`,Core container 内的主进程
- **Hash-named static assets**:`/_app/immutable/HASH.css` —— Core 静态资源命名,**跟 Legacy `/var/www/evidence/_app/` 不匹配** → 切时必须删 Legacy 静态规则
- **Silent fallback**:Core 配置错时**不报错**,silent 退回默认 → 表现是"`unsupported type undefined`"或"`unsupported type postgres`"(默认 fallback)而不是清晰报错
- **Force-recreate**:`docker compose up -d --force-recreate` 完整销毁 + 重建 container,确保 bind-mount 重新挂载 + AOT 进程重启
- **Backup chain**:`/etc/nginx/sites-enabled/.bak/sales-bi.legacy.YYYYMMDD-HHMMSS` —— 切前自动备份,5 秒回滚用

## Mental Models

- **Docker container = AOT binary 包装**:容器内 `0.0.0.0:3000` → host `:3002` → nginx → CF Tunnel → user
- **nginx 是 SPOF(单点)**:nginx upstream 一变,所有 page 同时变 → 切前必须 backup + 写回滚脚本
- **配置改 = 重建容器**(不是 hot reload):AOT binary 启动时 cache,改文件不通知
- **Silent fallback 是反模式**:Core 失败时**不抛错**,只给模糊错误 → 排查要靠多 pattern grep + 直连 DB

## Anti-patterns

- **❌ `docker compose restart` 改 config**:StartedAt 不变,bind-mount 不重读,AOT binary 仍用旧 config
- **❌ 假设 1 行 sed 切 Core**:nginx 实际需要 4 处 patch;insert catch-all 会 duplicate location
- **❌ Core 还在跑 Legacy 静态 root**:`root /var/www/evidence` 必须删,否则 CSS 走 Legacy 不走 Core
- **❌ 保留 `_app/` 静态规则**:Core hash 命名不匹配 → 404 → 修法是**删整块**
- **❌ Legacy 立刻清掉**:切 Core 后保留 1-2 周观察期,确认 Core 稳定再清

## Code Examples

```bash
# 切 Core(完整 4 处 patch)
sudo bash /opt/sales-bi/deploy/scripts/nginx-cutover/nginx_switch_to_core.sh

# 切前自动 backup + nginx -t 验证
ls /etc/nginx/sites-enabled/.bak/ | tail -3
# sales-bi.legacy.20261012-074453  (第一次切,失败回滚)
# sales-bi.legacy.20261012-074626  (第二次切,成功,当前 live)

# 5 秒回滚
sudo bash /opt/sales-bi/deploy/scripts/nginx-cutover/nginx_switch_to_legacy.sh

# 切后强制 Core 重读 config
cd /opt/sales-bi/evidence-core
sudo docker compose up -d --force-recreate
```

## Reference Tables

| Legacy 行为 | Core 后必须 | 原因 |
|---|---|---|
| 主入口 `location / { try_files ... }` | **替换**整块为 `proxy_pass :3002` | nginx 不允许 duplicate `location /`;insert 会冲突 |
| 5 个 vite dev proxy `proxy_pass :3000` | 改 `:3002` | Core 内部 API 也走这些路径 |
| `location ~ "^/_app/"` 静态规则 | **删除** | hash 名跟 Core 不匹配 → 404 |
| `location ~* \.(js\|css\|...)$ { expires ... }` | 加 `proxy_pass :3002` 在 `expires` 前 | regex 优先于 catch-all,不带 proxy_pass 直接走磁盘 root → 404 |
| `location = / { sub_filter ... }` | 改成 `proxy_pass :3002`,删 sub_filter | Legacy sub_filter 注入 CSS,Core 不需要 |

## Key Takeaways

1. **Core 是 AOT binary,不是 Node 服务** —— 这是所有差异的根因
2. **改 Core config 必须 force-recreate** —— `restart` 不会重读 bind-mount
3. **nginx 切 Core 是 4 处 patch** —— 不是 1 行 sed
4. **silent fallback 是反模式** —— Core 错时不抛,排查要直连 DB + 多 pattern grep
5. **切前 backup,切后保留 Legacy 1-2 周** —— 不要急着清

## Connects To

- **ch02**:Configuration 改完 → 必须 force-recreate
- **ch04**:Cutover 实战时间线,5 critical bugs
- **ch06**:nginx 4 处 patch 详细 diff
- **ch10**:server paths 速查

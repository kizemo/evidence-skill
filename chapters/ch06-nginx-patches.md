# Chapter 6: nginx Patches — 4 Required Edits for Core Cutover

## Core Idea

切 Core **不是 1 行 sed**,是 **4 处 patch**(实际是 5 处,5 是 sub_filter 删除):5 个 dev proxy port + catch-all replace + `_app/` 删除 + `\.css?$` 加 proxy_pass + `sub_filter` 删除。每处都有特定原因,任一漏掉都会触发具体 404 或样式错。

## Frameworks Introduced

- **5 处 patch 必做**:逐一覆盖 Legacy 静态 root + Core 静态资源 + dev proxy + catch-all + sub_filter
  - When to use: 任何 Legacy 40.x → Core 切 prod
  - How: 写脚本统一改 5 处,每处用 `nginx -t` 验证编译

- **`m.end()` replace 而非 `m.start()` insert**:
  - When to use: 用 Python regex 改 nginx config
  - How: `re.sub(pattern, replacement, content)` 其中 replacement 锚到 `m.end()` 之后

- **regex 优先于 catch-all**:
  - When to use: nginx `location` 匹配
  - How: `location ~* \.(js|css|...)$` 必须自己加 `proxy_pass`,否则 regex 命中后走磁盘 root

- **5 秒回滚 + backup 链**:
  - When to use: 任何高风险 nginx 切
  - How: 切前自动 cp 到 `.bak/sales-bi.legacy.YYYYMMDD-HHMMSS`,切后跑 5 page 烟雾测试,失败立即回滚

## Key Concepts

- **`location = /` vs `location /`**:精确匹配 vs 前缀匹配,nginx 不允许 duplicate
- **`location ~ "^/_app/"`**:regex 匹配,优先级**高于** `location /` catch-all
- **`location ~*`**:case-insensitive regex
- **`sub_filter '...' '...'`**:Legacy 注入 CSS 用的,Core 不需要
- **`.bak/` 目录**:nginx live config 备份链,自动滚动
- **`proxy_pass` 整块**:`proxy_http_version 1.1` + `proxy_set_header` + `proxy_buffering off` + `proxy_read_timeout 300s`

## Mental Models

- **nginx 匹配优先级**:精确 `=` > 前缀 `^~` > regex `~` > 普通前缀(最长匹配)
- **patch 5 处的内在逻辑**:Core 不再需要 vite dev(`/api/*`、`/@*`、`/src/*`、`/node_modules/*`、`/\.svelte-kit/*` 都改 `:3002` 走 Core)+ Core 静态资源走 `_app/immutable/HASH.css`(Legacy `_app/` 删除)+ catch-all 走 Core + CSS/JS regex 走 Core + Legacy CSS 注入不需要(sub_filter 删除)
- **patch 错的最常见症状**:CSS 404(Error badge 顶部)+ 主页空 + sub_filter 冲突 Core CSS

## Anti-patterns

- **❌ 1 行 sed 切 Core** —— 至少 5 处 patch
- **❌ 用 `m.start()` insert catch-all** —— duplicate `location /` nginx 编译失败
- **❌ 保留 `location ~ "^/_app/"` 静态规则** —— Core hash 命名不匹配
- **❌ `location ~* \.(js|css|...)$` 不加 `proxy_pass`** —— regex 命中后走磁盘 root,Core 静态资源 404
- **❌ 保留 Legacy `sub_filter`** —— Core 自带 CSS 跟 Legacy 冲突

## Code Examples

**Patch 1:5 个 vite dev proxy `:3000` → `:3002`**:
```diff
- proxy_pass http://127.0.0.1:3000;
+ proxy_pass http://127.0.0.1:3002;
```
匹配位置(`/etc/nginx/sites-enabled/sales-bi`):
- L257: `location ~ "^/@"`
- L267: `location ~ "^/src/"`
- L283: `location ~ "^/node_modules/"`
- L291: `location ~ "^/\.svelte-kit/"`
- L301: `location ~ ^/api/.*\.(json|arrow)$`

**Patch 2:`location / { try_files ... }` replace 整块为 catch-all `proxy_pass :3002`**:
```diff
  location / {
-     try_files $uri $uri/ /index.html =404;
+     proxy_pass http://127.0.0.1:3002;
+     proxy_http_version 1.1;
+     proxy_set_header Host $host;
+     proxy_set_header X-Real-IP $remote_addr;
+     proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
+     proxy_set_header X-Forwarded-Proto $scheme;
+     proxy_buffering off;
+     proxy_read_timeout 300s;
+     proxy_connect_timeout 10s;
  }
```
⚠ **不能 insert 前,必须 replace**!nginx duplicate `location /` 会 fail 编译。

**Patch 3:**删除 `location ~ "^/_app/"` 静态规则:
```diff
- location ~ "^/_app/" {
-     root /var/www/evidence;
-     try_files $uri =404;
-     access_log off;
-     expires 30d;
-     add_header Cache-Control "public";
- }
```
原因: Core 的 `/_app/immutable/HASH.css` hash 名跟 Legacy `/var/www/evidence/_app/` 不同 → 404。删后让 catch-all 处理。

**Patch 4:`location ~* \.(js|css|...)$ { expires ... }` 加 `proxy_pass :3002`**:
```diff
  location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff2?|webmanifest)$ {
+     proxy_pass http://127.0.0.1:3002;
      expires $css_cache_age;
      add_header Cache-Control "public";
      access_log off;
  }
```

**Patch 5:`location = / { sub_filter ... }` 改 `proxy_pass :3002`,删 sub_filter**:
```diff
  location = / {
-     sub_filter '</head>' '<link rel="stylesheet" href="/static/...">...';
-     try_files /index.html =404;
+     proxy_pass http://127.0.0.1:3002;
+     proxy_http_version 1.1;
+     proxy_set_header Host $host;
+     proxy_set_header X-Real-IP $remote_addr;
+     proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  }
```
原因:Legacy sub_filter 注入 `/static/finance-light-overrides-v7.css` 等,**Core 不需要**(会跟 Core 自带 CSS 冲突)。

**5 秒回滚**:
```bash
sudo bash /opt/sales-bi/deploy/scripts/nginx-cutover/nginx_switch_to_legacy.sh
```
回滚脚本自动:
- 找最新 `/etc/nginx/sites-enabled/.bak/sales-bi.legacy.YYYYMMDD-HHMMSS`
- `cp backup sales-bi`
- `nginx -t` + `nginx -s reload`
- 5 page 烟雾测试

## Reference Tables

| # | 位置 | 操作 | 原因 |
|---|---|---|---|
| 1 | 5 个 vite dev proxy `:3000` | 改 `:3002` | Core 内部 API 走这些路径 |
| 2 | `location / { try_files ... }` | **replace 整块**为 `proxy_pass :3002` | nginx 不允许 duplicate `location /`;insert 会冲突 |
| 3 | `location ~ "^/_app/"` | **删除** | hash 名跟 Core 不匹配 → 404 |
| 4 | `location ~* \.(js\|css\|...)$` | 加 `proxy_pass :3002` 在 `expires` 前 | regex 优先于 catch-all,不带 proxy_pass 直接走磁盘 root → 404 |
| 5 | `location = / { sub_filter ... }` | 改 `proxy_pass :3002`,删 sub_filter | Legacy sub_filter 注入 CSS,Core 不需要 |

## 已知未处理的 nginx 块(故意保留)

| 块 | 作用 |
|---|---|
| 29 个 `location = /path/ { 308 redirect ... }` | URL 重写(`/sales-overview/` → `/大盘/sales-overview/`) |
| `location /static/finance-light-overrides-v7.css` | Legacy CSS 服务(若 hard-rolled URL 直接命中) |
| `location /api/empty_queries` 兜底 | prerender 404 fallback |
| `sub_filter ... ...` HTML no-store block | HTML response cache-control: no-store |

这些对 Core 工作无害,1-2 周 Legacy cleanup 时一起删。

## Key Takeaways

1. **5 处 patch 不是 1 行 sed** —— 每处都有特定原因
2. **Patch 2 必须 replace 整块** —— insert 会触发 `duplicate location /`
3. **Patch 3 删 `_app/`** —— Core hash 命名不同
4. **Patch 4 regex 优先于 catch-all** —— 必须自己加 `proxy_pass`
5. **5 秒回滚脚本** —— 切前 backup,切后 5 page 烟雾测试

## Connects To

- **ch01**:4 处 patch 是 Legacy → Core 架构差异的直接表现
- **ch04**:第一次切 prod 时 Patch 2 用 insert 触发 duplicate location,自动回滚
- **ch10**:server paths 速查(脚本位置 + backup 链)

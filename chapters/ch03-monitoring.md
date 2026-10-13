# Chapter 3: Monitoring & Operations — systemd timer + 3-Dim Health Check

## Core Idea

Core 健康监控通过 `systemd timer` 每 5 分钟跑一次 `monitor_core.sh`,做 3 维检查:container 状态 + 5 page 烟雾测试 + Docker logs 错误扫描。已 enable + start。监控**只写 journal**,**未接飞书/钉钉 webhook**(待 user 决定)。

## Frameworks Introduced

- **3 维健康检查**:
  - When to use: Core 切 prod 后必建
  - How: container 状态 + 5 page 烟雾测试 + Docker logs 错误扫描

- **systemd timer oneshot**:用 `OnUnitActiveSec=5min` 触发 `Type=oneshot` service
  - When to use: 周期性短任务(健康检查、清理、备份)
  - How: timer 触发 → service 跑一次 → exit

- **Exit code 语义化**:`0`=OK,`1`=WARNING,`2`=CRITICAL —— 配合 systemd `SuccessExitStatus` 区分
  - When to use: 让 systemd 知道怎么报 alert
  - How: 脚本 `exit 0/1/2`,service unit 配 `SuccessExitStatus=0 WarningExitStatus=1 CriticalExitStatus=2`

- **journalctl 查监控日志**:
  - When to use: 5 分钟内想知道 Core 状态
  - How: `journalctl -u sales-bi-core-monitor.service -n 20`

## Key Concepts

- **`monitor_core.sh`**:3 维检查脚本,在 `/opt/sales-bi/deploy/scripts/nginx-cutover/`
- **5 page 烟雾测试**:`/`、`/home`、`/大盘/sales-overview`、`/售后风险/after-sales`、`/mart-health`
- **`WARN` vs `CRIT` threshold**:`grep -E "ERROR|Exception|TypeError|Syntax Error|Binder Error|Catalog Error"` ≥1 行 WARN,≥5 行 CRIT
- **`OnBootSec=2min`**:开机 2 分钟后首次跑,避免 docker 还没启动完
- **`AccuracySec=10s`**:timer 精度 10 秒,避免多个 timer 同时触发
- **oneshot service**:`Type=oneshot` 表示跑一次就退出,不等持续运行

## Mental Models

- **监控 = 三道防线**:container(进程层) + page(应用层) + log(异常层),任一异常都要爆
- **`set -uo pipefail`**:monitor 脚本开了这个,`grep -c` 返回 `0\n0` 时会触发 `[: 0` 错(脚本微 bug)
- **systemd 优先于 cron**:timer 比 cron 更现代(支持依赖、日志、状态),Core 监控用 timer 不用 cron
- **journal 是默认目的地**:Core 监控不接 webhook 之前,所有信号进 journal,要查用 `journalctl`

## Anti-patterns

- **❌ 只查 container 状态**:Core 进程在但应用挂(比如 connection error 静默)是查不到的
- **❌ 只 grep 单一 error pattern**:详见 ch09 postmortem(漏报 90% error)
- **❌ `docker compose restart` 模拟 reload**:见 ch02(StartedAt 不变,config 不重读)
- **❌ `grep -c` 直接当 integer 用**:`grep -c` 在多文件时返回 `file: count\nfile: count`,触发 `[: 0` 错

## Code Examples

**`monitor_core.sh` 关键段**:
```bash
# 1. Container 状态
docker ps --filter name=sales-bi-core | grep -q sales-bi-core
[ $? -ne 0 ] && exit 2

# 2. 5 page 烟雾测试
for url in / /home /大盘/sales-overview /售后风险/after-sales /mart-health; do
  code=$(curl -s -o /dev/null -w "%{http_code}" "http://127.0.0.1:3002${url}")
  [ "$code" != "200" ] && FAIL_COUNT=$((FAIL_COUNT+1))
done
[ $FAIL_COUNT -ge 2 ] && exit 2
[ $FAIL_COUNT -ge 1 ] && exit 1

# 3. Docker logs 错误扫描
ERROR_COUNT=$(docker logs sales-bi-core --tail 100 2>&1 | grep -cE "ERROR|Exception|TypeError|Syntax Error|Binder Error|Catalog Error")
[ $ERROR_COUNT -ge 5 ] && exit 2
[ $ERROR_COUNT -ge 1 ] && exit 1

exit 0
```

**`/etc/systemd/system/sales-bi-core-monitor.service`**(oneshot):
```ini
[Unit]
Description=Evidence Core (sales-bi) health monitor
After=docker.service
Requires=docker.service

[Service]
Type=oneshot
User=root
WorkingDirectory=/opt/sales-bi
ExecStart=/bin/bash /opt/sales-bi/deploy/scripts/nginx-cutover/monitor_core.sh
StandardOutput=journal
StandardError=journal
SyslogIdentifier=sales-bi-core-monitor
SuccessExitStatus=0
WarningExitStatus=1
CriticalExitStatus=2
```

**`/etc/systemd/system/sales-bi-core-monitor.timer`**:
```ini
[Unit]
Description=Run Evidence Core health check every 5 minutes
Requires=sales-bi-core-monitor.service

[Timer]
OnBootSec=2min
OnUnitActiveSec=5min
AccuracySec=10s
Unit=sales-bi-core-monitor.service

[Install]
WantedBy=timers.target
```

**启用 + 状态**:
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now sales-bi-core-monitor.timer

sudo systemctl list-timers --all | grep sales-bi-core-monitor
# Mon 2026-10-12 07:51:48 CST 4min 56s left

sudo journalctl -u sales-bi-core-monitor.service -n 20
sudo /opt/sales-bi/deploy/scripts/nginx-cutover/monitor_core.sh
```

## Reference Tables

| 检查维度 | 命令 | fail 标准 | exit |
|---|---|---|---|
| Container 状态 | `docker ps --filter name=sales-bi-core` | 容器名不在 | 2 |
| 5 page 烟雾测试 | `curl :3002/{5 paths}` | 任一 ≠ 200 | 1 (1 fail) / 2 (≥2 fail) |
| Docker logs 错误 | `docker logs ... \| grep -E "ERROR\|..."` | ≥1 行 WARN,≥5 行 CRIT | 1 / 2 |

## Key Takeaways

1. **3 维检查缺一不可** —— container + page + log
2. **exit code 语义化** —— 让 systemd 知道怎么报 alert
3. **journal 是默认 destination** —— 没接 webhook 之前,所有信号进 journal
4. **monitor 脚本微 bug**:`grep -c` 返回多行触发 `[: 0` 噪音,仅 stderr 不影响 exit
5. **下一步**:接飞书/钉钉 webhook + 自动 `nginx_switch_to_legacy.sh` 回滚(等 user 决定)

## Connects To

- **ch04**:5 critical bugs 切 prod 后才有 monitor 需求
- **ch06**:nginx 切换后需要 monitor 守住
- **ch09**:真验证 SOP = 5 维检查(监控只是其中 1 维)
- **ch10**:server paths 速查(脚本位置)

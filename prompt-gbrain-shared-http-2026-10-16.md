# Prompt — 接力 v3.1 ingest + HTTP serve 恢复 — 2026-10-16

承接本轮 gbrain 共享 HTTP serve 迁移 + evidence-skill 优化。本会话结束时 HTTP serve 死了,stdios PID 742340 + 953392 仍跑,port 3131 空。

**第一步必做**(开新会话就先跑,不等命令):

```powershell
taskkill /F /IM gbrain.exe /T
```

**第二步必读**(按序,4 份文档就够):

1. `F:\soft\00selfmade\evidence-skill\handoff-gbrain-shared-http-2026-10-16.md` ← 本轮完整 handoff(必读)
2. `F:\soft\00selfmade\evidence-skill\handoff-v3.1-ingest-2026-10-15.md` ← 上轮 ingest 上下文
3. `F:\soft\00selfmade\evidence-skill\task.md` ← active phase 状态
4. `E:\Users\Duanyi\.claude\skills\evidence-skill\SKILL.md` ← 升级后 995-word skill

**未经验证**:Codex wired + Claude Code 5 hooks 落盘 + mcpServers 段已加,但都没经过运行时验证 — 必须先 taskkill + 起重启 serve 才能继续。

**禁止事项**:启动第二个 `gbrain serve --http` / 删 `C:/Temp/gbrain-bootstrap-premint-*.log` / 不 plan 跑 `gbrain self-upgrade` / 跑 `gbrain skillopt` 不先 taskkill。

**下阶段必交付**(按优先级):① taskkill + 起 serve 跑 `gbrain doctor harness_wiring` 验证双 harness ② ingest 168 页 docs mirror + R1-R21 到 brain(此前未跑,被锁阻塞) ③ 观察 Anti-Patterns 5 行飞轮 1-2 周

收到先 taskkill + 读 handoff,再 "ready" + 你的推进计划。

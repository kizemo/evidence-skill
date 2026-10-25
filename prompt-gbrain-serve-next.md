# 下一会话开场提示

承接:`handoff-gbrain-serve-and-skill-v32-2026-10-25.md`
(F:/soft/00selfmade/evidence-skill/handoff-gbrain-serve-and-skill-v32-2026-10-25.md)

## 当前基线
- 分支 `main`,HEAD `9d67476`,已 push,working tree clean
- evidence-skill v3.2 **已完成**:SKILL.md 触发成本 2,662 → 1,520 tok(-43%)
- gbrain 版本 **0.60.287.0**;共享 HTTP serve **死的**,port 3131 空
- 3 个 stdio gbrain 进程占着 PGLite 锁(249632 / 263644 / 413556)

## 开工第一步(顺序不能反)
1. `powershell.exe -NoProfile -Command "taskkill /F /IM gbrain.exe /T"`
2. `gbrain serve --http`(background)
3. `curl http://127.0.0.1:3131/health` → 期望 `{"status":"ok","engine":"pglite"}`
4. `gbrain doctor --only harness_wiring --json` → claude-code + codex 双 healthy

## 必交付(按优先级)
① Phase A serve 复活 + 双 harness 验证
② Phase B ingest 168 页 docs mirror + R1-R21 到 brain(验 `gbrain stats --json`)
③ Phase C Anti-Patterns 飞轮观察 1-2 周

## 禁止
- 起第二个 `gbrain serve --http`
- 删 `C:/Temp/gbrain-bootstrap-premint-*.log`
- 跑 `gbrain skillopt` 不先 taskkill
- 在 MSYS2 bash 直接 `taskkill /F /IM`(必须套 PowerShell)
- 伪造 skillopt benchmark 行
- repo `git reset --hard` / `clean -fd`

## 加速模式
- 临时文件一律写 `C:\Temp\`
- Bash ≥80 次就停,改用 Write/Edit

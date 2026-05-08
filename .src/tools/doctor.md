---
description: 只读扫描当前项目的 .pensieve/ 用户数据目录，检查 frontmatter、链接、目录结构、关键种子文件、auto memory 与项目指令文件 Pensieve 短路由对齐情况，输出固定格式报告。
---

# Doctor 工具

> 工具边界见 `.src/references/tool-boundaries.md` | 共享规则见 `.src/references/shared-rules.md` | 目录约定见 `.src/references/directory-layout.md`

## Use when

- 初始化后复检
- 升级后复检
- 迁移后确认 MUST_FIX 清零
- 怀疑 graph、frontmatter、目录结构、memory 指引或 `CLAUDE.md` / `AGENTS.md` 短路由漂移

## 标准执行

> All `.src/` paths below are relative to the skill root (`$PENSIEVE_SKILL_ROOT`, typically `~/.claude/skills/pensieve/`).

```bash
bash "${PENSIEVE_SKILL_ROOT:-$HOME/.claude/skills/pensieve}/.src/scripts/run-doctor.sh" --strict
```

Doctor 只维护：

- `<project>/.pensieve/state.md`（生命周期状态 + Graph）
- 运行时图谱输出如 `.pensieve/.state/pensieve-user-data-graph.md`
- Claude auto memory `~/.claude/projects/<project>/memory/MEMORY.md`

Doctor 会把缺失或漂移的 `CLAUDE.md` / `AGENTS.md` Pensieve 短路由报告为 MUST_FIX，但不会自动修改它们。修复使用 `sync-instructions`。

不会改你的业务代码。

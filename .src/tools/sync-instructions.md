---
description: 将当前项目已有 Pensieve pipeline 写入 CLAUDE.md / AGENTS.md 的短路由块。幂等，不覆盖用户已有内容。
---

# Sync Instructions 工具

> 工具边界见 `.src/references/tool-boundaries.md` | 共享规则见 `.src/references/shared-rules.md`

## Use when

- 用户要求把 Pensieve pipeline 写入 `CLAUDE.md`、`AGENTS.md`、`agent.md` 或项目级 agent 指令文件
- 用户希望新 agent 下次直接知道 commit / refactor / review 该走哪个 pipeline
- 项目已经初始化 `.pensieve/pipelines/`，但入口说明文件缺少短路由

这个工具只写短路由，不生成项目总结，不内联完整 pipeline 内容。

## Failure fallback

- `.src/scripts/sync-instructions.sh` 缺失：停止并报告 skill 安装不完整
- `<project>/.pensieve/pipelines/` 缺失：先执行 `init`
- 目标文件中 Pensieve marker 不成对：停止，要求用户手动修复 marker

## 标准执行

> All `.src/` paths below are relative to the skill root (`$PENSIEVE_SKILL_ROOT`, typically `~/.claude/skills/pensieve/`).

```bash
bash "${PENSIEVE_SKILL_ROOT:-$HOME/.claude/skills/pensieve}/.src/scripts/sync-instructions.sh" --target all
```

如果只想更新已有入口文件，不创建新的 `CLAUDE.md` / `AGENTS.md`：

```bash
bash "${PENSIEVE_SKILL_ROOT:-$HOME/.claude/skills/pensieve}/.src/scripts/sync-instructions.sh" --target auto
```

## 写入内容

写入块只有一个标题和短路由：

```markdown
## How To Use Pensieve

Use `.pensieve/` as the first source of architectural intent.

- `maxims/` are active engineering rules.
- `decisions/` are active project decisions.
- `knowledge/` explains boundary maps and debugging paths.
- `pipelines/` gives executable workflows.

Use these project pipelines directly when trigger words match; do not rediscover them through skills first.

- Commit requests (`commit`, `提交`, `git commit`): use `.pensieve/pipelines/run-when-committing.md`. Check staged diff, decide whether reusable insight should be captured, then make atomic commits.
- Refactor requests (`重构`, `refactor`, `大改`, `拆代码`): use `.pensieve/pipelines/run-when-refactoring.md`. Confirm the real problem, fix upstream data authority first, split large work into 2-3 user-visible steps, delete old paths when new paths work, and avoid compatibility/fallback branches.
- Review requests (`review`, `代码审查`, `检查代码`): use `.pensieve/pipelines/run-when-reviewing-code.md`. Start from git history and changed hot spots, verify candidate issues, and report only high-signal findings with evidence and file locations.
```

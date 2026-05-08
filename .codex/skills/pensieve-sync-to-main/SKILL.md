---
name: pensieve-sync-to-main
description: 将 Pensieve 的中文开发/发布分支同步到英文 main 分支。用户在 Pensieve 仓库中要求同步 zh 或 experimental 到 main、发布英文分支、翻译 Pensieve README/工具规格/模板/SKILL.md，或执行 zh-to-main 发布流程时必须使用。不要用于普通项目本地化或通用 i18n。
---

# Pensieve Sync to Main

只在 Pensieve 仓库自身使用。

这个 skill 负责把 `experimental` 或 `zh` 的变更同步到 `main`，保留贡献者历史，并确保 `main` 是纯英文发布分支。这是维护者发布流程，不是应该播种到每个 Pensieve 用户项目里的默认 pipeline。

## 硬规则

- `main` 必须保持纯英文、发布级。
- 保留源分支 commit 历史；除非用户明确要求，否则不要 squash 掉贡献者历史。
- 语言分支之间的代码逻辑必须一致，只允许文本内容差异。
- 不翻译代码标识符、路径、协议名、命令名、frontmatter key、markdown 链接目标，除非下面的分支规则明确要求。
- 工作区有无关本地修改时，不执行破坏性 git 清理命令。

## 分支策略

- `experimental`：开发分支
- `zh`：中文发布分支
- `main`：英文发布分支

默认源分支是 `zh`，除非用户明确指定 `experimental`。默认远端是 `kingkongshot`。

## 翻译范围

这些文件中的中文内容需要翻译成英文：

- `README.md`
- `SKILL.md`
- `.src/references/*.md`
- `.src/tools/*.md`
- `.src/templates/**/*.md`

这些文件默认直接同步；只有出现英文分支用户可见中文时才翻译：

- 脚本与代码
- JSON/YAML 配置
- 删除文件
- 二进制资源

## 执行流程

### 1. 确认范围

1. 检查当前状态：
   ```bash
   git branch --show-current
   git status --short
   git remote -v
   ```
2. 确认源分支：
   ```bash
   git fetch kingkongshot
   git diff --stat main..kingkongshot/<source-branch>
   ```
3. 将变更文件分类为：需要翻译、直接同步、删除、二进制/配置。
4. 如果工作区存在无关本地修改，先汇报再切分支。

### 2. 创建同步分支

1. 从最新 `main` 开始：
   ```bash
   git checkout main
   git pull kingkongshot main
   git checkout -b sync/zh-to-main-<date>[-topic]
   ```
2. merge 源分支以保留历史：
   ```bash
   git merge kingkongshot/<source-branch> -X theirs --no-edit
   ```
3. 如果用户明确要求只同步少数文件，使用定向 checkout：
   ```bash
   git checkout kingkongshot/<source-branch> -- <file>
   ```
4. 先解决冲突，再开始翻译。

### 3. 翻译

规则：

- 所有中文 prose 翻译成英文。
- 保持 markdown 结构、frontmatter、代码块、命令语法和 HTML 标签不变。
- 有意用于匹配的双语正则保持不变，例如 `探索减负|Exploration Reduction`。
- 英文分支文档里的安装命令要从 `git clone -b zh` 改为 `git clone -b main`。
- 英文分支语言切换链接应指回中文 README，例如 `[中文 README](...zh...)`。
- 脚本里的中文输出字符串只有在英文分支用户可见时才翻译。

验证：

```bash
grep -rln '[一-龥]' README.md SKILL.md .src docs 2>/dev/null
```

翻译后，结果只能剩下明确故意保留的中文，例如语言切换链接或双语正则。

### 4. 验证、提交、PR

1. 运行当前仓库存在的检查：
   ```bash
   git diff --check
   ```
2. 提交：
   ```bash
   git add -A
   git commit -m "translate: sync <source-branch> to main"
   ```
3. 推送并创建 PR：
   ```bash
   git push kingkongshot sync/zh-to-main-<date>[-topic]
   gh pr create --repo kingkongshot/Pensieve --base main --head sync/zh-to-main-<date>[-topic]
   ```
4. 除非用户明确托管完整发布流程，否则合并 PR 前必须让用户确认。

## 失败处理

- merge 冲突残留：停止，列出冲突文件，直接解决。优先保留源分支内容，然后翻译。
- 翻译后仍有中文：逐条检查，翻译或说明为什么故意保留。
- push 被拒：fetch 后将同步分支 rebase 到最新 `kingkongshot/main`，再重试。
- GitHub CLI 认证失败：汇报准确认证错误，不要换用不明远端。

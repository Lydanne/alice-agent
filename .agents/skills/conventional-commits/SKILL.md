---
name: conventional-commits
description: 在 alice-agent 仓库创建 Git 提交时，确定提交格式、中文说明和暂存范围。
---

# Commit 规范

使用 Conventional Commits：`<type>(<scope>): <中文说明>`，scope 可省略。

- type 按改动选择：feat、fix、docs、refactor、test、build、ci、chore。
- scope 使用实际模块或职责，例如 projects、skills；说明写清具体变化。
- 每个提交对应一个完整任务或可独立交付的阶段，仅暂存相关文件或 hunks，保留无关改动。
- 提交前执行适用验证，并检查 `git diff --cached` 和 `git diff --cached --check`。不提交凭证、运行状态或被忽略的嵌套项目仓库。

示例：`chore(projects): 登记 OpenClaw 插件项目`、`docs(skills): 简化提交与推送流程`。

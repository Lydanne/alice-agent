---
name: git-workflow
description: 在 alice-agent 仓库完成任务交付时，按既有授权自动提交并直接推送到 origin/main。
---

# Git 工作流

日常直接使用 main，不创建主题分支或 PR，不要求每次先 fetch/pull。

1. 用 `git status --short --branch` 确认当前仓库、分支及改动。保留无关工作；当前不是 main 时先核对原因，不强制切换。
2. 完成适用验证，按 [Commit 规范](../conventional-commits/SKILL.md) 暂存并提交本次任务。没有改动时不创建空提交。
3. 确认待推送提交属于已授权工作，然后直接执行 `git push`。本仓库已有 origin/main 上游，无需额外发布流程。
4. 推送成功后报告提交号和验证结果；推送失败时保留本地提交并说明原因。只有非快进拒绝时再 fetch 检查，能安全快进则同步后重试；不强推、不自动 stash、不覆盖他人改动或改写历史。

本仓库已授权每个完成并验证的任务自动 commit + push，无需再次确认。用户当次要求不提交或不推送时遵守。此 Git 自动交付约定不单独授权其他操作；部署、重启和对外消息依据 AGENTS.md 的真实发送者授权与具体任务执行；`projects/` 中的独立仓库按其自身约定交付。

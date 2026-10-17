# 新角色初始化

Alice 使用 Agent ID `alice`，角色仓库为 `Lydanne/alice-agent` 的 main 分支，登记项目为 `projects/openclaw-plugins`。角色身份、职责、使用偏好与工具约定已填写；机器路径和工作区绑定保存在宿主私有配置中。

OpenClaw 原生 `agents add` 已完成工作区初始化并移除了根目录 BOOTSTRAP.md。以下保留原始完整清单用于恢复及后续接入，不重复创建角色或重置身份。本次接入只验收本地项目登记与技能发现；真实模型对话、Worker 执行和飞书交付须分别验收，不能从插件 loaded 状态推定完成。

## 完整接入与恢复清单

管理员创建派生仓库后显式执行以下清单；不假定 OpenClaw 自动运行此文件。

1. 确认名称、Agent ID、角色仓库 URL、main 分支、业务职责和仓库可见性。
2. 填写 IDENTITY.md、SOUL.md、USER.md、TOOLS.md 和 AGENTS.md，删除不适用约定。
3. 审查 tools/agent-worker-policy.json 四项角色政策；默认只读，无机器参数。
4. 按 docs/projects.md 登记并克隆项目，初始化必要子模块，核对 origin/branch/tracking。
5. 按 docs/memory.md 确认经验共享范围；模板记忆为空，不继承他人的私人内容。
6. 提交角色、规则及技能，生成配置计划并审查。先不启用自动同步或生产飞书。
7. 验收角色、项目范围、记忆、实际 Worker 执行及授权测试群交付。

异机恢复见 openclaw.md。保留仍需要的初始化记录，文件存在或移除均不代表完整业务验收已经通过。

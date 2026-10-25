# 2026-10-25 插件迁移验收

## 安装与配置

- 来源：https://github.com/Lydanne/agent-plugins ，main 提交 `15a618bea0eb7c6688203499d118122daf78c2cf`。
- 本地项目：`projects/agent-plugins`；旧 `projects/openclaw-plugins` 保留回退，不再登记为当前项目。
- 三个插件均使用新仓库构建的 npm-pack 产物替换，安装入口与源码构建产物逐一一致。包版本未增加，因此不能只根据版本号判断是否完成替换。
- Worker 删除旧顶层 model、reasoningEffort，迁移为 modelProfiles.default（Codex / gpt-6.1-sol / low）、defaultModelId=default、thinkingLevel=0，cwd 指向新项目。
- 保留 read-only、skillsEnabled=true、projectRouting=agent-repo、asyncCompletion=false，以及原 state/codexHome。未开启飞书异步交付、个人授权或文档交付。
- 原 channels、agents、bindings 与备份逐项比较一致。凭证和宿主配置未进入 Git。
- 私有备份：`~/.openclaw/backups/agent-plugins-20261025-195522/`，含原 openclaw.json 和 migration.patch.json。

## 验证

- Node 24.22.0、pnpm 10.33.2、宿主 OpenClaw 2026.10.4。
- frozen-lockfile 安装、typecheck、build、隔离 test 全部通过：287 + 34 + 301 + 1357 = 1979 项。
- 三插件隔离安装及 runtime inspect 通过，Agent Repo hook 权限补齐后无诊断。
- Alice 配置生成器回归测试通过，项目远端、分支和技能目录核验通过。
- config validate、plugins doctor 通过。授权重启后 readyz HTTP 200。
- 实际 agent_repo list/skills 返回新项目；agent_worker models/projects 返回新版模型目录和新登记。
- 飞书 default 已 configured/running/connected，probe.ok=true、lastError=null。配套 Feishu CLI 1.0.98 和技能索引已就绪。

## 真实 Worker 验收未通过

原 jobId：`c6979349-da40-4a0e-9894-b163031f11f7`，requestId：`plugin-migration-verification-20261025-001`。

只读任务要求读取 cwd、Git HEAD 和 README 标题。Codex 已启动并返回技能确认，但应返回的版本字符串为 129 字符，实际仅 128 字符，缺失一个字符。插件报 `Skill revision acknowledgement mismatch; user task was not started`，job 为 failed，业务读取尚未开始。未关闭技能校验、未重放任务或修改原 job 状态。

完整宿主报告位于：`projects/agent-plugins/.tmp/codex-query/c6979349-da40-4a0e-9894-b163031f11f7/completion-report-99a727e9-703f-44e5-9b5e-6ab44654d8f5.md`。该运行产物不提交。

因此，本次确认安装、配置、服务连接及只读工具查询；不能确认真实 Worker 业务轮次或飞书消息/卡片/报告端到端交付成功。

## 升级注意

安装器支持热替换，旧 Worker 配置会阻止新版 schema 校验。先备份并删除旧字段，安装新版后立即应用模型补丁。运行中旧默认值仍可能影响热加载；此次在配置写入后通过已授权重启完成新实例加载。保留原任务和状态，不以新 run 代替状态未知任务。

仓库另附针对 OpenClaw 2026.10.5 的宿主表单补丁；当前宿主为 2026.10.4，本次未跨版本套用该补丁，也未升级宿主。

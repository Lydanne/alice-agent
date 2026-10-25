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

## 首次真实 Worker 验收未通过（修复前历史）

原 jobId：`c6979349-da40-4a0e-9894-b163031f11f7`，requestId：`plugin-migration-verification-20261025-001`。

只读任务要求读取 cwd、Git HEAD 和 README 标题。Codex 已启动并返回技能确认，但应返回的版本字符串为 129 字符，实际仅 128 字符，缺失一个字符。插件报 `Skill revision acknowledgement mismatch; user task was not started`，job 为 failed，业务读取尚未开始。未关闭技能校验、未重放任务或修改原 job 状态。

完整宿主报告位于：`projects/agent-plugins/.tmp/codex-query/c6979349-da40-4a0e-9894-b163031f11f7/completion-report-99a727e9-703f-44e5-9b5e-6ab44654d8f5.md`。该运行产物不提交。

因此，本次确认安装、配置、服务连接及只读工具查询；不能确认真实 Worker 业务轮次或飞书消息/卡片/报告端到端交付成功。

## 升级注意

安装器支持热替换，旧 Worker 配置会阻止新版 schema 校验。先备份并删除旧字段，安装新版后立即应用模型补丁。运行中旧默认值仍可能影响热加载；此次在配置写入后通过已授权重启完成新实例加载。保留原任务和状态，不以新 run 代替状态未知任务。

仓库另附针对 OpenClaw 2026.10.5 的宿主表单补丁；当前宿主为 2026.10.4，本次未跨版本套用该补丁，也未升级宿主。


## 技能版本确认修复与端到端复验

- 插件修复提交：`86010e0c0829743ecfadb627f5ca169d87e7c45d`，已推送 agent-plugins/main。
- 原因：自由文本确认轮次抄写 129 字符版本串时漏字。现用 Codex `--output-schema` 约束确认回执的 status 与完整 revision；宿主精确比对、刷新禁止工具调用、刷新后版本复查均保留。失败不降级、不自动纠正错误回执、不关闭 skillsEnabled。
- 临时 schema 私有保存并在成功或失败后清理，仅作用于确认轮次，业务轮次不带 schema。
- 新增回归先失败后通过；定向 37 项检查通过；全仓 typecheck、build、隔离 test 通过，共 1980 项。覆盖缺字/错误版本拒绝、取消、刷新中版本变化、禁止工具调用、新建及续接命令参数、临时文件清理。
- npm-pack 重新安装后 Gateway generation 3 热加载成功；runtime inspect 为 loaded 且无 diagnostics，安装的 index.mjs/configuration.mjs/skill-refresh.mjs 与修复源码逐文件相同。
- 本次新验收 jobId：`ebb57476-f3f8-4b53-99dd-7a2398e74253`，requestId：`plugin-migration-verification-20261025-ack-fix-001`。沿用原 topic，旧失败 job 保留。
- 本地 wait/status 协议返回 completed。skillsStatus=catalog_applied，目标版本与已确认版本精确一致（129 字符）。真实 Codex 在只读沙箱执行 pwd、git rev-parse HEAD、head -n 1 README.md，三项退出码均为 0；返回目录为当前新项目、HEAD 为上述修复提交、标题为 # agent-plugins，Main 已独立核对。
- 完整宿主报告：`projects/agent-plugins/.tmp/codex-query/ebb57476-f3f8-4b53-99dd-7a2398e74253/completion-report-a253f101-e9fb-49ef-9973-6cd2e8d9b7ff.md`。运行产物不提交。
- config validate 通过，readyz HTTP 200；飞书仍 running/connected、probe.ok=true、lastError=null。

此次端到端覆盖 Gateway 工具调用 → 项目/技能路由 → 真实 Codex 技能确认 → 业务命令 → 原 job 查询 → 完成报告。未发送飞书消息，飞书卡片与异步报告交付仍不在本次验收范围。


后续已完成真实飞书异步链路及群成员报告访问验收，见 [飞书端到端验收](feishu-e2e-2026-10-25.md)。

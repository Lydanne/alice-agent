# 权限配置部署验收（2026-10-25）

## 结论

权限规则与 Worker 全权限执行模式已部署，Gateway 重启后可达，飞书通道 connected。真实 Worker 验收停在身份授权核验阶段，尚不能判定创建者写入链路通过，更不能判定其他真实发送者的权限隔离通过。

## 已验证

- 配置 validate、角色仓库生成器测试通过；生成补丁保留既有执行模式的回归断言通过。
- 运行中 agent_repo list/skills 成功，明确选择 projects/agent-plugins。
- 本地管理员 RPC 提交一次临时文件创建、读取、删除测试。首次 RPC 等待超时后仅查询原会话，找到原 jobId，没有重复 run。
- 原 jobId：c701d691-3d1e-44e7-b500-12c3cf490633。技能版本确认成功，真实 Codex 返回 danger-full-access / approval_policy=never，插件记录 approvalMode=dangerously-bypass-approvals-and-sandbox。
- Worker 返回 awaiting_main，要求宿主核验的原始发送者身份或本次操作授权证明；它没有把 task/context 中的本地管理员自述当作凭据。未创建或删除测试文件，不能将文件不存在当作删除测试成功。
- 另一次无来源绑定的飞书会话模拟请求返回 tool execution failed，未产生目标文件；该通用错误不足以证明真实非所有者授权隔离。

## 清理与边界

测试结束取消原等待任务，恢复 asyncCompletion=true，保留飞书原有异步模式。原始本地结果位于 .tmp/auth-worker-result.json、.tmp/auth-worker-cancel.json 和 .tmp/auth-forged-source-result.json，不进入 Git。

当前规则是模型层约束，全权限沙箱本身不按发送者隔离。下一步需要将宿主核验的发送者及授权判定作为可信执行上下文交接，再用创建者和受限用户的真实渠道消息分别验证允许与拒绝。不能用正文里的身份声明替代，也不能将本地管理员 RPC 冒充飞书用户请求。本次没有发送飞书测试消息、修改插件源码或执行删除业务数据等操作。

请审查人工门禁 `CP5-CR037-DESIGN-EVIDENCE-BATCH`。

checklist 路径: `process/checkpoints/CP5-CR037.md`
自动预检结论: 14 份设计证据 lld-check 全部 PASS；CP5 IMPLEMENTABILITY 预检 9/9 PASS；无 blocks_lld open item。

审批者摘要:
- 本次确认服务的整体目标: 确认全部 14 份目标 Story 设计证据（P-1 + S1-S11），批准后进入 story-execution（Wave 0-4 开发/验证）。
- 推荐动作: 默认推荐 approve。
- approve 后会发生什么: 回填人工结果，按 Wave 调度 meta-dev 实现（CP6 开发完成 → CP7 验证）。
- approve 不授权什么: 不授权真实设备配置写入（--execute 需 DQ-037-04 独立授权）、凭据读取、外部接口调用、生产数据写入。
- 不确认会阻塞什么: story-execution（开发/验证）推进保持等待。

Context Capsule: `process/context/CP5-CR037-CONTEXT.yaml`（read_profile=compact）。
决策收集覆盖: 14 份设计证据 + 9 份预检 + CP3 讨论日志 + CR-037 评审修订；候选 0，纳入待决策 0。
决策分层:
- 必须用户决策: 0
- 高风险策略确认: 0（真机下发授权已由 DQ-037-04 独立管理）
- agent 默认处理: O-S3-01（实现期对齐 P-1 schema）、S10 层 B 依赖 P-2（未就绪标注风险）
- 仅审计记录: 14 份设计证据 + lld-check + 9 份预检
本轮待人工决策项: 0

本轮待人工决策项: 0。原因: 设计证据批量确认无新增人工取舍；O-S3-01 与 S10 层 B 为 agent 默认处理/风险标注。

如果你回复 approve，表示接受以上 0 项推荐方案（即确认 14 份设计证据作为实现基线），不表示授权以下不授权项。
不授权项: 真实设备配置写入（--execute）、凭据读取、外部接口调用、生产数据写入；以 checkpoint 为准。

请只回复以下三个 exact 选项之一:
- approve
- 修改: <具体修改点>
- reject

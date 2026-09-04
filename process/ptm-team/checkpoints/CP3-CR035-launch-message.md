请审查人工门禁 `CP3-CR035-HLD-REVIEW`。

checklist 路径: `process/checkpoints/CP3-CR035-HLD-REVIEW.md`
自动预检结论: 已生成 Decision Brief；发起前请以 `meta-flow check human-gate --checkpoint process/checkpoints/CP3-CR035-HLD-REVIEW.md` 的结果为准。

审批者摘要:
- 本次确认服务的整体目标: 请见 checkpoint `### 审批者摘要`；本消息只承载发起确认所需摘要。
- 推荐动作: 默认推荐 approve，除非你要求调整范围、风险、授权边界或推荐方案。
- approve 后会发生什么: 接受本轮 Decision Brief 中列出的推荐方案，并允许进入 checkpoint 声明的下一阶段。
- approve 不授权什么: 不授权真实运行、凭据读取、安全边界变更、外部接口调用、数据写入、publish、live / 交易类操作。
- 不确认会阻塞什么: 当前 checkpoint 后续推进会保持等待，直到你回复 approve / 修改 / reject。

Context Capsule: 请见 checkpoint 的 `### Context Capsule Summary`，其中包含 capsule、read_profile、默认读取策略和全文档读取边界。
决策收集覆盖: 请见 checkpoint 的 `### Decision Collection Coverage`，本消息只承载发起确认所需摘要。
决策分层:
- 必须用户决策: 1
- 高风险策略确认: 以 checkpoint `### 决策分层` 为准。
- agent 默认处理: 低风险、可逆、局部实现细节默认不进入用户主确认表，完整清单以 checkpoint 为准。
- 仅审计记录: 已按项目规则处理且无需用户拍板的事项只保留审计摘要。
本轮待人工决策项: 1
blocking / high-risk 决策摘要: DQ-035-06

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-035-06 | implementation | AGA-2：dry-run 模式 tg_config_interface 是否 skip（CP2 DQ-035-04 表述歧义澄清）。fw_login dry-run 实际不 skip（L1474 `and not dry_run`），"与 fw_login dry-run 一致"应理解为不 skip | A. dry-run 不 skip，仅 --execute config-once（与 fw_login 严格一致）：每个用例 build_command 展示完整命令，不置标志 | B. dry-run 首次 build_command 后置标志后续 skip（模拟 --execute skip 计数） | 推荐与 fw_login（config-once 既定类比 Gotcha#8）行为一致，认知成本最低；dry-run 产物每用例展示完整 tg_config_interface 命令便于审查；不置标志逻辑简单。B 与 fw_login 不一致，需 dry-run 维护标志，config-once skip 计数 dry-run/--execute 不同，S05 测试断言复杂 | 影响 HLD §6.2 判断表 + S03 execute_steps skip 守卫 + S05 dry-run 测试断言；低风险（dry-run 行为可逆） |

如果你回复 approve，表示你接受以上 1 项推荐方案，不表示授权以下不授权项。
不授权项: 不授权真实运行、凭据读取、安全边界变更、外部接口调用、数据写入、publish、live / 交易类操作；若 checkpoint 中列有额外不授权项，以 checkpoint 为准。

请只回复以下三个 exact 选项之一：
- approve
- 修改: <具体修改点>
- reject

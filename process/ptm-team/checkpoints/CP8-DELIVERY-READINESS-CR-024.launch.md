请审查人工门禁 `CP8-DELIVERY-READINESS-CR-024`。

checklist 路径: `process/checkpoints/CP8-DELIVERY-READINESS-CR-024.md`
自动预检结论: 已生成 Decision Brief；发起前请以 `meta-flow check human-gate --checkpoint process/checkpoints/CP8-DELIVERY-READINESS-CR-024.md` 的结果为准。

审批者摘要:
- 本次确认服务的整体目标: 请见 checkpoint `### 审批者摘要`；本消息只承载发起确认所需摘要。
- 推荐动作: 默认推荐 approve，除非你要求调整范围、风险、授权边界或推荐方案。
- approve 后会发生什么: 接受本轮 Decision Brief 中列出的推荐方案，并允许进入 checkpoint 声明的下一阶段。
- approve 不授权什么: 不授权真实运行、凭据读取、安全边界变更、外部接口调用、数据写入、publish、live / 交易类操作。
- 不确认会阻塞什么: 当前 checkpoint 后续推进会保持等待，直到你回复 approve / 修改 / reject。

Context Capsule: 请见 checkpoint 的 `### Context Capsule Summary`，其中包含 capsule、read_profile、默认读取策略和全文档读取边界。
决策收集覆盖: 请见 checkpoint 的 `### Decision Collection Coverage`，本消息只承载发起确认所需摘要。
决策分层:
- 必须用户决策: 2
- 高风险策略确认: 以 checkpoint `### 决策分层` 为准。
- agent 默认处理: 低风险、可逆、局部实现细节默认不进入用户主确认表，完整清单以 checkpoint 为准。
- 仅审计记录: 已按项目规则处理且无需用户拍板的事项只保留审计摘要。
本轮待人工决策项: 2
blocking / high-risk 决策摘要: CP8-DQ-01, CP8-DQ-02

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| CP8-DQ-01 | scope | ptm-te v1 交付就绪确认（关闭 CR-024） | **approve**：批准 ptm-te v1 交付（static 验证 PASS，安装稳定），关闭 CR-024 | A: 有条件批准（要求补充 runtime）/ B: reject 暂不交付 | static 12/12 PASS + 三平台 dry-run PASS + 映射一致性校验 PASS；runtime 留 follow-up 是 CP2 DQ-01 dry-run 默认门预期 | 选 A: 延迟交付；选 B: ptm-te 无法使用 |
| CP8-DQ-02 | risk_acceptance | runtime 未覆盖风险接受（R-RUNTIME-01 + R-O-08） | **accept**：接受 runtime 未覆盖风险，留 T-01（端到端验证）+ T-06（O-08 确认）follow-up | A: reject（要求先完成 runtime 再交付） | runtime 需设备 + --execute 授权，独立 CR 推进；static 已覆盖映射正确性 | 选 A: 阻塞交付，需先约设备 + 授权 |

如果你回复 approve，表示你接受以上 2 项推荐方案，不表示授权以下不授权项。
不授权项: 不授权真实运行、凭据读取、安全边界变更、外部接口调用、数据写入、publish、live / 交易类操作；若 checkpoint 中列有额外不授权项，以 checkpoint 为准。

请只回复以下三个 exact 选项之一：
- approve
- 修改: <具体修改点>
- reject

请审查人工门禁 `CP2-CR163-TRIAL-LINEAGE-INSTRUMENTATION-SCOPE`。

checklist 路径: `process/checkpoints/CP2-CR163-TRIAL-LINEAGE-INSTRUMENTATION-SCOPE.md`
自动预检结论: 已生成 Decision Brief；发起前请以 `meta-flow check human-gate --checkpoint process/checkpoints/CP2-CR163-TRIAL-LINEAGE-INSTRUMENTATION-SCOPE.md` 的结果为准。

审批者摘要:
- 本次确认服务的整体目标: 请见 checkpoint `### 审批者摘要`；本消息只承载发起确认所需摘要。
- 推荐动作: 默认推荐 approve，除非你要求调整范围、风险、授权边界或推荐方案。
- approve 后会发生什么: 接受本轮 Decision Brief 中列出的推荐方案，并允许进入 checkpoint 声明的下一阶段。
- approve 不授权什么: 不授权真实运行、凭据读取、安全边界变更、外部接口调用、数据写入、publish、live / 交易类操作。
- 不确认会阻塞什么: 当前 checkpoint 后续推进会保持等待，直到你回复 approve / 修改 / reject。

Context Capsule: 请见 checkpoint 的 `### Context Capsule Summary`，其中包含 capsule、read_profile、默认读取策略和全文档读取边界。
决策收集覆盖: 请见 checkpoint 的 `### Decision Collection Coverage`，本消息只承载发起确认所需摘要。
决策分层:
- 必须用户决策: 3
- 高风险策略确认: 以 checkpoint `### 决策分层` 为准。
- agent 默认处理: 低风险、可逆、局部实现细节默认不进入用户主确认表，完整清单以 checkpoint 为准。
- 仅审计记录: 已按项目规则处理且无需用户拍板的事项只保留审计摘要。
本轮待人工决策项: 3
blocking / high-risk 决策摘要: DQ-CP2-CR163-001, DQ-CP2-CR163-002, DQ-CP2-CR163-003

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP2-CR163-001 | scope | 是否批准 CR163 以 2 条 producer chains / 4 mappings、5 个 Story 目标覆盖 family lifecycle、automatic lineage、seal/validator 与现有 gate integration？ | 批准当前基线；S03 单 Story 覆盖全部 4 mappings。 | 修改范围后重跑 CP1/CP2；或拒绝并将 CR163 退回 candidate。 | 推荐方案覆盖已确认绕过点且不增加 Story；缩减会留下 lineage gap，扩大则混淆 producer/consumer。 | 决定 CP3 架构边界、Story owner 和 100% coverage 分母。 |
| DQ-CP2-CR163-002 | scope | 是否批准 raw-lineage claim ceiling：完整 instrumented future run 才可 present，uninstrumented 为 typed_unavailable，invalid/tampered 为 blocked；C1 仍不可计算？ | 批准；effective count/ref/method 继续 unavailable/empty，CR155 保持 blocked。 | 将统计计算并入本 CR；或维持所有路径 typed_unavailable。 | 推荐方案实现事实源而不制造统计过度声明；合并统计会扩大方法和验证风险；全 unavailable 无法达成 CR163 价值。 | 决定 admission consumer 语义和后续 FU-002 边界。 |
| DQ-CP2-CR163-003 | security | 是否批准 deny-default 权限与 standard 路由：CP2 后只进入设计，任何实现/真实数据/runtime/外部写入仍受后续门禁与独立授权约束？ | 批准 architecture-major CP3→CP4→CP5→CP6→独立 CP7→CP8 路由。 | 暂停在 CP2；或拆出独立 runtime-authorization CR 后再评估。 | 推荐方案允许离线 contract/fixture 设计继续且不放宽权限；暂停无风险但阻塞价值；混入 runtime 授权会扩大审计边界。 | 决定是否可启动 meta-se-critical；不会授权实现或运行。 |

如果你回复 approve，表示你接受以上 3 项推荐方案，不表示授权以下不授权项。
不授权项: 不授权真实运行、凭据读取、安全边界变更、外部接口调用、数据写入、publish、live / 交易类操作；若 checkpoint 中列有额外不授权项，以 checkpoint 为准。

请只回复以下三个 exact 选项之一：
- approve
- 修改: <具体修改点>
- reject

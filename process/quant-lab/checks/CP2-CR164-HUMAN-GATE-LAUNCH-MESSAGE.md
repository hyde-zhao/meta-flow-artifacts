请审查人工门禁 `CP2-CR164-MULTIPLE-TESTING-PBO-DSR-SCOPE`。

checklist 路径: `process/checkpoints/CP2-CR164-MULTIPLE-TESTING-PBO-DSR-SCOPE.md`
自动预检结论: 已生成 Decision Brief；发起前请以 `meta-flow check human-gate --checkpoint process/checkpoints/CP2-CR164-MULTIPLE-TESTING-PBO-DSR-SCOPE.md` 的结果为准。

审批者摘要:
- 本次确认服务的整体目标: 请见 checkpoint `### 审批者摘要`；本消息只承载发起确认所需摘要。
- 推荐动作: 默认推荐 approve，除非你要求调整范围、风险、授权边界或推荐方案。
- approve 后会发生什么: 接受本轮 Decision Brief 中列出的推荐方案，并允许进入 checkpoint 声明的下一阶段。
- approve 不授权什么: 不授权真实运行、凭据读取、安全边界变更、外部接口调用、数据写入、publish、live / 交易类操作。
- 不确认会阻塞什么: 当前 checkpoint 后续推进会保持等待，直到你回复 approve / 修改 / reject。

Context Capsule: 请见 checkpoint 的 `### Context Capsule Summary`，其中包含 capsule、read_profile、默认读取策略和全文档读取边界。
决策收集覆盖: 请见 checkpoint 的 `### Decision Collection Coverage`，本消息只承载发起确认所需摘要。
决策分层:
- 必须用户决策: 4
- 高风险策略确认: 以 checkpoint `### 决策分层` 为准。
- agent 默认处理: 低风险、可逆、局部实现细节默认不进入用户主确认表，完整清单以 checkpoint 为准。
- 仅审计记录: 已按项目规则处理且无需用户拍板的事项只保留审计摘要。
本轮待人工决策项: 4
blocking / high-risk 决策摘要: DQ-CP2-CR164-001, DQ-CP2-CR164-002, DQ-CP2-CR164-003, DQ-CP2-CR164-004

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP2-CR164-001 | scope | 是否批准四方法 MVP 与 conservative no-OR-pass claim ceiling？ | 批准 BH + WRC/SPA + PBO/CSCV + DSR；任何 claim-relevant mandatory FAIL/BLOCKED/unavailable 均不能被其他 PASS 覆盖。 | A. 缩减为 BH-only 后重跑 CP1/CP2；B. 暂停 CR164，保持全部统计证据 typed_unavailable。 | 推荐方案闭合 CR154 Gate-1 四类证据并保持严格 fail-closed；BH-only 较小但无法支持 robustness/Sharpe/data-snooping claims；暂停最保守但无当前价值。 | 决定 CP3 method/schema 边界、Story 数与 positive claim 条件。 |
| DQ-CP2-CR164-002 | scope | 是否批准 method-specific minima、10 项 QAC，以及 raw-count DSR/effective-count ceiling？ | 批准当前阈值与 QAC；`effective_trial_count` 保持 typed_unavailable，DSR schema 明确 raw-count provenance/non-alias。 | A. 采用更严格的 10-candidate/8-split/60-sample floor 后重跑 CP1/CP2；B. 延后 DSR 与 effective-count 相关 claims。 | 推荐阈值可验证且不制造 effective-count 过度声明；更严格方案减少假信心但增加 unavailable；延后 DSR 缩小价值。 | 决定数值边界、fixture 分母、CP7 量化出口和 deflated-performance claim ceiling。 |
| DQ-CP2-CR164-003 | scope | 是否批准 UC-58 实现、UC-59/60 compatibility-only、五个产品规划候选及三个 deferred 项？ | 批准；ML/event 无相同 sealed inputs 时 fail closed，不实现 real adapters；effective estimator 和 real recomputation 保持 deferred。 | A. 把 fixture/static ML/event adapters 纳入本轮并重跑 CP1/CP2；B. 删除 ML/event compatibility contract，仅保留 multifactor。 | 推荐方案防止 schema 分叉且控制实现范围；扩大 adapters 增加跨模块负担；删除 compatibility 会制造未来漂移。 | 决定 CP3 consumer boundary、正式 Story 拆解与 deferred backlog。 |
| DQ-CP2-CR164-004 | security | 是否批准 deny-default 权限与 architecture-major 路由：CP2 后只进入设计，CP5 前不实现，任何真实统计/data/runtime/external/write 操作仍需独立授权？ | 批准 CP3→CP4→CP5→CP6→fresh independent CP7→CP8 路由；当前仅允许设计。 | A. 暂停在 CP2；B. 另起 runtime-authorization CR 后再评估真实运行，但不混入当前 gate。 | 推荐方案允许本地设计推进且不放宽权限；暂停无风险但阻塞价值；混入真实运行会扩大审计与安全边界。 | 决定是否可启动 `meta-se-critical`；不授权代码实现或运行。 |

如果你回复 approve，表示你接受以上 4 项推荐方案，不表示授权以下不授权项。
不授权项: 不授权真实运行、凭据读取、安全边界变更、外部接口调用、数据写入、publish、live / 交易类操作；若 checkpoint 中列有额外不授权项，以 checkpoint 为准。

请只回复以下三个 exact 选项之一：
- approve
- 修改: <具体修改点>
- reject

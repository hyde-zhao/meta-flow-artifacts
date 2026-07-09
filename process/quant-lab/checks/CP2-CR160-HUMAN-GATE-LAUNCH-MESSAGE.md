请审查人工门禁 `CP2-CR160-STAGE4-OBSERVATION-REVIEW-WORKFLOW-SCOPE`。

checklist 路径: `process/checkpoints/CP2-CR160-STAGE4-OBSERVATION-REVIEW-WORKFLOW-SCOPE.md`
自动预检结论: 已生成 Decision Brief；发起前请以 `meta-flow check human-gate --checkpoint process/checkpoints/CP2-CR160-STAGE4-OBSERVATION-REVIEW-WORKFLOW-SCOPE.md` 的结果为准。

审批者摘要:
- 本次确认服务的整体目标: 请见 checkpoint `### 审批者摘要`；本消息只承载发起确认所需摘要。
- 推荐动作: 默认推荐 approve，除非你要求调整范围、风险、授权边界或推荐方案。
- approve 后会发生什么: 接受本轮 Decision Brief 中列出的推荐方案，并允许进入 checkpoint 声明的下一阶段。
- approve 不授权什么: 不授权真实运行、凭据读取、安全边界变更、外部接口调用、数据写入、publish、live / 交易类操作。
- 不确认会阻塞什么: 当前 checkpoint 后续推进会保持等待，直到你回复 approve / 修改 / reject。

Context Capsule: 请见 checkpoint 的 `### Context Capsule Summary`，其中包含 capsule、read_profile、默认读取策略和全文档读取边界。
决策收集覆盖: 请见 checkpoint 的 `### Decision Collection Coverage`，本消息只承载发起确认所需摘要。
决策分层:
- 必须用户决策: 5
- 高风险策略确认: 以 checkpoint `### 决策分层` 为准。
- agent 默认处理: 低风险、可逆、局部实现细节默认不进入用户主确认表，完整清单以 checkpoint 为准。
- 仅审计记录: 已按项目规则处理且无需用户拍板的事项只保留审计摘要。
本轮待人工决策项: 5
blocking / high-risk 决策摘要: DQ-CP2-CR160-EVIDENCE-PROFILE, DQ-CP2-CR160-CR155-SEED-CLASSIFICATION, DQ-CP2-CR160-DELIVERABLE-SHAPE, DQ-CP2-CR160-AUTHORIZATION-BOUNDARY, DQ-CP2-CR160-CP4-CP5-ROUTE

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP2-CR160-EVIDENCE-PROFILE | scope | Observation review 的输入是 contract-level evidence、real-data evidence，还是双轨？这决定 CR157/CR158 contract artifact 和 CR155 real-data fail-closed 样例能否被同一 workflow 审查。 | 双轨 fail-closed：`contract_only` lane 只能输出 `contract_review_complete` / `blocked` / `needs_real_data_evidence` 等低等级结论；`real_data_validated` lane 仍需 admission 非阻断才可进入 observation review。 | A. contract-only；B. real-data only；C. 暂缓定义 evidence profile，只写 checklist。 | 推荐方案同时覆盖 contract artifact 和真实数据反例，并防止 contract lane 输出 `paper_candidate=true` / `simulation_ready`；A 无法消费 CR155 反例；B 会排除 CR157/CR158 contract artifact；C 会让 Stage 4 gate 继续语义不完整。 | 影响 observation review input contract、decision table、后续验证矩阵和 no-overclaim 规则。 |
| DQ-CP2-CR160-CR155-SEED-CLASSIFICATION | scope | CR155 的 `READY_WITH_RISK` artifact closure、`BLOCKED` admission package、`FAIL` statistical gate、`paper_candidate=false` 应如何进入 CR160？ | 将 CR155 分类为 `blocked_admission_failed`，只作为 fail-closed 标准样例，不作为 observation candidate。 | A. 暂不使用 CR155 样例；B. 误用 CR155 为 observation candidate；C. 将 CR155 仅作为文字风险说明，不进入 decision table。 | 推荐方案利用 CR155 的确定性 rerun + 正确阻断证据验证 fail-closed；A 浪费最强反例；B 会越权提升 blocked artifact；C 可验证性不足。 | 防止 READY_WITH_RISK 被误读为 admission ready，并要求 CP7 复现该分类。 |
| DQ-CP2-CR160-DELIVERABLE-SHAPE | implementation | CR160 交付设计文档、代码，还是两者？这决定是否触发 Story/LLD/CP6 实现路径。 | 纯设计交付：`docs/design/HLD-STAGE4-OBSERVATION-REVIEW-WORKFLOW.md`、`docs/quality/OBSERVATION-REVIEW-CHECKLIST.md`、`process/checks/CR160-CR155-SEED-CLASSIFICATION.md` 和产品基线更新；不新增代码。 | A. 增加 `engine/observation_review.py` schema/static checker + fixture tests；B. 只更新产品需求，不做 HLD/checklist；C. 暂缓 CR160。 | 推荐方案先定义语义，风险最低且符合 Stage 4 缺口性质；A 会扩大到代码实现并需重算 CP4/CP5/CP6；B 无法交付 gate contract；C 保持 CR157 语义缺口。 | 影响 CP3/CP7 交付物、是否需要 Story decomposition、是否需要 implementation dispatch。 |
| DQ-CP2-CR160-AUTHORIZATION-BOUNDARY | runtime_authorization | CR160 是否授权 simulation/paper/runtime 或新的 real lake 操作？同时如何解释 `scope_authz_consistency=NEEDS_REVIEW` 中 CR155 real lake evidence 与 no-new-lake-read 的张力？ | 不授权任何新操作。CR160 只消费 CR155 既有 real lake validation evidence 作为 fail-closed 样例；“消费既有 evidence”不等于“授权新 lake 读取”。 | A. 另起 runtime authorization CR；B. 另起 real-data validation CR；C. 暂停使用 CR155 evidence，避免 real-data 字样。 | 推荐方案边界最清楚且保留 CR155 样例价值；A/B 属于不同高风险目标；C 会降低 fail-closed 验证质量但可作为保守退路。 | 防止 observation review 被误读为 paper/simulation/live/real-data execution authorization。 |
| DQ-CP2-CR160-CP4-CP5-ROUTE | implementation | 纯设计路径下 CP4/CP5/CP6 如何处理，设计交付 evidence 归属哪个 CP？ | 纯设计：CP4 N/A、CP5 N/A、CP6 N/A；设计交付物在 CP7 前作为验证输入记录。CP7 通过 meta-dev/meta-qa design review 或批准的 inline-fallback 验证 contract completeness、checklist testability、CR155 classification 和 no-overclaim wording。 | A. 强制所有设计走 Story/LLD/CP6；B. 静默跳过 CP4/CP5/CP6；C. 改为代码路径并启用标准 CP4/CP5/CP6。 | 推荐方案与 route plan 和无代码事实一致；A 增加无意义过程负担；B 不符合审计规则；C 只有选择 schema/checker 代码实现时合理。 | 影响 route plan、agent dispatch evidence、CP7 验证输入和 CP8 release readiness。 |

如果你回复 approve，表示你接受以上 5 项推荐方案，不表示授权以下不授权项。
不授权项: 不授权真实运行、凭据读取、安全边界变更、外部接口调用、数据写入、publish、live / 交易类操作；若 checkpoint 中列有额外不授权项，以 checkpoint 为准。

请只回复以下三个 exact 选项之一：
- approve
- 修改: <具体修改点>
- reject

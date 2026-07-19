请审查重新发起的人工门禁 `CP2-CR172-REQUIREMENTS-BASELINE`。

checklist 路径: `process/checkpoints/CP2-CR172-REQUIREMENTS-BASELINE.md`
自动预检结论: `PASS`；上一轮评审要求的 UC-58 业务视角整改已完成，Context Pack Check 与 Human Gate Check 均需在本次发起前通过。

审批者摘要:
- 本次确认服务的整体目标: 冻结 PATH-B/C/A 路由、五字段授权前提、effective-trial 审批边界、C1 降级、C2/C3 治理、E1/OI-005 与 joint approval 契约。
- 用户业务锚点: 研究负责人需要为策略 X 获得真实 multiple-testing / overfit 证据；fixture/static evidence 不能作为成熟准入依据，未来 C1 real typed evidence 用于研究评估和后续 mature SAP 证据基础，但不直接产生 admission PASS。
- 整改结果: 业务动机、用户痛点、使用价值、业务触发 `4/4 PASS`；SGQ `2/2`；REQ/场景/矩阵 `8/8/8`；8 个正式 DQ 仍为 `OPEN`。
- 推荐动作: 当前五字段具体值为 `0/5` 且 data owner identity 为 `OPEN`，直接 `approve` 推荐 PATH-B；只有同时提供 5/5 具体值与 owner，才条件推荐 PATH-C。
- approve 后会发生什么: 当前证据下只转入 FU-CR164-004 独立离线 estimator 前置；CR-172 activation deferred，不进入 CP3。五字段可冻结后必须恢复 CR-172 并重开 CP2。
- approve 不授权什么: 不授权真实 lake/NAS/credential/env/provider 读取、任何 write、C1-C4 computation/binding、runtime、aggregate、CR155、trading、publish/deploy 或 Git remote write。
- 不确认会阻塞什么: CR-172 保持 CP2 pending，不得进入 CP3、Story、LLD、实现或真实 evidence activation。

Context Capsule: 请见 checkpoint 的 `### Context Capsule Summary`，其中包含 capsule、read_profile、默认读取策略和全文档读取边界。
决策收集覆盖: 请见 checkpoint 的 `### Decision Collection Coverage`，本消息只承载发起确认所需摘要。
决策分层:
- 必须用户决策: 8
- 高风险策略确认: 4
- agent 默认处理: 0
- 仅审计记录: 3
本轮待人工决策项: 8
blocking / high-risk 决策摘要: DQ-CR172-001, DQ-CR172-002, DQ-CR172-003, DQ-CR172-004, DQ-CR172-005, DQ-CR172-006, DQ-CR172-007, DQ-CR172-008

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CR172-001 | runtime_authorization | 五字段具体值与授权 data owner 是否可冻结？ | `not_freezable_currently` → PATH-B | 提供 5/5 具体值 + owner identity | 推荐不伪造授权，代价是 activation 延后；备选可解锁 C1-first，但必须给出真实、有限、可审计值。 | 任一字段缺失/模糊仍批准 PATH-C 会造成授权逃逸。 |
| DQ-CR172-002 | scope | FU-CR164-004 是否独立？ | `separate_methodology_cr` | 全部 merge 条件满足时 `joint_merge` | 独立 CR 审批/回滚最清晰；合并可减少 CR，但混合方法学与数据授权域。 | 审批人或风险边界不同会使联合范围失真。 |
| DQ-CR172-003 | implementation | C1 是否支持 effective count 不可用降级？ | `typed_unavailable_required` | `block_c1_until_estimator` | 推荐可先验证 C1 通道且 truthful fail-closed；备选耦合 estimator、进度慢。 | 禁止把 `raw_trial_count` 重标为 effective count；否则 evidence 不可信。 |
| DQ-CR172-004 | risk_acceptance | 最终采用 PATH-B/C/A？ | 当前 `PATH-B`；DQ-001=5/5+owner 时条件推荐 `PATH-C` | `PATH-A` 或 `reject` | PATH-B 最安全但不完成 activation；PATH-C blast radius 最小；PATH-A CR 少但首次归因/回滚更难。 | PATH-C/A 与五字段条件不一致时门禁无效。 |
| DQ-CR172-005 | follow_up_tracking | E1 何时触发？ | `before_later_admission_pass_or_pass_with_risk` | 提前到 CR-172 CP8 | 推荐与 CR-171 定义一致，不让 design/activation CP8 无谓阻塞；提前更保守但语义错误。 | 延后越过 admission 决策会违反 verifier fence。 |
| DQ-CR172-006 | scope | OI-CR171-005 是否排除？ | `independent_revalidation_audit_lane` | 扩入 CR-172 | 推荐保持 audit 与 activation 的审批/证据边界独立。 | 扩入会混合 classification、历史审计和新数据授权。 |
| DQ-CR172-007 | follow_up_tracking | PATH-C 后 C2/C3 如何治理？ | `two_independent_runtime_high_risk_crs`，总 activation CR=`3` | `same_parent_sequential_slices` | 独立 CR 的 owner、回滚、证据和风险接受最清晰；slice 数少但更耦合。 | 模糊“后续 CR 或 slice”会使总 CR 数与审批边界不可估。 |
| DQ-CR172-008 | risk_acceptance | 不同 owner 合并时如何记录联合审批？ | `dual_owner_same_revision_hash_ledger_with_intersection` | estimator 保持独立，不启用联合审批 | 推荐可机械验证，且权限取更严格交集；独立方案最清晰但增加 CR。 | partial approval、hash mismatch 或 revision 变化会导致授权被宽松解释。 |

如果你回复 approve，表示你接受以上 8 项当前推荐方案，其中 DQ-001=`not_freezable_currently`、DQ-004=`PATH-B`；不表示 activation 完成，也不表示授权以下不授权项。
不授权项: 真实 lake/NAS/credential/env/provider 读取、任何 write、C1-C4 computation/binding、runtime、aggregate、CR155、trading、publish/deploy 或 Git remote write。

请只回复以下三个 exact 选项之一：
- approve
- 修改: <具体修改点>
- reject

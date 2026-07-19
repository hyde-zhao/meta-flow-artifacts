---
status: completed-for-cp2-resubmission
cr_id: CR-172
stage: requirement-clarification
owner: meta-pm
nickname: pm-wu
created_at: "2026-07-16T10:06:12+08:00"
context_ref: process/context/CP2-CR172.context.json
updated_at: "2026-07-16T11:29:49+08:00"
---

# CR172 CP2 Scenario Discussion Log

## 用户真实意图

用户认可 CR172 整改稿的核心结构，要求补齐 PATH-B 恢复链、PATH-C 后 C2/C3 的治理归属和 effective-trial joint approval 记录机制，并在整改后推进到下一个人工门禁。用户明确表达：五字段可冻结时，倾向 PATH-C（C1-first）而不是把 PATH-A 作为默认。

该表达确认了 CP2 推荐方向，不等于确认五字段具体值、owner identity、最终 PATH、三 producer blast-radius 风险接受或真实数据授权。

在 CP2 业务视角评审中，用户进一步确认：UC-58 必须先回答研究负责人为什么需要真实证据、fixture/static evidence 为什么不能支撑成熟准入、未来 C1 real typed evidence 将如何被研究员使用；“策略 X 需要真实数据证据”应成为旅程第 0 步。业务价值是主锚点，PATH、blast radius、五字段和审批合同是安全实现约束。该确认同样不改变 8 个 OPEN DQ 或任何授权事实。

在随后 CP2 最终评审中，用户确认“策略 X”只应作为 CP2 业务示例，不是当前已批准策略身份；PATH-B estimator 与具体策略无关；只有未来 PATH-C/A activation 才在 CP3 Entry 冻结 `strategy_id + strategy_name`，并在 CP6 C1 typed evidence 中锚定同一身份。该收敛是既有范围的可审计身份契约，不新增第 9 个 DQ。

## Scenario Gray Areas

| Gray Area | 问题 | 用户输入 / 处理 | 状态 | CP2 影响 |
|---|---|---|---|---|
| SGA-CR172-001 | 五字段可冻结时首次 activation 半径 | 用户倾向 PATH-C；整改基线采用 C1-first 默认。 | recommendation-resolved / final-open | DQ-001、DQ-004 |
| SGA-CR172-002 | PATH-B 完成后是否还需要 activation | 用户要求明示衔接；已冻结为“前置而非替代”。 | resolved-contract / formal-open | DQ-001、DQ-002 |
| SGA-CR172-003 | PATH-C 后 C2/C3 的治理形态 | 用户指出独立 CR 与 sibling slice 不同；默认改为两个独立 runtime-high-risk CR，总 activation CR 数预计为 3。 | resolved-recommendation / final-open | DQ-007 |
| SGA-CR172-004 | 不同 owner 的 joint approval 如何成立 | 用户要求明确记录方式；已定义同 revision/hash 双 ledger、approval group 和风险边界取交集。 | resolved-contract / acceptance-open | DQ-002、DQ-008 |

## 用户可见场景确认交互

| 字段 | 内容 |
|---|---|
| question_id | SGQ-CR172-001 |
| question | 五字段可冻结时，首次真实数据 activation 默认采用 PATH-C、PATH-A、PATH-B 前置还是暂缓？ |
| options | A PATH-C C1-first；B PATH-A C1-C3；C PATH-B estimator 前置；D 暂不 activation。 |
| recommendation | A PATH-C；先验证授权合同→run identity→PIT/lineage→typed evidence→回滚全链，降低首次 blast radius。 |
| user_response | 用户明确表示“五字段可冻结时，倾向 PATH-C（C1-first）而非 PATH-A 作为默认”，并要求整改后推进到下一个人工门禁。 |
| confirmed_understanding | CP1/CP2 baseline 将 PATH-C 作为默认推荐；不把该倾向伪装为最终 PATH、五字段值、owner 或风险授权，8 个 DQ 仍由 CP2 人工确认。 |
| impact_surface | scope、runtime authorization、validation、follow-up CR count、rollback |
| status | confirmed-for-cp2-submission |

| 字段 | 内容 |
|---|---|
| question_id | SGQ-CR172-002 |
| question | UC-58 是否应以研究负责人对策略 X 的真实 multiple-testing / overfit 证据需求为旅程起点，并显式说明 fixture 局限与未来 C1 evidence 的使用价值？ |
| options | A 以业务触发为第 0 步、治理作为实现约束；B 继续以 CP2 路径选择作为旅程起点。 |
| recommendation | A；先冻结业务问题和用户价值，再讨论有限授权与 activation 半径。 |
| user_response | 用户评审明确要求补齐业务动机、用户痛点、使用价值，并在现有旅程前增加“策略 X 需要真实数据证据”的第 0 步。 |
| confirmed_understanding | 未来 C1 real typed evidence 用于真实 multiple-testing / overfit 评估并为 mature SAP 提供证据基础，但本身不构成 admission PASS/PASS_WITH_RISK；授权和 claim ceiling 不变。 |
| impact_surface | user value、journey、scope、validation、gate |
| status | confirmed-for-cp2-resubmission |

| 字段 | 内容 |
|---|---|
| question_id | SGQ-CR172-003 |
| question | “策略 X”是当前已批准真实策略，还是 CP2 业务示例，并把真实身份冻结与 evidence 锚定留给未来 activation gate？ |
| options | A 仅为占位；PATH-B 策略无关；PATH-C/A 在 CP3 冻结身份、CP6 锚定 evidence；B 当前 CP2 直接指定策略身份。 |
| recommendation | A；避免把业务示例误读为已授权目标策略，同时为未来 evidence 建立可验证身份链。 |
| user_response | 用户自由文本评审要求明确策略占位契约、PATH-B 策略无关，并将 `strategy_id + strategy_name` 冻结与 C1 evidence 锚定分别设为未来 CP3/CP6 义务。 |
| confirmed_understanding | 当前不批准、不推断策略身份；未来身份必须非空、无通配符、可审计，并与 five-field scope、run identity、PIT/lineage 一致，缺失或不一致 fail-closed。 |
| impact_surface | scope、identity、traceability、validation、gate |
| status | confirmed-for-cp2-resubmission |

## Deferred Ideas

| ID | 内容 | 延后原因 | 重启条件 |
|---|---|---|---|
| DEF-CR172-001 | C2 独立 real-evidence activation | 首次 activation 推荐只做 C1。 | C1 CP7 通过且独立五字段/owner/rollback 获批。 |
| DEF-CR172-002 | C3 独立 real-evidence activation | 同上，且 C3 数据/方法风险需单独审查。 | 独立 runtime-high-risk CR。 |
| DEF-CR172-003 | C4 rework / authorization | C4 当前是 incompatible，不是 insufficient。 | 独立 C4 CR。 |
| DEF-CR172-004 | OI-CR171-005 revalidation audit | 不属于 activation。 | 独立 audit authorization lane。 |

## CP2 Decision Batch

| DQ | 类型 | 推荐 | 备选 / 切换条件 | 状态 |
|---|---|---|---|---|
| DQ-CR172-001 | runtime_authorization | data owner 冻结 5/5 精确有限值；不能则 PATH-B。 | 保持未启动，直至字段可冻结。 | OPEN-CP2 |
| DQ-CR172-002 | scope / ownership | FU-CR164-004 默认独立。 | 全部 merge conditions 满足才合并。 | OPEN-CP2 |
| DQ-CR172-003 | implementation contract | 必须支持 effective count typed-unavailable 降级。 | 不支持则阻断 activation 设计。 | OPEN-CP2 |
| DQ-CR172-004 | risk_acceptance | 默认 PATH-C，不接受隐式三 producer blast radius。 | 显式接受后可 PATH-A；字段不可冻则 PATH-B。 | OPEN-CP2 |
| DQ-CR172-005 | follow_up_tracking | E1 只在后续 admission maturity action 前触发。 | 若解释变化，先回退重审 waiver。 | OPEN-CP2 |
| DQ-CR172-006 | scope | OI-005 保持独立 audit lane。 | 纳入将扩大审批域，需另行 CR 重构。 | OPEN-CP2 |
| DQ-CR172-007 | follow_up_tracking | C2/C3 各自独立 CR，总 activation CR 数 3。 | 同 revision/审批/风险/回滚 + C1 CP7 后可顺序 slice。 | OPEN-CP2 |
| DQ-CR172-008 | risk_acceptance | 不同 owner 合并须双 ledger、同 revision/hash、风险交集。 | 不接受则拆分 estimator。 | OPEN-CP2 |

## 交还判定

- 用户可见 SGQ：`3/3` 已回答并复述确认；`SGQ-CR172-002..003` 均来自用户 CP2 自由文本评审，不重复询问。
- UC-58 业务视角：业务动机、用户痛点、使用价值、业务触发 `4/4` 明示。
- 策略占位契约：placeholder、CP3 identity freeze、CP6 evidence anchor、mismatch fail-closed=`4/4`；PATH-B strategy-agnostic=`1/1`。
- Scenario Gray Areas：`4/4` 已处理，正式决策仍由 CP2 承担。
- CP1 阻断问题：`0`。
- CP2 待人工决策：`8`。
- 真实读取、凭据、provider、写入、runtime/trading：`0/0/0/0/0`。
- 建议下一步：Host Orchestrator 保持 8 个 DQ 不变，刷新并重新发起同一 CP2 人工门禁。

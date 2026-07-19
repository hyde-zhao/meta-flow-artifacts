---
checkpoint_id: "CP2-CR172-REQUIREMENTS-BASELINE"
checkpoint_name: "CR172 需求、场景与范围基线自动预检"
type: "auto_precheck"
status: "PASS"
owner: "meta-pm"
created_at: "2026-07-16T10:06:12+08:00"
checked_at: "2026-07-16T11:29:49+08:00"
target:
  phase: "requirement-clarification"
  story_id: ""
  artifacts:
    - docs/product/REQUIREMENTS.md
    - docs/product/SCENARIOS.yaml
    - docs/product/MVP-SCOPE.md
manual_checkpoint: "process/checkpoints/CP2-CR172-REQUIREMENTS-BASELINE.md; host-orchestrator-owned; prior review changes_requested; not modified by meta-pm"
---

# CP2 CR172 需求、场景与范围基线自动预检

## Entry Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| CP1 通过 | PASS | `process/checks/CP1-CR172-USE-CASE-COMPLETENESS.result.json` | 无 CP1 blocker。 |
| 需求草案存在 | PASS | `docs/product/REQUIREMENTS.md#cr172-stage-3-real-evidence-activation-requirements` | 8 项 P0，ready_for_design=false。 |
| 工程场景与矩阵存在 | PASS | `docs/product/SCENARIOS.yaml`, `docs/product/TEST-MATRIX.md` | 8/8 覆盖。 |
| 产品规划输入存在 | PASS | `docs/product/STORY-MAP.md`, `docs/product/MVP-SCOPE.md`, `docs/product/RELEASE-SLICES.md`, `docs/product/BACKLOG.md` | outcome-only，不拆 Story。 |
| 用户可见确认存在 | PASS | `process/checks/CP2-CR172-DISCUSSION-CHECKPOINT.json` | SGQ-CR172-001..003 共 3/3 confirmed。 |

## Checklist

| # | 检查项 | 状态 | 证据 | 处理意见 |
|---:|---|---|---|---|
| 1 | 功能需求完整 | PASS | `REQ-CR172-001..008` | 8/8 均有量化 AC。 |
| 2 | 非功能需求量化 | PASS | `REQ-CR172-006`, `REQ-CR172-008` | 审计、权限、零操作和 claim ceiling 可检验。 |
| 3 | 范围清晰 | PASS | `docs/product/MVP-SCOPE.md#cr172-mvp-scope` | In/Out/Deferred 完整。 |
| 4 | 验收标准明确 | PASS | `docs/product/REQUIREMENTS.md` | 每条 P0 有数值/布尔失败行为。 |
| 5 | 约束、依赖与风险 | PASS | `process/discussions/CP2-CR172-SCENARIO-DISCUSSION-LOG.md` | 方法学 owner、data owner、五字段、C1 CP7 前置明确。 |
| 6 | 需求冲突 | PASS | `process/plans/CR172-STAGE3-REAL-EVIDENCE-ACTIVATION-PHASE-A-SCOPE-REMEDIATION-2026-07-16.yaml` | PATH-B/C/A 通过条件路由消除默认冲突。 |
| 7 | 变更机制 | PASS | `process/changes/summaries/CR-172.summary.json` | CR171 保留关闭基线，CR172 增量更新。 |
| 8 | 追溯矩阵 | PASS | `docs/product/TEST-MATRIX.md` | UC→REQ→SC→planned test 8/8。 |
| 9 | Gray Areas / Deferred | PASS | discussion log / checkpoint | 4/4 gray areas 与 4 deferred 均隔离。 |
| 10 | 用户可见场景确认 | PASS | `SGQ-CR172-001..003` | PATH-C 倾向、业务视角与策略占位整改均被确认，但不冒充最终 DQ 或策略身份审批。 |
| 11 | 六类工程场景 | PASS | `docs/product/SCENARIOS.yaml` | positive/negative/boundary/permission/failure-recovery/precheck 均覆盖。 |
| 12 | MVP 可确认 | PASS | 八个产品文档 | 业务价值是主锚点；PATH-C 推荐、PATH-B/A 备选与 rollback 作为实现约束保持清晰。 |
| 13 | 人工 DQ 完整 | PASS | `DQ-CR172-001..008` | 每项有推荐、备选、风险、切换条件。 |
| 14 | 授权不扩张 | PASS | `REQ-CR172-008` | CP2 前零真实行为；CP2 approve 不自动授权实现。 |
| 15 | Story 拆分阻断 | PASS | `docs/product/STORY-MAP.md#cr172-product-planning-outcomes非正式-story` | 仅 outcome，不创建正式 Story。 |
| 16 | 策略占位与未来 evidence anchor | PASS | `UC-58-CR172`, `SGQ-CR172-003` | “策略 X”不是已批准身份；PATH-B strategy-agnostic；未来 CP3/CP6 身份链与 mismatch fail-closed=`4/4`。 |

## CP2 Decision Brief 输入

### 审批者摘要

- 用户真实意图：研究负责人需要为策略 X 获得可用于真实 multiple-testing / overfit 评估的 C1 typed evidence，并为后续 mature SAP 提供证据基础；fixture/static evidence 只验证契约/流程，不能作为成熟准入依据。
- 推荐动作：当前五字段仍为 `0/5` 且 data owner identity=`OPEN`，因此当前直接 `approve` 只应选择 PATH-B；未来补齐 `5/5+owner` 后默认推荐 PATH-C，PATH-A 仍需显式三 producer blast-radius 风险接受。
- approve 后：当前事实下只进入 FU-CR164-004 独立离线前置，CR172 activation 保持 deferred；五字段可冻结后恢复 CR172 并重开 CP2。不会自动执行读取、计算或实现。
- approve 不授权：credential/provider/lake/NAS write、catalog pointer、QMT/trading、publish/deploy、C2/C3/C4、aggregate、OI-005。
- 不确认会阻塞：CR172 solution-design、Story 规划和任何 activation 实现。
- 非新增 DQ 的身份契约：“策略 X”只为业务占位；PATH-B 不要求策略身份；未来 PATH-C/A activation CP3 Entry 冻结非空、无通配符、可审计的 `strategy_id + strategy_name`，CP6 C1 evidence 必须携带同一身份并与 five-field scope、run identity、PIT/lineage 一致，否则 fail-closed。

### 待人工决策清单

| DQ | 类型 | 待确认 | 推荐 | 备选与优劣 | 风险 / 回退 |
|---|---|---|---|---|---|
| DQ-CR172-001 | runtime_authorization | 五字段具体值与 data owner 是否可冻结 | 5/5 精确有限值；不可冻结则 PATH-B | 暂缓 activation 最安全但无进展；PATH-B 可先降方法学风险 | 缺/模糊值即 BLOCKED，回 requirement-clarification |
| DQ-CR172-002 | scope | FU-CR164-004 是否独立 | 默认独立 | 合并可少一个 CR，但会混合审批域 | 任一 merge condition 不满足即拆分 |
| DQ-CR172-003 | implementation | C1 typed-unavailable 降级 | 必须支持 | 不支持会让 estimator 阻塞通道 | 不支持则停止 CP3，回需求整改 |
| DQ-CR172-004 | risk_acceptance | 最终 PATH / blast radius | PATH-C C1-first | PATH-A CR 少但首次半径大；PATH-B 无真实数据但不是 activation | 共享 blocker 难归因时回 PATH-C；字段不可冻回 PATH-B |
| DQ-CR172-005 | follow_up_tracking | E1 触发时点 | 后续 admission PASS/PASS_WITH_RISK 前 | 提前到 CR172 CP8 会无谓阻塞；延后会越过 verifier fence | 语义变化则重审 FU-006 waiver |
| DQ-CR172-006 | scope | OI-005 是否排除 | 独立 audit lane | 纳入会混合审计/数据授权与 activation | 需 classification 时另起 audit CR |
| DQ-CR172-007 | follow_up_tracking | C2/C3 独立 CR 还是顺序 slice | 两个独立 runtime-high-risk CR，总 activation CR=3 | 顺序 slice 文件少，但审批/回滚耦合 | 仅同 revision/owner/risk/rollback + C1 CP7 后切换 |
| DQ-CR172-008 | risk_acceptance | joint approval 合同 | 双 owner 同 revision/hash ledger，风险取交集 | 不接受双 ledger 则保持 estimator 独立 | partial/mismatch/revision change 均使批准失效 |

## Exit Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| 自动预检无 FAIL | PASS | 本文件 Checklist | 16/16 PASS。 |
| 人工决策包完整 | PASS | `DQ-CR172-001..008` | 8 个 DQ 语义与 OPEN 状态未变，可由 Host Orchestrator 刷新既有 checkpoint。 |
| 人工确认完成 | N/A | host-orchestrator-owned | 上次人工评审为 changes_requested；整改后仍待重新发起和批准，不能进入 CP3。 |

## Deliverables

| 交付物 | 路径 | 状态 | 说明 |
|---|---|---|---|
| CP1 result | `process/checks/CP1-CR172-USE-CASE-COMPLETENESS.result.json` | PASS | 机器真相源。 |
| CP2 自动预检 | `process/checks/CP2-CR172-REQUIREMENTS-BASELINE.result.json` | PASS | 不替代人工 checkpoint。 |
| Return summary | `process/handoffs/CR172-CP2-META-PM-UC-REMEDIATION-RETURN-SUMMARY.md` | PASS | Host 刷新同一 CP2 人工门禁的聚合输入。 |

## 结论

- 自动预检结论：`PASS`
- 人工门状态：`pending re-launch / host-orchestrator-owned`；先前评审为 `changes_requested`，本轮不修改人工 checkpoint。
- 阻断项：`0` 个 CP1/自动预检 blocker；`8` 个必须人工决策项。
- 下一步：Host Orchestrator 消费整改 return summary，刷新并重新发起同一 CP2 Decision Brief / checkpoint。

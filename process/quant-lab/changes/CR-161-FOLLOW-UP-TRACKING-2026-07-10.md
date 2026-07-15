---
tracking_id: "CR-161-FOLLOW-UP-TRACKING-2026-07-10"
cr_id: "CR-161"
status: "candidate-tracking"
created_at: "2026-07-10T06:20:54+08:00"
owner: "host-orchestrator"
---

# CR161 Follow-up Tracking

| Candidate ID | Title | Status | Activation Condition |
|---|---|---|---|
| `FU-CR161-001` | Research-engine trial lineage instrumentation | closed (`CR-163`) | CR-163 已关闭；formal CR: `process/changes/CR-163.md`；为未来原生 instrumented runs 提供 lineage foundation。 |
| `FU-CR161-002` | Multiple testing / PBO / DSR computable evidence implementation | closed (`CR-164`) | CR-164 completed CP2-CP8 and closed `READY_WITH_RISK` on 2026-07-12; formal CR: `process/changes/CR-164.md`. |
| `FU-CR161-003` | Walk-forward / OOS evidence producer foundation | closed (`CR-166`) | CR-166 于 2026-07-13 经 CP8 批准以 `READY_WITH_RISK` 关闭；只交付 fixture/static producer foundation，不连接真实数据湖，Stage 3 未启动。 |
| `FU-CR161-004` | Economic cost / slippage / impact evidence producer foundation | closed (`CR-168`) | CR-168 于 2026-07-14 经 CP8 批准以 `READY_WITH_RISK` 关闭；交付 fixture/static-only C3 foundation，不授权真实 TCA、C4 计算或 Stage 3。 |
| `FU-CR161-005` | C4 capacity / liquidity / ADV evidence producer foundation | closed (`CR-169`，CP8 approved，READY_WITH_RISK) | 5/5 Story、CP6/CP7、Stage 2 合同 7/7、CP8 人工批准及提交后 repository suite 2159/0 已完成。真实 ADV/liquidity、canonical Gate 4 全局硬化、aggregate、Stage 3 与 CR-155 promotion 仍须独立 CR 和授权。 |
| `FU-CR161-006` | Independent CP7 verifier-lane resilience | candidate | A later high-risk CR needs to rely on CR161 CP7, or the CP8 verifier-independence waiver expires. |
| `FU-CR161-007` | Canonical cross-strategy reliability N/A semantics and admission hardening | closed (`CR-170`，CP8 approved，READY_WITH_RISK) | `CR-170` 已关闭 canonical-hardening 子切片；Gate 1-5 N/A 语义和 Gate 6 admission policy 已交付，aggregate 与 CR155 regression 仍由 `FU-CR161-009` 承接。 |
| `FU-CR161-008` | Alpha-decay evidence ownership and C2-adjacent method evaluation | candidate | CR169 CP3 已确认 alpha-decay 不进入 C4 v1；需要冻结预测衰减 owner、OOS/C2 边界、输入窗口与 schema version 时再独立启动。 |
| `FU-CR161-009` | C1-C4 aggregate integration and CR155 regression (former FU-007 aggregate slice) | candidate | `CR-170` 完成 canonical hardening 后，再通过独立 CR 接入 aggregate orchestration、最终 StrategyAdmissionPackage 与 CR155 regression；不得自动 promotion。 |
| `FU-CR161-010` | Stage 3 Launch / Real-Lake Entry Decision Gate | active (`CR-171`, CP0/CP1 in progress) | 已完成范围评审并固化在 `process/plans/STAGE3-LAUNCH-DECISION-MEMO-2026-07-15.yaml`；`CR-171` 已由用户明确授权创建。CP2 前不授予 lake-read、real computation、Stage 3 start、aggregate、CR155 promotion 或 runtime。 |

当前无 active formal CR；所有剩余 candidate 均未启动，启动任一 candidate 仍需独立 CR、CP0 冲突预检和明确授权。

## Structured activation record

```yaml
follow_up_items:
  - id: CR-170
    title: Canonical cross-strategy reliability N/A semantics and admission hardening
    kind: requirement-change
    status: closed
    lifecycle_status: closed
    readiness_status: READY_WITH_RISK
    gate_status: cp8_closed
    gate_profile: architecture-major
    formal_cr_path: process/changes/CR-170.md
    blocked_by: ""
    next_action: "Closed under CR-170 CP8 approval as READY_WITH_RISK. Stage3 Launch、FU-006、FU-008 与 FU-009 均保持独立候选，须另立 CR 和明确授权。"
  - id: FU-CR161-001
    title: Research-engine trial lineage instrumentation
    kind: implementation-gate
    status: closed
    lifecycle_status: closed
    readiness_status: READY_WITH_RISK
    gate_status: cp8_closed
    gate_profile: architecture-major
    formal_cr_path: process/changes/CR-163.md
    blocked_by: ""
    next_action: "Closed under CR163; real historical backfill and real ML/event runners remain separately authorized work."
  - id: FU-CR161-002
    title: Multiple testing / PBO / DSR computable evidence implementation
    kind: implementation-gate
    status: closed
    lifecycle_status: closed
    readiness_status: READY_WITH_RISK
    gate_status: cp8_closed
    gate_profile: architecture-major
    formal_cr_path: process/changes/CR-164.md
    blocked_by: ""
    next_action: "Closed under CR164 CP8 approval; follow-up boundaries are tracked in process/changes/CR-164-FOLLOW-UP-TRACKING-2026-07-12.md."
  - id: FU-CR161-003
    title: Walk-forward / OOS evidence producer foundation
    kind: implementation-gate
    status: closed
    lifecycle_status: closed
    readiness_status: READY_WITH_RISK
    gate_status: cp8_closed
    gate_profile: architecture-major
    formal_cr_path: process/changes/CR-166.md
    related_cr: CR-166
    blocked_by: ""
    next_action: "Closed under CR166 CP8 approval; real data, runtime and Stage 3 execution remain separately authorized work."
  - id: FU-CR161-004
    title: Economic cost / impact evidence producer
    kind: implementation-gate
    status: closed
    lifecycle_status: closed
    readiness_status: READY_WITH_RISK
    gate_status: cp8_closed
    gate_profile: architecture-major
    formal_cr_path: process/changes/CR-168.md
    related_cr: CR-168
    blocked_by: ""
    next_action: "Closed under CR168 CP8 approval as READY_WITH_RISK. Real TCA, C4 capacity/liquidity, canonical global Gate4/aggregate work and Stage 3 remain separately authorized follow-up scope."
  - id: FU-CR161-005
    title: C4 capacity / liquidity / ADV evidence producer foundation
    kind: implementation-gate
    status: closed
    lifecycle_status: closed
    readiness_status: ready_with_risk
    gate_status: cp8_closed
    gate_profile: architecture-major
    formal_cr_path: process/changes/CR-169.md
    related_cr: CR-169
    related_active_cr: ""
    blocked_by: ""
    next_action: "Closed under CR-169 CP8 approval as READY_WITH_RISK. FU-006/FU-007/FU-008、真实 C4 与 Stage 3 保持独立候选，须经正式 CR 与明确授权启动。"
  - id: FU-CR161-006
    title: Independent CP7 verifier-lane resilience
    kind: ledger-maintenance
    status: candidate
    lifecycle_status: candidate
    readiness_status: not_ready
    gate_status: not_started
    gate_profile: process-lite
    formal_cr_path: ""
    blocked_by: "A later high-risk verification route requires it"
    next_action: "Activate only through a separate CR and explicit authorization."
  - id: FU-CR161-007
    title: Canonical cross-strategy reliability N/A semantics and admission hardening
    kind: implementation-gate
    status: closed
    lifecycle_status: closed
    readiness_status: READY_WITH_RISK
    gate_status: cp8_closed
    gate_profile: architecture-major
    formal_cr_path: process/changes/CR-170.md
    related_cr: CR-170
    related_active_cr: ""
    source_slice: canonical-hardening
    blocked_by: ""
    next_action: "Closed under CR-170 CP8 approval as READY_WITH_RISK. aggregate/CR155 regression 已拆为 FU-CR161-009，且不会自动启动或 promotion。"
  - id: FU-CR161-008
    title: Alpha-decay evidence ownership and C2-adjacent method evaluation
    kind: architecture-realignment
    status: candidate
    lifecycle_status: candidate
    readiness_status: not_ready
    gate_status: not_started
    gate_profile: architecture-major
    formal_cr_path: ""
    blocked_by: "CR-169 C4 v1 deliberately excludes alpha-decay; owner, OOS/C2 relation, method assumptions and schema evolution require a separate architecture decision."
    next_action: "Activate only through a separate formal CR, CP0 conflict precheck and explicit user authorization; do not add an alpha-decay calculator to CR-169."
  - id: FU-CR161-009
    title: C1-C4 aggregate integration and CR155 regression
    kind: implementation-gate
    status: candidate
    lifecycle_status: candidate
    readiness_status: not_ready
    gate_status: not_started
    gate_profile: architecture-major
    formal_cr_path: ""
    parent_candidate: FU-CR161-007
    former_slice_label: FU-CR161-007b
    blocked_by: ""
    next_action: "After CR-170 closes, activate only through a separate formal CR, CP0 conflict precheck and explicit user authorization; preserve CR155 BLOCKED until aggregate regression is independently approved."
  - id: FU-CR161-010
    title: Stage 3 Launch / Real-Lake Entry Decision Gate
    kind: runtime-authorization
    status: active
    lifecycle_status: active
    readiness_status: not_ready
    gate_status: cp2_pending
    gate_profile: runtime-high-risk
    formal_cr_path: process/changes/CR-171.md
    related_active_cr: CR-171
    proposed_cr_id: CR-171
    decision_memo_ref: process/plans/STAGE3-LAUNCH-DECISION-MEMO-2026-07-15.yaml
    risk_alias_policy_ref: process/policies/RISK-ID-ALIASES.yaml
    blocked_by: "related_active_cr=CR-171; CP0 conflict precheck and CP1 product-baseline refresh must pass; CP2 route/verifier/read-scope approval remains mandatory."
    next_action: "related_active_cr=CR-171; CR-171 is active. Do not treat its creation or CP0/CP1 progress as lake-read, real-computation, Stage 3, aggregate, CR155 or runtime authorization."
```

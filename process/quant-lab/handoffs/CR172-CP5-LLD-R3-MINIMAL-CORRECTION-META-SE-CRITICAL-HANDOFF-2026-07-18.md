---
handoff_id: "CR172-CP5-LLD-R3-MINIMAL-CORRECTION"
change_id: "CR-172"
checkpoint: "CP5"
canonical_role: "meta-se"
codex_agent_name: "meta-se-critical"
reasoning_profile: "critical"
context_ref: "process/context/CP5-CR172-PATH-I-LLD-CONTEXT.yaml"
r2_findings_ref: "process/checks/CP5-CR172-LLD-REVIEW-R2-FINDINGS.md"
status: "ready"
---

# CR-172 CP5 LLD R3 最小架构整改

只修改 HLD/ADR/Feature Matrix、三个 CR172 Feature、五张 Story 卡与 Development Plan；不得修改 LLD、源码、测试、fixture、state、ledger、checkpoint。

按 R2 findings 机械关闭四项，不申请新权限：

1. 当前 v1 `approved_ledger` 必须固定双 false，reason=`APPROVED_LEDGER_ADAPTER_UNAVAILABLE`；caller 自报枚举不能解锁真实 action。未来可信 issuer/envelope/adapter 仅由独立 runtime-high-risk CR 引入。
2. S04 仍只能从 S03 selected replica 取数，但允许 verifier-library dependency：staging port 必须返回精确 sealed bundle + selection，S04 调用 S02 唯一 verifier（直接 library dependency 或 S03 窄 facade）完成 bytes-level 原 seal复验；禁止绕过 S03 selection，也禁止第二 digest。
3. 删除 S05 `evidence_kind=fixture` 第二来源；唯一来源为 `ActionDecisionV1.decision_origin` + `ActionScopeContextV1.target_kind` + fixture URI/port binding。
4. REQ-013 明确为 `contract_ready/runtime_enforcement_deferred`；在既有 future native-producer 前置下增加 path-enforcement 子前置，要求未来 launch/workspace first side effect 前消费 `RunPathDecisionV1`，证明 new default=1、legacy write=0。当前 runner diff=0，不得在 CP8 宣称 runtime delivered。

同步版本与修订记录，Story 状态改为 `lld-rework-required-r3`，Development Plan CP5 batch=`lld-r3-required`。写 return summary `process/handoffs/CR172-CP5-LLD-R3-MINIMAL-CORRECTION-META-SE-CRITICAL-RETURN-SUMMARY.md`，报告精确修改文件与 LLD R3 输入。

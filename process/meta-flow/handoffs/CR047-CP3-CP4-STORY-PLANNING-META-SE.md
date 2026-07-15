# CR-047 CP3 → CP4 Story Planning Handoff

## Dispatch

- canonical_role: `meta-se`
- mode: `inline-fallback`
- dispatch_id: `IF-CR047-CP3-CP4-STORY-PLANNING`
- dispatch_trigger: `CP3 approved; route plan requires CP4 story planning`
- approved_by: `user`
- fallback_reason: 用户明确要求“不使用子agent，继续推进”，并在 CP3 R2 后批准自动推进到下一人工门禁。
- platform_attestation: `unavailable`

## Context policy

- context_ref: `process/context/CP4-CR047-STORY-PLANNING-CONTEXT.yaml`
- read_profile: `compact`
- do_not_read/write: prelink backup、`process/quant-lab/**`、凭据/runtime/SaaS、CR-046 protected originals。

## Assigned outcome

以已批准 HLD/ADR 为约束，生成 5 个 Feature 设计包、7 张 Story 卡、单一 `DEVELOPMENT-PLAN.yaml`、无环 DAG、LLD/开发/QA Waves，并执行 CP4 自动预检。不得开始源码实现或声称独立 meta-se attestation。

## Exit

- Feature evidence: 5/5 DESIGN + TEST-PLAN + TASKS。
- Story evidence: 7/7 Story cards；7/7 `full-lld` 路由。
- DAG: invalid refs=0, cycles=0。
- CP4 result: PASS 才可进入 CP5 设计证据准备。

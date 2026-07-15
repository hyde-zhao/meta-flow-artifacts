# CR-047 CP5 → CP6 Implementation Handoff

## Dispatch

- canonical_role: `meta-dev`
- mode: `inline-fallback`
- codex_agent_name: `meta-dev`
- reasoning_profile: `default`
- dispatch_trigger: `CP5 approved; execute dependency-controlled CR-047 Stories`
- dispatch_id: `IF-CR047-CP5-CP6-IMPLEMENTATION`
- approved_by: `user`
- approved_at: `2026-07-15T12:05:17Z`
- fallback_reason: 用户此前明确要求“不使用子agent，继续推进”，本轮又批准 CP5 并要求推进到下一个人工门禁。
- tool_name: `host-orchestrator-inline`
- platform_attestation: `unavailable`

## Context contract

- context_ref: `process/context/CP6-CR047-IMPLEMENTATION-CONTEXT.yaml`
- read_profile: `compact`
- Story truth: `process/DEVELOPMENT-PLAN.yaml`
- approved design batch: `process/checkpoints/CP5-CR047-ALL-STORIES-LLD-BATCH.md`

## Assignment

按 DAG 依次实现 ST-WT-001～007，逐 Story 生成 IMPLEMENTATION、Return Packet、Evidence Index 与 CP6 result。ST-WT-007 在其实现前生成对象身份 manifest，并在 CP6 完成时验证；任何受保护原件变化立即阻断。

## Authorization boundary

允许修改批准的源码、测试、规则、文档和 CR-047 过程证据；不允许修改 prelink backup、`process/quant-lab/**`、CR-046 protected originals，不允许读取凭据、外部 runtime/SaaS/production write/publish/trading，也不允许 commit/push 或声称独立 Agent/platform receipt。

## Exit

7/7 CP6 result 可校验，Return/Evidence 契约通过，blocking implementation finding=0，Story 可进入 CP7。

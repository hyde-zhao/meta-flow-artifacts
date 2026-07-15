# CR-047 CP4 → CP5 LLD Handoff

## Dispatch

- canonical_role: `meta-dev`
- mode: `inline-fallback`
- dispatch_id: `IF-CR047-CP4-CP5-LLD`
- approved_by: `user`
- fallback_reason: 用户明确要求不使用子 Agent，并批准自动推进到下一个人工门禁。
- platform_attestation: `unavailable`

## Context contract

- context_ref: `process/context/CP5-CR047-LLD-CONTEXT.yaml`
- read_profile: `minimal`
- batch: `CR047-all-stories`，7/7 full-lld。
- clarification broker: Host Orchestrator；当前 blocking items=0。

## Assignment

为 ST-WT-001..007 生成 0–14 章节 full LLD，建立文件影响、数据/API、失败路径、安全、测试、TASK、回滚和量化 DoD。不得修改源码、不得确认 LLD、不得生成 ST-WT-007 CP6 manifest。

## Exit

7/7 `lld-check` PASS；clarification blocking=0；CP5 aggregate result PASS；只开放人工 CP5，不进入实现。

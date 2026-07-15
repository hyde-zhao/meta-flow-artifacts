# CR-047 CP6 → CP7 Verification Handoff

## Dispatch

- canonical_role: `meta-qa`
- mode: `inline-fallback`
- dispatch_id: `IF-CR047-CP6-CP7-VERIFICATION`
- approved_by: `user`
- fallback_reason: 用户明确要求不使用子 Agent，并批准连续推进至下一个人工门禁。
- platform_attestation: `unavailable`

## Context and assignment

- context_ref: `process/context/CP7-CR047-VERIFICATION-CONTEXT.yaml`
- CP6 evidence: `process/evidence/ST-WT-001..007.CP6.index.json`
- 执行七个 Story 的设计契约、负向 fixture、全量 pytest、Ruff、guardrail、Doctor、CR tracking、三平台 dry-run 和 CR-046 protected hash 再验。
- 每个 Story 输出 CP7 verify packet、return、evidence index 和 result；没有独立 meta-qa/runtime receipt 时结论最高为 `PASS_WITH_RISK`。

## Boundaries and exit

不触碰 backup、quant-lab、凭据、外部 runtime/SaaS/production write/publish/trading、CR-046 protected originals，也不 commit/push。七个 CP7 结果可校验、blocker=0 且 release 资料完成后，进入 CP8 `READY_WITH_RISK` 人工门。

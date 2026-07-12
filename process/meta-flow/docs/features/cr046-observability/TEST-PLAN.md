---
status: draft
version: "1.0"
feature_id: "FEAT-EI-OBSERVABILITY"
---

# Feature Test Plan: Replay, Audit and Cost Observability

| Case | 覆盖 | 期望 |
|---|---|---|
| CT-OBS-01 | measured/proxy/unavailable互斥 | 无source measured 100% reject |
| CT-OBS-02 | CR/phase/attempt/checker attribution | 聚合与golden oracle一致 |
| CT-OBS-03 | as-executed/current replay | 双结果+diff+input hashes保留 |
| CT-OBS-04 | null provenance R1 | 原hash不变；strict非fully replayable |
| CT-OBS-05 | event/attempt/thread/outcome/token audit | 五维计数独立且准确率100% |
| CT-OBS-06 | PC-01..19 platform fixtures | 合法接受/非法拒绝率100% |
| CT-OBS-07 | A→B切换与退化 | 条件全满足才B；任一退化回A |
| CT-OBS-08 | `MIG-EI-03-LEGACY-CODEX-AGENT-NAME-D3` | legacy name=>self-declared-unverifiable；resolved fields unavailable |

真实平台 telemetry/receipt unavailable时允许 NOT_RUN，但必须与 mock/fixture PASS分开报告。

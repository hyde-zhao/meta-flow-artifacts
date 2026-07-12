---
status: draft
version: "1.0"
feature_id: "FEAT-EI-OBSERVABILITY"
related_stories: ["ST-EI-005", "ST-EI-006"]
lld_policy_summary: "2 full-lld"
---

# Feature Design: Replay, Audit and Cost Observability

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-12 | meta-se | 冻结 token三态、checker registry、双口径 replay、machine audit、platform conformance与legacy D3分类。 |

## 目标与边界

本 Feature只读 validated evidence snapshot，输出 provenance-bearing derived reports。它拥有 UsageRecord、ReplayRun、AuditReport与 platform conformance checker；不写 canonical ledgers/results/state，也不把估算填入 measured。

## 现有位置与计划面

| 区域 | 路径 | 计划 |
|---|---|---|
| token/context checks | `meta_flow/checks/token_budget.py`, `meta_flow/context_pack/*` | measurement status/attribution adapter |
| replay/audit | 新模块（LLD定名）+ `meta_flow/cli.py` | checker registry、as-executed/current、generated report |
| tests | `tests/test_token_budget.py`, `test_cp_result_event_ledger.py`, 新 CR046 fixtures | golden oracle、PC fixtures、legacy replay |

## 关键合同

- UsageRecord状态互斥：`measured`仅平台 telemetry；`proxy`必须有算法/source；否则`unavailable`。
- ReplayRun保存 checker name/version or commit、schema/policy/input hashes、as-executed/current outcomes与diff class；缺历史provenance标 legacy/unavailable。
- AuditReport分别计数 event rows、attempts、threads、terminal outcomes、token，不从一维推断另一维。
- Platform conformance消费 CORE request/receipt/thread evidence并输出 execution/profile/model三轴。
- legacy `codex_agent_name` 永远分类 D3 `self-declared-unverifiable`；strict replay的 resolved profile/model/effort为 unavailable，不从名字或TOML补值。

## A/B dogfood

A-baseline可验证 repository contract/fixtures，但 runtime conformance NOT_RUN/UNAVAILABLE，CP7最高PASS_WITH_RISK。Conditional-B要求 D0+selector+spawn/reuse receipts、PC-01..19全部通过且新 spawn QA thread；能力退化立即回A。

## Story LLD 合同与 Gotchas

ST-EI-005 LLD下沉 usage schema/source precedence/aggregation；ST-EI-006下沉 checker registry、replay diff、audit schema、PC-01..19、D3 migration classification和A/B判定。不要把 event row count 当 attempt/thread count；不要把 legacy D3自报解析为 runtime事实。

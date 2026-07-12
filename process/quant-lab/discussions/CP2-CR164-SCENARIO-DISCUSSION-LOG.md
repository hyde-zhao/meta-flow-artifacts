---
workflow_id: "CR-164"
phase: "requirement-clarification"
checkpoint: "CP2-discussion"
status: "completed"
owner: "meta-pm"
question_broker: "host-orchestrator"
created_at: "2026-07-12T18:30:00+08:00"
---

# CR164 Scenario Gray Areas Discussion Log

## 用户回答解释（2026-07-12）

用户原文：`批准，继续推进项目`。

Host Orchestrator 根据紧邻的强化推荐上下文，将该回答记录为：`SGQ-CR164-001=A`、`SGQ-CR164-002=A`、`SGQ-CR164-003=A`、`SGQ-CR164-004=A`。meta-pm 复述确认如下：

- MVP 方法集为 BH + WRC/SPA + PBO/CSCV + DSR；claim-relevant mandatory methods 采用保守聚合，禁止 OR-pass，BH PASS + PBO FAIL 不能得到 clean admission PASS。
- `effective_trial_count` 保持 `typed_unavailable`；DSR 使用 sealed raw count 时，CP3 必须输出 `dsr_input_method=raw_trial_count`、lineage provenance 与 effective-count non-alias limitation。
- 采用方法特定最低充分性阈值，并冻结 consolidated Quantitative Acceptance Criteria table。
- UC-58 multifactor 是当前实现主体；UC-59 ML / UC-60 event 仅作为 contract-compatibility consumers，缺同等 sealed-family/statistical inputs 时 fail-closed。
- WRC/SPA 纳入后，CP3 必须冻结 stationary-bootstrap block-length selection（`automatic_politis_romano_1994` 或 `fixed_window`）、参数/seed provenance、deterministic replay 与切换条件。
- 该回答只确认产品范围与方法论边界，不是 CP2 formal gate approval，不授权 runtime/data/credential/NAS/provider/broker/trading/publish/remote-write。

Canonical answer：`process/context/CR164-CP2-SGQ-BATCH.yaml`；ledger：`process/state/QUESTION-LEDGER.ndjson`。

## Entry Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| CR164 已激活且 CP0 通过 | PASS | `process/checks/CP0-CR-164-BOOTSTRAP.result.json` | architecture-major route，CP2 为下一个人工门。 |
| 产品基线必须增量更新 | PASS | `process/changes/summaries/CR-164.summary.json` | 八份产品文档均采用增量修订。 |
| 问题必须经 host relay | PASS | `process/handoffs/CR164-CP2-META-PM-HANDOFF-2026-07-12.md` | meta-pm 不直接向用户提问。 |

## Repository-Fact Inventory

| Fact ID | 当前事实 | 产品影响 | Evidence |
|---|---|---|---|
| RF-CR164-001 | CR163 trusted projection 提供 sealed family ref/hash/raw count；effective count 被强制 typed_unavailable。 | CR164 不得把 raw count 冒充 effective count。 | `engine/experiment_family_lineage.py`; `engine/strategy_admission_statistical_gate.py` |
| RF-CR164-002 | CR151 gate 当前校验 report shape/threshold，但不证明 report 的计算 provenance。 | CR164 必须定义可计算 producer 与 provenance，而不是只填 report。 | `engine/strategy_admission_statistical_gate.py` |
| RF-CR164-003 | CR154 Gate 1 已预留 multiple-testing、FDR/BH、WRC/SPA、PBO/CSCV、DSR 与 trial-count slots。 | 应复用既有 consumer，不建立竞争 gate。 | `engine/cross_strategy_reliability_gates.py` |
| RF-CR164-004 | anomaly helper 已有 Bonferroni/BH，但使用 sorting t-stat 的 normal approximation，并以 Bonferroni OR BH 作为 pass。 | 可复用性留给 CP3；不得直接视为 strategy-family 已批准算法或聚合语义。 | `engine/anomaly_multiple_testing.py` |
| RF-CR164-005 | admission package consumer 只允许状态保持或恶化，并保留 no-runtime flags。 | CR164 evidence 不得提升已有 blocked 状态或改变授权。 | `engine/strategy_admission_package.py` |

## Scenario Gray Areas

| 灰区 | 高价值问题 | 推荐 | 备选与影响 | 状态 |
|---|---|---|---|---|
| SGA-CR164-001 | MVP method set | BH + WRC/SPA + PBO/CSCV + DSR，逐方法 fail-closed。 | BH-only 交付小但不支持 robustness/Sharpe claims；不含 WRC/SPA 会持续阻断 significance wording。 | confirmed-A |
| SGA-CR164-002 | effective-trial boundary | 当前 MVP 保持 typed_unavailable。 | 同切片引入 estimator 会扩大方法/审计范围；延后 DSR 会减少当前用户价值。 | confirmed-A |
| SGA-CR164-003 | quantitative sufficiency | 采用方法特定的保守最低线。 | 更严格阈值增加 unavailable；留给 CP3 会导致 CP2 AC 不可度量。 | confirmed-A |
| SGA-CR164-004 | UC-59/60 compatibility | UC-58 实现；ML/event compatibility-only。 | 同时实现 adapters 扩大跨域验证；完全不兼容会造成 schema 漂移。 | confirmed-A |

## Scenario Confirmation Interactions

| Question ID | Relay 状态 | 推荐 | 用户回答 | 复述确认 | 影响面 |
|---|---|---|---|---|---|
| SGQ-CR164-001 | answered through host relay | A：四类方法均纳入、claim-relevant mandatory methods 保守聚合、无 OR-pass | `批准，继续推进项目` → A | BH PASS + PBO FAIL 不得产生 clean PASS。 | scope / validation / claim ceiling |
| SGQ-CR164-002 | answered through host relay | A：effective count 保持 typed_unavailable | `批准，继续推进项目` → A | DSR raw-count mode 必须显式标识且不别名 effective count。 | scope / methodology / risk |
| SGQ-CR164-003 | answered through host relay | A：方法特定最低线 + consolidated quantitative AC | `批准，继续推进项目` → A | CP2 基线必须包含单一量化 AC 表。 | AC / numerical risk / availability |
| SGQ-CR164-004 | answered through host relay | A：UC-59/60 compatibility-only | `批准，继续推进项目` → A | UC-58 实现；UC-59/60 缺同等输入时 fail-closed。 | compatibility / Story planning |

## Exit Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| 至少一条 SGQ 用户可见回答并复述确认 | PASS | `process/context/CR164-CP2-SGQ-BATCH.yaml` | 4/4 选择 A，复述见本日志。 |
| 方法范围与定量 AC 可冻结 | PASS | SGQ-CR164-001..003 | 四类方法、保守聚合、raw-count ceiling 与量化表已确认。 |
| compatibility scope 可冻结 | PASS | SGQ-CR164-004 | UC-58 implementation；UC-59/60 compatibility-only。 |

## Deliverables

| 交付物 | 路径 | 状态 | 说明 |
|---|---|---|---|
| CR164 discussion log | `process/discussions/CP2-CR164-SCENARIO-DISCUSSION-LOG.md` | PASS | 已建立，可继续追加。 |
| Discussion checkpoint | `process/checks/CP2-CR164-DISCUSSION-CHECKPOINT.json` | PASS | 状态 awaiting-user。 |
| CP1 result | `process/checks/CP1-CR164-MULTIPLE-TESTING-PBO-DSR-USE-CASE-COMPLETENESS.result.json` | pending recheck | 八份基线完成后重检。 |

## Deferred Ideas

- `DF-CR164-001` effective-trial estimator：推荐后续独立方法边界。
- `DF-CR164-002` real ML/event statistical adapters：需真实 runner 与独立授权。
- `DF-CR164-003` real research recomputation / historical migration：当前不授权。

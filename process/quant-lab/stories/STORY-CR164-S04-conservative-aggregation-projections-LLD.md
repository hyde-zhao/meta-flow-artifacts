---
story_id: "CR164-S04-conservative-aggregation-projections"
title: "Conservative aggregation and existing-consumer projections"
story_slug: "conservative-aggregation-projections"
lld_version: "0.1"
tier: "L"
status: "confirmed"
confirmed: true
created_by: "host-orchestrator inline meta-dev"
created_at: "2026-07-12T20:25:00+08:00"
confirmed_by: "user"
confirmed_at: "2026-07-12T20:50:00+08:00"
feature_design_refs: ["docs/features/statistical-evidence-projection/DESIGN.md", "docs/features/statistical-evidence-projection/TEST-PLAN.md", "docs/features/statistical-evidence-projection/TASKS.md", "docs/features/statistical-evidence-contract/DESIGN.md"]
lld_policy: {required_level: "full-lld", trigger_reasons: ["cross-module-contract", "admission-policy", "three-consumers"], rationale: "Final claim semantics cross existing consumers."}
open_items: 0
---

# LLD: CR164-S04 — Conservative aggregation and projections

## 0. 上游设计依据（工程依据）

HLD aggregation table、ADR-005/006、FEAT-24/27、existing CR151/CR154/admission contracts。

## 1. 目标（Goal）

创建 claim-specific severity aggregator，并最小修改三个现有 consumers 以消费 summary projection。

## 2. 需求（Requirements）

Mandatory set immutable in summary；BLOCKED>FAIL>TYPED_UNAVAILABLE>PASS；all mandatory PASS only；orphan ref blocked；status never improves；3/3 projections；UC59/60 compatibility-only。

## 3. 模块拆分

Aggregator/summary builder 放在 `engine/statistical_evidence.py`（由 S04 经 S01 merge owner review）；三个 thin consumer adapters 分别留在现有 modules；tests 独立。

## 4. 代码结构与文件影响

修改 `engine/statistical_evidence.py`、`engine/strategy_admission_statistical_gate.py`、`engine/cross_strategy_reliability_gates.py`、`engine/strategy_admission_package.py`；创建 `tests/research/test_statistical_evidence_projection.py`。禁止新 gate 与 lineage writes。

## 5. 数据模型

Summary 含 claim id、mandatory_methods、method refs/hashes/statuses、aggregate status/reasons/hash、effective limitation。Projection 只复制 validated summary refs/status/limitations。

## 6. API

`aggregate_statistical_evidence(evidences, mandatory_methods)`；`consume_computable_statistical_evidence(summary)` in CR151/CR154；`attach_statistical_evidence_summary(package, summary)`。每个接口正/负 tests。

## 7. 流程

validate all refs → compare exact mandatory set → select worst state → build canonical summary → each consumer worst-state merge → preserve effective ceiling。Missing/orphan/mismatch stops before attachment。

## 8. 技术细节

Decision table以枚举 rank实现，但 final reason按 child state确定；informational method 单独列出且不影响 claim。Consumer merge 使用 existing status severity mapping，不得用布尔 OR。raw-count DSR evidence 与 effective claim 分栏。

## 9. 安全与性能

Pure local mapping；不读取 artifact path/credentials；O(number of methods)；禁止 runtime promotion flags。

## 10. 测试

all PASS；BH pass/PBO fail；PBO pass/DSR unavailable；any blocked；orphan；mandatory shrink attempt；consumer already blocked；3/3 projection；new gate=0；raw/effective fields；CR155 unchanged。

## 11. 实施步骤（TASKS）

S04-T01 aggregator；T02 CR151/CR154 adapters；T03 package attach；T04 combination/compatibility/regression tests。

## 12. 风险与灰区

无 LCQ。风险为 consumer status enum差异与 legacy effective requirement；用 explicit adapter tests/worst-state mapping，任何 schema conflict回 CP5/CP3，不自由裁量。

## 13. 回滚

移除/禁用新 summary attachment，consumer 恢复 typed_unavailable/blocked；不删除 evidence artifacts。无发布授权。

## 14. DoD / Definition of Done

- [ ] decision combinations 100% covered；3/3 projections
- [ ] status improvement=0；OR-pass=0；new gate=0；effective alias=0
- [ ] CP5 前不实现

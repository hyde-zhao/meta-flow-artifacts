---
story_id: "CR164-S02-bh-wrc-spa-evidence"
title: "BH and WRC SPA computable evidence"
story_slug: "bh-wrc-spa-evidence"
lld_version: "0.1"
tier: "L"
status: "confirmed"
confirmed: true
created_by: "host-orchestrator inline meta-dev"
created_at: "2026-07-12T20:25:00+08:00"
confirmed_by: "user"
confirmed_at: "2026-07-12T20:50:00+08:00"
feature_design_refs: ["docs/features/multiple-testing-calculators/DESIGN.md", "docs/features/multiple-testing-calculators/TEST-PLAN.md", "docs/features/multiple-testing-calculators/TASKS.md", "docs/features/statistical-evidence-contract/DESIGN.md"]
lld_policy: {required_level: "full-lld", trigger_reasons: ["statistical-algorithm", "bootstrap-provenance", "legacy-contract-risk"], rationale: "Method/version and bootstrap must be exact."}
open_items: 0
---

# LLD: CR164-S02 — BH and WRC/SPA evidence

## 0. 上游设计依据（工程依据）

HLD §§5/10、ADR-001/003、FEAT-24/25；Politis-Romano stationary bootstrap、White Reality Check、Hansen SPA primary methods。

## 1. Goal

创建 pure BH、stationary-bootstrap primitive、WRC 与 SPA calculators，输出 S01 MethodEvidence。

## 2. 需求（Requirements）

- BH：stable ordering/ties、p/q domain、alpha/config/membership provenance。
- WRC/SPA：candidate≥2、aligned finite return matrix、explicit benchmark/null、fixed window≥1、seed、replications。
- 不得把 legacy Bonferroni OR BH flag 作为最终 evidence。

## 3. 模块拆分

`bh.py` logical section、bootstrap sampler、WRC evaluator、SPA evaluator、adapter boundary；物理文件统一为 `engine/multiple_testing_evidence.py`，避免过早包拆分。

## 4. 代码结构与文件影响

创建 `engine/multiple_testing_evidence.py` 与 `tests/research/test_multiple_testing_evidence.py`；仅当 golden tests 证明 primitive 等价时才最小修改/调用 `engine/anomaly_multiple_testing.py`，否则保持不变。

## 5. 数据模型

无持久化。BH result 含 ordered candidate ids/raw p/q/alpha/decision；WRC/SPA config 含 algorithm_version/bootstrap_method/block_length_mode/block_length/seed/replications/benchmark/null/tail；result 含 finite corrected statistic/p-value/decision。

## 6. API

`calculate_bh(input, config)->MethodEvidence`；`stationary_bootstrap_indices(n, window, reps, seed)`；`calculate_wrc(...)`；`calculate_spa(...)`。每个接口在 tests 有正/负至少各 1 条。

## 7. 流程

validate S01 input/config → deterministic sampling/calculation → finite/domain check → evidence with hashes/provenance。invalid value/shape/config=blocked；insufficient candidates/data=typed_unavailable；valid policy miss=fail。

## 8. 技术细节

BH 的 tie ordering 使用 `(p_value,candidate_id)` 稳定排序并按原 candidate id 投影。WRC/SPA tail、centering、benchmark difference 方向、replication floor 和 golden numbers 必须在代码 docstring/tests 引用固定 primary-source version；MVP 不实现 automatic selector。

## 9. 安全与性能

Pure CPU/local fixture；无随机全局状态，使用显式 seed 的局部 generator；禁止真实 research batch。性能只做 fixture 可完成，不声明生产规模。

## 10. 测试

BH known vector/ties/0/1/invalid/membership mismatch；bootstrap repeatability/window bounds；WRC/SPA golden small matrices、shape mismatch、NaN、missing seed/benchmark、policy fail；legacy OR-pass regression。

## 11. 实施步骤（TASKS）

S02-T01 BH；T02 bootstrap；T03 WRC/SPA；T04 golden/negative/legacy tests，分别对应两个目标文件。

## 12. 风险与灰区

无 LCQ。风险：公式/centering/tail 漂移；用 algorithm_version+golden fixture 缓解。Automatic selector、第三方 package、real returns 均为 OUT。

## 13. 回滚

停用 S02 producer，BH/WRC/SPA 恢复 typed_unavailable；不回写 lineage/consumer。无发布授权。

## 14. DoD / Definition of Done

- [ ] BH/WRC/SPA 独立 evidence；fixed window provenance 完整
- [ ] deterministic/negative/golden tests 全通过；OR-pass=0
- [ ] CP5 前不实现

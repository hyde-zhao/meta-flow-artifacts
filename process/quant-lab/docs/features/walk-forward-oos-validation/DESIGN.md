---
status: "ready-for-cp5-review"
version: "0.1"
cr_id: "CR-166"
feature_id: "FEAT-166-02"
owner: "meta-se-inline"
---

# FEAT-166-02 Walk-forward/OOS Validation 设计

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 0.1 | 2026-07-13 | host-orchestrator inline meta-se | 冻结 common validator、half-open fold、purge/embargo、daily/ML adapters、event N/A 与授权预检。 |

## 1. 目标与模块

在 producer 前形成唯一 fail-closed validation boundary。实现位于 `engine/walk_forward_oos_evidence.py`；只读取传入 value。`engine/daily_multifactor_baseline_artifact.py` 与 `engine/research_production_contracts.py` 是只读 source contracts，不在 S02 修改。

## 2. 输入模型

| 字段族 | 类型 / 关键字段 | 缺失 | 冲突/非法 |
|---|---|---|---|
| manifest | id/ref/hash、declared count、ordered unique ids | typed_unavailable | count/hash/id mismatch blocked |
| split policy | strategy kind、mode、window、ref/version | typed_unavailable | unknown/contradiction blocked |
| temporal | 每 fold train/validation/OOS `[start,end)` | typed_unavailable | reverse/illegal overlap blocked |
| purge/embargo | unit、applicability、horizon、required/applied、ref | typed_unavailable | applied<required blocked |
| metrics | mandatory id/direction/threshold、finite values | typed_unavailable | NaN/Inf/threshold contradiction blocked |
| lineage | ref/hash、membership hash、source refs/hashes | typed_unavailable | ref/hash/membership mismatch blocked |
| authorization | mode、ref classification、operation counters | typed_unavailable | external/real ref or nonzero counter blocked |

## 3. 校验次序与规则

1. authorization/ref classification；不调用 resolver，dereference=0。
2. schema、identity、declared/observed inventory。
3. ISO-8601 parse 与 fold 内 `train_end <= validation_start`、`validation_end <= oos_start`；允许合法 rolling/expanding 跨 fold 训练重叠。
4. overlap applicable 时 purge mandatory；purge/embargo 使用同一显式 unit，`applied >= required`；exact eligible、one-below blocked。
5. mandatory metrics 完整且 finite。
6. lineage ref/hash/membership/source binding。

校验返回 immutable `ValidationResult(status, normalized_input, issues)`；只有 `validated` 可进入 S03。问题必须含 code、field、fold_id（适用时）、message；blocked/unavailable reason coverage=100%。

## 4. Adapter 合同

| Adapter | 输入 | 输出 | 降级 |
|---|---|---|---|
| `adapt_daily_walk_forward_input` | `WalkForwardSplitManifest` + 显式 validation/OOS bounds、metrics、lineage/auth | common input/result | 不推断缺字段；missing→unavailable。 |
| `adapt_ml_walk_forward_input` | `MLPurgedEmbargoCVPolicy` + actual train/validation/OOS bounds、metrics、lineage/auth | common input/result | policy 不替代 actual folds；count/policy mismatch blocked。 |
| `event_walk_forward_applicability` | `EventTimeSemantics` contract fact | structured `not_applicable_with_reason` | reason=`event_fold_semantics_unfrozen`；producer/fixture/feed access=0。 |

## 5. 失败行为、性能与安全

校验为 total boundary：bad input 返回 typed result，不 silent PASS。复杂度 `O(folds × mandatory_metrics)`，不做组合搜索。禁止 file/env/network/lake/NAS/provider/calendar/runtime/broker 调用。operation counter 任一非零为 blocked。

## 6. Gotchas 与重访

- 相邻半开区间 end=start 合法；不要用“所有跨 fold 区间不得重叠”误阻断 expanding window。
- purge ref 非充分性证明；required/applied/unit 必须显式。
- ML policy 的 `test_*` 不自动等同通用 OOS，adapter 必须由调用者显式声明映射。
- event time contract 不等于 event fold contract；未来只有独立 CR 冻结窗口/available-at/reference fixture 后才能重访。

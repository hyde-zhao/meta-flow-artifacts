---
status: "ready-for-cp5-review"
version: "0.1"
cr_id: "CR-166"
feature_id: "FEAT-166-03"
owner: "meta-se-inline"
---

# FEAT-166-03 Deterministic C2 Producer 设计

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 0.1 | 2026-07-13 | host-orchestrator inline meta-se | 冻结纯函数 producer、fold outcome、declared denominator、hash/provenance 与 self-validation。 |

## 1. 接口与职责

`produce_walk_forward_oos_evidence(validation: ValidationResult) -> WalkForwardOOSResult` 位于 `engine/walk_forward_oos_evidence.py`。仅接受 S02 `validated` input；其他状态原样保守转换，绝不计算 present component。输出包含 immutable fold evidence、aggregate、component descriptor 与 envelope，不调用 consumers。

## 2. 计算契约

- 每个 fold 按 mandatory metric direction/threshold 重算 metric decision 和 fold outcome；调用方不得直接传 `passed` 作为真相。
- `declared_fold_count` 来自已绑定 manifest；`passed_fold_count` 只计明确 pass；`pass_rate=passed/declared`。
- observed/validated 与 declared 不一致时 availability 非 present，`pass_rate=null`；不得过滤坏 fold。
- stability/degradation 只有显式 policy 才计算；无 policy 时字段 N/A/limitations，不做趋势推断。
- outcome 为 `pass/fail/needs_review`，仅在 availability=`present` 时存在。

## 3. Identity 与 provenance

| Hash | Domain | 输入 |
|---|---|---|
| input hash | `quant-lab.walk-forward-oos.input.v1` | normalized 7-field input |
| config hash | `quant-lab.walk-forward-oos.config.v1` | metric/purge/embargo/split policies |
| component hash/ref | `quant-lab.walk-forward-oos.component.v1` | unsigned component；ref 从 type/version/hash 派生 |
| envelope hash | neutral envelope domain | ordered descriptors + logical provenance/auth/limitations |

provenance 记录 schema/producer version、lineage identity、fixture/static validation mode，不记录 current clock、物理 path、credential 或 runtime state。

## 4. Self-validation 与失败路径

producer 完成后从 evidence 重算 fold counts、outcomes、pass rate、input/config/component hash；差异=0 才可返回 present。tamper/hash mismatch、non-finite、fold identity 缺失或非 validated precondition 均 blocked/typed_unavailable 并带 reason。S04 只接收 self-validated component。

## 5. NFR、Gotchas 与回退

10 次同 normalized fixture 只产生 1 个 component/envelope hash；单遍 `O(folds × metrics)`；I/O=0。Gotcha：pass rate 是 evidence 事实，不是整体 admission；consumer threshold/policy 不得回灌 producer。若 canonical/C1 compatibility 不通过，S03 不绕过 S01 wrapper；若 validator status 不确定，宁可 non-present。

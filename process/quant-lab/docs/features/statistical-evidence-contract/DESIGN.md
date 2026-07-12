# FEAT-24 Statistical Evidence Contract — DESIGN

## 修订记录

| 版本 | 日期 | 修订人 | 变更 |
|---|---|---|---|
| 0.1 | 2026-07-12 | host inline meta-se | 初始实现设计。 |

## 职责与非职责

拥有 `StatisticalEvidenceInput`、`MethodConfig`、`MethodEvidence`、`StatisticalEvidenceSummary` 的 typed schema、validation、canonical hash 与 reason codes；只读 CR163 lineage。不计算方法、不写 admission、不把 raw count alias 为 effective。

## 接口与失败路径

| 接口 | 输入 | 输出 | 失败 |
|---|---|---|---|
| `normalize_statistical_evidence_input` | lineage projection + method payloads | immutable normalized input | absent→typed_unavailable；mismatch/NaN/Inf→blocked |
| `validate_method_evidence` | method evidence + expected identity | validation result | orphan/ref/hash/domain mismatch→blocked |
| `canonical_evidence_hash` | schema-versioned JSON-safe payload | SHA-256 | volatile/non-finite/unknown version→blocked |

Minima：BH/WRC≥2 candidates；PBO≥4 candidates/4 splits；DSR≥2 trials/sample≥30/variance>0。状态枚举 `pass/fail/typed_unavailable/blocked`。调用方向固定 validator→calculator→evidence validator→aggregator。

## 文件与兼容

后续实现创建独立 contract/validator module 和 tests；精确路径由 S01 LLD 冻结。现有 CR151/CR154 schema 通过 adapter 消费，不直接修改 lineage。回滚为停止 producer/projection并恢复 typed_unavailable。

## Gotchas

- schema 存在不代表 computation provenance 存在。
- 缺失与冲突不同：缺失可 unavailable，矛盾必须 blocked。
- candidate membership 不得按“成功候选”后验收缩。


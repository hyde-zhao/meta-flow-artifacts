---
status: "ready-for-cp5-review"
version: "0.1"
cr_id: "CR-166"
feature_id: "FEAT-166-02"
---

# FEAT-166-02 任务清单

| Task ID | Story | 任务 | 依赖 | 文件 / DoD |
|---|---|---|---|---|
| F166-02-K01 | S02 | 实现 authorization/identity/sufficiency 校验 | S01 | C2 module；7/7 字段有 result。 |
| F166-02-K02 | S02 | 实现 temporal/purge/embargo 校验 | K01 | three leakage classes blocked；boundary exact/one-below。 |
| F166-02-K03 | S02 | 实现 metric/lineage 校验 | K01 | non-finite acceptance=0；orphan=0。 |
| F166-02-K04 | S02 | 实现 daily/ML adapter 与 event N/A evaluator | K01-K03 | compatibility=2/2；event implementation=0。 |
| F166-02-K05 | S02/S05 | 完成 validation fixtures | K01-K04 | TEST-PLAN 10/10；external dereference=0。 |

执行顺序 K01→K02/K03→K04→K05；同一 C2 文件由 S02 单写。CP5 批准前禁止实现。

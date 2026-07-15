---
status: "ready-for-cp5-review"
version: "1.0"
cr_id: "CR-169"
feature_id: "FEAT-169-02"
feature_name: "C3/C4 Correlation Boundary"
related_stories: ["CR169-S01", "CR169-S04", "CR169-S05"]
---

# FEAT-169-02 C3/C4 Correlation Boundary 设计

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-14 | host-orchestrator inline meta-se/meta-dev | 冻结 exact 13-field header、view 构造、identity/hash 分域与 fail-before-canonical 合同。 |

## 1. 精确合同

`C3C4CorrelationHeaderV1` 精确包含 13 字段：

1. `manifest_ref`、`run_ref`、`strategy_ref`、`package_ref`；
2. `price_basis`、`notional_basis`、`currency`、`calendar`；
3. `as_of`、`horizon_start`、`horizon_end`；
4. `lineage_context_ref`、`authorization_context_ref`。

字段必须 exact match；空值、额外字段、无法唯一构造或 `horizon_start > horizon_end` / `horizon_end > as_of` 均返回 `BLOCKED/c4_c3_c4_correlation_header_mismatch`，canonical 调用数=0。attachment identity 只进入 envelope binding；basis/temporal fields 仍分别进入 component semantic body；header 不形成第三个 hash 域。

## 2. 调用与相邻边界

| 调用方 | 调用时机 | 输入 | 输出 | 降级 |
|---|---|---|---|---|
| S04 joint adapter | typed C3/C4 self-validation 后、payload 构造前 | 两个 attachment、两个 component view、显式 static join context | `CorrelatedC3C4Pair` 或 ordered issues | 任一 mismatch BLOCKED；不调用 Gate4。 |
| S05 verification | negative/tamper cases | fixture pair | call count + exact reason | 无 fallback/default。 |

CR-169 不修改 `economic_cost@v1` schema；C3 view 从既有 attachment、basis/audit refs 与显式 static temporal context构造。component-specific provenance list 不要求互相相等，只要求共同审计 context ref 相等。

## 3. 安全、演进与 Gotchas

- 不创建 header registry、parallel envelope 或 runtime join service。
- 不允许只匹配 strategy_ref 而忽略 run/package/as-of/basis。
- 不把“component hash 相同”当成同 subject；subject join 必须 13 字段成立。
- header 字段变化属于 public contract/schema 变化，必须回 CP3/新 CR。

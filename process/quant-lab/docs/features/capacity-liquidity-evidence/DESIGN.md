---
status: "ready-for-cp5-review"
version: "1.0"
cr_id: "CR-169"
feature_id: "FEAT-169-01"
feature_name: "Capacity / Liquidity / ADV Fixture Evidence"
source_hld: "docs/design/HLD-CAPACITY-LIQUIDITY-ADV-EVIDENCE-PRODUCER.md"
source_adr: "docs/design/ARCHITECTURE-DECISION-CAPACITY-LIQUIDITY-ADV-EVIDENCE-PRODUCER.md"
related_stories: ["CR169-S01", "CR169-S02", "CR169-S03", "CR169-S05"]
---

# FEAT-169-01 Capacity / Liquidity / ADV Fixture Evidence 设计

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-14 | host-orchestrator inline meta-se/meta-dev | 冻结 C4 v1 静态合同、12 类 fail-closed、Decimal proxy 算术、component/envelope hash 分域及 producer 编排。 |

## 1. 目标与非目标

| 类型 | 内容 |
|---|---|
| Goal | 将显式 synthetic/static ADV、notional、participation cap 与 sizing 假设确定性转换为 `capacity_liquidity@v1`。 |
| Goal | 12/12 P0 类别有稳定 machine reason，10 次相同规范化输入只产生 1 个 component hash。 |
| Non-Goal | 真实 ADV/liquidity/capacity、provider/lake/NAS、经验校准、alpha-decay、aggregate 或 Stage 3。 |

## 2. 边界与相邻对象

| 对象 | 本 Feature 负责 | 不负责 | 相邻 owner |
|---|---|---|---|
| C4 input/component | immutable values、normalize、validate、proxy calculation、availability、hash | attachment join 与 Gate4 policy | FEAT-169-02/03 |
| Envelope | 提供 component descriptor 与 attachment inputs | neutral catalog/serialization 实现 | CR-166 neutral envelope / S03 |
| C3 | 不读取或修改 C3 calculation body | economic-cost 算术与 hash | CR-168 |
| Alpha decay | calculator=0；只保留 follow-up ref | 时间预测衰减方法 | FU-CR161-008 |

## 3. 数据模型与精确算术

| Object | 关键字段 | 不变量 |
|---|---|---|
| `CapacityLiquidityEvidenceInput` | attachment identity、13-field header、synthetic ADV、requested/turnover notional、participation cap、model/version、currency/minor unit、lineage/auth | typed input；binary float/非有限值拒绝；不接真实 source handle。 |
| `NormalizedCapacityLiquidityInputV1` | identity 外的规范化 computational body | identity 不进入 component hash；basis/temporal/static limitation 进入。 |
| `CapacityLiquidityBreakdownV1` | participation ratio、capacity amount、headroom、within-cap outcome | 只使用显式值；无隐式 ADV、FX 或市场校准。 |
| `CapacityLiquidityEvidenceV1` | type/schema、3 个 typed refs、availability/reasons、semantic hash、limitations/no-real claims | type=`capacity_liquidity`、schema=`v1`。 |
| `CapacityLiquidityBuildResult` | normalized、attachment、ordered issues、evidence-or-none | issues 非空时 calculator 调用数=0。 |

v1 唯一 active method 为 `static_adv_cap_v1`：

```text
adv_participation = requested_notional / synthetic_adv
raw_capacity_amount = synthetic_adv * participation_cap
capacity_amount = quantize(raw_capacity_amount, currency_minor_unit, ROUND_HALF_EVEN)
liquidity_headroom = quantize(capacity_amount - requested_notional, currency_minor_unit, ROUND_HALF_EVEN)
within_declared_cap = adv_participation <= participation_cap
```

`synthetic_adv > 0`、`requested_notional >= 0`、`0 < participation_cap <= 1`、`currency_minor_unit > 0`。超过 cap 是本 v1 的显式 conservative policy violation，不是对所有市场 participation 的数学断言。`capacity_dollars_ref` 是 canonical Gate4 的历史字段名；其引用对象必须携带声明 currency，不能据字段名假设 USD。

## 4. Public 接口与编排

| Interface | 输入 | 输出 | 失败 |
|---|---|---|---|
| `normalize_capacity_liquidity_input(raw)` | typed C4 input | normalized + attachment/header context | ordered issues |
| `validate_capacity_liquidity_input(value)` | normalized | immutable issues | N01..N12 |
| `calculate_capacity_liquidity_breakdown(value)` | validation-clean body | breakdown | numeric invariant failure |
| `build_capacity_liquidity_evidence(raw)` | full typed input | build result | unavailable/blocked；无 I/O |
| `capacity_liquidity_component_hash(evidence)` | normalized body + outcome/limitations | domain-separated sha256 | tamper blocked |

唯一 producer entry 固定执行 normalize → validate → issue short-circuit → calculate → produce/self-validate。调用方不得传入 `validated=true` 绕过校验；任何 issue 都不得调用 calculator。

## 5. N01..N12 reason code

| 顺序 | Reason code | 默认结果 |
|---:|---|---|
| N01 | `c4_identity_binding_missing` | typed_unavailable |
| N02 | `c4_static_liquidity_basis_missing` | typed_unavailable |
| N03 | `c4_proxy_model_version_missing` | typed_unavailable |
| N04 | `c4_nonfinite_numeric_invalid` | blocked |
| N05 | `c4_negative_or_participation_cap_invalid` | blocked |
| N06 | `c4_unit_currency_basis_mismatch` | blocked |
| N07 | `c4_calendar_temporal_mismatch` | blocked |
| N08 | `c4_c3_c4_correlation_header_mismatch` | blocked |
| N09 | `c4_lineage_provenance_authorization_missing_or_mismatch` | missing→typed_unavailable；mismatch/越权→blocked |
| N10 | `c4_component_or_envelope_hash_tampered` | blocked |
| N11 | `c4_gate4_ref_not_typed_present` | blocked |
| N12 | `c4_projection_reason_escape_or_postcondition_violation` | rejected/blocked |

## 6. Hash、演进、安全与回退

- semantic domain=`quant-lab.capacity-liquidity-input.v1`；component domain=`quant-lab.capacity-liquidity-component.v1`。
- component hash 不含 manifest/run/strategy/package identity；envelope hash 必须绑定 identity、component hash 与 audit refs。
- v1 schema、method 与 rounding 冻结不可变；新方法、alpha-decay 或真实输入只能新 schema + 独立 CR。
- refs 仅为 opaque/content-addressed logical refs，不解引用文件、URL、credential、provider 或 lake。
- 回退时保留 typed unavailable slot，不把失败 component attach 成 present；catalog activation 由 S03 独立回退。

## 7. Gotchas

- 不要把 `capacity_dollars_ref` 的名字当成 USD 货币保证。
- 不要把 synthetic ADV、cap 或 fixture PASS 描述为真实容量结论。
- 不要在 issue path 调 calculator，也不要用默认 currency/minor unit/model 填空。
- 不要把 alpha-decay 字段塞进 v1 optional extension；它由 FU-CR161-008 决策。

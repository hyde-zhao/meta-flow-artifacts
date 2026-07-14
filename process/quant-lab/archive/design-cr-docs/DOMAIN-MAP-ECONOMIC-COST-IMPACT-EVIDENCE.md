---
status: approved-cp3
version: "1.3"
cr_id: "CR-168"
source_blueprint: "process/archive/design-cr-docs/BLUEPRINT-ECONOMIC-COST-IMPACT-EVIDENCE.md"
source_requirements: "docs/product/REQUIREMENTS.md"
---

# Domain Map：Economic Cost / Slippage / Impact Evidence

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-14 | host-orchestrator inline meta-se-critical | 定义 C3 输入、假设、分项、evidence、availability、projection outcome 和 Gate4 guard 领域规则。 |
| 1.1 | 2026-07-14 | host-orchestrator inline meta-se-critical | 明确 identity/component hash 分域、proxy 合法域、舍入点、minor unit 缺失行为、candidate release profile、postcondition reason code 与 schema 演进。 |
| 1.2 | 2026-07-14 | host-orchestrator inline meta-dev | CP5 评审整改：固定五个成本分项的显式 basis、`net_return` 公式，以及 10 个输入 fail-closed reason code 的精确枚举；不改变 CP3 范围或架构决策。 |
| 1.3 | 2026-07-14 | host-orchestrator inline fallback | 按 CR131 design-surface 规则归档；仅更新 archive route 元数据，领域语义不变。 |

## 1. 术语表

| Term | 定义 | 来源 | 备注 |
|---|---|---|---|
| C3 economic cost evidence | 由显式 fixture/static 输入计算的 fee/tax/spread/slippage/impact approximation typed component | REQ-CR168-001..005 | 不是真实 TCA |
| transparent impact approximation | 参数、单位、公式、限制可审计的静态近似 | REQ-CR168-003 | v1 只支持 `square_root` active path |
| cost underestimation status | 表明成本分项和限制是否足以支持当前 C3 claim 的状态 | REQ-CR168-002/003 | present component 只允许 PASS；否则不进入 projection |
| component semantic hash | 绑定 subject-neutral 的规范化成本语义、schema、model/version、参数和算术输出的 hash | REQ-CR168-005 | 不含 manifest/run/strategy/package identity；identity 由 envelope hash 绑定 |
| envelope canonical hash | CR166 neutral envelope 对 subject、components、provenance、authorization、limitations 的完整 hash | CR166 | 不要求跨 strategy type 相同 |
| absent-no-na-reason | C4 未建时 Gate4 flat payload 不含三个 C4 refs，也不含任何对应/通用 N/A reason key | REQ-CR168-006 | 这是 CR168 adapter 的安全表示 |
| reason escape | 试图以字段级或通用 N/A reason 绕过 C4 missing blocked claim | SC-CR168-B02 | 8/8 key presence 均拒绝 |

## 2. 领域对象

| Object ID | 对象 | Owner | 关键字段 | 状态 | 规则来源 |
|---|---|---|---|---|---|
| OBJ-CR168-01 | EconomicCostEvidenceInput | FEAT-168-01 | identity、gross basis、trade summary、fee/tax/spread/slippage/impact assumptions、unit/basis、lineage/auth | raw / normalized / unavailable / blocked | REQ-CR168-002/004 |
| OBJ-CR168-02 | StaticImpactAssumption | FEAT-168-01 | family=`square_root`、coefficient、static_reference_notional、model/version、limitations | valid / unavailable / blocked | REQ-CR168-003 |
| OBJ-CR168-03 | CostBreakdown | FEAT-168-01 | fee、tax、spread、slippage、impact、total、gross_pnl、net_pnl、net_return | computed / reconciled / blocked | REQ-CR168-003/004 |
| OBJ-CR168-04 | EconomicCostEvidenceV1 | FEAT-168-01 | header、breakdown、outcome、reason codes、lineage、limitations、cost status、component hash | present / typed_unavailable / blocked | REQ-CR168-001..005 |
| OBJ-CR168-05 | EconomicCostComponentDescriptor | FEAT-166-01 | `economic_cost@v1`、ref、hash、availability | active / invalid | REQ-CR168-001 |
| OBJ-CR168-06 | C3Gate4ProjectionOutcome | FEAT-168-02 | status、Gate4 summary ref/value、blocked reason、canonical_invoked、payload keys | blocked / rejected / gate4_non_pass | REQ-CR168-006 |
| OBJ-CR168-07 | C4AvailabilityMarker | future C4 owner | reserved/not-built/typed_unavailable | unavailable only in CR168 | REQ-CR168-006/009 |

## 3. 状态机

| State Machine | 对象 | 合法转换 | 条件 | 非法转换处理 |
|---|---|---|---|---|
| SM-CR168-01 | EconomicCostEvidenceInput | raw → normalized → present | 9/9 规则、授权、数值、basis、算术、lineage 全部通过 | 缺失 → typed_unavailable；冲突/越权/tamper → blocked |
| SM-CR168-02 | EconomicCostEvidenceV1 | typed_unavailable → present | 只能通过重新提供完整显式输入重新构建；不原地提升旧对象 | 原地改状态或旧 hash → blocked |
| SM-CR168-03 | Projection | input-check → canonical-call → non-pass | C3 present、C4 unavailable、无 forbidden keys、operation counts=0 | 任一前置失败拒绝且 canonical_invoked=false；unexpected PASS 转 blocked |

## 4. 数值与算术规则

| Rule ID | 规则 | 精确合同 | 失败行为 |
|---|---|---|---|
| NUM-CR168-01 | Decimal normalization | 数字以十进制字符串或整数输入，转 `Decimal`；禁止 binary-float 直接进入 semantic hash | 非有限/不可解析 blocked |
| NUM-CR168-02 | Unit normalization | rate 统一为 fraction；金额统一到声明 currency；`currency` 和正数 `currency_minor_unit` 必填；中间精度 28 位，最终按 minor unit、ROUND_HALF_EVEN | 缺 currency/minor unit、minor unit 非正、缺 conversion 或跨字段冲突 blocked；无默认 fallback |
| NUM-CR168-03 | Itemized cost | `fee=traded_notional×fee_rate+fee_fixed_amount`；`tax=sell_notional×tax_rate+tax_fixed_amount`；`spread=traded_notional×effective_spread_rate`；`slippage=traded_notional×effective_slippage_rate`。所有 basis 为显式、非负且与声明 price/notional basis 一致；固定额不存在时显式为 `0`。 | 分项不可重算、basis 缺失或不一致 blocked |
| NUM-CR168-04 | Square-root impact | `participation_proxy=traded_notional/static_reference_notional`；合法域为有限 `[0,+∞)`，允许 `>1`；`impact_rate=coefficient*sqrt(proxy)`；`impact_cost=traded_notional*impact_rate` | reference<=0、coefficient<0、proxy<0、非有限或字段缺失 blocked；`proxy>1` 不 blocked 但不得产生 capacity claim |
| NUM-CR168-05 | Reconciliation | 五分项以 precision=28 未量化 canonical Decimal 保留；先求 raw sum，再量化 `total_cost`；`net_pnl=gross_pnl-total_cost` 后再按同一 minor unit 量化；`net_return=quantized_net_pnl/performance_notional`，其中 `performance_notional` 必须为显式正数。若 gross 只给 rate，也必须使用同一 `performance_notional` 推导 gross PnL；不得逐项先量化。 | 任一差异超过最终 currency quantization 1 unit、performance_notional 缺失或非正 blocked |

`static_reference_notional` 必须由 fixture/static 输入显式给出，lineage 标为 synthetic/static；它不是 ADV、capacity 或市场校准值。

## 5. Availability 决策表

| 输入事实 | Component availability | `cost_underestimation_status` | 是否调用 Gate4 | 结果 |
|---|---|---|---:|---|
| 9/9 完整、算术/授权/hash 全通过、square_root 参数完整 | present | PASS | 是 | C4 absent 使 Gate4 非 PASS |
| 必需输入缺失或 impact 为 structured N/A | typed_unavailable | 不投影 | 否 | projection blocked/unavailable |
| 数值冲突、basis 冲突、tamper、越权、负成本违规 | blocked | 不投影 | 否 | projection rejected |
| C4 被上游声称 present | out-of-scope/block | 不变 | 否 | 路由 FU-CR161-005/007，不由 CR168 验证 |
| 任一 forbidden reason key 出现 | blocked | 不变 | 否 | `gate4_reason_escape_rejected` |
| safe absent 调用后 Gate4 意外 PASS | blocked | PASS 不外传 | 已调用 1 次 | `gate4_unexpected_pass` |
| safe absent 调用后 Gate4 非 PASS 但缺任一 C4 missing claim | blocked | canonical 结果不外传 | 已调用 1 次 | `gate4_postcondition_violation` |

## 6. Projection Forbidden-Key Set

精确集合如下，匹配规则为 key presence，值为空也拒绝：

1. `adv_participation_ref_na_reason`
2. `adv_participation_ref_n_a_reason`
3. `capacity_dollars_ref_na_reason`
4. `capacity_dollars_ref_n_a_reason`
5. `liquidity_sizing_refs_na_reason`
6. `liquidity_sizing_refs_n_a_reason`
7. `na_reason`
8. `n_a_reason`

允许送入 canonical Gate4 的 payload key 精确为四个 C3 字段；三个 C4 ref key 直接省略。`release_profile` 固定为 `"candidate-release"`，与零 operation counts 作为函数参数传递，不进入任意 mapping 透传面，也不允许 caller override。

## 7. Reason Code 最小集合

10 个 P0 输入类必须一一映射到下列精确 code；`availability_effect` 由同一 issue 明示，纯缺失可为 `typed_unavailable`，数值、冲突、篡改或授权矛盾必须为 `blocked`，不得以模糊字符串替代。

| P0 类 | 精确 reason code | 目标计数 |
|---|---|---:|
| N01 gross / performance basis 缺失 | `c3_gross_performance_basis_missing` | 1 |
| N02 trade / turnover / notional basis 缺失 | `c3_trade_turnover_notional_basis_missing` | 1 |
| N03 cost model / version 缺失 | `c3_cost_model_version_missing` | 1 |
| N04 非有限数值 | `c3_nonfinite_numeric_invalid` | 1 |
| N05 非法负成本 | `c3_negative_cost_invalid` | 1 |
| N06 unit / price / notional basis 冲突 | `c3_unit_price_notional_basis_mismatch` | 1 |
| N07 currency / price-basis / calendar 跨字段冲突 | `c3_currency_price_calendar_mismatch` | 1 |
| N08 gross / cost / net 算术冲突 | `c3_gross_cost_net_arithmetic_mismatch` | 1 |
| N09 lineage / provenance / authorization 缺失或不一致 | `c3_lineage_provenance_authorization_missing_or_mismatch` | 1 |
| N10 component identity/hash 篡改 | `c3_component_hash_tampered` | 1 |

| 类别 | Code family | 目标计数 |
|---|---|---:|
| N/A / unavailable | `c3_impact_not_applicable`、`c3_required_input_unavailable` | 至少覆盖每个 unavailable 分支 |
| Projection guard | `gate4_reason_escape_rejected`、`gate4_unexpected_pass`、`gate4_postcondition_violation`、`gate4_c3_not_present` | 4 |
| 权限 | `c3_authorization_missing`、`c3_external_operation_forbidden` | 2 |

## 8. 业务规则与 Gotchas

- RULE-CR168-01：present C3 不能以通用 N/A reason 代替 C4 evidence。
- RULE-CR168-02：`cost_underestimation_status=PASS` 只说明静态合同完整，不声明成本模型经真实市场验证。
- RULE-CR168-03：event-specific 请求返回 N/A/deferred，但不得把该 N/A 送进 Gate4 C4 字段。
- RULE-CR168-04：CR155 lifecycle 已关闭不代表 admission 可提升；现有 package 继续 BLOCKED、`paper_candidate=false`。
- RULE-CR168-05：attachment identity 与 package/run provenance/auth 只由 neutral envelope hash 绑定；component hash 只绑定字段族 2-9 的 subject-neutral 成本语义投影。
- RULE-CR168-06：`economic_cost@v1` 冻结不可变；新增 active family、改变舍入/算术或启用 rebate 必须通过独立 method CR 发布新 schema version。
- Gotcha：不要把 canonical Gate4 当前 permissive 行为写成必须永久保持的测试合同；只验证 CR168 adapter 的安全结果。
- Gotcha：不要用 full envelope hash 做跨 strategy type 等价性断言；比较 component semantic hash 和算术结果。

---
status: "ready-for-cp5-review"
version: "1.1"
cr_id: "CR-168"
feature_id: "FEAT-168-01"
feature_name: "Economic Cost Evidence"
source_blueprint: "process/archive/design-cr-docs/BLUEPRINT-ECONOMIC-COST-IMPACT-EVIDENCE.md"
source_hld: "process/archive/design-cr-docs/HLD-ECONOMIC-COST-IMPACT-EVIDENCE.md"
source_adr: "process/archive/design-cr-docs/ARCHITECTURE-DECISION-ECONOMIC-COST-IMPACT-EVIDENCE.md"
related_stories: ["CR168-S01", "CR168-S02", "CR168-S03", "CR168-S05"]
---

# FEAT-168-01 Economic Cost Evidence 设计

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-14 | host-orchestrator inline meta-se | 冻结 C3 九族输入、subject-neutral identity、Decimal/rounding、static square-root、typed outcome 与 v1 演进合同。 |
| 1.1 | 2026-07-14 | host-orchestrator inline meta-dev | CP5 评审整改：显式化 S01→S02 producer 编排、issue 短路、五分项 basis 与 `net_return` 公式；不改变 C3/C4 或真实数据边界。 |

## 1. 目标与非目标

| 类型 | 内容 |
|---|---|
| Goal | 把显式 synthetic/static fee/tax/spread/slippage/impact 输入确定性转换为 `economic_cost@v1` typed component。 |
| Goal | 九字段族 9/9 可校验，10 类 P0 输入失败全部 fail-closed，10 次同语义输入得到 1 个 component hash。 |
| Non-Goal | 真实数据/TCA/calibration、C4、event producer、Gate4 policy、aggregate、I/O 或 registry。 |

## 2. 边界与相邻对象

| 对象 | 本 Feature 负责 | 不负责 | 相邻 owner |
|---|---|---|---|
| C3 input/component | schema、normalization、validation、calculation、availability、semantic hash | package identity canonical binding | FEAT-166-01 |
| Impact | explicit static square-root | ADV/capacity/market calibration/其他 active family | FU-005 / future method CR |
| Gate4 | 输出四个可投影 C3 值 | payload guard/canonical call | FEAT-168-02 |
| Verification | 提供 pure interfaces 与 reason codes | 跨 Feature claim/QAC 汇总 | FEAT-168-03 |

## 3. 模块与数据模型

| 文件 | 动作 | 职责 |
|---|---|---|
| `engine/economic_cost_evidence.py` | 创建 | input/assumption/evidence immutable values、normalizer、validator、producer、semantic hash。 |
| `engine/economic_cost_calculator.py` | 创建 | pure Decimal fee/tax/spread/slippage/square-root impact 与 reconciliation。 |

| Object | 核心字段 | 不变量 |
|---|---|---|
| `EconomicCostEvidenceInput` | 9 families + attachment context | 九族全校验；family 1 不进入 component semantic projection。 |
| `EconomicCostSemanticInputV1` | families 2-8 + family 9 assumption lineage/limitations | subject-neutral；finite Decimal；schema immutable。 |
| `CostBreakdownV1` | fee/tax/spread/slippage/impact/raw_total/total/net | 五分项不先按 minor unit 量化。 |
| `EconomicCostEvidenceV1` | type/schema、semantic input hash、breakdown、availability/reasons、C3 fields、limitations | type=`economic_cost`、schema=`v1`、no-real-TCA=true。 |
| `EconomicCostBuildResult` | evidence 或 issues + attachment context | unavailable/blocked 不产生 present projection。 |

## 4. 接口合同

| Interface | 输入 | 输出 | 失败 |
|---|---|---|---|
| `normalize_economic_cost_input(raw)` | typed input；不接 arbitrary external mapping passthrough | normalized input + attachment context | missing/invalid/basis/auth issues |
| `validate_economic_cost_input(value)` | normalized input | ordered immutable issues | 10 类 reason code；false PASS=0 |
| `calculate_cost_breakdown(value)` | validated semantic input | `CostBreakdownV1` | numeric/rounding/reconciliation blocked |
| `build_economic_cost_evidence(value)` | full input | `EconomicCostBuildResult` | fail-closed，不做 I/O |
| `economic_cost_component_hash(evidence)` | subject-neutral component | domain-separated sha256 | identity 不在输入域；tamper blocked |

### 4.1 Producer 编排合同

`build_economic_cost_evidence` 是唯一 public producer entry，不接收调用方声称的 “validated” 标记，也不接收任意 mapping 作为 S01 result。它必须按以下固定顺序组合 S01/S02：

1. `normalize_economic_cost_input(raw)` 产生 `NormalizedEconomicCostInput` 与 `AttachmentContext`；
2. `validate_economic_cost_input(normalized)` 产生按 N01..N10 排序的不可变 issues；
3. 若 issues 非空，**禁止调用** `calculate_cost_breakdown`：任一 `blocked` issue 时返回 `EconomicCostBuildResult(status=blocked, evidence=None, issues, attachment_context)`；否则返回 `status=typed_unavailable` 的同形结果；
4. 只有 issues 为空才调用 calculator，随后构造 present evidence、component hash 与携带同一 attachment context 的 build result。

本文所谓 “S01 result” 精确定义为上述 `normalized_input + attachment_context + ordered issues` 三元 typed result，而不是未约束的字典或另一个 public API。计算器只接受 validation-clean 的 normalized semantic input。

## 5. 数值与 hash 决策

- 输入数值只允许整数/十进制字符串/Decimal；binary float、NaN、Infinity 全部 blocked。
- Decimal context precision=`28`。`participation_proxy` 只要求有限且 `>=0`，允许 `>1`；不得转译为 capacity。
- 五分项保留 precision=28 canonical value；先求 raw total，再按正数 `currency_minor_unit` + `ROUND_HALF_EVEN` 量化 total；net_pnl 随后量化。缺 currency/minor unit 或 minor unit 非正直接 blocked，无 fallback。
- basis 固定为：fee 使用 `traded_notional` 与可选显式 `fee_fixed_amount`；tax 使用 `sell_notional` 与可选显式 `tax_fixed_amount`；spread/slippage/impact 均使用 `traded_notional`。spread/slippage rate 为已声明的 effective all-in rate，不由实现隐式除以二或转换。`net_return=quantized_net_pnl/performance_notional`；`performance_notional` 必须是同一 gross/net 口径的显式正数。
- semantic input domain=`quant-lab.economic-cost-input.v1`；component domain=`quant-lab.economic-cost-component.v1`。
- component hash 包含 families 2-9 subject-neutral semantic projection、schema/model/limitations/breakdown/status；不含 manifest/run/strategy/package identity 与 package/run auth/provenance。
- v1 active family 精确为 `square_root`。新增 family、改变公式/舍入或 rebate 必须用新 schema + 独立 method CR；v1 不可变。

## 6. 失败与安全

| Failure | 结果 | Reason family | 恢复 |
|---|---|---|---|
| gross/basis/trade/model 缺失 | typed_unavailable 或 blocked（按必填数值合同） | `c3_*_missing` | 完整重建，不原地提升 |
| 非有限/负成本/单位/币种/算术冲突 | blocked | `c3_*_invalid/mismatch` | 修正显式输入 |
| lineage/auth 缺失或矛盾 | unavailable/blocked | `c3_lineage_*` / `c3_authorization_*` | 提供新 refs |
| component hash tamper | blocked | `c3_component_hash_tampered` | 由 canonical input 重建 |

所有 refs 只作 opaque string，不解引用路径/URL，不读 credential/env/provider/lake/NAS，不写 store/catalog/registry。

## 7. 实现顺序、测试与回退

1. S01 创建 immutable contract、normalization/validation 与 hash domains。
2. S02 创建 pure calculator，并严格执行 normalize→validate→issues short-circuit→calculate→produce；不改变 S01 hash domain。
3. S03 只消费 present component 进入 neutral envelope。
4. S05 汇总 10/10 negatives、10→1、auth/claim regression。

回退：若 C1/C2 neutral compatibility 失败，保留 subject-neutral component 但不激活/attach；若数值合同需改变，停止 v1 并重开 method/schema 决策，不修改 golden 掩盖。

## 8. Gotchas

- 不要把 family 1 identity 偷放进 component hash；也不要让 identity 脱离 envelope hash。
- 不要逐分项舍入后再 sum；这会引入可重算漂移。
- 不要把 `proxy>1` 判为无效或解释为 ADV participation。
- 不要把 `cost_underestimation_status=PASS` 解释为真实成本准确。

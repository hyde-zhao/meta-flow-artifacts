---
status: "approved-cp3"
version: "1.1"
change: "CR-169"
source_blueprint: "docs/design/BLUEPRINT-CAPACITY-LIQUIDITY-ADV-EVIDENCE-PRODUCER.md"
confirmed_by: "user"
confirmed_at: "2026-07-14T18:48:03+08:00"
---

# C4 Capacity / Liquidity / ADV Evidence Producer Domain Map

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-14 | host-orchestrator inline meta-se-critical | CR-169 CP3 草案：定义 C4 static input/evidence、correlation decision、joint outcome 和 Stage2 exit result。 |
| 1.1 | 2026-07-14 | host-orchestrator inline meta-se-critical | CP3 已批准：冻结 `C3C4CorrelationHeaderV1` 13 字段、alpha-decay follow-up 和 Stage2 exit 失败分流。 |

## 术语表

| Term | 定义 | 来源 | 备注 |
|---|---|---|---|
| C4 evidence | capacity / liquidity / ADV 的 fixture/static typed evidence | REQ169-001 | 不等同真实 ADV 或 capacity。 |
| C3/C4 correlation header | 用于 join 的 13 字段 identity、basis、currency、calendar、as-of/horizon、lineage/auth context exact binding | REQ169-004 | attachment identity 不进入 component semantic hash；basis/temporal semantics 仍属于各 component body。 |
| fixture contract pass | 静态 7-key payload 满足 local joint adapter 后置条件的局部结果 | REQ169-005 | 不是 aggregate/admission PASS。 |
| Stage2 exit verification | 七项合同逐项证据核验 result | REQ169-009 | CP8 才生成。 |

## 领域对象

| Object ID | 对象 | Owner Feature | 关键字段 / 属性 | 状态 | 规则来源 |
|---|---|---|---|---|---|
| OBJ-CR169-01 | `CapacityLiquidityInputV1` | FEAT-169-01 | synthetic ADV/notional/turnover、participation cap、capacity/liquidity model、unit/calendar、lineage/auth | raw / normalized / invalid | REQ169-001..003 |
| OBJ-CR169-02 | `CapacityLiquidityEvidenceV1` | FEAT-169-01 | `adv_participation_ref`、`capacity_dollars_ref`、`liquidity_sizing_refs`、limitations、semantic hash | unavailable / present_fixture / blocked | REQ169-001..003 |
| OBJ-CR169-03 | `C3C4CorrelationHeaderV1` | existing neutral envelope / CR169 correlation boundary | `manifest_ref`、`run_ref`、`strategy_ref`、`package_ref`、`price_basis`、`notional_basis`、`currency`、`calendar`、`as_of`、`horizon_start`、`horizon_end`、`lineage_context_ref`、`authorization_context_ref` | matched / mismatched | REQ169-004 / ADR-CR169-002 |
| OBJ-CR169-04 | `C3C4CorrelationDecision` | FEAT-169-02 | matched fields、ordered mismatch reasons、input evidence refs | matched / blocked | REQ169-004 |
| OBJ-CR169-05 | `Gate4FixtureCompatibilityOutcome` | FEAT-169-03 | exact seven field audit, canonical status, local status, reason code, `aggregate_admission_pass=false` | blocked / rejected / fixture_contract_pass | REQ169-005 |
| OBJ-CR169-06 | `Stage2ExitVerification` | FEAT-169-04 | exact 7 contract names、evidence refs、per-item status、stage3_entry_ready=false | pending / pass / fail | REQ169-008,009 |

## 状态机

| State Machine ID | 对象 | 状态 | 合法转换 | 非法转换处理 |
|---|---|---|---|---|
| SM-CR169-01 | C4 evidence | raw -> normalized -> present_fixture | P0 通过且 deterministic calculation 成功 | 任一 P0 failure -> blocked，不生成 present component |
| SM-CR169-02 | C4 evidence | raw / normalized -> unavailable | input required fields 缺失或未授权 | unavailable 不得装作 ref present |
| SM-CR169-03 | correlation decision | pending -> matched / blocked | all minimal header fields exact match | mismatch -> blocked、canonical call=0 |
| SM-CR169-04 | joint outcome | pending -> fixture_contract_pass | 7 exact fields、no reason escape、canonical PASS、local postcondition PASS | 任一条件不符 -> blocked/rejected |
| SM-CR169-05 | stage exit result | pending -> pass / fail / blocked | 7/7 named contracts each classified with evidence ref；all PASS 才允许 Stage2 complete claim | 历史 6 项缺失 / fail -> claim false，并回 CR157 / 新治理 CR；CR169 不代修且不得改 stage3_entry_ready |

## 业务规则

| Rule ID | 规则 | Owner | 影响场景 | 验证入口 |
|---|---|---|---|---|
| RULE-CR169-01 | semantic hash 不包含 manifest/run/strategy/package identity；envelope hash/binding 必须包含；13 字段 header 无独立 hash 域 | FEAT-169-01 / existing envelope | P02,B03 | determinism / correlation tests |
| RULE-CR169-02 | C3/C4 只有 header 全等才可进入 joint mapping | FEAT-169-02 | P02,B03,B04 | correlation tests |
| RULE-CR169-03 | joint mapping 是 exact 7-key allowlist；任何 N/A reason / extra key 均拒绝 | FEAT-169-03 | P03,B01,B02 | adapter tests |
| RULE-CR169-04 | canonical Gate4 只经公开 callable read-only 调用；不得修改或使用私有 helper | FEAT-169-03 | B01,B02 | import / diff guard |
| RULE-CR169-05 | fixture contract pass 不能产生 aggregate admission、capacity-scalable、real-ready 或 CR155 promoted claim | FEAT-169-03 / 04 | P03,E01 | claim regression |
| RULE-CR169-06 | Stage2 exit 需 7/7 逐项核验；历史 6 项失败回 CR157 / 新治理 CR；stage3_entry_ready 始终 false，除非独立授权 CR | FEAT-169-04 | E01 | CP8 result check |
| RULE-CR169-07 | alpha-decay calculator 在 CR169 v1 为 0；不得写成默认值或 synthetic C4 ref；owner 评估保留 `FU-CR161-008` | FEAT-169-01 | E01 | scope / contract tests |

## Gotchas

1. `unavailable`、`blocked`、`rejected` 不可互换：未提供 C4 ref 与 reason escape 是不同失败原因，后者需要明确拒绝。
2. `fixture_contract_pass` 是有限 domain state；不得简化为通用 PASS。
3. Stage2 exit result 的 `pass` 不是 Stage3 authorization state，两个状态模型必须独立保存。

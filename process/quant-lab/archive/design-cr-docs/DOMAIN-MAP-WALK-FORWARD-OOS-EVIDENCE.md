---
status: "confirmed-cp3-archived-baseline"
version: "0.2"
cr_id: "CR-166"
---

# Domain Map：CR166 Walk-forward / OOS Evidence

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 0.1 | 2026-07-13 | host-orchestrator inline meta-se-critical | 定义 envelope、fold、policy、metric、lineage、availability/outcome、component registry 与 consumer projection 规则。 |
| 0.2 | 2026-07-13 | host-orchestrator | 回填 CP3 人工批准；领域对象、状态机、分母、unknown component 与 Stage claim 规则转为 CP4/CP5 强制约束。 |

## 术语表

| Term | 定义 | 来源 / 限制 |
|---|---|---|
| Strategy Evidence Envelope | 稳定 header 与 versioned typed components 的容器；不拥有具体方法或 admission policy | REQ-CR166-005/006 |
| C2 Component | `walk_forward_oos` 类型的可计算 evidence component | 只由 CR166 producer 生成；不是 Stage 3 真实证据声明 |
| Half-open Interval | 时间区间 `[start,end)`，start 包含、end 不包含 | 防止相邻边界双重计入；真实 session/calendar resolver 不在范围 |
| Purge | 从训练侧移除会与后续评价区间产生 label/information overlap 的样本策略 | 必须由 policy 显式声明 required/applied，不从 ref 或日期差推断 |
| Embargo | 在相邻 evaluation/train 使用之间施加的隔离量 | 必须有 unit、required、applied 和 policy ref |
| Declared Fold Denominator | manifest 声明并通过 identity 校验的 fold 总数 | missing/invalid fold 不得从分母中删除以抬高 pass rate |
| Availability | evidence 是否可信存在：`present`、`typed_unavailable`、`not_applicable_with_reason`、`blocked` | 与计算 outcome 分离 |
| Outcome | 仅在 availability=`present` 时表达 `pass`、`fail`、`needs_review` | consumer 采用保守映射，不得由一个 PASS 覆盖更差状态 |
| Static Component Catalog | 编译期/源码内显式 type+version 描述表 | 不是动态 plugin/runtime registry；当前 active 只有 C2 |

## 领域对象

| Object ID | 对象 | Owner Feature | 关键字段 / 属性 | 状态 | 规则来源 |
|---|---|---|---|---|---|
| OBJ-166-01 | StrategyEvidenceEnvelope | FEAT-166-01 | envelope schema/version、subject ref、component descriptors/refs、provenance、envelope hash | unvalidated/valid/blocked | REQ-005/006 |
| OBJ-166-02 | ComponentDescriptor | FEAT-166-01 | component type、schema version、required flag、component ref/hash | active/reserved/unknown | REQ-006 |
| OBJ-166-03 | WalkForwardEvidenceInput | FEAT-166-01 | fold manifest、split policy、temporal bounds、purge/embargo、metric policy/values、lineage、authorization metadata | unvalidated/validated/typed_unavailable/blocked | REQ-001..004/009 |
| OBJ-166-04 | TemporalFold | FEAT-166-02 | fold id、train/validation/OOS `[start,end)`、sequence、source refs | valid/typed_unavailable/blocked | REQ-001/002/003 |
| OBJ-166-05 | PurgeEmbargoPolicy | FEAT-166-02 | unit、overlap applicability、label horizon、required/applied purge、required/applied embargo、policy ref | valid/not_applicable/typed_unavailable/blocked | REQ-002/008 |
| OBJ-166-06 | MetricPolicyAndValues | FEAT-166-02 | mandatory metric IDs、direction、threshold、finite fold values | valid/typed_unavailable/blocked | REQ-003 |
| OBJ-166-07 | LineageBinding | FEAT-166-02 | lineage ref/hash、fold membership hash、split/metric/source refs+hashes | valid/typed_unavailable/blocked | REQ-004 |
| OBJ-166-08 | WalkForwardOOSComponent | FEAT-166-03 | availability、outcome、fold evidence、declared/valid count、pass rate、reasons、input/config/component hash、limitations | present/typed_unavailable/blocked | REQ-003/005 |
| OBJ-166-09 | FoldEvidence | FEAT-166-03 | fold id、metric decisions、fold outcome、reason codes、source refs | pass/fail/needs_review/blocked | REQ-003/005 |
| OBJ-166-10 | ConsumerProjection | FEAT-166-04 | consumer id、same component ref/hash、availability/outcome/reasons、legacy fields | attached/blocked | REQ-007 |
| OBJ-166-11 | ApplicabilityDecision | FEAT-166-05 | strategy kind、status、reason、owner、switch trigger | applies/not_applicable_with_reason | REQ-008 |

## 状态机

### Input → Component

| State Machine ID | 当前状态 | 触发 / 条件 | 合法转换 | 非法转换处理 |
|---|---|---|---|---|
| SM-166-01 | unvalidated | required input 缺失且无矛盾 | typed_unavailable | 不得进入 producer PASS 路径 |
| SM-166-02 | unvalidated | 时间/数值/hash/membership 冲突、未授权 ref、篡改 | blocked | 保留 reason；禁止 consumer projection 为 PASS |
| SM-166-03 | unvalidated | 7/7 字段族、policy、lineage、authorization 全通过 | validated | 仅 validated input 可计算 |
| SM-166-04 | validated | 纯函数计算完成且 evidence 自校验通过 | present + pass/fail/needs_review | hash/ref 不一致回 blocked |
| SM-166-05 | any | event-specific 语义未冻结 | not_applicable_with_reason | 不创建空 component 或覆盖声明 |

### Availability 与 Outcome 决策表

| 条件 | Availability | Outcome | Mandatory C2 claim |
|---|---|---|---|
| required input absent、无矛盾 | `typed_unavailable` | N/A | blocked |
| invalid/contradictory/tampered/unauthorized | `blocked` | N/A | blocked |
| strategy semantic 明确不适用且 reason/owner/trigger 完整 | `not_applicable_with_reason` | N/A | 若 C2 mandatory 则不能 PASS；event compatibility 可接受为 N/A |
| evidence complete，任一 mandatory fold/metric threshold fail | `present` | `fail` | fail |
| evidence complete但显式 policy 标为人工复核 | `present` | `needs_review` | needs_review/blocked，依 consumer tier |
| evidence complete且所有 mandatory decision pass | `present` | `pass` | eligible；仍不等于 runtime/Stage3 readiness |

保守严重度用于 claim 合并：`blocked > typed_unavailable > fail > needs_review > pass`；`not_applicable_with_reason` 只处理适用性，不能作为 mandatory C2 PASS 的替代。

### Component Catalog 决策表

| Component classification | 当前目录状态 | Mandatory 时 | Optional 时 |
|---|---|---|---|
| `walk_forward_oos@v1` | active | 按 C2 schema 校验并计算 | 同左 |
| `economic_cost` / `capacity_liquidity` | reserved；calculator=0 | typed_unavailable，不能满足 mandatory | 可声明 deferred descriptor，但无 present/PASS |
| unknown type/version | unknown | blocked：`mandatory_component_unknown` | 原样保留作 forward-compatible audit，但不参与 outcome/claim |

## 业务规则

| Rule ID | 规则 | Owner | 影响场景 | 验证入口 |
|---|---|---|---|---|
| RULE-166-01 | 7/7 输入字段族必须有 schema 与 validation result | FEAT-01/02 | P01/P02/N01 | QAC-04 |
| RULE-166-02 | 时间使用 ISO-8601 normalized value 与 `[start,end)`；每 fold 满足 `train_start < train_end <= validation_start < validation_end <= oos_start < oos_end` | FEAT-02 | N02 | QAC-05 |
| RULE-166-03 | fold id 唯一、manifest 顺序稳定、cutoff 单调；不得删除 invalid fold 后重算分母 | FEAT-02/03 | N01/N02 | denominator fixture |
| RULE-166-04 | overlap applicable 时 purge policy/ref/required/applied 100% 存在，`applied >= required` | FEAT-02 | N03 | QAC-05 |
| RULE-166-05 | embargo 有明确 unit/ref/required/applied，`applied >= required`；one-below blocked、exact-boundary eligible | FEAT-02 | N04 | QAC-05 |
| RULE-166-06 | mandatory metric 缺失、NaN、Inf 3/3 fail-closed；present evidence 中 non-finite=0 | FEAT-02/03 | N05 | QAC-01 |
| RULE-166-07 | fold outcome 从 metric policy 重算；pass rate=`passed declared folds / declared fold count`，不得排除失败或缺失 fold | FEAT-03 | P01/N01 | QAC-03 |
| RULE-166-08 | lineage missing→typed_unavailable；ref/hash/membership mismatch→blocked；orphan refs=0 | FEAT-02 | N06 | QAC-06 |
| RULE-166-09 | canonical hash 必须使用 explicit domain；相同 normalized fixture 10 次 distinct hash=1 | FEAT-01/03 | P01/H01 | QAC-07 |
| RULE-166-10 | old hash + tampered canonical field→blocked；semantically equivalent component ordering 必须 canonical-equal | FEAT-01/03 | H01 | integrity fixture |
| RULE-166-11 | 三个 consumers 只消费同一 component ref/hash/availability/reasons，projection 3/3，不新建 gate | FEAT-04 | P01 | QAC-08 |
| RULE-166-12 | projection 采用 worse-state merge；CR155 `paper_candidate=false`/blocked 1/1 保持 | FEAT-04/05 | regression | QAC-12 |
| RULE-166-13 | daily+ML P0 compatibility=2/2；ML policy 不能替代显式 fold bounds，缺 bounds 仍 unavailable | FEAT-02/05 | P01/P02 | QAC-02 |
| RULE-166-14 | event applicability=1/1 N/A；event producer/fixture 数=0，feed access=0 | FEAT-05 | E01 | QAC-03 |
| RULE-166-15 | external dereference=0、forbidden operations=0、C3/C4 calculators=0 | FEAT-05 | A01/H01 | QAC-09/10 |
| RULE-166-16 | Stage2 complete=true；Stage3 started/runtime-authorized/real-evidence-available=false | FEAT-05 | release claim | QAC-11 |

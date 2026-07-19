---
feature_id: "FEAT-173-01"
status: "lld-r2-remediation-synced-awaiting-r3-independent-review"
version: "0.3"
change_id: "CR-173"
validation_mode: "fixture-static-mixed"
story_count: 3
wave_count: 3
public_c1_projection_story_count: 0
---

# FEAT-173-01 Effective-Trial Offline Estimator — TEST-PLAN

## 修订记录

| 版本 | 日期 | 修订人 | 变更 |
|---|---|---|---|
| 0.1 | 2026-07-16 | meta-se-critical | 冻结 estimator-only 分层测试、六类 golden vectors、八类 failure、3/3 确定性与 public-boundary/authorization 零操作验收。 |
| 0.2 | 2026-07-16 | meta-se-critical | 处理 CP5 Round-1 required findings：stable computation/evidence + external audit oracle、F03/F04 唯一 mapping、public 新代码零集成与 12/12 只读回归分 lane。 |
| 0.3 | 2026-07-16 | meta-se-critical | 处理 CP5 Round-2 F-R2-002/003：把含糊的 forbidden operation 数量冻结为非 public `9/9`，单列 public 双 lane 六项指标并增加零重复计数 oracle；刷新 S01/S02 LLD v1.2、S03 LLD v1.3/R3 复核时点。 |

## 1. 测试目标与边界

验证对象仅为未来 CP5 批准后实现的 repository-local pure contracts、exact-rational estimator、standalone evidence 和 fixture/static guard。测试不得读取真实数据、环境凭据、provider、lake/NAS，不得执行 runtime/trading/publish，不得修改或调用 public C1 positive projection。

| 指标 | 目标 |
|---|---:|
| REQ / SC 覆盖 | 8/8 / 8/8 |
| golden classes / repeats | 6/6 / 每类 3/3 |
| 合法组 result/hash | 每组 1/1 |
| 每组 stable computation ref / 每次 attempt audit ref | 1 / 每次 1；3 repeats 对应 1 + 3 |
| failure classes fail-closed | 8/8 |
| present evidence schema | 7/7 |
| raw fallback / orphan refs / strategy identity | 0 / 0 / 0 |
| public projection Story | 0 |
| CR173 new-code public dependency edge/call/production diff/write | 0/0/0/0 |
| CP7 read-only public regression inventory / existing expected edits | 12/12 / 0 |
| 非 public deny-default operation class / class duplicate | 9/9 各 counter=0 / 0 |
| public 六项指标 | new-code 四项均0；read-only inventory=12/12；expected edits=0 |

## 2. 分层测试计划

| 层 | Story | 验证对象 | 入口文件 | 结论门槛 |
|---|---|---|---|---|
| Contract unit | S01 | immutable objects、七字段、stable computation ref、外置 audit、status/reason、canonical number/bytes/hash | `tests/research/test_effective_trial_evidence_contracts.py` | schema 7/7；failure不 present；3/3为1 computation/hash+3 audits |
| Numeric unit | S02 | exact decimal parser、matrix validator、LDLᵀ、formula、双 invariant、half-even | `tests/research/test_effective_trial_estimator.py` | valid analytic values全相等；float bridge=0 |
| Fixture integration | S03 | six vectors×3、external append-only audit recovery、stable computation/evidence hash | `tests/research/test_effective_trial_cr173_qac.py` | 6/6×3/3；每组1 computation/hash+3 audits；失败 available=0 |
| Static authorization | S03 | CR173 new-code import/dependency/call/file/write/claim boundary | `tests/research/test_effective_trial_authorization.py` | dependency edge/call/diff/write=0/0/0/0 |
| Existing regression read-only | S03 | 12 个 public C1 regression/authorization 文件的 current-unavailable 语义；其既有 public 调用不计入 new-code counter | 只读既有测试入口 | inventory=12/12；existing expected edits=0；选择性回归 PASS |

## 3. Golden vectors v1

所有 matrix token 都是 canonical decimal 字符串，source mode=`declared_exact`，trial IDs 是 synthetic IDs。每个 vector 在相同 normalized input、method version/hash 下重复三次。

| Vector ID | 类别 | n / matrix 摘要 | 预期 state/count | 3/3 断言 |
|---|---|---|---|---|
| GV-ET-01 | independent | n=4，`R=I₄` | present / `4` | stable computation/evidence hash=1/1；external audit refs=3/3；raw provenance独立 |
| GV-ET-02 | positive correlation | n=4，equicorrelation `ρ=0.5` | present / `2.285714285714` | exact `16/7` 后一次 half-even；computation/hash=1/1；audits=3/3 |
| GV-ET-03 | fully correlated | n=4，`R=11ᵀ` | present / `1` | singular PSD zero-pivot稳定；computation/hash=1/1；audits=3/3 |
| GV-ET-04 | singleton | n=1，`R=[[1]]` | present / `1` | 下界/上界通过；computation/hash=1/1；audits=3/3 |
| GV-ET-05 | invalid token | n=2，含 `"NaN"`/`"Inf"` 或其他 non-canonical string token | typed_unavailable / F03 / null | parser前拒绝；F04命中=0；computation/hash=1/1；audits=3/3 |
| GV-ET-06 | provenance/hash mismatch | valid matrix，固定单一 labels/input hash mismatch | blocked / F06 / null | available/PASS=0；computation/hash=1/1；audits=3/3 |

fixture 文件必须显式携带 `case_id`、schema/version、ordered trial IDs、matrix tokens、source mode、method ID/version/hash、expected state/reason/count token。不得携带 strategy identity 或真实数据引用。

## 4. 八类 fail-closed 测试

| Failure ID | 注入 | 预期 state/reason | 禁止结果 |
|---|---|---|---|
| F-01 | sealed identity 缺失 | typed_unavailable / `missing_sealed_identity` | present、raw fallback |
| F-02 | matrix/metadata 缺失 | typed_unavailable / `missing_dependency_matrix` | estimator call |
| F-03 | unsupported representation/source 或任一 non-canonical token（含 `NaN`/`Inf`/exponent/负零） | typed_unavailable / `unsupported_dependency_representation` | implicit conversion；进入 matrix validator |
| F-04 | token 已全部解析为有限 exact rational 后 shape/symmetry/diag/range/PSD 非法 | typed_unavailable / `invalid_dependency_matrix_domain` | 将 grammar/NaN/Inf 双重映射到 F04；tolerance/clamp |
| F-05 | method spec 缺失 | typed_unavailable / `missing_method_spec` | default method |
| F-06 | labels/count/input hash/lineage 矛盾 | blocked / `identity_or_input_integrity_mismatch` | forged present |
| F-07 | method version/hash 与批准 spec 不一致 | blocked / `method_spec_mismatch` | auto-upgrade/downgrade |
| F-08 | orphan ref、canonical hash/invariant 矛盾 | blocked / `evidence_integrity_mismatch` | overwrite old attempt |

每类都必须断言七个顶层键存在、count=null、stable computation ref非空；同一 F01-F08 canonical basis 的 3/3 repeats 只有 1 computation ref/evidence hash，并有 3 个外置 audit refs。F-01..F-05 的已验证 method/input metadata 可保留但不得合成；F-06..F-08 必须保留足以审计矛盾的 ref，禁止把 tamper 降级成普通 unavailable。

F01-F08 attempt-basis oracle 必须逐项比对 `basis_schema`、`validation_stage`、3-bit presence bitmap、4 个 component snapshot digest/absent marker、validated refs/null marker、primary failure ID 与 outcome。F03 的 dependency snapshot 对 raw invalid string token tree 做 restricted canonical digest；F04 的 snapshot 必须来自有限 exact-rational matrix。run/case/ordinal/audit ref 出现在 basis 的接受数=`0`。

## 5. 数值测试矩阵

| 维度 | 正向 | 边界/失败 | Oracle |
|---|---|---|---|
| decimal grammar / F03 | `0`,`1`,`-1`,`0.5`,`-0.5` | `NaN`、`Inf`、exponent、`-0`、`.5`、`01`、`1.0`、>12 decimals、trailing zero | parser前 exact grammar；F04 calls=0 |
| matrix domain / F04 | 全 token 已是 finite exact rational 的 symmetric unit-diagonal PSD | ragged、asymmetric、diag≠1、out-of-range、indefinite；label/hash mismatch另走F06 | exact comparisons |
| singular PSD | fully correlated、rank-deficient valid case | zero pivot + residual coupling | exact LDLᵀ branch |
| formula | identity、ρ=.5、all-ones、singleton | denominator/invariant contradiction fixture | rational equality |
| rounding | exact integer、repeating rational `16/7`、half-even tie vectors | second invariant violation injection | expected numeric token |
| serialization | key sorting、UTF-8 domain、unique numeric token | Decimal→float bridge、alternate equal token | byte-for-byte/hash |

S02 LLD 必须补齐 exact LDLᵀ pivot comparator 的至少 4 个矩阵 oracle：positive definite、singular PSD、indefinite negative pivot、zero pivot residual coupling。

## 6. 场景与需求覆盖

| Scenario | Requirement | Story | 测试入口 | 验收 |
|---|---|---|---|---|
| SC-CR173-P01 | REQ-001 | S02 | estimator unit + GV-01..04 | method-derived；alias/overclaim=0 |
| SC-CR173-Q01 | REQ-002 | S01/S02 | contract inventory + LLD gate | 输入/有效域/缺失行为=100% |
| SC-CR173-F01 | REQ-003 | S01/S03 | F-01..08 + recovery | old failure保留，新 ref/version |
| SC-CR173-N01 | REQ-004 | S01/S03 | seven-field mutation/orphan tests | 7/7；缺一 present=0 |
| SC-CR173-B01 | REQ-005 | S01/S03 | fixture schema + authorization scan | strategy/real identity=0 |
| SC-CR173-D01 | REQ-006 | S02/S03 | GV-01..06×3 | 每合法组1 result/hash |
| SC-CR173-C01 | REQ-007 | S01/S03 | boundary diff/call/claim guard | standalone=1；projection/write=0/0 |
| SC-CR173-A01 | REQ-008 | S03 | authorization/workflow assertions | forbidden=0；CR172 auto-resume=0 |

覆盖并集：REQ `8/8`，SC `8/8`。SC-C01 只通过 negative/static boundary 验证，不创建或调用 public projection。

## 7. Public-boundary 与授权守卫

### 7.1 禁止写入生产路径

S03 必须把以下路径登记为 read-only/forbidden，并验证 `cr173_new_code_public_dependency_edges=0`、`cr173_new_code_public_calls=0`、`public_production_diff=0`、`public_writes=0`：

- `engine/experiment_family_lineage.py`
- `engine/experiment_family_lineage_store.py`
- `engine/strategy_admission_statistical_gate.py`
- `engine/statistical_evidence.py`
- `engine/multiple_testing_evidence.py`
- `engine/overfit_evidence.py`
- `engine/cross_strategy_reliability_gates.py`
- `engine/strategy_admission_package.py`

### 7.2 既有回归清单

未来 CP7 的选择性回归必须满足 `cp7_read_only_public_regression_inventory=12/12`，并证明 current public effective-trial 仍 unavailable、Gate1 blocker/worst-state 未被改善。执行这些既有测试时发生的 current public type/adapter/gate 调用只属于 read-only verification lane，不计入 `cr173_new_code_public_calls`。`existing_expected_edits=0`；若任一回归需要修改 expected 才通过，结论为 `NEEDS_DESIGN_CLARIFICATION`，不得把修改并入 CR173。

### 7.3 零操作计数

非 public deny-default operation inventory 固定为 `9/9`，每个 class 有唯一 counter owner=S03 authorization verifier：

| Class ID | Operation class | Counter | Probe/guard 边界 | PASS |
|---|---|---|---|---:|
| NP-01 | credential | credential/env/account read | 0 |
| NP-02 | real data | real data read；不含 lake/NAS/provider | 0 |
| NP-03 | lake/NAS | lake/NAS read or write | 0 |
| NP-04 | provider/network | provider/network fetch；不含 QMT/trading/Git remote | 0 |
| NP-05 | catalog/store/pointer | catalog/store/current-pointer write | 0 |
| NP-06 | strategy runtime | strategy runtime/external framework/simulation；不含 QMT/trading | 0 |
| NP-07 | QMT/trading | QMT/broker/trading | 0 |
| NP-08 | publish/deploy | publish/deploy | 0 |
| NP-09 | Git remote | Git remote write | 0 |

唯一归类 oracle：每个 probe/guard 只能声明一个 NP-01..09 `operation_class_id`，九个 class ID 必须 `9/9` 唯一，重复/缺失/额外均为失败；cross-cutting policy tags 不递增 counter。NP-03 不再计 NP-02/04，NP-07/09 不再计 NP-04。

public 边界使用独立六项指标：`cr173_new_code_public_dependency_edges=0`、`cr173_new_code_public_calls=0`、`public_production_diff=0`、`public_writes=0`、`cp7_read_only_public_regression_inventory=12/12`、`existing_expected_edits=0`。它们不属于 NP-01..09；existing public calls 只属于 read-only lane。验证报告必须分别输出 `non_public_operation_inventory=9/9` 与 public 六项指标，禁止重复计数或用 public 项补成第十类。

## 8. 前置条件与失败路由

| 阶段 | Entry | PASS | FAIL route |
|---|---|---|---|
| CP5 设计证据 | S01/S02 LLD v1.2、S03 LLD v1.3 共3/3均为 ready-for-review；Round-2 整改已同步 | Round-3 独立复核 required=0 后才可准备人工门禁 | meta-se/meta-dev design clarification；实现数保持0 |
| S01/S02 unit | CP5 approved；依赖/ownership满足 | 全量 Feature unit PASS | meta-dev rework；不进入S03 |
| S03 fixture/static | S01/S02 evidence通过；fixture-only授权有效 | 6×3、8/8、NP-01..09各0、class duplicate=0 | NEEDS_REWORK/BLOCKED |
| public regression | 生产 diff=0、read-only test run | 12/12语义不恶化 | NEEDS_DESIGN_CLARIFICATION；future projection CR |

## 9. 回滚验证

- 删除/禁用新 estimator-only source 与 fixtures 后，现有 public C1 行为、schema、Gate1 blocker、admission worst-state必须 byte/semantic-equivalent。
- 任何 failed audit 不能原地改写；修正输入创建新 canonical basis/computation ref/evidence hash/audit，并通过 parent/supersedes 链接旧 failure audit。
- 若 exact algorithm、canonical renderer 或 hash domain 无法跨重复保持唯一，整个 Feature 回到 typed unavailable，不允许降级为 binary float、tolerance PSD 或 raw count。

## 10. Gotchas

- static boundary test 不得新增“临时 public adapter”来证明拒绝；这会把 projection Story 偷渡进本 CR。
- non-canonical token（含 NaN/Inf）与 tamper/mismatch 的 state 不同，不能只测试 `count is None`。
- non-canonical token（含 `NaN`/`Inf`）只能 F03；F04 fixture 必须由已解析 finite exact rationals 构成。
- repeat hash 唯一要求 canonical evidence 内容唯一；stable computation ref 进入七字段，逐执行 audit ref 外置。3/3 的正确 oracle 是 `1 computation ref + 1 evidence hash + 3 attempt audit refs`。
- 12/12 regression 内的既有 public 调用是验证行为，不是 CR173 新集成边；但任何 expected edit、production diff/write 或新 code dependency/call 仍为失败。
- public 六项指标与 NP-01..09 不共享 counter；一个 read-only regression invocation、guard 或 static probe 不得跨 inventory 重复记账。
- public regression PASS 不代表 C1 computable；它恰恰证明 C1 继续 unavailable。

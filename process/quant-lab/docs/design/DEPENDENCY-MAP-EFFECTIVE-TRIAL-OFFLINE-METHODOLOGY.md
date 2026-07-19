---
status: draft-awaiting-cp3
version: "1.0"
change_id: CR-173
---

# Dependency Map：Effective-Trial Offline Methodology

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-16 | meta-se-critical | 冻结 estimator-only 依赖方向，并完成 public C1 contract 触达分类。 |

## 允许依赖

| From | To | 类型 | 方向 | 原因 | 验证 / 监控 |
|---|---|---|---|---|---|
| estimator input validator | sealed lineage identity | read-only contract | allowed | 证明 `n`、trial IDs 与 provenance | SC-CR173-Q01/B01 |
| estimator core | validated dependency envelope | pure data | allowed | 公式只消费 canonical matrix | SC-CR173-P01/D01 |
| standalone evidence builder | estimator result + method spec | pure projection | allowed | 形成七字段 typed evidence | SC-CR173-N01 |
| golden-vector verifier | standalone estimator contract | fixture/static | allowed-after-CP5 | 6 类 × 3 次确定性验证 | SC-CR173-D01 |
| public C1 migration analysis | standalone evidence schema | read-only design | allowed | 为后续 CR 建 migration 输入 | DO-CR173-CP3-002 |

## 禁止依赖

| Forbidden ID | From | To | 禁止原因 | 替代路径 | 违反风险 |
|---|---|---|---|---|---|
| FD-ET-01 | estimator | raw count fallback | 会把未识别依赖伪装成有效试验 | typed_unavailable | 虚假精度、C1 overclaim |
| FD-ET-02 | estimator | returns/lake/provider/runtime | CR-173 不估计 correlation matrix，且无真实数据授权 | 由未来 activation 提供 sealed matrix 及误差证据 | scope/authz escape |
| FD-ET-03 | estimator | strategy identity | 方法必须 strategy-agnostic | synthetic case ID | 偷换为 activation |
| FD-ET-04 | CR-173 evidence builder | FamilyEvidenceProjection/C1InputStatus write | current public type 明示 effective count 禁止且 count 为 int | 后续 versioned projection CR | 非兼容公共 API、跨 owner 写入 |
| FD-ET-05 | CR-173 evidence builder | StatisticalEvidenceSummary/Gate-1/admission package write | 三个 consumer adapter 和 worst-state 语义由其他 owner 控制 | 后续 CR 做 adapter + migration + regressions | competing truth、状态改善错误 |
| FD-ET-06 | estimator | DSR/FWER/admission claim | participation ratio 不是 tail/alpha calibration | 保留 method limitation；需要时转 Spike | 方法 overclaim |

## Public C1 contract touch classification

结论：`cross-owner + cross-domain + non-compatible`。因此 CR-173 的 public projection 必须拆出，当前 HLD 为 estimator-only。仓库级只读 `rg` 对定义、调用与有效字段断言进行全集扫描；下表覆盖所有发现的生产调用路径 `8/8`。

| # | 生产路径 / 对象 | 调用方向与时机 | 当前输入 / 输出 | 触达分类 | 调用方同步修改面 | CR-173 disposition |
|---:|---|---|---|---|---|---|
| 1 | `engine/experiment_family_lineage.py`：`FamilyLineageValidationResult`、`FamilyEvidenceProjection`、`C1InputStatus` | lineage validation→projection | 4 个 legacy effective aliases；`int|None`；强制 unavailable | public type + invariant，非兼容 | dataclass、validation invariant、enum、serialization | deferred |
| 2 | `engine/experiment_family_lineage_store.py`：`_validation_from_dict` | store deserialize→lineage result | 反序列化 legacy availability/type | persistence contract，非兼容 | schema version、reader/migration | deferred |
| 3 | `engine/strategy_admission_statistical_gate.py`：`consume_family_lineage_projection` | lineage→statistical gate；三个既有 consumer 共用 | 拒绝 serialized positive truth并硬编码 unavailable | public adapter，跨 owner | trust binding、malformed rules、reconciliation、gate tests | deferred |
| 4 | `engine/statistical_evidence.py`：`project_summary` | method summary→三类 consumer | 4 legacy fields硬编码 unavailable | public summary projection，非兼容 | schema version、summary hash/domain、consumer tests | deferred |
| 5 | `engine/multiple_testing_evidence.py` | BH/WRC/SPA→StatisticalEvidenceSummary | 间接上游；无七字段直接输出 | indirect C1 method producer | evidence aggregation compatibility | no direct change in CR-173 |
| 6 | `engine/overfit_evidence.py`：DSR | raw trial count→DSR method evidence | 明示 raw input且断言 effective unavailable | method semantic dependency | DSR config/result/provenance/non-alias tests | deferred; no DSR calibration claim |
| 7 | `engine/cross_strategy_reliability_gates.py`：Gate 1 / `project_computable_statistical_evidence` | lineage/summary→`trial_count_and_effective_trials` | `float` slot但 projection 强制 blocked；legacy aliases | cross-domain gate contract | forced blocker、artifact validity、claim floor、Gate1 tests | deferred |
| 8 | `engine/strategy_admission_package.py`：attach lineage/computable evidence | projections→CR151/CR155 package | worst-state merge + hard-coded unavailable limitations | admission package contract | limitations、evidence refs、package status、CR155 regression | deferred |

### 回归 / authorization inventory（12/12）

| 测试路径 | 锁定的 contract 语义 | 后续 projection CR 必须同步 |
|---|---|---|
| `tests/research/test_experiment_family_lineage_contracts.py` | int/legacy aliases、C1 RAW_INPUT_READY、effective claim forbidden | 是 |
| `tests/research/test_trial_lineage_integrity.py` | 10 次 determinism、effective unavailable | 是 |
| `tests/research/test_trial_lineage_authorization.py` | project/attach authorization 边界 | 是 |
| `tests/research/test_trial_lineage_admission_projection.py` | 三 consumer 同一 projection、forged serialized input blocked | 是 |
| `tests/research/test_trial_lineage_legacy_admission_regression.py` | CR155 legacy package保持 blocked | 是 |
| `tests/research/test_cross_strategy_reliability_gates.py` | Gate1 trial slot、forced effective blocker、approximation review | 是 |
| `tests/research/test_statistical_evidence_projection.py` | 三 adapter映射、无新 gate、worst-state | 是 |
| `tests/research/test_statistical_evidence_contracts.py` | summary hash与 effective unavailable | 是 |
| `tests/research/test_statistical_evidence_qac.py` | 三 consumer IDs、negative fail-closed | 是 |
| `tests/research/test_statistical_evidence_cr155_regression.py` | passing summary不能改善 lineage-blocked CR155 | 是 |
| `tests/research/test_overfit_evidence.py` | DSR raw input/non-alias | 是 |
| `tests/research/test_statistical_evidence_authorization.py` | pure calculator/forbidden dependency boundary | 是 |

## 调用契约结论

| 要素 | Estimator-only（CR-173） | Public projection（future CR） |
|---|---|---|
| 调用方向 | fixture harness→input validator→estimator→standalone evidence | trusted standalone evidence→versioned adapter→statistical/Gate1/package consumers |
| 调用时机 | CP5 批准后的 fixture-only execution | 独立 CP2/CP3/CP5 与 owner approval 后 |
| 输入 | sealed identity + explicit canonical matrix + method spec | 七字段 evidence + validation binding + migration version |
| 输出 | standalone typed evidence | public C1 positive/unavailable/blocked projection |
| 降级 | unavailable/blocked；raw fallback=0 | 旧 consumer 保持 typed_unavailable；不允许 dual positive truth |
| 同步修改面 | 新 estimator 本地对象与 fixtures（CP3 不实现） | 上述 8 个生产路径 + 12 个回归/授权文件 |

## 循环风险

| Cycle ID | 涉及对象 | 风险 | 当前处理 |
|---|---|---|---|
| CYCLE-ET-01 | lineage→estimator→lineage projection | estimator 若回写 lineage 会形成 owner/serialization 环 | eliminated：standalone evidence，不回写 lineage |
| CYCLE-ET-02 | summary→Gate1→admission→method readiness | admission 状态反向决定 estimator 可用性会循环 | eliminated：method availability只由 input/method contract决定 |
| CYCLE-ET-03 | CR173 completion→CR172 activation→real matrix→CR173 | 自动恢复会越权 | eliminated：五字段 + fresh precheck + reopened CR172 CP2 |

## Gotchas

- `multiple_testing_evidence.py` 是间接方法生产者，不能因为没有七字段就从 inventory 中删除。
- `cross_strategy_reliability_gates.TrialCountAndEffectiveTrials` 接受 float 不代表整个 public path 兼容；上游 FamilyEvidenceProjection 仍是 `int|None`，而且 consumer 强制 unavailable。
- 只新增七字段同时保留 legacy aliases 会产生双真相；后续 CR 必须定义 versioned migration，不能靠“同时写两套”规避 breaking classification。

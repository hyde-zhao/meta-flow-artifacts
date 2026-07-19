---
feature_id: "FEAT-173-01"
title: "Effective-Trial Offline Estimator"
status: "lld-r2-remediation-synced-awaiting-r3-independent-review"
version: "0.3"
change_id: "CR-173"
boundary: "estimator-only"
source_hld: "docs/design/HLD-EFFECTIVE-TRIAL-OFFLINE-METHODOLOGY.md"
source_adr: "docs/design/ARCHITECTURE-DECISION-EFFECTIVE-TRIAL-OFFLINE-METHODOLOGY.md"
story_count: 3
wave_count: 3
task_count: 12
public_c1_projection_story_count: 0
---

# FEAT-173-01 Effective-Trial Offline Estimator — DESIGN

## 修订记录

| 版本 | 日期 | 修订人 | 变更 |
|---|---|---|---|
| 0.1 | 2026-07-16 | meta-se-critical | CP3 批准后建立 estimator-only Feature 实现设计：冻结合同/证据、精确有理数 estimator、golden/边界验证三个 Story；public C1 projection Story 为 0。 |
| 0.2 | 2026-07-16 | meta-se-critical | 关闭 CP5 Round-1 F-001..003 的 Feature 基线问题：stable computation ref + 外置 attempt audit、F03/F04 唯一映射、public 新代码/只读回归分 lane；刷新三份 LLD 已存在但待 resync 的状态。 |
| 0.3 | 2026-07-16 | meta-se-critical | 处理 CP5 Round-2 F-R2-002/003：冻结非 public operation classes `9/9`、public 双 lane 六项指标与禁止重复计数规则；刷新 S01/S02 LLD v1.2、S03 LLD v1.3 已同步 R2 整改并待 R3 独立复核。 |

## 1. 目标与成功标准

本 Feature 在仓库本地、synthetic/fixture/golden-vector 范围内，把显式提供且与 sealed trial IDs 对齐的 canonical correlation matrix 转换为谱 participation-ratio 二阶 effective dimensionality，并生成独立七字段 typed evidence。它不估计相关矩阵，不读取真实数据，不接入 public C1、Gate 1、DSR 或 admission consumer。

| 指标 | 目标 |
|---|---:|
| Feature / Story / Wave / Task | 1 / 3 / 3 / 12 |
| REQ-CR173 覆盖 | 8/8 |
| SC-CR173 覆盖 | 8/8 |
| 七个顶层字段 | 7/7 |
| golden-vector 类别 / 每类重复 | 6/6 / 3/3 |
| 合法组 canonical result/hash | 每组 1/1 |
| 同 basis computation ref / 每次执行 attempt audit ref | 1 / 每次 1；3/3 repeat 为 1 ref + 3 audits |
| 枚举失败类 fail-closed | 8/8 |
| raw fallback / strategy identity / real operation | 0 / 0 / 0 |
| 非 public deny-default operation classes | 9/9；各 counter=0 |
| public C1 projection Story | 0 |
| CR173 新代码 public dependency edge/call/diff/write | 0/0/0/0 |
| CP7 read-only public regression / existing expected edit | 12/12 / 0 |

## 2. Feature 边界

### 2.1 In Scope

- `SealedTrialIdentity`、`DependencyMatrixEnvelope`、`EffectiveTrialMethodSpec`、`EffectiveTrialEvidence` 的仓库内纯数据合同。
- canonical decimal token、exact base-10 rational、versioned method/spec hash 与 canonical JSON number renderer。
- deterministic lexicographic symmetric-pivot、fraction-free exact-rational `LDLᵀ` PSD 校验。
- `n_eff=n²/ΣRᵢⱼ²` 的 exact-rational 计算、未舍入/舍入后双 `[1,n]` invariant、一次 round-half-even 输出。
- 七字段 present / `typed_unavailable` / `blocked` evidence；第七字段为 stable content-addressed computation ref。
- 外置 `ComputationAttemptAudit` 合同：每次 repository-local verification execution 一个 append-only audit ref，不进入七字段或 evidence hash；当前 persistence/retention=N/A。
- 六类 synthetic golden vectors、3/3 repeat、失败恢复、strategy-agnostic 与 public-boundary 静态守卫。

### 2.2 Out of Scope

- 估计真实 correlation matrix、采样误差、stability、tail/copula、alpha/FWER/Šidák、Li–Ji、BH、DSR 或 admission calibration。
- `strategy_id` / `strategy_name`、真实 run identity、five-field activation scope、producer binding。
- public `FamilyEvidenceProjection`、`StatisticalEvidenceSummary`、Gate 1、DSR、admission package 的字段、adapter、migration、write 或 blocker 解除。
- lake/NAS/provider/credential/env/account、runtime、QMT/broker、simulation/paper/live trading、publish/deploy、Git remote write。
- 本设计阶段的源码、测试、fixture 实现；所有实现必须等待 CP5 批准。

### 2.3 相邻能力边界

| 相邻对象 | 本 Feature 的职责 | 相邻对象职责 | 禁止重叠 |
|---|---|---|---|
| experiment-family lineage | 只消费 fixture/static sealed identity 合同 | 持有真实 lineage、trial IDs 与 store | 不重放、不修复、不写 lineage store |
| dependency-evidence producer | 校验显式 matrix | 未来生成真实/empirical matrix 并证明误差/稳定性 | 不从 returns、目录、manifest 或 raw count 推断 matrix |
| public C1 contract | 只读 inventory 与 boundary guard | 未来 versioned projection、trust binding、migration、consumer regression | 不双写 legacy/new positive truth |
| reliability/admission | 证明当前状态没有被改善 | 拥有 Gate 1、worst-state 与 admission policy | 不新增 competing gate，不解除 blocker |

### 2.4 Deny-default operation-class inventory

非 public operation class 的单一真相源固定为下列 `NP-01..09`。counter owner 均为 S03 authorization verifier；每个 guard/probe 只能登记一个 `operation_class_id`，cross-cutting policy tag 只作诊断，不得再次递增其他 class counter。

| Class ID | Operation class | 采集边界 / 非重复规则 | Counter target |
|---|---|---|---:|
| NP-01 | credential | credential/env/account read；不得复用为 provider/network counter | 0 |
| NP-02 | real data | 非 fixture real data read；lake/NAS 与 provider 来源分别归 NP-03/04 | 0 |
| NP-03 | lake/NAS | lake/NAS read or write；不再计 NP-02/04 | 0 |
| NP-04 | provider/network | provider/network fetch；QMT/trading 与 Git remote 分别归 NP-07/09 | 0 |
| NP-05 | catalog/store/pointer | catalog/store/current-pointer write；public production diff/write 另属 public lane | 0 |
| NP-06 | strategy runtime | strategy runtime/external framework/simulation；QMT/trading 归 NP-07 | 0 |
| NP-07 | QMT/trading | QMT/broker/trading | 0 |
| NP-08 | publish/deploy | publish/deploy | 0 |
| NP-09 | Git remote | Git remote write；不再计 NP-04 | 0 |

public 边界不属于 NP-01..09，必须独立报告六项指标：`cr173_new_code_public_dependency_edges=0`、`cr173_new_code_public_calls=0`、`public_production_diff=0`、`public_writes=0`、`cp7_read_only_public_regression_inventory=12/12`、`existing_expected_edits=0`。public read-only regression calls 不进入九类，九类 guard 命中也不进入 public 六项指标。

## 3. 模块与文件布局

```text
engine/
├── effective_trial_evidence.py                 # S01：输入/方法/状态/七字段合同、canonical serialization
└── effective_trial_estimator.py                # S02：exact validator、LDLᵀ、participation-ratio estimator
tests/
├── fixtures/effective_trial/
│   └── golden_vectors_v1.json                  # S03：六类 declared_exact fixtures
└── research/
    ├── test_effective_trial_evidence_contracts.py  # S01
    ├── test_effective_trial_estimator.py           # S02
    ├── test_effective_trial_cr173_qac.py            # S03
    └── test_effective_trial_authorization.py        # S03
```

以上为 CP5 批准后的预期布局；本轮文件创建数为 0。

## 4. 组件职责与调用契约

| 组件 | 调用方 → 被调用方 / 时机 | 输入 | 输出 | 失败/降级 | 后续衔接 |
|---|---|---|---|---|---|
| Contract objects | fixture harness → contracts；构造时 | sealed identity、matrix envelope、method spec | immutable typed values | 缺失/unsupported=`typed_unavailable`；矛盾/篡改=`blocked` | 交给 validator |
| Canonical serializer | evidence builder → serializer；evidence 边界 | 七字段对象、已验证 numeric token | 唯一 UTF-8 canonical bytes/hash | 非 canonical token 或缺键 fail-closed | 只生成 standalone artifact |
| Dependency validator | harness → validator；估计前 | identity + matrix + method | normalized exact-rational input | 八类失败之一；不调用 estimator | valid 才进入 estimator |
| Estimator core | validator → estimator；valid 后 | normalized matrix | exact rational + canonical count | invariant/serializer mismatch=`blocked` | 交给 evidence builder |
| Evidence builder | validator/estimator → builder；每个 canonical outcome | outcome、lineage、method、`EffectiveTrialAttemptBasisV1` | 七字段 present/unavailable/blocked + stable computation ref | count 只在 present 非 null；raw fallback=0；不得接收逐执行 audit ref | 到 standalone 边界即停止 |
| Attempt audit builder | S03 harness → audit；每次 verification execution 后 | run/case/ordinal、stable computation ref、evidence hash、parent/supersedes | 外置 immutable `ComputationAttemptAudit` | ref 重用/覆盖 fail-closed；不进入 evidence hash | 只进入 repository-local verification evidence |
| QAC/authorization verifier | CP7 harness → authorization inventory/public contract read-only inventory | fixture、source inventory、NP-01..09、public 六项指标 | 6×3、8 failure、9/9 non-public zero-operation + public 双 lane 证据 | 任一 class/public zero 非0、重复 class 或 overclaim 失败 | 只执行既有 read-only regressions，不新增 public adapter |

调用方向固定为：

```text
repository fixture
  -> contract construction
  -> exact validator
  -> spectral estimator (valid only)
  -> seven-field evidence builder
  -> standalone canonical bytes/hash
  -> external append-only attempt audit (not in evidence/hash)
  -> STOP at public C1 boundary
```

## 5. 数据合同

### 5.1 输入对象

| 对象 | 必填字段 | 关键约束 | 缺失/不支持 | 矛盾/篡改 |
|---|---|---|---|---|
| `SealedTrialIdentity` | family ref/hash、raw count `n`、ordered trial IDs | `n≥1`；IDs 唯一、canonical 顺序、数量=n | `typed_unavailable` | `blocked` |
| `DependencyMatrixEnvelope` | schema/version、labels、matrix tokens、input hash、lineage ref、source mode | n×n、finite、symmetric、diag=1、[-1,1]、PSD、labels 1:1 | `typed_unavailable` | `blocked` |
| `EffectiveTrialMethodSpec` | method ID/version/hash、formula/numeric/serialization descriptor | 与批准 v1 spec 完全一致 | `typed_unavailable` | `blocked` |

`source_mode` 在 CR173 v1 只允许 `declared_exact`，输入来源只允许 repository synthetic/fixture/golden-vector。empirical/real、超过 12 位小数、exponent、binary float 或 tolerance-PSD 要求均返回 `typed_unavailable`。

### 5.2 七字段 evidence

| 顶层字段 | present | unavailable / blocked |
|---|---|---|
| `effective_trial_count` | canonical JSON number，`1≤x≤n` | `null` |
| `effective_trial_count_status` | `{state:"present",reason_code:"ok"}` | 对应 state + reason enum |
| `effective_trial_method` | `spectral_participation_ratio` | 已验证时保留，否则 `null` |
| `effective_trial_method_version` | 非空 v1 version | 已验证时保留，否则 `null` |
| `effective_trial_method_hash` | canonical spec hash | 已验证时保留，否则 `null`；mismatch 为 blocked |
| `effective_trial_input_lineage_ref` | validation-bound ref | 可验证时保留，否则 `null`；orphan/mismatch 为 blocked |
| `effective_trial_computation_ref` | stable content-addressed computation identity | 始终非空；同 canonical basis/outcome 只产生 1 个 ref；不是逐执行 attempt ID |

reason code v1 枚举冻结为：

| state | reason code | 语义 |
|---|---|---|
| present | `ok` | input/method/domain/invariant 全部通过 |
| typed_unavailable | `missing_sealed_identity` | sealed identity 不足 |
| typed_unavailable | `missing_dependency_matrix` | matrix 或必要 metadata 缺失 |
| typed_unavailable | `unsupported_dependency_representation` | F03：非 v1 表示/source 或任一 non-canonical token；包含 `NaN`/`Inf`/exponent/负零，失败发生在 exact parser 前 |
| typed_unavailable | `invalid_dependency_matrix_domain` | F04：所有 token 已成功解析为有限 exact rational 后，shape/symmetry/diag/range/PSD 不满足 |
| typed_unavailable | `missing_method_spec` | method/version/hash 不完整 |
| blocked | `identity_or_input_integrity_mismatch` | labels/count/input hash/lineage 矛盾或 tamper |
| blocked | `method_spec_mismatch` | 已提供 spec/hash 与批准 v1 不一致 |
| blocked | `evidence_integrity_mismatch` | forged/orphan ref、canonical hash 或 invariant 矛盾 |

前八个非 `ok` reason 是 REQ-003 的 `8/8` failure classes。F03/F04 以 parser 成功为不可跨越的阶段边界；同一 token 不得双重映射。public projection attempt 不伪装成 estimator failure，而由 scope guard 直接拒绝并报告 `public_projection_forbidden` 的验证事件。

### 5.3 Stable computation ref、attempt basis 与外置 audit

| 对象 | Owner | Identity / schema | 是否进入七字段/evidence hash |
|---|---|---|---|
| `EffectiveTrialAttemptBasisV1` | methodology owner | schema、validation stage、presence bitmap、restricted component snapshot digests/absent markers、validated refs/null markers、F01-F08 primary failure、outcome | basis 本身不作为字段；用于生成 computation ref |
| `effective_trial_computation_ref` | methodology owner | `sha256(computation-domain-v1 + canonical attempt basis)`；同 basis/outcome 稳定 | 是；第七字段，因此进入 evidence hash |
| `CanonicalEvidenceHash` | methodology owner | evidence domain + 七字段 canonical bytes | evidence 外部索引；可重算 |
| `ComputationAttemptAudit` | schema owner=methodology owner；当前 lifecycle/write owner=S03 verification harness | audit ref、run/case/ordinal、computation ref、evidence hash、state/reason、parent/supersedes、safe diagnostic codes | 否；不得成为第八字段或改变 method/evidence hash |

attempt basis 的 canonical 字段必须完整存在；F01/F02/F05 用 presence bitmap + absent marker，F03 保存 non-canonical raw string token tree 的 restricted canonical digest，F04 保存已解析 finite exact-rational matrix digest，F06/F07 保存冲突 component digests，F08 保存 attempted-evidence digest。basis 禁止包含 run/case/ordinal/time/worker/random/audit ref。

`attempt_audit_ref = sha256(audit-domain-v1 + verification_run_ref + synthetic_case_id + attempt_ordinal + computation_ref + evidence_hash + parent/supersedes markers)`。3/3 repeat 必须得到 computation ref/evidence hash=`1/1`，attempt audit refs=`3/3` 唯一。failure recovery B 改变 basis 并链接/取代 audit A；A 保留。

当前 persistence=`N/A`、retention=`N/A`：只允许 immutable/in-memory repository-local verification collection；不创建 production audit store/catalog/pointer/writer。

## 6. 数值与算法合同

### 6.1 Canonical numeric grammar

- 输入 token 必须是普通十进制、无 exponent、最多 12 位小数；`NaN`/`Inf`、负零、前导零、整数 `.0` 和非必要尾零均在 parser 前唯一 F03，绝不进入 F04。
- token 解析为 `coefficient / 10^scale` 的 exact rational；任何路径禁止先转 binary float。
- trial IDs 的 canonical 顺序决定矩阵行列顺序、pivot tie-break 与 input hash。

### 6.2 Exact PSD validator

按 canonical trial-ID 顺序执行 deterministic lexicographic symmetric-pivot、fraction-free exact-rational `LDLᵀ`：

1. 每轮从剩余对角项中按“非零优先、绝对值/值规则与 lexicographic index tie-break”选择唯一 symmetric pivot；详细 pivot comparator 在 S02 LLD 冻结，不得依赖 tolerance。
2. 负 pivot 立即判定非 PSD。
3. 零 pivot 若对应 residual row/column 仍有非零 coupling，判定非 PSD；否则保留零维并继续。
4. 所有算术用 exact rational；输出只包含 normalized input 或 typed failure，不暴露浮点近似。

LLD 必须把 pivot comparator 写成可执行伪代码和 golden cases；若无法证明唯一性，S02 不得进入开发。

### 6.3 Estimator 与输出

对 valid correlation matrix：

`q = n_eff = n² / Σᵢⱼ Rᵢⱼ²`

执行顺序固定：

1. exact rational 计算平方和与 `q`；不要求 eigenvalue 求解。
2. 对未舍入 `q` 检查 `[1,n]`；失败为 `blocked`，禁止 clamp。
3. 只在 evidence 边界 round-half-even 到最多 12 位小数。
4. 对量化结果再次检查 `[1,n]`；失败为 `blocked`。
5. 由 coefficient/scale 直接渲染 JSON number token；禁止 `Decimal→float`。

canonical domain 固定为 `quant-lab.effective-trial-evidence.spectral-participation-ratio.v1`。method hash 覆盖 formula、input grammar、F03/F04 mapping/precedence、PSD 规则、输出范围/精度/舍入、renderer、七字段 schema、reason enum、attempt-basis v1 schema、stable computation-ref derivation与 canonical serialization。外置 audit schema/ref domain 单独版本化，不进入 method/evidence hash。

## 7. Story 与 Wave

| Wave | Story | 交付 | depends_on | 主要文件 owner |
|---|---|---|---|---|
| CR173-W1-CONTRACT-EVIDENCE | CR173-S01-contract-evidence-canonicalization | 输入/方法/七字段合同与 canonical serializer | none | `engine/effective_trial_evidence.py` |
| CR173-W2-EXACT-ESTIMATOR | CR173-S02-exact-spectral-estimator | exact validator、LDLᵀ、participation-ratio estimator | S01 contract | `engine/effective_trial_estimator.py` |
| CR173-W3-GOLDEN-BOUNDARY | CR173-S03-golden-failure-boundary-verification | 6×3、8 failures、append-only recovery、public-boundary/authorization guard | S01+S02 runtime | CR173 fixtures/QAC/auth tests |

三个 Story 均为 `full-lld`。CP4 已 PASS；S01/S02 LLD v1.2、S03 LLD v1.3 均为 `ready-for-review/confirmed=false`，已同步 Round-2 整改；CP5 Round-2 decision=`revise`。本 0.3 完成 operation-class/freshness 权威回写并等待 Round-3 独立复核，`design_evidence_confirmed/lld_confirmed` 仍为 false，实现继续锁定。

## 8. 追踪矩阵

| Story | REQ | SC | Outcome |
|---|---|---|---|
| S01 | 001,002,003,004,005,007 | Q01,F01,N01,B01,C01 | O01,O02,O04；stable identity/attempt audit owner |
| S02 | 001,002,003,005,006 | P01,Q01,F01,B01,D01 | O01,O03 |
| S03 | 003,004,005,006,007,008 | F01,N01,B01,D01,C01,A01 | O02,O03,O04；audit lifecycle + public read-only lane |

并集覆盖 REQ `8/8`、SC `8/8`。SC-C01 由“停止于边界”的 negative/static guard 覆盖，不构成 projection Story。

## 9. 前置校验、失败与回退

| 阶段 | 前置校验 | 失败行为 | 回退/恢复 |
|---|---|---|---|
| S01 LLD | ADR-001..005 approved；7 fields/reason enum/numeric grammar一致 | `NEEDS_DESIGN_CLARIFICATION` | 回 meta-se，不创建实现 |
| S01 开发 | CP5 approved；文件 owner 独占 | `BLOCKED` | 保持源码未改或回滚 S01 slice |
| S02 LLD | S01 contract frozen；pivot comparator可唯一证明 | `NEEDS_DESIGN_CLARIFICATION` | 回 estimator design/Spike |
| S02 开发 | S01 contract已合并；exact rational无 float bridge | `NEEDS_REWORK` | 回滚 S02，不影响 standalone unavailable contract |
| S03 验证 | S01/S02 implementation evidence通过；fixture-only authorization有效 | `BLOCKED` / `NEEDS_REWORK` | 保持 public C1 unavailable；修复后新 basis/computation/evidence + audit，旧 audit 保留 |
| public C1 需求 | 独立 projection CR 不存在 | 立即停止 | 登记 future CR，当前生产修改数保持 0 |

Feature 回滚单位是 estimator-only 新文件与 fixtures；因为不写既有 public consumer，回滚不需要迁移公共 schema。任何半成品或未验证结果只能保持 typed unavailable，不能用 raw count 补位。

## 10. 非功能与安全

| 维度 | 设计目标 | 验证 |
|---|---|---|
| 正确性 | exact `1≤n_eff≤n`，双 invariant，raw alias=0 | analytic + golden tests |
| 确定性 | 6 类各 3/3 仅 1 result/hash | QAC repeat |
| 可追溯性 | 7/7 stable computation ref；每执行1个外置 audit；orphan refs=0 | contract/negative/repeat/recovery tests |
| 安全 | NP-01..09=`9/9` 且各 counter=0；class duplicate=0 | authorization inventory + unique-class static scan |
| 兼容性 | new dependency edges/calls/diff/writes=0；read-only regressions=12/12；expected edits=0 | static inventory + selected regression lane |
| 可维护性 | method/numeric/schema/reason enum 同域版本化 | spec hash mismatch tests |

## 11. 风险与重访条件

| 风险 | 控制 | 剩余风险 / 重访 |
|---|---|---|
| 二阶 estimand 被误读为 FWER/DSR | method 名、limitation、claim scan | 需要 alpha/tail claim 时转 methodology Spike |
| exact LDLᵀ 实现不唯一或退化 | LLD 冻结 comparator，exact golden matrix | 无法证明时保持 unavailable |
| canonical number 经过 float 漂移 | coefficient/scale renderer，禁止 Decimal→float | 新 serializer/runtime 需重证 |
| 七字段被投影到 legacy 4-field/int contract | forbidden owner + zero-diff guard | future projection CR 独立门禁 |
| fixture exact 被外推真实可信 | source_mode 与 release wording | empirical matrix 需 future activation evidence |

## 12. Gotchas

- `n_eff=n` 可能是合法独立矩阵结果，但仍必须有独立 method/input/computation provenance；数值相等不等于 raw alias。
- fully correlated 与 singleton 都为 1；不能据此宣称 tail event、FWER 或 DSR 等价。
- `typed_unavailable` 与 `blocked` 不可互换：前者是缺失/unsupported/domain 不可用，后者是矛盾、篡改或 forged positive truth。
- canonical decimal 的 12 位是序列化合同，不是统计精度声明。
- public-boundary 测试只能验证“没有写、没有调用、没有 claim”；不得为了测试 guard 而新增 public adapter。
- 上述“没有调用”只指 CR173 新代码 integration calls；12/12 selected regressions 的既有 public 调用属于 read-only verification lane。
- NP-01..09 与 public 六项指标是两张正交清单；不得把 public 项补成“第十类”，也不得让一个 guard/probe 同时为两个 operation class 或 public metric 提供计数。
- stable computation ref 进入七字段；逐执行 attempt audit ref 外置。把 ordinal/run ref 写进 attempt basis 会破坏 3/3 determinism。
- `NaN`/`Inf` 只能 F03；F04 从 finite exact-rational matrix 开始。
- 本文的 planned file layout 不是实现授权；CP5 批准前 source/test/fixture 修改数必须为 0。

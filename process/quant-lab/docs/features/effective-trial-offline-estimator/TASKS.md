---
feature_id: "FEAT-173-01"
status: "cp5-approved-wave-execution"
version: "0.3"
change_id: "CR-173"
story_count: 3
wave_count: 3
task_count: 12
---

# FEAT-173-01 Effective-Trial Offline Estimator — TASKS

## 修订记录

| 版本 | 日期 | 修订人 | 变更 |
|---|---|---|---|
| 0.1 | 2026-07-16 | meta-se-critical | 建立 3 Story、3 serial Waves、12 个 CP5 后才可执行的原子任务。 |
| 0.2 | 2026-07-16 | meta-se-critical | 按 CP5 Round-1 findings 重映射 T01-T12：stable computation/外置 audit、F03/F04 parser-domain 分层、public new-code/read-only regression 六计数。 |
| 0.3 | 2026-07-16 | meta-se-critical | 按 CP5 Round-2 F-R2-002/003 重映射 T11/T12：非 public operation classes `9/9` 各0、public 双 lane 六项指标独立且禁止重复计数；刷新 S01/S02 LLD v1.2、S03 LLD v1.3/R3 状态。 |

> CP5 已于 2026-07-17 批准。S01/S02 LLD v1.3、S03 LLD v1.4 均为 `approved/confirmed=true`；pointer-only refresh 的规范合同变化数为 `0`。S01 的 T01-T04 已就绪，S02/S03 继续受串行依赖门控；public C1 projection/真实数据/runtime 任务数仍为 0。

## Wave 1 — CR173-S01 Contract / Evidence / Canonicalization

| 任务编号 | 动作 | 目标文件 | 完成准则 | 状态 |
|---|---|---|---|---|
| CR173-F01-T01 | 创建 | `engine/effective_trial_evidence.py` | 定义四类核心 contract + 外置 `ComputationAttemptAudit`；七字段仍7/7，audit不是第八字段；owner/persistence/retention边界显式 | ready |
| CR173-F01-T02 | 创建 | `engine/effective_trial_evidence.py` | 冻结 `EffectiveTrialAttemptBasisV1`、F01-F08 precedence 与 F03 token-parse/F04 finite-domain 唯一映射 | ready |
| CR173-F01-T03 | 创建 | `engine/effective_trial_evidence.py` | 实现 stable computation ref、canonical bytes/evidence hash、独立 audit-ref generator；audit不进入method/evidence hash；Decimal→float=0 | ready |
| CR173-F01-T04 | 创建 | `tests/research/test_effective_trial_evidence_contracts.py` | 七字段7/7、F01-F08 basis、3/3=1 computation/hash+3 audits、recovery parent/supersedes、raw fallback=0 | ready |

## Wave 2 — CR173-S02 Exact Spectral Estimator

| 任务编号 | 动作 | 目标文件 | 完成准则 | 状态 |
|---|---|---|---|---|
| CR173-F01-T05 | 创建 | `engine/effective_trial_estimator.py` | non-canonical token（NaN/Inf等）parser前F03；仅 finite exact rationals 进入 shape/symmetry/diag/range/PSD F04 | blocked-by-dependency |
| CR173-F01-T06 | 创建 | `engine/effective_trial_estimator.py` | deterministic lexicographic symmetric-pivot fraction-free exact `LDLᵀ`；tolerance/float=0 | blocked-by-dependency |
| CR173-F01-T07 | 创建 | `engine/effective_trial_estimator.py` | exact `n²/ΣRij²`、未舍入/舍入后双范围检查、一次 half-even、禁止 clamp | blocked-by-dependency |
| CR173-F01-T08 | 创建 | `tests/research/test_effective_trial_estimator.py` | analytic matrices、F03/F04不可重叠、PSD分支、rounding/hash deterministic unit tests PASS | blocked-by-dependency |

## Wave 3 — CR173-S03 Golden / Failure / Boundary Verification

| 任务编号 | 动作 | 目标文件 | 完成准则 | 状态 |
|---|---|---|---|---|
| CR173-F01-T09 | 创建 | `tests/fixtures/effective_trial/golden_vectors_v1.json` | 六类 vectors 6/6；GV05=`NaN`/non-canonical token固定F03，独立 finite-exact F04 oracle；真实/strategy字段=0 | blocked-by-dependency |
| CR173-F01-T10 | 创建 | `tests/research/test_effective_trial_cr173_qac.py` | 每类3/3=1 stable computation ref+1 evidence hash+3 audit refs；F01-F08 basis/recovery parent链；available=0 | blocked-by-dependency |
| CR173-F01-T11 | 创建 | `tests/research/test_effective_trial_authorization.py` | NP-01..09（credential、real data、lake/NAS、provider/network、catalog/store/pointer、strategy runtime、QMT/trading、publish/deploy、Git remote）inventory=`9/9`、各 counter=0、duplicate=0；public new-code edge/call/diff/write=0/0/0/0，且两组不复用 counter | blocked-by-dependency |
| CR173-F01-T12 | 创建 | `tests/research/test_effective_trial_cr173_qac.py` | standalone=1/1；CP7 read-only public inventory=12/12、existing expected edits=0；existing public calls 只属 read-only lane且不进入 NP-01..09；overclaim/CR172 auto-resume=0 | blocked-by-dependency |

## 依赖、所有权与执行顺序

| Story | depends_on | dependency type | primary owner | merge rule |
|---|---|---|---|---|
| S01 | none | none | evidence module + contract tests | 独占 |
| S02 | S01 | contract | estimator module + numeric tests | S01 contract frozen后开始 |
| S03 | S01,S02 | runtime | fixtures + QAC/auth tests | S01/S02 implementation evidence通过后开始 |

执行顺序固定 `S01 → S02 → S03`；3 nodes、3 edges、3 serial Waves。每个 Wave 只有 1 个 Story，同 Wave 文件冲突=0。

## 禁止任务

- 不创建 public C1 projection/adaptor/migration Story 或 task。
- 不修改 8 个 public C1 production paths，不修改 12 个既有 regression expected 以放宽语义。
- 不创建真实 correlation producer、strategy binding、runtime、data/provider/lake/NAS、credential、trading、publish/deploy 或 remote-write task。
- CP5 只授权按 `S01 → S02 → S03` 执行 7 个 repository-local 计划路径与本地 `uv run`；不得借执行扩大到 public C1、真实数据、runtime 或远端操作。

## 量化自检

| 检查 | 结果 |
|---|---|
| Feature/Story/Wave/Task | 1/3/3/12 |
| Story任务分配 | S01=4、S02=4、S03=4；合计12 |
| public projection/real/runtime task | 0/0/0 |
| 同 Wave primary file overlap | 0 |
| identity/audit task mapping | T01 object/owner、T02 basis/F01-F08、T03 refs/hash、T04/T10 repeat+recovery |
| operation/public task mapping | T11 non-public NP-01..09各0 + public new-code四项0；T12 read-only 12/12 + expected edits=0；跨组重复计数=0 |

## Gotchas

- T06 的 exact PSD 规则必须由 S02 full LLD 写成唯一算法；不得在实现时临时选 tolerance 或第三方浮点 eigen solver。
- T01-T04 的 stable computation ref 与外置 attempt audit 不能合并；audit ref 进入七字段/hash 的实现一律拒绝。
- T05/T09/T10 必须证明 `NaN`/`Inf` 只走 F03；F04 入口不得接收未解析 token。
- T11/T12 是“无触达”的守卫，不是 public integration 工作。
- T12 允许执行 12/12 existing regressions 的既有 public calls，但不得把它们记作 CR173 新代码 dependency/call，也不得修改 expected。
- T11 的 NP-01..09 与 public 四个 new-code counter 不共享 class/counter；public 指标不得被补成第十类。
- 任务状态只能依照 CP5 批准的串行 DAG 推进；上游未通过 CP7 时，下游保持 `blocked-by-dependency`。

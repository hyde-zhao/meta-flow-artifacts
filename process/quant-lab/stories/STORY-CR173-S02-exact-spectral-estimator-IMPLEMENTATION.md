---
status: "implemented"
version: "1.0"
story_id: "CR173-S02-exact-spectral-estimator"
story_slug: "exact-spectral-estimator"
feature_id: "effective.trial.offline.estimator"
implementation_type: "story"
source_story: "process/stories/STORY-CR173-S02-exact-spectral-estimator.md"
source_design_evidence: "process/stories/STORY-CR173-S02-exact-spectral-estimator-LLD.md"
created_by: "meta-dev"
created_at: "2026-07-17T11:12:37+08:00"
updated_at: "2026-07-17T11:12:37+08:00"
---

# Implementation: CR173-S02 Exact-rational estimator

## 1. 实现摘要

| 项目 | 内容 |
|---|---|
| 实现目标 | 在 S01 已验证合同上实现纯离线 exact-rational matrix domain、确定性 fraction-free `LDLᵀ`、participation ratio 与 standalone evidence 输出。 |
| 行为变化 | 新增 repository-local estimator；合法 fixture 可得到二阶相关结构有效维度，非法输入按 F03/F04/F08 fail-closed。 |
| 范围边界 | 不估计 matrix、不读取真实数据、不接 public C1、不执行 runtime/I/O、不声明 production SLA 或 FWER/DSR calibration。 |
| CP6 证据 | `process/checks/CP6-CR173-S02-IMPLEMENTATION-DONE.result.json` |

## 2. 上游设计引用

| 来源 | 路径 / ID | 本次消费内容 |
|---|---|---|
| Story | `process/stories/STORY-CR173-S02-exact-spectral-estimator.md` | 范围、AC、文件所有权、依赖门控 |
| Story 设计证据 | `process/stories/STORY-CR173-S02-exact-spectral-estimator-LLD.md` v1.3 | exact parser/domain、pivot、LDLT、公式、双 invariant |
| Feature 设计 | `docs/features/effective-trial-offline-estimator/TEST-PLAN.md` v0.3 | 数值矩阵、F03/F04 边界、public 隔离 |
| S01 合同 | `engine/effective_trial_evidence.py` | 已验证 bundle、method/hash、attempt basis、evidence builder |

## 3. 实现前置检查

| 检查项 | 结果 | 证据 / 说明 |
|---|---|---|
| 上游 Feature 设计存在 | PASS | Feature refs `3/3` 已在 Story/packet 固化 |
| Story 范围明确 | PASS | 两个 primary 文件，S01/public 只读 |
| 待确认问题已关闭 | PASS | LLD `open_items=0` |
| 影响范围可定位 | PASS | 新增 source/test `2/2` |
| 验证方式明确 | PASS | work packet verification plan `3/3` |
| 当前 Wave / dev_gate 满足 | PASS | S01 CP7 PASS；S02 `dev-ready` |
| 文件所有权无冲突 | PASS | primary owner=S02，forbidden touched=`0` |

## 4. 实现对象清单

| 对象 | 类型 | 目标 | 必须 | 验证方式 |
|---|---|---|---|---|
| `engine/effective_trial_estimator.py` | code / numeric-validator | exact domain、PSD、formula、evidence facade | yes | unit + compile + static boundary |
| `tests/research/test_effective_trial_estimator.py` | guardrail-test | 4+4 oracle、F03/F04、rounding、determinism | yes | pytest `27/27` |
| 本文及 CP6 return/evidence/result | docs-handoff | 可恢复 CP6 审计链 | yes | return/result check |

## 5. 设计契约映射

| 设计要求 | 来源 | 实现位置 | 动作 | 验证 |
|---|---|---|---|---|
| S01 precedence 先于 F04 | LLD §7/8.1 | `validate_raw_token_contract` / facade | 复用 S01 validator | 9 个 F03 + 5 个 F04 参数化用例 |
| finite exact-rational domain | LLD §8.1 | `validate_exact_matrix_domain` | shape/symmetry/diag/range exact check | F04/domain tests |
| 唯一 symmetric pivot | LLD §8.2 | `select_symmetric_pivot` | abs desc/signed desc/label asc | pivot repeat=`3/3` |
| fraction-free exact LDLT | LLD §8.3 | `fraction_free_ldlt_step` / `_prove_exact_psd` | integer scaled residual + exact division | PSD branch `4/4` |
| `n²/ΣRij²` + 双 invariant | LLD §8.5 | `estimate_participation_ratio_exact` / `quantize_and_validate_count` | exact Fraction；round once | analytic `4/4`；invariant `2/2` |
| standalone non-alias evidence | HLD claim ceiling | `estimate_effective_trial` | S01 basis/builder | computation/evidence hash `1/1` |
| 禁止 float/tolerance/clamp/random/public | LLD §8.6/8.7 | 新 source 静态边界 | 不引入相关依赖/调用 | static test PASS |

## 6. 单元测试 / Fixture 结果

| 测试对象 | 类型 | 输入 | 期望 | 风险 | 状态 |
|---|---|---|---|---|---|
| analytic formula | unit | identity/ρ=.5/all-ones/singleton | exact/token/rank `4/4` | 方法错误 | passed |
| PSD algorithm | unit | PD/singular/negative/zero-coupling | branch `4/4` | 不确定性/伪 PSD | passed |
| parser/domain split | contract | noncanonical `9` + finite invalid `5` | F03/F04 overlap=`0` | 双重映射 | passed |
| output boundary | unit | repeating/tampered renderer | round=`1`；range `2/2` | clamp/二次舍入 | passed |
| determinism | unit | 同输入 3 repeats | exact/ref/hash `1/1/1` | hash 漂移 | passed |
| authorization | static | estimator source | public/float/random paths=`0` | scope escape | passed |

## 7. 最小实现切片

| Slice ID | 对应设计契约 | 输出文件 | 局部验证 | 状态 |
|---|---|---|---|---|
| T05 | S01 parser + F03/F04 domain | estimator | parser/domain unit | done |
| T06 | unique pivot + fraction-free LDLT | estimator | four PSD branches | done |
| T07 | exact formula + once-only rounding | estimator | four analytic + invariant tests | done |
| T08 | determinism/static boundary | estimator tests | targeted pytest/static scan | done |

## 8. 变更说明

### 8.1 代码变更

| 文件 | 动作 | 说明 |
|---|---|---|
| `engine/effective_trial_estimator.py` | create | immutable result/proof types、exact validator、LDLT、formula、standalone facade |

### 8.2 Guardrail / 测试变更

| 文件 / 命令 | 动作 | 说明 |
|---|---|---|
| `tests/research/test_effective_trial_estimator.py` | create | 27 个 S02 tests；与 S01 合同合计 87 个回归 |

### 8.3 文档与过程证据

| 文件 | 动作 | 说明 |
|---|---|---|
| 本文、CP6 return/evidence/result/summary | create | 实现映射、验证、边界与下一路由 |

Prompt/Skill、模板/schema、安装器与平台适配均为 N/A；本 Story 是 repository-local pure Python。

## 9. 平台差异处理

| 平台 | 检查项 | 结果 |
|---|---|---|
| Claude/Codex/install | 无 agent、skill、installer 或 direct-ask 变更 | N/A |
| Python | Python 3.11；仅标准库 + S01 repository-local contract | PASS |

## 10. 验证结果

| 命令 / 检查 | 结果 | 证据 |
|---|---|---|
| S01+S02 targeted pytest | PASS | `87 passed in 0.17s` |
| S02 collect-only | PASS | S02 tests=`27` |
| `py_compile` | PASS | modules/tests `3/3` |
| `git diff --check` | PASS | whitespace errors=`0` |
| static authorization/numeric scan | PASS | source 中 public import/call、numpy/scipy、float/random/clamp=`0` |

## 11. 未覆盖项

| 未覆盖内容 | 原因 | 后续处理 |
|---|---|---|
| S03 six golden × 3、attempt audit recovery | 由下游 Story 所有 | S03 CP6/CP7 |
| public 12/12 read-only regression | 本 Story 禁止 public 接入 | S03 CP7，expected edits=`0` |
| 真实规模性能/SLA | CR173 fixture-only，无真实 activation 授权 | CR172 PATH-C/A 或真实规模目标出现前重评 |
| FWER/DSR/admission calibration | participation ratio 仅二阶有效维度 | future Spike/CR |

## 12. 风险与回滚

| Risk ID | 风险 | 缓解 | 回滚 / 重访条件 |
|---|---|---|---|
| R-CR173-METHOD-NONDETERMINISM | pivot/result/hash 漂移 | exact total order、3/3 repeat | 任一漂移即停用 S02，保持 unavailable |
| R-CR173-EXACT-INTEGER-GROWTH | integer 位宽随 `O(n³)` 消元增长 | 当前 fixture 最大 `n=4`，不声明 production SLA | CR172 PATH-C/A 或真实规模/性能目标出现前重新设计/压测 |
| R-CR173-SECOND-ORDER-MODEL-BIAS | 被误当多重检验校正数 | module/证据只声明二阶有效维度 | 需要 FWER/DSR 时转 Spike |

回滚只需移除新增 estimator/test；不得退化为 binary float、阈值 PSD、clamp 或 raw count alias。

## 13. 设计缺口反馈

实现阶段没有发现 S01 合同缺口或需要修改 HLD/Feature/LLD 的 design delta。`CURRENT.json` 一度滞后于 `STATE.current`，已由 Host 同步，不构成 Story 合同缺口。

## 14. QA / Review / Doc 后续交接

### QA 关注点

- 独立复验 exact division、negative pivot 与 zero-coupling 分支；确认 F03 输入不进入 F04。
- 复核 once-only half-even、前后范围 invariant 与 stable computation/evidence hash。
- 对 exact integer growth 只验证 fixture-only 边界与无 SLA claim，不擅加 cap/优化。

### Review 关注点

- 检查 source 无 public module dependency、real/runtime/I/O/credential/remote 行为。
- 检查 `Fraction`/integer residual 全链路无 binary float 或容差分支。

### Doc 关注点

- 对外措辞保持“二阶相关结构有效维度”，不得表述为 Li-Ji effective independent tests、FWER/DSR/admission calibration。

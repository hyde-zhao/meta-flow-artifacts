---
handoff_type: "stage-return"
change_id: "CR-173"
stage: "story-planning-cp4-preparation"
canonical_role: "meta-se-critical"
reasoning_profile: "critical"
status: "completed-cp4-pass-recommended"
created_at: "2026-07-16T15:20:41+08:00"
context_ref: "process/context/CP3-CR173.context.json"
source_handoff: "process/handoffs/CR173-CP4-META-SE-STORY-PLANNING-HANDOFF-2026-07-16.md"
feature_count: 1
story_count: 3
wave_count: 3
task_count: 12
cp4_recommendation: "PASS"
blocker_count: 0
cp5_gate_launched: false
lld_files_created: 0
source_test_fixture_files_modified: 0
public_c1_projection_story_count: 0
---

# CR173 CP4 Story Planning — meta-se Return Summary

## 1. 结论

CR173 已完成 CP3 批准后的 estimator-only Story planning 与 CP4 准备：新增 `1` 个 required Feature、`3` 个 `full-lld` Story、`3` 个串行 Wave、`12` 个 CP5 后任务。REQ `8/8`、SC `8/8`、Product Outcome `4/4` 均有 Story 覆盖；DAG 无环、无 dangling reference、无 isolated node，同 Wave 文件冲突为 `0`。

meta-se 对 Host Orchestrator 的正式建议为：`CP4 PASS`，blocker=`0`。本 return 不创建 CP4 result/context/ledger，不发起 CP5 人工门禁；这些 Host-owned 动作必须由主进程继续执行。

## 2. 范围与数量

| 项 | 结果 |
|---|---:|
| Feature | 1：`FEAT-173-01 Effective-Trial Offline Estimator` |
| Story | 3 |
| Wave | 3，全部串行 |
| Task | 12，S01/S02/S03 各 4 |
| `lld_policy=full-lld` | 3/3 |
| 实际 LLD 文件 | 0 |
| CP5 gate launch | 0 |
| source/test/fixture 修改 | 0/0/0 |
| public C1 projection Feature/Story/Task | 0/0/0 |
| public C1 production write/call | 0/0 |
| real data/strategy/provider/credential/runtime/trading/remote | 0/0/0/0/0/0/0 |

## 3. Story / Wave / 依赖

| Wave | Story | depends_on | 类型 | Primary owner | 结果 |
|---|---|---|---|---|---|
| CR173-W1-CONTRACT-EVIDENCE | S01 contract-evidence-canonicalization | none | none | evidence module + contract tests | ready for future LLD queue |
| CR173-W2-EXACT-ESTIMATOR | S02 exact-spectral-estimator | S01 | contract | estimator module + numeric tests | gated by S01 contract |
| CR173-W3-GOLDEN-BOUNDARY | S03 golden-failure-boundary-verification | S01,S02 | runtime,runtime | CR173 fixtures + QAC/auth tests | gated by S01/S02 evidence |

DAG 合计 `3 nodes / 3 typed edges`，topological order=`S01 → S02 → S03`。S03 只读 S01/S02 source 和 public C1 inventory；它没有 source 写权限。

## 4. 覆盖结果

| Story | REQ | SC | Outcome |
|---|---|---|---|
| S01 | 001,002,003,004,005,007 | Q01,F01,N01,B01,C01 | O01,O02,O04 |
| S02 | 001,002,003,005,006 | P01,Q01,F01,B01,D01 | O01,O03 |
| S03 | 003,004,005,006,007,008 | F01,N01,B01,D01,C01,A01 | O02,O03,O04 |

并集结果：

- `REQ-CR173-001..008 = 8/8 = 100%`，uncovered=`0`。
- `SC-CR173-P01/Q01/F01/N01/B01/D01/C01/A01 = 8/8 = 100%`，uncovered=`0`。
- `CR173-O01..O04 = 4/4 = 100%`，uncovered=`0`。
- SC-C01 由“standalone evidence 到 public boundary 即停止”的 negative/static guard 覆盖；它不构成 public projection Story。

## 5. 产物清单

| 类别 | 路径 | 状态 |
|---|---|---|
| Feature matrix | `docs/design/FEATURE-DESIGN-MATRIX.md` v1.27 CR173 increment | updated |
| Feature DESIGN | `docs/features/effective-trial-offline-estimator/DESIGN.md` | created |
| Feature TEST-PLAN | `docs/features/effective-trial-offline-estimator/TEST-PLAN.md` | created |
| Feature TASKS | `docs/features/effective-trial-offline-estimator/TASKS.md` | created |
| Development truth source | `process/DEVELOPMENT-PLAN.yaml` v1.13 `cr173_story_planning` | updated |
| Story S01 | `process/stories/STORY-CR173-S01-contract-evidence-canonicalization.md` | created |
| Story S02 | `process/stories/STORY-CR173-S02-exact-spectral-estimator.md` | created |
| Story S03 | `process/stories/STORY-CR173-S03-golden-failure-boundary-verification.md` | created |
| Stage return | 本文件 | created |

所有 Story 均包含非空 `feature_design_refs`、`lld_policy`、typed dependencies、primary/read-only/forbidden ownership、量化验收、实现/验证上下文和失败路由。

## 6. CP5 设计证据队列（只登记，不创建）

| 顺序 | Future evidence path | 必须冻结 |
|---|---|---|
| 1 | `process/stories/STORY-CR173-S01-contract-evidence-canonicalization-LLD.md` | 七字段/null/ref、8 failure reason、canonical bytes/hash、append-only recovery |
| 2 | `process/stories/STORY-CR173-S02-exact-spectral-estimator-LLD.md` | deterministic pivot/LDLT 伪代码、exact rational、双 invariant、一次 half-even |
| 3 | `process/stories/STORY-CR173-S03-golden-failure-boundary-verification-LLD.md` | 6×3 oracle、8 failures、8+12 inventory、zero-operation/public-diff guard |

当前三条 evidence path 均为 `not-started`，实际文件不存在。这符合“不要创建 LLD、不要发起 CP5”的 handoff 边界。

## 7. 自检结果

| 检查 | 结果 | 证据摘要 |
|---|---|---|
| YAML parse / CR173 counts | PASS | 1 Feature、3 cards、3 Waves、12 tasks、8 REQ、8 SC、3 nodes/3 edges、projection stories=0 |
| Story frontmatter / mandatory sections | PASS | 3/3 `feature_refs`、3/3 `feature_reference_ids`、3/3 design refs、scalar `lld_policy=full-lld` + detail、3/3 acceptance/failure route |
| DAG | PASS | acyclic=true、invalid refs=0、isolated=0、merge order S01→S02→S03 |
| File ownership | PASS | primary paths=7、unique=7、same-wave conflicts=0、public primary paths=0 |
| LLD absence | PASS | `STORY-CR173-*-LLD.md` matches=0 |
| Whitespace/diff | PASS | targeted `git diff --check`无错误；新增规划文档无 trailing whitespace |
| Canonical global plan-check | inherited FAIL, unrelated | 仍仅有 CR013 四个 legacy `draft-pending-cp4` invalid status 与历史 ref warnings；CR173 scoped checks全部 PASS |
| Canonical global feature trace/ownership | inherited FAIL, unrelated | 仓库缺少 `docs/design/FEATURE-REGISTRY.yaml` 且历史 Story 有 schema/ref debt；CR173 三卡字段已被 checker 识别，scoped schema PASS，未知 feature 仅由全局 registry 缺失导致 |

全局历史债务未被修改、未被 CR173 吸收，也不构成 CR173 blocker。`process/DEVELOPMENT-PLAN.yaml` 的 `global_plan_check` 已显式记录这一 inherited-unrelated 事实。

## 8. Public projection 与授权证明

CR173 的 planned primary files 只有两个新 engine estimator-only 文件、四个新 CR173 tests/fixture path；以下 public paths 只登记为 read-only/forbidden，planned primary count=`0`：lineage、lineage store、statistical gate、statistical evidence、multiple-testing、overfit/DSR、cross-strategy gates、admission package。

设计与验收共同要求：

- standalone seven-field evidence=`1/1`，public projection/write/call=`0/0/0`。
- `public_effective_trial_count_populatable=false`、`c1_computable=false`。
- raw alias、Li–Ji/BH/FWER/Šidák/DSR/admission calibration overclaim=`0`。
- CR172 auto-resume/close=`0/0`；CR173 只关闭 methodology prerequisite。
- public versioned projection 必须等待独立 future CR、owner approval、migration 与 8+12 全量回归。

## 9. Blocker、风险与下一步

Blocker=`0`。两项剩余风险均是已接受的后续边界，不阻断 estimator-only CP4：

1. 二阶 participation ratio 不覆盖 empirical sampling error、tail/FWER/DSR calibration；触发时转 methodology Spike/新 method version。
2. public C1 是 cross-owner/cross-domain/non-compatible migration；必须由 future projection CR 处理。

建议 Host 下一步：生成并执行正式 CP4 自动结果/context/ledger；若正式结果 PASS，则按 post-approval 路由准备三个 full-LLD 设计证据队列并在全部证据齐备后发起 CP5。当前 return 本身没有启动这些动作。

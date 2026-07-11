---
story_id: "CR163-S04-existing-admission-projection"
change_id: "CR-163"
stage: "CP6"
status: "implemented"
rework_round: 2
created_by: "meta-dev-debugger"
created_at: "2026-07-11T13:51:29+08:00"
context_ref: "process/context/stories/STORY-CR163-S04.CP6.work-packet.json"
design_ref: "process/stories/STORY-CR163-S04-existing-admission-projection-LLD.md"
updated_at: "2026-07-11T14:31:44+08:00"
---

# CR163-S04 Existing Admission Projection Implementation

## 修订记录

| 轮次 | 日期 | 修订人 | 变更 |
|---|---|---|---|
| CP6 initial | 2026-07-11 | meta-dev-debugger | 三 existing consumer projection 与初始 fixture coverage。 |
| Rework 1 | 2026-07-11 | meta-dev-debugger | 修复 QA-CR163-S04-001..003：可信 validation binding、canonical payload/diagnostic 分离、真实 CR155 admission 回归。 |
| Rework 2 | 2026-07-11 | meta-dev-debugger | 修复 QA-CR163-S04-004：迁移 CR151/CR154 stale native fixtures 到可信 lineage 前置，并保留 missing-lineage BLOCKED。 |

## 实现前置检查

| 条目 | 结论 | 证据 |
|---|---|---|
| CP5 / Story 门控 | PASS | Story `status=dev-ready`、`design_evidence_confirmed=true`、LLD `confirmed=true` |
| S01 / S02 依赖 | PASS | `process/evidence/STORY-CR163-S01.CP7.index.json`、`STORY-CR163-S02.CP7.index.json` |
| 文件冲突与写入范围 | PASS | 仅修改 packet `write_scope` 的三个 existing consumers、一个新测试和 CP6 证据文件 |
| 授权边界 | PASS | 仅 repository-local source/test 与 synthetic fixtures；未执行真实 data、credential、external、statistical、backfill、release 或 runtime 操作 |

## 实现对象与设计契约映射

| 对象 | 实现 | 契约与验证 |
|---|---|---|
| 单一 consumer projection adapter | `ValidationBoundFamilyEvidence` + `consume_family_lineage_projection()` | 正向 truth 只接受 manifest + validation result 绑定并调用 S01 projector；serialized Mapping 与裸 present DTO 均 fail closed；不读 store、不 replay、不 re-hash、不重算 raw count |
| CR151 consumer | `evaluate_strategy_admission_statistical_gate(..., family_lineage_projection=...)` | canonical projection 与 `family_lineage_reconciliation` 分离；`trial_count` 只作 `absent/match/mismatch` 手工对账，mismatch 阻断 gate 但不改写 canonical payload |
| CR154 Gate1 consumer | `evaluate_gate1_statistical_reliability(..., family_lineage_projection=...)` | 仅投影 validated raw count/ref/hash；effective count 仍为 typed unavailable，ref/method 空，C1 non-computable，Gate1 继续 BLOCKED |
| Existing admission package | `attach_family_lineage_to_admission_package()` | 附加 lineage summary/ref/limitations，复用 `_worse_admission_status`；不新增 gate，不改变 runtime authorization flags |
| Synthetic fixture tests | `tests/test_cr163_trial_lineage_admission_projection.py` | 3/3 full-payload equality、forged Mapping/bare DTO、五类 blocked projection、typed unavailable、manual diagnostic、status precedence、真实 CR155 compose/attach/no-new-gate 边界 |

## 最小实现切片与验证

| Slice | 内容 | 结果 |
|---|---|---|
| S04-01 | 冻结共享 projection adapter 与 effective/C1 claim ceiling | PASS |
| S04-02 | 接入 CR151、CR154 与 package 三个 existing consumers | PASS |
| S04-03 | 增加 13 个 fixture/parameterized cases，并回归 S01 contract suite | PASS，178 passed |
| S04-R1-01 | 拒绝 untrusted serialized/bare-positive input；只允许 validation-bound result path | PASS |
| S04-R1-02 | canonical projection 与 CR151 manual diagnostic 分离 | PASS |
| S04-R1-03 | 真实 CR155 compose PASS/candidate=true → missing-lineage attach BLOCKED/candidate=false | PASS，组合 195 passed |
| S04-R2-01 | CR151 PASS/FAIL/NEEDS_REVIEW threshold fixtures 注入可信 validation-bound lineage；新增 no-lineage BLOCKED | PASS |
| S04-R2-02 | CR154 artifact-complete fixture 注入可信 raw lineage；effective/C1 ceiling 与 missing-lineage 仍 BLOCKED | PASS |
| S04-R2-03 | 运行 migrated suites 与全 affected subset | PASS，16 passed / 220 passed |

## 验证结果

| 命令 | 结果 | 摘要 |
|---|---|---|
| `uv run --python 3.11 pytest -q tests/test_cr163_trial_lineage_admission_projection.py tests/test_experiment_family_lineage_contracts.py` | PASS | `178 passed in 0.13s` |
| `uv run --python 3.11 python -m py_compile engine/strategy_admission_statistical_gate.py engine/cross_strategy_reliability_gates.py engine/strategy_admission_package.py` | PASS | 三个 consumer module 均可编译 |
| `git diff --check` | PASS | 无 whitespace error |
| `uv run --python 3.11 pytest -q tests/test_cr163_trial_lineage_admission_projection.py tests/test_experiment_family_lineage_contracts.py tests/research/test_daily_multifactor_baseline_artifact.py` | PASS | Rework 1：`195 passed in 0.15s` |
| `uv run --python 3.11 pytest -q tests/research/test_strategy_admission_statistical_gate.py tests/research/test_cross_strategy_reliability_gates.py` | PASS | Rework 2 migrated suites：`16 passed in 0.07s` |
| `uv run --python 3.11 pytest -q tests/research/test_strategy_admission_statistical_gate.py tests/research/test_cross_strategy_reliability_gates.py tests/research/test_strategy_admission_package.py tests/test_cr163_trial_lineage_admission_projection.py tests/test_experiment_family_lineage_contracts.py tests/research/test_daily_multifactor_baseline_artifact.py` | PASS | Full affected subset：`220 passed in 0.18s` |

## Rework 1 findings closure

| Finding | 结论 | 证据 |
|---|---|---|
| QA-CR163-S04-001 | RESOLVED | arbitrary ref/hash/raw、effective injection、unknown、missing binding 与 bare present DTO 全部在 3 consumers 中 blocked；positive 只来自 `ValidationBoundFamilyEvidence` 调用 S01 projector |
| QA-CR163-S04-002 | RESOLVED | 三 consumer canonical projection exact full-payload equality；manual reconciliation 仅在 CR151 `family_lineage_reconciliation` diagnostics |
| QA-CR163-S04-003 | RESOLVED | 测试实际调用 CR155 `compose_admission_package()` 先得到 PASS/`paper_candidate=true`，missing lineage attachment 后强制 `package_status=BLOCKED`、`paper_candidate=false`、candidate derivation false、backfill 0 |
| QA-CR163-S04-004 | RESOLVED | 两个 stale native suites 迁移完成：CR151 threshold outcomes 在可信 lineage 下保持原意；CR154 artifact policy 在可信 raw lineage 下仍遵守 effective/C1 BLOCKED ceiling；missing lineage 明确 BLOCKED；affected subset 220/220 PASS |

## 边界与剩余风险

- 未修改 `engine/experiment_family_lineage.py`、lineage store、S03 producer、CR155 module、`process/state/**` 或 `data/**`；工作树中这些路径已有的并行改动不属于 S04。
- 未新增 gate ID 或 gate family；CR154 的 `GATE_IDS` 仍为既有 6 项。
- CR155 无 native lineage projection 时保持 package blocked、`paper_candidate=false`、historical backfill count 0；本 Story 不执行 backfill。
- effective trial 方法仍未获授权，故 CR154 Gate1 的 C1/deflated-performance claim 按设计保持不可计算与 BLOCKED。这是预期 claim ceiling，不是待修缺陷。
- 无长期 Feature/HLD/ADR 设计变更，design delta 不需要。

## QA 交接重点

- 复核相同 projection 的 target ref/hash/raw count 在 3/3 consumer 输出完全一致。
- 复核 absent、unknown、tamper/invalid 与 manual mismatch 均不能产生 present truth。
- 复核 package status 只恶化、不改善，且四个 `not_*_authorization` flags 不变。
- 复核 S01/S02/S03 文件未被 S04 修改，真实操作与 forbidden counters 均为 0。

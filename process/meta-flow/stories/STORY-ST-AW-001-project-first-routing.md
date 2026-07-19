---
story_id: "ST-AW-001"
title: "唯一解析 project-first artifact 路由并保留显式 legacy 兼容"
story_slug: "project-first-routing"
status: "verified-with-risk"
priority: "P0"
wave: "W1"
depends_on: []
dependency_contracts: []
feature_design_refs:
  - "process/docs/features/cr051-routing/DESIGN.md"
  - "process/docs/features/cr051-routing/TEST-PLAN.md"
  - "process/docs/features/cr051-routing/TASKS.md"
lld_policy:
  required_level: "full-lld"
  trigger_reasons: ["portable-schema", "legacy-write-conflict", "cross-module-contract"]
  rationale: "ProjectArtifactConfig/RouteDecision 是四个下游 Feature 的安全输入，且双布局歧义会直接影响写入归属。"
  waiver_reason: ""
  revisit_condition: "Resolver 无法在保持旧 process route 兼容的同时提供唯一 write target，或需改变已批准 layout 边界。"
  evidence_path: "process/stories/STORY-ST-AW-001-project-first-routing-LLD.md"
file_ownership:
  primary: ["meta_flow/workspace/project_artifact_routing.py", "tests/test_cr051_project_artifact_routing.py"]
  shared: ["meta_flow/workspace/routing.py", "tests/test_workspace_routing.py"]
  merge_owner: "ST-AW-001"
  forbidden: ["meta_flow/workflow/artifact_leg_lifecycle.py", "meta_flow/workflow/artifact_aggregate.py", "sibling project content", "Git/worktree mutation"]
lld_gate:
  required_inputs: ["HLD", "ADR", "FEATURE-DESIGN-MATRIX", "Feature DESIGN", "Story"]
  design_evidence_type: "full-lld"
  design_evidence_path: "process/stories/STORY-ST-AW-001-project-first-routing-LLD.md"
  status: "approved"
  confirmed: true
  approval_ref: "process/checkpoints/CP5-CR051-ALL-STORIES-LLD-BATCH.md"
  cp5_precheck_result: "process/checks/CP5-CR051-ST-AW-001-LLD-IMPLEMENTABILITY-R2.result.json"
  supersedes_precheck_result: "process/checks/CP5-CR051-ST-AW-001-LLD-IMPLEMENTABILITY.result.json"
  revision_finding_refs: ["CP5-QA-R1-F01"]
implementation_gate:
  evidence_required: true
  evidence_path: "process/stories/STORY-ST-AW-001-project-first-routing-IMPLEMENTATION.md"
  evidence_type: "implementation-md"
  implementation_objects: ["code", "template-schema", "guardrail-test", "docs-handoff"]
  test_plan_refs: ["process/docs/features/cr051-routing/TEST-PLAN.md"]
  local_validation_results:
    - "36 targeted tests passed"
    - "ruff PASS"
    - "py_compile PASS"
    - "git diff --check PASS"
    - "return-check PASS"
    - "evidence-check PASS"
  cp6_result: "process/checks/CP6-CR051-ST-AW-001-project-first-routing-CODING-DONE-R2.result.json"
  return_packet_ref: "process/returns/ST-AW-001.CP6-R2.return.json"
  evidence_index_ref: "process/evidence/ST-AW-001.CP6-R2.index.json"
  status: "PASS-R2"
verification_gate:
  validation_mode: "mixed"
  validation_target:
    sut_type: "meta-flow-core-code"
    native_test_required: true
    workflow_eval_required: false
    prompt_bundle_required: false
    eval_suite_refs: []
    prompt_bundle_refs: []
    runtime_authorization_required: []
  cp7_result: "process/checks/CP7-CR051-ST-AW-001-project-first-routing-VERIFICATION-DONE-R2.result.json"
  remaining_risks:
    - "仓库缺少 canonical MODULE-BOUNDARIES.yaml，模块边界检查当前不可适用"
    - "尚未执行真实跨设备 portability pilot"
  findings: ["CP7-AW-001-F01", "CP7-AW-001-F02"]
  status: "PASS_WITH_RISK"
dev_gate:
  design_evidence_confirmed: true
  lld_confirmed: true
  dependencies_satisfied: true
  file_conflict_free: true
task_count: 6
created_at: "2026-07-18T06:35:00Z"
updated_at: "2026-07-18T12:36:00Z"
---

## 目标

以显式 project identity、layout version 和 anchor-relative metadata 唯一解析当前项目的 artifact docs/process；legacy 与 project-first 共存时保留确定读取但不猜测写目标，所有歧义在任何下游 mutation 前 fail closed。

## 开发上下文（dev_context）

- **输入文件**：CP3 HLD/ADR/Blueprint/Domain/Dependency Map、CR051 Feature Matrix、`cr051-routing` 三件套、REQ-AW-001..003/013、TC-AW-001..003/010/012。
- **输出文件**：`meta_flow/workspace/project_artifact_routing.py`、`tests/test_cr051_project_artifact_routing.py`；仅在兼容需要时修改两个 shared 文件。
- **设计约束**：canonical metadata 只保存 anchor + relative path；write target 基数只能是 0 或 1；resolver 不调用 Git/lifecycle、不读取 sibling 内容、不移动文件或 link。
- **Feature 设计引用**：本卡 frontmatter 的 3 个 `feature_design_refs` 全部是 CP5 必读输入。
- **LLD 策略**：`full-lld`；必须冻结 schema、函数签名、metadata discovery、错误投影和旧 API adapter。
- **平台目标**：Python typed domain + 纯解析 fixture；跨设备语义由 relocation fixture 证明，无 SaaS/凭据/真实 remote。

### 依赖与并行门控

| 上游 Story | 类型 | LLD 门控 | 开发门控 | 说明 |
|---|---|---|---|---|
| N/A | N/A | CP3 + required Feature pack 可读 | CP5 confirmed | DAG root；下游只消费本 Story 冻结的 value objects/API。 |

### 文件所有权

| 类型 | 文件 | Owner / 合并规则 |
|---|---|---|
| primary | `meta_flow/workspace/project_artifact_routing.py`、`tests/test_cr051_project_artifact_routing.py` | ST-AW-001 独占写入 |
| shared | `meta_flow/workspace/routing.py`、`tests/test_workspace_routing.py` | ST-AW-001 为 merge owner；仅单向兼容 adapter/回归 |
| forbidden | leg/aggregate 主模块、sibling 内容、Git/worktree mutation | 发现依赖即停止并返回设计澄清 |

### 文件系统布局

```text
meta_flow/workspace/
├── project_artifact_routing.py       # 新建；schema、resolver、owned target proof
└── routing.py                        # 条件修改；旧 process route 单向 adapter
tests/
├── test_cr051_project_artifact_routing.py
└── test_workspace_routing.py         # 条件修改；兼容回归
```

### AI 可执行任务清单

| TASK-ID | 动作 | 目标 | 描述 |
|---|---|---|---|
| TASK-AW-R01 | 创建 | routing 主模块 | 冻结 PathRef、ProjectArtifactConfig、RouteDecision 和错误码 |
| TASK-AW-R02 | 修改 | routing 主模块 | 实现 anchor-relative/layout/read-order/唯一 write target |
| TASK-AW-R03 | 修改 | routing 主模块 | 实现 project/namespace/owned-target proof |
| TASK-AW-R04 | 修改 | `routing.py` | 增加单向 compatibility adapter |
| TASK-AW-R05 | 创建/修改 | 定向测试 | 覆盖 project-first/legacy/ambiguity/traversal/relocation |
| TASK-AW-R06 | 验证 | 批准文件范围 | Ruff、定向回归、touched-path 与禁止依赖审计 |

## 技术说明

正式设计证据为 `process/stories/STORY-ST-AW-001-project-first-routing-LLD.md`（`lld_version=1.1`）；R2 单 Story 自动预检为 `process/checks/CP5-CR051-ST-AW-001-LLD-IMPLEMENTABILITY-R2.result.json`，supersede R1 result。全量 CP5 已于 2026-07-18 批准，初次 CP6 通过；CP7 独立验证发现 `CP7-AW-001-F01/F02`，当前进入同 Story 回修。回修不授权真实 Git/worktree/ref/remote/link/migration mutation。

## 实现执行上下文（implementation_context）

| 项目 | 内容 |
|---|---|
| 实现证据类型 | implementation-md |
| 实现证据路径 | `process/stories/STORY-ST-AW-001-project-first-routing-IMPLEMENTATION.md` |
| 实现对象清单 | code / template-schema / guardrail-test / docs-handoff |
| 设计契约映射状态 | complete；见 IMPLEMENTATION.md |
| 单元测试 / Fixture 计划 | `process/docs/features/cr051-routing/TEST-PLAN.md` |
| 最小实现切片 | schema → resolver → adapter → fixtures |
| 局部验证结果 | CP6 PASS：36 tests、ruff、py_compile、diff、return/evidence |
| 平台差异检查 | N/A；纯 Python/local filesystem fixture |
| 未覆盖项 | canonical MODULE-BOUNDARIES.yaml 缺失；真实跨设备 pilot 留 CP7/CP8 风险判定 |
| QA / Review / Doc 关注点 | 唯一 write target、portable path、旧 route 无回归、零 sibling 读取 |

## 验证上下文（validation_context）

| 项目 | 内容 |
|---|---|
| validation_mode | mixed |
| validation_target.sut_type | meta-flow-core-code |
| 验证重点 | schema/path unit、TC-AW-001..003/010/012、relocation、write/Git spy=0 |
| 真实运行授权 | 不需要；真实 artifact 写入与迁移未授权 |
| CP7 结论 | NEEDS_REWORK；跨项目 identity mismatch 未 fail closed，legacy malformed owned_paths 泄漏裸 TypeError |

## 量化验收标准（acceptance_criteria）

- [ ] project-first fixture 恰好返回当前项目 `docs`、`process` 两类目标，sibling target 数为 0。
- [ ] legacy 与 project-first 写目标歧义的阻断率为 100%，歧义路径写入次数为 0。
- [ ] 相同 metadata 重复解析的 logical decision/digest 一致率为 100%，workspace relocation 后设备绝对前缀违规数为 0。
- [ ] absolute/traversal/project mismatch/未知 layout 负例拒绝率为 100%，Git/file/link mutation 次数为 0。
- [ ] TC-AW-001/002/003/010/012 的适用断言覆盖率为 100%，REQ-AW-001..003/013/NF001..002 均有测试引用。

## 阻塞说明

无当前 blocker；若兼容 adapter 需要 lifecycle→resolver 反向依赖或按路径存在性猜写目标，路由 `NEEDS_DESIGN_CLARIFICATION`。

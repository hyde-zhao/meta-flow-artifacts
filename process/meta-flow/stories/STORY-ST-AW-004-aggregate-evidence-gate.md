---
story_id: "ST-AW-004"
title: "以单写 evidence gate 聚合全部 required legs"
story_slug: "aggregate-evidence-gate"
status: "verified-with-risk"
priority: "P0"
wave: "W3"
depends_on: ["ST-AW-003"]
dependency_contracts:
  - {story_id: "ST-AW-003", type: "contract", lld_gate: "PublishedLegResultHandle declared", dev_gate: "payload/receipt/handle frozen at CP5 R2"}
feature_design_refs:
  - "process/docs/features/cr051-aggregate/DESIGN.md"
  - "process/docs/features/cr051-aggregate/TEST-PLAN.md"
  - "process/docs/features/cr051-aggregate/TASKS.md"
lld_policy:
  required_level: "full-lld"
  trigger_reasons: ["single-writer", "correlation", "evidence-dag", "workflow-projection"]
  rationale: "AggregateResult 是 overall 单写边界并控制 CR/state/current 投影，必须冻结 correlation、evidence 和 failure route。"
  waiver_reason: ""
  revisit_condition: "2/2 matching PASS 无法与 Git 执行隔离，或既有 controlled writer 无法表达 non-PASS hold。"
  evidence_path: "process/stories/STORY-ST-AW-004-aggregate-evidence-gate-LLD.md"
file_ownership:
  primary: ["meta_flow/workflow/artifact_aggregate.py", "tests/test_artifact_aggregate.py"]
  shared: ["meta_flow/cli.py", "meta_flow/workflow/cr_lifecycle.py", "meta_flow/state/current.py"]
  merge_owner: "ST-AW-004"
  forbidden: ["meta_flow/workflow/artifact_leg_lifecycle.py", "Git/worktree executor calls", "artifact main/manual sync", "automatic close/rollback for non-PASS"]
lld_gate:
  required_inputs: ["HLD", "ADR", "FEATURE-DESIGN-MATRIX", "Feature DESIGN", "Story"]
  design_evidence_type: "full-lld"
  design_evidence_path: "process/stories/STORY-ST-AW-004-aggregate-evidence-gate-LLD.md"
  status: "approved"
  confirmed: true
  approval_ref: "process/checkpoints/CP5-CR051-ALL-STORIES-LLD-BATCH.md"
  cp5_precheck_result: "process/checks/CP5-CR051-ST-AW-004-LLD-IMPLEMENTABILITY-R2.result.json"
implementation_gate:
  evidence_required: true
  evidence_path: "process/stories/STORY-ST-AW-004-aggregate-evidence-gate-IMPLEMENTATION.md"
  evidence_type: "implementation-md"
  implementation_objects: ["code", "template-schema", "guardrail-test", "docs-handoff"]
  test_plan_refs: ["process/docs/features/cr051-aggregate/TEST-PLAN.md"]
  local_validation_results: ["179 tests + 9 subtests passed after canonical nested-digest repair", "ruff/py_compile/diff/return/evidence/forbidden-import checks passed"]
  result_ref: "process/checks/CP6-CR051-ST-AW-004-aggregate-evidence-gate-CODING-DONE.result.json"
  return_ref: "process/returns/ST-AW-004.CP6.return.json"
  evidence_ref: "process/evidence/ST-AW-004.CP6.index.json"
  status: "PASS"
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
  cp7_result: "process/checks/CP7-CR051-ST-AW-004-aggregate-evidence-gate-VERIFICATION-DONE.result.json"
  return_ref: "process/returns/ST-AW-004.CP7.return.json"
  evidence_ref: "process/evidence/ST-AW-004.CP7.index.json"
  remaining_risks: ["selector-lock-crash", "windows-cross-process-lock", "real-remote-unverified"]
  status: "PASS_WITH_RISK"
dev_gate:
  design_evidence_confirmed: true
  lld_confirmed: true
  dependencies_satisfied: true
  file_conflict_free: true
task_count: 5
created_at: "2026-07-18T06:35:00Z"
updated_at: "2026-07-18T14:35:30Z"
---

## 目标

只读两个matching `PublishedLegResultHandle`，从result_ref重读payload并校验external receipt/ref/digest/single-write key/correlation后，以纯函数和固定优先级`BLOCKED > FAIL > IN_PROGRESS > PASS`单写immutable AggregateResult；仅2/2 validated payload terminal PASS才允许controlled projection。

## 开发上下文（dev_context）

- **输入文件**：ST-AW-003 LegResultPayload/WriteReceipt/PublishedHandle contract、CP3设计基线、Matrix、`cr051-aggregate`三件套、REQ-AW-011..013/016..017、TC-AW-008/009/012/014。
- **输出文件**：`artifact_aggregate.py`、定向测试，以及由本 Story 单写的 CR-051 CLI/controlled projection 接线。
- **设计约束**：aggregate只消费reread-validated published handles，不接受raw/unpublished；payload/aggregate均不含自己的receipt/ref；aggregate不import Git/worktree；PARTIAL不属于overall；persist/readback后才可projection；旧结果不覆盖。
- **Feature 设计引用**：frontmatter 的 3 个 aggregate pack 文件。
- **LLD 策略**：`full-lld`；需冻结 validator、pure function、single-writer key、projection boundary 和重试。
- **平台目标**：纯函数/unit + local result fixture；不需要真实 Git/remote。

### 依赖与并行门控

| 上游 Story | 类型 | LLD 门控 | 开发门控 | 说明 |
|---|---|---|---|---|
| ST-AW-003 | contract | Payload/Receipt/PublishedHandle已声明 | CP5 R2冻结；primary文件不重叠 | aggregate只经reader/validator消费published handle，不修改leg primary。 |

### 文件所有权

| 类型 | 文件 | Owner / 合并规则 |
|---|---|---|
| primary | `artifact_aggregate.py`、`test_artifact_aggregate.py` | ST-AW-004 独占写 |
| shared | `meta_flow/cli.py`、`cr_lifecycle.py`、`state/current.py` | ST-AW-004 为 CR-051 唯一 merge owner；只走 controlled writer |
| contract read-only | `artifact_leg_lifecycle.py` | ST-AW-003 owner；本 Story 不修改 |
| forbidden | Git/worktree executor、manual sync、artifact main、non-PASS 自动关闭/回滚 | 违反即停止 |

### 文件系统布局

```text
meta_flow/workflow/
├── artifact_aggregate.py
└── cr_lifecycle.py                  # shared/guarded projection
meta_flow/state/current.py           # shared/controlled writer only
meta_flow/cli.py                     # CR-051 唯一 merge owner=ST-AW-004
tests/test_artifact_aggregate.py
```

### AI 可执行任务清单

| TASK-ID | 动作 | 目标 | 描述 |
|---|---|---|---|
| TASK-AW-004-01 | 设计 | full LLD | 冻结 AggregateRequest/ValidatedLegSet/Result、precedence 与 single-writer key |
| TASK-AW-004-02 | 创建 | aggregate 模块 | 实现 result validator 与纯聚合函数 |
| TASK-AW-004-03 | 创建 | aggregate 模块 | 实现 persistence/readback/idempotency/conflict detection |
| TASK-AW-004-04 | 修改 | aggregate + shared 接线 | 实现 projection guard、next route 与 CLI |
| TASK-AW-004-05 | 创建 | 定向测试 | 覆盖 16 组合、invalid correlation、concurrency、projection、回归 |

## 技术说明

正式证据为 `process/stories/STORY-ST-AW-004-aggregate-evidence-gate-LLD.md`；R2 自动预检为 `process/checks/CP5-CR051-ST-AW-004-LLD-IMPLEMENTABILITY-R2.result.json` 并 supersede R1。全量 CP5 已批准，payload→receipt→validated handle→aggregate→aggregate receipt→projection 契约已冻结；本 Story 在 W3 调度前保持 `lld-approved`，不得绕过前序 Wave 或修改 Git/worktree executor。

## 实现执行上下文（implementation_context）

| 项目 | 内容 |
|---|---|
| 实现证据类型 | implementation-md |
| 实现证据路径 | `process/stories/STORY-ST-AW-004-aggregate-evidence-gate-IMPLEMENTATION.md` |
| 实现对象清单 | code / template-schema / guardrail-test / docs-handoff |
| 设计契约映射状态 | pending |
| 单元测试 / Fixture 计划 | `process/docs/features/cr051-aggregate/TEST-PLAN.md` |
| 最小实现切片 | schema/validator → pure aggregate → persistence → projection/CLI → tests |
| 局部验证结果 | pending |
| 平台差异检查 | N/A；纯 result/projection contract |
| 未覆盖项 | CP5 前不实现 |
| QA / Review / Doc 关注点 | 16 组合、invalid set、single writer、PARTIAL 非 overall、Git import=0 |

## 验证上下文（validation_context）

| 项目 | 内容 |
|---|---|
| validation_mode | mixed |
| validation_target.sut_type | meta-flow-core-code |
| 验证重点 | pure function、correlation、single-writer、input digest、projection hard gate、legacy writer regression |
| CP7 结论 | not-started |

## 量化验收标准（acceptance_criteria）

- [ ] source/artifact 的 4×4 共 16 个状态组合 overall 与固定优先级一致率为 100%；仅 PASS/PASS 组合得到 overall PASS。
- [ ] raw payload/ref、unpublished outcome、stale/duplicate/wrong correlation/required set，以及receipt/ref/payload digest/single-write key错配的拒绝率为100%，上述路径pure aggregate/projection次数为0。
- [ ] artifact 单腿 PASS、缺失 result、PARTIAL effect 或任一 non-PASS 导致 CR close/state-current completion 的次数为 0。
- [ ] 相同 validated input 重算 aggregate ID/payload digest 一致率为 100%；conflicting payload 被 BLOCKED 的覆盖率为 100%。
- [ ] aggregate 模块 import/call Git adapter 或 worktree executor 的次数为 0；manual sync、artifact main mutation 与自动跨 leg rollback 次数为 0。
- [ ] `LegResultPayload`与`AggregateResult`自身持久化ref/receipt字段数为0；完整evidence DAG无回边/二次覆盖，published handle验证必须发生重读。
- [ ] TC-AW-008/009/012/014 和 REQ-AW-011..013/016..017/C004/NF001/NF005 的适用断言覆盖率为 100%。

## 阻塞说明

若受控 writer 不能保证 aggregate persisted/readback 后才投影，或非 PASS 仍可推进，必须回设计澄清，不得用最终一致性假设放行。

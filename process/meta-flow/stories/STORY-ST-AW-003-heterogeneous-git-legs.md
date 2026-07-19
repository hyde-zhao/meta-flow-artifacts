---
story_id: "ST-AW-003"
title: "执行 source-default 与 artifact-integration 异构 Git legs"
story_slug: "heterogeneous-git-legs"
status: "verified-with-risk"
priority: "P0"
wave: "W3"
depends_on: ["ST-AW-002"]
dependency_contracts:
  - {story_id: "ST-AW-002", type: "runtime", lld_gate: "worktree health contract declared", dev_gate: "upstream verified or approved equivalent fixture"}
feature_design_refs:
  - "process/docs/features/cr051-legs/DESIGN.md"
  - "process/docs/features/cr051-legs/TEST-PLAN.md"
  - "process/docs/features/cr051-legs/TASKS.md"
lld_policy:
  required_level: "full-lld"
  trigger_reasons: ["cross-repo", "authorization", "expected-oid", "shared-contract", "recovery"]
  rationale: "异构 target override、typed authz、expected OID、逐 leg 单写和 CR-050 兼容构成跨模块高风险契约。"
  waiver_reason: ""
  revisit_condition: "实现需要 artifact main refresh、paired-default target、跨 leg 自动回滚或共享 CLI 并行写。"
  evidence_path: "process/stories/STORY-ST-AW-003-heterogeneous-git-legs-LLD.md"
file_ownership:
  primary: ["meta_flow/workflow/artifact_leg_lifecycle.py", "tests/test_artifact_leg_lifecycle.py"]
  shared: ["meta_flow/workspace/git_sync.py", "meta_flow/workflow/git_branch_lifecycle.py", "meta_flow/cli.py"]
  merge_owner: "ST-AW-004"
  forbidden: ["artifact main/default target", "control/sibling checkout mutation", "cross-leg rollback", "meta_flow/workflow/artifact_aggregate.py"]
lld_gate:
  required_inputs: ["HLD", "ADR", "FEATURE-DESIGN-MATRIX", "Feature DESIGN", "Story"]
  design_evidence_type: "full-lld"
  design_evidence_path: "process/stories/STORY-ST-AW-003-heterogeneous-git-legs-LLD.md"
  status: "approved"
  confirmed: true
  approval_ref: "process/checkpoints/CP5-CR051-ALL-STORIES-LLD-BATCH.md"
  cp5_precheck_result: "process/checks/CP5-CR051-ST-AW-003-LLD-IMPLEMENTABILITY-R2.result.json"
implementation_gate:
  evidence_required: true
  evidence_path: "process/stories/STORY-ST-AW-003-heterogeneous-git-legs-IMPLEMENTATION.md"
  evidence_type: "implementation-md"
  implementation_objects: ["code", "template-schema", "guardrail-test", "docs-handoff"]
  test_plan_refs: ["process/docs/features/cr051-legs/TEST-PLAN.md"]
  local_validation_results: ["REV-AW003-001 R2: 6 targeted default-clock tests passed", "44 directed tests passed", "62 adjacent tests passed", "185 tests + 9 subtests passed in combined W3 suite", "ruff/format/py_compile/diff/return/evidence checks passed"]
  result_ref: "process/checks/CP6-CR051-ST-AW-003-heterogeneous-git-legs-CODING-DONE-R2.result.json"
  return_ref: "process/returns/ST-AW-003.CP6-R2.return.json"
  evidence_ref: "process/evidence/ST-AW-003.CP6-R2.index.json"
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
  cp7_result: "process/checks/CP7-CR051-ST-AW-003-heterogeneous-git-legs-VERIFICATION-DONE-R2.result.json"
  return_ref: "process/returns/ST-AW-003.CP7-R2.return.json"
  evidence_ref: "process/evidence/ST-AW-003.CP7-R2.index.json"
  closed_findings: ["REV-AW003-001"]
  superseded_cp7_result: "process/checks/CP7-CR051-ST-AW-003-heterogeneous-git-legs-VERIFICATION-DONE.result.json"
  remaining_risks: ["real-remote-unverified", "windows-native-git-pilot"]
  status: "PASS_WITH_RISK"
dev_gate:
  design_evidence_confirmed: true
  lld_confirmed: true
  dependencies_satisfied: true
  file_conflict_free: true
task_count: 5
created_at: "2026-07-18T06:35:00Z"
updated_at: "2026-07-18T15:20:00Z"
---

## 目标

为一个逻辑 CR 生成和执行相互独立的 source-default 与 artifact-integration legs；artifact 只消费 ST-AW-002 `WorktreeHealth.observation` rich snapshot；每条 leg 将写前可确定 digest 的 immutable payload 单写发布为 external receipt/handle，且绝不把 artifact main 放入 per-CR target。

## 开发上下文（dev_context）

- **输入文件**：ST-AW-001 route contract、ST-AW-002 worktree health contract、CP3 设计基线、Matrix、`cr051-legs` 三件套、REQ-AW-008..011/013/016、TC-AW-005..010/012/014。
- **输出文件**：`artifact_leg_lifecycle.py` 与定向测试；`git_sync.py`/`git_branch_lifecycle.py` 默认只读复用；CLI 接线由 ST-AW-004 merge。
- **设计约束**：source base/target=source default；artifact base/target=project integration；仅 HEALTHY+non-null observation+digest match 放行；payload 不含自身 ref/receipt；artifact main/control/sibling mutation=0；PARTIAL 只记录 effect；无跨 leg rollback。
- **Feature 设计引用**：frontmatter 的 3 个 legs pack 文件。
- **LLD 策略**：`full-lld`；需冻结 LegRequest/Plan/Attempt/Result、mode dispatch、authz、恢复与 CR-050 applicability。
- **平台目标**：local bare remote/command spy；无 forge API 或真实凭据。

### 依赖与并行门控

| 上游 Story | 类型 | LLD 门控 | 开发门控 | 说明 |
|---|---|---|---|---|
| ST-AW-002 | runtime | Lane B WorktreeHealth+observation port 已声明 | ST-AW-002 verified 或批准的等价 fixture | 只读 `health.observation`；HEALTHY+non-null+digest match；禁止复制第二套 schema。 |
| ST-AW-004 | contract peer | Payload/receipt/PublishedLegResultHandle 在 CP5 R2 同批冻结 | primary 文件不重叠；CLI 仅 ST-AW-004 写 | W3 条件并行，不形成反向 runtime 依赖。 |

### 文件所有权

| 类型 | 文件 | Owner / 合并规则 |
|---|---|---|
| primary | `artifact_leg_lifecycle.py`、`test_artifact_leg_lifecycle.py` | ST-AW-003 独占写 |
| shared-read/limited-write | `git_sync.py`、`git_branch_lifecycle.py` | 只复用通用 adapter/safety contract；默认不改 legacy paired behavior |
| shared | `meta_flow/cli.py` | ST-AW-004 merge owner；本 Story 禁止并行修改 |
| forbidden | aggregate primary、artifact main/default、control/sibling mutation、跨 leg rollback | 违反即设计偏差 |

### 文件系统布局

```text
meta_flow/workflow/
├── artifact_leg_lifecycle.py
└── git_branch_lifecycle.py            # 默认只读复用
meta_flow/workspace/git_sync.py         # shared adapter
tests/test_artifact_leg_lifecycle.py
```

### AI 可执行任务清单

| TASK-ID | 动作 | 目标 | 描述 |
|---|---|---|---|
| TASK-AW-003-01 | 设计 | full LLD | 冻结 LegRequest/Plan/Attempt/Result、mode dispatch 与错误枚举 |
| TASK-AW-003-02 | 创建 | lifecycle 模块 | 实现 source-default leg policy/executor |
| TASK-AW-003-03 | 创建 | lifecycle 模块 | 实现 artifact-integration leg policy/executor |
| TASK-AW-003-04 | 修改 | lifecycle 模块 | 实现 resume/abort、fresh drift、evidence 与 no-cross-leg-rollback |
| TASK-AW-003-05 | 创建 | 定向测试 | 建立 target/authz/correlation/partial/legacy regression fixture |

## 技术说明

正式设计证据为 `process/stories/STORY-ST-AW-003-heterogeneous-git-legs-LLD.md`；R2 自动预检为 `process/checks/CP5-CR051-ST-AW-003-LLD-IMPLEMENTABILITY-R2.result.json`，显式 supersede R1。消费端只读 `WorktreeHealth.observation`，结果端固定 payload→external receipt→published handle；全量 CP5 已批准，但 ST-AW-002 runtime dependency 未 verified 前仍不得实现。若其 `git_sync.py` 原语不足，必须提交 design delta 并请求单写窗口。

## 实现执行上下文（implementation_context）

| 项目 | 内容 |
|---|---|
| 实现证据类型 | implementation-md |
| 实现证据路径 | `process/stories/STORY-ST-AW-003-heterogeneous-git-legs-IMPLEMENTATION.md` |
| 实现对象清单 | code / template-schema / guardrail-test / docs-handoff |
| 设计契约映射状态 | pending |
| 单元测试 / Fixture 计划 | `process/docs/features/cr051-legs/TEST-PLAN.md` |
| 最小实现切片 | schema/policy → source/artifact executors → resume/evidence → fixture |
| 局部验证结果 | pending |
| 平台差异检查 | native Git argv-only；real remote N/A |
| 未覆盖项 | 真实 shared remote pilot |
| QA / Review / Doc 关注点 | target override、authz、fresh OID、no-main/no-sibling/no-cross-rollback |

## 验证上下文（validation_context）

| 项目 | 内容 |
|---|---|
| validation_mode | mixed |
| validation_target.sut_type | meta-flow-core-code |
| 验证重点 | target matrix、current/sibling dirty、OID drift、authz、dry-run、correlation、legacy regression |
| CP7 结论 | not-started |

## 量化验收标准（acceptance_criteria）

- [ ] source/artifact 两条 required leg 的 base/target 正确率为 2/2；artifact main/default plan、control checkout mutation、sibling touched path 均为 0。
- [ ] 每个当前 logical attempt 恰好有 source/artifact 各 1 个 matching immutable payload和可重读验证的PublishedLegResultHandle；raw/unpublished、receipt/ref/digest/key错配接受数为0。
- [ ] artifact leg 的HEALTHY+non-null observation+digest match判定正确率100%；消费侧平铺/复制WorktreeObservation字段数为0。
- [ ] payload中自身result_ref、append/write receipt、writer/time/receipt_digest字段数为0；writer失败生成PASS handle数为0，evidence-only retry重复Git数为0。
- [ ] current dirty、identity/ownership mismatch、expected OID stale、authz mismatch 的 mutation 前阻断率为 100%；sibling dirty 误阻断数为 0。
- [ ] 含 traversal、选项前缀、换行或 shell 元字符的 project/ref 输入拒绝率为 100%，额外命令执行数为 0。
- [ ] dry-run 本地/远端 mutation=0；artifact main refresh、自动 sync、cross-leg rollback 和禁止恢复命令执行数均为 0。
- [ ] TC-AW-005..010/012/014 的适用断言及 REQ-AW-008..011/013/016/C003..C004/NF003..NF005 覆盖率为 100%。

## 阻塞说明

若 CR-050 policy 无法显式 override target 而不改变 legacy 默认行为，或必须修改 ST-AW-004 primary，路由 `NEEDS_DESIGN_CLARIFICATION`。

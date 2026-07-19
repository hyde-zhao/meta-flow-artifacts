---
story_id: "ST-AW-002"
title: "管理可恢复的长期项目 worktree 与 create-only integration"
story_slug: "recoverable-project-worktree"
status: "verified-with-risk"
priority: "P0"
wave: "W2"
depends_on: ["ST-AW-001"]
dependency_contracts:
  - {story_id: "ST-AW-001", type: "contract", lld_gate: "route/context declared", dev_gate: "ProjectContext/RouteDecision frozen at CP5"}
feature_design_refs:
  - "process/docs/features/cr051-worktree/DESIGN.md"
  - "process/docs/features/cr051-worktree/TEST-PLAN.md"
  - "process/docs/features/cr051-worktree/TASKS.md"
lld_policy:
  required_level: "full-lld"
  trigger_reasons: ["non-atomic-state-machine", "concurrency", "filesystem", "remote-ref", "recovery"]
  rationale: "worktree switch 非原子且涉及本地/远端竞态、capacity、durable store、锁、权限和安全恢复。"
  waiver_reason: ""
  revisit_condition: "O-AW-01/02 无法证明时禁用 auto switch 并保留 manual/read-only 路径；不得引入 destructive fallback。"
  evidence_path: "process/stories/STORY-ST-AW-002-recoverable-project-worktree-LLD.md"
file_ownership:
  primary: ["meta_flow/workspace/project_worktree.py", "meta_flow/workspace/worktree_capacity.py", "meta_flow/workspace/worktree_journal.py", "tests/test_cr051_project_worktree.py", "tests/test_cr051_worktree_faults.py"]
  shared: ["meta_flow/workspace/git_sync.py", "tests/test_workspace_git_sync.py", "tests/test_git_branch_lifecycle.py", "meta_flow/cli.py"]
  merge_owner: "ST-AW-002"
  forbidden: ["artifact main per-CR update", "sibling project paths/refs/index", "destructive recovery", "real worktree/ref mutation during implementation"]
lld_gate:
  required_inputs: ["HLD", "ADR", "FEATURE-DESIGN-MATRIX", "Feature DESIGN", "Story"]
  design_evidence_type: "full-lld"
  design_evidence_path: "process/stories/STORY-ST-AW-002-recoverable-project-worktree-LLD.md"
  status: "approved"
  confirmed: true
  approval_ref: "process/checkpoints/CP5-CR051-ALL-STORIES-LLD-BATCH.md"
  cp5_precheck_result: "process/checks/CP5-CR051-ST-AW-002-LLD-IMPLEMENTABILITY-R2.result.json"
implementation_gate:
  evidence_required: true
  evidence_path: "process/stories/STORY-ST-AW-002-recoverable-project-worktree-IMPLEMENTATION.md"
  evidence_type: "implementation-md"
  implementation_objects: ["code", "template-schema", "guardrail-test", "docs-handoff"]
  test_plan_refs: ["process/docs/features/cr051-worktree/TEST-PLAN.md"]
  local_validation_results: ["83 tests passed", "ruff PASS", "py_compile PASS", "git diff PASS", "return/evidence PASS"]
  cp6_result: "process/checks/CP6-CR051-ST-AW-002-recoverable-project-worktree-CODING-DONE.result.json"
  return_packet_ref: "process/returns/ST-AW-002.CP6.return.json"
  evidence_index_ref: "process/evidence/ST-AW-002.CP6.index.json"
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
  cp7_result: "process/checks/CP7-CR051-ST-AW-002-recoverable-project-worktree-VERIFICATION-DONE.result.json"
  remaining_risks: ["real-remote-unverified"]
  status: "PASS_WITH_RISK"
dev_gate:
  design_evidence_confirmed: true
  lld_confirmed: true
  dependencies_satisfied: true
  file_conflict_free: true
task_count: 11
created_at: "2026-07-18T06:35:00Z"
updated_at: "2026-07-18T13:06:00Z"
---

## 目标

提供长期 project worktree 的 create/check/list/safe-remove、integration exact-OID create-only bootstrap 和 CP3-DC-01 可恢复 switch；任何容量、durability、权限、identity 或现场状态不确定都在 Git mutation 前阻断或保留现场。

## 开发上下文（dev_context）

- **输入文件**：ST-AW-001 route contract、CP3 HLD/ADR/Domain/Dependency、CR051 Matrix、`cr051-worktree` 三件套、O-AW-01/02、TC-AW-004/005/007/010..012/014/015。
- **输出文件**：3 个 primary 模块、2 个 primary 测试；`git_sync.py` 与回归测试使用单写窗口；`meta_flow/cli.py` 由 ST-AW-004 统一 merge。
- **设计约束**：existing-control + sibling-worktree；idle=integration、active=project CR branch；main 不是工作分支；intent durable 且 fresh observe 后才判 terminal；禁止 reset/clean/stash/force/delete 恢复。
- **冻结公共输出端口**：`observe_worktree(...) -> WorktreeObservation` 产生 rich immutable snapshot；`evaluate_worktree_health(...) -> WorktreeHealth` 通过 `health.observation` 原样承载 snapshot。`HEALTHY` 必须有非空 observation 且 digest 精确一致；非 HEALTHY 不授权 mutation，ST-AW-003 不得读取平铺的第二套 Health snapshot 字段。
- **Feature 设计引用**：frontmatter 的 3 个 worktree pack 文件。
- **LLD 策略**：`full-lld`；O-AW-01/02 是 CP5 强制证明义务而非可豁免风险。
- **平台目标**：临时 repo/local bare remote 与 filesystem fault fixture；真实 artifact/remote mutation 未授权。

### 依赖与并行门控

| 上游 Story | 类型 | LLD 门控 | 开发门控 | 说明 |
|---|---|---|---|---|
| ST-AW-001 | contract | RouteDecision/owned target 已声明 | ProjectContext/RouteDecision CP5 frozen | 不得修改 ST-AW-001 primary；接口变化走 design delta。 |

### 文件所有权

| 类型 | 文件 | Owner / 合并规则 |
|---|---|---|
| primary | `project_worktree.py`、`worktree_capacity.py`、`worktree_journal.py`、两个 CR051 test 文件 | ST-AW-002 独占写 |
| shared | `git_sync.py`、两个旧回归测试 | ST-AW-002 单写 merge window |
| shared | `meta_flow/cli.py` | ST-AW-004 是 CR-051 唯一 merge owner；本 Story 只提交接线契约 |
| forbidden | artifact main、sibling owned 对象、destructive recovery、真实仓 mutation | 违反即阻断 |

### 文件系统布局

```text
meta_flow/workspace/
├── project_worktree.py
├── worktree_capacity.py
├── worktree_journal.py
└── git_sync.py                         # shared/单写窗口
tests/
├── test_cr051_project_worktree.py
├── test_cr051_worktree_faults.py
├── test_workspace_git_sync.py          # shared 回归
└── test_git_branch_lifecycle.py         # shared 回归
```

### AI 可执行任务清单

| TASK-ID | 动作 | 目标 | 描述 |
|---|---|---|---|
| TASK-AW-W01 | 创建 | 3 个 primary 模块 | 冻结 registration/health/operation/bootstrap/capacity/journal schema |
| TASK-AW-W02 | 修改 | `git_sync.py` | 增加 argv-only worktree/common-dir/HEAD/Git-op/fresh-ref probes |
| TASK-AW-W03 | 创建 | capacity 模块/故障测试 | 实现 deterministic upper bound 与 bounded 512MiB eligibility |
| TASK-AW-W04 | 创建 | journal 模块/故障测试 | 实现 store-local durability chain 与 checksum/readback |
| TASK-AW-W05 | 修改 | worktree 模块 | 实现 project lock、registration、health、branch role、stale diagnosis |
| TASK-AW-W06 | 修改 | worktree 模块 | 实现 integration exact-OID create-only bootstrap/race 分类 |
| TASK-AW-W07 | 修改 | worktree 模块 | 实现 create 与独立授权 safe-remove |
| TASK-AW-W08 | 修改 | worktree 模块 | 实现 switch/post-observe/conditional rollback/resume |
| TASK-AW-W09 | 创建 | worktree integration tests | 建立双项目/bare-remote/dry-run fixture |
| TASK-AW-W10 | 接线 | CLI 契约 | 条件式提交给 ST-AW-004 merge owner |
| TASK-AW-W11 | 验证 | 批准文件范围 | Ruff、回归、危险命令、真实路径/ref deny、契约审计 |

## 技术说明

正式设计证据为 `process/stories/STORY-ST-AW-002-recoverable-project-worktree-LLD.md`；当前单 Story自动预检为 `process/checks/CP5-CR051-ST-AW-002-LLD-IMPLEMENTABILITY-R2.result.json`，显式 supersede R1。R2 冻结 `WorktreeObservation -> WorktreeHealth(observation)` 输出端口；全量 CP5 已批准。O-AW-01/02 继续是不可豁免 CP6/CP7 义务，本 Story 等 W1 调度收敛后进入 W2；批准不授权真实 Git/worktree/ref/remote mutation。

## 实现执行上下文（implementation_context）

| 项目 | 内容 |
|---|---|
| 实现证据类型 | implementation-md |
| 实现证据路径 | `process/stories/STORY-ST-AW-002-recoverable-project-worktree-IMPLEMENTATION.md` |
| 实现对象清单 | code / template-schema / guardrail-test / docs-handoff |
| 设计契约映射状态 | pending |
| 单元测试 / Fixture 计划 | `process/docs/features/cr051-worktree/TEST-PLAN.md` |
| 最小实现切片 | schema/probes → capacity+journal → health/bootstrap → switch/recover → fixture |
| 局部验证结果 | pending |
| 平台差异检查 | filesystem fsync/replace/cross-device 与 Git 2.43+ fixture |
| 未覆盖项 | 真实 shared remote pilot（Out of Scope） |
| QA / Review / Doc 关注点 | O-AW-01/02、fresh observation、dangerous command=0、sibling isolation |

## 验证上下文（validation_context）

| 项目 | 内容 |
|---|---|
| validation_mode | mixed |
| validation_target.sut_type | meta-flow-core-code |
| 验证重点 | CAP-01..11、DUR-01..14、WT-01..14、TC-AW-004/005/007/010..012/014/015 |
| 真实运行授权 | 仅临时 fixture；真实 worktree/ref/remote 仍未授权 |
| CP7 结论 | not-started |

## 量化验收标准（acceptance_criteria）

- [ ] integration 缺失 fixture 恰好执行 1 次 exact-OID create-only；已存在/并发同值路径 recreate/reset/orphan 次数为 0。
- [ ] 两项目 fixture 的 idle/active branch role 命名匹配率为 100%，main 作为 working branch、index.lock 争用、cross-project touched path 均为 0。
- [ ] O-AW-01 的适用 bounded profile false-safe=0、underestimate=0；无权限/无法枚举/误差未知均 100% 在 mutation 前 BLOCKED。
- [ ] O-AW-02 覆盖 ENOSPC、EACCES、file-fsync、replace、dir-fsync、torn/corrupt、process-kill、cross-device 至少 8 类故障，提前 Git mutation=0，重复 resume 终态一致率=100%。
- [ ] 每次 switch/rollback attempt 均有 durable intent 与 fresh post-observation；命令退出码单独产生 PASS 的次数为 0。
- [ ] dirty/unknown/active/recovery-required remove 负例删除次数为 0；reset-hard/clean/stash/force/branch-delete 等禁止恢复 argv 次数为 0。
- [ ] worktree pack P0/P1 fixture 与 TC-AW 适用断言覆盖率为 100%，真实 artifact 变更数为 0。

## 阻塞说明

O-AW-01/02 当前是 non-blocking CP5 义务；任一证明失败必须禁用 auto switch 或在 Git mutation 前 BLOCKED，不能用风险接受绕过。

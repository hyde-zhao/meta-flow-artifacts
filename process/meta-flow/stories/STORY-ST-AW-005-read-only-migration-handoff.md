---
story_id: "ST-AW-005"
title: "生成零 mutation 的逐项目 migration manifest 与人工交接"
story_slug: "read-only-migration-handoff"
status: "verified-with-risk"
priority: "P1"
wave: "W4"
depends_on: ["ST-AW-003", "ST-AW-004"]
dependency_contracts:
  - {story_id: "ST-AW-003", type: "runtime", lld_gate: "leg handoff contract declared", dev_gate: "upstream verified"}
  - {story_id: "ST-AW-004", type: "runtime", lld_gate: "aggregate/readiness contract declared", dev_gate: "upstream verified"}
feature_design_refs:
  - "process/docs/features/cr051-migration/DESIGN.md"
  - "process/docs/features/cr051-migration/TEST-PLAN.md"
  - "process/docs/features/cr051-migration/TASKS.md"
lld_policy:
  required_level: "technical-note"
  trigger_reasons: ["read-only-migration", "manifest-schema", "deny-mutation", "follow-up-tracking"]
  rationale: "当前实现只读并只产出 manifest/checklist；无运行态 writer、无文件/link/worktree/ref mutation，正式技术说明足以审查。"
  waiver_reason: ""
  revisit_condition: "增加任何 file/link/worktree/ref/remote mutation、自动同步、可执行 migration script 或跨 Story mutation contract。"
  evidence_path: "process/stories/STORY-ST-AW-005-read-only-migration-handoff.md#技术说明"
file_ownership:
  primary: ["meta_flow/workspace/artifact_migration.py", "tests/test_artifact_migration.py"]
  shared: ["meta_flow/workspace/project_artifact_routing.py", "meta_flow/workspace/project_worktree.py", "meta_flow/cli.py"]
  merge_owner: "ST-AW-004"
  forbidden: ["target project file/link/worktree/ref mutation", "artifact main/remote/scheduler", "process/quant-lab/**", "sibling content", "sync helper enable/invoke"]
lld_gate:
  required_inputs: ["HLD", "ADR", "FEATURE-DESIGN-MATRIX", "Feature DESIGN", "Story"]
  design_evidence_type: "technical-note"
  design_evidence_path: "process/stories/STORY-ST-AW-005-read-only-migration-handoff.md#技术说明"
  status: "approved"
  confirmed: true
  approval_ref: "process/checkpoints/CP5-CR051-ALL-STORIES-LLD-BATCH.md"
implementation_gate:
  evidence_required: true
  evidence_path: "process/stories/STORY-ST-AW-005-read-only-migration-handoff-IMPLEMENTATION.md"
  evidence_type: "implementation-md"
  implementation_objects: ["code", "template-schema", "guardrail-test", "docs-handoff"]
  test_plan_refs: ["process/docs/features/cr051-migration/TEST-PLAN.md"]
  local_validation_results: ["REV-AW005-001 R2: 3 red-to-green probes passed", "26 directed tests passed", "95 adjacent tests passed", "280 combined tests + 9 subtests passed", "ruff/format/py_compile/return/evidence/boundary checks passed"]
  result_ref: "process/checks/CP6-CR051-ST-AW-005-read-only-migration-handoff-CODING-DONE-R2.result.json"
  return_ref: "process/returns/ST-AW-005.CP6-R2.return.json"
  evidence_ref: "process/evidence/ST-AW-005.CP6-R2.index.json"
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
  cp7_result: "process/checks/CP7-CR051-ST-AW-005-read-only-migration-handoff-VERIFICATION-DONE-R2.result.json"
  return_ref: "process/returns/ST-AW-005.CP7-R2.return.json"
  evidence_ref: "process/evidence/ST-AW-005.CP7-R2.index.json"
  closed_findings: ["REV-AW005-001"]
  superseded_cp7_result: "process/checks/CP7-CR051-ST-AW-005-read-only-migration-handoff-VERIFICATION-DONE.result.json"
  remaining_risks: ["real-project-migration-not-authorized", "real-remote-not-validated"]
  status: "PASS_WITH_RISK"
dev_gate:
  design_evidence_confirmed: true
  lld_confirmed: true
  dependencies_satisfied: true
  file_conflict_free: true
task_count: 4
created_at: "2026-07-18T06:35:00Z"
updated_at: "2026-07-18T16:39:00Z"
---

## 目标

对一个显式目标项目执行只读 migration preflight，生成 portable mapping、hash/count、link/worktree/ref readiness、验证和回滚 checklist；CR-051 内真实文件、link、worktree、ref、remote、commit 和 publish 变化全部保持为 0。

## 开发上下文（dev_context）

- **输入文件**：ST-AW-001/002 route+health contract、ST-AW-003/004 已验证结果契约、CP3 设计、Matrix、`cr051-migration` 三件套、REQ-AW-014..017/C001/NF005、TC-AW-012/013/015、O-AW-03。
- **输出文件**：`artifact_migration.py`、`test_artifact_migration.py`；不直接修改 route/worktree primary；CLI 如确需接线串行提交给 ST-AW-004 merge owner。
- **设计约束**：allowlisted lstat/enumeration/hash/count、symlink no-follow、incomplete/unreadable fail closed、manifest 只含 proposed/manual steps、O-AW-03 只生成 candidate。
- **Feature 设计引用**：frontmatter 的 3 个 migration pack 文件。
- **LLD 策略**：`technical-note`；本节是 CP5 正式证据，任何 mutation 范围增加即升级 full-lld。
- **平台目标**：临时 filesystem fixture + snapshot/mutation spy；真实项目迁移由未来独立 CR。

### 依赖与并行门控

| 上游 Story | 类型 | LLD 门控 | 开发门控 | 说明 |
|---|---|---|---|---|
| ST-AW-003 | runtime | leg handoff fields 已声明 | upstream verified | 不消费 mutable executor state。 |
| ST-AW-004 | runtime | aggregate/readiness contract 已声明 | upstream verified | manifest 只引用稳定结果，不触发 aggregate/leg。 |

### 文件所有权

| 类型 | 文件 | Owner / 合并规则 |
|---|---|---|
| primary | `artifact_migration.py`、`test_artifact_migration.py` | ST-AW-005 独占写；只读 preflight |
| shared-read | routing/worktree primary | 不修改、不调用 mutation API |
| shared | `meta_flow/cli.py` | ST-AW-004 merge owner；ST-AW-005 只串行提交最小接线 |
| forbidden | 目标项目/sibling/quant-lab 内容写入、Git/ref/remote、scheduler/helper | touched/mutation/read expansion 均按 deny contract 检查 |

### 文件系统布局

```text
meta_flow/workspace/
├── artifact_migration.py               # 新建；read-only preflight/manifest/evaluator
├── project_artifact_routing.py         # 只读 contract
└── project_worktree.py                 # 只读 health contract
tests/test_artifact_migration.py
```

### AI 可执行任务清单

| TASK-ID | 动作 | 目标 | 描述 |
|---|---|---|---|
| TASK-AW-005-01 | 设计 | 本技术说明 | 冻结 read-only API、Manifest schema、scope/symlink/readiness/errors 与重访条件 |
| TASK-AW-005-02 | 创建 | migration 模块 | 实现 allowlisted lstat/enumeration/hash/count 与 handoff |
| TASK-AW-005-03 | 创建 | migration 模块 | 实现 O-AW-03 threshold evaluator 和去重 candidate |
| TASK-AW-005-04 | 创建 | 定向测试 | 覆盖 schema、permission/symlink/scope、snapshot、threshold/idempotency |

## 技术说明

| 项目 | 内容 |
|---|---|
| 设计证据类型 | technical-note（CP5 正式证据） |
| 确认状态 | CP5 `approved`、`confirmed=true`；仍需等待 ST-AW-003/004 通过 runtime gate 后才可实现 |
| 设计依据 | ADR-AW-006/007、FEAT-AW-05 DESIGN/TEST-PLAN/TASKS、REQ-AW-014..017/C001/NF005、TC-AW-012/013/015 |
| 文件影响 | 新建 `meta_flow/workspace/artifact_migration.py` 与 `tests/test_artifact_migration.py`；routing/worktree 模块只读；`meta_flow/cli.py` 仅由 ST-AW-004 merge owner 串行接线；删除文件=0 |
| 接口 / API | `build_migration_manifest(explicit_project, route_decision, worktree_health, metrics_summary) -> MigrationManifest`；只接收 validated immutable inputs，不调用 lifecycle/mutator |
| 数据 | Manifest 必含 identity、allowlisted scope、portable mapping、file/link/dir count、bytes/hash、link plan、worktree/ref readiness、`READY|BLOCKED|MANUAL_REVIEW`、validation、rollback、authorization、ops follow-up、evidence digest；设备绝对路径不可作为 canonical 字段 |
| 权限与读取 | 仅显式目标项目 allowlisted roots；使用 lstat 且不跟随越界 symlink；sibling、`process/quant-lab/**` 和未显式目标读取数=0；无 runtime authorization 不影响只读生成，但所有 proposed mutation 标记为未授权 |
| 主流程 | validate project/context → read fresh health → allowlisted enumerate/hash/count → classify conflict/unreadable → render proposed manual steps → evaluate O-AW-03 → emit manifest/evidence；全程 mutation=0 |
| 异常与回退 | identity/route/scope/worktree stale→BLOCKED；unreadable/incomplete/symlink ambiguity→MANUAL_REVIEW 或 BLOCKED；不通过写入/修复探测；渲染失败不执行 migration |
| O-AW-03 | 单项目每周 sync >=3 次且连续 4 周，或中位耗时 >10 分钟，或可避免调度阻塞率 >5%；任一满足只生成去重 follow-up CR candidate，helper enable/invoke/scheduler/remote write=0；数据不足返回 `insufficient-data` |
| 测试入口 | TEST-PLAN 的 schema/scope/symlink/EACCES/hash/snapshot/mutation-spy/threshold/idempotency fixture；TC-AW-012/013/015 |
| 回滚交接 | Manifest 只列未来执行方的备份、验证、恢复触发条件与 owner；不生成或执行 mutation script，不宣称真实迁移完成 |
| 风险与重访条件 | 任何 move/copy/delete/write/link/worktree/ref/commit/push、自动 sync、可执行迁移脚本、跨 Story mutation contract、或写入 route/runtime truth source，立即升级 full-lld 并重开范围/授权 |
| 偏离记录 | 任何实现偏离上述 deny-mutation/read scope 必须停止并路由 `NEEDS_DESIGN_CLARIFICATION`，不得在 CP6 静默接受 |

### Technical-note 取舍与重访记录

| Clarification / Open | 问题 | 推荐与备选 | 当前结论 | 影响面 | 重访条件 |
|---|---|---|---|---|---|
| TN-AW-005-01 | 是否保持 technical-note，还是因“迁移”字样直接升级 full LLD | 推荐：保持只读 technical-note；备选：若加入任何 mutation/runtime writer 则升级 full LLD | 推荐成立；本 Story 只生成 manifest/checklist，目标 mutation=0 | 文件 owner、权限、测试、CP5 证据规模 | 任一 move/copy/delete/write/link/worktree/ref/remote/commit/push、可执行迁移脚本或跨 Story mutation contract |
| O-AW-03 | 达到 manual-sync 阈值后是否自动启用同步助手 | 推荐：只生成独立 follow-up CR candidate；备选：自动启用（违反已批准边界） | 推荐已由 CP3/Matrix 固定；candidate-only | 运维权限、scheduler、remote write、回滚 | 后续独立 CR 经 scope/security/runtime authorization/rollback 确认 |

当前需要 Host broker 的 clarification item：0；OPEN/Spike：0。O-AW-03 为 `non-blocking-open / follow-up-tracking`，不阻断本技术说明审查。

## 实现执行上下文（implementation_context）

| 项目 | 内容 |
|---|---|
| 实现证据类型 | implementation-md |
| 实现证据路径 | `process/stories/STORY-ST-AW-005-read-only-migration-handoff-IMPLEMENTATION.md` |
| 实现对象清单 | code / template-schema / guardrail-test / docs-handoff |
| 设计契约映射状态 | pending |
| 单元测试 / Fixture 计划 | `process/docs/features/cr051-migration/TEST-PLAN.md` |
| 最小实现切片 | manifest schema → scope/hash → O-AW-03 evaluator → fixtures |
| 局部验证结果 | pending |
| 平台差异检查 | symlink/no-follow/permission/path separator；真实迁移 N/A |
| 未覆盖项 | 真实 artifact 迁移、link 挂接、worktree/ref/remote pilot |
| QA / Review / Doc 关注点 | snapshot 零变化、read scope、portable mapping、candidate-only、用户手册交接 |

## 验证上下文（validation_context）

| 项目 | 内容 |
|---|---|
| validation_mode | mixed |
| validation_target.sut_type | meta-flow-core-code |
| 验证重点 | schema、scope/symlink/permission、hash/count、readiness、snapshot/mutation spy、O-AW-03 boundaries |
| 人工 / 语义审查 | Manifest/rollback checklist 可操作，但不得误读为授权 |
| CP7 结论 | not-started |

## 量化验收标准（acceptance_criteria）

- [ ] Manifest 的 identity/scope/mapping/summary/link plan/worktree-ref readiness/readiness/validation/rollback/authorization/ops follow-up/evidence 11 类分区字段覆盖率为 100%。
- [ ] preflight 前后目标项目文件/link/worktree/local-ref/remote-ref 快照差异为 0，commit/push/publish/helper/scheduler 调用次数为 0。
- [ ] sibling、`process/quant-lab/**`、未显式目标内容读取数为 0；EACCES、broken/out-of-scope symlink、枚举不完整路径误报 READY 次数为 0。
- [ ] portable mapping 中设备绝对路径违规数为 0；相同输入重复生成 mapping/hash/count/readiness/digest 一致率为 100%。
- [ ] O-AW-03 三个阈值的边界用例判定正确率为 100%；阈值命中只生成 1 个去重 follow-up candidate，自动 sync/helper enable 次数为 0。
- [ ] TC-AW-012/013/015 与 REQ-AW-014..017/C001/NF005 的适用断言覆盖率为 100%，真实迁移和软链接变更数为 0。

## 阻塞说明

无当前 blocker；technical-note 当前仅因纯只读范围成立。任何 mutation 或 executable migration 能力会立即阻断并升级 full-lld。

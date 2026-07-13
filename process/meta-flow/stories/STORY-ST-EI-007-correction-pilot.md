---
story_id: "ST-EI-007"
title: "建立 append-only correction 与 CR-163 pilot adapter"
story_slug: "correction-pilot"
cr_id: "CR-046"
status: "verified-with-risk"
priority: "P1"
required_level: "full-lld"
wave: "W6"
lld_wave: "LW3"
qa_wave: "QW6"
depends_on: ["ST-EI-004", "ST-EI-006"]
feature_refs: ["FEAT-EI-CORRECTION"]
feature_contract_summary: "Apply only append-only, source-hash-bound corrections and keep CR-163 migration in a separately authorized dry-run adapter."
cr_delta_summary: "Formalizes post-close correction lifecycle and produces a 23-target synthetic pilot preflight."
dependency_inputs: ["ST-EI-004 governance", "ST-EI-006 replay/audit", "CP5 approved full LLD batch"]
risk_profile: "architecture-major"
allowed_write_paths: ["meta_flow/checks/correction.py", "meta_flow/evidence/pilot_adapter.py", "tests/test_cr046_replay_correction.py"]
forbidden_write_paths: ["quant-lab/**", "process/archive/**", "delivery/**", ".git/**"]
acceptance: ["invalid correction append=0", "original mutation=0", "synthetic pilot targets=23", "real target writes=0"]
verification_plan: ["uv run pytest tests/test_cr046_replay_correction.py -q"]
authz_policy_refs: ["NO_CREDENTIAL_READ", "NO_RUNTIME", "NO_PRODUCTION_WRITE", "NO_REPOSITORY_PUBLICATION"]
dependency_contracts: [{story_id: "ST-EI-004", type: "runtime"}, {story_id: "ST-EI-006", type: "runtime"}]
feature_design_refs:
  - "/home/hyde/workspace/meta-flow-artifacts/process/meta-flow/docs/features/cr046-correction/DESIGN.md"
  - "/home/hyde/workspace/meta-flow-artifacts/process/meta-flow/docs/features/cr046-correction/TEST-PLAN.md"
  - "/home/hyde/workspace/meta-flow-artifacts/process/meta-flow/docs/features/cr046-correction/TASKS.md"
  - "/home/hyde/workspace/meta-flow-artifacts/process/meta-flow/docs/features/cr046-observability/DESIGN.md"
lld_policy:
  required_level: "full-lld"
  trigger_reasons: ["cross-module", "public-contract", "state-or-migration", "security-or-audit"]
  rationale: "CR-046 architecture-major Story; technical-note/waived prohibited without CP5 decision"
  waiver_reason: ""
  revisit_condition: "none; any downgrade requires CP5 user decision"
  evidence_path: "process/stories/STORY-ST-EI-007-correction-pilot-LLD.md"
file_ownership:
  primary: ["meta_flow/evidence/correction.py", "meta_flow/evidence/pilot_adapter.py", "tests/test_cr046_correction.py"]
  shared: ["meta_flow/workflow/cr_lifecycle.py", "meta_flow/cli.py"]
  merge_owner: "ST-EI-007"
  forbidden: ["quant-lab lineage business source", "process/archive/** in-place mutation", "credentials/runtime/production-write/publish/trading", "repository commit/push"]
lld_gate:
  required_inputs: ["confirmed HLD", "accepted ADR", "CR046-FEATURE-DESIGN-MATRIX", "Feature DESIGN/TEST-PLAN/TASKS", "Story"]
  design_evidence_type: "full-lld"
  design_evidence_path: "process/stories/STORY-ST-EI-007-correction-pilot-LLD.md"
  status: "approved"
implementation_gate:
  evidence_required: true
  evidence_path: "process/stories/STORY-ST-EI-007-correction-pilot-IMPLEMENTATION.md"
  evidence_type: "implementation-md"
  test_plan_refs: ["/home/hyde/workspace/meta-flow-artifacts/process/meta-flow/docs/features/cr046-correction/TEST-PLAN.md"]
  status: "PASS"
verification_gate:
  validation_mode: "mixed"
  validation_target: {sut_type: "meta-flow-core-code", native_test_required: true, workflow_eval_required: false, runtime_authorization_required: []}
  cp7_result: "PASS_WITH_RISK"
  remaining_risks: []
  status: "PASS_WITH_RISK"
dev_gate:
  design_evidence_confirmed: true
  lld_confirmed: true
  dependencies_satisfied: true
  file_conflict_free: true
task_count: 4
created_at: "2026-07-12T03:34:00Z"
updated_at: "2026-07-12T07:56:00Z"
---

# ST-EI-007 建立 append-only correction 与 CR-163 pilot adapter

## 目标

用versioned append-only correction lifecycle承载历史修正，并隔离真实CR-163 pilot授权与业务代码边界。

## 开发上下文（dev_context）

- 来源需求：REQ-EI-016..020, REQ-EI-NF002
- 输入：confirmed CR046 HLD/ADR/platform contract、Feature DESIGN/TEST-PLAN/TASKS、本 Story。
- 输出：`meta_flow/evidence/correction.py`、`meta_flow/evidence/pilot_adapter.py`、`tests/test_cr046_correction.py` 及相应测试；具体新增模块名由 LLD 冻结。
- 设计约束：canonical files保持事实源；不可伪造receipt/telemetry/provenance；CURRENT与REQUIRED分层；A-baseline默认。
- 平台目标：repository contract/fixture可独立验证；真实custom profile/model未有平台receipt时必须unavailable。

### 依赖与并行门控

| 上游 | 类型 | LLD门控 | 开发门控 |
|---|---|---|---|
| ST-EI-004 | runtime | declared contract可起草 | 上游verified |
| ST-EI-006 | runtime | declared contract可起草 | 上游verified |

### 文件所有权

| 类型 | 文件 | 规则 |
|---|---|---|
| primary | `meta_flow/evidence/correction.py`<br>`meta_flow/evidence/pilot_adapter.py`<br>`tests/test_cr046_correction.py` | 当前 Story独占 |
| shared | `meta_flow/workflow/cr_lifecycle.py`<br>`meta_flow/cli.py` | merge_owner=ST-EI-007；按DAG串行合并 |
| forbidden | quant-lab lineage业务源码、历史原位改写、runtime/credentials/publish/commit/push | 零写入 |

### AI 可执行任务清单

| TASK-ID | 动作 | 描述 |
|---|---|---|
| TASK-EI-007-01 | 修改/创建 | 按Feature TASKS与本Story LLD实现并验证 |
| TASK-EI-007-02 | 修改/创建 | 按Feature TASKS与本Story LLD实现并验证 |
| TASK-EI-007-03 | 修改/创建 | 按Feature TASKS与本Story LLD实现并验证 |
| TASK-EI-007-04 | 修改/创建 | 按Feature TASKS与本Story LLD实现并验证 |

## 实现执行上下文（implementation_context）

| 项目 | 内容 |
|---|---|
| 实现证据 | `process/stories/STORY-ST-EI-007-correction-pilot-IMPLEMENTATION.md`（CP6前生成） |
| 设计契约映射 | pending；必须覆盖Feature DESIGN、Story LLD和AC |
| Fixture/测试 | `/home/hyde/workspace/meta-flow-artifacts/process/meta-flow/docs/features/cr046-correction/TEST-PLAN.md` |
| 平台差异 | CURRENT工具无selector/discovery/receipt；不得声称attested |
| QA关注 | failure paths、immutable history、三轴attestation及负向fixtures |

## 技术说明

本 Story 为 `full-lld`；CP5正式证据路径为 `process/stories/STORY-ST-EI-007-correction-pilot-LLD.md`。当前卡片只冻结边界，不替代LLD。

## 验证上下文（validation_context）

采用 mixed（静态、单元、contract fixture、集成、回归、人工语义审查）。真实Codex runtime conformance只有Conditional-B前置全部满足时运行，否则记录NOT_RUN/UNAVAILABLE并按CP3-DQ-05限制CP7/CP8结论。

## 量化验收标准（acceptance_criteria）

- [ ] 非法原位修改拒绝率=100%且原历史hash mutation=0
- [ ] correction allowlist/supersedes/cycle规则覆盖率=100%
- [ ] legacy codex_agent_name迁移为D3 self-declared-unverifiable且不解析model/profile
- [ ] fixture replay 23/23；真实pilot保持未执行直到独立授权
- [ ] quant-lab lineage业务源码写入数=0
- [ ] file_ownership.forbidden命中数=0，未授权runtime/credentials/publish操作=0。
- [ ] CP6 return/evidence与CP7独立结论可回链到全部AC。

## 阻塞说明

当前设计无CP4 blocker。平台REQUIRED extension尚非CURRENT，只限制runtime attestation，不阻塞A-baseline repository设计/实现。

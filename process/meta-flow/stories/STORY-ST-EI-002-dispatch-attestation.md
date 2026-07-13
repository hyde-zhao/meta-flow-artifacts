---
story_id: "ST-EI-002"
title: "建立 dispatch attempt 与 custom-agent 平台证明链"
story_slug: "dispatch-attestation"
cr_id: "CR-046"
status: "verified-with-risk"
priority: "P0"
wave: "W2"
lld_wave: "LW1"
qa_wave: "QW2"
required_level: "full-lld"
depends_on: ["ST-EI-001"]
dependency_contracts: [{story_id: "ST-EI-001", type: "contract"}]
feature_refs: ["FEAT-EI-CORE"]
feature_contract_summary: "Normalize typed dispatch/attempt/thread/receipt evidence; D2 config stays distinct from D0 runtime discovery and no ledger declaration can become a platform fact."
cr_delta_summary: "Adds fail-closed custom-agent profile attestation, attempt closure and immutable thread reuse contracts while retaining the A-baseline for current platform limits."
dependency_inputs: ["ST-EI-001 CP7 PASS_WITH_RISK frozen chronology contract", "CP5 approved full LLD batch", "CR046 custom-agent capability probe"]
risk_profile: "architecture-major"
allowed_write_paths: ["meta_flow/evidence/**", "meta_flow/state/event_ledger.py", "tests/test_cr046_dispatch_attestation.py"]
forbidden_write_paths: ["quant-lab/**", "process/archive/**", "delivery/**", "meta_flow/checks/cp_result.py", ".git/**"]
acceptance:
  - "适用 attempt terminal closure=100%"
  - "D2提升为D0次数=0"
  - "PC-18 freshness/re-probe覆盖 expiry/session/epoch/hash/schema/reload 6/6"
  - "PC-19 无 reuse receipt 时 followup verified/model_attested 均为 false"
  - "profile升级通过new spawn覆盖率=100%"
  - "file_ownership.forbidden命中数=0，未授权runtime/credentials/publish操作=0"
verification_plan:
  - "uv run pytest tests/test_cr046_dispatch_attestation.py -q"
  - "uv run pytest tests/test_cp_result_event_ledger.py -q"
  - "uv run meta-flow event dispatch-check --ledger process/state/AGENT-DISPATCH-LEDGER.ndjson"
authz_policy_refs: ["NO_CREDENTIAL_READ", "NO_RUNTIME", "NO_PRODUCTION_WRITE", "NO_REPOSITORY_PUBLICATION"]
feature_design_refs:
  - "/home/hyde/workspace/meta-flow-artifacts/process/meta-flow/docs/features/cr046-core/DESIGN.md"
  - "/home/hyde/workspace/meta-flow-artifacts/process/meta-flow/docs/features/cr046-core/TEST-PLAN.md"
  - "/home/hyde/workspace/meta-flow-artifacts/process/meta-flow/docs/features/cr046-core/TASKS.md"
lld_policy:
  required_level: "full-lld"
  trigger_reasons: ["cross-module", "public-contract", "state-or-migration", "security-or-audit"]
  rationale: "CR-046 architecture-major Story; technical-note/waived prohibited without CP5 decision"
  waiver_reason: ""
  revisit_condition: "none; any downgrade requires CP5 user decision"
  evidence_path: "process/stories/STORY-ST-EI-002-dispatch-attestation-LLD.md"
file_ownership:
  primary: ["meta_flow/evidence/dispatch.py", "meta_flow/evidence/platform_contract.py", "tests/test_cr046_dispatch_attestation.py"]
  shared: ["meta_flow/state/event_ledger.py", "meta_flow/cli.py"]
  merge_owner: "ST-EI-002"
  forbidden: ["quant-lab lineage business source", "process/archive/** in-place mutation", "credentials/runtime/production-write/publish/trading", "repository commit/push"]
lld_gate:
  required_inputs: ["confirmed HLD", "accepted ADR", "CR046-FEATURE-DESIGN-MATRIX", "Feature DESIGN/TEST-PLAN/TASKS", "Story"]
  design_evidence_type: "full-lld"
  design_evidence_path: "process/stories/STORY-ST-EI-002-dispatch-attestation-LLD.md"
  status: "approved"
implementation_gate:
  evidence_required: true
  evidence_path: "process/stories/STORY-ST-EI-002-dispatch-attestation-IMPLEMENTATION.md"
  evidence_type: "implementation-md"
  test_plan_refs: ["/home/hyde/workspace/meta-flow-artifacts/process/meta-flow/docs/features/cr046-core/TEST-PLAN.md"]
  status: "completed"
verification_gate:
  validation_mode: "mixed"
  validation_target: {sut_type: "meta-flow-core-code", native_test_required: true, workflow_eval_required: false, runtime_authorization_required: []}
  cp7_result: "PASS_WITH_RISK"
  remaining_risks: ["CR046-RISK-PLATFORM-RECEIPT-UNAVAILABLE", "CR046-RISK-NO-INDEPENDENT-CP7-AGENT"]
  status: "PASS_WITH_RISK"
dev_gate:
  design_evidence_confirmed: true
  lld_confirmed: true
  dependencies_satisfied: true
  file_conflict_free: true
task_count: 3
created_at: "2026-07-12T03:34:00Z"
updated_at: "2026-07-12T03:34:00Z"
---

# ST-EI-002 建立 dispatch attempt 与 custom-agent 平台证明链

## 目标

让每个 dispatch attempt 有 typed identity、terminal closure、D0 discovery、显式 selector、receipt 与 immutable thread identity。

## 开发上下文（dev_context）

- 来源需求：REQ-EI-003..005, REQ-EI-019, REQ-EI-023, REQ-EI-C001
- 输入：confirmed CR046 HLD/ADR/platform contract、Feature DESIGN/TEST-PLAN/TASKS、本 Story。
- 输出：`meta_flow/evidence/dispatch.py`、`meta_flow/evidence/platform_contract.py`、`tests/test_cr046_dispatch_attestation.py` 及相应测试；具体新增模块名由 LLD 冻结。
- 设计约束：canonical files保持事实源；不可伪造receipt/telemetry/provenance；CURRENT与REQUIRED分层；A-baseline默认。
- 平台目标：repository contract/fixture可独立验证；真实custom profile/model未有平台receipt时必须unavailable。

### 依赖与并行门控

| 上游 | 类型 | LLD门控 | 开发门控 |
|---|---|---|---|
| ST-EI-001 | contract | declared contract可起草 | 上游contract frozen |

### 文件所有权

| 类型 | 文件 | 规则 |
|---|---|---|
| primary | `meta_flow/evidence/dispatch.py`<br>`meta_flow/evidence/platform_contract.py`<br>`tests/test_cr046_dispatch_attestation.py` | 当前 Story独占 |
| shared | `meta_flow/state/event_ledger.py`<br>`meta_flow/cli.py` | merge_owner=ST-EI-002；按DAG串行合并 |
| forbidden | quant-lab lineage业务源码、历史原位改写、runtime/credentials/publish/commit/push | 零写入 |

### AI 可执行任务清单

| TASK-ID | 动作 | 描述 |
|---|---|---|
| TASK-EI-002-01 | 修改/创建 | 按Feature TASKS与本Story LLD实现并验证 |
| TASK-EI-002-02 | 修改/创建 | 按Feature TASKS与本Story LLD实现并验证 |
| TASK-EI-002-03 | 修改/创建 | 按Feature TASKS与本Story LLD实现并验证 |

## 实现执行上下文（implementation_context）

| 项目 | 内容 |
|---|---|
| 实现证据 | `process/stories/STORY-ST-EI-002-dispatch-attestation-IMPLEMENTATION.md`（CP6前生成） |
| 设计契约映射 | pending；必须覆盖Feature DESIGN、Story LLD和AC |
| Fixture/测试 | `/home/hyde/workspace/meta-flow-artifacts/process/meta-flow/docs/features/cr046-core/TEST-PLAN.md` |
| 平台差异 | CURRENT工具无selector/discovery/receipt；不得声称attested |
| QA关注 | failure paths、immutable history、三轴attestation及负向fixtures |

## 技术说明

本 Story 为 `full-lld`；CP5正式证据路径为 `process/stories/STORY-ST-EI-002-dispatch-attestation-LLD.md`。当前卡片只冻结边界，不替代LLD。

## 验证上下文（validation_context）

采用 mixed（静态、单元、contract fixture、集成、回归、人工语义审查）。真实Codex runtime conformance只有Conditional-B前置全部满足时运行，否则记录NOT_RUN/UNAVAILABLE并按CP3-DQ-05限制CP7/CP8结论。

## 量化验收标准（acceptance_criteria）

- [ ] 适用 attempt terminal closure=100%
- [ ] D2提升为D0次数=0
- [ ] PC-18 freshness/re-probe覆盖 expiry/session/epoch/hash/schema/reload 6/6
- [ ] PC-19 无 reuse receipt 时 followup verified/model_attested 均为 false
- [ ] profile升级通过new spawn覆盖率=100%
- [ ] file_ownership.forbidden命中数=0，未授权runtime/credentials/publish操作=0。
- [ ] CP6 return/evidence与CP7独立结论可回链到全部AC。

## 阻塞说明

当前设计无CP4 blocker。平台REQUIRED extension尚非CURRENT，只限制runtime attestation，不阻塞A-baseline repository设计/实现。

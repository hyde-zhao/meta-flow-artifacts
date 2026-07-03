# SCOPE-Pack 状态

| 字段 | 值 |
|---|---|
| state_version | 1 |
| initialized_at | 2026-05-15 |
| orchestrator | meta-po |
| engagement_mode | production |
| scenario_subject_type | target-artifact |
| current_phase | story-execution |
| phase_status | CR-008-implemented-pending-review |
| active_change | CR-008 |
| workflow_mode | standard |
| fast_lane_reason | N/A |
| last_updated | 2026-06-25T10:34:00+0800 |

## 阶段状态

| 阶段 | 状态 | 证据 / 说明 |
|---|---|---|
| init | COMPLETED | 已创建 `process/`、`checkpoints/` 与初始运行态文件；按 `README.md` 修正后不使用 `delivery/` 作为交付目录。 |
| requirement-clarification | COMPLETED | `CR-001` 已完成需求阶段增量写入；CP2 人工确认已通过，`USE-CASES.md` 与 `REQUIREMENTS.md` 已确认。 |
| solution-design | COMPLETED | `process/HLD.md` v1.1 已确认，CP3 自动预检 PASS，`checkpoints/CP3-HLD-REVIEW.md` 人工确认 approved；`CR-002` 已关闭为 approved-and-baselined。 |
| story-planning | COMPLETED | `checkpoints/CP5-ALL-STORIES-LLD-BATCH.md` 已 approved；用户接受 U-001/U-002/U-003 默认建议和 meta-se D-003/D-004/D-005/D-006 裁决；6 个 LLD 均已回填 `confirmed=true`。 |
| story-execution | COMPLETED | `STORY-001`..`STORY-006` 全部 CP6 PASS、CP7 PASS，且 CP6/CP7 均含真实子 agent dispatch evidence；story-execution 退出条件已满足。 |
| documentation | COMPLETED | `process/checks/CP8-DELIVERY-READINESS.md` 自动预检 PASS；`checkpoints/CP8-DELIVERY-READINESS.md` 已 approved。 |
| delivered | COMPLETED | CP8 人工确认已通过；工作流已交付。 |
| CR-004 minimal loop | COMPLETED | 用户预授权跳过人工审批；宿主 `spawn_agent` 调度 `dev-you` 完成 CP6、`qa-zhang` 完成 CP7，CP8 已按 `approval_source=user-preauthorized` 自动回填，CR-004 已关闭为 `approved-and-delivered`。 |
| CR-005 init | COMPLETED | 2026-06-05T09:25:50+0800：CR-005 代码改动（tags + parameters_summary）已实现并交付。 |
| CR-006 init | COMPLETED | 2026-06-06T14:10:00+0800: CR-006 已交付。Operation 定义增加 aliases 同义字段，5 个 interface op 已填充首批 aliases。 |
| CR-007 init | COMPLETED | 2026-06-08T17:00:00+0800: CR-007 已交付。CLI 架构重构：run 命令拆分为 7 个嵌套子命令（auth/object/policy-route/interface/ospf/ospfv3），消除 80+ 参数扁平注册，每个叶子命令参数 ≤ 15 个。错误输出统一到 errors.py。 |
| CR-008 rollback contract | IMPLEMENTED_PENDING_REVIEW | 2026-06-25T10:34:00+0800: 已实现 rollback contract 基线。新增 schema 字段、rollback 规则模块、`validate --check rollback`、`rollback check`、list JSON 元数据、run `metadata.rollback_context`，并补齐策略路由、接口和流量首批 atom 契约。CP6/CP7 通过；等待用户审查。 |

## 工作目录状态

| 路径 | 状态 | 用途 |
|---|---|---|
| `process/` | READY | 运行态文档与状态文件。 |
| `process/checks/` | READY | 自动检查点结果（CP0/CP1/CP2 当前已生成）。 |
| `process/stories/` | READY | Story 卡片与 Story 级 LLD。 |
| `process/changes/` | READY | 需求或设计变更单。 |
| `checkpoints/` | READY | 人工确认稿。 |
| `atoms/` | DELIVERY_SURFACE | 原子操作 YAML 定义，路径为 `atoms/<device_type>/<op_id>.yaml`。 |
| `schemas/` | DELIVERY_SURFACE | atom YAML schema。 |
| `packages/` | DELIVERY_SURFACE | package 过滤视图。 |
| `docs/` | DELIVERY_SURFACE | 用户文档、工程师手册、字段参考、命名规范、错误码与模板。 |
| `src/atomic_ops/` | DELIVERY_SURFACE | `atomic-ops` CLI 源码。 |
| `scripts/` | DELIVERY_SURFACE | 仓库级检查与构建脚本。 |
| `pyproject.toml` / `uv.lock` | DELIVERY_SURFACE | Python 包依赖声明与锁定结果。 |

## 已确认事实

- 当前仓库项目名为 `atomic-ops`，定位是防火墙测试原子操作规范仓库与本地消费 CLI。
- Python 依赖以 `pyproject.toml` 与 `uv.lock` 为准，开发、测试和脚本执行应通过 `uv run`。
- `.input/` 是参考输入区，当前包含 `.input/capacity/` 与 `.input/ngfw-install/` 其他项目资料，不作为当前仓库源码直接修改。
- 初始化前未发现既有 `process/`、`checkpoints/`、`delivery/` 目录；`delivery/` 已确认不属于本项目 README 定义的交付面。
- 初始化前 `git status --short` 显示 `.gitignore` 已有未提交修改；本次初始化不回退该修改。

## 门控状态

| Gate | 状态 | 解除条件 |
|---|---|---|
| 需求确认 | PASSED | CP2 已通过，`process/USE-CASES.md` 与 `process/REQUIREMENTS.md` 已确认，`CR-001` 已关闭。 |
| HLD 确认 | PASSED | CP3 已通过，`process/HLD.md` v1.1 已 confirmed，`checkpoints/CP3-HLD-REVIEW.md` status=`approved`。 |
| Story 计划确认 | AUTO_PASSED_MANUAL_SUPERSEDED | CP4 自动预检 PASS；人工审查稿 `checkpoints/CP4-STORY-PLAN-REVIEW.md` 未 approved，已被 `process/changes/CR-003.md` 标记为 superseded / changes_requested。 |
| Story LLD 确认 | PASSED | `checkpoints/CP5-ALL-STORIES-LLD-BATCH.md` 已 approved；6 个 LLD 均已 `confirmed=true`；ADR 和 Platform spec 已按用户默认建议回填 `confirmed=true`。 |
| Story 执行 | PASSED | `STORY-001`..`STORY-006` 已 verified；全部目标 Story 的 CP6/CP7 均为 PASS 且含 dispatch evidence。 |
| 终验 | PASSED | CP8 自动预检 PASS；用户回复“通过，继续推进”，`checkpoints/CP8-DELIVERY-READINESS.md` 已回填 approved。 |

## 检查点状态

| Checkpoint | 类型 | 状态 | 证据 / 下一步 |
|---|---|---|---|
| CP0 Request Intake | 自动 | PASS | `process/checks/CP0-REQUEST-INTAKE.md` |
| CP1 Use Case Completeness | 自动 | PASS_WITH_OPEN_ITEMS | `process/checks/CP1-USE-CASE-COMPLETENESS.md` |
| CP2 Requirements Baseline | 自动预检 + 人工确认 | APPROVED | 自动预检见 `process/checks/CP2-REQUIREMENTS-BASELINE.md`；人工确认结果见 `checkpoints/CP2-REQUIREMENTS-BASELINE.md` |
| CP3 HLD Review | 自动预检 + 人工确认 + 子 Agent review gate | APPROVED | 自动预检见 `process/checks/CP3-HLD-CONSISTENCY.md`；人工审查稿见 `checkpoints/CP3-HLD-REVIEW.md`；`CR-002` 修订后 required findings 未关闭项为 0，用户已回复“通过”。 |
| CP4 Story Plan Review | 自动预检 + 人工确认 | AUTO_PASS_MANUAL_SUPERSEDED_BY_CR003 | 自动预检见 `process/checks/CP4-STORY-DAG-PARALLEL-SAFETY.md`，status=`PASS`；人工审查稿见 `checkpoints/CP4-STORY-PLAN-REVIEW.md`，status=`changes_requested`，未 approved，被 `process/changes/CR-003.md` 取代。 |
| CP5 All Stories LLD Batch | 全量自动预检 + 人工确认 | APPROVED | `checkpoints/CP5-ALL-STORIES-LLD-BATCH.md` 已 approved；用户接受 U-001/U-002 默认建议并已回填 ADR/Platform spec confirmed；用户接受 D-003/D-004/D-005/D-006 裁决。 |
| CP6 Story Coding Done | Story 级自动检查 | STORY-001-002-003-004-005-006_PASS | `STORY-001`..`STORY-006` CP6 均为 PASS，且对应 handoff 均有真实 dispatch evidence；`STORY-006` CP6 见 `process/checks/CP6-STORY-006-update-user-facing-docs-and-release-guidance-CODING-DONE.md`。 |
| CP6 CR-004 Adapter Profile Layer Coding Done | CR 级自动检查 | PASS | `process/checks/CP6-CR-004-adapter-profile-layer-CODING-DONE.md`；handoff 见 `process/handoffs/HANDOFF-20260519T122140-meta-po-to-meta-dev-CR-004-minimal-implementation.md`，`tool_name=spawn_agent`，`agent_name=dev-you`。 |
| CP7 CR-004 Adapter Profile Layer Verification Done | CR 级自动检查 | PASS | `process/checks/CP7-CR-004-adapter-profile-layer-VERIFICATION-DONE.md`；handoff 见 `process/handoffs/HANDOFF-20260519T122833-meta-po-to-meta-qa-CR-004-verification.md`，`tool_name=spawn_agent`，`agent_name=qa-zhang`。 |
| CP7 Story Verification Done | Story 级自动检查 | STORY-001-002-003-004-005-006_PASS | `STORY-001`..`STORY-006` 六个 CP7 文件均为 PASS，含 `qa-yan` / `qa-kong` subagent dispatch evidence；`STORY-006` CP7 见 `process/checks/CP7-STORY-006-update-user-facing-docs-and-release-guidance-VERIFICATION-DONE.md`。 |
| CP8 CR-004 Delivery Readiness | 自动预检 + 用户预授权终验 | APPROVED | 自动预检见 `process/checks/CP8-CR-004-DELIVERY-READINESS.md`；人工结果见 `checkpoints/CP8-CR-004-DELIVERY-READINESS.md`，`approval_source=user-preauthorized`。 |
| CP8 Delivery Readiness | 自动预检 + 人工确认 | APPROVED | 自动预检见 `process/checks/CP8-DELIVERY-READINESS.md`，status=`PASS`；人工审查稿见 `checkpoints/CP8-DELIVERY-READINESS.md`，status=`approved`。 |

## 编排器会话状态

| 字段 | 值 |
|---|---|
| role | meta-po |
| status | active |
| workflow_id | atomic-ops-aliases-field |
| active_change | none |
| agent_id | unavailable-current-api |
| thread_id | unavailable-current-api |
| pending_gate | none |
| pending_checklist_path |  |
| pending_user_decision | none |
| resume_instruction | CR-006 已交付。aliases 字段方案 C1 实施完成。 |
| awaiting_since |  |
| last_seen_at | 2026-06-06T14:20:00+0800 |

## Agent 生命周期登记

| 字段 | 值 |
|---|---|
| platform_capabilities.subagent_dispatch.available | true |
| platform_capabilities.subagent_dispatch.method | host-spawn-agent |
| platform_capabilities.subagent_dispatch.note | 宿主已使用 `spawn_agent` 真实调度 6 个 meta-dev LLD 子 agent，并通过 `send_input` 恢复 `dev-xu` 完成 STORY-006 F-006 修订；handoff 与 active_agents 已回填 agent_id / thread_id。 |
| platform_capabilities.subagent_dispatch.current_tool_surface | 本轮可用工具面未暴露 `spawn_agent` / `resume_agent` / `send_input`；meta-po 已创建 handoff，但不能将目标子 agent 标记为 completed。 |

| role | agent_id | agent_name | thread_id | workflow_id | story_id | wave_id | handoff_path | status | evidence | reusable | last_seen_at |
|---|---|---|---|---|---|---|---|---|---|---:|---|
| meta-se | 019e39ac-5f21-7063-a912-4afc2b0eb45f | Archimedes | 019e39ac-5f21-7063-a912-4afc2b0eb45f | atomic-ops-ngfw-install-config-chain |  |  | `process/handoffs/HANDOFF-20260518T140106-meta-po-to-meta-se-story-planning.md` | closed | host-dispatched-default-acting-role-meta-se | false | 2026-05-18T14:16:32+0800 |
| meta-dev | 019e39f6-6f53-75f2-9f0f-796c6cb92b08 | dev-yang | 019e39f6-6f53-75f2-9f0f-796c6cb92b08 | atomic-ops-ngfw-install-config-chain | STORY-001 | W1 | `process/handoffs/HANDOFF-20260518T151444-meta-po-to-meta-dev-STORY-001-lld.md` | completed | spawn_agent | false | 2026-05-18T15:26:58+0800 |
| meta-dev | 019e39f6-7759-7122-9216-871849ef788b | dev-zhang | 019e39f6-7759-7122-9216-871849ef788b | atomic-ops-ngfw-install-config-chain | STORY-002 | W1 | `process/handoffs/HANDOFF-20260518T151444-meta-po-to-meta-dev-STORY-002-lld.md` | completed | spawn_agent | false | 2026-05-18T15:26:58+0800 |
| meta-dev | 019e39f6-806b-75f1-b5c0-0b859d019ede | dev-shi | 019e39f6-806b-75f1-b5c0-0b859d019ede | atomic-ops-ngfw-install-config-chain | STORY-003 | W1 | `process/handoffs/HANDOFF-20260518T151444-meta-po-to-meta-dev-STORY-003-lld.md` | completed | spawn_agent | false | 2026-05-18T15:28:40+0800 |
| meta-dev | 019e39fc-797a-72d0-84d9-95f91f067ab7 | dev-kong | 019e39fc-797a-72d0-84d9-95f91f067ab7 | atomic-ops-ngfw-install-config-chain | STORY-004 | W2 | `process/handoffs/HANDOFF-20260518T152840-meta-po-to-meta-dev-STORY-004-lld.md` | completed | spawn_agent | false | 2026-05-18T15:34:50+0800 |
| meta-dev | 019e39fc-7b44-72d0-84d9-9619676e914c | dev-you | 019e39fc-7b44-72d0-84d9-9619676e914c | atomic-ops-ngfw-install-config-chain | STORY-005 | W2 | `process/handoffs/HANDOFF-20260518T152840-meta-po-to-meta-dev-STORY-005-lld.md` | completed | spawn_agent | false | 2026-05-18T15:34:50+0800 |
| meta-dev | 019e39fc-f3e8-7381-beae-85bef34273ab | dev-xu | 019e39fc-f3e8-7381-beae-85bef34273ab | atomic-ops-ngfw-install-config-chain | STORY-006 | W3 | `process/handoffs/HANDOFF-20260518T152840-meta-po-to-meta-dev-STORY-006-lld.md` | completed | spawn_agent | false | 2026-05-18T15:34:50+0800 |
| meta-dev | 019e39fc-f3e8-7381-beae-85bef34273ab | dev-xu | 019e39fc-f3e8-7381-beae-85bef34273ab | atomic-ops-ngfw-install-config-chain | STORY-006 | W3 | `process/handoffs/HANDOFF-20260518T161951-meta-po-to-meta-dev-STORY-006-lld-revise-F-006.md` | completed | resume_agent+send_input | false | 2026-05-18T16:25:14+0800 |
| meta-dev | 019e39f6-6f53-75f2-9f0f-796c6cb92b08 | dev-yang | 019e39f6-6f53-75f2-9f0f-796c6cb92b08 | atomic-ops-ngfw-install-config-chain | STORY-001 | W1 | `process/handoffs/HANDOFF-20260518T164738-meta-po-to-meta-dev-STORY-001-implementation.md` | completed | resume_agent+send_input | false | 2026-05-18T17:13:56+0800 |
| meta-dev | 019e39f6-7759-7122-9216-871849ef788b | dev-zhang | 019e39f6-7759-7122-9216-871849ef788b | atomic-ops-ngfw-install-config-chain | STORY-002 | W1 | `process/handoffs/HANDOFF-20260518T171519-meta-po-to-meta-dev-STORY-002-implementation.md` | completed | resume_agent+send_input | false | 2026-05-18T17:32:34+0800 |
| meta-dev | 019e39f6-806b-75f1-b5c0-0b859d019ede | dev-shi | 019e39f6-806b-75f1-b5c0-0b859d019ede | atomic-ops-ngfw-install-config-chain | STORY-003 | W1 | `process/handoffs/HANDOFF-20260518T171519-meta-po-to-meta-dev-STORY-003-implementation.md` | completed | resume_agent+send_input | false | 2026-05-18T17:32:34+0800 |
| meta-dev | 019e39fc-797a-72d0-84d9-95f91f067ab7 | dev-kong | 019e39fc-797a-72d0-84d9-95f91f067ab7 | atomic-ops-ngfw-install-config-chain | STORY-004 | W2 | `process/handoffs/HANDOFF-20260518T173351-meta-po-to-meta-dev-STORY-004-implementation.md` | completed | resume_agent+send_input | false | 2026-05-18T17:49:39+0800 |
| meta-dev | 019e39fc-7b44-72d0-84d9-9619676e914c | dev-you | 019e39fc-7b44-72d0-84d9-9619676e914c | atomic-ops-ngfw-install-config-chain | STORY-005 | W2 | `process/handoffs/HANDOFF-20260518T175231-meta-po-to-meta-dev-STORY-005-implementation.md` | completed | resume_agent+send_input | false | 2026-05-18T18:05:19+0800 |
| meta-qa | 019e3a91-a7c7-7920-814d-74c955d0ad70 | qa-yan | 019e3a91-a7c7-7920-814d-74c955d0ad70 | atomic-ops-ngfw-install-config-chain | STORY-001,STORY-002,STORY-003,STORY-004,STORY-005 | W1-W2 | `process/handoffs/HANDOFF-20260518T180625-meta-po-to-meta-qa-CP7-STORY-001-005-verification.md` | completed | host-spawned-meta-qa-for-CP7-STORY-001-005 | true | 2026-05-18T18:20:21+0800 |
| meta-qa | 019e3a91-a7c7-7920-814d-74c955d0ad70 | qa-yan | 019e3a91-a7c7-7920-814d-74c955d0ad70 | atomic-ops-ngfw-install-config-chain | STORY-001,STORY-002,STORY-003,STORY-004,STORY-005 | W1-W2 | `process/handoffs/HANDOFF-20260518T182147-meta-po-to-meta-qa-CP7-STORY-001-005-revision.md` | completed | host-resumed-existing-meta-qa-for-CP7-blocker-revision | false | 2026-05-18T18:30:31+0800 |
| meta-dev | 019e39fc-f3e8-7381-beae-85bef34273ab | dev-xu | 019e39fc-f3e8-7381-beae-85bef34273ab | atomic-ops-ngfw-install-config-chain | STORY-006 | W3 | `process/handoffs/HANDOFF-20260518T183239-meta-po-to-meta-dev-STORY-006-implementation.md` | completed | host-resumed-existing-meta-dev-for-STORY-006-implementation | false | 2026-05-19T08:55:24+0800 |
| meta-qa | 019e3dc0-2f13-7023-8b53-5d25cbd95a05 | qa-kong | 019e3dc0-2f13-7023-8b53-5d25cbd95a05 | atomic-ops-ngfw-install-config-chain | STORY-006 | W3 | `process/handoffs/HANDOFF-20260519T085651-meta-po-to-meta-qa-STORY-006-verification.md` | completed | host-spawned-new-meta-qa-for-STORY-006-CP7 | false | 2026-05-19T09:06:45+0800 |
| meta-doc | 019e3dca-2c35-7fc1-bc97-ed5c3db3af13 | doc-cao | 019e3dca-2c35-7fc1-bc97-ed5c3db3af13 | atomic-ops-ngfw-install-config-chain | ALL-STORIES | documentation | `process/handoffs/HANDOFF-20260519T090828-meta-po-to-meta-doc-documentation-cp8.md` | completed | host-spawned-meta-doc-for-CP8-documentation-readiness | false | 2026-05-19T09:18:00+0800 |
| meta-pm |  |  |  | atomic-ops-ngfw-install-config-chain |  | requirement-clarification | `process/handoffs/HANDOFF-20260519T121233-meta-po-to-meta-pm-cr004-requirements.md` | closed | superseded-by-CR004-minimal-loop-user-preauthorized | false | 2026-05-19T12:32:34+0800 |
| meta-dev | 019e3e7a-24c2-7ad1-8707-5e66eae0f69b | dev-you | 019e3e7a-24c2-7ad1-8707-5e66eae0f69b | atomic-ops-ngfw-install-config-chain | CR004-MINIMAL | CR-004 | `process/handoffs/HANDOFF-20260519T122140-meta-po-to-meta-dev-CR-004-minimal-implementation.md` | completed | spawn_agent | false | 2026-05-19T12:28:16+0800 |
| meta-qa | 019e3e80-5c2e-7072-ae15-bfc91e1a6aa4 | qa-zhang | 019e3e80-5c2e-7072-ae15-bfc91e1a6aa4 | atomic-ops-ngfw-install-config-chain | CR004-MINIMAL | CR-004 | `process/handoffs/HANDOFF-20260519T122833-meta-po-to-meta-qa-CR-004-verification.md` | completed | spawn_agent | false | 2026-05-19T12:32:37+0800 |

## 并行执行状态

| 队列 / 字段 | Story | 状态 / 说明 |
|---|---|---|
| lld_design_batch | `STORY-001`, `STORY-002`, `STORY-003`, `STORY-004`, `STORY-005`, `STORY-006` | `CR-003-LLD-BATCH`，覆盖全部目标 Story；LLD 输出已完成。 |
| next_lld_dispatch_window | 无 | 全部 6 个 LLD 子 agent 已完成并关闭。 |
| lld_ready | 无 | 全部目标 Story 均已输出 LLD。 |
| lld_running | 无 | 全部 6 个 LLD 子 agent 已完成并关闭。 |
| lld_review | 无 | 已收敛到全量批次人工确认队列。 |
| lld_batch_review | 无 | CP5 已 approved。 |
| lld_revision_required | 无 | STORY-006 F-006 wording 已关闭。 |
| dev_ready | 无 | 全部目标 Story 已完成实现。 |
| dev_running | 无 | 无开发任务运行中。 |
| verify_ready | 无 | 全部目标 Story 已完成 CP7。 |
| verify_running | 无 | 无 QA 任务运行中。 |
| documentation_ready | 无 | CP8 自动预检已完成。 |
| documentation_running | 无 | `doc-cao` 已完成 CP8 自动预检并关闭。 |
| documentation_review | 无 | CP8 人工确认已 approved。 |
| delivered | `ALL-STORIES` | `checkpoints/CP8-DELIVERY-READINESS.md` 已 approved；当前工作流已进入 delivered。 |
| cr004_lld_design_batch | `CR004-MINIMAL` | 用户预授权 CR-004 最小闭环；以单一最小实现 / 验证批次替代完整回放。 |
| cr004_dispatch_blocked | 无 | 已由宿主 `spawn_agent` 完成 meta-dev / meta-qa 调度证据回填。 |
| blocked_by_dependency | 无 | Story 依赖均已解除；无剩余阻断项。 |
| lld_pending_next_window | 无 | 全部目标 Story 均已完成 LLD 起草。 |

## 当前编排决策

- 历史记录：早期 `meta-po` 曾完成对应轮次编排分析并关闭；当前轮次为 CR-003 编排，状态见上方“编排器会话状态”。
- 2026-05-18T10:57:59+0800：当前 API 工具面未暴露自定义 `agent_type=meta-po`；本线程以 `agent_type=default, acting_role=meta-po` 恢复编排，只执行状态核验、最小状态回写与下一跳 handoff 准备，不代写 HLD、ADR 或 Story 计划。
- `meta-pm` 已完成 `CR-001` 的本轮需求写入：已增量修订 `process/USE-CASES.md`、`process/REQUIREMENTS.md` 与 `process/CLARIFICATION-LOG.md`，并已通过 CP2 人工确认。
- `meta-se` 曾被错误唤醒，已关闭且未写出 `process/HLD.md`。
- `USE-CASES.md` 与 `REQUIREMENTS.md` 已确认；下一步允许启动 `meta-se`，但不得跳过 HLD 评审。
- 同一阶段核心对象遵循单写规则：`meta-pm` 负责 `USE-CASES.md`、`REQUIREMENTS.md` 与澄清日志，完成后关闭。
- `.input/` 仅作为参考输入，不直接复制为当前交付物。
- `process/changes/CR-001.md` 已关闭；后续需求变更必须创建新的 CR。
- `meta-po` 已补齐 CP0/CP1/CP2 检查点文档；CP2 已通过，当前可进入 `solution-design`。
- 2026-05-18T11:00:00+0800：已由宿主拉起 `agent_type=default, acting_role=meta-se`，平台返回 `agent_id=019e3906-9216-7ad3-a198-1ad71d51fc00`、`agent_name=Rawls`；handoff 记录见 `process/handoffs/HANDOFF-20260518T110000-meta-po-to-meta-se-solution-design.md`。由于当前 API 未暴露原生 `agent_type=meta-se`，调度证据必须按 `default acting_role=meta-se` 记录。
- 2026-05-18T11:01:06+0800：`agent_type=default, acting_role=meta-se` 已完成 `process/HLD.md` 与 `process/checks/CP3-HLD-CONSISTENCY.md`，未生成独立 ADR、Story backlog、开发计划或 Story 文件。
- 2026-05-18T11:07:44+0800：当前线程以 `agent_type=default, acting_role=meta-po` 创建 `checkpoints/CP3-HLD-REVIEW.md` 并发起 CP3 人工确认；等待用户回复 `approve`、`修改: <具体修改点>` 或 `reject`。CP3 人工确认通过前继续禁止 ADR、Story planning 和任何 Story 文件。
- 2026-05-18T11:37:30+0800：当前线程以 `agent_type=default, acting_role=meta-po` 聚合 CP3 子 Agent 评审，生成 `process/reviews/CP3-HLD-review-summary.md`。`meta-dev` review 为 blocking=0、required=3、optional=1，建议 revise-and-resubmit；`meta-qa` review 为 blocking=0、required=0、optional=1，场景覆盖 6/6、需求覆盖 22/22，建议 approve。聚合决策为 `revise`，当前不建议 CP3 approve。
- 2026-05-18T11:47:32+0800：用户要求关闭不需要的子 agent 并唤醒 `meta-se` 修订 HLD；已确认旧 `meta-dev`、`meta-qa`、旧 `meta-se` 线程不存在或已关闭。已创建 `process/changes/CR-002.md`，修订范围包括 3 个 meta-dev required findings、多防火墙设备批量配置、命令命名规范、参数校验和 atomic 规范说明。
- 2026-05-18T11:55:10+0800：当前线程以 `agent_type=default, acting_role=meta-po` 确认 `CR-002` HLD 修订完成；`process/HLD.md` 为 v1.1、`confirmed=false`，`process/checks/CP3-HLD-CONSISTENCY.md` status=`PASS` 且 required findings 未关闭项为 0。已更新 `checkpoints/CP3-HLD-REVIEW.md`，等待用户重新审查并回复 `approve`、`修改: <具体修改点>` 或 `reject`。
- 2026-05-18T13:55:13+0800：用户回复“通过，唤醒meta-se，插接stroy后，拉起子agent并行开展lld的设计。” 当前线程以 `agent_type=default, acting_role=meta-po` 回填 CP3 人工确认 approved，确认 `process/HLD.md` v1.1，关闭 `CR-002` 为 approved-and-baselined，并推进到 `story-planning` ready。因 CP4 Story plan 自动预检和人工确认尚未完成，暂不得拉起 LLD 设计子 agent。
- 2026-05-18T14:01:06+0800：已由宿主拉起 `agent_type=default, acting_role=meta-se`，平台返回 `agent_id=019e39ac-5f21-7063-a912-4afc2b0eb45f`、`agent_name=Archimedes`；handoff 记录见 `process/handoffs/HANDOFF-20260518T140106-meta-po-to-meta-se-story-planning.md`。由于当前 API 未暴露原生 `agent_type=meta-se`，调度证据按 `default acting_role=meta-se` 记录。CP4 Story Plan 自动预检和人工确认未通过前，不得拉起 LLD 设计子 agent。
- 2026-05-18T14:16:32+0800：`agent_type=default, acting_role=meta-se`（agent_id=`019e39ac-5f21-7063-a912-4afc2b0eb45f`，nickname=`Archimedes`）已完成并关闭；`process/checks/CP4-STORY-DAG-PARALLEL-SAFETY.md` status=`PASS`，阻断项 0，豁免项 0。当前线程以 `agent_type=default, acting_role=meta-po` 创建 `checkpoints/CP4-STORY-PLAN-REVIEW.md` 并进入 CP4 人工确认等待态；CP4 approved 前不得启动 LLD，不得拉起 meta-dev/meta-qa。
- 2026-05-18T15:05:18+0800：当前线程以 `agent_type=default, acting_role=meta-po` 复核 CP4 与 Story 计划事实。`checkpoints/CP4-STORY-PLAN-REVIEW.md` 仍为 `status=pending`，本轮用户请求“拉起meta-po，分析当前项目状态，组织子agent并行推进项目”不能按 exact 文本协议视为 CP4 `approve`。正式目标 Story 为 `STORY-001`..`STORY-006` 共 6 个，`LLD-BATCH-001 = STORY-001, STORY-002, STORY-003` 仅为 CP4 approved 后的首批调度窗口候选。当前 API 工具面未暴露 `spawn_agent` / `resume_agent` / `send_input`，且 CP4 未 approved，未创建任何 meta-dev handoff，未启动 LLD。
- 2026-05-18T15:14:44+0800：用户明确变更门控顺序：Story 拆解完成后先组织全部目标 Story LLD，全部 LLD + CP5 自动预检完成后再人工统一确认。当前线程创建 `process/changes/CR-003.md`，将 `checkpoints/CP4-STORY-PLAN-REVIEW.md` 标记为 `changes_requested / superseded_by_CR-003`，不写为 approved。已更新 `process/STORY-BACKLOG.md`、`process/DEVELOPMENT-PLAN.yaml`、`process/STORY-STATUS.md` 和 6 张 Story 卡进入 `lld-ready`。当前 API 工具面无 `spawn_agent`，已生成首批 `STORY-001`、`STORY-002`、`STORY-003` 的 meta-dev handoff，状态为 `spawn-required`，等待宿主真实调度。
- 2026-05-18T15:22:03+0800：宿主按 CR-003 首批 LLD handoff 使用 `spawn_agent` 真实调度 3 个 `meta-dev`：`STORY-001` 为 `dev-yang`（agent_id=`019e39f6-6f53-75f2-9f0f-796c6cb92b08`），`STORY-002` 为 `dev-zhang`（agent_id=`019e39f6-7759-7122-9216-871849ef788b`），`STORY-003` 为 `dev-shi`（agent_id=`019e39f6-806b-75f1-b5c0-0b859d019ede`）。对应 handoff dispatch 证据已回填为 `spawn_agent`，当前只允许输出 LLD，不得实现代码或进入 CP6/CP7。
- 2026-05-18T15:26:58+0800：首批 LLD 中 `STORY-001` 与 `STORY-002` 已完成，分别输出 `process/stories/STORY-001-freeze-schema-v11-contract-and-field-docs-LLD.md` 与 `process/stories/STORY-002-model-ngfw-install-init-login-guard-atoms-LLD.md`；两者均为 `confirmed=false` 且带 OPEN 项，等待 CP5 自动预检和批量人工确认。`STORY-003` 仍在运行。
- 2026-05-18T15:28:40+0800：`STORY-003` LLD 已完成，输出 `process/stories/STORY-003-model-capacity-ten-domain-config-and-verification-atoms-LLD.md`；首批三个完成 agent 已关闭。宿主继续按 CR-003 调度第二批 LLD：`STORY-004` 为 `dev-kong`（agent_id=`019e39fc-797a-72d0-84d9-95f91f067ab7`），`STORY-005` 为 `dev-you`（agent_id=`019e39fc-7b44-72d0-84d9-9619676e914c`），`STORY-006` 为 `dev-xu`（agent_id=`019e39fc-f3e8-7381-beae-85bef34273ab`）。对应 handoff dispatch 证据已回填为 `spawn_agent`。
- 2026-05-18T15:34:50+0800：第二批 LLD 全部完成并关闭：`STORY-004` 输出 `process/stories/STORY-004-model-multi-device-batch-configuration-contract-LLD.md`，`STORY-005` 输出 `process/stories/STORY-005-add-read-only-security-gate-and-validation-checks-LLD.md`，`STORY-006` 输出 `process/stories/STORY-006-update-user-facing-docs-and-release-guidance-LLD.md`。全部 6 个目标 Story LLD 均已生成，下一步进入 CP5 自动预检与批量人工确认准备。
- 2026-05-18T15:43:59+0800：当前线程以 `agent_type=default, acting_role=meta-po` 完成 `STORY-001`..`STORY-006` 的 CP5 自动预检，6 个结果均为 `PASS` 且带 OPEN / 实现前门禁；已生成 `checkpoints/CP5-ALL-STORIES-LLD-BATCH.md` 并发起 CP5 全量 LLD 批量人工确认。`process/ARCHITECTURE-DECISION.md` 和 `process/PLATFORM-INSTALL-SPEC.md` 的 `confirmed=false` 未被改写，作为 CP5 OPEN / 风险接受项提交人工确认。CP5 approved 前继续禁止实现、禁止 story-execution、禁止 CP6/CP7。
- 2026-05-18T16:19:51+0800：当前线程以 `agent_type=default, acting_role=meta-po` 读取 `process/reviews/CP5-LLD-meta-se-architecture-review.md` 并处理用户“按 meta-se 评审意见修改”的指令。F-001/F-002 分类为确认态 / CP5 用户决策项，不交给 meta-dev；F-003/F-004/F-005 分类为架构裁决 / 后续 dev gate，已写入 CP5 批量稿；F-006 需要 `STORY-006` LLD wording 修订。已生成 `process/handoffs/HANDOFF-20260518T161951-meta-po-to-meta-dev-STORY-006-lld-revise-F-006.md`，当前 API 工具面未暴露 `spawn_agent` / `resume_agent`，等待宿主真实调度。CP5 状态为 `changes_requested`，仍禁止实现、story-execution、CP6/CP7。
- 2026-05-18T16:26:06+0800：宿主已真实恢复 `dev-xu` 并完成 `STORY-006` F-006 修订，handoff 证据为 `dispatch.tool_name=send_input`、`dispatch.evidence=resume_agent+send_input`、`completed_at=2026-05-18T16:25:14+0800`。当前线程确认 STORY-006 LLD 不再声称只读取到 STORY-004/005 Story 卡，已改为 `STORY-004/005 LLD 尚未 confirmed，最终实现与验证事实待定`；已更新 STORY-006 CP5 自动预检并将 CP5 批量稿恢复为 pending 人工审查。CP5 approved 前继续禁止实现、story-execution、CP6/CP7。
- 2026-05-18T16:47:38+0800：用户回复“全部接受。恢复meta-po。并行拉起meta-dev推进项目。” 当前线程按 CP5 人工确认 approved 处理：回填 `checkpoints/CP5-ALL-STORIES-LLD-BATCH.md` approved；按 U-001/U-002 默认建议回填 `process/ARCHITECTURE-DECISION.md` 与 `process/PLATFORM-INSTALL-SPEC.md` `confirmed=true`；6 个 LLD 均回填 `confirmed=true`。已推进到 `story-execution`，按 `DEVELOPMENT-PLAN.yaml`、依赖类型、文件所有权和 `max_parallel_dev=2` 计算首批仅 `STORY-001` dev_ready。已生成 `process/handoffs/HANDOFF-20260518T164738-meta-po-to-meta-dev-STORY-001-implementation.md`；当前 API 工具面未暴露 `spawn_agent` / `resume_agent`，等待宿主真实调度。
- 2026-05-18T17:15:19+0800：宿主已真实恢复 `dev-yang` 完成 STORY-001 实现，并回填 implementation handoff dispatch evidence。当前线程复核 `process/checks/CP6-STORY-001-freeze-schema-v11-contract-and-field-docs-CODING-DONE.md` 为 `PASS` 且含 Agent Dispatch Evidence；STORY-001 schema/docs contract frozen，进入 `ready-for-verification`。按 `max_parallel_dev=2`、依赖类型和文件所有权复算，`STORY-002` 与 `STORY-003` primary 文件无交叠且 contract dependency 已满足，已生成两个 implementation handoff，等待宿主真实调度；本轮不启动 meta-qa。
- 2026-05-18T17:33:51+0800：宿主已并行恢复 `dev-zhang` / `dev-shi` 完成 STORY-002 / STORY-003 实现，并回填 implementation handoff dispatch evidence。当前线程复核两个 CP6 均为 `PASS` 且含 Agent Dispatch Evidence；STORY-002 install chain 与 STORY-003 10-domain contract 均稳定，二者进入 `ready-for-verification`。按依赖与文件所有权复算，`STORY-004` contract dependency 已满足且 primary 文件无冲突，已生成 implementation handoff；`STORY-005` 仍等待 STORY-004 batch contract，`STORY-006` 仍 runtime blocked。本轮不启动 meta-qa。
- 2026-05-18T17:52:31+0800：宿主已恢复 `dev-kong` 完成 STORY-004 实现，并回填 implementation handoff dispatch evidence。当前线程复核 STORY-004 CP6 为 `PASS` 且含 Agent Dispatch Evidence；STORY-004 batch contract facts 稳定，进入 `ready-for-verification`。按依赖与文件所有权复算，`STORY-005` contract dependency 已满足且 primary 文件无冲突，已生成 implementation handoff；`STORY-006` 仍 runtime blocked。本轮不启动 meta-qa。
- 2026-05-18T18:06:25+0800：宿主已恢复 `dev-you` 完成 STORY-005 实现，并回填 implementation handoff dispatch evidence。当前线程复核 STORY-005 CP6 为 `PASS` 且含 Agent Dispatch Evidence；security gate facts 稳定，进入 `ready-for-verification`。按 STORY-006 runtime gate 复算，STORY-001..005 实现事实已具备但 CP7 验证事实仍缺失，STORY-006 继续 blocked。已生成 CP7 batch QA handoff，等待宿主真实调度 meta-qa。
- 2026-05-18T18:21:47+0800：宿主已 spawn `qa-yan` 完成 CP7 batch 功能验证，并回填 handoff `completed_at=2026-05-18T18:20:21+0800`。当前线程确认 QA 功能验证 PASS，但 CP7 文件仍因缺 `process/VALIDATION-ENV.yaml` 与历史 dispatch completed_at 缺口而 BLOCKED。由于本批验证仅为离线本地只读检查，不需要用户 secrets、真实防火墙或外部服务，meta-po 已创建 `process/VALIDATION-ENV.yaml` 且 `approval.confirmed=true`、`user_confirmation_required=false`。下一步恢复 `qa-yan` 修订 CP7；STORY-006 仍 blocked。
- 2026-05-18T18:32:39+0800：宿主已恢复 `qa-yan` 完成 CP7 blocker revision，并回填 revision handoff `completed_at=2026-05-18T18:30:31+0800`。当前线程复核 `STORY-001`..`STORY-005` 五个 CP7 文件均为 `PASS`，且含原始 `spawn_agent` 与修订 `resume_agent+send_input` 的 meta-qa dispatch evidence；`process/VERIFICATION-REPORT.md` status=`PASS`，QA-BLOCK-001/002 resolved。已将 `STORY-001`..`STORY-005` 标记为 verified，解除 `STORY-006` runtime gate，并生成 `process/handoffs/HANDOFF-20260518T183239-meta-po-to-meta-dev-STORY-006-implementation.md`，等待宿主恢复既有 `dev-xu`。
- 2026-05-19T08:56:51+0800：宿主已恢复 `dev-xu` 完成 STORY-006 实现，并回填 implementation handoff `status=completed`、`completed_at=2026-05-19T08:55:24+0800`。当前线程复核 `process/checks/CP6-STORY-006-update-user-facing-docs-and-release-guidance-CODING-DONE.md` 为 `PASS`，handoff 具备 `resume_agent+send_input`、`agent_id/thread_id=019e39fc-f3e8-7381-beae-85bef34273ab` 和 completed_at。已将 STORY-006 更新为 `ready-for-verification`，并生成 `process/handoffs/HANDOFF-20260519T085651-meta-po-to-meta-qa-STORY-006-verification.md`。按 exact reuse key，既有 `qa-yan` 是 STORY-001..005/W1-W2 CP7，不默认复用；下一步宿主 spawn 新 `meta-qa` 执行 STORY-006 CP7。
- 2026-05-19T09:08:28+0800：宿主已 spawn `qa-kong` 完成 STORY-006 CP7，并回填 handoff `status=completed`、`completed_at=2026-05-19T09:06:45+0800`。当前线程复核 `process/checks/CP7-STORY-006-update-user-facing-docs-and-release-guidance-VERIFICATION-DONE.md` 为 `PASS` 且含 `spawn_agent` / `agent_id=019e3dc0-2f13-7023-8b53-5d25cbd95a05` / `thread_id=019e3dc0-2f13-7023-8b53-5d25cbd95a05` dispatch evidence。已将 STORY-006 标记为 verified；`STORY-001`..`STORY-006` 全部 verified，story-execution 退出条件满足。已推进到 documentation dispatch-ready，并生成 `process/handoffs/HANDOFF-20260519T090828-meta-po-to-meta-doc-documentation-cp8.md`，等待宿主 spawn meta-doc。

## 当前 CP8 收敛记录

- 2026-05-19T09:18:42+0800：宿主已 spawn `doc-cao` 完成 CP8 delivery readiness 自动预检，并回填 handoff `status=completed`、`completed_at=2026-05-19T09:18:00+0800`。当前线程复核 `process/checks/CP8-DELIVERY-READINESS.md` 为 `PASS` 且含 `spawn_agent` / `agent_id=019e3dca-2c35-7fc1-bc97-ed5c3db3af13` / `thread_id=019e3dca-2c35-7fc1-bc97-ed5c3db3af13` dispatch evidence。已创建 `checkpoints/CP8-DELIVERY-READINESS.md` 人工审查稿；当前等待用户回复 `approve`、`修改: <具体修改点>` 或 `reject`，未标记 delivered。
- 2026-05-19T10:27:58+0800：用户回复“通过，继续推进”，按本轮指令等价为 CP8 manual review approval。当前线程重新复核 CP8 自动预检 PASS 与 `doc-cao` dispatch evidence 后，已回填 `checkpoints/CP8-DELIVERY-READINESS.md` status=`approved`，接受 `CP8-RISK-001` / `CP8-RISK-002` 为非阻断风险，并将 workflow 推进到 `delivered`。

## 下一步

CR-005 fast-lane 工作流进行中。当前阶段：`init`。等待 CP0 自动检查通过后推进到实现阶段。因 fast-lane，CP2/CP3 可合并为 Intent + Approach Brief。

## CR-005 编排记录

- 2026-06-05T09:25:50+0800：当前线程以 `agent_type=default, acting_role=meta-po` 启动 CR-005 fast-lane 工作流。目标：为 `atomic-ops list` 输出增加 `tags` 和 `parameters_summary` 字段。改动范围：`repository.py` + `list_ops.py`，~25 行代码。无架构/权限/安装/外部接口变更。

## CR-004 编排记录

- 2026-05-19T12:12:33+0800：当前线程以 `agent_type=default, acting_role=meta-po` 读取 `process/STATE.md`、`process/changes/CR-004.md`、Story backlog 与开发计划后，确认 CR-004 影响需求层、场景层、计划层、安全层和交付层；影响级别保持 `high`。因 CR 明确影响 `USE-CASES.md` 与 `REQUIREMENTS.md`，`rollback_to` 修正为 `requirement-clarification`。
- 2026-05-19T12:12:33+0800：用户本轮明确授权“这个需求比较简单，可以跳过人工审批，自动推行”。已记录到 `process/changes/CR-004.md` 和 `checkpoints/CR-004-AUTO-PROCEED-AUTHORIZATION.md`；该授权只跳过人工审批，不豁免 CP6 / CP7 的真实子 agent dispatch evidence。
- 2026-05-19T12:12:33+0800：当前工具面未暴露 `spawn_agent` / `resume_agent` / `send_input`。已创建 `process/handoffs/HANDOFF-20260519T121233-meta-po-to-meta-pm-cr004-requirements.md`，状态为 `handoff-created`；不得写成 `meta-pm` 已完成。
- 2026-05-19T12:28:16+0800：宿主以 `spawn_agent` 调度 `meta-dev`（`agent_name=dev-you`，`agent_id=019e3e7a-24c2-7ad1-8707-5e66eae0f69b`）完成 CR-004 minimal implementation；已新增 / 更新 adapter profile schema、NGFW default profile、`fw_config_policy_route` adapter ref、adapter validator、最小文档和 CP6 检查结果。`process/checks/CP6-CR-004-adapter-profile-layer-CODING-DONE.md` 结论为 PASS，下一步交给 meta-qa 执行 CP7。
- 2026-05-19T12:32:37+0800：宿主以 `spawn_agent` 调度 `meta-qa`（`agent_name=qa-zhang`，`agent_id=019e3e80-5c2e-7072-ae15-bfc91e1a6aa4`）完成 CR-004 minimal verification；已运行 schema、adapter profile、negative sensitive adapter profile、layout、安全门禁和 CLI help 验证。`process/checks/CP7-CR-004-adapter-profile-layer-VERIFICATION-DONE.md` 与 `process/checks/CP7-CR-004-ADAPTER-PROFILE-VERIFICATION-DONE.md` 结论均为 PASS；`.input/capacity` 不作为验证必需输入且 `.input/` 未修改。
- 2026-05-19T12:32:34+0800：meta-po 按用户预授权执行 CR-004 自动终验；`process/checks/CP8-CR-004-DELIVERY-READINESS.md` PASS，`checkpoints/CP8-CR-004-DELIVERY-READINESS.md` 已回填 approved / `approval_source=user-preauthorized`。`process/changes/CR-004.md` 已关闭为 `approved-and-delivered`，`active_change` 清空为 `none`，当前阶段恢复 `delivered`。

## delivery_routing

归档路由（2026-07-03 整改）：交付文件留本仓推 GitLab，过程文件归档到 sibling `meta-flow-artifacts` 仓并经符号链接回挂。

| 类别 | 本仓路径 | 归档目标仓 | 归档路径 / 方式 |
|---|---|---|---|
| 交付文件 | `atoms/`、`src/`、`docs/`、`packages/`、`schemas/`、`scripts/`、`tests/`、`adapters/`、`pyproject.toml`、`uv.lock`、`README.md`、`CLAUDE.md`、`AGENTS.md`、`CHANGELOG.md` | GitLab `git@10.113.53.21:tgfw/tgfw-ptm/ptm-atomic.git` | 直接 git 跟踪 |
| 过程文件 | `process/`（符号链接） | GitHub `hyde-zhao/meta-flow-artifacts.git` | `process/ptm-atomic/`（symlink → `../meta-flow-artifacts/process/ptm-atomic`） |
| 遗留 checkpoints | （根目录已删除） | GitHub `hyde-zhao/meta-flow-artifacts.git` | `process/ptm-atomic/legacy-root-checkpoints/`（7 个 pre-migration 文件归档） |
| 外部参考 | `.input/`（86M，capacity / ngfw-install 其他项目源码） | 不归档 | 本地参考，git 忽略 |
| 安装清单 | `.ptm-team-manifest.json` | 不归档 | 本地安装状态，git 忽略 |

约定：`process/ptm-atomic/.meta-flow-process.yaml` 声明 `routing_mode: symlink`，与 `meta-flow`、`ptm-team`、`quant-lab` 项目一致。

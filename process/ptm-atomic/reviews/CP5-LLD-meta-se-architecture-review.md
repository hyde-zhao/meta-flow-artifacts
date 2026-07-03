---
artifact: "process/reviews/CP5-LLD-meta-se-architecture-review.md"
reviewer: "meta-se"
lane: "lane-architecture"
round: 1
status: "completed"
governance_mode: "review-gated"
review_target: "CP5-ALL-STORIES-LLD-BATCH"
active_change: "CR-003"
created_at: "2026-05-18T16:20:00+0800"
---

# Review Findings

## 1. 审查范围

- 目标对象：`checkpoints/CP5-ALL-STORIES-LLD-BATCH.md` 与 `STORY-001`..`STORY-006` 六份 LLD。
- 审查目标：架构一致性、HLD/ADR/CR-003/Story plan 契约一致性、OPEN / BLOCKED_FOR_IMPLEMENTATION 项收敛、CP5 人工确认建议。
- 审查依据：`process/HLD.md` v1.1 confirmed、`process/ARCHITECTURE-DECISION.md`、`process/PLATFORM-INSTALL-SPEC.md`、`process/changes/CR-003.md`、`process/STORY-BACKLOG.md`、`process/DEVELOPMENT-PLAN.yaml`、`process/STORY-STATUS.md`、6 个 CP5 自动预检、6 份 Story LLD、`checkpoints/CP5-ALL-STORIES-LLD-BATCH.md`。
- 审查边界：本评审不 approve CP5、不回填人工确认、不修改产品文件、不进入实现、不执行产品验证。

## 总体结论

架构评审结论：`revise-before-approval-or-approve-with-explicit-risk-acceptance`。

六份 LLD 与 HLD v1.1、CR-003 新门控、Story Backlog 和 Development Plan 的主架构方向一致：所有 LLD 均保持 `confirmed=false`，未要求提前实现；文件所有权、Wave、contract/runtime 依赖和 forbidden 范围基本自洽；`STORY-006` 被正确建模为 runtime 收口，不能在 `STORY-001`..`STORY-005` 实现与验证事实完成前开发。

未发现需要 reject 整批 LLD 的架构阻断项。存在 2 个治理状态问题必须在用户 CP5 决策中显式处理：`ARCHITECTURE-DECISION.md confirmed=false` 被 6 个 LLD 消费，`PLATFORM-INSTALL-SPEC.md confirmed=false` 被 `STORY-006` 消费。这两个问题不要求重写 LLD，但不能由 meta-se 代替用户完成确认或回填。

建议 CP5 不直接无条件 approve。推荐用户选择：`修改: 接受 meta-se 架构裁决，并要求 meta-po 在 CP5 approval 回填 ADR/Platform spec confirmed=true 后再计算 dev_ready`。若用户愿意把这两个 frontmatter 状态作为 CP5 风险接受项，也可以 approve，但 meta-po 后续必须把风险接受写入 CP5 人工结果或状态记录，避免 meta-dev ready-check 再次阻断。

## 逐 Story Finding

| Story | 结论 | Finding | 影响 | 建议 |
|---|---|---|---|---|
| STORY-001 | 架构可接受，需 CP5 处理 ADR 状态 | schema v1.1 字段族覆盖 `risk/credential_ref/session_ref/state_ref/gate/verification/batch`，兼容 v1.0、`additionalProperties: false`、敏感互斥、gate 和 batch 边界均有设计；`schema_version="1.1"` 与现有 schema 正则冲突被 E-02 正确建为实现阻断。 | 可作为后续 Story contract 输入；实现仍需先冻结本 Story。 | CP5 可接受为强输入；若实现发现版本正则冲突，必须停下请求决策，不得静默改 `"1.1.0"`。 |
| STORY-002 | 架构可接受 | 5 个安装链路 atom 与 `ngfw_installation` package 与 HLD 安装、初始化、登录、守卫、健康检查边界一致；health check 留在安装链路 package 中合理。 | 依赖 STORY-001 schema/gate；依赖 STORY-005 最终安全 gate 做批次回归。 | CP5 可接受；实现前不得绕过 STORY-001 contract 和 security gate。 |
| STORY-003 | 架构可接受，package 范围需裁决 | 10 个 capacity 配置域、20 个 atom、`ngfw_capacity_config` package 与 HLD/ADR-004 一致；`ngfw_verification` package 默认 10 个 capacity 验证 op_id，CP5 留出是否纳入 STORY-002 health check 的问题。 | 若把 `fw_verify_ngfw_health` 纳入 `ngfw_verification`，会形成 STORY-003 package 对 STORY-002 op_id 的跨 Story 引用和包语义扩大。 | meta-se 裁决：`ngfw_verification` 默认只含 10 个 capacity 验证 op_id；安装健康检查保留在 `ngfw_installation`。 |
| STORY-004 | 架构可接受 | 多设备批次配置独立 Story，覆盖 `device_inventory_ref`、selector、`batch.max_concurrency` 默认 1 最大 5、幂等键、`per_device_results[]`、`failed_devices[]`、`partial_failed` 和验证汇总。 | 依赖 STORY-001 batch 字段族和 STORY-003 10 域参数契约；当前实现阶段应被阻断。 | CP5 可接受；实现前必须消费 STORY-001/003 已确认契约，不得创建第二套域名。 |
| STORY-005 | 架构可接受 | `security_gate_check.py` 的只读范围、退出码 31/32/33、敏感脱敏、high-risk gate 和 `.input/` 排除均与 HLD 最小机器校验入口一致；shared CLI 修改被定义为 optional。 | shared CLI 文件为潜在冲突面，但不影响 CP5；实现阶段必须先读当前代码和并行状态。 | meta-se 裁决：shared CLI 修改范围可保留到实现阶段判定；脚本满足 AC 时不应为统一而修改 shared CLI。 |
| STORY-006 | 架构可接受，必须 runtime 阻断 | 文档收口明确依赖上游实际实现和验证事实；5 个文档文件范围清晰；不提前承诺未实现 op_id/package/CLI 能力。 | 即使 CP5 approved，也不得立即实现 STORY-006；必须等待 STORY-001..005 的 CP6/验证证据。 | meta-se 裁决：STORY-006 必须等待 STORY-001..005 实现与验证事实后再实现。 |

## 2. Findings

<!-- findings-table -->

| ID | Severity | Rule Ref | Evidence | Impact | Suggestion | Anchor |
|----|----------|----------|----------|--------|------------|--------|
| F-001 | 一般 / required-before-dev | HLD 门控、确认态不得由 reviewer 代回填 | `process/ARCHITECTURE-DECISION.md` frontmatter 为 `status=draft`、`confirmed=false`；6 个 LLD 与 6 个 CP5 预检均消费 ADR 内容并将其标记为 OPEN。 | 不阻断 CP5 语义评审；会阻断 meta-dev 严格 ready-check 和 dev_ready 计算，除非用户在 CP5 中接受或要求回填。 | 用户 CP5 决策中明确：接受 CP3 approved + CR-003 + CP5 approval 作为 ADR 等价确认，并要求 meta-po 回填 confirmed；或要求先修订/回填后再 approve。 | `process/ARCHITECTURE-DECISION.md` frontmatter；`checkpoints/CP5-ALL-STORIES-LLD-BATCH.md#OPEN / 实现前阻断项汇总` |
| F-002 | 一般 / required-before-story-006-dev | 平台契约优先、文档不得基于未确认安装边界发布 | `process/PLATFORM-INSTALL-SPEC.md` frontmatter 为 `confirmed=false`；`STORY-006` 以 README 原生交付面、uv 安装入口和禁止 `delivery/` 为文档强输入。 | 不阻断 CP5 评审；阻断 `STORY-006` 实现，除非用户接受 Platform spec 或要求 meta-po 回填。 | 用户 CP5 决策中明确：接受 README 原生交付面为 STORY-006 强输入，并要求 meta-po 回填 Platform spec confirmed；或要求先修订 Platform spec。 | `process/PLATFORM-INSTALL-SPEC.md` frontmatter；`process/stories/STORY-006-update-user-facing-docs-and-release-guidance-LLD.md#12. 风险、难点与预研建议` |
| F-003 | 轻微 / architecture-clarification | Package ownership and semantic cohesion | `STORY-003` 默认 `ngfw_verification` 只含 10 个 capacity 验证 op_id，同时 CP5-O-05 留出是否纳入 STORY-002 health check。 | 若纳入 health check，会扩大 package 语义并引入跨 Story package 引用；非必要增加实现耦合。 | meta-se 裁决为不纳入：`ngfw_verification` 只含 10 个 capacity 验证 op_id；`fw_verify_ngfw_health` 留在 `ngfw_installation`。 | `process/stories/STORY-003-model-capacity-ten-domain-config-and-verification-atoms-LLD.md#12. 风险、难点与预研建议` |
| F-004 | 轻微 / implementation-gate | Shared file ownership | `STORY-005` 将 `src/atomic_ops/commands/*.py` 与 `src/atomic_ops/cli.py` 标为 shared optional；O-04 留到实现阶段读取当前代码后决定。 | 不影响 LLD 可评审性；实现阶段若误把 optional 变成默认改动，可能与用户或其他 agent 改动冲突。 | meta-se 裁决为保留 optional：先实现 `scripts/security_gate_check.py`；只有 AC 不能满足时才修改 shared CLI，且不得新增真实设备动作命令。 | `process/stories/STORY-005-add-read-only-security-gate-and-validation-checks-LLD.md#4. 代码结构与文件影响范围` |
| F-005 | 一般 / runtime-gate | Runtime dependency must not be flattened | `DEVELOPMENT-PLAN.yaml` 将 STORY-006 dependency_type 标为 `runtime`；`STORY-006` B-02 要求上游实现和验证证据完成后才解除。 | CP5 approved 后仍不得把 STORY-006 与 W1/W2 并行实现；否则文档会引用未实现事实。 | meta-po 后续计算 dev_ready 时必须继续排除 STORY-006，直到 STORY-001..005 的实现和验证证据可读。 | `process/DEVELOPMENT-PLAN.yaml` W3；`process/stories/STORY-006-update-user-facing-docs-and-release-guidance-LLD.md#12. 风险、难点与预研建议` |
| F-006 | 轻微 / wording | Review evidence wording | `STORY-006` O-03 写到“当前只读取到 STORY-004 与 STORY-005 的 Story 卡”，但本轮 CP5 已实际存在 STORY-004/005 LLD；其核心意图是“未读取 confirmed LLD 或实现结果”。 | 不改变架构决策；可能让审查者误解为 STORY-006 未看到同批 LLD。 | 不要求阻断 CP5；若后续修订 STORY-006，可将 O-03 改为“STORY-004/005 LLD 尚未 confirmed，最终实现与验证事实待定”。 | `process/stories/STORY-006-update-user-facing-docs-and-release-guidance-LLD.md#12. 风险、难点与预研建议` |

## meta-se 已确认裁决

| 裁决 ID | 问题 | 架构裁决 | 理由 | 影响范围 | 是否需要修改 LLD / CP5 / 后续计划 |
|---|---|---|---|---|---|
| D-001 | ADR frontmatter `confirmed=false` 是否使 6 个 LLD 设计不可用 | 从架构内容看，ADR 与 HLD v1.1、CP3 approved、CR-003、Story plan 一致，可作为 CP5 语义评审输入；但 meta-se 不能代替用户把 ADR 确认为 approved。 | ADR 中 ADR-001..006 与 HLD 的推荐方案 B、CLI 只读、状态引用、10 域、批次配置和无自动回滚一致；当前问题是确认态治理，不是架构内容冲突。 | 全部 Story；meta-dev ready-check；meta-po CP5 回填。 | 不要求改 LLD；需要用户 CP5 决策或 meta-po 回填确认状态后再 dev_ready。 |
| D-002 | Platform spec `confirmed=false` 是否影响 STORY-006 | 从架构内容看，Platform spec 与 HLD/README 原生交付面一致，不影响 STORY-006 LLD 可评审；但 STORY-006 实现前必须由用户 CP5 接受或由 meta-po 回填确认。 | Platform spec 只声明本 production 项目不使用 `delivery/`，使用 README 原生交付面和 uv 入口；与 HLD/AGENTS 输出隔离原则一致。 | STORY-006；README/docs/CHANGELOG 文档收口。 | 不要求改 STORY-006 LLD；需要用户确认处理 Platform spec 状态。 |
| D-003 | STORY-001 schema v1.1 字段族是否足够冻结 | 足够作为后续 Story 实现输入，前提是 CP5 approved 后先实现/冻结 STORY-001，并保持版本冲突 fail-fast。 | LLD 明确定义 7 个字段族、v1.0 兼容、敏感互斥、session/state 生命周期、gate、verification、batch，并把 `"1.1"` vs 正则冲突设为实现阻断。 | STORY-001..005 contract；schema/docs/atom/package/security gate。 | 不要求修改 LLD；后续实现若发现 schema 正则冲突，必须回到决策，不得静默改值。 |
| D-004 | `ngfw_verification` package 范围 | 默认只含 STORY-003 的 10 个 capacity 验证 op_id；不纳入 STORY-002 的 `fw_verify_ngfw_health`。 | 健康检查属于安装链路 package `ngfw_installation`；`ngfw_verification` 保持 capacity 配置后验证语义更清晰，避免跨 Story package ownership 扩大。 | STORY-002、STORY-003、STORY-006 文档引用。 | 不要求修改 STORY-003 LLD；建议 CP5 人工结果记录该裁决，后续文档按此写。 |
| D-005 | STORY-005 shared CLI 修改范围是否可保留到实现阶段判定 | 可以。`scripts/security_gate_check.py` 是 primary 交付；shared CLI 修改是 optional，应由实现阶段读取当前代码后最小化决定。 | LLD 已定义只读边界、退出码和安全 gate；shared CLI 文件存在冲突风险，提前强制修改会扩大不必要变更。 | STORY-005；`src/atomic_ops/commands/*.py`、`src/atomic_ops/cli.py`。 | 不要求修改 LLD；后续实现必须先确认文件冲突和必要性。 |
| D-006 | STORY-006 runtime 收口是否必须等待上游实现与验证 | 必须等待 STORY-001..005 的实现和验证事实完成后再实现。CP5 approved 只允许 LLD 成为输入，不解除 runtime dev_gate。 | Development Plan 明确 STORY-006 dependency_type=`runtime`；LLD 要求文档引用实际 op_id/package/CLI/security gate 事实，不能用草案替代。 | STORY-006；README、docs、CHANGELOG。 | 不修改 LLD；后续 meta-po 计算 dev_ready 时不得把 STORY-006 放入 W1/W2 dev_ready。 |

## 需用户决策清单

| 决策 ID | 问题 | 选项 | 默认建议 | 后果 |
|---|---|---|---|---|
| U-001 | 如何处理 `ARCHITECTURE-DECISION.md confirmed=false` | A. 接受 CP3 approved + CR-003 + CP5 approval 作为 ADR 等价确认，并要求 meta-po 回填 `confirmed=true`；B. 仅在 CP5 人工结果中记录风险接受，不回填 ADR；C. 要求先修订/回填 ADR 后再 CP5 approve。 | A | A 最干净，后续 ready-check 不再反复阻断；B 需要所有下游都识别风险接受，易产生二次阻塞；C 最保守但会延后 CP5。 |
| U-002 | 如何处理 `PLATFORM-INSTALL-SPEC.md confirmed=false` | A. 接受 README 原生交付面为 STORY-006 强输入，并要求 meta-po 回填 Platform spec `confirmed=true`；B. 只在 CP5 中风险接受，不回填；C. 要求先修订/确认 Platform spec。 | A | A 避免 STORY-006 文档实现前再次阻断；B 可能让 STORY-006 ready-check 继续卡住；C 适合用户仍不接受安装/交付面边界。 |
| U-003 | CP5 总体处理 | A. `approve` 并显式接受 U-001/U-002 默认建议；B. `修改: 回填 ADR/Platform spec confirmed 后重提 CP5`；C. `reject`。 | B 或 A 均可；若重视治理闭环选 B，若接受 meta-po 后置回填选 A。 | A 进入 meta-po 回填与 dev_ready 计算，但必须保留 runtime gate；B 多一轮回填但状态更干净；C 回退 story-planning。 |

## 对 CP5 的建议

建议结论：`revise`，但修订范围应限定为确认态回填或 CP5 人工结果风险接受，不要求重写 6 份 LLD。

最低可接受 CP5 approval 条件：

- 用户明确接受或要求回填 `ARCHITECTURE-DECISION.md confirmed=false` 的处理方式。
- 用户明确接受或要求回填 `PLATFORM-INSTALL-SPEC.md confirmed=false` 的处理方式。
- CP5 人工结果记录 `ngfw_verification` package 裁决：只包含 10 个 capacity 验证 op_id。
- meta-po 后续计算 dev_ready 时保留所有 dev_gate：STORY-001 先冻结 contract；STORY-002/003/004/005 按依赖和文件所有权调度；STORY-006 等待 STORY-001..005 实现与验证事实。

不建议 `reject`，因为未发现 HLD/ADR/CR-003/Story plan 与 LLD 的根本架构冲突。

## 后续状态建议

| 对象 | 建议状态 / 动作 |
|---|---|
| `checkpoints/CP5-ALL-STORIES-LLD-BATCH.md` | 继续由用户做人工决策；本评审不回填 approve。 |
| `process/ARCHITECTURE-DECISION.md` | 若用户选择 U-001-A 或 U-003-B，由 meta-po 回填确认态和确认时间。 |
| `process/PLATFORM-INSTALL-SPEC.md` | 若用户选择 U-002-A 或 U-003-B，由 meta-po 回填确认态和确认时间。 |
| `STORY-001` | CP5 approved 后优先进入实现，冻结 schema/docs contract。 |
| `STORY-002` / `STORY-003` | 等 STORY-001 contract frozen 后实现；`ngfw_verification` package 按 D-004 执行。 |
| `STORY-004` | 等 STORY-001/003 contract frozen 后实现。 |
| `STORY-005` | 等 STORY-001..004 contract 和文件所有权稳定后实现；shared CLI 修改按 D-005 最小化。 |
| `STORY-006` | 保持 `BLOCKED_FOR_IMPLEMENTATION`，直到 STORY-001..005 实现和验证事实可读。 |

## 3. 汇总结论

- blocking_count: 0
- required_count: 2
- optional_count: 4
- recommended_next_action: `revise-and-resubmit`
- cp5_recommendation: `revise confirm-state handling; do not reject LLD batch`
- blocking_issue_found: false

## 4. 待确认项

- U-001：用户选择 ADR `confirmed=false` 的处理方式。
- U-002：用户选择 Platform spec `confirmed=false` 的处理方式。
- U-003：用户选择 CP5 总体处理：`approve`、`修改: <具体修改点>` 或 `reject`。

---
checkpoint_id: "CP2-CR046-REQUIREMENTS-BASELINE-R2"
checkpoint_name: "CR-046 Requirements, Scenarios and Scope Baseline R2"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-07-12T01:31:00Z"
reviewed_by: "user"
reviewed_at: "2026-07-12T01:43:13Z"
auto_check_result: "process/checks/CP2-CR046-REQUIREMENTS-BASELINE-R2.result.json"
supersedes_checkpoint_ref: "process/checkpoints/CP2-CR046-REQUIREMENTS-BASELINE.md"
target:
  phase: "requirement-clarification"
  story_id: ""
  artifacts: ["docs/product/USE-CASES.md", "docs/product/REQUIREMENTS.md", "docs/product/SCENARIOS.yaml", "docs/product/TEST-MATRIX.md", "docs/product/STORY-MAP.md", "docs/product/MVP-SCOPE.md", "docs/product/RELEASE-SLICES.md", "docs/product/BACKLOG.md"]
---

# CP2 CR-046 Requirements, Scenarios and Scope Baseline R2

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP1-CR046-USE-CASE-COMPLETENESS-R2.result.json` | PASS | 0 | 12 UC、59 REQ、36 TC、20 ST；带 checker provenance/input hashes |
| `process/checks/CP2-CR046-REQUIREMENTS-BASELINE-R2.result.json` | PASS | 0 | R1 五项 finding 全部进入 required scope；traceability PASS |
| `process/reviews/CR046-CP2-SCOPE-REVIEW-R1.md` | changes resolved | 0 | F01..F05 均已状态化为 RESOLVED |

## Decision Brief

### 审批者摘要

| 字段 | 内容 |
|---|---|
| 本次确认服务的整体目标 | 确认补齐 compaction、post-close correction、机器 audit report、provenance dogfooding 和 dispatch 证明等级后的 CR-046 R2 基线 |
| 推荐动作 | `approve` R2：五项 review finding 均已进入 required MVP scope，可进入 CP3 架构设计 |
| approve 后会发生什么 | 自动进入 solution-design，真实调度 meta-se-critical；CP3 必须把五项新契约落到模块、schema、失败路径、Story DAG 与验收 fixture |
| approve 不授权什么 | credentials、runtime、production write、publish、交易、commit/push、quant-lab lineage 业务代码修改、真实 CR-163 pilot 执行 |
| 不确认会阻塞什么 | 阻塞 CP3、Story 拆解、LLD 和实现；当前停留在 requirement-clarification |

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP2-CR046-REQUIREMENT-CONTEXT.yaml` |
| capsule 状态 | `ready-r2-host-review` |
| read_profile | compact |
| 默认读取策略 | capsule-first；只有 conflict/audit/deep review 才展开产品全文 |
| 全文档读取扩展 | `RE-20260712T012815Z0000-1646fc7b`：CP2 scope deep review |
| 缺失 / waived 理由 | 无；WAIVED=0 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue | `STATE.current.json` / gate ledger | scanned | 1 | 1 | CP2 R2 pending |
| 委托 Agent 交还摘要 | `CR046-CP2-SCOPE-REWORK-R2-META-PM-RETURN-SUMMARY.md` | scanned | 5 | 0 | F01..F05 resolved into scope |
| 自动预检结果 | CP1/CP2 R2 result | scanned | 0 | 0 | PASS；blocker=0 |
| discussion/review | `CR046-CP2-SCOPE-REVIEW-R1.md` | scanned | 5 | 0 | 用户已明确要求全部纳入；不是待选项 |
| 下游正式产物 | artifact `docs/product/*` | scanned | 2 | 0 | 既有 DEF-EI-001..002 保持 deferred |
| 用户显式输入 | 本轮评审意见 | scanned | 1 | 1 | 是否批准整改后的整体 R2 基线 |

### 决策分层

| 分类 | 数量 | 处理方式 |
|---|---:|---|
| 必须用户决策 | 1 | CP2-DQ-01-R2 |
| 高风险策略确认 | 0 | 无 runtime/credential/publish 授权请求 |
| agent 默认处理 | 5 | 已按用户明确评审意见更新 F01..F05 |
| 仅审计记录 | 4 | R1 null provenance、session-only dispatch、route metadata observation、R1 supersession |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| CP2-DQ-01-R2 | scope | 是否批准已补齐五项 finding 的 CR-046 产品/场景/MVP R2 基线？ | approve R2，进入 CP3 | A. 修改指定 REQ/TC/ST acceptance 后 R3；B. reject 并保持 CP2 changes_requested | approve 建立完整端到端证据生命周期；修改可继续精确范围但延长 CP2；reject 保留 compaction/correction/audit 断链风险 | 主要风险为 legacy compatibility、receipt/telemetry unavailable、路由兼容布局和历史证据不可变 | 修改回 meta-pm 局部 rework；reject 停止推进；CP3 若不能形成明确 contract 则 changes_requested |

| 字段 | 内容 |
|---|---|
| 推荐决策 | `approve` CP2-DQ-01-R2 |
| 备选方案 | 指定 REQ/TC/ST 修改后 R3；或 reject |
| 影响维度 | 用户价值、实现复杂度、可验证性、维护、平台兼容、安全、交付 |
| 优劣分析 | R2 完整覆盖评审缺口且不增加 Story 数；代价是扩大四个 Story 的实现/验收面 |
| 风险与回退 | CP3 必须设计 compaction restore、correction schema、audit generator、strict provenance 与 evidence levels；设计不足则拒绝 CP3 |
| 用户需决策事项 | `CP2-DQ-01-R2`：approve、修改或 reject R2 基线 |

### R2 Scope Delta

| Finding | Product contract | Scenario | Story acceptance | 状态 |
|---|---|---|---|---|
| Compaction semantic preservation | REQ-EI-019 | TC-EI-014 | ST-EI-002/004 | RESOLVED |
| Post-close correction lifecycle | REQ-EI-020 | TC-EI-015 | ST-EI-007 | RESOLVED |
| Machine-generated audit report | REQ-EI-021 | TC-EI-016 | ST-EI-004/006 | RESOLVED |
| CP1/CP2 null-provenance dogfooding | REQ-EI-022 | TC-EI-017 | ST-EI-006 | RESOLVED |
| Dispatch evidence-level disclosure | REQ-EI-023 | TC-EI-018 | ST-EI-002 | RESOLVED |

### 已知证据限制

- R1 CP1/CP2 原文件仍无 checker provenance；它们被保留为 strict negative fixtures，没有被改写。
- R2 CP1/CP2 为新 result，包含实际 checker invocation、`HEAD=844d351...`、input hashes 和 supersedes refs。
- 当前 Codex dispatch 是 `session-observed/repository-unverifiable`；`/root/pm_cr046_r2` 不是平台签名 receipt。
- handoff `agent_path` 已由 Host 修正为实际存在的 `delivery/agents/meta-pm.md`。
- `workspace check` 在 local-directory compatibility 下报告 OK，但 `routing_ref` 目标缺失；该 cross-truth 观察须在 ST-EI-004/CP3 设计中处理，不在本次 CP2 静默修复。

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| R1 changes_requested 已记录 | PASS | R1 checkpoint/gate ledger/review |  |
| R2 产品基线增量完成 | PASS | artifact `docs/product/*` |  |
| R2 CP1/CP2 自动结果通过 | PASS | R2 results |  |
| Provenance/hash/supersession 可核对 | PASS | R2 results + Host hash verification |  |
| BLOCKING 澄清项为 0 | PASS | R2 return |  |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | F01..F05 全部进入 required scope | 待审查 | R2 result/product baseline |  |
| 2 | 原 ID 和修订历史保留 | 待审查 | product revision records |  |
| 3 | compaction/correction/audit 具有量化 AC | 待审查 | REQ-EI-019..021 |  |
| 4 | null provenance 与 dispatch 上限诚实披露 | 待审查 | REQ-EI-022..023 |  |
| 5 | quant-lab lineage business code 仍 out of scope | 待审查 | MVP-SCOPE/CR |  |
| 6 | CP3 设计责任与失败回退明确 | 待审查 | Decision Brief |  |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| R2 产品/场景/MVP 基线可冻结 | 待审查 | CP2 R2 result |  |
| CP2-DQ-01-R2 关闭 | 待审查 | 人工结论 |  |
| 可进入 solution-design | 待审查 | route plan |  |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| R2 product baseline | `docs/product/*` artifact truth | 待审查 |  |
| CP1 R2 result | `process/checks/CP1-CR046-USE-CASE-COMPLETENESS-R2.result.json` | PASS |  |
| CP2 R2 result | `process/checks/CP2-CR046-REQUIREMENTS-BASELINE-R2.result.json` | PASS |  |
| R2 capsule | `process/context/CP2-CR046-REQUIREMENT-CONTEXT.yaml` | PASS |  |
| R2 return | `process/handoffs/CR046-CP2-SCOPE-REWORK-R2-META-PM-RETURN-SUMMARY.md` | PASS |  |

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-07-12T01:43:13Z
- 修改意见：R2 整改合格。CP3 必须把 `routing_ref` 的 local-directory compatibility 策略作为 architecture 决策项进入 Decision Brief，不得延后到 CP8。
- 风险接受项：
  - 接受当前 R1 null-provenance 作为 strict negative fixture，而非 fully replayable evidence。
  - 接受当前 dispatch 暂为 `session-observed/repository-unverifiable`，由 Story 2 闭环。

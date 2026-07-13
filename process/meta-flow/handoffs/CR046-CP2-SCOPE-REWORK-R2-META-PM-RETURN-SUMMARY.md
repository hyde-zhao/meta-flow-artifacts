---
handoff_id: "HO-CR046-CP2-SCOPE-REWORK-R2-META-PM-RETURN"
from_agent: "meta-pm"
to_agent: "host-orchestrator"
workflow_id: "meta-flow-self-dev"
change_id: "CR-046"
stage: "requirement-clarification-rework"
status: "completed"
created_at: "2026-07-12T01:29:00Z"
context_ref: "process/context/CP2-CR046-REQUIREMENT-CONTEXT.yaml"
cp1_result_ref: "process/checks/CP1-CR046-USE-CASE-COMPLETENESS-R2.result.json"
cp2_result_ref: "process/checks/CP2-CR046-REQUIREMENTS-BASELINE-R2.result.json"
superseded_results_retained:
  - "process/checks/CP1-CR046-USE-CASE-COMPLETENESS.result.json"
  - "process/checks/CP2-CR046-REQUIREMENTS-BASELINE.result.json"
decision: "PASS_TO_HOST_FOR_CP2_PREPARATION"
cp2_launched: false
cp2_approved: false
---

# CR-046 CP2 Scope Rework R2 — meta-pm Return Summary

## 用户真实意图

1. Meta Flow 的 evidence-integrity 范围必须证明 compaction/restore 前后语义等价，不能把 `event_id`、`dispatch_id`、`attempt_id` 与 `run_id` 混为 fallback 展示 ID，也不能丢失 terminal selection、correction chain 或 workflow-health refs。
2. 历史修正必须成为 CP8 后可复用、版本化、append-only 的 correction lifecycle；audit report 必须由工具生成、携带 provenance/input hashes，并把 event rows、attempts、threads、terminal outcomes 与 token measurement metrics 分开。
3. CR-046 自身必须 dogfood 新契约：R1 CP1/CP2 null-provenance results 保持原样并在 strict profile 下失败或降级；当前 Codex dispatch 在无 platform receipt 时只可表述为 `session-observed/repository-unverifiable`。

## Scenario Gray Areas

| 灰区 / Finding | 用户选择 | R2 处理 | 状态 | 证据 |
|---|---|---|---|---|
| CR046-SCOPE-F01 compaction interaction | required scope | 新增 REQ-EI-019 / TC-EI-014；扩展 ST-EI-002/004 | RESOLVED | `docs/product/REQUIREMENTS.md`, `docs/product/SCENARIOS.yaml`, `docs/product/STORY-MAP.md` |
| CR046-SCOPE-F02 post-close correction | required scope | 新增 REQ-EI-020 / TC-EI-015；ST-EI-007 必须消费通用 lifecycle | RESOLVED | 同上 |
| CR046-SCOPE-F03 generated audit report | required scope | 新增 REQ-EI-021 / TC-EI-016；扩展 ST-EI-004/006 | RESOLVED | 同上 |
| CR046-SCOPE-F04 CP1/CP2 dogfooding | required scope | 新增 REQ-EI-022 / TC-EI-017；R1 results 保留不改 | RESOLVED | R1 result refs + R2 product baseline |
| CR046-SCOPE-F05 dispatch limitation | required disclosure | 新增 REQ-EI-023 / TC-EI-018；明确证明上限 | RESOLVED | R2 handoff + product baseline |

原始 discussion checkpoint 继续使用 `process/checks/CP2-CR046-DISCUSSION-CHECKPOINT.json`。R2 没有新歧义需要用户访谈；用户已在 `process/reviews/CR046-CP2-SCOPE-REVIEW-R1.md` 明确接受五项 finding 为 required scope。

## Deferred Ideas

- 既有 `DEF-EI-001..002` / `BL-EI-001..002` 保持不变。
- CR046-SCOPE-F01..F05 均不得 deferred；`docs/product/BACKLOG.md` 已显式记录该边界。

## 需求摘要

| 项目 | R2 结果 |
|---|---|
| 总 requirement rows | 59 |
| P0 / P1 / P2 | 34 / 21 / 4 |
| 新增 requirement IDs | REQ-EI-019..023 |
| BLOCKING 未决项 | 0 |
| ready_for_design | true，但仅在 Host Orchestrator 发起且用户批准 CP2 后生效 |
| 历史 ID 处理 | UC/REQ/TC/ST 既有 ID 均保留；未删除、重排或复用 |

## SCENARIOS 覆盖摘要

| 类型 | 数量 |
|---|---:|
| total | 36 |
| positive | 15 |
| negative | 13 |
| boundary | 3 |
| permission | 2 |
| failure-recovery | 1 |
| precheck | 2 |

新增 `TC-EI-014..018` 分别覆盖 compaction 负例、post-close correction 正例、generated audit golden fixture、null-provenance dogfooding 负例和 dispatch evidence-level 边界。

## TEST-MATRIX 缺口摘要

- 未覆盖 Scenario：0。
- 无 Requirement / UC / Story 回链 Scenario：0。
- N/A / WAIVED：0。
- 自动化状态：0 implemented / 36 planned；这是 CP2 产品基线，不是 CP7 执行证据，且本轮未授权实现。
- 已知接受降级：platform receipt 或 token telemetry 可 unavailable，但不可合成 verified/measured 值。

## STORY-MAP 摘要

- 总 Story：20（ST-PG-001..013 + ST-EI-001..007）；CR-046 Story 数仍为 7。
- 总用户活动：10；CR-046 活动仍为 5。
- R2 不新增 Story ID；只扩展 ST-EI-002/004/006/007 的 requirement refs 和关键验收。
- 重点验收：ST-EI-002 包含 compaction 与 dispatch 证明上限；ST-EI-004/006 包含 machine audit report；ST-EI-006 dogfood R1 null provenance；ST-EI-007 消费通用 correction lifecycle。

## MVP 范围

| 分类 | R2 结论 |
|---|---|
| In Scope | 五项 accepted finding 全部纳入既有 IN-EI-002/004/006/007 |
| Out of Scope | quant-lab lineage business implementation、伪造 receipt/telemetry/provenance、runtime/credentials/production write/publish/trading/commit/push |
| Deferred | 仅保留既有跨平台加密 receipt 与基于估算 token 的强制计费候选；本轮五项 finding 不可 deferred |
| 成功指标 | 新增 compaction/correction 语义完整与 audit/dogfooding 可信口径 |

## 发布切片与 Backlog

- `SL-EI-01` 增加 compaction replay 与 dispatch evidence-level 验证入口。
- `SL-EI-02` 增加 machine audit golden fixture 与 R1 null-provenance dogfooding。
- `SL-EI-03` 要求 CR-163 pilot 消费通用 post-close correction lifecycle。
- Backlog 保留原 ID，并新增明确说明：CR046-SCOPE-F01..F05 是 required scope，不得静默转 deferred。

## CP1 / CP2 证据

| 结果 | 路径 | 结论 | Checker provenance |
|---|---|---|---|
| CP1 R2 | `process/checks/CP1-CR046-USE-CASE-COMPLETENESS-R2.result.json` | PASS | `meta-flow cp result-check`, commit `844d351abee5b3872f30f0c6cab389c12e15fb83`, fallback=false |
| CP2 R2 auto precheck | `process/checks/CP2-CR046-REQUIREMENTS-BASELINE-R2.result.json` | PASS | 同上 |
| CP1 R1 historical input | `process/checks/CP1-CR046-USE-CASE-COMPLETENESS.result.json` | 保留 | checker_provenance absent；TC-EI-017 negative fixture |
| CP2 R1 historical input | `process/checks/CP2-CR046-REQUIREMENTS-BASELINE.result.json` | 保留 | checker_provenance absent；TC-EI-017 negative fixture |

执行结果：

- `uv run meta-flow cp result-check --result process/checks/CP1-CR046-USE-CASE-COMPLETENESS-R2.result.json --project-root .` → OK
- `uv run meta-flow cp result-check --result process/checks/CP2-CR046-REQUIREMENTS-BASELINE-R2.result.json --project-root .` → OK
- Traceability check → PASS：36 scenarios，59 requirements，20 stories，12 use cases，bad refs=0，R2 matrix missing IDs=0。
- `process/state/READ-EXPANSION-LEDGER.ndjson` 新增 `RE-20260712T012815Z0000-1646fc7b`，记录 handoff 要求的 CR deep review；两个 R2 result 均引用该事件。

## 待人工决策项

| 决策 ID | 类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣 / 风险 | 回退条件 |
|---|---|---|---|---|---|---|
| CP2-DQ-01-R2 | scope | 是否接受补齐五项 finding 后的 CR-046 产品/场景/MVP 基线 | approve R2 baseline | 修改指定 REQ/TC/ST acceptance；或 reject 并保持 CP2 changes_requested | approve 使架构阶段可完整覆盖审计语义；修改会延长 CP2；reject 保留现状缺口 | 修改则回 meta-pm 局部 rework；reject 保持 requirement-clarification |

Host Orchestrator 需要自行生成/刷新 CP2 Decision Brief 与人工 checkpoint，并发起用户确认。本 return 不发起、不批准 CP2。

## N/A / WAIVED 与不授权项

- 实现、自动化 fixture 执行、CP7 evidence：N/A at CP2；触发条件为 CP2/CP3/CP5 依次批准并进入相应 Story。
- CP2 human checkpoint launch/approval：N/A for meta-pm；仅 Host Orchestrator 有权执行。
- WAIVED：0。
- 不授权：credentials、runtime、production write、publish、trading、repository commit/push、quant-lab lineage business-code changes。

## 已知限制与非阻塞观察

1. 当前 Codex dispatch 的 agent/thread/tool 信息是会话观察事实，但仓库内无 platform receipt；证明等级只能是 `session-observed/repository-unverifiable`。这是 R2 必须披露的产品事实，不是可在本轮修复的 checker blocker。
2. handoff 的 `agent_path: agents/meta-pm.md` 在工作树不存在；本轮实际读取 canonical delivery contract `delivery/agents/meta-pm.md`。未修改 handoff 或 agent 安装结构。
3. `STATE.current.json.routing_ref` 指向 `process/.meta-flow-process.yaml`，但该文件当前不存在，且 `process` 是 local directory 而非 symlink。Host Orchestrator 已明确下发本次 bounded R2 写入；本轮未自行创建、修复或重建路由/状态。后续正式 CP2 launch 前建议 Host Orchestrator 单独确认 workspace route health。
4. 未追加 CHECKPOINT-LEDGER、未刷新人工 checkpoint、未修改 `STATE.current.json`：这些均属于 Host Orchestrator 收敛职责。

## 交还结论

R2 产品基线与自动预检已完成，结果校验和追踪检查均通过，无产品范围 blocker。交还 Host Orchestrator 准备 CP2 Decision Brief；不得把本 return 或 CP2 auto PASS 解释为人工批准、实现授权或运行/发布授权。

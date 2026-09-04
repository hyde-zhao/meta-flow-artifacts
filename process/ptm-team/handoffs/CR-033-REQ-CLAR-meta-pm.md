---
handoff_id: "CR-033-REQ-CLAR-meta-pm"
cr_id: "CR-033"
phase: "requirement-clarification"
canonical_role: "meta-pm"
delegated_interaction: true
agent_role: "meta-pm"
status: "completed"
created_at: "2026-07-28T08:40:00+00:00"
owner: "host-orchestrator"
context_ref: "process/changes/CR-033.md"
cp0_result_ref: "process/checks/CP0-CR-033-BOOTSTRAP.result.json"
---

# Handoff: CR-033 requirement-clarification -> meta-pm

## 委托背景

CR-033（统一目标包 parent CR）已完成 CP0 bootstrap（PASS），STATE 已切换至 `requirement-clarification / standard`。按 Meta Flow 流程，requirement-clarification 阶段委托 meta-pm 直连用户完成场景发现与需求结构化。

**关键前提**：范围目标已由用户与 host-orchestrator 在前置对话中确认并定稿（含审核修订），meta-pm **不重新做范围决策**，只基于已确认范围产出正式产物并补充 Scenario Gray Areas。

## 必读上下文

- `process/changes/CR-033.md`：完整范围目标 + 决策定稿 + 12 条改进分期 + 量化成功标准 + 非目标 + 开放项
- `process/REQUEST.md`：三需求原始请求
- `process/state/STATE.current.json`：当前状态
- `process/context/CP0-CR033.context.json`：CP0 上下文

## 任务清单

1. **阶段零快速调研**：调研 ptm-team 现有 `skills/device-management/`、`skills/policy-route-execution/scripts/op_mapper.py`、`skills/trex-traffic/`，以及 `/home/hyde/projects/ptm-te/exec_v4.py`，记录到 `process/CLARIFICATION-LOG.md`（追加，不覆盖）。
2. **Scenario Gray Areas**：识别 3-4 个影响交付的灰区（候选：TG 设备是否纳入快照已决=不纳入、case-execution 的 dry-run/runtime_authorization 边界、跨仓库路由 ptm-team↔ptm-te、known_issue 的 DUT 行为差异判定、ARP 预热与正式流 template 一致性），让用户选 1-3 个重点讨论，**至少 1 条 SGQ 用户确认**，写入 `process/discussions/CP2-SCENARIO-DISCUSSION-LOG.md`。
3. 产出 `docs/product/USE-CASES.md`（场景，含画像/指标/候选理解/认知盲区/Deferred Ideas）。
4. 产出 `docs/product/REQUIREMENTS.md`（结构化需求，含风险/里程碑，追加 `## 修订记录`）。
5. 产出 `docs/product/SCENARIOS.yaml` + `docs/product/TEST-MATRIX.md`（工程验证场景）。
6. 产出 `docs/product/STORY-MAP.md` / `MVP-SCOPE.md` / `RELEASE-SLICES.md` / `BACKLOG.md`。
7. 识别目标项目交付路由（ptm-team canonical + ptm-te workspace），写入过程元数据。
8. 产出 CP2 Decision Brief 输入（用户真实意图、灰区处理、Deferred、范围取舍、成功指标、风险）。
9. 写 CP1/CP2 自动检查输入供 host-orchestrator 发起 CP2。

## 已确认决策（不可推翻）

| 决策点 | 结论 |
|---|---|
| 打包 | parent CR，统一 HLD，多 Story 分期 |
| 引擎形态 | `skills/case-execution/`（SKILL.md + `scripts/case_runner.py` 兼 argparse CLI，op_mapper 模式） |
| 范围 | 全量 12 条，P0->P3 四期 |
| TG 建模 | `type:TG` + `api_server`，纳入清单不纳入快照，不引入 pydantic |
| #4 | 补 op_mapper 新增 `fw_logout` op + case_runner cleanup 登出 |
| #3 | 补 24 用例 md ARP 预热批量整改 |

## 约束

- `scenario_subject_type=target-artifact`（ptm-te 工作流产品，非 meta 自身）
- `engagement_mode=production`（ptm-te 是目标产品，skill 源在 ptm-team）
- 跨仓库：exec_v4.py + 24 用例 md 在 `/home/hyde/projects/ptm-te/`（workspace）
- 交还时只传 `context_ref`、产物路径、自动检查路径、待决策项摘要、evidence refs

## dispatch

| 字段 | 值 |
|---|---|
| mode | subagent |
| canonical_role | meta-pm |
| reasoning_profile | medium |
| dispatch_trigger | CP0-passed-auto-advance |
| tool_name | Agent |
| spawned_at | 2026-07-28T08:40:00+00:00 |
| agent_id | acac6b9b618f6bc43 |
| completed_at | 2026-07-28T09:40:00+00:00 |

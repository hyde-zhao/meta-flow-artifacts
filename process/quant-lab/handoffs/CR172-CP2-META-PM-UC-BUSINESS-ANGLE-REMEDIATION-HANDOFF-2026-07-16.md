---
handoff_id: "CR172-CP2-META-PM-UC-REMEDIATION-2026-07-16"
workflow_id: "CR-172"
change_id: "CR-172"
stage: "requirement-clarification"
from_agent: "host-orchestrator"
to_agent: "meta-pm"
canonical_role: "meta-pm"
mode: "subagent"
status: "returned"
created_at: "2026-07-16T10:53:18+08:00"
completed_at: "2026-07-16T11:04:35+08:00"
context_ref: "process/context/CP2-CR172.context.json"
route_plan_ref: "process/checks/CP0-CR172.route-plan.json"
result_ref: "process/checks/CP1-CR172-USE-CASE-COMPLETENESS.result.json"
return_ref: "process/handoffs/CR172-CP2-META-PM-UC-REMEDIATION-RETURN-SUMMARY.md"
dispatch:
  canonical_role: "meta-pm"
  codex_agent_name: "meta-pm"
  nickname: "pm-wu"
  reasoning_profile: "default"
  dispatch_trigger: "CP2 user review requested UC-58 business-angle remediation without changing the decision batch or authorization boundary"
  tool_name: "followup_task"
  agent_id: "/root/pm_wu_cr172"
  thread_id: "/root/pm_wu_cr172"
  agent_name: "pm_wu_cr172"
---

# CR-172 CP2 Meta-PM Use-Case 业务视角整改 Handoff

## 任务目标

增量修订 `UC-58-CR172`，把已通过结构与覆盖评审的治理型用例补足为可回答“谁因为什么业务痛点、在什么触发下、如何使用未来产出、获得什么价值”的用户使用场景。重跑 CP1 与 CP2 自动预检，为 Host Orchestrator 重新发起同一 CP2 人工门禁提供证据；不得自行批准或发起门禁。

## 必须读取与上下文策略

1. 先读取 `process/context/CP2-CR172.context.json`、其中 `must_read` / `allowed_reads`、CR summary 与原 meta-pm return summary。
2. 完整读取 `docs/product/USE-CASES.md` 前，按 capsule 规则执行 `meta-flow context read-log` 并在 CP1/CP2 result 记录 event ID。
3. 仅当覆盖校验需要时展开 REQUIREMENTS / SCENARIOS / TEST-MATRIX；展开前同样记录 read-expansion event。
4. 当前用户评审可作为第二次可见确认 `SGQ-CR172-002`，不得为了形式再次询问同一问题。

## 必须完成的整改

1. 在 `USE-CASES.md` 修订记录追加本轮增量修订，保留旧基线与既有追溯。
2. 在 `UC-58-CR172` 显式增加：
   - **业务动机**：研究负责人需要用真实数据判断策略在 multiple-testing / overfit 风险下的可靠性；fixture/static evidence 和当前 `effective_trial_count=typed_unavailable` 不能支撑成熟准入判断。
   - **用户痛点**：合成 fixture 只能验证契约/流程，不能作为真实策略准入依据。
   - **使用价值**：未来 C1 real typed evidence 供策略研究员做真实 multiple-testing / overfit 评估，并为后续 mature SAP 提供证据基础；该产出本身不构成 admission PASS/PASS_WITH_RISK。
3. 把触发条件与旅程起点改为用户业务触发：研究负责人提出“策略 X 需要真实数据证据”，随后才选择 activation 半径和治理路径；在现有旅程步骤前增加明确的第 0 步。
4. 更新 UC 的动机维度说明：业务价值是主锚点，blast-radius/授权治理是实现约束；不得把 CP2 路径决策继续当作全部用户价值。
5. 在 `process/CLARIFICATION-LOG.md`、`process/discussions/CP2-CR172-SCENARIO-DISCUSSION-LOG.md`、`process/checks/CP2-CR172-DISCUSSION-CHECKPOINT.json` 记录 `SGQ-CR172-002` 已确认及本次结论。
6. 复核但不改写 `REQ-CR172-001~008` 语义；确认 requirements/scenarios/matrix 仍为 `8/8/8`，8 个正式 DQ 仍为 OPEN。
7. 更新并重跑以下自动证据：
   - `process/checks/CP1-CR172-USE-CASE-COMPLETENESS.md`
   - `process/checks/CP1-CR172-USE-CASE-COMPLETENESS.result.json`
   - `process/checks/CP2-CR172-REQUIREMENTS-BASELINE.md`
   - `process/checks/CP2-CR172-REQUIREMENTS-BASELINE.result.json`
8. 输出 `process/handoffs/CR172-CP2-META-PM-UC-REMEDIATION-RETURN-SUMMARY.md`，列出修改、覆盖计数、未变边界、read expansion refs 与退出状态。

## 冻结不变项

- `REQ-CR172-001~008` 的语义、优先级和验收标准不变。
- `DQ-CR172-001~008` 数量、推荐/备选/风险/回退与 OPEN 状态不变。
- 当前事实仍是五字段 `0/5`、data owner `OPEN`；不得补值、推断或隐式继承。
- 当前直接 `approve` 仍只选择 PATH-B；`5/5+owner` 后默认 PATH-C；PATH-A 仍需显式接受三 producer blast radius。
- PATH-B 仍是 activation 前置而非替代；PATH-C 后 C2/C3 默认两个独立 runtime-high-risk CR。
- joint approval 仍要求同 revision/hash 双 owner ledger，授权取交集。
- claim ceiling 与 9 类 deny-default 不变；本任务真实 lake/NAS、credential、provider、write、computation、runtime、trading 和 remote Git 操作均为 0。

## 允许写入

- `docs/product/USE-CASES.md`
- `process/CLARIFICATION-LOG.md`
- `process/discussions/CP2-CR172-SCENARIO-DISCUSSION-LOG.md`
- `process/checks/CP2-CR172-DISCUSSION-CHECKPOINT.json`
- `process/checks/CP1-CR172-USE-CASE-COMPLETENESS.md`
- `process/checks/CP1-CR172-USE-CASE-COMPLETENESS.result.json`
- `process/checks/CP2-CR172-REQUIREMENTS-BASELINE.md`
- `process/checks/CP2-CR172-REQUIREMENTS-BASELINE.result.json`
- `process/handoffs/CR172-CP2-META-PM-UC-REMEDIATION-RETURN-SUMMARY.md`

## 禁止写入

- `process/state/STATE.current.json`、`process/STATE.md`、`process/current/CURRENT.json`
- `process/changes/CR-172.md`
- `process/checkpoints/CP2-CR172-REQUIREMENTS-BASELINE.md`
- `process/context/CP2-CR172.context.json`
- `process/state/*-LEDGER.ndjson`
- REQUIREMENTS、SCENARIOS、TEST-MATRIX、STORY-MAP、MVP-SCOPE、RELEASE-SLICES、BACKLOG（仅复核，不修改；发现真实冲突则返回 Host）
- Story、HLD、LLD、实现、源码、测试源码、真实数据或外部系统

## 退出条件

- UC-58 的业务动机、用户痛点、使用价值、业务触发共 `4/4` 明示。
- `SGQ-CR172-002` 记录为用户已确认，不重复提问。
- CP1 与 CP2 自动预检均为 `PASS`，blocker=`0`，manual gate 仍为 `pending`。
- requirements/scenarios/matrix=`8/8/8`，human decisions=`8` 且均为 `OPEN`。
- CP2 不被批准，`ready_for_design=false`，所有 operation count 保持 `0`。

## 返回结果

- 返回状态：`PASS / returned`
- Host 验收时间：`2026-07-16T11:04:35+08:00`
- CP1 机器结果：`process/checks/CP1-CR172-USE-CASE-COMPLETENESS.result.json`
- CP2 自动预检：`process/checks/CP2-CR172-REQUIREMENTS-BASELINE.result.json`
- 交还摘要：`process/handoffs/CR172-CP2-META-PM-UC-REMEDIATION-RETURN-SUMMARY.md`
- 停止原因：`required_human_gate`

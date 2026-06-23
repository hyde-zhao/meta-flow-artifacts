---
checkpoint_id: "CP2-CR120"
checkpoint_name: "CR Tracking Formal/FU Row Convention Scope Review"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-06-22T19:44:17+08:00"
reviewed_by: "user"
reviewed_at: "2026-06-22T19:54:11+08:00"
auto_check_result: "process/checks/CP2-CR120-CR-TRACKING-ROW-CONVENTION-PRECHECK.md"
target:
  phase: "requirement-clarification"
  story_id: ""
  artifacts:
    - "process/changes/CR-120-CR-TRACKING-FORMAL-FU-ROW-CONVENTION-HARDENING-2026-06-22.md"
    - "process/docs/design/CR-TRACKING-FORMAL-FU-ROW-CONVENTION-HARDENING-CR120.md"
    - "process/context/CP2-CR120-CR-TRACKING-ROW-CONVENTION-CONTEXT.yaml"
---

# CP2 CR120 CR Tracking Row Convention 人工审查

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP2-CR120-CR-TRACKING-ROW-CONVENTION-PRECHECK.md` | PASS | 0 | 可发起 CP2 scope confirmation。 |

## Decision Brief

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP2-CR120-CR-TRACKING-ROW-CONVENTION-CONTEXT.yaml` |
| capsule 状态 | ready |
| read_profile | minimal |
| 默认读取策略 | 先读 capsule；仅在缺失、冲突、字段不足、人工审计或用户要求时读取完整来源。 |
| 全文档读取扩展 | 0 次 |
| 缺失 / waived 理由 | N/A |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue | `process/STATE.md.human_gate_decisions.pending_decision_ids` | scanned | 4 | 4 | 本轮 CP2 四项决策纳入待确认。 |
| 用户显式选择题 | 当前对话 | scanned | 1 | 1 | 用户明确启动 `FU-CR114-004`，转为 `DQ-CP2-CR120-01`。 |
| source follow-up tracking | `process/changes/CR-116-FOLLOW-UP-TRACKING-2026-06-22.md#FU-CR114-004` | scanned | 2 | 2 | source/tests/checker authorization 与独立实现 CR 要求纳入 `DQ-CP2-CR120-02`。 |
| original follow-up tracking | `process/changes/CR-114-FOLLOW-UP-TRACKING-2026-06-22.md#FU-CR114-004` | scanned | 1 | 1 | 原始 no-code/no-runtime 边界纳入 `DQ-CP2-CR120-03`。 |
| CR tracking index | `process/changes/CR-INDEX.yaml` | scanned | 1 | 1 | 当前候选状态与 formal CR 转换纳入 CP2。 |
| runtime / NAS / credentials | N/A | n/a | 0 | 0 | 本 CR 不需要外部运行时；明确列为不授权。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP2-CR120-01 | scope | 是否接受将 `FU-CR114-004` 升格为 CR120，并以 CR tracking formal/FU row convention hardening 为独立范围？ | 接受 CR120 作为独立 standard CR，CP2 只确认范围和授权边界。 | A. 暂缓，退回 follow-up candidate；B. reject 并取消候选。 | 推荐方案保留追溯并解决高频一致性缺口；暂缓不新增风险但继续依赖人工同步；reject 结束该候选。 | 影响 CR tracking、follow-up row、mirror row 和后续 checker 目标。 | reject 时关闭 CR120 并把 `FU-CR114-004` 标记 cancelled；暂缓时回 candidate。 |
| DQ-CP2-CR120-02 | implementation | 是否接受分阶段路线：CP2 仅做 scope / owner discovery / target definition，源码、tests 和 checker implementation 必须等 CP5？ | 接受 staged route，CP2 不改代码，CP5 再决定最小实现切片。 | A. 仅继续写设计说明，不进入 CP5；B. 合并到后续更大的 CR tracking checker 统一改造。 | 推荐方案边界清晰且可小步验证；design-only 无 enforcement；统一改造更完整但范围更大。 | 影响是否可以后续触碰 `scripts/` 与 `tests/`。 | 若 CP5 前 owner 不清，回到 CP2/CP5 修改范围。 |
| DQ-CP2-CR120-03 | runtime_authorization | 是否继续禁止 runtime / NAS / credentials / trading / provider / lake / catalog / publish？ | 继续禁止，仅允许本地静态 / fixture 检查。 | A. 暂停所有验证命令直到 CP5；B. 另起 runtime gate。 | 推荐方案权限最小且足够支撑本地 checker 设计；暂停会阻断后续验证；runtime gate 与本任务无关。 | 防止 row convention hardening 被误解为真实运行授权。 | 任何 runtime/NAS/publish 需求必须另起独立门禁。 |
| DQ-CP2-CR120-04 | risk_acceptance | 是否接受在当前工作区已有 CR118 / CR119 未提交 checker diff 的前提下启动 CR120 process gate？ | 接受；CR120 CP2 只写 process artifacts，不回滚既有源码/tests diff。 | A. 先要求提交或清理 CR118/CR119 diff；B. 暂缓 CR120。 | 推荐方案不破坏已验证改动；清理更干净但延后；暂缓保持缺口。 | 若后续 CP5 实施，需叠加验证 CR118 / CR119 测试，避免回归。 | 若现有测试失败，停止 CP5 并先收敛 CR118/CR119 diff。 |

| 字段 | 内容 |
|---|---|
| 推荐决策 | `approve` 接受 CP2 四项推荐方案，允许进入 CP5 scope / implementation authorization 准备。 |
| 备选方案 | `修改: <具体修改点>` 指明要调整的决策；或 `reject` 停止 CR120。 |
| 影响维度 | CR tracking row convention、follow-up mirror row、checker / tests owner、权限边界、工作区风险 |
| 风险与回退 | 当前风险低到中；CP2 不改源码/tests。若不接受，退回 `FU-CR114-004` candidate 或取消。 |
| 用户需决策事项 | 是否批准 CR120 scope、staged route、不授权 runtime、接受当前工作区叠加风险。 |
| 自动终验授权 | `auto_final_authorization: false`；CP2 approve 不等于 CP5 实现授权或 CP8 终验授权。 |

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| 用户明确启动候选 | 通过 | 当前对话：启动 `FU-CR114-004` | 已确认。 |
| 冲突预检通过 | 通过 | `meta-flow check cr-tracking --project-root .` | 已确认。 |
| CP2 context ready | 通过 | `process/context/CP2-CR120-CR-TRACKING-ROW-CONVENTION-CONTEXT.yaml` | 已确认。 |
| 自动预检 PASS | 通过 | `process/checks/CP2-CR120-CR-TRACKING-ROW-CONVENTION-PRECHECK.md` | 已确认。 |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | CR120 范围清楚 | 通过 | CR120 变更单 | formal/FU row convention hardening。 |
| 2 | 分阶段授权清楚 | 通过 | DQ-CP2-CR120-02 | CP2 不改代码，CP5 再授权实现。 |
| 3 | 不授权范围清楚 | 通过 | DQ-CP2-CR120-03 | runtime、NAS、凭据、交易写、publish 禁止。 |
| 4 | 工作区叠加风险已暴露 | 通过 | DQ-CP2-CR120-04 | CR118 / CR119 diff 不回滚。 |
| 5 | Decision Collection Coverage 完整 | 通过 | 本文件 Decision Brief | 6 个来源已扫描。 |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| 用户确认 CP2 | 通过 | 本文件人工审查结果 | 用户已 approve。 |
| 可进入 CP5 准备 | 通过 | DQ-CP2-CR120-01..04 | 仍不授权源码/tests/checker implementation。 |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| CR120 formal CR | `process/changes/CR-120-CR-TRACKING-FORMAL-FU-ROW-CONVENTION-HARDENING-2026-06-22.md` | 通过 |  |
| CR120 scope note | `process/docs/design/CR-TRACKING-FORMAL-FU-ROW-CONVENTION-HARDENING-CR120.md` | 通过 |  |
| CP2 context capsule | `process/context/CP2-CR120-CR-TRACKING-ROW-CONVENTION-CONTEXT.yaml` | 通过 |  |
| CP2 precheck | `process/checks/CP2-CR120-CR-TRACKING-ROW-CONVENTION-PRECHECK.md` | 通过 |  |

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-06-22T19:54:11+08:00
- 修改意见：用户回复 `approve`，接受 `DQ-CP2-CR120-01`、`DQ-CP2-CR120-02`、`DQ-CP2-CR120-03`、`DQ-CP2-CR120-04` 推荐方案；允许进入 CP5 implementation authorization 准备。
- 风险接受项：接受在当前工作区已有 CR118 / CR119 未提交 checker diff 的前提下继续 CR120 process gate；CP2 仍不授权源码、tests、checker implementation、bulk historical STATE rewrite、runtime、NAS、凭据、交易写或 publish。

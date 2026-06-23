---
checkpoint_id: "CP5-CR120"
checkpoint_name: "CR Tracking Row Convention Implementation Authorization Review"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-06-22T19:54:11+08:00"
reviewed_by: "user"
reviewed_at: "2026-06-22T20:04:30+08:00"
auto_check_result: "process/checks/CP5-CR120-CR-TRACKING-ROW-CONVENTION-PRECHECK.md"
target:
  phase: "story-planning"
  story_id: "STORY-CR120-CR-TRACKING-ROW-CONVENTION"
  artifacts:
    - "../meta-flow/meta_flow/checks/cr_tracking.py"
    - "../meta-flow/tests/test_cr120_cr_tracking_row_convention.py"
    - "process/context/CP5-CR120-CR-TRACKING-ROW-CONVENTION-CONTEXT.yaml"
---

# CP5 CR120 CR Tracking Row Convention 实现授权审查

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP5-CR120-CR-TRACKING-ROW-CONVENTION-PRECHECK.md` | PASS | 0 | 可发起 CP5 implementation authorization。 |

## Decision Brief

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP5-CR120-CR-TRACKING-ROW-CONVENTION-CONTEXT.yaml` |
| capsule 状态 | ready |
| read_profile | minimal |
| 默认读取策略 | 先读 capsule；仅在缺失、冲突、字段不足、人工审计或用户要求时读取完整来源。 |
| 全文档读取扩展 | 1 次：CP2 approved 后为 owner discovery 读取 checker / test owner。 |
| 缺失 / waived 理由 | N/A |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue | `process/STATE.md.human_gate_decisions.pending_decision_ids` | scanned | 5 | 5 | 本轮 CP5 五项决策纳入待确认。 |
| CP2 人工审查 | `process/checkpoints/CP2-CR120-CR-TRACKING-ROW-CONVENTION-REVIEW.md` | scanned | 4 | 0 | 已 approved；作为 CP5 输入。 |
| owner discovery | `../meta-flow/meta_flow/checks/cr_tracking.py`; `../meta-flow/tests/` | scanned | 2 | 2 | owner 与 test target 纳入 `DQ-CP5-CR120-01` / `02`。 |
| legacy checker | `scripts/check_cr_tracking_consistency.py`; `tests/test_cr093_cr_tracking_consistency.py` | scanned | 1 | 1 | 是否改 legacy checker 纳入 `DQ-CP5-CR120-04`。 |
| 工作区状态 | git status | scanned | 1 | 1 | CR118 / CR119 未提交 diff 风险纳入 `DQ-CP5-CR120-05`。 |
| runtime / NAS / credentials | N/A | n/a | 0 | 0 | 本 CR 不需要外部运行时；明确列为不授权。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP5-CR120-01 | implementation | 是否授权最小 source / checker implementation owner？ | 授权仅修改 `../meta-flow/meta_flow/checks/cr_tracking.py`，为 formal CR / FU row convention 增加 focused validation。 | A. 只新增 tests 不改 checker；B. 改当前仓库 legacy `scripts/check_cr_tracking_consistency.py`；C. 暂停实现。 | 推荐方案命中实际 `meta-flow check cr-tracking`；test-only 不能 enforcement；legacy checker 不是主入口；暂停保留缺口。 | 会产生外部 meta-flow source diff；必须保持范围最小。 | 若外部 repo owner 不可写或测试失败，停止并回 CP5 修改 owner。 |
| DQ-CP5-CR120-02 | implementation | 是否授权新增 focused tests？ | 新增 `../meta-flow/tests/test_cr120_cr_tracking_row_convention.py`，覆盖 candidate->active、formal active mirror、closed/no-follow-up 等 row convention fixture。 | A. 扩展既有 `test_cr_lifecycle.py`；B. 只用手工 checker run；C. 暂缓测试。 | 新文件隔离清晰；扩展旧测试会混合生命周期主题；手工 run 不足以防回归。 | 增加外部 meta-flow tests diff。 | 若测试结构与 repo 规范冲突，改为扩展现有测试文件。 |
| DQ-CP5-CR120-03 | runtime_authorization | CP5 approve 是否授权任何 runtime / NAS / credentials / trading / provider / lake / catalog / publish？ | 不授权；只允许本地静态 / fixture 验证命令。 | A. 暂停所有验证命令；B. 另起 runtime gate；C. 只运行 checker 不跑 tests。 | 推荐方案可验证且权限最小；暂停会阻断实施；runtime gate 与本任务无关；只跑 checker 覆盖不足。 | 防止 implementation gate 被误解为真实运行授权。 | 任何真实外部动作都必须另起高风险门禁。 |
| DQ-CP5-CR120-04 | implementation | 是否同步修改当前仓库 legacy checker？ | 不修改 `scripts/check_cr_tracking_consistency.py`；本 CR 先硬化实际 `meta-flow check cr-tracking`。 | A. 同时改 legacy checker；B. 只改 legacy checker；C. 创建后续 legacy cleanup 候选。 | 推荐方案避免双实现扩散；同时改增加维护成本；只改 legacy 无法影响正式命令。 | legacy checker 与 package checker 可能继续存在能力差异。 | 若后续仍消费 legacy checker，再另起 follow-up。 |
| DQ-CP5-CR120-05 | risk_acceptance | 是否接受在现有 CR118 / CR119 dirty diff 上叠加 CR120 外部 meta-flow diff？ | 接受叠加；实现时不回滚当前仓库 CR118 / CR119 diff，并额外运行 CR118 / CR119 目标测试。 | A. 先提交或清理当前工作区；B. 暂停 CR120；C. 将 CR120 改为 design-only。 | 推荐方案推进最快且不破坏现有 diff；清理更干净但延后；design-only 无 enforcement。 | 多仓库 diff 并存，验证必须明确分层。 | 若 CR118 / CR119 tests 或 meta-flow tests 失败，停止并回 CP5/相关 CR 收敛。 |

| 字段 | 内容 |
|---|---|
| 推荐决策 | `approve` 接受 CP5 五项推荐方案，授权最小实现切片。 |
| 备选方案 | `修改: <具体修改点>` 指明要调整的决策；或 `reject` 停止 CP5。 |
| 影响维度 | meta-flow checker、fixture coverage、legacy checker divergence、工作区多仓库 diff、安全 / runtime 边界 |
| 风险与回退 | 风险中等；回退方式为停止实现并回 CP5 修改 owner / test target，或退回 design-only。 |
| 用户需决策事项 | 是否授权外部 meta-flow source/tests 最小修改、是否不改 legacy checker、是否接受 dirty diff 叠加。 |
| 自动终验授权 | `auto_final_authorization: false`；CP5 approve 不等于 CP8 终验关闭。 |

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| CP2 已 approved | 通过 | `process/checkpoints/CP2-CR120-CR-TRACKING-ROW-CONVENTION-REVIEW.md` | 已确认。 |
| owner discovery 完成 | 通过 | `process/docs/design/CR-TRACKING-FORMAL-FU-ROW-CONVENTION-HARDENING-CR120.md` | 已确认。 |
| CP5 context ready | 通过 | `process/context/CP5-CR120-CR-TRACKING-ROW-CONVENTION-CONTEXT.yaml` | 已确认。 |
| 自动预检 PASS | 通过 | `process/checks/CP5-CR120-CR-TRACKING-ROW-CONVENTION-PRECHECK.md` | 已确认。 |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | 最小实现 owner 清楚 | 通过 | `DQ-CP5-CR120-01` | 推荐外部 meta-flow checker。 |
| 2 | 最小测试 owner 清楚 | 通过 | `DQ-CP5-CR120-02` | 推荐新增 focused tests。 |
| 3 | 不授权边界清楚 | 通过 | `DQ-CP5-CR120-03` | 不授权 runtime/NAS/credentials/trading/publish。 |
| 4 | legacy checker 策略清楚 | 通过 | `DQ-CP5-CR120-04` | 默认不改 legacy checker。 |
| 5 | 多仓库 dirty diff 风险清楚 | 通过 | `DQ-CP5-CR120-05` | 需额外回归 CR118 / CR119 tests。 |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| 用户确认 CP5 | 通过 | 本文件人工审查结果 | 用户已 approve。 |
| 可进入最小实现 | 通过 | DQ-CP5-CR120-01..05 | 已授权最小实现切片。 |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| proposed checker owner | `../meta-flow/meta_flow/checks/cr_tracking.py` | 通过 | CP5 已授权最小修改。 |
| proposed tests owner | `../meta-flow/tests/test_cr120_cr_tracking_row_convention.py` | 通过 | CP5 已授权新增。 |
| CP5 context capsule | `process/context/CP5-CR120-CR-TRACKING-ROW-CONVENTION-CONTEXT.yaml` | 通过 |  |
| CP5 precheck | `process/checks/CP5-CR120-CR-TRACKING-ROW-CONVENTION-PRECHECK.md` | 通过 |  |

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-06-22T20:04:30+08:00
- 修改意见：用户回复 `approve`，接受 `DQ-CP5-CR120-01`..`DQ-CP5-CR120-05` 推荐方案；授权最小修改 `../meta-flow/meta_flow/checks/cr_tracking.py` 并新增 focused tests。
- 风险接受项：接受在现有 CR118 / CR119 dirty diff 上叠加 CR120 外部 meta-flow diff；不修改 legacy checker；runtime、NAS、凭据、交易写、provider fetch、lake write、catalog publish 继续不授权。

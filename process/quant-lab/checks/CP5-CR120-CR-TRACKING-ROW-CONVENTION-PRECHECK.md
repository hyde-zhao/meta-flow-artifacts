---
checkpoint_id: "CP5-CR120"
checkpoint_name: "CR Tracking Row Convention Implementation Authorization Precheck"
type: "auto_precheck"
status: "PASS"
owner: "host-orchestrator"
created_at: "2026-06-22T19:54:11+08:00"
checked_at: "2026-06-22T19:54:11+08:00"
target:
  phase: "story-planning"
  story_id: "STORY-CR120-CR-TRACKING-ROW-CONVENTION"
  artifacts:
    - "process/changes/CR-120-CR-TRACKING-FORMAL-FU-ROW-CONVENTION-HARDENING-2026-06-22.md"
    - "process/context/CP5-CR120-CR-TRACKING-ROW-CONVENTION-CONTEXT.yaml"
manual_checkpoint: "process/checkpoints/CP5-CR120-CR-TRACKING-ROW-CONVENTION-IMPLEMENTATION-REVIEW.md"
---

# CP5 CR120 CR Tracking Row Convention Implementation Authorization 预检

## Entry Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| CP2 已 approved | PASS | `process/checkpoints/CP2-CR120-CR-TRACKING-ROW-CONVENTION-REVIEW.md` | 用户已 approve scope / staged route。 |
| owner discovery 完成 | PASS | `process/docs/design/CR-TRACKING-FORMAL-FU-ROW-CONVENTION-HARDENING-CR120.md` | 推荐 owner 为 `../meta-flow/meta_flow/checks/cr_tracking.py`。 |
| CP5 context ready | PASS | `process/context/CP5-CR120-CR-TRACKING-ROW-CONVENTION-CONTEXT.yaml` | read_profile=minimal。 |
| 未实施源码/tests 修改 | PASS | git status / 当前工作区 | CP5 approve 前未新增 CR120 source/tests diff。 |

## Checklist

| # | 检查项 | 状态 | 证据 | 处理意见 |
|---|---|---|---|---|
| 1 | 最小实现 owner 明确 | PASS | owner discovery | 推荐修改 `../meta-flow/meta_flow/checks/cr_tracking.py`。 |
| 2 | 最小测试 owner 明确 | PASS | owner discovery | 推荐新增 `../meta-flow/tests/test_cr120_cr_tracking_row_convention.py`。 |
| 3 | 实现范围足够小 | PASS | CP5 Decision Brief | 仅覆盖 formal CR / FU row convention fixture，不做全量历史扫描。 |
| 4 | 现有 CR118 / CR119 diff 风险已暴露 | PASS | CP2 DQ-04 | 后续若改 checker，需回归 CR118 / CR119 tests。 |
| 5 | runtime 不授权 | PASS | CP5 Decision Brief | 不连接 runtime / NAS / credentials / trading / publish。 |

## Exit Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| 可发起 CP5 人工门禁 | PASS | 本预检 | 可以请求用户确认最小实现切片。 |
| 无阻断项 | PASS | 本预检 | 0 blockers。 |

## Deliverables

| 交付物 | 路径 | 状态 | 说明 |
|---|---|---|---|
| CP5 context capsule | `process/context/CP5-CR120-CR-TRACKING-ROW-CONVENTION-CONTEXT.yaml` | PASS | ready。 |
| CP5 review draft | `process/checkpoints/CP5-CR120-CR-TRACKING-ROW-CONVENTION-IMPLEMENTATION-REVIEW.md` | PASS | 待人工审查。 |
| owner discovery note | `process/docs/design/CR-TRACKING-FORMAL-FU-ROW-CONVENTION-HARDENING-CR120.md` | PASS | 已更新。 |

## 结论

- 结论：`PASS`
- 阻断项：0
- 豁免项：0
- 下一步：发起 CP5 人工确认；`approve` 才授权最小 source/tests/checker implementation 切片。

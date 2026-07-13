# CP1 — CR168 Use Case / Requirement / Scenario 完备性

## Entry Criteria

- CP0 已 `PASS`。
- CR168 产品文档处理决策已在正式 CR 中记录。
- 用户已批准 `meta-pm inline-fallback`；handoff 与 dispatch ledger 可追溯。

## Checklist

| 检查项 | 结果 | 精确结论 |
|---|---|---|
| Use Case | PASS | 新增 1 个 UC-58-CR168；用户、触发、9 类输入、处理、输出、旅程、排除和 claim ceiling 完整。 |
| 用户可见 SGQ | PASS | 1 条 confirmed；5 条方法/边界决策转入 CP2。 |
| Requirements / QAC | PASS | REQ 9 项；QAC 15 项；无“尽量/尽可能/不少于”类模糊目标。 |
| Scenarios | PASS | 17 项：P0=16、P1=1；10/10 C3 输入 fail-closed 类别独立覆盖，另有 1 个 projection reason-escape guard 场景。 |
| Test Matrix | PASS | 17/17 映射；missing scenario=0；missing requirement ref=0。 |
| Scope boundary | PASS | Gate 4=C3+C4；C4 unavailable 映射为 absent-no-na-reason，reason 逃逸由 projection 阻断；canonical Gate 4、C4/FU-007/event/真实 TCA/runtime/Stage3 保持范围外。 |
| Planning boundary | PASS | 仅 5 个产品 outcome 候选；正式 CR168 Story/LLD=0；设计文档修改=0。 |
| Baseline traceability | PASS | 8 个产品文档 + 当前需求基线增量更新；旧基线与修订记录保留。 |
| Authorization | PASS | 真实数据/TCA/calibration/runtime/trading/remote write 授权数=0。 |
| 文档质量 | PASS | YAML/JSON 解析、引用、scenario/matrix、`git diff --check` 均通过；质量文档目标路径为 `docs/quality/`。 |

## Exit Criteria

- 产品范围足以形成 CP2 Decision Brief。
- 5 个开放问题均有推荐、备选、影响和切换条件。
- `ready_for_design=false`，必须等待 CP2 人工批准。

## Deliverables

- 机器真相源：`process/checks/CP1-CR168-USE-CASE-COMPLETENESS.result.json`
- 产品证据：`process/checks/CR168-PRODUCT-BASELINE-EVIDENCE.json`
- 上下文胶囊：`process/context/CP1-CR168.context.json`
- 场景讨论恢复点：`process/checks/CP2-CR168-DISCUSSION-CHECKPOINT.json`

## 结论

`PASS`。下一步只能生成 CP2 context、自动预检、Decision Brief 与人工 checklist；不得进入 HLD/CP3、Story、LLD、实现或验证。

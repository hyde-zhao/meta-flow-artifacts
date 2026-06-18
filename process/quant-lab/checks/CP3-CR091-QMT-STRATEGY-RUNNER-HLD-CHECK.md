# CP3-CR091 HLD Check

## Entry Criteria

| 条目 | 结果 | 证据 |
|---|---|---|
| HLD 已生成 | PASS | `docs/qmt/CR091-QMT-STRATEGY-RUNNER-HLD.md`。 |
| 研究笔记已生成 | PASS | `docs/qmt/CR091-RUNNER-RESEARCH-NOTES.md`。 |
| 本地静态证据已审阅 | PASS | 多因子合同、order intent、strategy base、QMT client / gateway contract 已静态阅读。 |
| 外部参考只读研究已完成 | PASS | lite-qmt-executor、qmt-gateway、xqshare、vnpy_qmt 仅静态阅读。 |

## Checklist

| 检查项 | 结果 | 说明 |
|---|---|---|
| 候选方案对比完整 | PASS | HLD 覆盖 clean-room、lite-qmt-executor、gateway/proxy、vn.py、手工脚本五类方案。 |
| 推荐方案明确 | PASS | 推荐 clean-room package-driven runner。 |
| 多因子优先且不排斥其他策略 | PASS | StrategyAdapter 层同时覆盖多因子、legacy strategy、future package。 |
| 集成契约显式 | PASS | HLD 第 6 节覆盖调用方向、输入、输出、降级和调用方修改。 |
| 前置校验和失败路径明确 | PASS | HLD 第 8 节定义 fail closed 行为。 |
| 下单 / runtime 边界明确排除 | PASS | HLD 第 7、10、13 节和不授权清单覆盖。 |

## Exit Criteria

| 条目 | 结果 |
|---|---|
| CP3 checkpoint 已生成 | PASS |
| HLD 支持人工决策 | PASS |
| 方案不依赖未授权 runtime | PASS |

## Deliverables

- `docs/qmt/CR091-QMT-STRATEGY-RUNNER-HLD.md`
- `process/checkpoints/CP3-CR091-QMT-STRATEGY-RUNNER-HLD-REVIEW.md`
- `process/context/CP3-CR091-QMT-STRATEGY-RUNNER-CONTEXT.yaml`

## 结论

`PASS_WITH_RISK`：HLD 可进入人工确认。剩余风险是方案尚未实现，且未来任何真实 trading host smoke 都必须另行逐 run 授权。


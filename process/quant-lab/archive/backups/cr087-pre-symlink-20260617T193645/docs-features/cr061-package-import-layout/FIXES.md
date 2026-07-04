# CR061 Batch B Fixes

## Fix Inputs

当前无 BLOCKER / HIGH / MEDIUM 缺陷需要回修。

## Accepted Risks

| Risk ID | Severity | 内容 | 处理 |
|---|---|---|---|
| R-CR061B-V01 | MEDIUM | full pytest 未运行。 | 进入 CP8 风险接受或后续扩大离线回归。 |
| R-CR061B-V02 | LOW | `trading` 仅验证 package-level namespace alias。 | 后续 bulk relayout 前重新设计。 |
| R-CR061B-V03 | LOW | `quant_lab.market_data` alias 继承 legacy init 行为。 | 当前 smoke 通过，保留观察。 |

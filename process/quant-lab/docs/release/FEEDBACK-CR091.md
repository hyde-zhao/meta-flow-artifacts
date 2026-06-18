# CR091 Feedback

> release_decision: `READY_WITH_RISK`
> profile: `compact`
> generated_at: `2026-06-18T15:24:12+08:00`

## 反馈入口

CR091 CP8 之后的反馈不得直接扩展当前交付范围。反馈需要先分类，再决定是否创建后续 CR。

| 反馈类型 | 分流 |
|---|---|
| 离线 runner 合同缺陷 | 回到 CR091 回修，重新 CP6 / CP7。 |
| 真实 QMT / MiniQMT / XtQuant / gateway / runner 验证需求 | 新建 runtime authorization gate。 |
| NAS package exchange / publish 需求 | 新建 NAS package exchange gate。 |
| submit / cancel / simulation / live 需求 | 新建高风险 order-write CR。 |
| 历史 CR tracking 旧账 | 新建 ledger hygiene / governance 任务。 |
| evidence 落盘目录或保留策略 | 新建 evidence output gate 或纳入 runtime gate。 |

## 观察信号

| 信号 | 阈值 | 处理 |
|---|---|---|
| CR091 contract tests 失败 | 任一失败 | 回到实现回修。 |
| offline checker `passed=false` | 任一失败 | 回到实现或 fixture 修复。 |
| forbidden counters 非 0 | 任一非 0 | 阻断交付或回修。 |
| 新增真实 runtime / NAS / credential 入口 | 任一入口 | 阻断并要求独立授权。 |
| 用户要求真实 smoke | 任一明确要求 | 新建 runtime authorization gate，不在 CP8 静默执行。 |

## 后续候选

| 候选 ID | 类型 | 状态 | 说明 |
|---|---|---|---|
| CR091-FU-01 | runtime_authorization | candidate-not-started | 真实只读 runner smoke，必须逐 run 授权。 |
| CR091-FU-02 | nas_package_exchange | candidate-not-started | NAS package exchange / pull / publish。 |
| CR091-FU-03 | order_write | candidate-not-started | submit/cancel 或 simulation/live，独立高风险 CR。 |
| CR091-FU-04 | ledger_hygiene | candidate-not-started | CR025 / CR019 历史台账旧账治理。 |

## 不授权提醒

本反馈文档不是执行授权。任何涉及真实运行、NAS、凭据、账户、交易写、simulation/live、provider/lake/publish 的动作，都必须由用户在独立门禁中明确授权。

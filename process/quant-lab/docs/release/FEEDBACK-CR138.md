---
status: "ready_with_risk"
release_decision: "READY_WITH_RISK"
release_artifact_profile: "compact"
change_id: "CR-138"
created_at: "2026-06-24T18:58:00+08:00"
---

# CR138 Feedback

## 反馈入口

CR138 当前关闭后，后续反馈按以下类别分流。反馈记录不等于启动正式 CR，也不授权真实 runtime 或交易动作。

| 类别 | 处理 |
|---|---|
| Fixture / contract bug | 可进入轻量修复 CR，继续禁止真实 runtime。 |
| Runtime smoke 需求 | 进入 CR138-FU-RT-01 或新 runtime_authorization gate。 |
| QMT terminal direct-run 需求 | 进入 CR138-FU-RT-02。 |
| Order-write / simulation / live 需求 | 进入 CR138-FU-TRD-01，高风险授权门。 |
| CR tracking warning cleanup | 进入 CR138-FU-GOV-01，低优先级治理项。 |

## 后续候选

| Candidate | 状态 | 不授权边界 |
|---|---|---|
| CR138-FU-RT-01 Gateway xtquant adapter Spike / readonly integration design | candidate | 不自动授权 runtime、凭据、账户读取或交易。 |
| CR138-FU-RT-02 QMT terminal direct-run validation authorization gate | candidate | 不自动授权 QMT terminal 运行。 |
| CR138-FU-TRD-01 Order-write / simulation / live design authorization gate | candidate | 不自动授权 submit/cancel/simulation/live。 |
| CR138-FU-GOV-01 CR tracking warning cleanup | candidate | 不涉及 runtime。 |

## 观察信号

| 信号 | 触发处理 |
|---|---|
| 后续代码直接导入 `xtquant` | 阻断并要求 runtime_authorization。 |
| 测试读取 `.env` / 凭据 | 阻断并要求安全回修。 |
| 文档声称 runtime-ready / trade-ready | 阻断并修正文档。 |
| no-real-operation counters 非 0 | 阻断后续 runner/gateway 开发门。 |

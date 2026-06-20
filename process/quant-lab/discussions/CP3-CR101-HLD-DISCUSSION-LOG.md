# CP3 CR101 HLD Discussion Log

## 元信息

| 字段 | 内容 |
|---|---|
| CR | CR-101 |
| 阶段 | CP3 HLD |
| 生成时间 | 2026-06-20T08:33:18+08:00 |
| 主持 | host-orchestrator |
| 子 agent 调度 | `meta-se/se-chu` via `multi_agent_v1.spawn_agent`，agent_id `019ee26c-b0d6-7c51-bcd2-b49b61310de9`；两轮等待未落盘，随后关闭并由主进程收敛。 |

## 输入边界

| 输入 | 路径 | 读取策略 | 结论 |
|---|---|---|---|
| CP2 approved review | `process/checkpoints/CP2-CR101-CROSS-PLATFORM-STRATEGY-DELIVERY-SCOPE-REVIEW.md` | must_read | CP2 已批准，不授权真实系统。 |
| 当前需求基线 | `process/baseline/CURRENT-REQUIREMENT-BASELINE.yaml` | must_read | 当前策略交付 target 为 QMT direct-run；MiniQMT 是 adapter target。 |
| CR101 | `process/changes/CR-101-CROSS-PLATFORM-STRATEGY-DELIVERY-ADAPTER-REALIGNMENT-2026-06-20.md` | must_read | 本 CR 只做离线重对齐。 |
| CR091 HLD | `docs/qmt/CR091-QMT-STRATEGY-RUNNER-HLD.md` | read_if_needed | runner 是 quant-lab 侧 clean-room package-driven runner。 |
| CR098 HLD | `docs/qmt/CR098-QMT-RUNNER-READONLY-GATEWAY-INTEGRATION-HLD.md` | read_if_needed | readonly gateway 是 runner-owned adapter facade。 |
| CR100 HLD | `docs/qmt/CR100-NAS-PACKAGE-EXCHANGE-OFFLINE-READINESS-HLD.md` | read_if_needed | fake exchange manifest 中 `miniqmt_runner` 是需迁移的历史字段。 |
| strategy_runner 代码 | `trading/strategy_runner/*` | read_if_needed | 当前存在 AdapterRegistry、ReadonlyGatewayClient、PackageLoader、EvidenceSummary、fake exchange。 |

## Architecture Gray Areas

| 灰区 ID | 讨论问题 | 选项 | 推荐 | 处理状态 |
|---|---|---|---|---|
| AGA-CR101-01 | target taxonomy 用 QMT-only 还是 cross-platform？ | QMT-only / cross-platform with QMT current / abstract-only | cross-platform with `qmt_terminal_direct` current | decision-item |
| AGA-CR101-02 | MiniQMT 是否为 runner host？ | adapter target / runner host / dual-runner | adapter target | decision-item |
| AGA-CR101-03 | manifest/checker 是否继续使用 `miniqmt_runner`？ | 保留 / 迁移 / 文档勘误 | 迁移到 target + adapter | decision-item |
| AGA-CR101-04 | CP3 是否授权真实 runtime evidence？ | 不授权 / 用户手工 / agent 代跑 | 不授权 | decision-item |

## Advisor Table

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| A. Target + adapter 双边界 | 清晰、可扩展、符合 CP2 | 需要迁移 schema 和 checker | manifest、checker、fixture、evidence、docs | 推荐 | 若 QMT direct-run target 无法离线建模，转 Spike。 |
| B. 沿用 `miniqmt_runner` | 变更小 | 继续混淆 runner host 和 adapter | CR100 package exchange、runner、future runtime | 不推荐 | 仅在独立 CR 决定 MiniQMT host runner 时切换。 |
| C. 抽象-only | 权限最小 | 不可验证，不能解除阻塞 | docs、future CR | 不推荐 | 用户要求暂停实现时采用。 |

## 分类结果

| 问题类型 | 数量 | 处理 |
|---|---:|---|
| decision-item | 5 | 纳入 CP3 待人工决策清单。 |
| non-blocking-open | 1 | Goldminer / generic target 只保留 future slot。 |
| n/a-with-reason | 6 | NAS、凭据、runtime、simulation/live、交易、publish 均未授权。 |
| blocking-open | 0 | 无 CP3 阻断开放项。 |

---
id: "CR-144"
title: "Experiment Real Lake Remediation"
status: "ready-for-final-validation"
kind: "requirement-change"
lifecycle_status: "active"
readiness_status: "ready_with_risk"
gate_profile: "compact"
created_at: "2026-06-30T23:00:00+08:00"
created_by: "host-orchestrator"
source_decision_id: "USER-20260630-EXPERIMENT-REAL-LAKE-REMEDIATION"
authorization_summary: "用户授权完成 experiment 所有整改，包括必要的数据湖读取、写入、修复、发布和验证操作。"
not_authorized:
  - "qmt_miniqmt_xtquant_gateway_runtime"
  - "simulation_live_or_trading"
  - "provider_fetch_without_explicit_command_boundary"
---

# CR-144: Experiment Real Lake Remediation

## 背景

`process/context/EXPERIMENT-OFFLINE-TO-REAL-LAKE-HANDOFF-2026-06-30.md` 将 E4 定义为真实 lake readonly semantic validation。用户在本轮扩大授权，要求完成 experiment 全部整改，并授权必要的数据湖读写操作。

## 整改计划

| Phase | 目标 | 退出条件 |
|---|---|---|
| Phase 1 | 补齐 experiment 真实湖输入合同 | experiment 15/16/17_21/23_29 均可通过 `--lake-root` 与 `--as-of` 读取 PIT lake 输入；离线测试通过 fixture 注入而非 CLI 本地目录绕行。 |
| Phase 2 | 离线回归 | experiment targeted tests、lake onboard no-bypass 测试和 full pytest 通过。 |
| Phase 3 | 真实 lake smoke | 检查 lake root、catalog、prices/index_members/trade_calendar 可用性；运行 turnover、15、16、17_21、23_29 的 bounded smoke。 |
| Phase 4 | 差异驱动修复 | 若 smoke 暴露 catalog 未发布、quality fail/warn、canonical 缺失、PIT available_at 缺失或字段语义不匹配，按最小范围执行 revalidate/repair/publish/backfill，并复跑对应 smoke。 |
| Phase 5 | 证据收口 | 输出真实 lake 差异表、experiment smoke report、后续生产化候选与剩余风险。 |

## 授权边界

- 已授权：读取真实 lake；必要时写入 canonical/quality/catalog/published 相关数据湖对象；执行数据湖修复、revalidate、publish 或同步命令。
- 仍不默认触碰：QMT/MiniQMT/xtquant/gateway runtime、simulation/live/trading。
- provider fetch 仅在 lake 缺口明确要求且命令边界、dataset、date range、source/interface 已在执行记录中明确时使用。

## 当前执行记录

- Phase 1 启动：补齐 experiment 16 与 17_21 的 `--lake-root` / `--as-of` 输入合同。
- Phase 1 完成：补齐 experiment 16/17_21 lake 输入合同；修复 experiment lake PIT key，保留 `trade_date + symbol` 时间序列。
- Phase 3 完成：真实 lake smoke 已覆盖 turnover、experiment 15、16、17_21、23_29。
- Phase 4 判定：无需执行 canonical/quality/catalog 写入；阻断差异均通过 source 合同修复解决。
- Review supplement 完成：修复 `index_members` canonical PIT key 漂移、turnover `--output-root` 单传过程目录派生、`read_panel_as_of` 重复排序列和 experiment 15 不可达 legacy 回退路径；targeted 回归 `30 passed`。

## 当前结论

E4 真实 lake 研究 smoke 与评审后合同回归通过。结果只表示 experiment 研究入口可消费当前真实 lake 形状，不表示 production admission、simulation、live、QMT 或交易准入。

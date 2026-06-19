---
id: CR-100
title: NAS Package Exchange Offline Readiness Gate
status: closed-current-delivery
created_at: "2026-06-19T18:10:00+08:00"
closed_at: "2026-06-19T18:45:00+08:00"
owner: host-orchestrator
source_candidate: CR091-FU-02
parent_cr: CR-091
workflow_mode: standard
impact_level: medium
rollback_to: "CR091 closed-current-delivery / READY_WITH_RISK; CR091-FU-02 candidate"
approval: "User authorized completing all non-real-NAS delivery and pushing project remote on 2026-06-19."
---

# CR-100 NAS Package Exchange Offline Readiness Gate

## 变更摘要

CR100 将 `CR091-FU-02` 从 follow-up candidate 转为正式 CR，目标是在当前真实 NAS 不可连接且未授权访问 NAS 的条件下，交付策略包 exchange 的离线 readiness：

- 定义 research_pc -> package exchange -> trading_pc local immutable cache 的离线合同。
- 实现本地 fake exchange root，要求 `.cr100_fake_exchange_root` marker 后才允许 fake publish / fake pull。
- 实现 CR100 package manifest / schema / hash / approval / permission fail-closed 检查。
- 实现本地 fake publish / fake pull / check CLI：`scripts/cr100_package_exchange.py`。
- 实现 pytest fixture，覆盖成功路径与 marker 缺失、checksum drift、未批准包、`.env` / secret marker 等失败路径。
- 输出真实 NAS 恢复后的人工 runbook，但本轮不执行真实 NAS 操作。

## 五维度影响分析

| 维度 | 是否影响 | 结论 |
|---|---:|---|
| 需求层 | 是 | 新增离线 readiness gate，不声明真实 NAS ready。 |
| 场景层 | 是 | 覆盖 fake exchange publish/pull/check、manifest 校验、hash mismatch fail closed、unapproved fail closed、forbidden file fail closed。 |
| 计划层 | 是 | CR091-FU-02 转正式 CR100；CR091-FU-03、CR089、CR020 route restore 不启动。 |
| 安全层 | 是 | 不授权 NAS、凭据、runtime、交易和 provider/lake/catalog publish；所有真实链路后置独立 gate。 |
| 交付层 | 是 | 新增 `trading/strategy_runner/package_exchange.py`、`scripts/cr100_package_exchange.py`、测试、HLD、runbook 与过程证据。 |

## 文档处理决策

| 文档 / 对象 | 处理方式 | 旧基线保留方式 | 说明 |
|---|---|---|---|
| `process/changes/CR-091-FOLLOW-UP-TRACKING-2026-06-18.md` | 原文档更新 | 保留原候选行，改为 closed 并链接 CR100 | 记录候选转正式交付。 |
| `process/changes/CR-INDEX.yaml` | 原文档更新 | 保留 CR091 / CR089 历史行，新增 CR100 closed 行 | 状态索引同步。 |
| `process/STATE.md` | 原文档更新 | 保留历史状态，更新 last_action / next_action / cr_tracking | 当前无 active formal CR。 |
| `docs/qmt/CR100-NAS-PACKAGE-EXCHANGE-OFFLINE-READINESS-HLD.md` | 新增 | N/A | CR100 架构与边界。 |
| `docs/qmt/CR100-NAS-PACKAGE-EXCHANGE-RECOVERY-RUNBOOK.md` | 新增 | N/A | NAS 恢复后人工验证步骤；不含真实路径 / 凭据。 |
| `docs/quality/CR100-*` | 新增 | N/A | 验证、测试、评审报告。 |

## 不授权范围

- 不访问、列取、读取、复制、拉取、写入、发布或删除真实 NAS。
- 不读取 Windows/Linux `.env`、NAS 凭据、token、secret、账号、账户、资金、持仓、委托、成交或原始日志。
- 不启动、连接、安装或运行 QMT / MiniQMT / XtQuant / gateway / runner runtime。
- 不执行 submit/cancel、buy/sell、simulation/live。
- 不执行 provider fetch、lake write、catalog publish。
- 不自动恢复 CR020 用户删除的 gateway 路线，不自动启动 CR089。

## 验收标准

| AC | 验收项 | 状态 | 证据 |
|---|---|---|---|
| CR100-AC-01 | fake exchange root 必须有 marker；缺 marker fail closed | PASS | `tests/test_cr100_package_exchange.py` |
| CR100-AC-02 | package manifest 必须包含目标平台、entrypoints、approval、hashes 和权限 false flags | PASS | `validate_package` |
| CR100-AC-03 | checksum drift / unapproved package / `.env` 或 secret marker fail closed | PASS | 聚焦 pytest |
| CR100-AC-04 | fake publish / fake pull 只写本地临时目录，生成 immutable cache 与 active pointer | PASS | 聚焦 pytest |
| CR100-AC-05 | CLI 不导入 runtime、network、env 模块 | PASS | AST 测试 |
| CR100-AC-06 | 真实 NAS 恢复验证只形成 runbook，本轮不执行 | PASS_WITH_RISK | `docs/qmt/CR100-NAS-PACKAGE-EXCHANGE-RECOVERY-RUNBOOK.md` |

## 关闭结论

CR100 当前交付关闭为 `closed-current-delivery / READY_WITH_RISK`。风险为真实 NAS 链路未验证；后续若要执行真实 NAS publish / pull / copy / 校验，必须另起独立授权 gate。

---
cr_id: "CR-127"
title: "State v2 / Process Retention Policy Landing"
cr_type: "process"
cr_kind: "ledger-maintenance"
lifecycle_status: "closed"
readiness_status: "ready"
gate_status: "closed"
gate_profile: "compact"
status: "closed-current-delivery"
impact_level: "medium"
workflow_mode_before: "standard"
workflow_mode_after_change: "standard"
rollback_to: "process/archive/state/STATE.legacy-before-state-v2-2026-06-23.md"
approval_result: "user-requested-implementation"
created_at: "2026-06-23T15:08:00+08:00"
created_by: "host-orchestrator"
approved_by: "user"
approved_at: "2026-06-23T15:08:00+08:00"
source: "user-request"
owner: "host-orchestrator"
acceptance_criteria: "STATE.current.json exists; process policies exist; STATE.md is compact human summary; full-doc read expansion is logged and checkable; governance and CR tracking checks pass."
close_condition: "All state v2, policy and read-expansion checks pass with process/STATE.md no longer used as default machine state."
conflict_keys: ["state-v2", "process-retention-policy", "read-policy", "state-md-legacy-summary"]
impact_surface: ["process/state", "process/policies", "process/context", "process/STATE.md", "cr-tracking"]
authz_policy_refs: ["NO_CREDENTIAL_READ", "NO_RUNTIME", "NO_NAS", "NO_TRADING", "NO_REMOTE_WRITE"]
risk_refs: []
cr_index_path: "process/changes/CR-INDEX.yaml"
historical_baseline_status: "active"
---

# CR-127 State v2 / Process Retention Policy Landing

## 变更描述

用户要求把分区整改收口，目标是把“少读大文档”从规则变成可检查的过程文件和默认入口。

本 CR 只处理过程治理对象：`process/state`、`process/policies`、`process/context` 和 `process/STATE.md` 摘要。它不修改业务源码，不启动 runner、runtime、NAS、QMT、凭据、provider、lake、catalog、Git 写入或交易动作。

## 当前问题

整改前存在三个核心缺口：

1. 缺少 `process/state/STATE.current.json`，机器状态仍只能回退到巨型 `process/STATE.md`。
2. 缺少 `process/policies/SOURCE-OF-TRUTH-MAP.yaml`、`RETENTION-POLICY.json` 和 `READ-POLICY.json`，默认读取边界无法被检查器稳定验证。
3. `process/STATE.md` 约 2.1M，仍承担 legacy 大文档角色，不符合“人类摘要 / legacy fallback”的目标形态。

## 文档处理决策

| 受影响文档 | 处理方式 | 旧基线保留方式 | 修订记录位置 | 批准状态 |
|---|---|---|---|---|
| `process/STATE.md` | 原文档更新 | 旧全文归档到 `process/archive/state/STATE.legacy-before-state-v2-2026-06-23.md`；新文件由 `meta-flow state render` 生成轻量人类摘要 | 本 CR | approved |
| `process/state/STATE.current.json` | 新增 | 从 legacy `process/STATE.md` 迁移最小机器状态 | 本 CR | approved |
| `process/state/*.ndjson` | 新增 | 作为 CR、Story、Checkpoint、Handoff、Dispatch、Gate、Run、Read Expansion 的 append-only ledger 入口 | 本 CR | approved |
| `process/policies/SOURCE-OF-TRUTH-MAP.yaml` | 新增 | 机器真相源策略；`process/STATE.md` 明确为 generated_summary / deny-default | 本 CR | approved |
| `process/policies/RETENTION-POLICY.json` | 新增 | 定义 active CR、closed CR、Story packet、CP result、ledger、audit appendix 的默认上下文保留规则 | 本 CR | approved |
| `process/policies/READ-POLICY.json` | 新增 | 定义默认 entrypoint、deny-default reads、summary reads、full-doc read allowed reasons 和 read expansion ledger | 本 CR | approved |

## 五维度影响分析

| 维度 | 评估问题 | 受影响对象 | 结论 | 处理动作 |
|------|----------|-----------|------|---------|
| 需求层 | 是否新增、删除或重定义 REQ-* | 无产品需求对象 | false | 不改 `docs/product/*`。 |
| 场景层 | 是否改变测试矩阵覆盖范围 | 无业务测试矩阵 | false | 不改 `SCENARIOS.yaml` / `TEST-MATRIX.md`。 |
| 计划层 | 是否改变 Phase、Wave、Story / 任务依赖 | 过程状态读取入口 | true | 新增 state v2 与 policy 文件；不改变业务 Story DAG。 |
| 安全层 | 是否引入新的高风险动作或权限要求 | 读取边界 / deny-default policy | true | 明确 `process/STATE.md`、完整 CR、LLD、实现证据和质量报告默认禁读；全文读取必须写 read expansion ledger。 |
| 交付层 | 是否需要重新生成交付物或回归子集 | process governance checks | true | 运行 state、governance、CR tracking、read-log 和 context 检查。 |

## 回退决策

- 影响范围：局部过程治理。
- 回退到阶段：`rollback_to=process/archive/state/STATE.legacy-before-state-v2-2026-06-23.md`。
- 需要重新确认的对象：若 `STATE.current.json` 或 policies 与后续工具不兼容，恢复归档 `STATE.md`，并将本 CR 标记为 `ready_with_risk` 或重新打开修复。

## fast-lane 判定

| 条件 | 是否命中 | 说明 |
|---|---|---|
| 仅低风险轻量实现 / 文档 / 规则修改 | true | 只写 process governance 文件和摘要。 |
| 修改架构、权限、安全边界或平台安装路径 | false | 不改平台路径；读取边界收敛为已有规则落地。 |
| 修改外部接口契约、文件所有权或多 Story 依赖 | false | 不涉及业务接口、源码或 Story 实现。 |
| 需要 HLD / LLD 才能解释影响 | false | 过程治理 CR 可由现有 state/context/governance 契约解释。 |
| 是否保持 fast-lane | true | 使用 compact/process-lite 门禁。 |

## LLD 设计批次门禁

- 是否需要 LLD 设计批次：false
- 理由：本 CR 不影响业务 Story、LLD、接口契约、文件所有权、`dev_gate` 或实现设计。

## 执行结果

| 验收项 | 状态 | 证据 |
|---|---|---|
| 生成 `process/state/STATE.current.json` | done | `meta-flow state migrate-v2 --force` |
| 生成 base ledgers | done | `process/state/*-LEDGER.ndjson` |
| 生成 source-of-truth policy | done | `meta-flow governance init` |
| 生成 retention policy | done | `meta-flow governance init` |
| 生成 read policy | done | `process/policies/READ-POLICY.json` |
| 降级 `process/STATE.md` 为人类摘要 | done | `meta-flow state compact --force` |
| 保留旧全文 | done | `process/archive/state/STATE.legacy-before-state-v2-2026-06-23.md` |
| 启用检查 | done | state / governance / cr-tracking / read-log / context checks |

## 不授权范围

- 不读取 `.env`、token、secret、账号、账户、资金、持仓、委托、成交、session、cookie 或原始日志。
- 不访问、列取、挂载、读取、复制、写入、发布、pull 或删除真实 NAS。
- 不启动、连接、安装或运行 QMT / MiniQMT / XtQuant / gateway runtime。
- 不执行 submit/cancel、buy/sell、simulation/live。
- 不执行 provider fetch、lake write、catalog publish 或 current pointer 修改。
- 不修改业务源码、不运行 runner、不修改 `.gitignore`。
- 不执行 commit、push 或 remote write。

## 处理结论

- 审批结论：user-requested-implementation。
- 关闭结论：closed-current-delivery / ready。
- 后续影响：先前未启动的“CR127 Runner Core MVP Implementation”标签已被本治理 CR 占用；如继续 runner core，应使用下一个可用 CR 编号。

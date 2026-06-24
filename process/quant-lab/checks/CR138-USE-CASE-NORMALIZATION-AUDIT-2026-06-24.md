---
cr_id: "CR-138"
title: "CR138 Use-Case Normalization Audit"
status: "PASS_WITH_RISK"
created_at: "2026-06-24T13:10:00+08:00"
created_by: "host-orchestrator"
scope: "UC-01..UC-50"
runtime_authorized: false
---

# CR138 Use-Case Normalization Audit

## 目标

回应用户反馈：在发起 CP2 前，再检查 `process/USE-CASES.md` 中其他场景是否需要合并和刷新。

## 检查范围

| 范围 | 说明 |
|---|---|
| UC-01..UC-09 | 本地研究、数据准备、数据湖早期基线 |
| UC-10..UC-18 | QMT foundation、阶段激活、复权隔离、阶段六准入、QMT C/S / FastAPI、后置能力 |
| UC-19..UC-27 | Backtrader optional backend、多因子研究闭环、研究输出合同 |
| UC-28..UC-32 | QMT / MiniQMT 双目标策略交付框架 |
| UC-33..UC-50 | CR138 Runner / QMT Gateway 运营 use-case baseline |

## 结论

| 结论 | 状态 | 说明 |
|---|---|---|
| 不删除旧 UC、不重编号 | PASS | UC-01..UC-32 中多项已有历史 CP2 / CR 追溯，破坏性合并会丢失决策链。 |
| 需要新增归一化映射 | PASS | UC-16/17 与 UC-44..50、UC-28..31 与 UC-33..43 存在边界相邻，已在 `process/USE-CASES.md` 新增“CR-138 场景归一化与合并审查”。 |
| UC-33..UC-50 不做编号合并 | PASS | 内部可按活动簇聚合，但每个 UC 触发条件、风险级别和授权边界不同。 |
| 旧 UC-16/17 后续可压缩 | PASS_WITH_RISK | 旧 FastAPI / endpoint 场景较长，后续 HLD 确认 Gateway service boundary 后可另起文档清理 CR；本轮不破坏性重写已确认基线。 |
| 不授权 runtime / QMT / 凭据 / 账户 / 行情 / 交易 | PASS | 本次只更新 use-case 文档和过程审计，不执行运行或外部连接。 |

## 合并 / 刷新建议

| 对象 | 处理建议 | 理由 |
|---|---|---|
| UC-10、UC-11、UC-15 | 保留为治理和准入前置 | 这些是 runtime gate、OMS/risk/admission 约束，不是 CR138 的日常运营路径本身。 |
| UC-16、UC-17 | 保留为架构 / endpoint 来源；运营路径交给 UC-44..UC-50 | 防止 Gateway HLD 同时按“接口列表”和“用户运营路径”重复拆分。 |
| UC-28..UC-32 | 保留为双目标交付框架；Runner 运营由 UC-33..UC-43 承接 | CR046 是策略包与目标适配框架，CR138 是用户如何运行、观察、恢复和变更策略。 |
| UC-33..UC-43 | 不合并编号，按 Runner 活动簇聚合 | 计划、盘前、执行、触发、再平衡、观察、证据、复盘、恢复、策略生命周期变更需要不同验收。 |
| UC-44..UC-50 | 不合并编号，按 Gateway 活动簇聚合 | health、只读查询、行情订阅、交易回报、恢复、审计、配置变更的授权和风险不同。 |

## 剩余风险

| 风险 ID | 风险 | 状态 | 处理 |
|---|---|---|---|
| RISK-CR138-UC16-UC17-LONGFORM | UC-16 / UC-17 历史文本较长，后续 HLD 可能重复读取 endpoint 细节 | accepted | CP2 review 明确后续以 UC-44..UC-50 组织 Gateway 用户路径，UC-16/17 只作为架构来源。 |
| RISK-CR138-REQUIREMENTS-NOT-REFRESHED | USE-CASES 已归一化，但 REQUIREMENTS 未同步抽取 | controlled | 等用户 approve CP2 后再进入 requirement-extraction，避免草案提前升格。 |

## 结论

本轮归一化审查结论为 `PASS_WITH_RISK`。用户已于 2026-06-24T13:20:00+08:00 回复 `approve` 通过 CR138 CP2；本审查作为 CP3 solution-design 的输入。当前仍不授权任何 runtime、QMT、MiniQMT、XtQuant、凭据、账户、行情、交易、NAS、provider/lake/catalog 或 Git remote 写入。

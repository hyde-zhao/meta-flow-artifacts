---
cr_id: "CR-092"
status: "closed-current-delivery"
impact_level: "high"
workflow_mode_before: "standard"
workflow_mode_after_change: "standard"
fast_lane_upgrade_reason: "命中真实 QMT / MiniQMT / XtQuant / gateway / runner runtime 边界、逐 run 授权、脱敏 evidence、账户只读查询和安全权限；不得使用 fast-lane。"
rollback_to: "CR091 closed-current-delivery / READY_WITH_RISK; CR089 blocked-readiness-approved; no active formal CR"
approval_result: "approved / CP2 / CP3 / CP5 / CP8; CP6 PASS; CP7 PASS_WITH_RISK; closed-current-delivery / READY_WITH_RISK"
created_at: "2026-06-18T15:58:59+08:00"
created_by: "host-orchestrator"
approved_by: "user"
approved_at: "2026-06-18T17:25:00+08:00"
source: "cp8-follow-up"
linked_issue: ""
parent_cr: "CR-091"
source_checkpoint: "process/checkpoints/CP8-CR091-DELIVERY-READINESS.md"
source_decision_id: "DQ-CP8-CR091-04"
follow_up_type: "runtime_authorization"
risk_class: "trading-runtime-boundary"
owner: "host-orchestrator"
revisit_condition: "用户于 2026-06-18 在清上下文恢复后选择 CR091-FU-01（真实只读 runtime smoke 设计门禁）。"
acceptance_criteria: "形成真实只读 runtime smoke 的范围、授权模型、禁止动作、脱敏 evidence、失败回退和人工执行边界；CP2/CP3/CP5 approved 前不得运行任何 QMT / MiniQMT / XtQuant / gateway / runner。"
close_condition: "CP8 approved 后关闭为设计门禁交付；或用户取消 / 合并到 CR089 / 回退到 CR091 follow-up backlog。"
cr_index_path: "process/changes/CR-INDEX.yaml"
---

# CR092 Real QMT Readonly Runtime Smoke Design Gate

## 变更描述

用户从 CR091 CP8 后续候选中明确选择 `CR091-FU-01`：真实只读 runtime smoke 设计门禁。

本 CR 的目标不是立即执行真实验证，而是先把真实交易主机只读 smoke 的设计门禁落地，包括逐 run 授权、只读 scope、输入 / 输出契约、模拟账户 evidence、失败回退、禁止动作计数和人工审查点。任何真实运行、连接、真实账户读取或 evidence 落盘都必须在本 CR 后续人工门禁明确批准后才可进入，并且仍需逐 run 授权。用户明确提供的模拟账户测试证据可按 CP3 / CP5 安全边界读取。

## CP8 Follow-up 来源

| 字段 | 内容 |
|---|---|
| 父级 CR | `CR-091` |
| 来源检查点 | `process/checkpoints/CP8-CR091-DELIVERY-READINESS.md` |
| 来源决策 ID | `DQ-CP8-CR091-04` |
| follow-up 类型 | `runtime_authorization` |
| 风险等级 | `trading-runtime-boundary` |
| owner | `host-orchestrator` |
| 重访条件 | 用户选择 `CR091-FU-01` |
| 验收标准 | 先完成真实只读 runtime smoke 的设计和授权边界，不直接运行 |
| 关闭条件 | CP8 approved，或取消 / 合并 / 回退到 follow-up backlog |

## 冲突预检结论

| 对象 | 当前状态 | 影响面 | 冲突判断 | 处理方式 |
|---|---|---|---|---|
| `STATE.md.active_change` | 空 | 执行锁 | 无 active formal CR 锁 | 允许创建 `CR-092` |
| `CR-091` | `closed-current-delivery / READY_WITH_RISK` | offline runner 合同和 fixture 验证 | 可作为输入，不重开 CR091 | 只消费 CR091 产物和风险接受项 |
| `CR-089` | `blocked-readiness-approved` | QMT 接口验证、NAS package exchange、只读 smoke、运行授权 | 相关但仍 blocked；不自动启动 | CR092 只做 runtime smoke 设计门禁，不激活 CR089，不做 NAS |
| `CR-046` | `closed-current-delivery / READY_WITH_RISK` | 双目标策略交付框架、runner 安装设计 | 已关闭，可作为架构背景 | 不恢复 CR046，不扩大到策略交付 |
| `CR-020` | `closed-current-delivery`，用户删除的 gateway 路线归档 | 历史 QMT gateway | 不得恢复 | 仅作审计背景，不作为当前入口 |
| `CR019 / CR025` 旧账 | 全局 cr-tracking 存在历史不一致 | 台账治理 | 不阻塞本 CR，但会继续导致 `meta-flow check cr-tracking` exit 1 | 记录为非阻断，保留给 `CR091-FU-04` |

结论：允许启动 `CR-092`，但初始状态只能进入 `active-cp2-intake`。本 CR 创建不授权任何真实 runtime、NAS、凭据、真实账户原文、交易写、simulation/live、provider/lake/publish 或 CR089 自动启动。

## 文档处理决策

| 受影响文档 | 处理方式 | 旧基线保留方式 | 修订记录位置 | 批准状态 |
|---|---|---|---|---|
| `process/changes/CR-092-REAL-QMT-READONLY-RUNTIME-SMOKE-DESIGN-GATE-2026-06-18.md` | 新增 | CR091 follow-up 完整摘录与映射 | 本文件 | approved / CP2 / CP3 / CP5; CP7 PASS_WITH_RISK |
| `process/changes/CR-INDEX.yaml` | 原文档更新 | 保留 CR091 closed、CR089 blocked 和历史候选 | CR092 entry | approved / CP2 / CP3 / CP5; CP7 PASS_WITH_RISK |
| `process/STATE.md` | 原文档更新 | 保留 delivered 历史和 CR091 closure history | `active_change` / `cr_tracking` / `history` | approved / CP2 / CP3 / CP5; CP7 PASS_WITH_RISK |
| `process/docs/release/FEEDBACK-CR091.md` | 不变 | 保留 CR091-FU-01 candidate 来源 | 不适用 | n/a |
| `docs/qmt/CR092-REAL-QMT-READONLY-RUNTIME-SMOKE-HLD.md` | 新增 | 不替换 CR091 HLD；作为真实只读 smoke 设计门禁 | `## 修订记录` | approved |
| `process/stories/CR092-REAL-QMT-READONLY-RUNTIME-SMOKE-LLD.md` | 新增 | 不替换 CR091 LLD；限定人工执行边界 | `## 修订记录` | approved / CP5 |
| `docs/qmt/CR092-REAL-QMT-READONLY-RUNTIME-SMOKE-TEST-PLAN.md` | 新增 | 不替换 CR091 TEST-PLAN；限定真实只读 smoke 验证策略 | `## 修订记录` | approved / CP5 |
| `process/checkpoints/CP2-CR092-READONLY-RUNTIME-SMOKE-SCOPE-REVIEW.md` | 新增 | CP2 人工门禁 | 人工审查结果 | approved |
| `process/checkpoints/CP3-CR092-READONLY-RUNTIME-SMOKE-HLD-REVIEW.md` | 新增 | CP3 人工门禁 | 人工审查结果 | approved |
| `process/checkpoints/CP5-CR092-READONLY-RUNTIME-SMOKE-READINESS.md` | 新增 | CP5 设计 / 授权准备门禁 | 人工审查结果 | approved |
| `docs/qmt/CR092-REAL-QMT-READONLY-RUNTIME-SMOKE-MANUAL-GUIDE.md` | 新增 | CP6 manual guide | 本文件 / CP6 evidence | PASS |
| `docs/qmt/CR092-REAL-QMT-READONLY-RUNTIME-SMOKE-EVIDENCE-TEMPLATE.yaml` | 新增 | CP6 simulated evidence template | checker self-check | PASS |
| `scripts/check_cr092_simulated_evidence.py` | 新增 | CP6 single-file static checker | pytest / py_compile | PASS |
| `tests/test_cr092_simulated_evidence_checker.py` | 新增 | CP6 checker tests | pytest | PASS |
| `docs/quality/CR092-REAL-QMT-READONLY-RUNTIME-SMOKE-VERIFICATION-REPORT.md` | 新增 | CP7 验证报告 | CP7 | PASS_WITH_RISK |
| `process/checks/CP7-CR092-READONLY-RUNTIME-SMOKE-VERIFICATION-DONE.md` | 新增 | CP7 自动检查点 | CP7 | PASS_WITH_RISK |
| `process/release/RELEASE-CONTEXT.yaml` | 原文档更新 | CP8 发布上下文胶囊 | CP8 | ready / approved |
| `docs/release/RELEASE-NOTES.md` | 原文档更新 | CP8 发布说明 | CP8 | ready |
| `docs/release/DEPLOY-CHECKLIST.md` | 新增 | CP8 部署检查 | CP8 | ready |
| `docs/release/ROLLBACK.md` | 新增 | CP8 回滚方案 | CP8 | ready |
| `docs/release/MIGRATION.md` | 新增 | CP8 迁移说明 | CP8 | ready |
| `docs/release/FEEDBACK.md` | 新增 | CP8 反馈回流 | CP8 | ready |
| `process/checkpoints/CP8-CR092-DELIVERY-READINESS.md` | 新增 | CP8 人工门禁 | 人工审查结果 | approved |
| `process/changes/CR-092-FOLLOW-UP-TRACKING-2026-06-18.md` | 新增 | CP8 后续分流台账 | CP8 | open |

## 旧基线映射

| 原基线对象 | 新增 / 修改对象 | 保留策略 | 映射说明 |
|---|---|---|---|
| CR091 offline runner | CR092 real readonly runtime smoke design | 原文保留 | CR092 只承接真实只读验证缺口，不修改 CR091 关闭范围 |
| CR091 fake readonly gateway wrapper | CR092 real gateway / runner readonly smoke contract | 原文保留 | 从 fake transport 过渡到真实只读设计，仍禁止实际运行 |
| CR091 redacted evidence model | CR092 simulated-account runtime evidence gate | 原文保留 | CR092 必须定义模拟账户 evidence 字段、脱敏字段和敏感扫描门禁 |
| CR089 qmt_interface_smoke | CR092 readonly runtime smoke design reference | 原文保留 | 可复用只读 scope 思路，但不激活 CR089、NAS 或交易主机 |
| CR020 historical gateway route | N/A | 审计保留 | 用户删除路线不得恢复为当前入口 |

## 五维度影响分析

| 维度 | 评估问题 | 受影响对象 | 结论（true/false） | 处理动作 |
|------|----------|-----------|--------------------|---------|
| 需求层 | 是否新增、删除或重定义 REQ-* | 真实只读 runtime smoke、逐 run 授权、脱敏 evidence | true | 新增 CR092 设计门禁需求，不改变 CR091 关闭范围 |
| 场景层 | 是否改变测试矩阵覆盖范围 | runtime health、capabilities、readonly query、禁止动作计数、失败回退 | true | 生成 scoped TEST-PLAN，不运行真实环境 |
| 计划层 | 是否改变 Phase、Wave、Story / 任务依赖 | CR092 CP2 / CP3 / CP5 / CP8；可能后续 CP6 / CP7 | true | 先串行完成设计门禁；CP5 approved 前不得实施或运行 |
| 安全层 | 是否引入新的高风险动作或权限要求 | QMT / MiniQMT / XtQuant / gateway / runner、账户只读、凭据边界 | true | 默认全部禁止；必须逐 run 授权并列出不授权项 |
| 交付层 | 是否需要重新生成交付物或回归子集 | HLD、LLD、TEST-PLAN、Decision Brief、后续 release context | true | 新增 CR092 scoped 设计与检查点产物 |

## 回退决策

- 影响范围：局部但高风险，限定在 QMT readonly runtime smoke 设计门禁。
- 回退到阶段：`CR091 closed-current-delivery / READY_WITH_RISK; CR089 blocked-readiness-approved; no active formal CR`
- 需要重新确认的对象：
  - `CR092` CP2 scope / runtime authorization boundary
  - `CR092` CP3 architecture and safety design
  - `CR092` CP5 execution readiness / no-runtime guarantee

## fast-lane 判定

| 条件 | 是否命中 | 说明 |
|---|---|---|
| 仅低风险轻量实现 / 文档 / 规则修改 | false | 涉及真实 runtime 边界和运行授权 |
| 修改架构、权限、安全边界或平台安装路径 | true | 需要定义真实只读 smoke 的权限和禁止动作 |
| 修改外部接口契约、文件所有权或多 Story 依赖 | true | 涉及 gateway / runner readonly contract 和 evidence contract |
| 需要 HLD / LLD 才能解释影响 | true | 必须先冻结范围、流程和回退 |
| 是否保持 fast-lane | false | 使用 standard 模式 |

## LLD 设计批次门禁

- 是否需要 LLD 设计批次：true
- batch_id：`CR092-READONLY-RUNTIME-SMOKE-LLD-BATCH`
- 批次范围来源：CR 影响分析 / 用户选择 CR091-FU-01
- 批次内 Story：
  - `CR092-READONLY-RUNTIME-SMOKE-DESIGN`
- 批次人工确认稿：`process/checkpoints/CP5-CR092-READONLY-RUNTIME-SMOKE-READINESS.md`
- 开发启动条件：
  - [x] HLD 已输出并通过 CP3 人工确认
  - [x] LLD / TEST-PLAN 已输出
  - [x] CP5 自动预检通过
  - [x] CP5 人工确认结论为 `approved`
  - [x] 用户明确授权的 scope 仍限定为设计 / dry-run / manual guide，不包含真实运行

## 执行链路

| 顺序 | 责任角色 | 动作 | 输入 | 输出 | 门控 | 完成后下一步 |
|---|---|---|---|---|---|---|
| 1 | `host-orchestrator` | 创建 CR092 并完成冲突预检 | 用户选择 `CR091-FU-01`、CR091 CP8 follow-up | 本 CR、STATE / CR-INDEX 更新 | CR 已登记 | 准备 CP2 scope gate |
| 2 | `meta-se` | 设计真实只读 runtime smoke HLD | CR092、CR091 / CR089 背景 | HLD、CP3 输入 | CP2 approved 后 | 输出 LLD / TEST-PLAN 输入 |
| 3 | `meta-dev` | 输出 LLD / TEST-PLAN 或技术说明 | HLD、CR092 scope | LLD、TEST-PLAN、CP5 输入 | CP3 approved 后 | 等待 CP5 |
| 4 | `host-orchestrator` | 发起 CP5 readiness | HLD / LLD / TEST-PLAN | CP5 checkpoint | 用户确认 | 才可进入任何后续实施或 manual guide |
| 5 | `host-orchestrator` | CP6 静态实现 | CP5 approved | manual guide、evidence template、static checker、tests、CP6 evidence | CP6 PASS | CP7 静态验证 |
| 6 | `host-orchestrator` | CP7 静态验证 | CP6 artifacts | Verification Report、Test Report、Review、Fixes、CP7 check | CP7 PASS_WITH_RISK | 准备 CP8 |
| 7 | `host-orchestrator` | 收敛终验 | CP7 result | CP8 自动预检与人工审查稿 | CP8 approved | closed-current-delivery / READY_WITH_RISK |

## 自动终验授权

- 是否启用：false
- 授权范围：不适用
- 适用检查点：CP8
- 自动通过条件：
  - [ ] 不适用
- 授权原文：
- 授权时间：
- 回填要求：必须等待人工确认。

## 不授权范围

本 CR 创建和后续 CP2 / CP3 / CP5 设计门禁均不授权：

- 启动、连接、安装或运行 QMT / MiniQMT / XtQuant / gateway / runner。
- 访问、列取、读取、复制、拉取、写入、发布或删除 NAS。
- 读取 `.env`、凭据、真实账号、真实账户、真实资金 / 持仓 / 委托 / 成交或未指定日志原文；用户明确提供的模拟账户测试证据按 CP3 / CP5 安全边界可读。
- 执行 `submit_order`、`cancel_order`、buy / sell。
- 执行 simulation / live。
- 执行 provider fetch、lake write、catalog publish。
- 自动启动 CR089。
- 恢复 CR020 用户删除的 gateway 路线。

## 后续事项台账

- 是否存在后续事项：true
- 台账路径：`process/changes/CR-092-FOLLOW-UP-TRACKING-2026-06-18.md`
- CR 索引路径：`process/changes/CR-INDEX.yaml`
- 一致性检查：`meta-flow check cr-tracking --project-root .`
- 状态取值：`active` / `candidate` / `blocked` / `closed` / `cancelled` / `superseded`

| 候选编号 | 标题 | 状态 | 类型 | 优先级 | 正式 CR 路径 | 相关 active CR / blocked_by / superseded_by | 当前门控 | 阻塞原因 | 下一步 |
|---|---|---|---|---:|---|---|---|---|---|
| CR092-FU-01 | User-provided simulated-account evidence checker run | candidate | CR | 1 |  |  | 未启动 | 需要用户明确提供 evidence 文件路径；不读取凭据 / 真实账户 / NAS | 等待用户选择 |
| CR092-FU-02 | Real readonly runtime smoke per-run authorization | candidate | CR | 2 |  |  | 未启动 | 真实 runtime 高风险，必须独立人工门禁 | 等待用户授权意图 |
| CR091-FU-02 | NAS package exchange gate | candidate | CR | 3 |  |  | 未启动 | 涉及 NAS，需独立授权 | 等待用户选择 |
| CR091-FU-03 | Order-write design gate | candidate | CR | 4 |  |  | 未启动 | 涉及 submit/cancel 或 simulation/live，高风险 | 不建议立即启动 |
| CR091-FU-04 | Ledger hygiene | candidate | CR | 5 |  |  | 未启动 | 历史 CR019 / CR025 账本旧账 | 等待用户选择 |
| CR091-FU-02 | NAS package exchange gate | candidate | CR | 2 |  |  | 未启动 | 涉及 NAS，需独立授权 | 等待用户选择 |
| CR091-FU-03 | Order-write design gate | candidate | CR | 3 |  |  | 未启动 | 涉及 submit/cancel 或 simulation/live，高风险 | 不建议立即启动 |
| CR091-FU-04 | Ledger hygiene | candidate | CR | 4 |  |  | 未启动 | 历史 CR019 / CR025 账本旧账 | 等待用户选择 |

## 处理结论

- 审批结论：`approved / CP2 / CP3 / CP5 / CP8; CP6 PASS; CP7 PASS_WITH_RISK; closed-current-delivery / READY_WITH_RISK`
- [ ] 自动批准（低风险）
- [ ] 待人工确认（中风险）
- [x] 待人工审批（高风险）

## 关联对象

| 类型 | 标识 | 说明 |
|---|---|---|
| CP8 follow-up | `CR091-FU-01` | 真实只读 runner smoke，必须逐 run 授权 |
| Parent CR | `CR-091` | offline runner 研究 / 设计 / 实现 / fixture 验证已关闭 |
| Related CR | `CR-089` | blocked-readiness-approved，不自动启动 |
| Checkpoint | `process/checkpoints/CP8-CR091-DELIVERY-READINESS.md` | 来源决策 `DQ-CP8-CR091-04` |
| Feedback | `process/docs/release/FEEDBACK-CR091.md` | 后续候选来源 |

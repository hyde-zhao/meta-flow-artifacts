---
cr_id: "CR-093"
status: "closed-current-delivery"
impact_level: "medium"
workflow_mode_before: "standard"
workflow_mode_after_change: "standard"
fast_lane_upgrade_reason: "命中状态机 / CR tracking checker / 历史账本文档治理；需要人工门禁，不使用 fast-lane。"
rollback_to: "CR092 closed-current-delivery / READY_WITH_RISK; no active formal CR"
approval_result: "approved CP2/CP3/CP5/CP8; CP6 PASS; CP7 PASS_WITH_RISK; closed-current-delivery / READY_WITH_RISK"
created_at: "2026-06-18T17:30:00+08:00"
created_by: "host-orchestrator"
approved_by: "user for CP2/CP3/CP5/CP8"
approved_at: "2026-06-19T08:39:22+08:00"
source: "cp8-follow-up"
parent_cr: "CR-092"
source_checkpoint: "process/checkpoints/CP8-CR092-DELIVERY-READINESS.md"
source_decision_id: "DQ-CP8-CR092-04"
follow_up_type: "ledger_hygiene"
risk_class: "process-ledger-consistency"
owner: "host-orchestrator"
revisit_condition: "用户于 2026-06-18 明确要求启动 CR091-FU-04。"
acceptance_criteria: "使 cr-tracking 检查不再被 CR019 / CR025 历史旧账阻断；active formal CR / blocked formal CR / follow-up candidate / spike_candidate / stale_status_conflicts 输出可解释、可恢复。"
close_condition: "CP8 approved at 2026-06-19T08:39:22+08:00; current delivery closed as READY_WITH_RISK."
cr_index_path: "process/changes/CR-INDEX.yaml"
---

# CR093 Ledger Hygiene for CR019 / CR025 Tracking Cleanup

## 变更描述

CR093 从 CR091 / CR092 后续候选 `CR091-FU-04 Ledger hygiene` 启动，目标是治理 `meta-flow check cr-tracking --project-root .` 当前被历史 CR019 / CR025 旧账阻断的问题。

本 CR 只处理过程账本、状态机索引和静态一致性检查器，不处理交易业务、行情数据、真实运行、NAS、账户、凭据、submit/cancel、simulation/live、provider/lake/publish。

## 来源与背景

| 字段 | 内容 |
|---|---|
| 直接来源 | CR092 CP8 follow-up tracking 中保留的 `CR091-FU-04` |
| 上游来源 | CR091 CP8 follow-up tracking 中的 `CR091-FU-04` |
| 用户触发 | 用户要求“启动 CR091-FU-04” |
| 当前基线 | CR092 已关闭为 `closed-current-delivery / READY_WITH_RISK`，顶层 `active_change` 为空 |
| 目标缺陷 | `cr-tracking` 静态检查仍把 CR019 / CR025 历史状态文本视为当前阻断 |

## 冲突预检

| 检查项 | 结果 | 证据 | 处理结论 |
|---|---|---|---|
| `process` 路由健康 | PASS | `meta-flow workspace check --project-root /home/hyde/workspace/quant-lab` | `process_link_health: ok`，允许写入 process 账本 |
| 顶层 active change | PASS | `process/STATE.md` 顶层 `active_change: ''` | 无 active formal CR 执行锁 |
| CR092 状态 | PASS | CR092 frontmatter / STATE / CR-INDEX | 已关闭当前交付，不与 CR093 执行范围重叠 |
| `cr-tracking` 基线 | TARGET_DEFECT | `meta-flow check cr-tracking --project-root .` exit 1 | 失败项为本 CR 的目标治理对象，不作为启动阻断 |
| 外部运行 / 数据 / 凭据 | PASS | 本 CR scope | 不授权任何 runtime、NAS、凭据、账户、交易或数据发布动作 |

## 当前已知失败项

| 类别 | 观测 | 初步判断 |
|---|---|---|
| CR019 follow-up tracking 状态不一致 | CR020 / CR030 / CR040..045 等历史行使用 `closed`、`closed-current-delivery`、`closed-spike-complete`、`cancelled-user-deleted` 等状态；检查器仍按硬编码片段要求旧值 | 需要统一状态等价关系或对旧台账做最小规范化 |
| CR025 历史 active_change | `process/STATE.md` 历史段落包含 `active_change: CR-025`；检查器全量扫描文本并误判为当前 active pointer | 需要把检查器改为读取权威顶层 / 当前状态对象，而不是全量文本误扫历史 |
| CR092 closed-current-delivery 警告 | nested historical `active_change: CR-092` 出现在 history 中，状态为 closed-current-delivery | 与 CR025 同类，历史审计段落不应被当作当前 active CR |

## 五维度影响分析

| 维度 | 影响判断 | 处理策略 |
|---|---|---|
| Requirements | 新增“账本一致性检查必须区分当前状态与历史审计文本”的过程需求 | CP2 冻结范围，CP3/CP5 明确 checker 与台账修复合同 |
| Scenario | 新增静态验证场景：无 active formal CR 时不被历史旧账阻断；存在 active formal CR 时能准确报告；follow-up / spike candidate 可解释 | 后续 TEST-PLAN 覆盖 PASS、known warning、target defect 回归 |
| Plan | 需要 standard 模式，经 CP2 / CP3 / CP5 后再实施 | 当前只生成 intake 与 CP2 review，不做修复 |
| Security | 仅静态文件治理，无外部副作用 | 明确禁止 runtime / NAS / 凭据 / 账户 / 交易 / provider / lake / publish |
| Delivery | 可能影响 `scripts/check_cr_tracking_consistency.py`、CR019 follow-up tracking、STATE 当前状态读取规则、CR-INDEX active / follow-up 分类 | 所有改动必须可 diff、可回滚，并保留历史追溯 |

## 文档处理决策

| 对象 | 决策 | 说明 |
|---|---|---|
| 正式 CR | 新增 CR093 | 本文件作为正式 CR |
| CR091 follow-up tracking | 更新 `CR091-FU-04` 为 active | 链接 CR093，不重开 CR091 |
| CR092 follow-up tracking | 更新 `CR091-FU-04` 为 active | 作为 CR092 后续台账的同步索引 |
| `STATE.md` / `CR-INDEX.yaml` | 更新为 CR093 active pending CP2 | 不修改历史审计段落 |
| CR019 follow-up tracking | CP2 后进入设计，CP5 通过后再做最小修复 | 当前不直接修旧账 |
| `scripts/check_cr_tracking_consistency.py` | CP3/CP5 后再改 | 当前仅记录目标缺陷和推荐方向 |

## 初始推荐方案

推荐采用“检查器语义修正 + 最小台账规范化”：

1. 检查器只从权威顶层状态、正式 CR frontmatter、follow-up tracking 当前表格和 CR-INDEX 读取当前状态。
2. 对历史正文 / history / discussion / archived snippets 只作为审计文本，不作为当前 active pointer。
3. 将 closed 等价状态显式化：`closed`、`closed-current-delivery`、`closed-spike-complete` 等应按语义匹配，而不是硬编码单一 Markdown 片段。
4. 对 CR019 follow-up tracking 做最小必要规范化，避免继续制造歧义；不重写历史决策，不删除追溯。

备选方案：

- 只改 CR019 / STATE 文本以满足旧检查器：改动小，但保留检查器误扫历史的设计缺陷。
- 只在检查器中 suppress 历史错误：短期通过，但容易隐藏真实不一致。
- 重写全量 CR ledger checker：更彻底，但当前成本和风险过高。

## LLD / TEST-PLAN 触发

本 CR 需要进入 CP5 LLD / TEST-PLAN / readiness，因为它会触碰状态机检查器与历史账本语义：

- Story 候选：`CR093-LEDGER-HYGIENE-CONSISTENCY`
- 设计对象：checker 当前状态读取、CR 状态等价关系、CR019 follow-up 最小规范化、STATE history 排除规则
- 验证对象：`meta-flow check cr-tracking --project-root .`、单元测试 / fixture、diff check、workspace health check

## 不授权范围

本 CR 的任何 CP2 / CP3 / CP5 / CP8 批准都不自动授权以下事项：

- QMT / MiniQMT / XtQuant / gateway / runner 启动、连接、安装或运行。
- NAS 访问、列取、读取、复制、拉取、写入、发布或删除。
- 读取 `.env`、凭据、真实账户、资金、持仓、委托、成交或原始日志。
- submit / cancel、buy / sell。
- simulation / live。
- provider fetch、lake write、catalog publish。
- 自动启动 CR089、恢复 CR020 gateway 路线，或推进 CR091-FU-02 / CR091-FU-03。

## 当前状态

- 当前门禁：closed by CP8 delivery readiness
- 当前结论：CP2 / CP3 / CP5 / CP8 已 approved；CP6 已 PASS；CP7 已 PASS_WITH_RISK；CR093 当前交付关闭为 `closed-current-delivery / READY_WITH_RISK`
- 用户确认：2026-06-19T08:39:22+08:00 用户回复“好了，同意接受所有项。下一步推进什么，你需要准备好上下文，我需要清除上下文了。”
- 风险接受：接受 `R-CR093-01` broader warning-only backlog 保留；接受 `R-CR093-02` standalone checker warning 明细低于主 CLI 的输出形态风险
- 后续候选：`CR093-FU-01` warning cleanup / strict-warnings readiness 与 `CR093-FU-02` checker output convergence 仅保留候选，不自动启动
- 当前禁止动作：不得执行 runtime、NAS、凭据、账户、交易、simulation/live 或 provider/lake/publish；不得清理 broader warning-only 项，除非用户明确选择后新建 / 恢复独立 CR 门禁

---
cr_id: "CR-120"
title: "CR Tracking Formal/FU Row Convention Hardening"
cr_type: "process"
cr_kind: "requirement-change"
lifecycle_status: "closed"
readiness_status: "ready"
gate_status: "closed"
gate_profile: "standard"
status: "closed-current-delivery"
impact_level: "medium"
workflow_mode_before: "production"
workflow_mode_after_change: "standard"
rollback_to: "FU-CR114-004 follow-up candidate"
source: "cp8-follow-up"
parent_cr: "CR-116"
source_checkpoint: "process/checkpoints/CP8-CR116-DELIVERY-READINESS.md"
source_decision_id: "DQ-CP8-CR116-02"
source_follow_up_id: "FU-CR114-004"
follow_up_type: "cr-tracking-formal-fu-row-convention-hardening"
risk_class: "medium-source-tests-checker-implementation-pending-authorization"
owner: "host-orchestrator"
created_at: "2026-06-22T19:44:17+08:00"
updated_at: "2026-06-23T08:58:36+08:00"
acceptance_criteria: "CP2 approves scope and staged authorization; CP5 must approve any source/tests/checker implementation; CP8 must confirm delivery closure."
close_condition: "CP8 approved by user after implementation and verification, or user rejects / cancels before CP5. Runtime/NAS/credentials/trading/provider/lake/catalog publish remain unauthorized unless separately gated."
cp2_approved_by: "user"
cp2_approved_at: "2026-06-22T19:54:11+08:00"
cp5_approved_by: "user"
cp5_approved_at: "2026-06-22T20:04:30+08:00"
cp8_approved_by: "user"
cp8_approved_at: "2026-06-23T08:58:36+08:00"
---

# CR-120 CR Tracking Formal/FU Row Convention Hardening

## 变更来源

用户明确启动 `FU-CR114-004 CR Tracking Formal/FU Row Convention Hardening`。该候选最早来自 CR114 的 CP8 follow-up，随后在 CR116 的 pending decision queue / checkpoint alignment audit 中重新排序为优先级 2，原因是最近 CR113-CR119 多轮启动、关闭和镜像台账同步暴露出 formal CR 行与 follow-up row 之间的状态约定需要更硬的执行口径。

本 CR 先创建独立正式 CR120。用户已于 2026-06-22T19:54:11+08:00 approve CP2 scope gate，确认范围、影响面、分阶段授权和不授权边界；CP2 不直接修改源码、tests 或 checker implementation。任何实现切片必须等待 CP5 人工批准。

## 当前授权边界

| 范围 | CP2 当前授权 | 说明 |
|---|---:|---|
| process artifacts 读取 / 写入 / 更新 | true | 仅用于 CR120 变更单、context、precheck、checkpoint 和索引状态。 |
| source code owner discovery | true | 可只读定位潜在 checker / tests owner。 |
| source code modify | completed-with-cp5-authorization | 已按 CP5 授权完成 `../meta-flow/meta_flow/checks/cr_tracking.py` 最小修改。 |
| tests modify | completed-with-cp5-authorization | 已按 CP5 授权新增 `../meta-flow/tests/test_cr120_cr_tracking_row_convention.py`。 |
| checker implementation change | completed-with-cp5-authorization | 已完成 focused validation hardening。 |
| bulk historical STATE rewrite | false | 不批量重写历史 `process/STATE.md`。 |
| runtime / NAS / credentials / trading / provider / lake / catalog / publish | false | 本 CR 当前完全不授权。 |

## 五维度影响分析

| 维度 | 影响 | 结论 |
|---|---|---|
| 需求层 | true | 明确 CR tracking 对 formal CR 行、follow-up row、mirror row、active/closed 状态迁移的约定需要可检查化。 |
| 场景层 | true | 覆盖候选启动、正式 CR active、CP8 closed、no-follow-up、downstream active / downstream closed 等状态迁移场景。 |
| 计划层 | true | `FU-CR114-004` 从 candidate 转为 active formal CR120；按 CP2 -> CP5 -> CP8 收敛。 |
| 安全 / 权限层 | true | 涉及源码/tests/checker implementation 授权边界，CP2 必须保持禁止。 |
| 交付层 | true | 若后续 CP5 实施，会影响 CR tracking checker 或 fixture 对 row convention 的验收口径。 |

## 文档处理决策

| 对象 | 决策 | 旧基线保留方式 | 说明 |
|---|---|---|---|
| `process/changes/CR-116-FOLLOW-UP-TRACKING-2026-06-22.md` | 原文档更新 | 保留 `FU-CR114-004` 原候选行并改为 active，链接 CR120 | CR116 是本次启动的直接排序来源。 |
| `process/changes/CR-114-FOLLOW-UP-TRACKING-2026-06-22.md` | 原文档更新 | 保留 `FU-CR114-004` 原始来源行并镜像 active 状态 | 只保留索引字段，不复制 CR120 长正文。 |
| `process/changes/CR-INDEX.yaml` | 原文档更新 | 保留 CR113-CR119 关闭记录，新增 CR120 active | 当前 CR 跟踪机器索引。 |
| `process/STATE.md` | 原文档更新 | 保留历史长表 audit-only，更新当前摘要和人工门禁状态 | 只同步当前态。 |
| `process/docs/design/CR-TRACKING-FORMAL-FU-ROW-CONVENTION-HARDENING-CR120.md` | 新增 | N/A | CR120 的轻量范围说明。 |
| `process/context/CP2-CR120-CR-TRACKING-ROW-CONVENTION-CONTEXT.yaml` | 新增 | N/A | CP2 默认上下文胶囊。 |
| `process/checks/CP2-CR120-CR-TRACKING-ROW-CONVENTION-PRECHECK.md` | 新增 | N/A | CP2 自动预检。 |
| `process/checkpoints/CP2-CR120-CR-TRACKING-ROW-CONVENTION-REVIEW.md` | 新增 | N/A | CP2 人工审查稿。 |

## 待人工决策项

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 影响 / 风险 |
|---|---|---|---|---|---|
| DQ-CP2-CR120-01 | scope | 是否接受将 `FU-CR114-004` 升格为 CR120，并以 CR tracking formal/FU row convention hardening 为独立范围？ | 接受 CR120 作为独立 standard CR，CP2 只确认范围和授权边界。 | A. 暂缓，退回 follow-up candidate；B. reject 并取消候选。 | 推荐方案保留追溯并解决高频一致性缺口；暂缓不新增风险但继续依赖人工同步。 |
| DQ-CP2-CR120-02 | implementation | 是否接受分阶段路线：CP2 仅做 scope / owner discovery / target definition，源码、tests 和 checker implementation 必须等 CP5？ | 接受 staged route，CP2 不改代码，CP5 再决定最小实现切片。 | A. 仅继续写设计说明，不进入 CP5；B. 合并到后续更大的 CR tracking checker 统一改造。 | 推荐方案边界清晰且可小步验证；备选会降低 enforcement 或扩大范围。 |
| DQ-CP2-CR120-03 | runtime_authorization | 是否继续禁止 runtime / NAS / credentials / trading / provider / lake / catalog / publish？ | 继续禁止，仅允许本地静态 / fixture 检查。 | A. 暂停所有验证命令直到 CP5；B. 另起 runtime gate。 | 推荐方案权限最小；本任务不需要外部运行时。 |
| DQ-CP2-CR120-04 | risk_acceptance | 是否接受在当前工作区已有 CR118 / CR119 未提交 checker diff 的前提下启动 CR120 process gate？ | 接受；CR120 CP2 只写 process artifacts，不回滚既有源码/tests diff。 | A. 先要求提交或清理 CR118/CR119 diff；B. 暂缓 CR120。 | 推荐方案不破坏已验证改动；若后续 CP5 实施，必须叠加验证 CR118/CR119 目标测试。 |

## 不授权范围

- 不授权修改源码、tests 或 checker implementation。
- 不授权批量历史 `process/STATE.md` rewrite。
- 不授权启动 runtime、连接 NAS、读取凭据、访问账户 / 资金 / 持仓 / 委托 / 成交 / raw log。
- 不授权 submit / cancel / buy / sell、simulation / live、provider fetch、lake write、catalog publish。
- 不授权 CR105、CR089、QMT / MiniQMT / XtQuant / gateway 路线被 CR120 隐式启动。

## 当前结论

CR120 已通过 CP2 / CP5 / CP6 / CP7 / CP8，关闭为 closed-current-delivery / READY。用户在 CP8 approve 同时提出的目录路由 / merge 回归检测问题不属于 CR120 row convention 实现范围，已单独记录为 `FU-CR120-001` 候选；runtime / NAS / 凭据 / 交易写 / publish 继续不授权。

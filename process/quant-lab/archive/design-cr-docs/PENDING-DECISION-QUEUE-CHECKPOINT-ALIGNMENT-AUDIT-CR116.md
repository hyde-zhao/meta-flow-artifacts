---
title: "CR116 Pending Decision Queue / Checkpoint Alignment Audit"
cr_id: "CR-116"
status: "closed-ready"
created_at: "2026-06-22T17:31:28+08:00"
owner: "host-orchestrator"
scope: "no-code-no-runtime"
---

# CR116 Pending Decision Queue / Checkpoint Alignment Audit

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| v0.1 | 2026-06-22 | host-orchestrator | 启动 CR116 CP2 范围、对齐问题定义、初始发现和不授权边界。 |
| v0.2 | 2026-06-22 | host-orchestrator | CP2 approved 后补充审计结论、对齐规则状态、关闭项、后续候选和 CP8 收口输入。 |
| v1.0 | 2026-06-22 | host-orchestrator | CP8 approved 后记录 closed READY 结论和后续候选不自动启动边界。 |

## 目标

CR116 只做 pending decision queue 与 checkpoint DQ 表的一致性审计和低风险清理建议。目标不是修改 checker、重写历史 STATE 或启动运行时，而是先冻结当前门禁的可判定规则：

- checkpoint Decision Brief / DQ 表是人工门禁的主要真相源。
- `STATE.md.human_gate_decisions.pending_human_decisions[]` 只应表达当前 gate 的待确认项。
- 历史已接受 DQ 可以保留在审计历史或 accepted 列表，但不应继续被识别为当前 pending。
- 对话发起消息只展示本轮 DQ，不把历史 accepted DQ 混入用户确认范围。

## 复盘范围

| 对象 | 本轮处理方式 | 排序关注点 | 不做事项 |
|---|---|---|---|
| STATE current gate fields | 复盘 `status`、`active_gate`、`pending_gate`、`pending_checklist_path`、`pending_decision_ids` | 是否指向当前 CP2 / CP8 checkpoint | 不重写历史 `history[]` |
| pending decision queue | 复盘 `pending_human_decisions[]` 中历史 accepted DQ 混入现象 | 是否会被误读为当前待确认项 | 不批量迁移所有历史 DQ |
| checkpoint DQ table | 复盘 CP2 / CP8 Decision Brief 中 DQ 表完整性 | DQ 是否有类型、推荐、备选、优劣、影响、回退 | 不改 checkpoint-manager 源码 |
| launch prompt contract | 复盘发起消息与 checkpoint DQ 的数量 / ID 一致性 | approve 是否只接受本轮推荐项 | 不实现 checker |
| CR tracking | 复盘 FU-CR114-003 转 CR116 active 的索引同步 | active formal CR 是否唯一 | 不恢复 CR089 |

## 初始发现

| 排名 | 发现项 | 严重度 | 状态 | 证据 | 建议处理 |
|---:|---|---|---|---|---|
| 1 | `human_gate_decisions.pending_human_decisions[]` 中仍包含大量历史 `status: accepted` 的 DQ，名称上与 pending 语义冲突 | HIGH | in-scope | `process/STATE.md` | 本 CR 先冻结语义：current pending 只看当前 gate DQ；历史 accepted 不作为 pending。 |
| 2 | `orchestrator_session.active_change` 存在历史 CR102 残留，容易和顶层 `active_change` 冲突 | MEDIUM | in-scope | `process/STATE.md` | 本轮只刷新当前门禁摘要；不批量重写旧 session 细节。 |
| 3 | checkpoint DQ 表与 STATE pending queue 不应互相复制完整历史，否则会放大发起消息长度和误确认范围 | MEDIUM | in-scope | CP2/CP8 checkpoint pattern | 采用 checkpoint 为真相源、STATE 只保留当前 gate 引用的规则。 |
| 4 | 若需要机器化校验 pending queue 必须为空或只含当前 DQ，将进入 checker implementation 范围 | MEDIUM | follow-up-candidate | `FU-CR115-001` / `FU-CR114-004` | 需要源码 / tests 授权时另起实现 CR。 |

## 推荐对齐规则

| 规则 ID | 规则 | 理由 | 本轮动作 |
|---|---|---|---|
| ALIGN-CR116-01 | 人工门禁 DQ 真相源以 `process/checkpoints/CP*.md` 的 Decision Brief 为准 | checkpoint 是用户审查入口，结构完整 | 在 CP2 决策中确认 |
| ALIGN-CR116-02 | `pending_decision_ids` 只列本轮发起消息实际展示的 DQ ID | 避免历史 DQ 污染当前 approve 语义 | 本轮 STATE 摘要指向 CR116 CP2 |
| ALIGN-CR116-03 | `pending_human_decisions[]` 中历史 accepted DQ 不应被当作当前 pending | 名称语义与状态冲突，容易误读 | 本轮记录为待清理发现，不批量删除 |
| ALIGN-CR116-04 | launch prompt 必须打印 checkpoint 路径、自动预检、DQ 数量、不授权项和三个 exact 回复 | 保持 CR115 draft contract | 本轮沿用，不实现 checker |

## 不授权边界

| 范围 | CR116 结论 |
|---|---|
| 源码修改 | 不授权 |
| tests 修改 | 不授权 |
| checker implementation change | 不授权 |
| 批量历史 STATE rewrite | 不授权 |
| CR105 / order-write / simulation / live | 不启动 |
| CR089 恢复 | 不启动 |
| QMT / MiniQMT / XtQuant / gateway runtime | 不启动 |
| NAS / 凭据 / 账户 / raw log | 不访问 |
| provider fetch / lake write / catalog publish | 不执行 |

## CP2 待确认

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 影响 / 风险 |
|---|---|---|---|---|---|
| DQ-CP2-CR116-01 | scope | 是否接受本 CR 只做 no-code/no-runtime pending queue 与 checkpoint 对齐审计 | 接受当前范围 | A. 暂停；B. 改 checker implementation；C. 改批量历史 STATE rewrite 设计 CR | 推荐方案风险低；B/C 需要新授权。 |
| DQ-CP2-CR116-02 | implementation | 当前门禁 DQ 真相源如何冻结 | checkpoint DQ 表为人工门禁真相源；STATE pending queue 只承载当前待确认项，历史 accepted 不算 pending | A. 保持现状只记录风险；B. 全量迁出历史 accepted DQ；C. 改 checker 容忍历史混入 | 推荐方案能低风险收敛语义；B/C 扩大范围。 |
| DQ-CP2-CR116-03 | runtime_authorization | approve 是否继续不代表 runtime / 源码 / NAS / trading 授权 | 继续禁止隐式授权 | A. 另起源码 CR；B. 另起 checker CR；C. 另起 runtime gate | 推荐方案避免误授权；备选需要独立门禁。 |

## 下一步

CP8 approved 后，CR116 可关闭为 READY。任何源码、checker、runtime 或批量历史 STATE 迁移动作都必须转入独立 CR。

## CP2 审批摘要

| 字段 | 内容 |
|---|---|
| 审批结论 | approved |
| 审批人 | user |
| 审批时间 | 2026-06-22T17:39:56+08:00 |
| 接受决策 | `DQ-CP2-CR116-01`、`DQ-CP2-CR116-02`、`DQ-CP2-CR116-03` |
| 不授权项 | 源码、tests、checker implementation、批量历史 STATE rewrite、runtime、NAS、凭据、交易写、provider/lake/catalog publish |

## Ranked Findings

| 排名 | 发现项 | 严重度 | 状态 | 证据 | 建议处理 |
|---:|---|---|---|---|---|
| 1 | `pending_human_decisions[]` 名称与内容不一致：历史 `accepted` DQ 仍在该数组中，容易被误读成当前 pending | HIGH | accepted-current-rule | `process/STATE.md.human_gate_decisions` | 当前门禁只使用 `pending_decision_ids` 和 checkpoint DQ 表判定；历史 accepted 只作为审计历史。 |
| 2 | checkpoint DQ 表已经具备完整人工门禁真相源字段，但 STATE 不应复制完整历史 DQ 表 | MEDIUM | accepted-current-rule | `process/checkpoints/CP2-CR116-PENDING-DECISION-QUEUE-CHECKPOINT-ALIGNMENT-REVIEW.md` | 后续 gate 继续以 checkpoint Decision Brief 为准，STATE 仅保存当前 gate refs。 |
| 3 | `orchestrator_session.active_change` 已由本轮从旧 CR102 残留刷新到 CR116，但历史 session 记录仍保留 | MEDIUM | closed-current-delivery | `process/STATE.md.orchestrator_session` | 本轮只刷新当前字段，不批量重写历史 session。 |
| 4 | 机器化 enforcement 仍需 checker implementation 才能保证未来不再混入历史 accepted DQ | MEDIUM | follow-up-candidate | `FU-CR115-001` / `FU-CR114-004` | 保留为实现候选，需源码 / tests 授权。 |

## 本轮关闭项

| 项目 ID | 内容 | 关闭证据 | 说明 |
|---|---|---|---|
| CLOSE-CR116-01 | CR116 CP2 范围、不授权边界和对齐原则 | `process/checkpoints/CP2-CR116-PENDING-DECISION-QUEUE-CHECKPOINT-ALIGNMENT-REVIEW.md` | 用户已 approve。 |
| CLOSE-CR116-02 | 当前 gate 的 pending decision ID 指向 CR116 CP8 | `process/STATE.md` | 后续只展示本轮 CP8 DQ。 |
| CLOSE-CR116-03 | CR116 tracking 从 FU candidate 转 active | `process/changes/CR-114-FOLLOW-UP-TRACKING-2026-06-22.md`，`process/changes/CR-INDEX.yaml` | `cr-tracking` 应可识别 CR116 active。 |

## 后续候选

| 候选编号 | 标题 | 类型 | 优先级 | 启动条件 | 不授权边界 |
|---|---|---|---:|---|---|
| FU-CR115-001 | Human Gate Launch Message Checker Implementation Candidate | requirement-change | 1 | 用户明确选择 checker implementation | 需要源码 / tests / checker 授权；不自动启动。 |
| FU-CR114-004 | CR Tracking Formal/FU Row Convention Hardening | requirement-change | 2 | 用户明确选择 row convention hardening | 需要源码 / tests / checker 授权；不自动启动。 |
| FU-CR114-002 | Context Path Alias Normalization Notes | ledger-maintenance | 3 | 用户选择路径展示约定复盘 | 不改目录、不改 symlink。 |

## CP8 推荐结论

| 字段 | 内容 |
|---|---|
| 推荐发布结论 | READY |
| 理由 | CR116 的 no-code/no-runtime 审计、对齐规则、当前状态刷新和后续分流已完成；剩余机器化 enforcement 已分流为后续候选，不阻断当前关闭。 |
| 风险 | 无 runtime / 源码风险；剩余风险为历史 accepted DQ 仍保留在 legacy STATE 长表中，后续应由实现 CR 处理机器化校验。 |
| 回退 | 若用户不接受 checkpoint DQ 表为真相源，可回退到 CP2 修改 DQ-CP2-CR116-02。 |

## CP8 审批后关闭结论

| 字段 | 内容 |
|---|---|
| CP8 结论 | approved |
| 审批时间 | 2026-06-22T17:46:19+08:00 |
| 关闭状态 | closed-current-delivery / READY |
| 已关闭范围 | CR116 no-code/no-runtime 审计、对齐规则、当前 gate refs 和后续分流。 |
| 保留候选 | `FU-CR115-001`、`FU-CR114-004`、`FU-CR114-002` 仅作为 candidate，不自动启动。 |
| 不授权边界 | 不授权源码、tests、checker implementation、批量历史 STATE rewrite、runtime、NAS、凭据、交易写、CR105、CR089 恢复或 publish。 |

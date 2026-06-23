---
title: "CR114 Context Capsule / Human Gate Consistency Review"
cr_id: "CR-114"
status: "draft-for-cp2"
created_at: "2026-06-22T16:48:11+08:00"
owner: "host-orchestrator"
scope: "no-code-no-runtime"
---

# CR114 Context Capsule / Human Gate Consistency Review

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| v0.1 | 2026-06-22 | host-orchestrator | 启动 CR114 CP2 复盘范围、排序准则和候选对象清单。 |
| v0.2 | 2026-06-22 | host-orchestrator | CP2 approved 后补充 ranked findings、关闭项、后续候选和 CP8 收口输入。 |

## 目标

CR114 只做 context capsule 与 human gate 一致性的候选复盘和排序。目标不是修复 checker、改源码或启动运行时，而是先把后续治理 backlog 分成可关闭、可后续设计、需要实现 CR、需要高风险授权和不启动几类。

## 复盘范围

| 对象 | 本轮处理方式 | 排序关注点 | 不做事项 |
|---|---|---|---|
| Context Capsule | 复盘 CP2 / CP3 / CP5 / CP8 的 capsule 字段、read_profile、read_expansion_log 口径 | 是否能作为默认读取入口；缺失时是否有 N/A / WAIVED / BLOCKED 理由 | 不改 schema 实现 |
| Human Gate Checkpoint | 复盘 Decision Brief、待人工决策清单、人工审查结果区 | checkpoint 文件与对话发起消息是否一致 | 不改 checkpoint-manager 源码 |
| Pending Decision Queue | 复盘 `human_gate_decisions.pending_human_decisions[]` 与 checkpoint 表格的一致性 | DQ 是否有类型、推荐、备选、影响 / 风险、回退条件 | 不批量重写历史 STATE |
| Launch Message | 复盘发起人工门禁时是否打印路径、自动预检、DQ 数量、不授权项和三个 exact 回复 | approve 语义是否被清楚限定 | 不引入新的 UI 或 runtime |
| CR Tracking | 复盘 candidate 转 active、active 关闭、follow-up tracking 与 CR-INDEX 对齐 | 是否避免假 active / 漏 candidate / stale conflict | 不实现 checker 改造 |

## 排序准则

| 排序维度 | 权重 | 判定说明 |
|---|---:|---|
| 门禁误导风险 | 5 | 是否可能导致用户 `approve` 被误解为授权 runtime / 源码 / publish。 |
| 状态漂移风险 | 4 | `STATE.md`、checkpoint、CR-INDEX、follow-up tracking 是否可能表达不一致。 |
| 上下文预算风险 | 4 | 是否会导致下游默认读取全文档而不是 capsule。 |
| 后续实现成本 | 3 | 是否需要源码 / tests / checker implementation 才能闭环。 |
| 当前可收敛性 | 3 | 是否能在 no-code/no-runtime 范围内形成可执行结论。 |

## 初始候选排序

| 排名 | 候选方向 | 初始判断 | 建议处理 |
|---:|---|---|---|
| 1 | CP2 / CP3 / CP5 / CP8 human gate launch contract | 高优先级 | 本 CR 内复盘并排序；若只是文档 / 状态口径，CP8 可关闭；若需 checker，转后续实现 CR。 |
| 2 | Context capsule read policy consistency | 高优先级 | 本 CR 内复盘 capsule 必填字段、缺失处理和扩展读取理由。 |
| 3 | Decision collection coverage consistency | 中高优先级 | 复盘 DQ 来源扫描、纳入 / N/A 理由和 checkpoint 表格一致性。 |
| 4 | CR candidate 转 active / close tracking consistency | 中优先级 | 复盘 FU 候选转正式 CR 的索引字段和状态同步。 |
| 5 | CP result JSON / ledger machine truth gap | 中优先级 | 本 CR 只分类；若需要机器校验或实现，转 `FU-CR113-003` 或后续实现 CR。 |
| 6 | Checker implementation alignment | 延后 | 明确转 `FU-CR113-004`；需要源码和 tests 授权，不在 CR114 内做。 |

## 不授权边界

| 范围 | CR114 结论 |
|---|---|
| 源码修改 | 不授权 |
| tests 修改 | 不授权 |
| checker implementation change | 不授权 |
| CR105 / order-write / simulation / live | 不启动 |
| CR089 恢复 | 不启动 |
| QMT / MiniQMT / XtQuant / gateway runtime | 不启动 |
| NAS / 凭据 / 账户 / raw log | 不访问 |
| provider fetch / lake write / catalog publish | 不执行 |

## CP2 待确认

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 影响 / 风险 |
|---|---|---|---|---|---|
| DQ-CP2-CR114-01 | scope | 是否接受本 CR 只做 no-code/no-runtime 一致性复盘和排序 | 接受当前范围 | A. 暂停；B. 改 architecture-only；C. 改 implementation CR | 推荐方案风险低；备选 C 需要新授权。 |
| DQ-CP2-CR114-02 | follow_up_tracking | 是否接受当前排序准则 | 接受五维排序准则 | A. 先 ledger JSON；B. 先 HLD；C. 只列清单不排序 | 推荐方案利于后续门禁治理；备选会改变收敛顺序。 |
| DQ-CP2-CR114-03 | runtime_authorization | approve 是否继续不代表 runtime / 源码 / NAS / trading 授权 | 继续禁止隐式授权 | A. 另起源码 CR；B. 另起 checker CR；C. 另起 runtime gate | 推荐方案避免误授权；备选需要独立门禁。 |

## 下一步

CP2 approved 后，本 CR 将只读复盘已存在的 capsule、checkpoint、CR-INDEX、STATE 顶层摘要和 follow-up tracking，输出 ranked findings 和 CP8 收口稿。

## CP2 审批摘要

| 字段 | 内容 |
|---|---|
| 审批结论 | approved |
| 审批人 | user |
| 审批时间 | 2026-06-22T16:56:35+08:00 |
| 接受决策 | `DQ-CP2-CR114-01`、`DQ-CP2-CR114-02`、`DQ-CP2-CR114-03` |
| 不授权项 | 源码、tests、checker implementation、runtime、NAS、凭据、交易写、provider/lake/catalog publish |

## Ranked Findings

| 排名 | 发现项 | 严重度 | 状态 | 证据 | 建议处理 |
|---:|---|---|---|---|---|
| 1 | Human gate 发起消息需要与 checkpoint 的 DQ 表、自动预检、capsule 摘要和不授权项保持可校验一致 | HIGH | follow-up-candidate | `process/checkpoints/CP2-CR114-CONTEXT-CAPSULE-HUMAN-GATE-CONSISTENCY-REVIEW.md`，`meta-flow check human-gate` | 启动 `FU-CR114-001`，只做 launch message draft / checker contract 复盘；若需要工具实现，另起 implementation CR。 |
| 2 | Context capsule 与 checkpoint / 对话中存在 `process/docs/design` 和 `docs/design` 两种可读路径表达 | MEDIUM | follow-up-candidate | `process/context/CP2-CR114-CONTEXT-CAPSULE-HUMAN-GATE-CONSISTENCY-CONTEXT.yaml`，本复盘稿 | 启动 `FU-CR114-002`，形成路径别名展示约定；当前不改源码。 |
| 3 | Candidate 转正式 CR 时，CR tracking checker 需要 follow-up 表中出现正式 `CR-数字` 行，否则会产生匹配 warning | MEDIUM | closed-current-delivery | `process/changes/CR-113-FOLLOW-UP-TRACKING-2026-06-22.md`，`meta-flow check cr-tracking --project-root .` | 本轮已在 CR113 台账补 `CR-114` active 行；后续可转 `FU-CR114-004` 做 checker / convention hardening。 |
| 4 | Pending decision queue 与 checkpoint DQ 表目前以 checkpoint 为主，STATE 顶层摘要只保留人工提示，不宜读取完整历史长表 | MEDIUM | follow-up-candidate | `process/STATE.md` 顶层摘要，CP2 checkpoint DQ 表 | 启动 `FU-CR114-003` 做 no-code 审计；需要机器化同步时另起实现 CR。 |
| 5 | CP result JSON / event ledger 机器真相源缺口仍属于独立候选，不应混入 CR114 | MEDIUM | deferred-existing-candidate | `FU-CR113-003` | 保留 `FU-CR113-003`，等待用户选择或前置一致性收敛。 |
| 6 | Checker implementation alignment 需要源码和 tests 授权 | HIGH | deferred-existing-candidate | `FU-CR113-004` | 不在 CR114 内启动；用户明确授权后另起 implementation CR。 |

## 本轮关闭项

| 项目 ID | 内容 | 关闭证据 | 说明 |
|---|---|---|---|
| CLOSE-CR114-01 | CR114 CP2 范围、排序准则和不授权边界 | `process/checkpoints/CP2-CR114-CONTEXT-CAPSULE-HUMAN-GATE-CONSISTENCY-REVIEW.md` | 用户已 approve。 |
| CLOSE-CR114-02 | `FU-CR113-001` 转正式 CR 的 active 状态同步 | `process/changes/CR-113-FOLLOW-UP-TRACKING-2026-06-22.md`，`process/changes/CR-INDEX.yaml` | `cr-tracking` 当前 OK。 |
| CLOSE-CR114-03 | 当前 ranked consistency findings | 本文件 | 已输出后续分流建议。 |

## 后续候选

| 候选编号 | 标题 | 类型 | 优先级 | 启动条件 | 不授权边界 |
|---|---|---|---:|---|---|
| FU-CR114-001 | Human Gate Launch Message Checkable Draft Contract | ledger-maintenance | 1 | 用户明确选择；保持 no-code/no-runtime | 不改源码、不改 checker implementation |
| FU-CR114-002 | Context Path Alias Normalization Notes | ledger-maintenance | 2 | 用户选择路径展示约定复盘 | 不改目录、不改 symlink、不做迁移 |
| FU-CR114-003 | Pending Decision Queue / Checkpoint Alignment Audit | ledger-maintenance | 3 | 用户选择队列与 checkpoint 对齐审计 | 不批量重写历史 STATE |
| FU-CR114-004 | CR Tracking Formal/FU Row Convention Hardening | requirement-change | 4 | 用户选择 checker / convention hardening | 需要独立实现 CR 才能改源码/tests |

## CP8 推荐结论

| 字段 | 内容 |
|---|---|
| 推荐发布结论 | READY |
| 理由 | 本 CR 的 no-code/no-runtime 复盘、排序和台账同步已完成；剩余事项已分流为后续候选，不阻断当前关闭。 |
| 风险 | 无 runtime / 源码风险；剩余风险为过程治理一致性后续优化。 |
| 回退 | 若用户不接受排序或后续候选，可回退到 CP2 修改排序准则。 |

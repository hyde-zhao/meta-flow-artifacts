---
cr_id: "CR-058"
status: "closed-current-delivery"
impact_level: "high"
workflow_mode_before: "standard"
workflow_mode_after_change: "standard"
fast_lane_upgrade_reason: "命中 repo-local relayout、文件所有权、历史审计证据保护、rollback_ref 和未来机械替换门禁，必须保持 standard。"
rollback_to: "CP2-CR058 requirements baseline"
approval_result: "cp8-approved-ready-with-risk"
created_at: "2026-06-14T13:59:00+08:00"
created_by: "host-orchestrator"
approved_by: "user"
approved_at: "2026-06-14T16:59:36+08:00"
source: "cp8-follow-up"
linked_issue: ""
parent_cr: "CR-053"
source_checkpoint: "process/checkpoints/CP8-CR053-DELIVERY-READINESS.md"
source_decision_id: "DQ-CP8-CR053-03"
follow_up_candidate_id: "FU-CR053-01"
follow_up_type: "follow_up_candidate"
risk_class: "high"
owner: "host-orchestrator / human"
revisit_condition: "用户确认要推进 README / USER-MANUAL / future-facing docs rewrite 或 repo-local relayout gate。"
acceptance_criteria: "CR058 自身 CP2/CP3/CP5/CP8、rollback_ref、candidate list、preserve-audit allowlist 均明确通过；真实 move / rename / delete 或路径替换仍需后续显式授权。"
close_condition: "CR058 只关闭 repo-local mechanical migration / relayout gate 设计与候选清单门禁；不关闭任何真实迁移动作。"
cr_index_path: "process/changes/CR-INDEX.yaml"
---

# CR-058 Repo-local Mechanical Migration / Relayout Gate

## 变更描述

从 `process/changes/CR-053-FOLLOW-UP-TRACKING-2026-06-14.md` 的 `FU-CR053-01` 创建正式 CR058。目标是为 repo-local mechanical migration / relayout 建立可审计门禁：候选列表、preserve-audit allowlist、rollback_ref、no-op guardrail、人工复核和后续执行授权边界。

本 CR 当前只允许创建正式 CR、影响分析、CP2 / CP3 / CP5 门禁和 repo-local mechanical migration 设计。不得执行真实 `move` / `rename` / `delete`、NAS 操作、`MARKET_DATA_LAKE_ROOT` 替换、git push / tag / remote rename / history rewrite、凭据读取、QMT / MiniQMT runtime、provider fetch、lake write 或 catalog publish。

## CP8 Follow-up 来源

| 字段 | 内容 |
|---|---|
| 父级 CR | `CR-053` |
| 来源检查点 | `process/checkpoints/CP8-CR053-DELIVERY-READINESS.md` |
| 来源决策 ID | `DQ-CP8-CR053-03` |
| follow-up 候选 | `FU-CR053-01` |
| follow-up 类型 | `follow_up_candidate` |
| 风险等级 | `high` |
| owner | `host-orchestrator / human` |
| 重访条件 | 用户确认推进 README / USER-MANUAL / future-facing docs rewrite 或 repo-local relayout gate。 |
| 验收标准 | CR058 自身 CP2 / CP3 / CP5 / CP8、`rollback_ref`、candidate list、preserve-audit allowlist 均通过；任何真实动作需另行授权。 |
| 关闭条件 | 只关闭门禁设计，不关闭真实迁移执行。 |

## 冲突预检

| 检查项 | 结论 | 说明 |
|---|---|---|
| 当前 active formal CR | `CR-046` paused | `CR046` 仍保持用户暂停的 CP6 PASS / ready-for-verification 恢复点；本 CR 不恢复、不调度、不验证 CR046。 |
| 与 CR046 影响面重叠 | false | CR046 是 QMT / MiniQMT 双目标策略框架；CR058 是 repo-local migration / relayout gate。共同点只有安全不授权边界，处理方式均为 fail-closed。 |
| 与 CR053 关系 | parent / follow-up | CR053 已 `closed-current-delivery`，`READY_WITH_RISK` 只关闭静态 inventory / dry-run；不继承任何真实迁移授权。 |
| 与 CR060+ / data lake / trading runtime | false for execution; true for boundary | CR058 必须把 NAS、data lake、trading runtime 继续列为不授权项，不得合并启动。 |
| 并行推进结论 | allowed-with-paused-CR046 | 用户本轮明确启动 CR058 且要求不要恢复 CR046；CR058 可作为独立 formal CR 进入设计门禁。 |

## 文档处理决策

| 受影响文档 | 处理方式 | 旧基线保留方式 | 修订记录位置 | 批准状态 |
|---|---|---|---|---|
| `process/changes/CR-053-FOLLOW-UP-TRACKING-2026-06-14.md` | 原文档更新 | 保留原 `FU-CR053-01` 行，追加正式 CR 路径和 closed-current-delivery 状态，不删除原候选语义。 | 台账表行内状态 | approved |
| `process/changes/CR-INDEX.yaml` | 原文档更新 | 保留 CR046 paused active 和 CR053 closed 记录；CR058 移入 closed-current-delivery。 | YAML 索引项 | approved |
| `process/STATE.md` | 原文档更新 | 保留 CR046 paused recovery point；CR058 CP8 决策回填为 approved。 | `cr_tracking` / `human_gate_decisions` / activity log | approved |
| `docs/design/HLD-CR058-REPO-LOCAL-MECHANICAL-MIGRATION-RELAYOUT-GATE.md` | 新增 | CR053 HLD / release docs 作为输入基线，不覆盖。 | 文档头部修订记录 | approved |
| `docs/design/ARCHITECTURE-DECISION-CR058.md` | 新增 | CR053 ADR 作为 parent decision context，不覆盖。 | 文档头部修订记录 | approved |
| `docs/design/FEATURE-DESIGN-MATRIX-CR058.md` | 新增 | 不替换全局矩阵；CR-scoped 矩阵用于本 CR CP5。 | 文档头部修订记录 | approved |
| `process/stories/CR058-*.md` | 新增 | 不改写 CR053 / CR046 Story。 | Story 内修订记录 / 技术说明 | approved |
| `README.md` / `docs/USER-MANUAL.md` | 不变 | 当前 CR 只设计候选清单；不执行文档 rewrite。 | 不适用 | approved |
| `docs/release/*-CR053.md` | 不变 | CR053 静态 close 证据保留，不重写。 | 不适用 | approved |

## 旧基线映射

| 原基线对象 | 新增 / 修改对象 | 保留策略 | 映射说明 |
|---|---|---|---|
| `FU-CR053-01` | `CR-058` | 台账保留，正式 CR 承载细节 | FU 行只保留状态、正式路径、当前门控、阻塞和下一步。 |
| `docs/release/MIGRATION-INVENTORY-CR053.md` | CR058 candidate list gate | 原文保留 | 只消费 path_pattern / move_action / risk / verification_rule，不改写 inventory。 |
| `docs/release/PATH-REFERENCES-CR053.md` | preserve-audit allowlist gate | 原文保留 | 只消费 `mechanical-candidate` / `manual-review` / `preserve-audit` 分类。 |
| `docs/release/MIGRATION-PLAN-CR053.md` | rollback_ref gate | 原文保留 | CR058 必须先满足 `pre_cr058_commit`、`pre_cr058_git_bundle`、`pre_reference_rewrite_manifest`。 |
| `process/checkpoints/CP8-CR053-DELIVERY-READINESS.md` | CP2 / CP3 / CP5 CR058 decisions | 原文保留 | CR053 CP8 的不授权边界是 CR058 的输入，不是授权继承。 |

## 五维度影响分析

| 维度 | 评估问题 | 受影响对象 | 结论 | 处理动作 |
|------|----------|-----------|------|---------|
| 需求层 | 是否新增、删除或重定义 REQ-* | CR-scoped requirements gate | true | 新增 CR058 CP2 baseline，定义 repo-local relayout gate 范围、不授权项和验收条件。 |
| 场景层 | 是否改变测试矩阵覆盖范围 | CR058 no-op / dry-run / manual review scenarios | true | 新增静态验证场景：candidate list 完整性、preserve-audit 覆盖、rollback_ref 缺失 fail-closed、禁止真实动作。 |
| 计划层 | 是否改变 Phase、Wave、Story / 任务依赖 | CR058 Story batch | true | 新增 CR058-S01..S05，全部 CP5 统一确认前不得进入 CP6 或真实执行。 |
| 安全层 | 是否引入新的高风险动作或权限要求 | repo-local move / rewrite authorization boundary | true | 标记 high；本 CR 不授权真实动作，只设计 gate；后续执行需 runtime_authorization / implementation decision。 |
| 交付层 | 是否需要重新生成交付物或回归子集 | HLD / ADR / Feature matrix / Story / checkpoints | true | 生成 CR-scoped 设计与门禁；不更新用户文档正文、不执行 migration。 |

## 回退决策

- 影响范围：CR-scoped，影响 repo-local relayout gate 和 CR tracking；不影响 CR046 paused recovery point。
- 回退到阶段：`CP2-CR058 requirements baseline`。
- 需要重新确认的对象：CR058 范围、不授权项、candidate list、preserve-audit allowlist、rollback_ref、后续执行授权。

## fast-lane 判定

| 条件 | 是否命中 | 说明 |
|---|---|---|
| 仅低风险轻量实现 / 文档 / 规则修改 | false | 涉及 repo-local relayout 和未来机械替换门禁。 |
| 修改架构、权限、安全边界或平台安装路径 | true | 需要定义真实动作不授权和后续授权边界。 |
| 修改外部接口契约、文件所有权或多 Story 依赖 | true | 需要 owner / artifact_class / preserve-audit 分类和 Story batch。 |
| 需要 HLD / LLD 才能解释影响 | true | 需要 HLD、ADR、Feature matrix 与 CP5 统一确认。 |
| 是否保持 fast-lane | false | 必须 standard。 |

## LLD 设计批次门禁

- 是否需要 LLD 设计批次：true
- batch_id：`CR058-RELAYOUT-GATE-BATCH-A`
- 批次范围来源：CR053 `FU-CR053-01`、CR053 migration inventory / path references / migration plan、用户本轮启动指令。
- 批次内 Story：
  - `CR058-S01-intake-conflict-and-scope-freeze`
  - `CR058-S02-candidate-list-and-owner-classification-gate`
  - `CR058-S03-preserve-audit-allowlist-and-sensitive-filter-gate`
  - `CR058-S04-rollback-ref-and-preflight-evidence-gate`
  - `CR058-S05-mechanical-migration-dry-run-and-authorization-boundary`
- 批次人工确认稿：`process/checkpoints/CP5-CR058-RELAYOUT-GATE-BATCH-A-LLD-BATCH.md`
- 开发启动条件：
  - [ ] 批次内全部 Story 设计证据已输出（S01 / S05 technical-note；S02 / S03 / S04 full-lld）。
  - [ ] 批次内全部 Story CP5 自动预检已通过。
  - [ ] 批次 CP5 人工确认结论为 `approved`。
  - [ ] `rollback_ref`、candidate list、preserve-audit allowlist 的必填字段和验收标准已确认。
  - [ ] 用户另行明确授权具体真实动作；否则 CP6 仍只能做文档 / dry-run 设计收敛。

## 执行链路

| 顺序 | 责任角色 | 动作 | 输入 | 输出 | 门控 | 完成后下一步 |
|---|---|---|---|---|---|---|
| 1 | `host-orchestrator` | 创建 CR 并完成冲突预检 | 用户请求 / CR053 follow-up | 本 CR、CR tracking 更新、CP2 gate | CR 已登记 | CP2 人工确认 |
| 2 | `meta-se` / `host-orchestrator` | 完成 repo-local relayout HLD 和 Story batch | CR053 inventory / path references / migration plan | HLD、ADR、Feature matrix、Story | CP3 / CP4 / CP5 | 等待 CP5 人工确认 |
| 3 | `meta-dev` | 后续可按 CP5 设计输出 candidate manifest / dry-run report | 已批准 CP5 | 仅文档或 dry-run 产物 | CP6；不得真实 move / rewrite | 交回验证 |
| 4 | `meta-qa` | 后续验证 no-op guardrail 和 manifest 完整性 | CP6 evidence | CP7 verification | CP7 | CP8 release readiness |
| 5 | `host-orchestrator` | CP8 收敛 | CP7 / release context | CP8 checkpoint | 用户确认 | 关闭 CR058 gate，不自动执行迁移 |

## 自动终验授权

- 是否启用：false
- 授权范围：不适用
- 适用检查点：CP8
- 自动通过条件：N/A
- 授权原文：N/A
- 授权时间：N/A
- 回填要求：CR058 所有真实动作和终验关闭均需人工确认。

## 后续事项台账

- 是否存在后续事项：true
- 台账路径：`process/changes/CR-053-FOLLOW-UP-TRACKING-2026-06-14.md`
- CR 索引路径：`process/changes/CR-INDEX.yaml`
- 一致性检查：`scripts/check_cr_tracking_consistency.py` 当前为历史 CR tracking 兼容检查，不完全覆盖 CR058；CR058 使用 human-gate check 验证门禁草稿。
- 状态取值：`closed-current-delivery` for `FU-CR053-01`

| 候选编号 | 标题 | 状态 | 类型 | 优先级 | 正式 CR 路径 | 相关 active CR / blocked_by / superseded_by | 当前门控 | 阻塞原因 | 下一步 |
|---|---|---|---|---:|---|---|---|---|---|
| FU-CR053-01 | CR058 repo-local mechanical migration / relayout gate | closed-current-delivery | CR | 1 | `process/changes/CR-058-REPO-LOCAL-MECHANICAL-MIGRATION-RELAYOUT-GATE-2026-06-14.md` | `CR-046` paused non-overlap; parent `CR-053` closed | CP8 READY_WITH_RISK approved | 用户已确认 CR058 CP8；static-only/no-op relayout gate 当前交付关闭；真实 move/rewrite 未授权。 | CR059 仅作为 repo-local mechanical migration execution gate 后续候选，不自动启动。 |

## 处理结论

- 审批结论：`approved-ready-with-risk`
- [ ] 自动批准（低风险）
- [x] 待人工确认（中风险）
- [x] 待人工审批（高风险）

## 关联对象

| 类型 | 标识 | 说明 |
|---|---|---|
| Parent CR | `CR-053` | `closed-current-delivery`，只交付静态 migration inventory / dry-run。 |
| Follow-up | `FU-CR053-01` | 本正式 CR 来源。 |
| Paused CR | `CR-046` | 用户暂停的 CP6 PASS / ready-for-verification 恢复点；本 CR 不恢复。 |
| Release context | `process/release/RELEASE-CONTEXT-CR053.yaml` | CR053 READY_WITH_RISK 与不授权边界。 |
| Inventory | `docs/release/MIGRATION-INVENTORY-CR053.md` | CR058 candidate source。 |
| Path references | `docs/release/PATH-REFERENCES-CR053.md` | CR058 preserve-audit / mechanical-candidate source。 |
| Migration plan | `docs/release/MIGRATION-PLAN-CR053.md` | CR058 rollback_ref source。 |

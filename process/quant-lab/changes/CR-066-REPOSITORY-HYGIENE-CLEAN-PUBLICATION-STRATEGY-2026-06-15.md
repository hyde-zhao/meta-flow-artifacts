---
cr_id: "CR-066"
status: "closed-current-delivery"
impact_level: "high"
workflow_mode_before: "standard"
workflow_mode_after_change: "standard"
fast_lane_upgrade_reason: "命中 Git publication surface、tracked forbidden paths、clean branch/snapshot/history governance 和敏感路径例外判定，必须保持 standard。"
rollback_to: "CR062 blocked publication baseline"
approval_result: "approved"
created_at: "2026-06-15T10:24:37+08:00"
created_by: "host-orchestrator"
approved_by: "user"
approved_at: "2026-06-15T10:46:59+08:00"
cp8_approved_by: "user"
cp8_approved_at: "2026-06-15T11:32:07+08:00"
closed_at: "2026-06-15T11:32:07+08:00"
source: "USER-20260615-APPROVE-CR062-BLOCKER-FOLLOW-UP"
linked_issue: ""
parent_cr: "CR-062"
source_checkpoint: "process/checks/CR062-POST-APPROVAL-PREFLIGHT.md"
source_decision_id: "DQ-CR062-BLOCKER-01"
follow_up_type: "repository-hygiene-clean-publication-strategy"
risk_class: "high"
owner: "host-orchestrator"
revisit_condition: "CR066 CP2 / CP3 / CP5 / CP8 均已 approved；CR067 Clean Publication Execution Gate 已保留为可由用户显式启动的 follow-up candidate；本 CR 不授权 git add/commit/remote add/push/tag/branch creation/rename/history rewrite/force push/remote deletion。"
acceptance_criteria: "冻结 tracked forbidden paths 处理策略、.env.example 例外策略、clean publication architecture、manifest policy、扫描和后续执行 CR 边界。"
close_condition: "Clean publication strategy 已通过 CP8 并以 READY_WITH_RISK 关闭；CR062 保持 blocked，CR067 保持 candidate。"
cr_index_path: "process/changes/CR-INDEX.yaml"
---

# CR-066 Repository Hygiene / Clean Publication Strategy

## 变更描述

CR062 已经通过 CP2 / CP3 / CP5，但 post-approval preflight 在当前 Git index 中发现 `reports/**`、`runs/**` 和 `.env.example` 已被跟踪。由于 `git push current branch` 会发布当前分支既有 tracked history，CR062 按 fail-closed 停止，未执行 `git add`、`git commit`、`git remote add` 或 `git push`。

用户回复“同意”后，接受 `DQ-CR062-BLOCKER-01` 推荐方案：暂停 CR062 publication execution，并启动 repository hygiene / clean publication strategy。CR063 / CR064 / CR065 已在迁移序列语义中分别保留给 NAS / data lake / runtime 候选，因此本正式 CR 使用下一个不冲突编号 `CR-066`。

本 CR 只做策略和门禁，不执行：

- `git add`
- `git commit`
- `git remote add` / `git remote set-url`
- `git push`
- `git tag`
- branch creation / branch rename / default branch rename
- history rewrite / force push / remote deletion
- NAS/data lake/provider/catalog/runtime/凭据操作
- physical move / rename / delete
- bulk import rewrite
- CR046 recovery

## 冲突预检

| 检查项 | 结论 | 说明 |
|---|---|---|
| CR062 状态 | blocked-publication-scan-forbidden-tracked-paths | 需要 clean publication strategy 决策；不能继续 current branch push。 |
| CR046 状态 | active-cp6-pass-ready-for-verification / user paused | 不恢复 CR046，不触发 CP7 或 runtime。 |
| CR063 / CR064 / CR065 编号 | reserved by migration sequence | 分别保留 NAS/archive、data lake/catalog、runtime/trading 候选；CR066 避免编号冲突。 |
| Git 写动作授权 | not authorized | 本 CR 只创建策略和门禁文档，不写 Git config、不 stage、不 commit、不 push。 |
| 历史治理影响 | high | clean branch / snapshot / rewrite / orphan publish 均会影响发布语义，必须单独门禁。 |

## CP8 Follow-up 来源

| 字段 | 内容 |
|---|---|
| 父级 CR | `CR-062` |
| 来源检查点 | `process/checks/CR062-POST-APPROVAL-PREFLIGHT.md` |
| 来源决策 ID | `DQ-CR062-BLOCKER-01` |
| follow-up 类型 | `repository-hygiene-clean-publication-strategy` |
| 风险等级 | `high` |
| owner | `host-orchestrator` |
| 重访条件 | 用户已批准 CR066 CP2 / CP3 / CP5 / CP8；后续 clean publication execution 必须通过 CR067 或新的正式 CR 单独授权。 |
| 验收标准 | tracked forbidden paths、`.env.example`、clean publication architecture、manifest policy 和后续执行边界可审计。 |
| 关闭条件 | CR066 已通过 CP8 并以 READY_WITH_RISK 关闭；CR062 保持 blocked。 |

## 文档处理决策

| 受影响文档 | 处理方式 | 旧基线保留方式 | 修订记录位置 | 批准状态 |
|---|---|---|---|---|
| `process/changes/CR-062-GIT-REMOTE-CUTOVER-REPOSITORY-PUBLICATION-GATE-2026-06-15.md` | 原文档更新 | 保留 blocked preflight 事实和 CR066 follow-up 映射 | N/A | approved |
| `process/checks/CR062-POST-APPROVAL-PREFLIGHT.md` | 原文档更新 | 保留原 blocker findings，修正 follow-up 编号为 CR066 | N/A | approved |
| `process/changes/CR-INDEX.yaml` | 原文档更新 | CR062 转 blocked，CR066 转 closed-current-delivery，CR067 保留 candidate | N/A | approved |
| `process/STATE.md` | 原文档更新 | 保留 CR046 paused、CR062 blocked，记录 CR066 closed 和 CR067 candidate 状态 | N/A | approved |
| `process/context/CP2-CR066-REPOSITORY-HYGIENE-CONTEXT.yaml` | 新增 | N/A | N/A | approved |
| `process/context/CP3-CR066-CLEAN-PUBLICATION-DESIGN-CONTEXT.yaml` | 新增 | N/A | N/A | approved |
| `process/context/CP5-CR066-CLEAN-PUBLICATION-READINESS-CONTEXT.yaml` | 新增 | N/A | N/A | approved |
| `docs/release/REPOSITORY-HYGIENE-SCOPE-CR066.md` | 新增 | N/A | N/A | approved |
| `docs/release/TRACKED-FORBIDDEN-PATHS-AUDIT-CR066.md` | 新增 | N/A | N/A | approved |
| `docs/release/CLEAN-PUBLICATION-STRATEGY-CR066.md` | 新增 | N/A | N/A | approved |
| `docs/release/CLEAN-PUBLICATION-MANIFEST-CR066.yaml` | 新增 | N/A | N/A | approved |
| `process/context/CP8-CR066-DELIVERY-CONTEXT.yaml` | 新增 | N/A | N/A | approved |
| `process/release/RELEASE-CONTEXT-CR066.yaml` | 新增 | N/A | N/A | approved |
| `docs/release/RELEASE-NOTES-CR066.md` | 新增 | N/A | N/A | approved |
| `docs/release/DEPLOY-CHECKLIST-CR066.md` | 新增 | N/A | N/A | approved |
| `docs/release/ROLLBACK-CR066.md` | 新增 | N/A | N/A | approved |
| `docs/release/MIGRATION-CR066.md` | 新增 | N/A | N/A | approved |
| `docs/release/FEEDBACK-CR066.md` | 新增 | N/A | N/A | approved |

## 旧基线映射

| 原基线对象 | 新增 / 修改对象 | 保留策略 | 映射说明 |
|---|---|---|---|
| CR062 current branch publication plan | CR066 clean publication strategy | CR062 blocked 证据保留 | CR066 不覆盖 CR062 preflight 事实，只决定后续如何解阻。 |
| CR062 exclude policy | CR066 refined publication surface policy | 原 exclude 保留为 fail-closed 基线 | CR066 明确 `.env.example` 是否例外、tracked reports/runs 是否可发布。 |
| current Git branch `work/chapter3-factor-gap-remediation-20260608` | clean publication candidate route | 当前分支保持不变 | CR066 不 rename、不 rewrite、不 push 当前分支。 |

## 五维度影响分析

| 维度 | 评估问题 | 受影响对象 | 结论 | 处理动作 |
|---|---|---|---|---|
| 需求层 | 是否新增或重定义发布需求 | GitHub publication readiness / clean public baseline | true | 将 CR062 blocker 转为 CR066 clean publication strategy。 |
| 场景层 | 是否改变验证矩阵覆盖范围 | tracked forbidden path audit、placeholder scan、clean manifest scan | true | 新增 CR066 audit / manifest / strategy 文档和 CP2/CP3/CP5 门禁。 |
| 计划层 | 是否改变 Phase / Gate | CR062 blocked -> CR066 CP2/CP3/CP5 | true | 暂停 CR062 execution，CR066 先做策略确认。 |
| 安全层 | 是否引入新的高风险动作或权限要求 | branch/snapshot/history governance、`.env.example` exception | true | 当前只设计，不授权 Git 写动作；后续执行另起 CR。 |
| 交付层 | 是否需要重新生成交付物或回归子集 | clean publication manifest、scope、audit、launch message | true | 新增 release 策略文件和 checkpoint。 |

## 回退决策

- 影响范围：repository publication surface / clean GitHub baseline。
- 回退到阶段：`CR062 blocked publication baseline`。
- 需要重新确认的对象：tracked `reports/**` / `runs/**` 处理、`.env.example` 例外、clean branch/snapshot 方案、history rewrite 是否完全禁止、后续执行 CR 编号和授权边界。
- 当前 rollback strategy：本 CR 未执行 Git 写动作；回退只需保留 CR062 blocked 状态并取消 CR066。

## fast-lane 判定

| 条件 | 是否命中 | 说明 |
|---|---|---|
| 仅低风险轻量实现 / 文档 / 规则修改 | false | 影响 GitHub 发布面和 tracked history。 |
| 修改架构、权限、安全边界或平台安装路径 | true | 涉及 clean branch/snapshot/history governance 和 `.env.example` 例外。 |
| 修改外部接口契约、文件所有权或多 Story 依赖 | true | 影响 GitHub remote publication gate 和未来 commit manifest。 |
| 需要 HLD / LLD 才能解释影响 | true | 需要 CP3 / CP5 决策 clean publication architecture。 |
| 是否保持 fast-lane | false | 必须 standard。 |

## LLD 设计批次门禁

- 是否需要 LLD 设计批次：true
- batch_id：`CR066-CLEAN-PUBLICATION-STRATEGY-BATCH-A`
- 批次范围来源：CR062 post-approval preflight blocker / DQ-CR062-BLOCKER-01
- 批次内 Story：
  - `CR066-S01-tracked-forbidden-paths-audit`
  - `CR066-S02-env-example-exception-policy`
  - `CR066-S03-clean-publication-architecture`
  - `CR066-S04-clean-manifest-and-scan-gate`
  - `CR066-S05-follow-up-execution-boundary`
- 批次人工确认稿：`process/checkpoints/CP5-CR066-CLEAN-PUBLICATION-READINESS.md`
- 开发启动条件：
  - [x] CP2 / CP3 / CP5 人工确认均为 `approved`。
  - [ ] clean publication manifest policy 明确排除或接受 tracked forbidden path 风险。
  - [ ] 后续执行 CR 单独批准前不执行 branch creation、history rewrite、commit、remote add 或 push。

## 执行链路

| 顺序 | 责任角色 | 动作 | 输入 | 输出 | 门控 | 完成后下一步 |
|---|---|---|---|---|---|---|
| 1 | `host-orchestrator` | 回填 CR062 blocker 决策并创建 CR066 | `DQ-CR062-BLOCKER-01` / CR062 preflight | 本 CR、contexts、scope、audit、strategy、manifest、CP2/CP3/CP5 | 当前文件创建 | 等待人工确认 |
| 2 | `host-orchestrator` | CP2 / CP3 / CP5 人工门禁 | 自动预检 + Decision Brief | approved | 用户 exact reply | 已进入 CR066 CP8 |
| 3 | `host-orchestrator` | CP8 readiness | approved CR066 strategy | RELEASE-CONTEXT / RELEASE-NOTES / DEPLOY / ROLLBACK / MIGRATION / FEEDBACK / CP8 | approved | CR066 关闭为 READY_WITH_RISK |
| 4 | `host-orchestrator` | 后续执行分流 | CR066 strategy | CR067 candidate 或 CR062 继续 blocked | CP8 approved | 不自动执行 Git 写动作 |

## 自动终验授权

- 是否启用：false
- 授权范围：不适用
- 适用检查点：CP8
- 自动通过条件：N/A
- 授权原文：N/A
- 授权时间：N/A
- 回填要求：必须等待人工确认。

## 后续事项台账

- 是否存在后续事项：true
- 台账路径：`process/changes/CR-INDEX.yaml`
- CR 索引路径：`process/changes/CR-INDEX.yaml`
- 一致性检查：`uv run --python 3.11 python scripts/check_cr_tracking_consistency.py --project-root .`
- 状态取值：`active` / `candidate` / `closed` / `blocked`

| 候选编号 | 标题 | 状态 | 类型 | 优先级 | 正式 CR 路径 | 相关 active CR / blocked_by / superseded_by | 当前门控 | 阻塞原因 | 下一步 |
|---|---|---|---|---:|---|---|---|---|---|
| CR-066 | Repository Hygiene / Clean Publication Strategy | closed-current-delivery | CR | 1 | `process/changes/CR-066-REPOSITORY-HYGIENE-CLEAN-PUBLICATION-STRATEGY-2026-06-15.md` | CR062 blocked; CR046 paused | closed | READY_WITH_RISK accepted | strategy-only 关闭；不执行 Git 写动作。 |
| CR-067 | Clean Publication Execution Gate | candidate-ready-after-CR066 | CR | 1 |  | requires explicit user start | not-started | 不在本轮 | 用户明确继续 CR067 后另起正式 CR + CP2/CP3/CP5。 |

## 处理结论

- 审批结论：`approved`
- [ ] 自动批准（低风险）
- [ ] 待人工确认（中风险）
- [x] 人工审批已通过（高风险，CP2/CP3/CP5）
- [x] CP8 人工终验已通过（READY_WITH_RISK）

## 关闭结果

- 关闭状态：`closed-current-delivery`
- 关闭时间：2026-06-15T11:32:07+08:00
- 关闭结论：CR066 作为 strategy-only delivery 关闭为 READY_WITH_RISK。
- 后续候选：CR067 Clean Publication Execution Gate 可由用户显式启动；启动前不得执行 Git 写动作或 GitHub publication execution。
- 不授权项：本关闭不授权 `git add`、`git commit`、`git remote add`、`git push`、`git tag`、branch creation / rename、history rewrite、force push、remote deletion、NAS/data lake/provider/catalog/runtime/凭据操作、physical move、bulk import rewrite 或 CR046 recovery。

## 关联对象

| 类型 | 标识 | 说明 |
|---|---|---|
| Parent CR | `CR-062` | Git remote cutover gate 已因 tracked forbidden paths 阻断。 |
| Source evidence | `process/checks/CR062-POST-APPROVAL-PREFLIGHT.md` | BLOCKED preflight。 |
| Source decision | `DQ-CR062-BLOCKER-01` | 用户回复“同意”接受推荐 follow-up。 |
| Scope | `docs/release/REPOSITORY-HYGIENE-SCOPE-CR066.md` | CR066 scope / non-scope。 |
| Audit | `docs/release/TRACKED-FORBIDDEN-PATHS-AUDIT-CR066.md` | tracked forbidden path 分类。 |
| Strategy | `docs/release/CLEAN-PUBLICATION-STRATEGY-CR066.md` | clean publication architecture。 |
| Manifest | `docs/release/CLEAN-PUBLICATION-MANIFEST-CR066.yaml` | gate-only manifest；`execute_allowed_now=false`。 |

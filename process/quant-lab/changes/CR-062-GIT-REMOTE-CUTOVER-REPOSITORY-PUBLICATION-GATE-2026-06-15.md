---
cr_id: "CR-062"
status: "closed-blocked-superseded"
impact_level: "high"
workflow_mode_before: "standard"
workflow_mode_after_change: "standard"
fast_lane_upgrade_reason: "命中 Git remote、外部 GitHub publication、提交范围、安全扫描、凭据边界和发布回滚，必须保持 standard。"
rollback_to: "pre-CR062 repository publication baseline"
approval_result: "approved"
created_at: "2026-06-15T09:34:08+08:00"
created_by: "host-orchestrator"
approved_by: "user"
approved_at: "2026-06-15T09:59:22+08:00"
source: "USER-20260615-START-CR062-GIT-REMOTE-CUTOVER-REPOSITORY-PUBLICATION-GATE"
linked_issue: ""
parent_cr: "CR-061"
source_checkpoint: "process/checkpoints/CP8-CR061-BATCH-B-DELIVERY-READINESS.md"
source_decision_id: "DQ-CP8-CR061-03"
follow_up_type: "migration-sequence"
risk_class: "high"
owner: "host-orchestrator"
revisit_condition: "用户已批准 CR062 CP2 / CP3 / CP5，但 post-approval preflight 发现已跟踪 forbidden publication paths：reports/**、runs/** 和 .env.example。publication execution 已 fail-closed 阻断；未执行 git add/commit/remote add/push。用户已批准 DQ-CR062-BLOCKER-01，后续策略由 CR066 承接，clean publication baseline 由 CR067 完成，.gitignore forward-fix 由 CR068 完成；CR069 于 2026-06-15T23:01:31+08:00 将 CR062 收口为 closed-blocked-superseded。"
acceptance_criteria: "CR062 必须冻结 GitHub 目标、include/exclude policy、scan plan、rollback/fail-closed 规则和不授权边界；真实 Git 操作只能在门禁 approve 后按 CP5 DQ 执行。"
close_condition: "CR069 CP2/CP3/CP5 已批准并执行 ledger-only cleanup；CR062 原 current-branch publication route 关闭为 blocked-superseded，替代路径为 CR066/CR067/CR068。"
cr_index_path: "process/changes/CR-INDEX.yaml"
closed_by: "CR-069"
closed_at: "2026-06-15T23:01:31+08:00"
superseded_by: "CR-066, CR-067, CR-068"
---

# CR-062 Git Remote Cutover / Repository Publication Gate

## 变更描述

CR062 从 CR061 `READY_WITH_RISK` 之后承接 GitHub 发布门禁，目标是在不恢复 CR046、不触碰 NAS / data lake / provider / runtime / 凭据、不执行 physical move 或 bulk import rewrite 的前提下，建立 `quant-lab` 仓库远端发布的正式准入门。

本次创建动作只启动正式 CR062、冻结 CP2 / CP3 / CP5 gate 包、发布范围、扫描计划、回滚策略和不授权边界。用户批准门禁前，不执行：

- `git remote add` / `git remote set-url`
- `git push`
- `git tag`
- branch rename / default branch rename
- history rewrite / force push
- commit / staging / add
- NAS/data lake/provider/catalog/runtime/凭据操作
- physical move / rename / delete
- bulk import rewrite
- CR046 recovery

推荐 GitHub 目标远端为 `git@github.com:hyde-zhao/quant-lab.git`。该值来自 `process/context/POST-CR061-MIGRATION-HANDOFF-CONTEXT.yaml`，在 CP5 通过前只作为候选，不写入 Git config。

## CR069 Supersession Outcome

| 字段 | 内容 |
|---|---|
| 收口时间 | `2026-06-15T23:01:31+08:00` |
| 收口 CR | `CR-069` |
| 最终分类 | `closed-blocked-superseded` |
| 原阻断事实 | post-approval preflight 发现 current branch 已跟踪 `reports/**`、`runs/**`、`.env.example`，因此 CR062 fail-closed。 |
| 替代路径 | `CR-066` clean publication strategy、`CR-067` clean publication execution、`CR-068` `.gitignore` forward-fix。 |
| 远端事实 | public repo `git@github.com:hyde-zhao/quant-lab.git` `master` 记录在 CR068：`3ade165e8b1edad031a911490cf6c1cee942616e`。 |
| 不授权项 | CR069 收口未执行 Git remote add / set-url / push / tag / branch rename / default branch governance / history rewrite / force push / remote deletion；未执行 NAS/data lake/provider/catalog/runtime/credential、physical move、bulk import rewrite 或 CR046 recovery。 |
| 后续处理 | branch/default branch governance、NAS archive/data migration、data lake/catalog 和 runtime/trading authorization 必须另起 CR。 |

## 冲突预检

| 检查项 | 结论 | 说明 |
|---|---|---|
| 当前 active formal CR | `CR-046` paused | CR046 仍为用户暂停恢复点；CR062 不恢复 CR046、不调度 CP7、不进入 runtime。 |
| CR061 状态 | closed-current-delivery | CR061 Batch B 已接受 `READY_WITH_RISK`，提供 package/import identity baseline。 |
| 与 CR046 影响面重叠 | controlled / no runtime | CR062 只处理 Git publication；不得运行 `trading/**`、不得读凭据、不得查询账户或交易。 |
| 与 CR063 / CR064 / CR065 关系 | excluded | NAS archive/data migration、data lake root/catalog migration、runtime/trading authorization 均不进入 CR062。 |
| Git 操作授权 | closed-blocked-superseded | CP2 / CP3 / CP5 已 approved，但 post-approval scan 命中 tracked forbidden paths；不执行 commit、remote add 或 push；CR069 已将该路线收口。 |
| blocker 后续 | CR066/CR067/CR068 completed | 用户已批准 DQ-CR062-BLOCKER-01；CR066/CR067/CR068 已完成替代路径并被 CR069 作为 supersession evidence。 |
| 顶层 `active_change` | preserved as `CR-046` | 为既有 consistency 脚本和 CR046 paused recovery 保持 legacy active pointer；CR062 通过 `cr_tracking.active_crs` 表达并行迁移门禁。 |

## CP8 Follow-up 来源

| 字段 | 内容 |
|---|---|
| 父级 CR | `CR-061` |
| 来源检查点 | `process/checkpoints/CP8-CR061-BATCH-B-DELIVERY-READINESS.md` |
| 来源决策 ID | `DQ-CP8-CR061-03` |
| follow-up 类型 | `migration-sequence` |
| 风险等级 | `high` |
| owner | `host-orchestrator` |
| 重访条件 | CR062 原 publication route 已由 CR069 收口为 closed-blocked-superseded；如需 branch/default branch governance 或新 Git 远端治理，另起 CR。 |
| 验收标准 | GitHub 目标、发布范围、扫描计划、回滚策略、不授权项和 post-approval execution sequence 可审计。 |
| 关闭条件 | 已通过 CR069 ledger-only cleanup 收口；CR062 不再继续执行原路线。 |

## 文档处理决策

| 受影响文档 | 处理方式 | 旧基线保留方式 | 修订记录位置 | 批准状态 |
|---|---|---|---|---|
| `process/changes/CR-INDEX.yaml` | 原文档更新 | 保留 CR061 closed 和 CR046 paused 记录，新增 CR062 active gate | N/A | approved |
| `process/STATE.md` | 原文档更新 | 顶层 `active_change=CR-046` 保留；新增 CR062 gate 状态 | N/A | approved |
| `process/context/CP2-CR062-REQUIREMENT-CONTEXT.yaml` | 新增 | N/A | N/A | approved |
| `process/context/CP3-CR062-DESIGN-CONTEXT.yaml` | 新增 | N/A | N/A | approved |
| `process/context/CP5-CR062-REPOSITORY-PUBLICATION-CONTEXT.yaml` | 新增 | N/A | N/A | approved |
| `docs/release/REPOSITORY-PUBLICATION-MANIFEST-CR062.yaml` | 新增 | N/A | N/A | approved |
| `docs/release/GIT-PUBLICATION-SCOPE-CR062.md` | 新增 | N/A | N/A | approved |
| `docs/release/PUBLICATION-SCAN-PLAN-CR062.md` | 新增 | N/A | N/A | approved |
| `docs/release/ROLLBACK-REF-CR062.md` | 新增 | N/A | N/A | approved |
| `pyproject.toml` / `uv.lock` / `quant_lab/**` / `tests/**` | post-approval manifest candidate | CR061 已形成 repo-local baseline，CR062 只发布扫描通过的 allowlist 路径 | N/A | approved-for-manifest-scan |
| `.env*` / `data/**` / `reports/**` / `runs/**` | 不变且排除 | 不读取、不扫描正文、不纳入 Git publication | N/A | excluded |

## 旧基线映射

| 原基线对象 | 新增 / 修改对象 | 保留策略 | 映射说明 |
|---|---|---|---|
| repo-local `quant-lab` baseline after CR061 | GitHub remote candidate `git@github.com:hyde-zhao/quant-lab.git` | 本地基线保持不变直到 CP5 approve | 远端发布只复制经 allowlist / scan 通过的仓库状态。 |
| CR061 `READY_WITH_RISK` | CR062 publication gate | 保留 CR061 风险接受记录 | full pytest 未运行风险在 CR062 CP5 再次显式暴露。 |
| local Git config | optional post-approval remote config | CP5 前不写入 Git config | 若 post-approval 检测到现有 remote 冲突，必须停止并回到用户决策。 |
| 当前 branch | post-approval exact branch push | 不 rename、不重写 | 若当前 branch 和 GitHub 默认分支策略冲突，停止并另行决策，不在 CR062 静默 rename。 |

## 五维度影响分析

| 维度 | 评估问题 | 受影响对象 | 结论 | 处理动作 |
|---|---|---|---|---|
| 需求层 | 是否新增或重定义迁移需求 | Repository publication / GitHub baseline | true | 将 CR061 follow-up 转为 CR062 正式 publication gate。 |
| 场景层 | 是否改变验证矩阵 | secret scan、data body scan、large file scan、path boundary scan、Git push evidence | true | 只规划 post-approval 扫描和发布证据，不做 runtime / provider / lake 验证。 |
| 计划层 | 是否改变 Phase / Gate | CP2 / CP3 / CP5 / post-approval execution / CP8 | true | 当前只生成 CP2/CP3/CP5，执行和 CP8 等待 approve 后另行推进。 |
| 安全层 | 是否引入外部接口或权限风险 | GitHub SSH remote、credential boundary、history safety | true | CP5 DQ 明确授权范围；tag / branch rename / history rewrite / force push 禁止。 |
| 交付层 | 是否需要发布文档和回滚证据 | manifest、scope、scan plan、rollback ref | true | 新增 docs/release CR062 门禁文件。 |

## 回退决策

- 影响范围：Git repository publication gate。
- 回退到阶段：`pre-CR062 repository publication baseline`。
- 需要重新确认的对象：GitHub target remote、commit include/exclude manifest、current branch handling、remote conflict behavior、scan result handling、rollback/fail-closed rule。
- 当前 rollback strategy：CP5 前没有 Git 写动作；CP5 后如果 publication 执行失败，不做 history rewrite / force push，保留本地状态并通过 forward-fix 或新 CR 处理。

## fast-lane 判定

| 条件 | 是否命中 | 说明 |
|---|---|---|
| 仅低风险轻量实现 / 文档 / 规则修改 | false | GitHub publication 是外部状态变更。 |
| 修改架构、权限、安全边界或平台安装路径 | true | 远端、SSH auth、提交范围和敏感排除属于安全边界。 |
| 修改外部接口契约、文件所有权或多 Story 依赖 | true | GitHub remote 与发布 manifest 会影响交付边界。 |
| 需要 HLD / LLD 才能解释影响 | true | 需要 CP3 / CP5 冻结发布架构与执行计划。 |
| 是否保持 fast-lane | false | 必须 standard。 |

## LLD 设计批次门禁

- 是否需要 LLD 设计批次：true
- batch_id：`CR062-REPOSITORY-PUBLICATION-BATCH-A`
- 批次范围来源：CR062 影响分析 / CR061 CP8 follow-up
- 批次内 Story：
  - `CR062-S01-publication-scope-and-allowlist`
  - `CR062-S02-secret-data-large-file-scan-gate`
  - `CR062-S03-git-remote-conflict-and-branch-policy`
  - `CR062-S04-post-approval-publication-sequence`
  - `CR062-S05-publication-evidence-and-rollback`
- 批次人工确认稿：`process/checkpoints/CP5-CR062-REPOSITORY-PUBLICATION-READINESS.md`
- 开发启动条件：
  - [ ] CP2 / CP3 / CP5 人工确认均为 `approved`。
  - [ ] post-approval preflight scan 全部 PASS。
  - [ ] commit manifest 冻结并排除 forbidden paths。
  - [ ] 远端冲突检测通过；如 remote 已存在且不同，停止并回到人工决策。
  - [ ] 不执行 tag / branch rename / history rewrite / force push。

## 执行链路

| 顺序 | 责任角色 | 动作 | 输入 | 输出 | 门控 | 完成后下一步 |
|---|---|---|---|---|---|---|
| 1 | `host-orchestrator` | 创建 CR062 gate 包 | CR061 CP8 / handoff / STATE / CR-INDEX | 本 CR、contexts、scope、manifest、scan plan、rollback、CP2/CP3/CP5 | 当前文件创建 | 等待人工确认 |
| 2 | `host-orchestrator` | CP2 / CP3 / CP5 人工门禁 | 自动预检 + Decision Brief | approved | 用户 exact reply | 已进入 post-approval preflight |
| 3 | `host-orchestrator` | post-approval preflight | CP5 DQ 授权 | `process/checks/CR062-POST-APPROVAL-PREFLIGHT.md` | BLOCKED | 已停止；需新决策 / CR |
| 4 | `host-orchestrator` | blocker follow-up | DQ-CR062-BLOCKER-01 | CR066 strategy gate | pending | 等待 CR066 CP2/CP3/CP5 人工确认 |
| 5 | `host-orchestrator` | publication execution | future approved clean execution CR | Git commit、remote config、push evidence | 禁止 tag / branch rename / rewrite，且必须另行授权 | 进入 CP8 |
| 6 | `host-orchestrator` | 发布终验 | push evidence / scan evidence | CP8 readiness | 用户 approve | 关闭 publication execution CR |

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
| CR-062 | Git Remote Cutover / Repository Publication Gate | closed-blocked-superseded | CR | 1 | `process/changes/CR-062-GIT-REMOTE-CUTOVER-REPOSITORY-PUBLICATION-GATE-2026-06-15.md` | closed_by=CR069; superseded_by=CR066/CR067/CR068 | closed | tracked forbidden paths found; replacement path completed | 不再执行原 current-branch publication route；后续 Git governance 另起 CR。 |
| CR-066 | Repository Hygiene / Clean Publication Strategy | closed-current-delivery | CR | 1 | `process/changes/CR-066-REPOSITORY-HYGIENE-CLEAN-PUBLICATION-STRATEGY-2026-06-15.md` | replacement path part 1 | closed | N/A | 保留 strategy evidence。 |
| CR-067 | Clean Publication Execution Gate | closed-current-delivery | CR | 1 | `process/changes/CR-067-CLEAN-PUBLICATION-EXECUTION-GATE-2026-06-15.md` | replacement path part 2 | closed | N/A | 保留 remote publication evidence。 |
| CR-068 | Repository Hygiene / .gitignore Forward-fix Gate | closed-current-delivery | CR | 1 | `process/changes/CR-068-REPOSITORY-HYGIENE-GITIGNORE-FORWARD-FIX-2026-06-15.md` | replacement path part 3 | closed | N/A | 保留 `.gitignore` forward-fix evidence。 |
| CR-063 | NAS archive / data migration | candidate | CR | 2 |  | blocked by CR062 publication or explicit reorder | not-started | 不在本轮 | CR062 后另起。 |
| CR-064 | Data lake root / catalog migration | candidate | CR | 3 |  | blocked by separate data authorization | not-started | 不在本轮 | 独立 CR。 |
| CR-065 | Runtime / trading authorization | candidate | CR | 4 |  | blocked by CR046 recovery decision | not-started | 不在本轮 | 仅用户明确恢复时处理。 |

## 处理结论

- 审批结论：`closed-blocked-superseded`
- [ ] 自动批准（低风险）
- [ ] 待人工确认（中风险）
- [x] 人工审批已通过（高风险）

## 关联对象

| 类型 | 标识 | 说明 |
|---|---|---|
| Parent CR | `CR-061` | package/import/layout convergence 已关闭为 READY_WITH_RISK。 |
| Handoff | `process/context/POST-CR061-MIGRATION-HANDOFF-CONTEXT.yaml` | 指定 CR062 为下一步推荐。 |
| Remote candidate | `git@github.com:hyde-zhao/quant-lab.git` | CP5 前不写入 Git config。 |
| Publication scope | `docs/release/GIT-PUBLICATION-SCOPE-CR062.md` | include/exclude 和不授权边界。 |
| Manifest | `docs/release/REPOSITORY-PUBLICATION-MANIFEST-CR062.yaml` | gate approved but execution blocked；`execute_allowed_now=false`。 |
| Post-approval preflight | `process/checks/CR062-POST-APPROVAL-PREFLIGHT.md` | BLOCKED：tracked forbidden paths found。 |
| Blocker follow-up CR | `process/changes/CR-066-REPOSITORY-HYGIENE-CLEAN-PUBLICATION-STRATEGY-2026-06-15.md` | 用户已批准 DQ-CR062-BLOCKER-01；CR066 已关闭为 clean publication strategy evidence。 |
| Replacement execution CR | `process/changes/CR-067-CLEAN-PUBLICATION-EXECUTION-GATE-2026-06-15.md` | clean publication baseline 已完成。 |
| Replacement forward-fix CR | `process/changes/CR-068-REPOSITORY-HYGIENE-GITIGNORE-FORWARD-FIX-2026-06-15.md` | `.gitignore` forward-fix 已完成。 |
| Supersession CR | `process/changes/CR-069-CR062-SUPERSESSION-PUBLICATION-LEDGER-CLEANUP-2026-06-15.md` | CR069 CP2/CP3/CP5 approved 后执行 ledger-only cleanup；CR062 原路线收口为 closed-blocked-superseded。 |
| Scan plan | `docs/release/PUBLICATION-SCAN-PLAN-CR062.md` | secret/data/large/path boundary scan gate。 |
| Rollback | `docs/release/ROLLBACK-REF-CR062.md` | fail-closed / no history rewrite rollback strategy。 |

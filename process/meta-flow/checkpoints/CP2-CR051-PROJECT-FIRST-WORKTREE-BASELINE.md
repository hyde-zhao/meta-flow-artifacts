---
checkpoint_id: "CP2-CR051-PROJECT-FIRST-WORKTREE-BASELINE"
checkpoint_name: "CR-051 Project-first Artifact Worktree Product Baseline"
type: "auto_then_manual"
status: "changes_requested"
owner: "host-orchestrator"
created_at: "2026-07-17T08:12:28Z"
reviewed_by: "user"
reviewed_at: "2026-07-17T08:44:20Z"
auto_check_result: "process/checks/CP2-CR051-REQUIREMENTS-BASELINE.result.json"
context_ref: "process/context/CP2-CR051-REQUIREMENT-CONTEXT.yaml"
machine_context_ref: "process/context/CP2-CR051.context.json"
decision_brief_profile: "full"
target:
  phase: "requirement-clarification"
  artifacts: ["process/docs/product/USE-CASES.md", "process/docs/product/REQUIREMENTS.md", "process/docs/product/SCENARIOS.yaml", "process/docs/product/TEST-MATRIX.md", "process/docs/product/STORY-MAP.md", "process/docs/product/MVP-SCOPE.md", "process/docs/product/RELEASE-SLICES.md", "process/docs/product/BACKLOG.md"]
---

# CP2 — CR-051 Project-first Artifact Worktree 产品基线人工审查

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 豁免项 | 说明 |
|---|---|---:|---:|---|
| `process/checks/CP0-CR-051-BOOTSTRAP.result.json` | PASS | 0 | 0 | 路由健康、CR 冲突与阶段计划有效；真实 artifact/Git mutation 未授权 |
| `process/checks/CP1-CR051-USE-CASE-COMPLETENESS.result.json` | PASS | 0 | 0 | 5 个 UC-AW、15 个 TC-AW、5 个候选 Story 可追溯 |
| `process/checks/CP2-CR051-REQUIREMENTS-BASELINE.result.json` | PASS | 0 | 0 | 27 条 AW 需求、In/Out/Deferred、三项策略决策已形成基线 |

> 自动预检 `PASS` 只表示可以发起人工审查，不表示 CP2 已批准。

## Decision Brief

### 审批者摘要

| 字段 | 内容 |
|---|---|
| 本次确认服务的整体目标 | 在保留共享 artifact Git 仓库的前提下，把 artifact 管理改为 project-first 命名空间和每项目独立 worktree，并让 Git 周期只作用于源码项目与当前项目 artifact worktree。 |
| 推荐动作 | `approve`：接受 CP2-DQ-01..03 的推荐方案，以长期常驻 worktree、显式 merge-main refresh、现有 control checkout 加 sibling project worktrees 进入 CP3 详细设计。 |
| approve 后会发生什么 | Host Orchestrator 调度 meta-se 生成 BLUEPRINT、DOMAIN-MAP、DEPENDENCY-MAP、HLD 和 ADR，并在 CP3 再提交 idle detached 状态、metadata schema、owned-path gate、refresh 状态机和失败恢复的详细设计；不会直接修改源码。 |
| approve 不授权什么 | 不授权跳过 CP3/CP5，不授权源码实现，不授权真实 artifact 文件迁移、软链接变更、worktree/branch/ref mutation、remote fetch/push/default update/delete、force、凭据读取或 runtime/production 操作。 |
| 不确认会阻塞什么 | CR-051 保持在 requirement-clarification；不得启动 meta-se、生成正式 HLD/Story/LLD，也不得修改源码。 |

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule | `process/context/CP2-CR051-REQUIREMENT-CONTEXT.yaml` |
| 机器 context pack | `process/context/CP2-CR051.context.json`，`meta-flow context check` 已通过 |
| capsule 状态 | ready |
| read_profile | compact |
| 默认读取策略 | capsule first；默认只消费 must_read/allowed_reads，完整 CR、STATE、历史归档和旧 Story 均 deny-default |
| 全文档读取 | meta-pm 对 REQUEST、CR-051 和 CP result schema 的三次扩展及 Host 的一次 STATE human audit 已写入 `process/state/READ-EXPANSION-LEDGER.ndjson` |
| 最小事实 | UC-AW-001..005；REQ-AW 共 27 条；TC-AW-001..015；ST-AW-001..005 仅为产品候选 |
| 不授权边界 | 本 CR 当前真实 artifact 文件搬迁、软链接变化、真实 worktree/branch/ref mutation均为 0；源码仓仍 clean |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| current state | `process/state/STATE.current.json`、`process/current/CURRENT.json` | scanned | 0 | 0 | 只保存轻量 refs；长决策以本 checkpoint 为人工门真相源 |
| CR 边界 | `process/changes/CR-051.md`、CR summary、route plan | scanned | 3 | 3 | 三项均会改变产品行为与 CP3 架构边界 |
| 委托 Agent 交还 | `process/handoffs/CR051-CP1-CP2-META-PM-RETURN-SUMMARY.md` | scanned | 3 | 3 | DQ-01..03 全部纳入 |
| 自动预检结果 | CP0、CP1、CP2 result | scanned | 0 | 0 | blockers=0、waivers=0 |
| discussion log / checkpoint | `process/discussions/CP2-CR051-SCENARIO-DISCUSSION-LOG.md`、`process/checks/CP2-CR051-DISCUSSION-CHECKPOINT.json` | scanned | 4 | 3 | project-first 已由用户解决；其余三项为 decision-item |
| 产品基线 | 八份 `process/docs/product/**` 当前文档 | scanned | 3 | 3 | worktree 生命周期、refresh、control/worktree 拓扑 |
| 用户显式选择 | 当前对话、SGQ-AW-001/002 | scanned | 2 | 0 | project-first、独立 worktree、能力先行/迁移后置均为 resolved-by-user |
| 源码事实 | `routing.py`、`git_sync.py`、`git_branch_lifecycle.py` 与现有测试 | scanned | 3 | 0 | 证明整仓 root、同名 branch、finish 回 main 三处缺口；具体实现留给 CP3/CP5 |

### 决策分层

| 分类 | 数量 | 处理方式 |
|---|---:|---|
| 必须用户决策 | 3 | CP2-DQ-01..03；用户回复 `approve` 时接受全部推荐方案 |
| 高风险策略确认 | 1 | CP2-DQ-02 涉及 branch 历史与共享 main 收敛，但本门不授权任何真实 Git 写 |
| agent 默认处理 | 4 | schema 字段名、错误码措辞、fixture 文件名、局部模块命名由后续设计按仓库规范处理 |
| 仅审计记录 | 6 | project-first 已选、迁移后置已选、meta-pm 真实调度、源码 clean、sibling dirty 隔离事实、真实 mutation 为 0 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| CP2-DQ-01 | architecture | 每个项目的 artifact worktree 应长期常驻、每 CR 临时创建，还是采用混合模式？这决定后续软链接是否稳定、恢复入口和清理成本。 | 长期常驻 per-project worktree；active CR 使用带 project identity 的分支；idle 状态推荐 detached fresh remote default，具体细节由 CP3 冻结。 | A：每 CR 临时 worktree；B：persistent 默认，高风险或实验 CR 临时 worktree。 | 推荐方案保持软链接目标稳定并允许多项目长期并行，代价是需要 stale/prunable 健康检查和安全 remove；A 清理简单但链接不稳定且中断恢复复杂；B 灵活但维护两套状态机。 | 影响 workspace CLI、metadata、branch 占用、磁盘、repair/remove、bootstrap、迁移手册与测试。 | 若真实 pilot 证明 stale、磁盘或 branch 占用不可接受，在 CP3 或后续 CR 切换 hybrid/ephemeral；project-first 方向不回退。 |
| CP2-DQ-02 | architecture | 其他项目先合入共享 artifact main 后，当前项目 CR branch 如何显式刷新为包含 fresh main 的可合并状态？ | 独立显式 refresh：merge fresh `origin/main` 到当前 project artifact CR branch；冲突 fail closed；禁止静默 rebase、force、force-with-lease 和自动解冲突。 | A：rebase 加 force-with-lease；B：从 fresh main 重建 branch，再受控 cherry-pick 当前项目提交。 | 推荐方案不改写历史、无需 force、保留跨项目合入因果，代价是可能产生 merge commit；A 历史线性但授权和恢复风险高；B 不产生 merge commit但提交选择和映射最复杂。 | 影响 publish freshness、OID 证据、CR-050 merge/finish 适用性、冲突恢复、branch protection 和历史可读性。 | 若真实 pilot 证明 merge commit 成本不可接受，另开 CR 评估 A/B；任何切换不得静默发生，冲突不得自动解决。 |
| CP2-DQ-03 | architecture | control repo 与 project worktree 默认放在哪里，如何兼容现有 clone 并避免 nested worktree 和跨项目误写？ | 保留现有 `meta-flow-artifacts` clone 作为 control checkout；project worktree 放在 control checkout 之外的可配置 sibling root；用 project namespace 和 sparse policy 限制当前项目写入面；canonical metadata 只存锚点与相对路径。 | A：bare control repo 加 sibling project worktrees；B：sibling full worktrees，不启用 sparse 限制但继续强制 owned-path gate。 | 推荐方案无需先转换 bare，兼容现有 remote/config 和逐项目迁移；代价是必须防止 control checkout 被误用且需验证 sparse；A 控制面最干净但迁移操作更大；B Git 最简单但磁盘和误写面更大。 | 影响 workspace link/check、worktree discovery、owned-path gate、portable metadata、clean clone、回滚与迁移手册。 | 若 control checkout 持续造成误操作或 branch 占用，可在 CP3/后续迁移 CR 切 bare；若 sparse 兼容不足，可降为 full worktree，但不得取消 owned-path gate。 |

| 字段 | 内容 |
|---|---|
| 推荐决策 | `approve` 接受 CP2-DQ-01..03 的推荐方案 |
| 备选方案 | 使用 `修改: <具体修改点>` 指定 DQ ID 和目标方案；或 `reject` 停止 CR-051 |
| 影响维度 | 用户价值、实现复杂度、可验证性、维护成本、平台兼容、安全/权限和后续迁移 |
| 优劣分析 | 推荐方案优先保证稳定软链接、无历史改写、兼容现有 control clone 和逐项目迁移；代价在 CP3 显式承接 |
| 风险与回退 | 任何真实 Git/worktree/link mutation 仍需后续门禁与操作级授权；策略可在 CP3 或后续 pilot CR 按切换条件回退 |
| 用户需决策事项 | CP2-DQ-01、CP2-DQ-02、CP2-DQ-03 |

### CP2 用户意图、场景与范围摘要

| 维度 | 结论 | 证据 |
|---|---|---|
| 用户真实意图 | project-first `<project>/docs` 与 `<project>/process`；每项目独立 worktree；能力先行，迁移后置 | SGQ-AW-001/002、CR-051 |
| 场景覆盖 | 15 个 TC-AW：6 positive、4 negative、2 boundary、1 permission、1 failure-recovery、1 precheck | SCENARIOS、TEST-MATRIX |
| 认知盲区补充 | Git worktree 是整仓 checkout，不能把 tracked `<project>` 子目录直接当同仓 nested worktree；同 CR ID 跨项目需 project-namespaced branch | discussion、源码事实检查 |
| Scenario Gray Areas | 1 项 resolved-by-user；3 项 decision-item | CP2 discussion checkpoint |
| Deferred Ideas | 真实逐项目迁移、真实 shared remote pilot、bare conversion、rebase/force 线性历史方案 | MVP-SCOPE、BACKLOG |
| 用户选择影响 | 路由、worktree、Git 生命周期、兼容层、测试和迁移手册均需修改；真实迁移不计入本 CR 完成范围 | REQUIREMENTS、STORY-MAP |
| 回退方式 | CP2 前可修改三项策略；CP3 设计不可行则回 CP2；真实迁移仍是后续逐项目 CR/授权 | CR-051、BACKLOG |

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| process 路由健康 | PASS | `meta-flow workspace check`：project_name=meta-flow、routing_mode=symlink | 待人工审查 |
| CR-051 active 且 route plan 有效 | PASS | CR summary、`CP0-CR-051.route-plan.json` | 待人工审查 |
| CP1 用户场景完备门通过 | PASS | CP1 result；blockers=0 | 待人工审查 |
| 八份产品基线增量完成 | PASS | `process/docs/product/**`；旧 ID/正文/修订记录保留 | 待人工审查 |
| 工程验证与产品规划输入存在 | PASS | SCENARIOS、TEST-MATRIX、STORY-MAP、MVP-SCOPE、RELEASE-SLICES、BACKLOG | 待人工审查 |
| 用户可见 SGQ 证据存在 | PASS | SGQ-AW-001/002、discussion log/checkpoint | 待人工审查 |
| CP2 自动预检通过 | PASS | CP2 result；blockers=0、waivers=0 | 待人工审查 |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---:|---|---|---|---|
| 1 | project-first 与 per-project worktree 用户目标准确 | 待审查 | UC-AW-001..005、REQ-AW-001..010 |  |
| 2 | shared artifact sibling dirty 不阻断、current dirty 阻断 | 待审查 | REQ-AW-008..010、TC-AW-005/006 |  |
| 3 | project-namespaced branch 防止跨项目 CR ID 冲突 | 待审查 | REQ-AW-007、TC-AW-007 |  |
| 4 | shared main 显式 refresh 与 fail-closed 边界 | 待审查 | REQ-AW-011..012、TC-AW-008/009、CP2-DQ-02 |  |
| 5 | legacy dual-read 与 project-first 写目标不歧义 | 待审查 | REQ-AW-001..003、TC-AW-001..003 |  |
| 6 | 真实迁移、软链接与真实 Git mutation 明确后置 | 待审查 | REQ-AW-C001/C005、MVP-SCOPE、BACKLOG |  |
| 7 | 三项策略均有推荐、备选、风险与切换条件 | 待审查 | CP2-DQ-01..03 |  |
| 8 | approve 的授权边界清晰 | 待审查 | 审批者摘要、不授权项 |  |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| P0/P1 需求 blocker 为 0 | PASS | CP2 result |  |
| CP2-DQ-01..03 均有人工结论 | 待审查 | 本 Decision Brief |  |
| 用户明确允许进入 solution-design | 待审查 | 人工审查结果 |  |
| 真实文件/link/worktree/ref mutation 为 0 | PASS | 源码仓 clean；artifact 新写入仅限 `process/meta-flow` 工作流证据 |  |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| 产品基线 | `process/docs/product/**` 八份文件 | 待审查 |  |
| CP1 result | `process/checks/CP1-CR051-USE-CASE-COMPLETENESS.result.json` | PASS |  |
| CP2 result | `process/checks/CP2-CR051-REQUIREMENTS-BASELINE.result.json` | PASS |  |
| Discussion | `process/discussions/CP2-CR051-SCENARIO-DISCUSSION-LOG.md` | 待审查 |  |
| Context Capsule | `process/context/CP2-CR051-REQUIREMENT-CONTEXT.yaml` | ready |  |
| Meta-PM Return | `process/handoffs/CR051-CP1-CP2-META-PM-RETURN-SUMMARY.md` | returned |  |

## 人工审查结果

- 结论：`changes_requested`
- 审查人：user
- 审查时间：2026-07-17T08:44:20Z
- 修改意见：CP2-DQ-01 改为每项目长期 `projects/<project-name>/integration` 分支，项目 worktree 空闲时驻留该分支；每个 CR 使用项目命名空间下的短期 CR 分支；保留 `main` 作为共享集成基线。CP2-DQ-02、CP2-DQ-03 接受推荐方案。
- 风险接受项：无；真实 Git/worktree/link/remote mutation 仍未授权。
- 授权边界：本 checkpoint 的 `approve` 不授权任何真实文件迁移、软链接、worktree/ref/remote mutation、源码实现、凭据或 runtime/production 操作。

## 可接受回复

- `approve`
- `修改: <具体修改点>`
- `reject`

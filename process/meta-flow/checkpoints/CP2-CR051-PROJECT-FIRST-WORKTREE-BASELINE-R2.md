---
checkpoint_id: "CP2-CR051-PROJECT-FIRST-WORKTREE-BASELINE-R2"
checkpoint_name: "CR-051 Project-first Artifact Worktree Product Baseline R2"
type: "auto_then_manual"
status: "changes_requested"
owner: "host-orchestrator"
created_at: "2026-07-17T09:02:59Z"
reviewed_by: "user"
reviewed_at: "2026-07-18T02:55:21Z"
auto_check_result: "process/checks/CP2-CR051-REQUIREMENTS-BASELINE-R2.result.json"
context_ref: "process/context/CP2-CR051-REQUIREMENT-CONTEXT-R2.yaml"
machine_context_ref: "process/context/CP2-CR051-R2.context.json"
decision_brief_profile: "compact"
supersedes_checkpoint: "process/checkpoints/CP2-CR051-PROJECT-FIRST-WORKTREE-BASELINE.md"
target:
  phase: "requirement-clarification"
  artifacts: ["process/docs/product/USE-CASES.md", "process/docs/product/REQUIREMENTS.md", "process/docs/product/SCENARIOS.yaml", "process/docs/product/TEST-MATRIX.md", "process/docs/product/STORY-MAP.md", "process/docs/product/MVP-SCOPE.md", "process/docs/product/RELEASE-SLICES.md", "process/docs/product/BACKLOG.md"]
---

# CP2 — CR-051 Project-first Artifact Worktree 产品基线 R2 人工审查

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 豁免项 | 说明 |
|---|---|---:|---:|---|
| `process/checks/CP0-CR-051-BOOTSTRAP.result.json` | PASS | 0 | 0 | 路由健康；真实 artifact/Git mutation 未授权 |
| `process/checks/CP1-CR051-USE-CASE-COMPLETENESS.result.json` | PASS | 0 | 0 | UC/TC ID 和数量未变化，R1 CP1 继续有效 |
| `process/checks/CP2-CR051-REQUIREMENTS-BASELINE-R2.result.json` | PASS | 0 | 0 | 27 条 AW 需求、15 个 TC-AW、5 个候选 Story；三项 DQ 已按用户回答收敛 |

> 自动预检 `PASS` 只表示 R2 可以发起人工审查，不表示 CP2 已批准。

> 2026-07-18 用户在人工评审中进一步明确源码仓与共享 artifact 仓使用不同 Git 生命周期、artifact CR 不接触 shared `main`、双 leg 采用单一聚合门，并批准 integration 从 fresh `origin/main` exact OID create-only 初始化。该输入使 R2 的 merge-main refresh 语义失效；R2 保留为 `changes_requested` 历史证据，由 R3 替代。

## Decision Brief

### 审批者摘要

| 字段 | 内容 |
|---|---|
| 本次确认服务的整体目标 | 确认 project-first artifact、每项目独立 worktree、长期项目 integration 分支、短期 CR 分支和共享 `main` 集成基线构成 CR-051 的产品基线。 |
| 推荐动作 | `approve`：批准 R2 产品基线，允许进入 CP3 设计；三项产品策略已按用户上一轮回答固定，不再作为未决 DQ。 |
| approve 后会发生什么 | Host Orchestrator 调度 meta-se 生成 BLUEPRINT、DOMAIN-MAP、DEPENDENCY-MAP、HLD 和 ADR；CP3 冻结 metadata/path schema、owned-path gate、attach/switch/finish/abort、精确 Git/OID 状态机、冲突恢复与 sparse 行为。 |
| approve 不授权什么 | 不授权跳过 CP3/CP5，不授权源码实现，不授权真实 artifact 文件迁移、软链接变更、worktree/branch/ref mutation、remote fetch/push/default update/delete、force、凭据读取或 runtime/production 操作。 |
| 不确认会阻塞什么 | CR-051 保持在 requirement-clarification；不得启动 meta-se、生成正式 HLD/Story/LLD 或修改源码。 |

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP2-CR051-REQUIREMENT-CONTEXT-R2.yaml` |
| 机器 context pack | `process/context/CP2-CR051-R2.context.json` |
| capsule 状态 | ready |
| read_profile | compact |
| 默认读取策略 | capsule first；默认只消费 must_read/allowed_reads，完整 CR、STATE、discussion、archive 和旧 Story 均 deny-default |
| 全文档读取扩展 | Host 为回写 changes_requested 五维影响读取完整 CR-051，事件 `RE-20260717T084624Z0000-a76f6628`；其余 R2 消费正式摘要和获准产品文档 |
| 最小事实 | UC-AW=5；REQ-AW=27；TC-AW=15/全局75；候选 Story=5；Slice=4；SGQ-AW=3 |
| 不授权边界 | 源码修改、真实 artifact 迁移、软链接变化、真实 worktree/branch/ref/remote mutation 均为 0 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| current state | `process/state/STATE.current.json`、`process/current/CURRENT.json` | scanned | 0 | 0 | 轻量状态；R2 gate 与 context refs 由 Host 回写 |
| R1 人工门 | `process/checkpoints/CP2-CR051-PROJECT-FIRST-WORKTREE-BASELINE.md` | scanned | 3 | 0 | `changes_requested` 已保存；三项回答进入 R2 resolved 决策 |
| 用户显式回答 | 当前对话、SGQ-AW-003 | scanned | 3 | 0 | DQ-01 采用长期 integration + 短期 CR + shared main；DQ-02/03 接受推荐，均 resolved-by-user |
| meta-pm R2 交还 | `process/handoffs/CR051-CP2-R2-META-PM-RETURN-SUMMARY.md` | scanned | 0 | 0 | 产品计数不变；剩余六项全部路由 CP3 设计细化 |
| 自动预检 | CP0、CP1、CP2 R2 result | scanned | 0 | 0 | blockers=0、waivers=0 |
| discussion log/checkpoint | `process/discussions/CP2-CR051-SCENARIO-DISCUSSION-LOG.md`、`process/checks/CP2-CR051-DISCUSSION-CHECKPOINT.json` | scanned | 3 | 0 | `decision_items=[]`；三项进入 `resolved_decision_items` |
| 八份产品基线 | `process/docs/product/**` 当前 CR-051 条目 | scanned | 0 | 0 | branch role、refresh、topology、In/Out/Deferred 已一致 |
| 源码事实 | `git_branch_lifecycle.py`、`git_sync.py` | scanned | 6 | 0 | 精确实现问题均属于 CP3/CP5，不在 CP2 再决策 |

### 决策分层

| 分类 | 数量 | 处理方式 |
|---|---:|---|
| 必须用户决策 | 0 | 三项策略均已由用户解决；本轮只需对 R2 整体门给出 approve / 修改 / reject |
| 高风险策略确认 | 0 | DQ-02 已按用户回答固定为显式 merge-main、冲突 fail closed、无 rebase/force |
| agent 默认处理 | 6 | metadata、owned-path gate、attach/switch/finish/abort、OID、冲突恢复和 sparse 由 meta-se 在 CP3 给出可审查设计 |
| 仅审计记录 | 9 | project-first、迁移后置、三项 resolved DQ、CP1 有效、meta-pm dispatch、源码 clean、真实 mutation 为 0 |

### 待人工决策清单

本轮待人工决策项：0，原因：三项产品策略均已由用户明确回答；当前需要的是对修订后完整基线的 CP2 门禁结论，而不是再次选择策略。

### 已解决决策基线

| 决策 ID | 决策类型 | 用户已选择的 R2 基线 | 状态 | CP3 允许细化但不得改变的边界 |
|---|---|---|---|---|
| CP2-DQ-01 | architecture | idle=`projects/<project-name>/integration`；active=`projects/<project-name>/cr/<cr-id>-<slug>`；`main`=共享集成基线 | resolved-by-user | 可细化 attach/switch/finish/abort、OID 和 cleanup，不得改回 idle detached 或长期占用 CR branch |
| CP2-DQ-02 | architecture | 显式 merge fresh `origin/main`；冲突 fail closed；禁止静默 rebase、force、force-with-lease、自动解冲突 | resolved-by-user | 可细化 refresh/abort/resume 命令和 CAS/OID 证据，不得引入隐式历史改写 |
| CP2-DQ-03 | architecture | existing control checkout + configurable sibling worktree root + project namespace/sparse policy + owned-path gate | resolved-by-user | 可细化 portable metadata、sparse 与 repair，不得创建 nested project worktree 或取消 owned-path gate |

| 字段 | 内容 |
|---|---|
| 推荐决策 | `approve` 批准 R2 整体产品基线并进入 CP3 |
| 备选方案 | `修改: <具体修改点>` 返回 requirement-clarification 再修订；或 `reject` 停止 CR-051 |
| 影响维度 | 用户价值、实现复杂度、可验证性、维护成本、Git 并发、平台兼容、安全/权限和后续逐项目迁移 |
| 优劣分析 | R2 以稳定项目分支承载日常工作、短期 CR 分支承载审查/回退、共享 main 承载完整集成；增加一层分支角色和状态机，但保留单仓完整快照与逐项目迁移能力 |
| 风险与回退 | CP3 若证明精确状态机不可实现则退回 CP2；真实 pilot 不满足时另开后续 CR，不静默改写本基线 |
| 用户需决策事项 | 无未解决 DQ；只需给出 CP2 R2 整体门禁结论 |

### CP2 用户意图、场景与范围摘要

| 维度 | 结论 | 证据 |
|---|---|---|
| 用户真实意图 | project-first；独立项目 worktree；长期 integration、短期 CR、shared main；能力先行、迁移后置 | SGQ-AW-001..003、CR-051 |
| 场景覆盖 | 15 个 TC-AW，覆盖正向、负向、边界、权限、失败恢复和 precheck | SCENARIOS、TEST-MATRIX |
| 认知盲区补充 | Git worktree 是整仓 checkout；branch 是整仓历史而非目录；项目 branch 名必须包含 project identity；ref 名使用独立 integration/CR 叶节点避免前缀冲突 | REQUIREMENTS、discussion |
| Scenario Gray Areas | SGA-AW-01..04 全部 resolved-by-user | discussion checkpoint |
| Deferred Ideas | 真实逐项目迁移、真实 shared remote pilot、bare conversion、rebase/force 方案 | MVP-SCOPE、BACKLOG |
| 用户选择影响 | Git 周期必须从“同名双仓分支”改为显式 source branch ↔ artifact project integration/CR branch 映射；finish 不删除项目 integration | REQUIREMENTS、STORY-MAP |
| 回退方式 | CP3 设计不可行则回 CP2；实现问题回 CP5；真实迁移仍需后续逐项目 CR/授权 | CR-051、BACKLOG |

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| process 路由健康 | PASS | `meta-flow workspace check`：project_name=meta-flow、routing_mode=symlink | 待人工审查 |
| R1 changes_requested 已留痕 | PASS | R1 checkpoint、GATE-LEDGER | 待人工审查 |
| meta-pm R2 真实调度完成 | PASS | `DISPATCH-CR051-CP2-R2-PM`、R2 handoff/return | 待人工审查 |
| 八份产品基线增量完成 | PASS | `process/docs/product/**`；旧 ID/正文/修订记录保留 | 待人工审查 |
| Discussion 与 SGQ-AW-003 完整 | PASS | discussion log/checkpoint | 待人工审查 |
| CP1 继续有效 | PASS | UC/TC ID 与数量不变；CP1 result PASS | 待人工审查 |
| CP2 R2 自动预检通过 | PASS | R2 result；blockers=0、waivers=0 | 待人工审查 |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---:|---|---|---|---|
| 1 | project-first 与 per-project worktree 目标准确 | 待审查 | UC-AW-001..005、REQ-AW-001..003 |  |
| 2 | idle integration、active CR、shared main 三类 branch role 明确 | 待审查 | REQ-AW-004/007、TC-AW-004/007、SGQ-AW-003 |  |
| 3 | branch 命名包含 project identity 且避免 ref 前缀冲突 | 待审查 | REQ-AW-007、TC-AW-007 |  |
| 4 | shared main 显式 refresh 与 fail-closed 边界明确 | 待审查 | REQ-AW-011..012、TC-AW-008/009 |  |
| 5 | control checkout、sibling worktree、sparse/owned-path 边界明确 | 待审查 | REQ-AW-013..015、TC-AW-011/012 |  |
| 6 | legacy dual-read 与 project-first 单写不歧义 | 待审查 | REQ-AW-001..003、TC-AW-001..003 |  |
| 7 | 真实迁移、软链接和真实 Git mutation 明确后置 | 待审查 | MVP-SCOPE、BACKLOG、不授权项 |  |
| 8 | CP3 细化项不会重新打开已解决产品决策 | 待审查 | R2 capsule、resolved decision table |  |
| 9 | approve 的授权边界清晰 | 待审查 | 审批者摘要、不授权项 |  |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| P0/P1 需求 blocker 为 0 | PASS | R2 result |  |
| 未解决 CP2 DQ 为 0 | PASS | discussion checkpoint `decision_items=[]` |  |
| 用户明确允许进入 solution-design | 待审查 | 本人工门结论 |  |
| 真实文件/link/worktree/ref/remote mutation 为 0 | PASS | 源码仓 clean；真实 worktree list 未变化 |  |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| R2 产品基线 | `process/docs/product/**` 八份文件 | 待审查 |  |
| CP1 result | `process/checks/CP1-CR051-USE-CASE-COMPLETENESS.result.json` | PASS |  |
| CP2 R2 result | `process/checks/CP2-CR051-REQUIREMENTS-BASELINE-R2.result.json` | PASS |  |
| R2 Discussion | `process/discussions/CP2-CR051-SCENARIO-DISCUSSION-LOG.md`、discussion checkpoint | 待审查 |  |
| R2 Context Capsule | `process/context/CP2-CR051-REQUIREMENT-CONTEXT-R2.yaml` | ready |  |
| R2 Meta-PM Return | `process/handoffs/CR051-CP2-R2-META-PM-RETURN-SUMMARY.md` | returned |  |

## 人工审查结果

- 结论：`changes_requested`
- 审查人：user
- 审查时间：2026-07-18T02:55:21Z
- 修改意见：源码仓与 artifact 仓必须采用不同 base/target；artifact CR 仅从最新项目 integration 创建并回到该 integration，不得 refresh 或直接交互 shared `main`。补充双 leg 聚合门、integration create-only 初始化和 CR 外人工同步边界后，以 R3 重新提交 CP2。
- 风险接受项：无；三项策略为产品基线选择，不是对真实 Git mutation 的风险授权。
- 授权边界：本 checkpoint 的 `approve` 只允许进入 CP3；不授权源码实现、文件迁移、软链接、worktree/ref/remote mutation、凭据或 runtime/production 操作。

## 可接受回复

- `approve`
- `修改: <具体修改点>`
- `reject`

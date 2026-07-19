---
checkpoint_id: "CP2-CR051-PROJECT-FIRST-WORKTREE-BASELINE-R3"
checkpoint_name: "CR-051 Project-first Artifact Worktree Product Baseline R3"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-07-18T03:19:17Z"
reviewed_by: "user"
reviewed_at: "2026-07-18T03:37:15Z"
auto_check_result: "process/checks/CP2-CR051-REQUIREMENTS-BASELINE-R3.result.json"
context_ref: "process/context/CP2-CR051-REQUIREMENT-CONTEXT-R3.yaml"
machine_context_ref: "process/context/CP2-CR051-R3.context.json"
decision_brief_profile: "compact"
supersedes_checkpoint: "process/checkpoints/CP2-CR051-PROJECT-FIRST-WORKTREE-BASELINE-R2.md"
target:
  phase: "requirement-clarification"
  artifacts: ["process/docs/product/USE-CASES.md", "process/docs/product/REQUIREMENTS.md", "process/docs/product/SCENARIOS.yaml", "process/docs/product/TEST-MATRIX.md", "process/docs/product/STORY-MAP.md", "process/docs/product/MVP-SCOPE.md", "process/docs/product/RELEASE-SLICES.md", "process/docs/product/BACKLOG.md"]
---

# CP2 — CR-051 Project-first Artifact Worktree 产品基线 R3 人工审查

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 豁免项 | 说明 |
|---|---|---:|---:|---|
| `process/checks/CP0-CR-051-BOOTSTRAP.result.json` | PASS | 0 | 0 | process 路由健康；真实 artifact/Git mutation 未授权 |
| `process/checks/CP1-CR051-USE-CASE-COMPLETENESS-R3.result.json` | PASS | 0 | 0 | UC-AW=5、TC-AW=15，异常、边界、权限与恢复场景完整 |
| `process/checks/CP2-CR051-REQUIREMENTS-BASELINE-R3.result.json` | PASS | 0 | 0 | REQ-AW=27、候选 Story=5、Slice=4；R3 决策分类完整 |

> 自动预检 `PASS` 只表示 R3 可以发起人工审查，不表示 CP2 已批准，也不表示已授权真实 Git/worktree/link/remote 操作。

## Decision Brief

### 审批者摘要

| 字段 | 内容 |
|---|---|
| 本次确认服务的整体目标 | 确认 CR-051 的最终产品基线：源码交付仓与共享 artifact 仓采用不同 Git 生命周期；artifact 仓使用每项目长期 integration、每 CR 短期分支和 CR 外人工 shared-main 同步，并由一个逻辑 CR 聚合两个异构 leg 的结果。 |
| 推荐动作 | `approve`：批准 R3 产品基线并允许进入 CP3 方案设计。当前 DQ 已按用户明确输入全部 resolved 或 superseded，不需要再次选择策略。 |
| approve 后会发生什么 | Host Orchestrator 调度 meta-se 生成 BLUEPRINT、DOMAIN-MAP、DEPENDENCY-MAP、HLD 和 ADR；CP3 细化双 leg 命令/OID/cleanup 状态机、aggregate schema、bootstrap CAS、ledger 单写、owned-path gate 与 CR 外人工同步 precheck，并再次发起 CP3 人工门。 |
| approve 不授权什么 | 不授权跳过 CP3/CP5，不授权源码实现，不授权真实 artifact 文件迁移、软链接变更、worktree/branch/ref mutation、main/integration 同步、remote fetch/push/default update/delete、force、凭据读取或 runtime/production/publish/trading 操作。 |
| 不确认会阻塞什么 | CR-051 保持在 requirement-clarification；不得启动 meta-se、生成正式 HLD/Story/LLD、修改源码或执行真实仓库操作。 |

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP2-CR051-REQUIREMENT-CONTEXT-R3.yaml` |
| 机器 context pack | `process/context/CP2-CR051-R3.context.json` |
| capsule 状态 | ready |
| read_profile | compact |
| 默认读取策略 | capsule first；默认只消费 must_read/allowed_reads，完整 CR、STATE、discussion、archive、旧 Story 和 sibling project 均 deny-default |
| 全文档读取 | Host 对 CR、USE-CASES、REQUIREMENTS 做 deep-review/field-conflict 扩展；meta-pm 对 discussion 做 human-audit 扩展，四条事件均已写入 READ-EXPANSION-LEDGER |
| 最小事实 | UC-AW=5；REQ-AW=27；TC-AW=15/全局75；候选 Story=5；Slice=4；SGA-AW=4；SGQ-AW=3 |
| 不授权边界 | 源码实现、真实 artifact 迁移、软链接变化、真实 worktree/branch/ref/remote mutation、真实 main/integration 同步均为 0 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| current state | `process/state/STATE.current.json`、`process/current/CURRENT.json` | scanned | 0 | 0 | 轻量状态；R3 gate/context refs 由 Host 在开门时回写 |
| R2 人工门 | `process/checkpoints/CP2-CR051-PROJECT-FIRST-WORKTREE-BASELINE-R2.md` | scanned | 3 | 0 | R2 已 `changes_requested`；DQ-01/03 保留，DQ-02 被 R3 supersede |
| 用户显式输入 | 当前对话、SGQ-AW-003、`CP2-CR051-R3-USER-DECISIONS.json` | scanned | 3 | 0 | DQ-04..06 均 `resolved-by-user`，无未决策略 |
| meta-pm R3 交还 | `process/handoffs/CR051-CP2-R3-META-PM-RETURN-SUMMARY.md` | scanned | 0 | 0 | 固定计数与 15 文件边界通过；七项路由 CP3 细化 |
| 自动预检 | CP0、CP1 R3、CP2 R3 result | scanned | 0 | 0 | blockers=0、waivers=0，result schema 校验通过 |
| discussion log/checkpoint | `process/discussions/CP2-CR051-SCENARIO-DISCUSSION-LOG.md`、discussion checkpoint | scanned | 6 | 0 | DQ-01/03/04/05/06 resolved；DQ-02 superseded；`decision_items=[]` |
| 八份产品基线 | `process/docs/product/**` 当前 CR-051 条目 | scanned | 0 | 0 | 双 leg、聚合门、bootstrap、In/Out/Deferred 与 R3 一致 |
| 主编排器独立复核 | result-check、YAML/context parse、stale scan、`git diff --check` | scanned | 0 | 0 | 全部 PASS；旧 refresh 只在历史、superseded 或禁止性表述中 |

### 决策分层

| 分类 | 数量 | 处理方式 |
|---|---:|---|
| 必须用户决策 | 0 | 没有未解决 DQ；本轮仍需用户对完整 R3 基线给出 approve / 修改 / reject 的总体门禁结论 |
| 高风险策略确认 | 0 | 异构双 leg、全 PASS 聚合、create-only bootstrap、CR 外人工同步及禁止自动 rollback 均已由用户确认 |
| agent 默认处理 | 7 | metadata/path、namespace/sparse/owned-path、双 leg 命令/OID/cleanup、aggregate schema、ledger/receipt、bootstrap CAS、人工同步 precheck、失败恢复由 meta-se 在 CP3 形成可审查设计 |
| 仅审计记录 | 10 | project-first、迁移后置、五项 resolved DQ、一项 superseded DQ、固定 ID/计数、meta-pm dispatch、零真实 mutation |

### 待人工决策清单

本轮待人工决策项：0，原因：CP2-DQ-01、03、04、05、06 已由用户明确确认，CP2-DQ-02 已被用户后续方案替代；当前只需要对修订后的完整 R3 产品基线给出总体门禁结论，无新增取舍。

### 已解决与已替代决策基线

| 决策 | 决策类型 | 当前 R3 基线 | 状态 | CP3 可细化但不得改变的边界 |
|---|---|---|---|---|
| CP2-DQ-01 | architecture | idle=`projects/<project>/integration`；active=`projects/<project>/cr/<cr-id>-<slug>`；shared `main`=共享集成基线 | resolved-by-user | 不得改回 idle detached、main idle 或长期占用 CR branch |
| CP2-DQ-02 | architecture | R2 的 per-CR merge-main/refresh 不再生效 | superseded-by-user / CP2-DQ-04 | 不得在 CP3 恢复为 artifact CR 直接接触 shared main |
| CP2-DQ-03 | architecture | existing control checkout + configurable sibling root + namespace/sparse/owned-path gate | resolved-by-user | 可细化 portable metadata、sparse 与 repair，不得取消项目隔离和 owned-path gate |
| CP2-DQ-04 | architecture | source 从/回源码默认分支；artifact 从/回同项目 integration；artifact CR 不接触 shared main | resolved-by-user | 可细化精确命令、expected OID、cleanup 和恢复，不得统一成 paired-default |
| CP2-DQ-05 | architecture | 单一协调者按 `BLOCKED > FAIL > IN_PROGRESS > PASS` 聚合；仅全部必需 leg PASS 才完成；`PARTIAL` 非终态 | resolved-by-user | 可细化 schema/ledger/resume/abort，不得把单 leg PASS 或 PARTIAL 报为整体完成 |
| CP2-DQ-06 | architecture | integration 仅在远端 ref 缺失时从 fresh `origin/main` exact OID create-only 初始化；main↔integration 同步在 CR 外人工执行 | resolved-by-user | 可细化 CAS、竞态、幂等、precheck 与冲突恢复，不得自动 recreate/reset/orphan 或塞入单 CR |

| 字段 | 内容 |
|---|---|
| 推荐决策 | `approve` 批准 R3 整体产品基线并进入 CP3 |
| 备选方案 | `修改: <具体修改点>` 返回 requirement-clarification 再修订；或 `reject` 停止 CR-051 |
| 影响维度 | 用户使用路径、跨仓完成语义、实现复杂度、可验证性、Git 并发、平台兼容、安全/权限、恢复能力和后续逐项目迁移 |
| 优劣分析 | R3 将项目内 CR 交付与跨项目 shared-main 同步分离，稳定 worktree 基点且避免 artifact CR 越权触碰 main；代价是双 leg 聚合、长期 integration 和人工同步需要更清晰的状态机与审计证据 |
| 风险与回退 | CP3 若证明精确状态机不可实现则退回 CP2；真实 pilot 不满足时另开后续 CR；任何失败都不得静默切回 per-CR main refresh、自动 rollback 或跨仓原子性声明 |
| 用户需决策事项 | 无未解决 DQ；只需给出 CP2 R3 整体门禁结论 |

### CP2 用户意图、场景与范围摘要

| 维度 | 结论 | 证据 |
|---|---|---|
| 用户真实意图 | project-first；每项目独立 worktree；artifact 长期 integration + 短期 CR；source 仍走 default + CR；shared main 同步在 CR 外人工执行 | SGQ-AW-001..003、DQ-01/03/04/05/06、CR-051 |
| 双 leg 完成语义 | 两 leg 独立出结果，单一协调者按最差状态聚合，仅全 PASS 完成；不自动回滚成功 leg | UC-AW-003/004、REQ-AW-008/012/016、TC-AW-008/009 |
| integration 初始化 | 远端 ref 缺失时从 fresh origin/main exact OID create-only；存在时不 recreate/reset/orphan | REQ-AW-004、TC-AW-004 |
| 场景覆盖 | 15 个 TC-AW，覆盖正向、负向、边界、权限、失败恢复与 precheck | SCENARIOS、TEST-MATRIX |
| Deferred Ideas | 真实逐项目迁移、真实 shared remote pilot、自动 main/integration 同步、bare conversion、rebase/force 方案 | MVP-SCOPE、BACKLOG |
| 回退方式 | CP3 设计不可行则回 CP2；实现问题回 CP5；真实迁移、软链接和同步仍需后续逐项目 CR/授权 | CR-051、BACKLOG |

### CP3 强制设计交接约束：worktree branch switch 非原子性

用户在 CP2 总体批准前指出：`REQ-AW-004` 的“空闲驻留 integration、CR 活动时切到 CR 分支”不能被实现或文档描述成原子事务。该问题不改变 CP2 产品范围，作为 `CP3-DC-01` 进入 CP3 强制设计输入，状态为 `accepted-design-constraint`。

CP3 必须采用“可判定、可恢复、失败不误报”的切换协议：

1. 切换前验证当前项目 worktree identity/ownership、操作互斥锁、clean 状态、无进行中的 Git operation、source/target branch 与 expected OID、worktree 与 gitdir 的写权限，以及两个所在文件系统的可用空间。
2. 空间门必须量化：推荐 `required_free_bytes = max(512 MiB, 1.5 × estimated_checkout_write_bytes)`，worktree filesystem 与 gitdir filesystem 分别检查；无法可靠估算时 fail closed，不得假定空间足够。
3. 执行切换前在 worktree 之外的受控状态目录持久化 operation intent，至少记录 operation/attempt ID、project、worktree path、original branch/OID、target branch/OID、expected integration OID 和阶段。
4. `git switch` 返回成功不等于操作完成；只有重新观察 symbolic HEAD、HEAD OID、worktree registration 和 clean 状态与目标一致后，才能写入 `VERIFIED/PASS`。
5. 异常后必须先重新观察实际状态，不得根据命令退出点猜测当前 branch。仅当 worktree clean、没有进行中的 Git operation、原 integration ref/OID 未漂移、权限和空间复检通过时，才允许自动回到 integration。
6. 自动回退不安全或失败时，必须保留当前 worktree 和 CR branch，进入 `RECOVERY_REQUIRED/BLOCKED`，输出基于实际观测的人工恢复入口；禁止自动 `reset --hard`、`clean`、stash、force、删除 branch 或覆盖用户文件。
7. resume/recover 必须幂等；重复执行只能得到已恢复、继续恢复或仍阻断的可复算结果，不得把不确定现场误报为 integration idle。

上述约束是 CP3 的 HLD/ADR/状态机验收项，不是本轮新增 CP2 DQ；CP3 不得将其降级为一般实现建议。

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| process 路由健康 | PASS | `meta-flow workspace check`：project_name=meta-flow、routing_mode=symlink | 通过 |
| R2 changes_requested 已留痕 | PASS | R2 checkpoint、GATE-LEDGER、DQ-02 supersession | 通过 |
| meta-pm R3 真实调度完成 | PASS | `ATTEMPT-CR051-CP2-R3-PM-01`、handoff/return/dispatch ledger | 通过 |
| 八份产品基线增量完成 | PASS | `process/docs/product/**`；R1/R2 修订历史保留 | 通过 |
| Discussion 与 SGQ-AW-003 完整 | PASS | discussion log/checkpoint、R3 decision JSON | 通过 |
| CP1 R3 自动检查通过 | PASS | CP1 R3 result；blockers=0、waivers=0 | 通过 |
| CP2 R3 自动预检通过 | PASS | CP2 R3 result；blockers=0、waivers=0 | 通过；worktree switch 非原子性已转 CP3-DC-01 |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---:|---|---|---|---|
| 1 | project-first 与 per-project worktree 目标准确 | 通过 | UC-AW-001..005、REQ-AW-001..003 |  |
| 2 | artifact idle integration、active CR、shared main 三类 branch role 明确 | 通过 | REQ-AW-004/007、TC-AW-004/007、SGQ-AW-003 |  |
| 3 | source-default 与 artifact-integration 两个 leg 的 base/target 差异明确 | 通过 | REQ-AW-008/011、TC-AW-008/009 |  |
| 4 | 聚合优先级、全 PASS 完成、PARTIAL 非终态和不自动回滚明确 | 通过 | REQ-AW-012/016、TC-AW-008/009/015 |  |
| 5 | integration create-only bootstrap 与 existing-ref 禁止覆盖明确 | 通过 | REQ-AW-004、TC-AW-004 |  |
| 6 | shared main 同步位于 CR 外、由人工执行且默认无活跃 artifact CR | 通过 | REQ-AW-005/011、TC-AW-004/009 |  |
| 7 | control checkout、sibling worktree、sparse/owned-path 边界明确 | 通过 | REQ-AW-013..015、TC-AW-011/012 |  |
| 8 | CR-050 paired-default 只在适用前提成立，shared-artifact 采用显式覆盖 | 通过 | CP2 R3 result CP2-07、REQUIREMENTS |  |
| 9 | 真实迁移、软链接、真实 Git mutation 和真实同步明确后置 | 通过 | MVP-SCOPE、BACKLOG、不授权项 |  |
| 10 | approve 授权边界与 CP3 细化边界清晰 | 通过 | 审批者摘要、决策分层、Context Capsule、CP3-DC-01 | switch 非原子性必须进入 HLD/ADR |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| P0/P1 需求 blocker 为 0 | PASS | CP2 R3 result |  |
| 未解决 CP2 DQ 为 0 | PASS | discussion checkpoint `decision_items=[]`、R3 decision JSON |  |
| 用户明确允许进入 solution-design | PASS | 本人工门结论 | 用户要求解决 switch 非原子性后批准 CP2 并推进到下一人工门 |
| 真实文件/link/worktree/ref/remote/main-sync mutation 为 0 | PASS | 本轮执行边界与检查记录 |  |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| R3 产品基线 | `process/docs/product/**` 八份文件 | 通过 |  |
| CP1 R3 result | `process/checks/CP1-CR051-USE-CASE-COMPLETENESS-R3.result.json` | PASS |  |
| CP2 R3 result | `process/checks/CP2-CR051-REQUIREMENTS-BASELINE-R3.result.json` | PASS |  |
| R3 Discussion / Decisions | discussion log、discussion checkpoint、R3 decision JSON | 通过 |  |
| R3 Context Capsule | `process/context/CP2-CR051-REQUIREMENT-CONTEXT-R3.yaml`、machine context | ready |  |
| R3 Meta-PM Return | `process/handoffs/CR051-CP2-R3-META-PM-RETURN-SUMMARY.md` | returned |  |

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-07-18T03:37:15Z
- 修改意见：CP2 产品基线无需重开；worktree branch switch 的非原子性必须作为 `CP3-DC-01` 写入 HLD/ADR，覆盖 precheck、持久化 intent、postcheck、条件自动回退和人工恢复。
- 风险接受项：无。用户没有接受切换中间态风险；设计必须 fail closed，并在状态不确定时进入 `RECOVERY_REQUIRED/BLOCKED`。
- 授权边界：本 checkpoint 的 `approve` 只允许进入 CP3；不授权源码实现、真实文件迁移、软链接、worktree/ref/remote/main-sync mutation、凭据或 runtime/production 操作。

## 可接受回复

- `approve`
- `修改: <具体修改点>`
- `reject`

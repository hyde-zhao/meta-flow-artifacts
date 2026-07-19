---
handoff_id: "CR051-CP1-CP2-META-PM-RETURN"
cr_id: "CR-051"
phase: "requirement-clarification"
from_agent: "meta-pm"
to: "host-orchestrator"
status: "returned"
created_at: "2026-07-17T16:04:17+08:00"
context_ref: "process/context/CP2-CR051-REQUIREMENT-CONTEXT.yaml"
cp1_result_ref: "process/checks/CP1-CR051-USE-CASE-COMPLETENESS.result.json"
cp2_result_ref: "process/checks/CP2-CR051-REQUIREMENTS-BASELINE.result.json"
next_required_gate: "CP2-human"
---

# CR-051 CP1/CP2 Meta-PM Return Summary

## 交还结论

- CP1自动检查：`PASS`，blockers=0，waivers=0。
- CP2自动预检：`PASS`，blockers=0，waivers=0。
- 人工门禁：`pending`；本交还不发起CP2，也不授权CP3、Story拆分、LLD或实现。
- `ready_for_design=false`，直到Host Orchestrator完成CP2 Decision Brief并获得用户`approve`/修改/reject结论。

## 用户真实意图

1. artifact采用project-first语义：`<project_name>/docs`与`<project_name>/process`。
2. 不同项目使用独立worktree，仍共享artifact Git仓库/对象/remote，避免共享checkout、index和branch。
3. 当前先完成Meta Flow的路由、worktree管理和项目作用域Git周期能力；真实artifact文件迁移与软链接挂接由用户后续逐项目执行。

## Scenario Gray Areas

| Gray Area | 用户选择 / 推荐 | 状态 | 证据 |
|---|---|---|---|
| SGA-AW-01 project-first vs type-first | 用户选择project-first + per-project worktree | resolved-by-user | SGQ-AW-001 |
| SGA-AW-02 persistent vs ephemeral | 推荐persistent；idle态候选为detached fresh remote default，CP3冻结 | decision-item / CP2-DQ-01 | discussion log/checkpoint |
| SGA-AW-03 shared-main refresh | 推荐显式merge fresh main到project CR branch；冲突fail-closed | decision-item / CP2-DQ-02 | discussion log/checkpoint |
| SGA-AW-04 control/worktree topology | 推荐existing control checkout + configurable sibling worktree root + project namespace/sparse policy | decision-item / CP2-DQ-03 | discussion log/checkpoint |

Canonical refs：

- `process/discussions/CP2-CR051-SCENARIO-DISCUSSION-LOG.md`
- `process/checks/CP2-CR051-DISCUSSION-CHECKPOINT.json`
- `process/docs/product/USE-CASES.md#cr-051-用户可见场景确认证据`

## 用户可见SGQ证据

| SGQ | 问题摘要 | 用户回答 | 复述确认 | 状态 |
|---|---|---|---|---|
| SGQ-AW-001 | 是否采用project-first + per-project worktree | 用户明确选择项目优先并要求不同worktree管理 | `<project>/docs|process`；每项目隔离checkout/index/branch，仍共享仓库 | confirmed |
| SGQ-AW-002 | 能力先行还是立即全量迁移 | 用户明确后续逐项目迁移文件并挂软链接 | CR-051只交付能力/fixture/preflight/手册；真实迁移/link/ref mutation为0 | confirmed |

## 产品基线增量摘要

| 工件 | CR-051增量 | 当前版本 |
|---|---|---:|
| `USE-CASES.md` | UC-AW-001..005、SM/SGA/SGQ/Deferred/8维覆盖 | 1.7 |
| `REQUIREMENTS.md` | REQ-AW-001..017、C001..005、NF001..005、RA-AW-001..006、M-AW1..4 | 1.8 |
| `SCENARIOS.yaml` | TC-AW-001..015 | 1.7 |
| `TEST-MATRIX.md` | 15/15 TC→REQ→候选Story矩阵 | 2.0 |
| `STORY-MAP.md` | 4个Activity、ST-AW-001..005、SL-AW-01..04 | 1.7 |
| `MVP-SCOPE.md` | IN-AW-001..005、OUT-AW-001..006、DEF-AW-001..003 | 1.7 |
| `RELEASE-SLICES.md` | SL-AW-01..04 | 1.5 |
| `BACKLOG.md` | BL-AW-001..005、NA-AW-001..005 | 1.5 |

旧UC/REQ/TC/Story/Slice/Backlog ID、正文和修订记录全部保留；CR-050历史正文未改写。CR-051只新增适用性规则：shared-artifact模式下，CR-050的artifact role解析为当前project artifact worktree + owned namespace，而不是control checkout/整仓working tree。

## 需求摘要

- CR-051 AW需求总数：27。
  - P0：25。
  - P1：2（migration handoff与幂等增强）。
  - P2：0。
- BLOCKING信息缺口：0。
- 待用户策略决策：3。
- `ready_for_design`：`false`（仅因CP2人工门尚未批准，不是自动预检失败）。

## SCENARIOS覆盖摘要

CR-051新增15个场景，全部`HIGH`，全部回链至少1个UC、REQ和候选Story：

| 类型 | 数量 | 代表场景 |
|---|---:|---|
| positive | 6 | project-first解析、legacy兼容、persistent worktree、sibling dirty隔离、显式refresh、双项目并发 |
| negative | 4 | 歧义写目标、current dirty、refresh冲突、identity mismatch |
| boundary | 2 | 重复CR ID的project branch命名、全操作dry-run |
| permission | 1 | unsafe worktree remove |
| failure-recovery | 1 | stale registry诊断与非破坏恢复 |
| precheck | 1 | migration manifest零真实迁移 |

真实remote验证不是`N/A`，而是明确`deferred-authorization`；自动验证只使用临时repo与本地bare remote。

## TEST-MATRIX缺口摘要

- 未覆盖场景：0。
- N/A：0。
- WAIVED：0。
- planned：15/15（CP2/CP3/CP5前不得实现）。
- 既有CR-046/047/050验证事实保持原结论，不被CR-051 planned状态覆盖。

## STORY-MAP摘要

- 用户活动/候选Epic：4。
- 产品候选Story：5（`ST-AW-001..005`）。
- 产品切片：4（`SL-AW-01..04`）。
- 执行顺序：project-first routing → per-project worktree → project-scoped lifecycle/shared-main refresh → migration handoff。
- 这些是CP2产品候选，不是CP4正式Story卡片，不授权提前拆解或实现。

## MVP范围

### In Scope

- project-first路由、layout version、legacy dual-read与portable metadata。
- 长期每项目worktree的create/check/list/safe-remove、registry/health和project-namespaced branch。
- source + current project artifact worktree的项目作用域Git lifecycle、dirty/ownership隔离。
- shared main前进后的显式refresh与冲突fail-closed。
- local bare-remote双项目fixture、migration preflight/manifest与迁移手册。

### Out of Scope / Not Authorized

- 真实artifact文件搬迁/删除。
- 任何现有软链接创建、替换、修复或删除。
- 真实artifact worktree/branch/ref/commit/tag/stash mutation。
- 真实remote fetch/push/default update/delete/force、forge API或credential读取。
- control repo立即转bare、自动rebase/force-with-lease、自动解冲突。

### Deferred

- 用户逐项目真实迁移与软链接挂接。
- 真实shared remote双项目pilot。
- bare control转换与线性历史refresh策略。

## 发布切片与Backlog

| Slice | 用户价值 | 前置 |
|---|---|---|
| SL-AW-01 | 唯一、兼容、portable的项目路由 | CP2/CP3/CP5 |
| SL-AW-02 | 稳定、隔离的project worktree | SL-AW-01 + DQ-01/03 |
| SL-AW-03 | 项目作用域Git周期与shared-main refresh | SL-AW-02 + DQ-02 |
| SL-AW-04 | migration handoff而不执行migration | SL-AW-03 |

后续候选在`process/docs/product/BACKLOG.md#cr-051-deferred--follow-up`；用户明确的逐项目迁移落入`BL-AW-001/002`，不得在CR-051中自动激活。

## 成功指标

- 路由：每个项目恰好解析1个docs和1个process；歧义写目标100%阻断。
- 隔离：branch/index/path crossover与index.lock争用均为0；sibling dirty误阻断为0。
- 当前项目保护：current dirty/identity/ownership异常的mutation前阻断率100%。
- refresh：before/after OID与恢复入口覆盖100%；自动rebase/force/解冲突为0。
- 兼容/dry-run：同一输入重复结果一致100%；所有dry-run副作用0。
- 本CR边界：真实文件搬迁、软链接变化、真实worktree/branch/ref mutation均为0。

## 待人工决策项

### CP2-DQ-01：每项目worktree生命周期

- 决策类型：`architecture`。
- 问题：project worktree长期常驻、每CR临时创建，还是混合？这会影响后续软链接稳定性、磁盘/清理成本、branch占用和恢复方式。
- 推荐：长期常驻per-project worktree；active CR使用project-namespaced branch，idle态推荐候选为detached fresh remote default，具体由CP3冻结。
- 推荐优势：软链接目标稳定；不为每个CR重建worktree；多个项目可长期并行；恢复入口固定。
- 推荐代价：需要stale/prunable health与safe-remove；占用磁盘；idle态需明确。
- 备选A：每CR临时worktree。优势是CR结束即可清理、branch状态简单；代价是软链接目标不稳定、反复创建/登记、异常中断恢复复杂。
- 备选B：persistent默认 + 高风险/实验CR临时worktree。优势是兼顾稳定与隔离；代价是两套生命周期和更多状态分支。
- 风险/影响：workspace CLI、metadata、repair/remove、bootstrap、迁移手册和测试矩阵。
- 回退/切换：若persistent在真实pilot出现不可接受的stale、磁盘或branch占用问题，回到CP3选择hybrid/ephemeral；不需要改project-first产品方向。

### CP2-DQ-02：共享main前进后的refresh策略

- 决策类型：`architecture`。
- 问题：其他项目先合入artifact main后，当前project CR branch如何恢复包含fresh main的可合并状态？
- 推荐：独立显式`merge fresh origin/main into current project CR branch`；冲突fail-closed；不自动force/rebase/解冲突。
- 推荐优势：不改写branch历史；无需force push；保留两个项目合入的因果关系；失败可观察/abort/resume。
- 推荐代价：project branch可能出现merge commit；仍需冲突处理的人工作用域与证据。
- 备选A：rebase + force-with-lease。优势是线性历史；代价是历史改写、force授权、stale lease和恢复复杂度明显增大。
- 备选B：从fresh main重建branch并受控cherry-pick项目commits。优势是不保留merge commit；代价是commit选择/映射/重复提交风险与实现复杂度最高。
- 风险/影响：CR-050 merge/finish适用性、publish freshness、OID证据、恢复路由和branch protection。
- 回退/切换：若真实pilot证明merge commit历史成本不可接受，另开CR评估备选A/B；任何切换都不得静默发生。

### CP2-DQ-03：control repo与project worktree默认拓扑

- 决策类型：`architecture`。
- 问题：迁移前如何放置control repo和project worktrees，既兼容现有clone又避免嵌套worktree/路径污染？
- 推荐：保留现有`meta-flow-artifacts` clone作为control checkout；project worktree放在可配置的sibling worktree root（control checkout之外）；worktree通过project namespace/sparse policy只暴露当前项目路径；metadata只保存锚点+相对路径。
- 推荐优势：不要求先转换bare；兼容现有remote/config；支持逐项目迁移；回滚到legacy读取简单。
- 推荐代价：control checkout仍存在，需要防止误当写入面；sibling root需统一命名/发现；sparse policy需兼容验证。
- 备选A：bare control repo + sibling project worktrees。优势是control不再有working tree/branch占用；代价是现有clone转换/重建和迁移操作面更大。
- 备选B：sibling full worktrees，不启用project namespace/sparse限制。优势是Git行为最简单；代价是每个worktree都可见所有项目，跨项目误写/磁盘成本更高。
- 风险/影响：workspace link/check、worktree registry、owned path、portable metadata、clean clone和迁移手册。
- 回退/切换：若existing control checkout持续造成误操作或branch占用，CP3/后续迁移CR可切bare；若sparse兼容性不足，可降为full worktree但仍强制owned-path gate。

## N/A / WAIVED

- N/A：0。
- WAIVED：0。
- 缺失正式产物：0（八份产品基线、discussion log/checkpoint、CP1/CP2 result+summary均已生成）。
- 人工checkpoint文件：本Agent不生成/发起；由Host Orchestrator在聚合Decision Brief后创建，属于职责分工而非N/A/WAIVED。

## CP1 / CP2证据

- CP1 result：`process/checks/CP1-CR051-USE-CASE-COMPLETENESS.result.json`（PASS）。
- CP1 summary：`process/checks/CP1-CR051-USE-CASE-COMPLETENESS.summary.md`。
- CP2 result：`process/checks/CP2-CR051-REQUIREMENTS-BASELINE.result.json`（PASS；仅自动预检）。
- CP2 summary：`process/checks/CP2-CR051-REQUIREMENTS-BASELINE.summary.md`。
- Context：`process/context/CP2-CR051-REQUIREMENT-CONTEXT.yaml`。

## 给Host Orchestrator的下一步

1. 扫描本return summary、CP1/CP2 result、discussion checkpoint和当前用户显式选择，聚合Decision Collection Coverage。
2. 将CP2-DQ-01..03写入`process/STATE.md.human_gate_decisions.pending_human_decisions[]`的受控状态入口，并生成CP2人工checkpoint/Decision Brief。
3. 发起CP2时明确：`approve`接受三项推荐；不授权真实迁移、软链接、Git/worktree/ref/remote mutation、credentials或runtime。
4. 用户未批准前停止；不得调度meta-se、不得进入CP3。

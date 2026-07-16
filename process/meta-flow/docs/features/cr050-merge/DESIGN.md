---
status: draft
version: "1.0"
feature_id: "FEAT-GB-04"
feature_name: "Paired Default Fast-forward"
source_blueprint: "process/docs/design/CR050-GIT-BRANCH-BLUEPRINT.md"
source_hld: "process/docs/design/CR050-GIT-BRANCH-HLD.md"
source_adr: "process/docs/design/CR050-GIT-BRANCH-ARCHITECTURE-DECISION.md"
source_matrix: "process/docs/design/CR050-FEATURE-DESIGN-MATRIX.md"
related_stories: ["ST-GB-004"]
lld_policy_summary: "1 full-lld"
---

# Feature Design: Paired Default Fast-forward

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-16 | host-orchestrator-inline/meta-se | 冻结artifact-first治理预写、typed authz、ordinary exact OID push、PARTIAL与2/2 projection firewall。 |

## 摘要

| 项目 | 内容 |
|---|---|
| Feature目标 | 显式将2/2 published CR tip以普通fast-forward更新到各自remote default。 |
| 推荐方案 | preflight-all→artifact exact push/post-check→project exact push/post-check→2/2 projection gate。 |
| 关键取舍 | artifact-first先固化治理基线；不自动补偿；单仓成功不推进workflow truth。 |
| 下游Story | ST-GB-004 |
| LLD策略 | full-lld：default写、authz、非原子恢复、跨源投影。 |

## 背景、目标与非目标

来源UC-GB-004、REQ-GB-011..014/C002..004/NF003、TC-GB-012..017。Goal是2/2 eligible+authz后以exact OID完成FF，PARTIAL可恢复且不可假PASS。Non-Goal是merge commit/rebase/squash/force/auto conflict、PR/queue API、保护策略绕过、自动rollback。

## 边界与模块

| 模块 | 负责 | 不负责 | 失败路径 |
|---|---|---|---|
| Merge planner/policy | fresh refs、matching publish、ancestry、authz、顺序plan | 远端写 | 任一失败BLOCKED且writes=0 |
| Git adapter | argv-only普通exact push与post-query | workflow投影 | remote拒绝/timeout返回typed outcome |
| Attempt writer | 逐仓事实、before/after、resume | CR close | schema不完整invalid |
| Paired Projection Gate | 2/2 aggregate后开放existing writer | 保存Git truth | PARTIAL时三项projection布尔均false |
| CR/state/current writer | 消费gate decision投影workflow状态 | 从单仓ref推断成功 | writer错误由fixture阻断 |

## 数据与接口

- `DefaultWriteAuthorization`绑定authorization ID、operation=merge、repo set、remote/default/CR ref、expected default/published OID、issuer、有效期/单次语义。
- `MergeEligibility`每仓含matching publish、fresh CR/default OID、ancestor、identity、authz match。
- `RepoMergeOutcome`含terminal、before/after、executed/skipped、mutation、remote rejection、resume。
- `PairedMergeProjection`含2个terminal与`paired_complete/paired_projection_advanced/finish_allowed/cr_close_allowed`。
- `plan_merge(intent,evidence,snapshots,authz)->Plan`无写；`execute_merge(plan)->Attempt`不写state；`project_merge(attempt)->ProjectionDecision`不执行Git。

## 核心流程与exact command

1. 读取matching publish result但fresh query remote CR/default；校验2/2 identity/OID/ancestry/authz。
2. 任一全仓preflight失败，default mutation=0。
3. dry-run只输出digest/steps；actual固定artifact→project。
4. 单仓写argv为`["git","push","--porcelain",remote,f"{published_oid}:refs/heads/{default_branch}"]`的等价封装；refspec不含`+`，argv不含`--force`/`--force-with-lease`；不checkout、不执行`git merge`。
5. 每仓立即`ls-remote`/等价query验证remote default exact expected OID；写后无法判定则PARTIAL。
6. 只有2/2 PASS/NO_CHANGE才投影overall PASS和finish eligibility；artifact PASS/project失败只追加PARTIAL attempt，CR active、projection/finish/close=false、2/2 branches retained。
7. resume fresh observe；artifact仍exact才NO_CHANGE，只继续project；事实漂移则BLOCKED。

## artifact-first安全论证与切换

artifact-first先让低风险过程仓承载已经批准的治理/证据基线，再写高风险源码default，避免payload领先治理。它不是优先级声明，且必须通过模块边界证明attempt writer与workflow projection writer分离。若unit/fixture无法证明`PARTIAL => paired_projection_advanced=false && cr_close_allowed=false && finish_allowed=false`，本Feature立即NEEDS_DESIGN_CLARIFICATION，切project-first并重开CP2/CP3。

## 权限、安全、测试

无matching typed authz时actual subprocess=0；CP gate approval不是authz。非法remote/ref/OID拒绝；禁止shell和所有force形态。测试包括17场景映射中的positive/non-FF/authz/policy reject/dry-run/partial/resume/finish-block；command spy断言merge/rebase/force=0；projection fixture模拟artifact PASS/project reject并断言state/current/CR未推进。

## 实现顺序、下游与风险

先freeze authz/eligibility/outcome/projection types，再planner，再Git argv/post-check，再paired executor/projection writer接线，最后fault/command/projection fixtures。ST-GB-003只消费current 2/2 projection并fresh reproof。真实branch protection行为和default写未授权；CP7最多证明bare/native contract并保持风险上限。

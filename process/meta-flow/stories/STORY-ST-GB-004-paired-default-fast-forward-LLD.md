---
story_id: "ST-GB-004"
title: "以独立授权快进两仓默认分支并守住2/2投影门"
story_slug: "paired-default-fast-forward"
lld_version: "1.1"
tier: "L"
status: "approved"
confirmed: true
created_by: "host-orchestrator-inline/meta-dev"
created_at: "2026-07-16"
confirmed_by: "user"
confirmed_at: "2026-07-16T15:56:00Z"
shared_fragments: []
feature_design_refs: ["process/docs/features/cr050-merge/DESIGN.md", "process/docs/features/cr050-merge/TEST-PLAN.md", "process/docs/features/cr050-merge/TASKS.md"]
lld_policy: {required_level: "full-lld", trigger_reasons: ["default-write", "authorization", "cross-repo-partial", "workflow-projection"], rationale: "default写和paired truth投影必须fail closed."}
open_items: 0
---

# LLD: ST-GB-004 — Paired Default Fast-forward

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-16 | host-orchestrator-inline/meta-dev | 冻结typed authz、ordinary exact OID push、artifact-first PARTIAL/resume和2/2 projection firewall。 |
| 1.1 | 2026-07-16 | host-orchestrator-inline/meta-dev | CP5独立评审精确化§4/6/13：固定CLI为`meta-flow cr branch-merge`，与open/publish/finish形成统一生命周期命令族。 |

## 0. 工程依据与上游设计依据

工程依据为CP3 R3 HLD v1.2、ADR-R2-002/003、Domain INV-GB-05..09/13、Dependency FD-GB-07/11、Feature Matrix、FEAT-GB-04 pack和ST-GB-002 `PublishEvidence` contract。R3的核心条件是单仓artifact成功绝不推进paired workflow truth。

## 1. 目标

创建显式merge handler：在2/2 fresh eligibility和typed authz通过后，按artifact→project以普通exact OID ref push更新remote default；2/2 post-check后才投影PASS，任何PARTIAL保持CR active、finish/close false。

## 2. 需求（Functional / Non-Functional）

- matching publish+fresh CR/default OID+ancestor+identity+authz必须2/2 preflight PASS。
- exact argv不得本地checkout/merge，不含`+`、force、force-with-lease、rebase或auto conflict。
- artifact→project顺序100%；每仓即时post-check；write后未知=PARTIAL。
- Projection Gate只接受2/2 PASS/NO_CHANGE且after=expected tip。
- dry-run mutation=0；相同snapshot/authz输出plan digest一致。

## 3. 模块拆分与职责

| 模块 | 职责 | 禁止 |
|---|---|---|
| merge planner/policy | validate evidence/refs/ancestry/authz并生成有序plan | 执行Git/写state |
| Git adapter | exact ref push/query/redaction | 推导overall/调用force |
| paired executor | artifact→project、per-repo outcome、resume | rollback成功仓 |
| attempt writer | append-only逐仓事实 | 写paired PASS |
| projection gate | 2/2聚合decision | 读取artifact单仓即关闭CR |
| existing writers | 按decision更新CR/state/current | 反向执行Git |

## 4. 代码结构与文件影响范围

| 动作 | 文件 | 内容 |
|---|---|---|
| 修改 | `meta_flow/workflow/git_branch_lifecycle.py` | authz/eligibility/outcome/projection types、planner/executor/resume |
| 修改 | `meta_flow/workspace/git_sync.py` | ordinary exact OID ref update、fresh remote query、typed rejection |
| 修改 | `meta_flow/workflow/cr_lifecycle.py` | 只消费validated projection decision |
| 修改 | `meta_flow/state/current.py` | paired projection字段/刷新约束（若现有schema需要，禁止另建truth） |
| 修改 | `meta_flow/cli.py` | `meta-flow cr branch-merge`、authz ref、dry-run/output |
| 修改 | `tests/test_git_branch_lifecycle.py` | bare/fault/spy/projection fixtures |

## 5. 数据模型与持久化设计

`DefaultWriteAuthorization`绑定operation=merge、repo set、remote/default/CR ref、expected before/default和published OIDs、issuer、有效期、consumption semantics。`MergeEligibility`为每仓fresh snapshot+matching evidence+ancestor/authz result。`RepoMergeOutcome`保留before/after/terminal/mutation/rejection/resume。`PairedMergeProjection`字段：attempt_ref、repo terminals、paired_complete、paired_projection_advanced、finish_allowed、cr_close_allowed。只有2/2 exact PASS/NO_CHANGE后三个布尔值才true；对象是派生decision，不是新数据库。

## 6. API / Interface设计

| 接口 | 输入 | 输出 | 调用方 | 错误 |
|---|---|---|---|---|
| `validate_default_write_authz(authz,intent,snapshots)` | typed authz/fresh OID | allow/deny[] | planner | missing/mismatch/expired |
| `plan_merge(intent,publish,snapshots,authz)` | 2/2 facts | immutable ordered Plan | CLI | non_ff/ref_drift/policy_precheck |
| `push_exact_remote_ref(repo,remote,oid,default)` | validated full OID/ref | typed command result | executor | remote_policy_rejected/timeout |
| `execute_merge(plan)` | Plan | append-only Attempt | Host | PARTIAL/post_verify_failed |
| `project_paired_merge(attempt)` | validated 2/2 outcomes | ProjectionDecision | existing writer adapter | invalid/incomplete |
| `meta-flow cr branch-merge` | publish evidence+typed authz | attempt+paired projection | operator/Host | 非2/2 exact时不得推进workflow truth |

## 7. 核心处理流程

1. 读取publish result只取identity；fresh query 2/2 remote CR/default。
2. 验证full OID、default是published祖先/相等、authz逐仓匹配；任一失败writes=0。
3. dry-run输出artifact→project plan，无runner调用。
4. actual对artifact执行普通exact OID push并post-query；失败则停止、overall BLOCKED/FAILED且project未写。
5. artifact成功后对project同样执行；拒绝/漂移/未知则PARTIAL。
6. attempt writer先落逐仓事实；Projection Gate聚合：2/2 exact才PASS，否则projection/finish/close=false。
7. resume是新attempt，fresh observe artifact exact才NO_CHANGE，只继续project；事实漂移BLOCKED。

```mermaid
sequenceDiagram
  participant P as Planner
  participant A as Artifact Remote
  participant S as Project Remote
  participant G as Projection Gate
  P->>A: exact ordinary push + post-check
  A-->>P: PASS or failure
  P->>S: only after artifact PASS
  S-->>P: PASS or failure
  P->>G: validated 2/2 outcomes
  G-->>P: PASS only for 2/2; otherwise PARTIAL projection=false
```

## 8. 技术细节

实际命令argv冻结为`["git","push","--porcelain",remote,f"{published_oid}:refs/heads/{default_branch}"]`的等价封装。禁止refspec前缀`+`以及`--force`、`--force-with-lease`。不checkout default、不执行`git merge --ff-only`，避免污染worktree或生成commit。preflight的expected-before不是force lease；并发若使push non-FF由remote拒绝，若remote推进仍是published tip祖先则普通push可成功，post-check记录实际before/after。

## 9. 安全与性能设计

| 维度 | 措施 | 验证 |
|---|---|---|
| 授权 | CP gate不等于authz；逐repo/ref/OID/operation匹配 | authz matrix |
| 命令 | argv-only、ref validation、no-force/no-local-merge | command spy |
| Truth | attempt与projection writer隔离；PARTIAL三布尔false | projection fixture |
| 可靠性 | preflight-all、post-check、fresh resume、no rollback | fault/race fixtures |
| 性能 | 每仓固定probe/push/post-query；output bounded | call count |

## 10. 测试设计

| 场景 | 前置/操作 | 预期 | 方式 |
|---|---|---|---|
| 2/2 eligible/authz | merge actual on bare | sequence artifact→project；defaults exact；projection PASS | integration |
| project preflight non-FF | plan | 2/2 writes=0 | negative |
| missing/mismatched authz | actual | subprocess=0 | unit |
| artifact pass/project hook reject | fault | PARTIAL；CR active；projection/finish/close false；branches 2/2 | integration |
| resume | artifact already exact | artifact NO_CHANGE，只写project | integration |
| remote drift/timeout after write | execute | fresh query；unknown不报PASS | fault |
| forbidden argv | all paths | merge/rebase/force/`+`=0 | spy |
| dry-run repeat | identical facts | plans equal；mutation=0 | golden |

## 11. 实施步骤

| TASK-ID | 动作 | 文件 | 描述 | 测试 |
|---|---|---|---|---|
| TASK-GB-004-01 | 修改 | lifecycle | authz/eligibility/outcome/projection types | unit |
| TASK-GB-004-02 | 修改 | git_sync/lifecycle | exact push/post-query/redaction | bare+spy |
| TASK-GB-004-03 | 修改 | lifecycle/cli | paired executor/PARTIAL/resume | fault |
| TASK-GB-004-04 | 修改 | lifecycle/cr_lifecycle/current/tests | 2/2 gate与不推进fixture | projection |

## 12. 风险、难点与预研建议

| Clarification ID | 问题 | 决策 | 影响 | 重访条件 |
|---|---|---|---|---|
| none | artifact-first安全前提是否可实现 | 设计为独立projection gate，CP5无blocking问题 | shared writer/tests | fixture无法隔离单仓attempt与paired projection |

最大风险是truth-ahead误投影；以INV-GB-13和fixture阻断。若无法保证，立即NEEDS_DESIGN_CLARIFICATION，切project-first并重开CP2/CP3。真实branch protection/receipt未验证，保持OPEN而非伪造PASS。

## 13. 回滚与发布策略

实现先隐藏在`meta-flow cr branch-merge`显式命令后；旧publish/finish无隐式merge。若projection gate回归失败，禁用merge CLI并保留attempt schema/tests，绝不自动reset已成功default。真实default-write dogfood需单独typed authz和用户明确批准。

## 14. DoD

- [ ] 工程依据/目标/需求/模块拆分/代码结构/数据模型/API/流程/技术细节/安全/测试/实施/风险/DoD完整。
- [ ] TC-GB-012..017为6/6，SC-GB-03..05/07满足。
- [ ] exact argv无local merge/`+`/force；禁止命令=0。
- [ ] PARTIAL fixture断言projection/finish/close=false、CR active、branches retained。
- [ ] 实现若违反切换条件不得进入CP6 PASS。
- [x] CP5批准后confirmed=true；仍须等待ST-GB-002通过依赖门。

## 人工确认区

- 结论：approved
- 审查人：user
- 风险接受：非原子PARTIAL、真实保护平台/receipt/独立QA未验证。

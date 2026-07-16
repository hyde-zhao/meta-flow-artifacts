---
status: archived-changes-requested
version: "1.0.1"
created_at: "2026-07-16"
owner: "host-orchestrator-inline / meta-se"
cr_ref: "CR-050"
review_gate: "CP3"
superseded_by_scope_change: "CP2 R2 explicit merge baseline; content revision deferred until CP2 R2 approval"
---

# CR-050 Governed Git Branch Lifecycle Domain Map

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-16 | host-orchestrator-inline / meta-se | 定义 branch intent、repo target、ref snapshot、plan、attempt、proof、recovery ref 与 protected policy 的归属、状态和不变量。 |
| 1.0.1 | 2026-07-16 | host-orchestrator | 仅更新审查状态：现有 operation 枚举缺少独立 merge；保留 v1.0 正文，待 CP2 R2批准后修订领域对象和不变量。 |

## 1. 领域边界

本领域只决定“某个已识别 CR 的 Git branch 操作是否允许、按什么顺序执行、观察到什么事实”。Git refs 仍由 Git repository/remote 持有；CR 生命周期仍由现有 CR/state writer 持有；本领域不建立分支状态数据库。

## 2. 统一术语

| 术语 | 定义 | 非定义 |
|---|---|---|
| CR branch | 由 CR ID + slug 经确定性策略生成并通过 `check-ref-format --branch` 的非 protected branch | 任意当前分支或用户输入 refspec |
| remote default | `ls-remote --symref <remote> HEAD` / remote symbolic HEAD 解析出的 branch，或经显式 override 验证的 branch | 静默猜测 `main`/`master` |
| exact tip | 一次观察或既有 result 中的完整 commit OID | branch 名、短 hash 或 patch-id |
| publish | 把调用前已经存在的 local commit ref 推送到同名 remote branch | stage、commit、amend、merge |
| merge proof | `git merge-base --is-ancestor <known-cr-tip> <fresh-remote-default-tip>` 成功且 identity/tip/policy 均匹配 | 内容相似、PR 页面文字或 remote branch 已消失 |
| recovery ref | 删除前建立的 local-only `refs/meta-flow/recovery/...`，精确指向 CR tip | merge receipt、远端备份或自动发布 tag |
| PARTIAL | 至少一个仓已发生允许的 mutation，整体目标未全部完成 | PASS、原子回滚完成 |

## 3. 领域对象

### 3.1 `BranchLifecycleIntent`

| 字段 | 约束 |
|---|---|
| `operation` | `start/open | publish | finish` |
| `cr_id` | canonical `CR-\d+` |
| `slug` | 可选；归一化后仍须通过 branch policy |
| `remote` | 单一 remote 名；作为 argv 参数，禁止选项前缀/控制字符 |
| `default_override` | 可选；必须解析到现存 remote branch 并加入 protected set |
| `expected_branch` / `expected_tip` | publish/finish 用于 identity 与 drift 检测 |
| `dry_run` | true 时 mutation command 执行数必须为 0 |
| `authorization_ref` | 真实 remote write/delete 时指向本次显式授权；dry-run 可为空 |

### 3.2 `RepositoryTarget`

由 workspace route 发现，固定 label=`project|artifact`、resolved root、repo fingerprint、remote。两个 label 若解析到同一 Git root，只保留一个执行目标并产生 blocking ambiguity，不能重复写同一仓。

### 3.3 `RefSnapshot`

一次只读观察：current branch、HEAD OID、dirty/index/worktree、local default/CR OID、remote symbolic HEAD、remote default/CR OID、upstream、ahead/behind、protected classification、observed_at。snapshot 不授权 mutation，也不等于持久 truth。

### 3.4 `BranchOperationPlan`

由相同 intent + snapshots + policy 产生相同有序 steps。每个 step 包含 repo、phase=`probe|local_mutation|remote_mutation|verify`、safe argv summary、precondition、expected before/after OID、skip/idempotency rule、failure route。不得包含 shell string。

### 3.5 `BranchOperationAttempt`

versioned append-only result；包含 attempt ID、operation、intent digest、plan digest、repo outcomes、overall decision、timestamps、authorization ref、source result refs、supersession/ref。它是审计证据，不覆盖 Git refs 的事实地位。

### 3.6 `RepoOutcome`

| 状态 | 含义 |
|---|---|
| `PLANNED` | dry-run 已产生完整 plan，未执行 mutation |
| `NO_CHANGE` | 目标事实已满足且 identity/OID 仍匹配 |
| `PASS` | 该仓全部必需 mutation 与 post-check 完成 |
| `BLOCKED` | precondition/proof/authorization 不满足，未执行该仓 destructive step |
| `PARTIAL` | 该仓已有 mutation，但后续 step 失败 |
| `FAILED` | Git invocation/IO 异常；必须记录是否存在已执行 mutation |
| `SKIPPED` | 因前仓失败或 global stop rule 未进入；必须有原因 |

overall 仅在所有 required repo 为 PASS/NO_CHANGE 时可为 PASS；任何 PARTIAL/FAILED 都不能降级为 warning。

### 3.7 `ProtectedRefPolicy`

包含 resolved remote default branch、默认 `main/master` 防误删集合、项目配置的 perennial/protected refs，以及“branch 必须与 CR naming contract 精确匹配”的规则。任何目标命中 protected set 或 identity ambiguous，finish BLOCKED。

### 3.8 `MergeProof`

包含 proof source、known CR tip、fresh remote default tip、remote target tip/absence、recorded/local source、tip drift decision、ancestry command/result。MVP 唯一 positive proof 是 Git DAG ancestor；squash/rebase 返回 `UNPROVABLE`，不是 FAIL 可绕过项。

### 3.9 `RecoveryRef`

local-only `refs/meta-flow/recovery/<normalized-cr-id>/<branch-fingerprint>`。创建规则：不存在则以 known exact tip 原子创建；存在且同 OID则 NO_CHANGE；存在但不同 OID 则 BLOCKED。finish 不自动删除或 push recovery ref。

## 4. 聚合与所有权

```text
BranchLifecycleAttempt (aggregate root)
├── BranchLifecycleIntent
├── BranchOperationPlan
├── RepositoryTarget[1..2]
│   ├── RefSnapshot before/after
│   ├── RepoOutcome
│   └── RecoveryRef? / MergeProof?
└── Evidence correlation (result ref + existing ledgers)
```

Planner 只能创建 plan；Executor 只能执行 plan 中允许的 argv；Coordinator 只能按 operation state machine 调用 planner/executor/现有 CR writer；ledger adapter 只能追加关联。

## 5. 状态机

### 5.1 attempt 状态

```text
RECEIVED
  -> OBSERVED
  -> PRECHECKED
  -> PLANNED -------------------------> DRY_RUN_COMPLETE
  -> EXECUTING -> VERIFYING -> PASS
                    |           |
                    +-> PARTIAL +-> FAILED
  -> BLOCKED
```

`BLOCKED` 可在任何 mutation 前产生；一旦任一 mutation 完成而目标未闭合，overall 至少为 PARTIAL，不能写成 BLOCKED-with-zero-side-effect。

### 5.2 lifecycle 关系

```text
start PASS -> branch opened
publish PASS/NO_CHANGE -> published_tip known
external merge (outside system)
finish proof PASS -> recovery anchored -> remote cleanup -> local cleanup
```

finish 不根据 CR lifecycle status 推断 merge；CR close 也不能反向证明 Git branch 已合并。

## 6. 核心不变量

| ID | 不变量 | 失败结果 |
|---|---|---|
| INV-GB-01 | branch name 通过 `check-ref-format --branch`，且不是选项/受保护 ref | BLOCKED，mutation=0 |
| INV-GB-02 | start 全仓初始 worktree/index clean、HEAD attached、route/remote/default 可解析 | BLOCKED，branch mutation=0 |
| INV-GB-03 | start base OID 必须等于 actual fetch 后 remote default exact tip | BLOCKED/PARTIAL，禁止 force 修复 |
| INV-GB-04 | publish 只执行 ordinary push；remote pre-tip 必须是 local HEAD 祖先或相等 | BLOCKED，remote unchanged |
| INV-GB-05 | publish 后 remote CR OID 必须等于调用时 local HEAD | PARTIAL/FAILED |
| INV-GB-06 | finish 的 known tip 必须来自 matching result、local branch 或 remote branch，且来源不冲突 | UNPROVABLE/BLOCKED |
| INV-GB-07 | remote delete 前 known tip 是 fresh remote default tip 祖先，且 remote target 不漂移 | BLOCKED，delete count=0 |
| INV-GB-08 | recovery ref 建立后才可 remote delete；两仓 remote phase完成后才开始 local `branch -d` | PARTIAL，保留 local refs/anchor |
| INV-GB-09 | dry-run mutation command count=0；输出 planned command 不等于 executed | FAILED/BLOCKED |
| INV-GB-10 | 每仓 terminal status、before/after OID、executed/skipped steps 与 recovery route 100% 完整 | result invalid，不能推进 |

## 7. 幂等与恢复规则

| Operation | 已满足事实 | 幂等结论 | 冲突事实 |
|---|---|---|---|
| start | local/remote same branch、base/upstream 均匹配 | NO_CHANGE 或 resume remaining repo | 同名不同 OID/upstream → BLOCKED |
| publish | remote CR OID == local HEAD | NO_CHANGE | remote ahead/diverged/wrong upstream → BLOCKED |
| finish | remote absent、known tip 可证明、local branch/recovery ref仍可关联 | 继续 local cleanup或 NO_CHANGE | remote absent 且 tip 不可恢复 → UNPROVABLE/BLOCKED |

恢复只执行尚未完成且 precondition 仍成立的 step；不得用 delete/recreate/force 让结果看似一致。重试必须生成新 attempt，并引用前一 result。

## 8. 失败分类

固定 error code 至少包含：`route_invalid`、`not_git_repo`、`dirty_tree`、`detached_head`、`default_unknown`、`invalid_branch`、`branch_collision`、`diverged_default`、`wrong_branch`、`wrong_upstream`、`non_fast_forward`、`ref_drift`、`protected_ref`、`ancestry_unproven`、`authorization_missing`、`remote_unavailable`、`partial_success`、`post_verify_failed`。

每个错误必须包含 repo label、step、safe command summary、相关 OID、是否已有 mutation、恢复路由；不得回显含凭据 remote URL。

## 9. Gotchas

1. remote symbolic HEAD 与本地 `origin/HEAD` 可能漂移；actual operation 必须刷新后验证，dry-run 则使用 read-only remote observation并披露 freshness 限制。
2. `branch -d` 只能删除未 checkout 的 branch；finish 必须先验证/快进默认分支并切换，但远端删除前仍要保留 local CR branch 与 recovery ref。
3. remote 自动删 branch 时不能把 absence 当 proof；known tip 为空即 BLOCKED。
4. result 中记录 OID 不能单独阻止 Git object GC；需要真实可恢复性时依赖 local recovery ref，而不是只存字符串。
5. 跨仓执行顺序是确定性的审计策略，不是原子性保证。

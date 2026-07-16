---
status: confirmed
version: "1.2"
created_at: "2026-07-16"
owner: "host-orchestrator-inline / meta-se"
cr_ref: "CR-050"
review_gate: "CP3-R3"
supersedes: "process/archive/CR-050/design/CR050-GIT-BRANCH-DOMAIN-MAP.v1.0.1.md"
---

# CR-050 Governed Git Branch Lifecycle Domain Map

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-16 | host-orchestrator-inline / meta-se | 定义 open/publish/finish intent、attempt、proof与recovery。 |
| 1.0.1 | 2026-07-16 | host-orchestrator | 因缺显式merge标记changes-requested并归档。 |
| 1.1 | 2026-07-16 | host-orchestrator-inline / meta-se | 新增MERGE intent、MergeEligibility/Authorization/RepoMergeOutcome、2/2 finish gate、artifact→project PARTIAL状态与恢复不变量。 |
| 1.2 | 2026-07-16 | host-orchestrator-inline / meta-se | CP3 R3新增PairedMergeProjection对象与write-ahead不变量；artifact单仓成功只写attempt，2/2聚合前禁止workflow PASS/finish/close。 |

## 1. 边界与术语

领域只决定某个CR的Git操作是否允许、顺序、实际观察和结果。Git refs仍由repository/remote持有；formal CR/state由既有writer持有；attempt只是append-only证据。

| 术语 | 定义 | 明确不是 |
|---|---|---|
| exact published tip | matching publish attempt与fresh remote CR ref共同确认的完整OID | branch名、短hash、local未发布HEAD |
| fast-forward eligible | fresh remote default tip是exact published tip祖先或相等 | “大概没有冲突”、内容相似、merge-base不明 |
| explicit merge | 独立`merge` intent，具有本次default-write authz与expected OID | publish/finish副作用、merge commit、PR API |
| PARTIAL | 至少一仓发生允许mutation，而整体目标未闭合 | PASS、自动回滚成功、warning |
| current merge PASS | 最新匹配attempt中2/2仓均PASS/NO_CHANGE，且post-check完整 | 历史PASS、单仓PASS、人工描述 |
| finish reproof | finish时重新fetch/observe并验证identity/tip/ancestry/protected | 复用merge结果跳过检查 |
| governance write-ahead | artifact先固化已批准治理基线，但单仓结果不改变paired workflow终态 | artifact比project重要、artifact ref等同overall PASS |

## 2. 领域对象

### 2.1 `BranchLifecycleIntent`

| 字段 | 约束 |
|---|---|
| `operation` | `open | publish | merge | finish` |
| `cr_id` / `branch` | canonical CR ID；branch通过确定性命名与`check-ref-format --branch` |
| `repository_targets` | route发现的`project`和可选/必需`artifact`，不得静默跳过已配置仓 |
| `expected_tip` | publish/merge/finish使用完整OID；与fresh ref冲突即BLOCKED |
| `authorization_ref` | actual remote mutation必填，且operation/repo/ref/OID必须匹配；dry-run可空 |
| `dry_run` | true时mutation command count=0 |

### 2.2 `RepositoryTarget` 与 `RefSnapshot`

`RepositoryTarget` 固定label、resolved root、fingerprint、remote；两个label解析到同一root时阻断歧义。`RefSnapshot`记录current/local/remote refs、default symref、HEAD、dirty、upstream、before OID、observed_at；它不授权写入。

### 2.3 `PublishEvidence`

匹配CR/repo/branch、result schema、attempt terminal、published exact OID和fresh remote CR ref。旧attempt可追溯但不能覆盖fresh drift；local HEAD变化不能自动提升为published tip。

### 2.4 `DefaultWriteAuthorization`

包含authorization ID、operation=`merge`、repo set、remote/default/branch、expected published/default OID、scope、issued_by与有效期/单次消费语义。CP gate approval不是该对象；对象缺失或不匹配时actual merge BLOCKED。

### 2.5 `BranchOperationPlan`

由intent+snapshots+policy产生确定有序steps；每步包含repo、phase=`probe|local_mutation|remote_mutation|verify`、argv summary、before/expected-after OID、precondition、idempotency和failure route。不得包含shell string或动态未校验refspec。

### 2.6 `BranchOperationAttempt` / `RepoOutcome`

attempt为versioned append-only证据，含intent/plan digest、repo outcomes、overall、authorization ref、timestamps、source/supersedes refs。RepoOutcome固定为`PLANNED|NO_CHANGE|PASS|BLOCKED|PARTIAL|FAILED|SKIPPED`；每个required repo必须有terminal、before/after OID、executed/skipped steps、mutation flag与resume route。

### 2.7 `MergeEligibility` 与 `RepoMergeOutcome`

Eligibility包含：matching publish、fresh remote CR/default OID、ancestor判定、branch/default identity、authz match、remote-policy pre-observation。只有所有仓全部eligible才可开始写。

执行顺序固定`artifact -> project`。每仓允许普通exact ref update后立即remote query验证；remote default必须等于expected published tip。artifact PASS/project失败时overall=PARTIAL，2/2 CR branch retain=true，finish_allowed=false，automatic_rollback_count=0。

### 2.8 `MergeProof` 与 `RecoveryRef`

finish的MergeProof记录known CR tip、fresh default tip、target ref/tip、source conflicts、protected policy与ancestry。MVP唯一positive proof是exact tip为fresh default祖先。RecoveryRef为local-only `refs/meta-flow/recovery/<cr>/<fingerprint>`；同OID幂等，不同OID阻断，不自动push/delete。

### 2.9 `PairedMergeProjection`

这是由validated attempt派生的轻量聚合决策，不是新truth source。字段至少含`attempt_ref`、2个`repo_terminal`、`paired_complete`、`paired_projection_advanced`、`finish_allowed`与`cr_close_allowed`。只有2/2 terminal均为`PASS|NO_CHANGE`且after OID等于expected tip时后三个布尔值才可为true；任何PARTIAL/BLOCKED/FAILED均全为false，并只允许existing writer保留active CR与resume ref。

## 3. 聚合与状态机

```text
BranchLifecycleAttempt
├── Intent + Authorization?
├── RepositoryTarget[1..2] + RefSnapshot before/after
├── Plan
├── RepoOutcome[1..2]
└── PublishEvidence? / MergeEligibility? / PairedMergeProjection? / MergeProof? / RecoveryRef?
```

```text
open PASS -> publish PASS -> merge ELIGIBLE
  -> preflight_all PASS
  -> artifact PASS
  -> project PASS -----------------> merge PASS -> finish may re-prove
                 \-> BLOCKED/FAILED -> merge PARTIAL -> paired projection false -> finish BLOCKED
```

重试总是新attempt：重新观察两仓；已成功仓只能在exact事实仍成立时NO_CHANGE，未完成仓重新preflight。不得从旧attempt直接跳过fresh observation。

## 4. 核心不变量

| ID | 不变量 | 失败结果 |
|---|---|---|
| INV-GB-01 | ref合法、非选项、非protected、identity唯一 | BLOCKED，mutation=0 |
| INV-GB-02 | open全仓clean/attached/route/default可解析 | BLOCKED，branch mutation=0 |
| INV-GB-03 | publish只推existing commit，remote最终等于local expected HEAD | BLOCKED/PARTIAL；隐式commit=0 |
| INV-GB-04 | merge是独立intent；publish/finish调用时default mutation=0 | result invalid/BLOCKED |
| INV-GB-05 | merge写前2/2仓fresh preflight与authz全部PASS | BLOCKED，2/2 default mutation=0 |
| INV-GB-06 | merge只允许`fresh_default <= exact_published_tip`，禁止merge commit/rebase/force/conflict resolution | BLOCKED，禁止命令数=0 |
| INV-GB-07 | merge顺序artifact→project且每仓即时post-check | result invalid，不能overall PASS |
| INV-GB-08 | artifact成功/project失败为PARTIAL；2/2 CR branches保留；auto rollback=0；finish=false | workflow BLOCKED |
| INV-GB-09 | overall merge PASS要求2/2 PASS/NO_CHANGE且after default=expected tip | 不能推进finish |
| INV-GB-10 | finish消费current matching merge PASS后仍重新证明ancestry/tip/protected | delete count=0 |
| INV-GB-11 | remote delete前recovery ref已建立；两仓remote完成前local delete=0 | PARTIAL并保留恢复面 |
| INV-GB-12 | dry-run local/remote mutation command count=0 | FAILED/BLOCKED |
| INV-GB-13 | artifact-first PARTIAL只追加attempt；`paired_projection_advanced=false`、`cr_close_allowed=false`、`finish_allowed=false` | result/projection invalid并阻断CP6/CP7 |

## 5. 失败分类与幂等

固定error code至少含：`route_invalid`、`dirty_tree`、`detached_head`、`default_unknown`、`invalid_branch`、`branch_collision`、`wrong_branch`、`wrong_upstream`、`publish_evidence_mismatch`、`authorization_missing`、`authorization_mismatch`、`non_fast_forward`、`ref_drift`、`protected_ref`、`remote_policy_rejected`、`ancestry_unproven`、`partial_success`、`post_verify_failed`。

| Operation | 已满足事实 | 幂等结论 | 冲突事实 |
|---|---|---|---|
| open | local/remote/upstream/base exact match | NO_CHANGE/resume另仓 | 同名不同OID→BLOCKED |
| publish | remote CR OID==expected local HEAD | NO_CHANGE | remote ahead/diverged→BLOCKED |
| merge | remote default==published tip且authz/identity匹配 | NO_CHANGE | default不是published tip祖先→BLOCKED |
| finish | remote CR absent且known tip可重新证明 | 继续local cleanup/NO_CHANGE | tip缺失/漂移/非祖先→BLOCKED |

## 6. Gotchas

1. `NO_CHANGE`仍是逐仓terminal，不是跳过identity/authz/post-check的捷径。
2. remote policy可能只在push时拒绝；preflight无法消除竞态，因此一仓成功后另一仓拒绝必须PARTIAL。
3. 保存OID不等于对象永久可达；finish删除前仍需local recovery ref。
4. remote CR branch存在与否不是default已合并证明；merge与finish分别验证不同权限和事实。
5. result append-only不等于另建branch数据库；恢复决策必须以fresh Git refs为主、历史result为关联证据。
6. `PairedMergeProjection`不复制Git ref truth，只判定existing writer能否推进；从artifact仓单独读取成功事实并关闭CR属于非法跨聚合推断。

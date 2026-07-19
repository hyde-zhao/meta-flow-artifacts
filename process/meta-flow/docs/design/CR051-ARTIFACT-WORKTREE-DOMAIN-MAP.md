---
status: confirmed
version: "1.1"
created_at: "2026-07-18"
owner: "meta-se"
cr_ref: "CR-051"
review_gate: "CP3"
confirmed: true
confirmed_by: "user-current-instruction"
confirmed_at: "2026-07-18T05:46:40Z"
---

# CR-051 Project-first Artifact Worktree Domain Map

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-18 | meta-se | 定义 project context、worktree、非原子 operation、异构 leg、aggregate、manual sync 和 migration manifest 的唯一领域语义。 |
| 1.1 | 2026-07-18 | meta-se-critical | 确认 DQ/ADR；把空间估算、durable intent store 与 manual-sync 运维指标固化为 CP5 可验证领域契约。 |

## 1. 术语表

| Term | 定义 | 来源 | 明确不是 |
|---|---|---|---|
| Project Artifact Context | 由 project identity、layout version、portable route、namespace、worktree 和 branch roles 组成的解析结果 | UC-AW-001；REQ-AW-001..003 | 设备绝对路径、自动写入授权 |
| Control Checkout | 现有 shared artifact clone 的控制工作树，只用于发现/管理 shared repository | CP2-DQ-03 | 当前 project 的默认写入面 |
| Project Worktree | configurable sibling root 下、注册到同一 common Git dir、只服务一个 project namespace 的长期 worktree | UC-AW-002 | nested control worktree、sibling project worktree |
| Project Integration | `projects/<project>/integration`，artifact idle branch 和 artifact CR completion target | REQ-AW-004/011 | shared main、source default、临时 CR branch |
| Project CR Branch | `projects/<project>/cr/<cr-id>-<slug>`，artifact leg 的短期 active branch | REQ-AW-007 | source CR branch、integration、main |
| Shared Main | shared artifact 跨项目集成基线，只能由 CR 外人工 sync operation 触碰 | CP2-DQ-06；REQ-AW-C004 | 单个 artifact CR 的 base/target |
| Fresh Observation | 当前 operation 内重新读取 symbolic HEAD、HEAD/ref OID、worktree registration、clean/Git-operation 状态和远端 exact ref | CP3-DC-01 | 旧 plan/result、命令退出码、进程内猜测 |
| Durable Intent | 在目标 worktree 外持久化且在 mutation 前可恢复的 operation/attempt/phase 记录 | CP3-DC-01 | Git 事务、自动恢复授权 |
| PARTIAL | progress/effect：已有 leg 或 mutation 成功，但逻辑 CR 尚未全 PASS | REQ-AW-012 | overall terminal success、自动 rollback 触发器 |
| Recovery Required | 现场无法安全证明为 original 或 target verified state，必须保留现场并人工处理 | CP3-DC-01 | 自动 reset/clean/stash/force/delete |

## 2. 领域对象与唯一 Owner

| Object ID | 对象 | Owner Feature | 关键属性 | 状态 / 规则来源 |
|---|---|---|---|---|
| OBJ-AW-01 | `ProjectArtifactConfig` | FEAT-AW-01 | project ID、layout version、anchor-relative control/sibling/worktree/namespace、sparse/owned paths、branch roles | REQ-AW-001..003、NF002 |
| OBJ-AW-02 | `RouteDecision` | FEAT-AW-01 | read targets、唯一 write target、resolved runtime paths、conflicts、decision | REQ-AW-002；TC-AW-001..003 |
| OBJ-AW-03 | `ProjectWorktreeRegistration` | FEAT-AW-02 | common Git dir identity、worktree path、project、current branch role、health、last observation | REQ-AW-004..007 |
| OBJ-AW-04 | `WorktreeOperationJournal` | FEAT-AW-02 | operation/attempt ID、project、worktree、original/target branch+OID、phase、store ID、record version、checksum、durability/readback 状态 | CP3-DC-01；单写、append-only phase history；O-AW-02 |
| OBJ-AW-05 | `FilesystemCapacityObservation` | FEAT-AW-02 | filesystem ID、estimator/profile version、enumeration coverage、estimated/upper-bound/required/available bytes、safety factor、false-safe/underestimate counters、observed time | O-AW-01；相同观测确定；估算/枚举失败 fail closed |
| OBJ-AW-06 | `LegAttempt` / `LegResult` | FEAT-AW-03 | CR/attempt correlation、repo role/mode、base/target、expected/current OIDs、terminal、steps、resume/abort | REQ-AW-008..013/016 |
| OBJ-AW-07 | `AggregateResult` | FEAT-AW-04 | required leg refs、validated correlation、overall、progress/effect、blockers、next route | REQ-AW-012/016 |
| OBJ-AW-08 | `IntegrationBootstrapAttempt` | FEAT-AW-02 | fresh main seed OID、integration before/after OID、create-only decision、remote race | REQ-AW-004；TC-AW-004 |
| OBJ-AW-09 | `ManualSyncAttempt` | CR-external sync coordinator | direction、project、main/integration before/expected/after OIDs、active-CR precheck、authorization、started/ended、duration、blocker category | CP2-DQ-06；REQ-AW-C004；O-AW-03 |
| OBJ-AW-10 | `MigrationManifest` | FEAT-AW-05 | mapping、hash/count、link plan、readiness、verification、rollback、authorization status | REQ-AW-014..015 |
| OBJ-AW-11 | `ObservationReceipt` | operation owner | observed refs/worktree/filesystem facts、source operation/attempt IDs、content digest | REQ-AW-016；不得自引用未来 commit OID |

Git refs 仍由 Git repository/remote 持有；formal CR/state/current 仍由既有 writer 持有。上述对象不得复制或替代这些 truth sources。

## 3. Worktree Operation 状态机（CP3-DC-01）

```text
PLANNED
  -> PRECHECKED
  -> INTENT_DURABLE
  -> SWITCH_ATTEMPTED
  -> OBSERVATION_REQUIRED
       -> VERIFIED_TARGET
       -> VERIFIED_ORIGINAL / NO_CHANGE
       -> ROLLBACK_ELIGIBLE
            -> ROLLBACK_INTENT_DURABLE
            -> ROLLBACK_ATTEMPTED
            -> OBSERVATION_REQUIRED
       -> RECOVERY_REQUIRED / BLOCKED
```

| State / Phase | 进入条件 | 允许动作 | 禁止/失败行为 |
|---|---|---|---|
| PLANNED | typed request 已校验 | 只读 precheck | 不调用 switch |
| PRECHECKED | identity/lock/clean/no-Git-op/ref/OID/permission/space 全 PASS | 持久化 intent | intent 写失败即 BLOCKED |
| INTENT_DURABLE | journal 已 durable，original/target 已记录 | 尝试一次 argv-only `git switch` | 不把退出码当 terminal truth |
| SWITCH_ATTEMPTED | 命令返回或超时/中断 | 立即 fresh observe | 不依据最后一步猜 current branch |
| OBSERVATION_REQUIRED | 任意 mutation/错误/恢复后 | 对 symbolic HEAD、HEAD OID、registration、clean、Git operation、refs 重新采样 | observation 缺失不得 PASS |
| VERIFIED_TARGET | 目标 branch/OID/registration/clean 全匹配 | 写 terminal PASS/active-cr 或 idle-integration | 任何字段不匹配不得降级为 warning |
| VERIFIED_ORIGINAL | original branch/OID/registration/clean 全匹配 | 写 NO_CHANGE/ROLLED_BACK | 不再次 switch |
| ROLLBACK_ELIGIBLE | 现场 clean、无 Git op、original integration ref/OID 无漂移、permission/space 复检 PASS | 先持久化 rollback intent，再尝试 switch original | 任一条件未知/失败不得自动 rollback |
| RECOVERY_REQUIRED | 状态为第三态、dirty、有 Git op、ref/OID 漂移、权限/空间不足、rollback 失败或 observation 不完整 | 保留 worktree/CR branch/journal，输出 observation-based manual entry | 禁止 reset --hard、clean、stash、force、branch delete、覆盖文件 |

`resume/recover` 每次都从 fresh observation 开始；重复调用只允许返回：`ALREADY_RECOVERED`、`RECOVERY_IN_PROGRESS`、`RECOVERY_REQUIRED/BLOCKED` 或新的 verified terminal。

### 3.1 Durable Intent Store 契约（O-AW-02）

store 必须位于目标 worktree 外。每次记录只允许在同一 store filesystem 内创建 store-local temp，完成 `write -> file fsync -> atomic replace -> parent-directory fsync -> checksum/readback` 后才能进入 `INTENT_DURABLE`；任一步失败均在 Git mutation 前返回 `BLOCKED`。跨设备路径不得依赖 rename，也不得把 copy+delete 冒充原子 replace；应选择同设备 store-local temp，无法满足则禁用 auto switch。

CP5 必须以 ENOSPC、EACCES、file fsync failure、replace failure、parent-dir fsync failure、torn/corrupt record、process kill、cross-device path fixture 证明：0 次提前 Git mutation、损坏记录不会被消费、resume 只依据 checksum/readback 通过的 durable record 与 fresh observation，并保持幂等。

### 3.2 Filesystem Capacity 契约（O-AW-01）

主路径必须输出可复算的 enumeration coverage、checkout write upper bound、profile version 和 `required_bytes=1.5*upper_bound`；相同 tree/index/sparse/profile/filesystem observation 必须得到相同结果。无法枚举、权限不足、profile 不匹配或误差界未知时一律 `BLOCKED`。

固定 512MiB 不是未知仓库的静默通行证。它只可作为经 bounded profile 校准的保守 floor/fallback：CP5 常见 checkout、误差边界、无权限/无法枚举 fixture 必须证明目标上界的 1.5 倍不超过 512MiB，且 false-safe=0、underestimate=0；任一不满足则禁用 auto switch。

## 4. Worktree Branch Role 状态机

| 当前状态 | 合法触发 | 目标状态 | 必要证明 |
|---|---|---|---|
| unregistered | create | idle-integration | project identity、non-nested/owned empty target、common git dir、integration create/check、post-observation |
| idle-integration | CR open | active-cr | integration exact expected OID、switch protocol VERIFIED_TARGET |
| active-cr | artifact finish | idle-integration | CR tip completion proof、integration expected OID 未漂移、switch protocol VERIFIED_TARGET |
| active-cr | abort | idle-integration 或 recovery-required | 不覆盖用户变更；只有 clean/no-Git-op/ref stable 才自动回归 |
| any registered | check/list | unchanged | 只读 observation；stale 只给诊断 |
| idle-integration | remove | removed | exact identity、clean、无 active CR、branch/ref/recovery proof、独立授权 |

`main` 不得成为 `idle-integration` 或 `active-cr`。

## 5. Integration Bootstrap 状态机

| Observation | Action | Result |
|---|---|---|
| remote integration exists | 不 create/reset/orphan；校验 identity/role | `NO_CHANGE` 或不匹配时 `BLOCKED` |
| integration absent + fresh origin/main exact OID available | 取得 project lock，以 ordinary exact create-only update 创建 | postcheck exact match→`PASS` |
| create 被并发拒绝，fresh integration == seed OID | 不重复创建 | `NO_CHANGE` |
| create 被并发拒绝，fresh integration != seed OID | 保留远端事实 | `BLOCKED`，重新规划/人工处理 |
| main OID/remote query 不可用 | 无 mutation | `BLOCKED` |

## 6. Leg 与 Aggregate 状态机

### Leg terminal

`PASS | FAIL | BLOCKED | IN_PROGRESS` 是聚合输入；`PARTIAL` 只放在 `progress/effect`，不是 overall terminal。

### Aggregate rule

```text
if any required leg == BLOCKED: overall=BLOCKED
else if any required leg == FAIL: overall=FAIL
else if any required leg == IN_PROGRESS or missing: overall=IN_PROGRESS
else if all required legs == PASS: overall=PASS
```

| 不变量 | 失败处理 |
|---|---|
| source/artifact results 必须属于相同 CR、logical attempt 和 required leg set | aggregate BLOCKED |
| artifact leg base/target 都是 project integration；source leg base/target 都是 source default | result invalid/BLOCKED |
| artifact main/control/sibling mutation count=0 | FAIL/BLOCKED；不得 overall PASS |
| 任一 leg 成功不自动回滚、不自动关闭失败 CR | 保留事实和 resume/abort route |
| aggregate writer 是唯一 overall writer | 多写/重复 ID 冲突即 BLOCKED |

## 7. Manual Sync 状态机（CR 外）

| Phase | Contract |
|---|---|
| PRECHECK | 明确 direction；项目无 active artifact CR；取得 project lock；fresh main/integration exact OIDs；typed authorization 匹配 target/expected OID |
| ELIGIBILITY | target 是 source 的祖先/相等才允许 ordinary fast-forward；否则 BLOCKED，不 rebase/force/解冲突 |
| EXECUTE | 单一 exact ordinary ref update；不由 CR lifecycle 隐式调用 |
| POSTCHECK | fresh target OID == expected source OID 才 PASS；超时/拒绝后重新观察 |
| RECOVERY | 保留 before/after OID 和 actual fact；不自动反向重写；重新计划或人工处理 |

每次 attempt 必须记录项目、方向、开始/结束时间、人工耗时、terminal、阻塞原因和是否属于可避免调度阻塞。滚动观测达到任一阈值：单项目每周同步 `>=3` 次且连续 `4` 周、人工同步中位耗时 `>10` 分钟、或可避免的调度阻塞率 `>5%`，只创建独立“条件式同步助手”CR 候选；CR-051 不因此自动同步。

## 8. Evidence DAG 与自引用规避

```text
Durable Intent
  -> Observation Receipt(s)
  -> Leg Result(s)
  -> Aggregate Result
  -> optional Publication Binding Event
```

- 每个对象只能引用更早对象的 stable ID/path/digest。
- `content_digest` 对 canonical payload 计算时排除 `content_digest` 自身。
- Receipt 可记录业务 Git ref 的 before/after OID；不得记录“未来承载该 receipt 的 artifact commit OID”。
- 后续 commit OID 只能写入更晚的 binding/ledger event，且不得回写旧 receipt/result。

## 9. 业务规则

| Rule ID | 规则 | Owner | 影响场景 | 验证入口 |
|---|---|---|---|---|
| RULE-AW-01 | route write target 必须唯一且由显式 layout version 决定 | FEAT-AW-01 | UC-AW-001 | TC-AW-001..003,010 |
| RULE-AW-02 | sibling dirty 不阻断；current dirty/identity/ownership 异常阻断 | FEAT-AW-02/03 | UC-AW-002/003 | TC-AW-005/006/010/014 |
| RULE-AW-03 | switch terminal 只由 fresh observation 判定 | FEAT-AW-02 | UC-AW-002 | CP3-DC-01、fault injection |
| RULE-AW-04 | integration 只 create-only；存在时不 recreate/reset/orphan | FEAT-AW-02 | UC-AW-002 | TC-AW-004 |
| RULE-AW-05 | shared-artifact route 禁止 artifact main per-CR plan | FEAT-AW-03 | UC-AW-003/004 | TC-AW-008/009/014 |
| RULE-AW-06 | 仅全部 required legs PASS 时 aggregate PASS | FEAT-AW-04 | UC-AW-004 | TC-AW-008/009/014 |
| RULE-AW-07 | migration manifest 永不隐式执行 migration/link/ref mutation | FEAT-AW-05 | UC-AW-005 | TC-AW-012/013/015 |

## 10. 决策状态与开放项

| 对象 | 状态 | 批准 / 路由 |
|---|---|---|
| CP3-CR051-DQ-01..03 | `approved` | 用户当前指令，`2026-07-18T05:46:40Z`；只确认架构，不授权运行 |
| ADR-AW-001..007 | `approved` | 与 DQ 推荐方案一致，具体落地仍受 CP5/运行授权门控制 |
| O-AW-01 | `non-blocking-open` | FEAT-AW-02 CP5 capacity fixture；不满足则禁用 auto switch |
| O-AW-02 | `non-blocking-open` | FEAT-AW-02 CP5 durable-store fault fixture；不满足则 Git mutation 前 BLOCKED |
| O-AW-03 | `non-blocking-open` | 运维指标阈值触发 follow-up CR candidate，不改变 CR-051 手工边界 |

## 11. Gotchas

1. `clean` 必须覆盖 staged、unstaged 和 untracked；只看工作树 diff 不够。
2. 没有进行中的 Git operation 必须覆盖 merge/rebase/cherry-pick/bisect/sequencer 等标志。
3. remote create 被拒绝后不能直接报 FAIL 或重试 force；先 fresh observe 才能区分并发同值 `NO_CHANGE` 与异值 `BLOCKED`。
4. Worktree journal 是恢复证据，不是 Git truth；任何 resume 必须重新观察。
5. `PARTIAL` 不得进入 aggregate terminal 枚举，否则会破坏 CP2 已冻结优先级。

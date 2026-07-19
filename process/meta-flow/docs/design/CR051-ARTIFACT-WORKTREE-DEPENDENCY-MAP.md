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

# CR-051 Project-first Artifact Worktree Dependency Map

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-18 | meta-se | 建立 routing、worktree、heterogeneous leg、aggregate、manual sync、migration 与既有 CR-050/native Git 边界的单向依赖。 |
| 1.1 | 2026-07-18 | meta-se-critical | 确认 DQ/ADR；补充 capacity profiler、durable store durability 与 manual-sync ops metrics 的依赖/禁止方向。 |

## 1. 依赖原则

```text
User / Host / CLI
  -> Project Artifact Capability Facade
      -> Project Context Resolver --------------------------+
      -> Worktree Lifecycle Coordinator -> Operation Lock  |
      -> Heterogeneous Leg Coordinator -------------------+ |
      -> Aggregate Gate (result-only)                     | |
      -> Migration Preflight (read-only)                  | |
                                                         v v
                  Existing typed policy / authz -> Native Git Probe/Executor
                                               -> filesystem probes

CR-external Manual Sync Coordinator -> Project Context + Lock + Authz + Native Git

Executors -> append-only evidence adapter -> existing ledgers
Validated aggregate -> existing CR/state/current writers
```

依赖只能自上而下；Git adapter、result、journal、route metadata 和 migration manifest 均不得反向驱动 workflow 或 mutation。

## 2. 允许依赖

| From | To | 依赖类型 | 允许方向 | 原因 | 验证 / 监控 |
|---|---|---|---|---|---|
| Host/CLI | Project Artifact Facade | runtime | allowed | 显式 route/worktree/leg/aggregate/migration 操作入口 | command/contract fixture |
| Facade | Project Context Resolver | read | allowed | 所有下游先得到唯一 project context | TC-AW-001..003,010 |
| Worktree Coordinator | Context Resolver | read | allowed | 校验 identity/layout/path/owned namespace | identity negative fixture |
| Worktree Coordinator | Operation Lock | runtime | allowed | 每 project mutation 单写互斥 | concurrent fixture |
| Worktree Coordinator | Capacity Profiler / Filesystem Probe | read | allowed | permission、filesystem identity、bounded checkout upper bound、required/available bytes | O-AW-01 deterministic/fault fixture |
| Worktree Coordinator | Native Git Adapter | read/write | allowed | worktree/ref probe 与 allowlisted mutation | command spy + post-observation |
| Worktree Coordinator | Durable Intent Store / Evidence Adapter | write | allowed | worktree 外、store-local temp、fsync+atomic replace+parent-dir fsync+checksum/readback 后追加 phase/receipt | O-AW-02 recovery/fault fixture |
| Leg Coordinator | Context Resolver / Worktree Health | read | allowed | 决定 source-default 与 artifact-integration mode | TC-AW-008/009 |
| Leg Coordinator | Existing CR-050 typed policy | contract | allowed-with-override | 复用 argv/ref/OID/authz/fresh-proof，不复用 paired-default target assumption | applicability contract test |
| Leg Coordinator | Native Git Adapter | read/write | allowed | 逐 leg plan/execute/postcheck | TC-AW-005..010,014 |
| Leg Executor | Evidence Adapter | write | allowed | 每 leg 追加独立 result | correlation/schema check |
| Aggregate Gate | Validated Leg Results | read | allowed | 只从 matching results 复算 overall | aggregate table test |
| Aggregate Gate | Existing CR/state/current writers | write-via-adapter | allowed | 仅 validated all-PASS 后推进；非 PASS 保持 active/blocked | projection negative fixture |
| Migration Preflight | Context Resolver / Worktree Health | read | allowed | 只读生成 manifest | TC-AW-012/013/015 |
| Manual Sync Coordinator | Context Resolver + Lock + Authz + Native Git + Ops Metrics Writer | runtime | allowed, CR-external-only | CR 外显式同步并记录频率/耗时/阻塞原因 | active-CR/authz/ff-only fixture；O-AW-03 threshold check |
| Executors/Coordinator | Existing Result/Ledger Adapter | write | allowed | 不新增第二套 ledger | schema/event checks |

## 3. 禁止依赖

| Forbidden ID | From | To / 行为 | 禁止原因 | 替代路径 | 违反风险 |
|---|---|---|---|---|---|
| FD-AW-01 | Native Git Adapter | CR/state/current writer | 底层工具不能决定 workflow 终态 | 返回 typed observation/outcome，由上层验证 | 单命令成功误关 CR |
| FD-AW-02 | Result/Journal/Receipt | Git mutation | 证据不是命令；读取即恢复不可审计 | 显式 resume operation + fresh observation | 重放旧证据破坏现场 |
| FD-AW-03 | Route Resolver | Worktree/branch mutation | 路由只决定 context，不执行生命周期 | Worktree Coordinator | 路径解析产生隐式写 |
| FD-AW-04 | Worktree Coordinator | Aggregate Result / CR close | worktree verified 只证明一个局部事实 | Leg Result → Aggregate Gate | artifact 局部成功误报 overall |
| FD-AW-05 | Aggregate Gate | Native Git Adapter / Manual Sync | aggregate 是纯验证/投影边界 | 返回 next route，由显式 operation 执行 | 聚合时隐式补偿或同步 |
| FD-AW-06 | Artifact leg | shared main/default update | shared-artifact 模式已显式 override CR-050 paired-default | target project integration；同步走 CR 外 coordinator | 跨项目耦合和越权 |
| FD-AW-07 | Source leg | project integration | source/artifact target 必须异构 | source target 为 source default | 作用域混淆 |
| FD-AW-08 | Current project operation | sibling owned path/ref/worktree/index | sibling dirty 不应被读写或阻断 | 只校验当前 project context | 跨项目污染 |
| FD-AW-09 | Sparse checkout | security authorization | sparse 不是安全边界 | owned-path + changed/staged/committed path hard gate | 隐藏路径仍可被 Git 写入 |
| FD-AW-10 | Any recovery path | reset --hard / clean / automatic stash / force / branch delete / overwrite | CP3-DC-01 明确禁止破坏性自动恢复 | preserve + RECOVERY_REQUIRED/BLOCKED | 用户数据丢失 |
| FD-AW-11 | Manual Sync Coordinator | active artifact CR / CR lifecycle implicit call | 同步是 CR 外运维，默认无 active artifact CR | 显式独立 operation + gate | ref 漂移覆盖活跃 CR |
| FD-AW-12 | Migration Preflight | file/link/worktree/ref mutation | 本 CR 只交付交接能力 | manifest + 后续项目级授权 | 提前迁移真实 artifact |
| FD-AW-13 | Receipt | 承载自身的未来 Git commit OID | 形成不可满足的自引用 | 后续 publication binding event | 不可验证/反复改写 evidence |
| FD-AW-14 | Shared-artifact mode | CR-050 paired-default merge projection | 两套 target 语义不能同时生效 | mode-specific policy dispatch | artifact main 被误写 |
| FD-AW-15 | Capacity fallback | 未校准的固定 512MiB 放行未知 repo | 可能 false-safe/underestimate | bounded profile + 1.5x upper bound + CP5 0 false-safe proof；否则 BLOCKED | 空间不足时进入非原子中间态 |
| FD-AW-16 | Durable Intent Store | 跨设备 rename、copy+delete 冒充 atomic replace，或未 readback 即 mutation | durability/atomicity 不可证明 | store-local temp，同设备 atomic replace；任一步失败在 Git mutation 前 BLOCKED | 崩溃后消费 torn/corrupt intent |
| FD-AW-17 | Ops Metrics / Manual Sync | 达阈值后由 CR-051 自动启用 sync helper | 阈值只授权治理评估，不授权自动化 | 只创建独立条件式同步助手 CR candidate | 偷渡 runtime/remote write 权限 |

## 4. Operation DAG

### 4.1 Project route/check

```text
load explicit metadata -> validate project identity/layout/anchors
  -> resolve runtime paths -> compare legacy/new candidates
  -> unique decision PASS | ambiguous BLOCKED
```

### 4.2 Worktree create/bootstrap

```text
context -> project lock -> identity/path/common-git-dir/branch-role precheck
  -> fresh remote integration query
     exists: verify, never recreate/reset/orphan
     absent: fresh origin/main OID -> exact create-only update -> fresh postcheck
  -> worktree registration/create -> fresh health observation -> result
```

### 4.3 Non-atomic switch/recover

```text
precheck(identity+lock+clean+no-git-op+refs+permissions+space)
  -> capacity profile deterministic + bounded; unknown/failure -> BLOCKED
  -> store-local temp write -> file fsync -> atomic replace
  -> parent-dir fsync -> checksum/readback -> INTENT_DURABLE
  -> git switch attempt
  -> fresh observation
     target exact -> VERIFIED
     original exact -> NO_CHANGE/ROLLED_BACK
     other/unknown -> conditional rollback eligibility
       eligible -> persist rollback intent -> switch -> fresh observation
       not eligible/failure -> RECOVERY_REQUIRED/BLOCKED
```

### 4.4 Heterogeneous legs and aggregate

```text
logical CR + project context
  -> source leg: source default <-> source CR -> independent result
  -> artifact leg: project integration <-> project CR -> independent result
  -> validate correlation/required set/mode targets
  -> aggregate precedence
  -> all PASS only: workflow may advance
     else: preserve facts + explicit resume/abort; no cross-leg rollback
```

### 4.5 CR-external manual sync

```text
explicit sync request/direction/authz
  -> no active artifact CR + project lock + fresh main/integration
  -> ff eligibility
  -> exact ordinary ref update -> fresh postcheck
  -> PASS | BLOCKED; never force/rebase/auto-resolve/reverse-write
```

## 5. CR-050 Dependency Applicability

| Existing boundary | Reuse | Override / guard |
|---|---|---|
| `meta_flow.workspace.git_sync` typed argv runner / exact remote observation | yes | 增加 worktree/common-git-dir/filesystem probes；不放业务状态机 |
| `git_branch_lifecycle` intent/plan/outcome/authz/fresh-proof concepts | yes | mode parameter必须显式；shared-artifact artifact target=integration |
| `REPOSITORY_ORDER` / paired default assumptions | no as universal contract | source/artifact legs独立执行；aggregate required set替代相同 target 假设 |
| paired default `artifact -> project` merge | source/dedicated legacy only | shared-artifact mode policy拒绝 artifact main/default plan |
| append-only JSON result and projection guard | yes | 扩展为 leg result + aggregate result，保持单写 |
| finish recovery refs / fresh reproof | yes with target override | artifact proof against integration/CR branch；switch另受CP3-DC-01约束 |

## 6. 文件/模块边界候选（非 Story 授权）

| 模块边界 | 高层职责 | 依赖限制 |
|---|---|---|
| workspace routing | portable project context、layout/legacy decision | 不 import lifecycle/aggregate |
| workspace Git adapter | argv-only probes/execution、bounded result | 不写 workflow state/evidence |
| project worktree lifecycle | registration/bootstrap/switch/recovery | 依赖 resolver/adapter/evidence；不写 aggregate |
| heterogeneous Git lifecycle | mode-specific base/target、leg result | 复用 typed safety；不触发 manual sync |
| aggregate gate | pure validation/projection decision | 只读 leg results；不 import Git adapter |
| migration preflight | read-only manifest | 不 import mutation executor |

具体 Python 路径和文件 owner 在 CP3 approved 后由 Feature Design Matrix/CP4 确认。

## 7. 循环风险与断环

| Cycle ID | 涉及对象 | 风险 | 当前处理 |
|---|---|---|---|
| CYCLE-AW-01 | route ↔ worktree registry | route 需要 worktree，worktree 需要 route | `ProjectArtifactConfig` 先解析语义；registration只提供read-only health，不反写config |
| CYCLE-AW-02 | journal ↔ recovery | 恢复读取journal后可能覆盖旧phase | 新resume attempt + fresh observation；旧journal append-only |
| CYCLE-AW-03 | leg result ↔ aggregate | aggregate若回写leg会形成多写 | aggregate只引用validated leg refs，不回写 |
| CYCLE-AW-04 | receipt ↔ artifact commit OID | 证据包含承载自身的OID | later binding event断环 |
| CYCLE-AW-05 | CR finish ↔ manual main sync | 单CR若要求main同步会自锁 | manual sync明确CR外；单CR只检查integration expected OID |

所有循环均 `eliminated`；无 accepted cycle 或 blocking spike。

## 8. 决策状态与开放路由

| 对象 | 状态 | 依赖图影响 / 路由 |
|---|---|---|
| CP3-CR051-DQ-01..03；ADR-AW-001..007 | `approved` | 用户当前指令，`2026-07-18T05:46:40Z`；依赖方向冻结但不授权 runtime mutation |
| O-AW-01 | `non-blocking-open` | Capacity Profiler → Filesystem Probe；CP5 证明 deterministic/0 false-safe，否则 auto switch disabled |
| O-AW-02 | `non-blocking-open` | Worktree Coordinator → Durable Intent Store；CP5 fault matrix 全通过前不得 mutation |
| O-AW-03 | `non-blocking-open` | Manual Sync → Ops Metrics；阈值只路由 follow-up CR candidate |

## 9. Gotchas

1. “调用已有 CR-050 service”不等于继承 paired-default 语义；mode/target override 必须在 policy 层可验证。
2. Aggregate Gate 依赖 result validator，不应 import executor；否则可能在判定时产生 mutation。
3. Project lock 是本地单写协调，不等于远端 CAS；远端操作仍需 exact OID 和 fresh postcheck。
4. 手工 sync 可以由同一 native Git adapter执行，但不能由 artifact CR handler隐式调用。
5. 文件所有权最终要在 CP4 落盘；本图只冻结模块方向和禁止依赖。

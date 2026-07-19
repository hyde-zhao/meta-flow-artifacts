---
status: draft-for-cp4
version: "1.2"
feature_id: "FEAT-AW-02"
feature_name: "Recoverable Project Worktree Lifecycle"
source_blueprint: "process/docs/design/CR051-ARTIFACT-WORKTREE-BLUEPRINT.md"
source_hld: "process/docs/design/CR051-ARTIFACT-WORKTREE-HLD.md"
source_adr: "process/docs/design/CR051-ARTIFACT-WORKTREE-ARCHITECTURE-DECISION.md"
source_matrix: "process/docs/design/CR051-FEATURE-DESIGN-MATRIX.md"
related_stories: ["ST-AW-002"]
lld_policy_summary: "ST-AW-002=full-lld"
confirmed_by: ""
confirmed_at: ""
---

# Feature Design：Recoverable Project Worktree Lifecycle

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-18 | meta-se-critical | 落实 CP3-DC-01、integration create-only、O-AW-01 capacity 与 O-AW-02 durable intent 故障契约 |
| 1.1 | 2026-07-18 | meta-dev-debugger | CP5 R2 关闭 CP5-QA-R1-F02 的 ST-AW-002 输出侧：冻结 `observe_worktree -> WorktreeObservation` 与 `evaluate_worktree_health -> WorktreeHealth(observation)` 唯一公共端口，不改变容量、持久化或 mutation 授权边界。 |
| 1.2 | 2026-07-19 | Host Orchestrator（inline-fallback） | CP8 终验回修：正式冻结只读/manual-only resume、attempt-bound CapacityProof、owner/calibration 持久化和显式 journal phase；关闭未记录 design delta。 |

## 摘要

| 项目 | 内容 |
|---|---|
| Feature 目标 | 为每个 project 管理长期 sibling worktree，使 idle integration、active CR、bootstrap、switch、recover 和 safe-remove 可判定、可审计、可重入 |
| 推荐方案 | 新增 project-worktree coordinator；复用 argv-only Git adapter与 CR-050 typed safety，不复用 paired-default target；所有 switch 前先完成 capacity proof 和 worktree 外 durable intent |
| 关键取舍 | Git switch 不宣称原子；无法证明 durability/capacity/recovery 时禁用自动 switch并 `BLOCKED` |
| 下游 Story | ST-AW-002 |
| LLD 策略 | `full-lld`：并发、文件系统持久性、远端 ref、授权与恢复均为高风险 |

## 背景与问题

| 问题 ID | 背景 | 触发场景 | 影响 | 若不设计的风险 |
|---|---|---|---|---|
| P-AW-W01 | `git switch`、checkout 写入、进程退出不是原子事务 | CP3-DC-01；UC-AW-002 | 中断后可能处于第三态 | 误报 active/idle 或覆盖用户文件 |
| P-AW-W02 | integration 首建存在远端竞态 | TC-AW-004 | 两个操作者可能同时创建 | 重置/覆盖既有 integration |
| P-AW-W03 | 空间不足可能在 checkout 中途暴露 | O-AW-01 | worktree/index 进入部分写状态 | false-safe 后仍执行 mutation |
| P-AW-W04 | intent store 自身可能 ENOSPC/EACCES/torn/corrupt | O-AW-02 | 恢复依据不可信 | 先 mutation 后无法恢复 |
| P-AW-W05 | 多项目共享 object DB 但必须隔离 index/path/branch | REQ-AW-NF003 | 并行项目与相同 CR ID | sibling 污染、index.lock 争用 |

## 上游依据与输入

| 来源 | 路径 / ID | 被本设计消费的内容 |
|---|---|---|
| Blueprint | FEAT-AW-02；FLOW-AW-02/03 | lifecycle 边界、单写 journal、禁止 aggregate 反向驱动 |
| HLD | §5.1/5.2/6/9/11 | CP3-DC-01、capacity/durable store、bootstrap、NFR、O-AW-01/02 |
| ADR | ADR-AW-001/002/004/005 | 非原子恢复、worktree 拓扑、evidence DAG、create-only |
| Domain Map | OBJ-AW-03/04/05/08；§3..5 | registration、journal、capacity、bootstrap 状态语义 |
| Dependency Map | §2..5 | resolver→coordinator→adapter/evidence 单向依赖和禁止命令 |
| Feature Matrix | `process/docs/design/CR051-FEATURE-DESIGN-MATRIX.md` | core planning lane 的 required/full-lld 判定；本 pack 不代写矩阵 |
| Product | ST-AW-002；REQ-AW-004..007/013/C002..003/NF003..005 | create/check/list/remove、分支角色、隔离、安全、可重入 |
| Scenario | TC-AW-004/005/007/010/011/012/014/015 | bootstrap、隔离、namespace、identity、remove、dry-run、并发、stale |

## 目标与非目标

| 类型 | 内容 | 来源 |
|---|---|---|
| Goal | missing integration 从 fresh `origin/main` exact OID create-only；existing 永不 recreate/reset/orphan | ADR-AW-005 |
| Goal | 任一 Git mutation 前通过 identity/lock/clean/no-op/ref/OID/permission/capacity 与 durable intent | CP3-DC-01 |
| Goal | mutation/error 后以 fresh symbolic HEAD/OID/registration/clean/Git-op 证明终态 | ADR-AW-001 |
| Goal | ENOSPC/EACCES/fsync/replace/corruption/kill/cross-device 下提前 Git mutation=0、resume 幂等 | O-AW-02 |
| Non-Goal | source leg 完成、aggregate overall 或 shared-main manual sync | FEAT-AW-03/04、ADR-AW-006 |
| Non-Goal | 自动 reset/clean/stash/force/delete/overwrite、未知现场强制修复 | CP3-DC-01 |
| Non-Goal | 真实 artifact worktree/ref mutation；实现期只允许临时 fixture | REQ-AW-C001 |

## Feature 边界与相邻对象

| 对象 | 本 Feature 负责 | 不负责 | 相邻 Feature | 边界依据 |
|---|---|---|---|---|
| `ProjectWorktreeRegistration` | common Git dir、worktree/project/branch role/health observation | project layout 决策 | FEAT-AW-01 | OBJ-AW-03 |
| `WorktreeOperationJournal` | operation/attempt/phase 单写及 durable record | leg/aggregate result | FEAT-AW-03/04 | OBJ-AW-04 |
| `FilesystemCapacityObservation` | deterministic enumeration/upper bound/available bytes | 业务数据容量规划 | filesystem probe | OBJ-AW-05 |
| `IntegrationBootstrapAttempt` | fresh seed、create-only、race classification | main↔integration 同步 | CR-external coordinator | OBJ-AW-08 |
| branch role | idle integration、active CR、recovery-required | CR branch completion proof | FEAT-AW-03 | Domain Map §4 |

## 现有代码位置与建议文件所有权

| 区域 | 路径 | 当前职责 | 变更方式 / Owner |
|---|---|---|---|
| typed Git adapter | `meta_flow/workspace/git_sync.py` | argv-only runner、exact remote ref/default、repo fingerprint | modify：增加 worktree/common-dir/operation probes；shared，需单写窗口 |
| paired lifecycle | `meta_flow/workflow/git_branch_lifecycle.py` | CR-050 typed intent/plan/outcome/authz/evidence | read/reuse concepts only；不得在本 Feature 继承 paired target |
| project worktree | `meta_flow/workspace/project_worktree.py` | 尚不存在 | create：coordinator/registration/bootstrap/switch/remove 主 owner |
| capacity | `meta_flow/workspace/worktree_capacity.py` | 尚不存在 | create：O-AW-01 主 owner |
| durable store | `meta_flow/workspace/worktree_journal.py` | 尚不存在 | create：O-AW-02 主 owner |
| route input | `meta_flow/workspace/project_artifact_routing.py` | FEAT-AW-01 计划新增 | import-only；不得修改 |
| tests | `tests/test_cr051_project_worktree.py`、`tests/test_cr051_worktree_faults.py` | 尚不存在 | create：本 Feature 主 owner |

## 现状分析

| 维度 | 当前状态 | 缺口 | 约束 |
|---|---|---|---|
| Git | `run_git` argv-only、remote OID 可 fresh query | 无 worktree list/common-dir/operation-state typed probe | adapter 不拥有业务终态 |
| Lifecycle | CR-050 有 typed attempt、fresh snapshot、append-only JSON | paired default、无 worktree journal/capacity/recover state | only safety reuse + target override |
| Persistence | `write_json_result(..., "x")` append-only | 未证明 fsync/dir-fsync/checksum/readback | mutation 前必须 durable |
| Testing | branch bare-remote fixtures 已存在 | 无 filesystem fault、kill、cross-device、双 worktree | 只能 temp repo/bare remote |

## 推荐方案与模块变更

| Module | 变更 | 输入 | 输出 | 失败路径 |
|---|---|---|---|---|
| `project_worktree` | typed coordinator、project lock、registration/role、bootstrap/create/check/list/remove/switch/resume | PASS `RouteDecision`、typed request/authz、adapter/probes | plan/attempt/health/observation refs | 任一未知→BLOCKED；不强制修复 |
| `worktree_capacity` | 枚举 checkout write set，产生确定 upper bound 与 profile eligibility | tree/index/sparse/profile/filesystem observations | `FilesystemCapacityObservation` | no permission/enumeration/profile/error-bound→BLOCKED |
| `worktree_journal` | store-local temp durability protocol、checksum/readback、append-only phases | canonical `SwitchIntent`/phase record | durable record ref | ENOSPC/EACCES/fsync/replace/readback/corruption→BLOCKED |
| `git_sync` probes | worktree list porcelain、common dir、symbolic HEAD、OID、clean、Git-op state、available bytes | validated argv/path | bounded typed observations | timeout/nonzero→unknown；上层 fresh observe |

## 方案对比与决策记录

| Decision ID | 方案 | Pros | Cons | Impact Surface | Recommendation | When to switch |
|---|---|---|---|---|---|---|
| CP3-DQ-01 | durable intent + conditional rollback | 自动旅程可恢复、可审计 | 状态机与 faults 较密 | files/worktree/recovery | 已批准推荐 | O-AW-01/02 不能闭环则切人工-only |
| CP3-DQ-01 | 禁用自动 switch，输出人工步骤 | 权限/实现最小 | 日常 begin/finish 人工化 | operator | 批准备选 | CP5 任一 safety proof 失败 |
| FD-AW-W01 | 分离 capacity/journal/coordinator | 故障注入和 owner 清晰 | 三模块 | implementation/testing | 推荐 | 仅模块极小且无相互 import cycle 时可合并私有实现 |
| FD-AW-W02 | 复用普通 append-only JSON writer | 代码少 | 缺 fsync/dir-fsync/readback | durability | 拒绝直接复用 | 只有升级公共 writer 达 O-AW-02 后才可复用 |

## 数据模型与状态

### 核心对象

| Object | Owner | 关键字段 | 兼容性 / 不变量 |
|---|---|---|---|
| `ProjectWorktreeRegistration` | lifecycle | schema/project/config digest/common-git-dir fingerprint/worktree ref/branch role/observed refs/health | runtime absolute path 不作 canonical identity；main 不是合法项目 role |
| `WorktreeOperationRecord` | journal | operation_id/attempt/sequence/project/worktree stable ID/original+target ref/OID/phase/capacity ref/checksum/previous ref | append-only phase DAG；单 writer；record checksum 排除自身字段 |
| `FilesystemCapacityObservation` | capacity | filesystem ID/profile/tree/index/sparse digests/enumeration coverage/estimated/upper-bound/required/available/safety/eligible/reason | 同观测确定；错误/未知不得 eligible |
| `IntegrationBootstrapAttempt` | lifecycle | fresh main seed OID/integration before/after/create decision/postcheck/terminal | existing 不 mutation；race 后必须 fresh observe |
| `WorktreeObservation` | lifecycle | schema、project/repo/worktree identity、common-dir、HEAD ref/OID、dirty/staged/untracked、Git-op、registry/role、observed_at、route config digest、observation digest | `observe_worktree` 产生 rich immutable snapshot；probe unknown 必须是显式 typed 值，不能用缺字段或 false 代替 |
| `WorktreeHealth` | lifecycle | project、decision、`observation: WorktreeObservation \| None`、`observation_digest`、worktree/journal state、active operation、reason codes | 只由 `evaluate_worktree_health` 产生；`HEALTHY` 必须携带 observation 且 digest 精确一致；任一非 HEALTHY 都不授权 mutation |

### Operation 状态机

```text
PLANNED
 -> PRECHECKED
 -> CAPACITY_PROVED
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

每次 `resume/recover` 首先 fresh observe，再依据 checksum/readback 通过的 journal 选择 `ALREADY_RECOVERED|RECOVERY_IN_PROGRESS|RECOVERY_REQUIRED/BLOCKED|verified terminal`；读取 record 本身不触发 mutation。

## API / 接口设计

| Interface ID | 调用方 | 被调用方 | 输入契约 | 输出契约 | 错误模型 |
|---|---|---|---|---|---|
| IF-AW-W01 `plan_worktree_operation` | Host/FEAT-AW-03 | coordinator | PASS context、operation、project、target role/ref/OID、dry-run/authz ref | ordered immutable plan | invalid identity/role/ref/authz→BLOCKED |
| IF-AW-W02 `observe_worktree` | coordinator/check/list | Git/filesystem probes | context + registered stable ID | fresh immutable `WorktreeObservation` | probe unknown→snapshot 中显式 typed unknown；无 mutation |
| IF-AW-W02B `evaluate_worktree_health` | coordinator/check/list/FEAT-AW-03 | lifecycle pure evaluator | observation（可空）、journal state、active operation、expected project/route digest、freshness policy | `WorktreeHealth`，原样嵌入 observation 并复制其 digest | observation 缺失、digest/project/route 不匹配、stale 或任何 unknown→非 HEALTHY；无 mutation |
| IF-AW-W03 `bootstrap_integration` | create flow | coordinator | project lock、fresh main/integration observations、typed authz | typed attempt + post-observed OID | race/query/authz failure→NO_CHANGE/BLOCKED |
| IF-AW-W04 `prove_capacity` | switch precheck | capacity profiler | checkout FS + tree/index/sparse/profile observations | eligible capacity record | error-bound unknown/permission/enumeration→BLOCKED |
| IF-AW-W05 `persist_phase` | coordinator | journal | next phase + previous record + capacity/observation refs | checksum/readback-verified durable ref | any persistence fault→BLOCKED before Git |
| IF-AW-W06 `execute_switch` | explicit operation | coordinator | durable intent ref + fresh precheck | verified terminal/recovery-required | command success alone never PASS |
| IF-AW-W07 `resume_worktree_operation` | user/Host | coordinator | operation/attempt + explicit request/authz when mutation possible | idempotent observed route | corrupt/missing/third-state→preserve+BLOCKED |
| IF-AW-W08 `safe_remove` | explicit operation | coordinator | exact identity、idle/no-active/recovery proof、clean/no-op、independent authz | removed/no-change/blocked | no force/remove unknown target |

错误至少包含 `project`、`repo_role=artifact`、`step`、safe argv summary、相关 before/expected/actual OID、`error_code`、`resume_route`。稳定 code 包含 `route_invalid`、`lock_unavailable`、`dirty_current`、`git_operation_active`、`capacity_unproven`、`intent_not_durable`、`post_observation_mismatch`、`rollback_ineligible`、`recovery_required`、`bootstrap_race`、`remove_unsafe`。

## O-AW-01：容量证明契约

1. 分别识别 checkout target filesystem 与 durable store filesystem；不得用一个 filesystem 的 free bytes 替代另一个。
2. 主路径输出可复算 `enumeration_coverage`、`estimated_checkout_write_bytes`、确定 upper bound、profile version/digest 与 available bytes。
3. `profile_required = ceil(1.5 * upper_bound)`；无权限、无法枚举、profile/tree/index/sparse 不匹配、误差上界未知均 `eligible=false/BLOCKED`。
4. 512MiB 不是未知 repo 的默认通行证。只有 bounded profile 在 CP5 fixtures 证明 `profile_required <= 512MiB` 且该 profile 的 false-safe=0、underestimate=0 时，才可取 `required_bytes=max(512MiB, profile_required)` 作为保守 floor/fallback；否则禁用 auto switch。
5. available 必须不小于 required，且观测与 mutation 间受同一 project operation lock/attempt 约束；重新尝试必须重新观测。

必须覆盖四类 fixture：常见 checkout；已知误差/阈值边界；目录/对象无权限；tree/index/sparse 无法枚举。任何实际写入量大于预测 upper bound 均记 underestimate 并使 profile 校准失败，不能只提高 warning。

## O-AW-02：Durable Intent Store 契约

store 位于目标 worktree 外，按 project/operation/attempt/sequence 使用 append-only record。每个 record 必须在同一 store filesystem 完成：

```text
canonical payload + checksum
 -> store-local exclusive temp write
 -> file flush + fsync
 -> atomic replace/commit to new final record
 -> parent-directory fsync
 -> reopen + checksum/readback
 -> INTENT_DURABLE
 -> 才允许一次 Git mutation
```

跨设备 path 不得依赖 rename，也不得用 copy+delete 冒充 atomic replace；无法保证 store-local temp 时直接禁用 auto switch。ENOSPC、EACCES、file-fsync、replace、dir-fsync、readback mismatch、torn/corrupt record、process kill、cross-device 任一故障均要求 Git runner mutation call count=0。恢复只能消费 checksum/readback 有效的最长连续 phase chain；损坏/缺口保留且 `RECOVERY_REQUIRED/BLOCKED`，不得覆盖修补旧 record。

## 关键流程

| Flow ID | 触发条件 | 主流程 | 异常流程 | 输出 / 状态变化 | 观测点 |
|---|---|---|---|---|---|
| FLOW-AW-W01 create | target 未注册 | context→lock→identity/non-nested/empty-owned→bootstrap→worktree add→fresh health | 任一步失败 fresh observe/保留事实 | idle-integration 或 BLOCKED | registration/ref/OID/role |
| FLOW-AW-W02 bootstrap | integration fresh-observed absent | fresh main exact OID→ordinary create-only→fresh integration postcheck | reject 后 fresh observe：同 seed=NO_CHANGE；异值/unknown=BLOCKED | attempt result | before/seed/after OID |
| FLOW-AW-W03 switch | idle/active role转换 | full precheck→capacity→durable intent→一次 argv switch→fresh observe | fresh error observe→conditional rollback 或 preserve | verified/rolled-back/recovery-required | HEAD/OID/registration/clean/op |
| FLOW-AW-W04 rollback | error 后满足全部 eligibility | fresh original ref/OID stable + clean/no-op/permission/space→durable rollback intent→一次 switch→observe | 任一未知/失败 preserve | ROLLED_BACK 或 BLOCKED | eligibility inputs |
| FLOW-AW-W05 check/list | 任意注册状态 | `observe_worktree` 只读生成 snapshot→`evaluate_worktree_health` 纯函数裁决→role/stale diagnostics | probe unknown、digest/project/route mismatch 或 stale→非 HEALTHY | 不 mutation；非 HEALTHY 不形成 mutation 授权 | snapshot digest、exact stale fields、health reason codes |
| FLOW-AW-W06 remove | 独立授权请求 | identity+clean/no-op+idle+no active/recovery/ref need→非 force remove→observe | dirty/unknown/active/mismatch→BLOCKED | removed/no-change/blocked | removal proof |

## Integration Bootstrap 细则

| Observation | 允许动作 | Terminal |
|---|---|---|
| remote integration exists | 只验证 identity/role；不 recreate/reset/orphan | NO_CHANGE 或 BLOCKED |
| absent + fresh `origin/main` OID | project lock 内一次 ordinary exact create-only update | exact postcheck→PASS |
| create rejected + fresh integration == seed | 不重试 | NO_CHANGE |
| create rejected + fresh integration != seed | 保留远端事实 | BLOCKED |
| main/ref query unknown | 零 mutation | BLOCKED |

## 人机协作与确认点

| 确认点 | 触发条件 | 需要谁确认 | 推荐方案 | 备选方案 | 不授权项 |
|---|---|---|---|---|---|
| O-AW-01 closure | CP5 capacity fixture | CP5 reviewer/user | 仅 0 false-safe/underestimate 的 bounded profile 开 auto switch | disable auto switch | 不授权真实 worktree |
| O-AW-02 closure | CP5 fault matrix | CP5 reviewer/user | 全 persistence fault 在 mutation 前阻断 | manual-only | 不授权 destructive recovery |
| runtime operation | 实现完成后的真实 create/switch/remove/bootstrap | 操作者 | operation/project/ref/OID scoped authz | dry-run/fixture | CP3/CP5 approval 不自动继承 |

## 异常、失败与降级策略

| Failure ID | 失败条件 | 系统行为 | 用户可见影响 | 恢复 / 回退 | 测试入口 |
|---|---|---|---|---|---|
| F-AW-W01 | current dirty 或 Git op active | mutation 前 BLOCKED | 保留现场 | 用户处理后 fresh retry | TC-AW-006/011 |
| F-AW-W02 | capacity 未证明 | 不写 intent、不调 Git switch | auto switch unavailable | manual-only 或修正 profile | O-AW-01 matrix |
| F-AW-W03 | intent durability 任一步失败 | BLOCKED，Git mutation=0 | operation 未开始 | 修复 store 后新 attempt | O-AW-02 matrix |
| F-AW-W04 | switch 命令异常/退出0但 post mismatch | fresh observe；不报 PASS | 可能 recovery-required | 条件 rollback 或人工入口 | fault/observation matrix |
| F-AW-W05 | rollback eligibility 任一 unknown | preserve worktree/CR branch/journal | BLOCKED | observation-based manual recovery | CP3-DC-01 |
| F-AW-W06 | bootstrap race different OID | 不 force/retry；BLOCKED | integration 需重规划 | fresh plan/人工 | TC-AW-004 race |
| F-AW-W07 | stale/prunable/moved registry | check/list 精确诊断，不 repair/switch/delete | health non-PASS | inspect/prune/relink 建议 | TC-AW-015 |
| F-AW-W08 | remove target unknown/dirty/active/mismatch | 不 force/remove/rm | removal BLOCKED | finish/abort/recover + 独立 authz | TC-AW-011 |

## 权限与安全

| Rule ID | 规则 | 触发条件 | 失败行为 | 测试入口 |
|---|---|---|---|---|
| SEC-AW-W01 | 只消费 FEAT-AW-01 PASS context/owned proof | 所有 operation | invalid/stale context BLOCKED | TC-AW-010 |
| SEC-AW-W02 | sibling dirty 不读取、不阻断、不改；current dirty 阻断 | precheck | crossover=0 | TC-AW-005/014 |
| SEC-AW-W03 | Git 全部 argv-only、ref 先 check-ref-format | Git operation | unsafe input 100% reject | TC-AW-007/NF004 |
| SEC-AW-W04 | 禁止 reset --hard/clean/stash/force/delete/overwrite | recovery/remove/bootstrap | command count=0 | dangerous-command matrix |
| SEC-AW-W05 | sparse 不是安全边界 | path gate | owned-path 不满足即 BLOCKED | sparse negative |
| SEC-AW-W06 | 真实 artifact deny by default | tests/CP6 | 仅 temp fixtures可 mutation | touched-path/ref audit |

## 测试与验收策略

| 验收对象 | 测试层级 | 覆盖场景 | 自动化方式 | 手工入口 |
|---|---|---|---|---|
| state/role/plan | unit | REQ-AW-004..007/NF005 | table/property tests | 无 |
| capacity | unit + fault fixture | O-AW-01 | fake filesystem/enumerator + measured write oracle | CP5 审查 calibration report |
| journal durability | fault injection | O-AW-02 | injectable file ops/kill boundary | CP5 审查 fault matrix |
| bootstrap/worktree | local integration | TC-AW-004/007/011/015 | temp existing-control + bare remote + sibling roots | 真实 remote 后置 |
| isolation/recovery | multi-project integration | TC-AW-005/010/012/014 | 两项目 temp worktrees、Git spy/touched audit | 无 |

## 实现顺序

| Step | 内容 | 前置条件 | 输出 | 验证入口 |
|---:|---|---|---|---|
| 1 | 冻结 types/error/state transition 与 probe contract | FEAT-AW-01 API frozen | pure contracts | unit |
| 2 | 独立完成 capacity profile 和 durable store fault harness | Step 1 | O-AW-01/02 proof candidates | fault matrix |
| 3 | 完成 observation/registration/role/check/list | Step 1 | read-only health | TC-AW-007/010/015 |
| 4 | 完成 bootstrap/create/remove plan/execute | Steps 2/3 | lifecycle operations | TC-AW-004/011 |
| 5 | 完成 switch/rollback/resume 状态机 | Steps 2/3 | CP3-DC-01 closure | kill/recovery/idempotency |
| 6 | 两项目集成、dry-run、危险命令和真实路径 deny 审计 | Steps 4/5 | CP5 evidence | TC-AW-005/012/014 |

## Story 拆分建议与 LLD 策略

| Story ID | feature_design_refs | lld_policy.required_level | 触发原因 | 必须进一步设计的问题 | 可用设计证据 |
|---|---|---|---|---|---|
| ST-AW-002 | `process/docs/features/cr051-worktree/{DESIGN,TEST-PLAN,TASKS}.md` | full-lld | non-atomic state、filesystem durability、concurrency、Git/authz、remote CAS | 精确 record schema、lock implementation、probe argv、capacity estimator 算法/校准数据、fault injection seams、CLI surface | Story LLD |

## 下游消费契约

| 消费方 | 消费时机 | 输入契约 | 输出 / 状态要求 | 降级策略 |
|---|---|---|---|---|
| FEAT-AW-03 | artifact leg begin/finish/abort | fresh PASS worktree health + explicit target role/ref/OID | verified worktree operation/observation refs | non-PASS 则 leg BLOCKED |
| story-manager | CP4 | refs + full-lld | Story 卡写入 design refs/policy | 缺失则 CP4 FAIL |
| lld-designer | CP5 前 | 本 pack + ST-AW-002 | 字段/函数/命令/fault seam LLD | O-AW-01/02 无可执行 test design 则 BLOCKED |
| meta-qa | CP7 | TEST-PLAN、fault evidence、touched audit | 独立验证 0 false-safe/underestimate、0 early mutation、幂等 recovery | 不满足则 disable auto switch/NEEDS_REWORK |

## 风险与回退

| Risk ID | 风险 | 影响 | 缓解 | 回退 |
|---|---|---|---|---|
| R-AW-W01 | 容量模型出现 underestimate | checkout 中间态 | measured oracle、0 tolerance、profile versioning | 立即禁用对应 profile/auto switch |
| R-AW-W02 | durable record 在崩溃后看似存在但未 durable | 错误恢复 | fsync/dir-fsync/checksum/readback + fault injection | preserve + manual-only |
| R-AW-W03 | project lock 被误当 remote CAS | integration race | fresh remote OID/create-only/postcheck | BLOCKED，不 force |
| R-AW-W04 | 自动 rollback 覆盖用户中断后改动 | 数据损失 | eligibility 全项 fresh recheck | 不 rollback，RECOVERY_REQUIRED |
| R-AW-W05 | 与 CR-050 paired default 语义串线 | shared main 被触碰 | mode/target contract，artifact main deny test | policy BLOCKED |

## Gotchas

1. `git switch` 退出 0 也必须重新观测 symbolic HEAD、OID、registration、clean 和 Git-operation 状态。
2. 512MiB 只能服务已证明 bounded profile；它不是无法枚举仓库的通行证。
3. file fsync 成功不代表目录项 durable；atomic replace 后仍必须 parent-directory fsync 和 readback。
4. project lock 只处理本地单写，不能替代远端 create-only 竞态后的 fresh observation。
5. journal 是证据而非 Git truth；resume 永远从 fresh observation 开始。
6. safe remove 只能用 Git 的非 force worktree remove；不得降级成 `rm -rf` 或 sibling prune。

## CP8 终验回修设计增量（ST-AW-002）

本节合并 `process/design-deltas/ST-AW-002.delta.json`。它收窄自动恢复能力，不放宽任何既有安全不变量：

1. `resume_worktree_operation` 固定为只读、manual-only 恢复入口。它只校验 durable chain 并执行 fresh observation，Git mutation 必须为 0；目标态或原态返回 `NO_CHANGE`，第三态、dirty、Git operation active、链异常或身份漂移返回 `RECOVERY_REQUIRED`。当前版本不创建自动 rollback attempt。
2. 只有新的 mutation attempt 才必须重新证明容量。`CapacityProof` 必须绑定 project、repository、operation、attempt、before observation、target ref/OID、profile/calibration digest、checkout/store filesystem observation、有效期和 canonical digest；`execute_switch` 在同一 project operation lock 内对持久化 proof、当前 calibration 与 fresh pre-observation重新校验。
3. durable store 必须持有 `owner.json`，绑定 project、repository、sibling-root 与 target-path digest；跨项目复用、权限错误或身份不一致均在 Git 前阻断。
4. calibration 与 revocation 记录持久化到 `calibrations/<profile-digest>.json`。已撤销、计数变化、profile/digest 不匹配或过期的 proof 不可跨 attempt 复用。
5. switch journal 的有效 phase 固定为 `CAPACITY_PROOF → INTENT → INTENT_SEAL → OBSERVATION_REQUIRED → FINAL_OBSERVATION`。每条 record 校验 project/repository/operation/attempt、sequence、previous digest 与 phase transition；没有 sealed intent 不得执行 Git。
6. `execute_switch` 每次至多执行一次 `git switch`，无论 exit、timeout 或 exception，终态只由 fresh post-observation 决定；`FINAL_OBSERVATION` 已存在时重复调用不得再次 mutation。

该增量替代旧状态图中“resume 可条件式创建 rollback attempt”的实现承诺；自动 rollback 保留为未来独立设计候选，切换条件是出现真实运维需求、重新通过容量/权限/clean/identity 证明并获得新的 CP3/CP5 授权。

---
feature_id: "FEAT-AW-03"
feature_name: "Heterogeneous Leg Lifecycle"
change_id: "CR-051"
story_ids: ["ST-AW-003"]
status: "ready-for-story-design"
applicability: "required"
lld_policy: "full-lld"
version: "1.2"
updated_at: "2026-07-19"
---

# FEAT-AW-03 异构 Git Leg 实现设计

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-18 | meta-se-critical | 固化 source-default / artifact-integration 异构目标、CR-050 契约适用性、独立 LegResult 与失败边界。 |
| 1.1 | 2026-07-18 | meta-dev-debugger | CP5 R2：关闭 F02/F03；只消费 WorktreeHealth.observation rich snapshot，并把 result payload、外置 write receipt、published handle 拆成无自引用 evidence DAG。 |
| 1.2 | 2026-07-19 | Host Orchestrator（inline-fallback） | CP8 终验回修：抽取 producer/consumer 共享纯 target policy，并以 containment proof + expected-OID CAS 限定 artifact CR ref 清理。 |

## 1. 目标与适用性

FEAT-AW-03 为同一个逻辑 CR 生成并执行两个相互独立的 Git leg：source leg 始终以 source remote default 为 base/target，artifact leg 始终以该项目的长期 integration branch 为 base/target。两条 leg 分别生成可验证结果，交给 FEAT-AW-04 聚合；本 Feature 不判定整体成功。

该 Feature 必须生成实现设计且 `ST-AW-003` 必须使用 full LLD，原因是它覆盖跨仓目标解析、typed authorization、expected OID、远端 ref 写入/删除、fresh proof、失败恢复以及对 CR-050 paired-default 语义的显式覆写。

量化成功标准：

- shared-artifact mode 下，100% artifact leg plan 的 base/target 均为 project integration；artifact main/default plan 数为 0。
- 每个 required leg 生成一个写前 digest 可确定的 immutable `LegResultPayload`，并由单写 writer 返回 external receipt；只有匹配且可重读验证的 `PublishedLegResultHandle` 可进入聚合，陈旧、错 attempt 或未发布 handle 的拒绝率 100%。
- dry-run、precheck failure 与 authorization failure 的 Git mutation 数均为 0。
- sibling artifact checkout 的 dirty 状态不阻断当前项目；current project worktree 的 identity、ownership、dirty 或进行中 Git operation 异常阻断率 100%。
- artifact leg 仅在 `WorktreeHealth.decision=HEALTHY`、`health.observation != null`、`health.observation_digest == digest(health.observation)` 时放行；消费侧平行 health schema 数=0。

## 2. 边界

### 2.1 In Scope

- 按 route mode 生成 source/artifact 两类 `LegPlan`。
- 对 CR-050 的 argv-only、ref 校验、expected OID、typed authz、fresh observation、no-force/no-reset/no-stash 契约进行复用或 target override。
- 每条 leg 独立执行 precheck、plan、execute、post-observe、finish/cleanup proof，生成 `LegAttempt` / immutable `LegResultPayload`，并在外置发布成功后输出 `PublishedLegResultHandle`。
- 记录每条 leg 的真实 progress/effect、阻塞原因和 resume/abort route。

### 2.2 Out of Scope 与相邻边界

| 相邻对象 | 本 Feature 不负责 | 责任归属 |
|---|---|---|
| FEAT-AW-01 | 推断 project identity、layout 或 write route | 只消费已验证 `ProjectContext` / `RouteDecision` |
| FEAT-AW-02 | 创建、切换、修复 project worktree 或 integration bootstrap | 只消费 fresh worktree health |
| FEAT-AW-04 | 计算 overall、推进 CR/state/current、把 PARTIAL 投影为成功 | Aggregate Gate 单写 |
| FEAT-AW-05 | 执行或隐式触发 migration | Migration 仅只读 handoff |
| Manual Sync | 更新 artifact main 或执行 main↔integration | CR 外、独立授权、独立锁与命令 |

## 3. 冻结契约

### 3.1 模式与目标矩阵

| mode / leg | base observation | active CR branch | completion target | 禁止目标 |
|---|---|---|---|---|
| source-default / source | fresh source remote default exact OID | source CR branch | source remote default | artifact ref、陈旧 default OID |
| shared-artifact-project-first / artifact | fresh project integration exact OID | artifact CR branch | project integration | artifact main/default、control checkout branch、sibling project branch |

当 mode=`shared-artifact-project-first` 时，只要 plan 中出现 artifact main/default target，policy dispatch 必须返回 `BLOCKED(policy-target-forbidden)`，且不得调用 Git mutation adapter。

### 3.2 `LegRequest`

| 字段 | 约束 |
|---|---|
| `operation_id` | 逻辑 CR operation 的稳定 ID |
| `logical_attempt` | 单调 attempt；resume 必须显式引用前一 attempt |
| `cr_id` / `project_id` | 与 ProjectContext 一致 |
| `leg_kind` | 枚举 `source` / `artifact` |
| `mode` | source 使用 `source-default`；artifact 使用 `shared-artifact-project-first` |
| `base_ref` / `target_ref` | 必须由 mode policy 解析，不接受自由字符串覆写 |
| `expected_base_oid` / `expected_target_oid` | 执行前 fresh observation；缺失即 BLOCKED |
| `authorization_ref` | typed、动作/仓/目标/attempt 绑定；不可跨 leg 复用 |
| `worktree_health_digest` | artifact leg 必填且必须等于 `WorktreeHealth.observation_digest`；source leg N/A |
| `dry_run` | true 时输出 argv/plan/evidence，但 mutation=0 |

### 3.3 ST-AW-002 `WorktreeHealth` 消费端口

ST-AW-003 不定义第二套 health/observation schema。`build_leg_plan` 只消费 Lane B 冻结的 `WorktreeHealth`：`project_id`、decision、`observation: WorktreeObservation | None`、`observation_digest`、worktree/journal state、active operation 和 reason codes。rich identity/common-dir/HEAD/ref/OID/dirty/staged/untracked/Git-op/registry/role/observed_at 字段只能从 `health.observation` 读取。

Artifact leg 的放行 predicate 固定为：decision=`HEALTHY`、observation 非空、recomputed observation digest 与 `observation_digest` 精确一致、observation project/repo/worktree identity 与 route/request一致、dirty/staged/untracked 全 false、Git-op=`NONE`、role/ref/OID满足当前 phase。unknown、null、digest mismatch 或任一 identity/状态不满足均在 Git 前 BLOCKED。非 HEALTHY 即使携带 observation 也只用于诊断，不授权 mutation。

### 3.4 Immutable `LegResultPayload`

`LegResultPayload` 的最小字段为：schema/version、single-write correlation、leg kind/mode、base/target refs、expected/observed before/after OID、status、terminal、progress、effect、step receipts、blockers、resume/abort route、fresh-observation timestamp 与 canonical payload digest。状态只允许 `BLOCKED | FAIL | IN_PROGRESS | PASS`；`PARTIAL` 只在 progress/effect。

Payload 严禁包含自身持久化 ref、append/write receipt、receipt digest、writer ID、written_at 或需要 append 后才能获得的字段。canonical payload digest 在调用 writer 前由稳定序列化计算；时间不参与 single-write key。Payload PASS 只表达 Git leg 事实已满足 required steps、fresh target/cleanup proof和 forbidden mutation=0，不等于已经发布或可聚合。

### 3.5 外置发布对象与无自引用 DAG

- `single_write_key = digest(operation_id, logical_attempt, cr_id, project_id, leg_kind)`。
- `LegResultWriter.append(single_write_key, payload)` 对该 key 只写 payload 一次；同 key+同 digest 幂等返回既有 receipt，同 key+不同 digest 返回冲突。
- `LegResultWriteReceipt(result_ref, payload_digest, writer_id, written_at, receipt_digest)` 位于 payload 外部。`receipt_digest` 绑定 derived single-write key 和其余 receipt 字段，不包含自身值。
- `PublishedLegResultHandle` 组合 payload或result_ref、payload_digest与receipt，仅作为 runtime/evidence-index handle；绝不回写 payload。
- validator 必须从 `result_ref` 重读 payload，重算 payload digest/single-write key/receipt digest，并逐项比对 handle、receipt、expected correlation/mode。
- writer失败返回 `UnpublishedLegResultOutcome(payload, evidence-pending recovery route)`；即使 payload status=PASS，也不得构造或返回可供 aggregate消费的 PASS handle。evidence-only retry复用 byte-identical payload，只调用 writer，不重跑 Git。

Evidence DAG 固定为：`LegResultPayload → LegResultWriteReceipt → validated PublishedLegResultHandle`。不得二次覆盖 payload，不得把 receipt/ref回填 payload。

### 3.6 调用契约

| 调用方 → 被调用方 | 时机 | 输入 | 输出 | 失败/降级 | 调用方同步 |
|---|---|---|---|---|---|
| Leg Coordinator → Project Context Resolver | plan 前 | project/cr identity | validated context/route | identity/layout 多解即 BLOCKED | ST-AW-001 contract |
| Leg Coordinator → Worktree Health | artifact plan 前、每次 resume 前 | project/worktree identity | Lane B frozen `WorktreeHealth` + nested observation | 非 HEALTHY/null/digest mismatch/current异常均 BLOCKED；禁止平铺 schema | ST-AW-002 contract |
| Policy Dispatch → CR-050 typed policy | 每 step plan | mode、leg、action、refs/OIDs | allowlisted argv plan | paired-default assumption 必须 override；无法 override 则 BLOCKED | full LLD 写适用性表 |
| Leg Executor → Native Git Adapter | precheck 通过后 | argv-only plan、cwd、expected OID | command result + observation | 非零/timeout/漂移分类 FAIL/BLOCKED | 禁止 shell 字符串 |
| Leg Executor → `LegResultWriter` | Git/post-proof 完成后 | single-write key + immutable payload | external write receipt | 写失败只返回 unpublished/evidence-pending；不得生成 aggregate handle | 不新增第二套 ledger |
| FEAT-AW-04 → Published Result Validator | 聚合时 | published handles + expected correlation | reread-validated published payloads | raw payload/ref、unpublished、receipt/ref/digest/key错配全部拒绝 | Aggregate 不 import executor |

## 4. 模块与文件影响候选

| 类型 | 候选路径 | 责任 / owner | 并行约束 |
|---|---|---|---|
| 新模块 | `meta_flow/workflow/artifact_leg_lifecycle.py` | ST-AW-003：policy、plan/executor、payload/writer receipt/published handle/validator | primary |
| 复用 | `meta_flow/workspace/git_sync.py` | typed argv、ref/OID/fresh observation | shared；不得放业务状态机 |
| 复用 | `meta_flow/workflow/git_branch_lifecycle.py` | CR-050 safety concepts | shared；只适用性复用，不继承 paired-default target |
| 测试候选 | `tests/test_artifact_leg_lifecycle.py` | ST-AW-003 fixture owner | primary |
| CLI | `meta_flow/cli.py` | 由 ST-AW-004 统一接线 | ST-AW-003 禁止并行修改 |

## 5. 主流程与失败路径

```text
validated context + WorktreeHealth(observation)
  -> require HEALTHY + non-null observation + digest match
  -> resolve mode-specific base/target
  -> fresh observe refs/OIDs + authorization
  -> build argv-only dry-run plan
  -> reject artifact-main/default plan
  -> execute one leg under its operation/attempt
  -> fresh post-observe + finish proof
  -> build immutable LegResultPayload + prewrite digest
  -> append once -> external LegResultWriteReceipt
  -> construct PublishedLegResultHandle without payload rewrite
  -> FEAT-AW-04 rereads, validates and aggregates published handles
```

| 前置校验 | 不通过行为 | mutation |
|---|---|---|
| route/identity/owned namespace 唯一 | BLOCKED | 0 |
| current worktree clean 且无 Git operation | BLOCKED | 0 |
| WorktreeHealth observation 为空、unknown 或 digest mismatch | BLOCKED(worktree-health-invalid) | 0 |
| expected refs/OIDs fresh | BLOCKED(stale-observation) | 0 |
| typed authorization action/target/attempt 匹配 | BLOCKED(authz) | 0 |
| artifact target 为 integration | BLOCKED(policy-target-forbidden) | 0 |
| step 后 target OID 与 proof 不匹配 | FAIL 或 RECOVERY_REQUIRED；保留现场 | 不自动回滚 |
| sibling dirty | 记录 observation，不阻断 | 不触碰 sibling |

## 6. 并发、幂等与恢复

- 同一个 `project_id + operation_id + leg_kind` 只允许一个 active attempt；重复请求返回已有状态或创建显式 resume attempt。
- source/artifact leg 可以独立推进，但不得互相回滚；一条 PASS、另一条失败时保留真实 PARTIAL effect。
- resume 必须重新观察 identity/ref/OID/worktree health，旧 plan 不可直接重放。
- abort 只结束协调状态；不得执行 reset/clean/stash/force/branch delete。
- finish/cleanup 使用 fresh proof；artifact proof 针对 integration/CR branch，绝不针对 main。
- evidence-only retry 必须复用相同 single-write key 与 byte-identical payload；writer成功前没有 published handle，Git runner调用=0。

## 7. Story 下游契约

`ST-AW-003` 的 full LLD 必须细化：函数签名、Lane B `WorktreeHealth.observation` 消费 predicate、`LegRequest/Plan/Payload/WriteReceipt/PublishedHandle` schema、CR-050 applicability table、逐动作 argv allowlist、typed authz、状态转换、resume/abort、exact OID竞态、evidence-only retry、文件影响范围与 fixture。若实现需要平铺第二套 health schema、把 receipt/ref写回 payload、artifact main refresh、跨 leg自动回滚或从 result反向触发worktree mutation，必须返回设计澄清。

## 8. 风险与 Gotchas

| 风险 | 缓解 | 验证 |
|---|---|---|
| 误继承 CR-050 paired-default target | mode-specific policy + forbidden target hard gate | target contract test |
| 单 leg PASS payload 被误当 published/overall PASS | aggregate只接收reread-validated published handle | raw/unpublished rejection test |
| sibling dirty 误阻断 | identity-scoped health | sibling fixture |
| 退出码 0 被当作完成证明 | fresh ref/OID/HEAD/proof post-observe | drift fixture |
| payload/receipt 自引用 | payload排除所有append-time字段；receipt外置且不回写 | prewrite digest + no-forbidden-field test |

Gotchas：integration 是 project 的长期 artifact base/target，不是 artifact main 的别名；`PARTIAL` 只能描述已经发生的效果；dry-run 输出 argv 并不构成 runtime authorization；稀疏检出不是安全边界，owned-path 与 forbidden target 仍须硬校验。

## 9. CP8 终验回修设计增量（ST-AW-003）

1. `artifact_policy.py` 是 source/artifact mode、base/target/active ref 的共享纯策略真相源。producer 构造 plan，consumer 重读 payload 后独立校验；两侧不得各自维护近似字符串规则。
2. artifact finish 只有在 fresh observation 证明 `integration target OID == active CR tip OID` 时才允许清理短期 remote CR ref；证明不足返回 `cleanup_containment_unproven`，不删除任何 ref。
3. 清理仅允许以下完整 argv 形态：`git push --force-with-lease=<active-ref>:<active-oid> origin :<active-ref>`。lease 的 ref 必须等于本 leg canonical active ref，OID 必须等于 current fresh-observed active OID，source 必须为空。
4. 上述 CAS delete 是唯一的 scoped `--force-with-lease` 例外；普通 force、main/default、通配符、错误 ref、错误 OID、非 finish step、reset/clean/stash/rebase 仍 hard deny。
5. cleanup 后 PASS 只表示 integration 已包含 CR tip 且短期 CR ref CAS 清理成功/已不存在；artifact main 既不是 target，也不是 containment proof 输入。

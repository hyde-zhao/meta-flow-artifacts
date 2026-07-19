---
status: draft-for-cp4
version: "1.2"
feature_id: "FEAT-AW-02"
related_story: "ST-AW-002"
open_obligations: ["O-AW-01", "O-AW-02"]
---

# FEAT-AW-02 Feature Test Plan

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-18 | meta-se-critical | 初版；覆盖 CAP、DUR、WT、权限与隔离验证。 |
| 1.1 | 2026-07-18 | meta-dev-debugger | CP5 R2 增加唯一 `WorktreeObservation -> WorktreeHealth(observation)` public port 契约测试；保留 CAP-01..11、DUR-01..14、WT-01..14 全部原义务。 |
| 1.2 | 2026-07-19 | Host Orchestrator（inline-fallback） | CP8 终验回修：增加 typed proof 绑定/过期/撤销、owner 隔离、calibration 持久化、phase/identity 防篡改与 terminal 幂等回归。 |

## 测试范围

| Scope ID | 覆盖内容 | 来源 | 测试层级 | 自动化状态 |
|---|---|---|---|---|
| TP-AW-W01 | registration、idle/active role、namespace 与 stale health | TC-AW-007/015 | unit + fixture | planned |
| TP-AW-W02 | integration exact-OID create-only、existing/no-change、race | TC-AW-004 | bare-remote integration | planned |
| TP-AW-W03 | sibling isolation、identity/owned path、safe remove | TC-AW-005/010/011/014 | multi-project negative | planned |
| TP-AW-W04 | switch/post-observe/rollback/resume state machine | CP3-DC-01、TC-AW-015 | fault integration | planned |
| TP-AW-W05 | capacity 512MiB bounded eligibility 与 fail closed | O-AW-01 | capacity oracle/fixture | planned |
| TP-AW-W06 | durable store 全故障矩阵、0 early Git mutation、恢复幂等 | O-AW-02 | filesystem fault injection | planned |
| TP-AW-W07 | create/check/list/remove/bootstrap/switch dry-run 与重入 | TC-AW-012；NF001/NF005 | golden/property | planned |
| TP-AW-W08 | `observe_worktree` snapshot 与 `evaluate_worktree_health` 包裹式公共端口 | CP5-QA-R1-F02 | unit + consumer contract | planned |

## Worktree Observation / Health Public Port 矩阵

| Case ID | 场景 | 期望 |
|---|---|---|
| PORT-W-01 | 全部 probe 可确定 | `observe_worktree` 返回 immutable snapshot，包含 project/repo/worktree identity、common-dir、HEAD ref/OID、dirty/staged/untracked、Git-op、registry/role、observed_at 与 canonical digest；不返回 `WorktreeHealth` |
| PORT-W-02 | 任一 probe 无法确定 | 对应字段写入带 reason/evidence 的 typed unknown；不得省略、不得折叠为 false，后续 health 必为非 HEALTHY |
| PORT-W-03 | observation 非空、project/route/freshness/journal/role/clean 全部满足且 digest 精确一致 | `evaluate_worktree_health` 返回 `HEALTHY`，`health.observation is observation` 且 `health.observation_digest == observation.observation_digest` |
| PORT-W-04 | observation 为 `None` | 返回 `BLOCKED` 或 `RECOVERY_REQUIRED`，包含 `observation_missing`，mutation calls=0 |
| PORT-W-05 | health 的 expected digest、project 或 route digest 与 observation 不一致 | 返回非 HEALTHY，稳定 reason code 精确区分 mismatch，mutation calls=0 |
| PORT-W-06 | observation stale、dirty/staged/untracked、Git-op active、registry/role unknown 或 journal 非安全终态 | 可在 health 中保留 observation 与 matching digest，但 decision 不得为 HEALTHY，mutation calls=0 |
| PORT-W-07 | 对同一输入重复 health evaluation | canonical 输出与 reason code 稳定；Git/filesystem probe calls=0，证明 evaluator 为 pure function |
| PORT-W-08 | 下游 consumer contract fixture | consumer 只能读取 `health.observation` 的 rich snapshot 字段；不存在平铺到 `WorktreeHealth` 的第二套 HEAD/OID/common-dir/clean schema |

## O-AW-01 容量 Fixture 矩阵

| Case ID | Fixture | 观测/注入 | 期望 |
|---|---|---|---|
| CAP-01 | 常见小 checkout | tree/index/sparse 全枚举，measured write oracle < upper bound | eligible；required=`max(512MiB,ceil(1.5*upper))` 仅在 bounded profile 已校准时；underestimate=0 |
| CAP-02 | 常见中型 checkout | 多文件/权限位/索引变化，完整 enumeration | estimate/upper/required 可复算；actual<=upper |
| CAP-03 | 512MiB 边界内 | `ceil(1.5*upper)=512MiB` | bounded profile 可 eligible；边界无 off-by-one |
| CAP-04 | 边界外 | `ceil(1.5*upper)>512MiB` | 不能以固定 fallback 放行；使用真实 profile_required，空间不足则 BLOCKED |
| CAP-05 | 已知误差模型最大样本 | actual write 等于 upper bound | underestimate=0；重复结果一致 |
| CAP-06 | 故意低估探针 | actual>upper | calibration FAIL；underestimate>0；profile/auto switch 禁用 |
| CAP-07 | false-safe 探针 | decision eligible 但 available<actual | calibration FAIL；false-safe>0；profile 禁用 |
| CAP-08 | object/tree/index 无权限 | EACCES | BLOCKED；intent/Git mutation=0 |
| CAP-09 | 无法枚举 tree/index/sparse | timeout/corrupt/missing | BLOCKED；512MiB 不得 fallback |
| CAP-10 | profile/version/digest 不匹配 | stale calibration | BLOCKED；要求重新估算 |
| CAP-11 | checkout/store 位于不同 filesystem | 两套 available observations | 分别验证；不得互相替代 |

CP5 关闭条件：适用 bounded profile 的常见/误差/无权限/无法枚举矩阵全部执行；`false-safe=0`、`underestimate=0`；任何反例使 O-AW-01 不关闭并禁用 auto switch。

## O-AW-02 Durable Store Fault 矩阵

| Case ID | 注入点 | 期望终态 | Git mutation calls | Resume 期望 |
|---|---|---|---:|---|
| DUR-01 | store-local temp write ENOSPC | BLOCKED / intent_not_durable | 0 | 新 attempt；旧 temp 不消费 |
| DUR-02 | temp/create/write EACCES | BLOCKED | 0 | 修复权限后新 attempt |
| DUR-03 | file fsync failure | BLOCKED | 0 | 未 durable record 不消费 |
| DUR-04 | atomic replace failure | BLOCKED | 0 | final 不存在/旧 final 不覆盖 |
| DUR-05 | parent-dir fsync failure | BLOCKED | 0 | terminal 不得 INTENT_DURABLE |
| DUR-06 | checksum/readback mismatch | BLOCKED / corruption | 0 | preserve，RECOVERY_REQUIRED |
| DUR-07 | torn/truncated final record | BLOCKED / corruption | 0 | 只消费此前连续有效 phase |
| DUR-08 | kill after temp write/file fsync | process interrupted | 0 | orphan temp 不消费；幂等诊断 |
| DUR-09 | kill after replace before dir fsync | durability unknown | 0 | 不推断 durable；preserve/BLOCKED |
| DUR-10 | kill after verified durable、before Git | INTENT_DURABLE | 0 | resume fresh observe 后至多一次 mutation |
| DUR-11 | kill after Git、before post-observation | OBSERVATION_REQUIRED | 1 | resume 先 observe，不盲目重放 switch |
| DUR-12 | cross-device temp/final | BLOCKED | 0 | 不 copy+delete；要求 store-local temp |
| DUR-13 | phase chain gap/previous digest mismatch | BLOCKED / corruption | 0 | 不修补/覆盖旧 evidence |
| DUR-14 | 相同 resume 重放 10 次 | stable terminal | 不超过所需一次 | ALREADY_RECOVERED/REQUIRED 稳定 |
| DUR-15 | store owner 与 project/repository/sibling-root/target digest 不一致 | BLOCKED / owner mismatch | 0 | 不复用跨项目 store |
| DUR-16 | calibration/revocation 跨进程重载 | 当前状态与计数可重读 | 0 | revoked proof 持续失效 |
| DUR-17 | record identity 被篡改后重算 record digest | BLOCKED / identity mismatch | 0 | 不接受“自洽但错身份”的链 |
| DUR-18 | 非法 phase transition | 写入前拒绝 | 0 | 保留既有连续链 |

## Bootstrap / Worktree / Recovery 矩阵

| Case ID | 场景 | 期望 |
|---|---|---|
| WT-01 | integration absent、fresh main seed | 一次 ordinary create-only；after=seed；force/reset/orphan=0 |
| WT-02 | integration exists | NO_CHANGE；ref mutation=0 |
| WT-03 | concurrent create same OID | rejection 后 fresh observe；NO_CHANGE |
| WT-04 | concurrent create different OID | BLOCKED；不 retry force |
| WT-05 | switch exit 0、post HEAD/OID mismatch | 非 PASS；fresh observe→recovery route |
| WT-06 | switch timeout/kill、target verified | resume 返回 VERIFIED_TARGET，不重放 |
| WT-07 | switch error、original verified | NO_CHANGE/VERIFIED_ORIGINAL |
| WT-08 | third state/dirty/Git op/ref drift | preserve worktree/CR branch；RECOVERY_REQUIRED |
| WT-09 | rollback eligible 全 PASS | 先 durable rollback intent，再一次 switch/observe |
| WT-10 | rollback 任一条件 unknown | 自动 rollback calls=0 |
| WT-11 | sibling dirty/current clean | current operation可继续；sibling path/ref/index变化=0 |
| WT-12 | current dirty/identity mismatch | mutation前 BLOCKED |
| WT-13 | unsafe remove 任一条件 | remove/rm/branch-delete calls=0 |
| WT-14 | stale/prunable/ended-CR registry | 只诊断建议；repair/switch/delete=0 |
| WT-15 | typed CapacityProof attempt/target/before observation 任一错绑或过期 | switch 前 BLOCKED；Git mutation=0 |
| WT-16 | persisted calibration 已撤销或 profile/counters 漂移 | switch 前 BLOCKED；不得复用旧 proof |
| WT-17 | 已存在 FINAL_OBSERVATION 的 attempt 重复 execute | 返回相同 observed terminal；新增 Git mutation=0 |

## 风险驱动测试

| Risk ID | 风险 | 测试方式 | 证据 | 未覆盖原因 |
|---|---|---|---|---|
| R-AW-W01 | 用户文件被自动恢复覆盖 | dirty/untracked/Git-op/ref drift fault matrix | destructive argv count=0 | N/A |
| R-AW-W02 | false-safe 空间判断 | measured write oracle 与 profile calibration | false-safe/underestimate counters | N/A |
| R-AW-W03 | torn intent 被当有效 | kill/corruption injection | durable chain validator result | N/A |
| R-AW-W04 | remote integration 被覆盖 | same/different OID race | before/seed/after OID evidence | N/A |
| R-AW-W05 | 多项目交叉 | 两 sibling worktree 并发/交错 | path/ref/index.lock audit=0 | N/A |

## 权限 / 安全 / 失败路径

| Case ID | 触发条件 | 期望行为 | 测试入口 |
|---|---|---|---|
| SEC-W-01 | ref/project/slug 含 option prefix、traversal、newline、shell chars | 100% 输入拒绝；额外命令=0 | argv/ref validator matrix |
| SEC-W-02 | 诱导 reset/clean/stash/force/branch delete/rm | 禁止 argv count=0 | command spy |
| SEC-W-03 | 真实 artifact path 被配置为 fixture target | deny before mutation | real-path deny fixture |
| SEC-W-04 | authz 缺失/项目/ref/OID/operation 不匹配 | BLOCKED；mutation=0 | typed authz matrix |

## 验证命令（实现后）

```bash
PYTHONDONTWRITEBYTECODE=1 PYTEST_ADDOPTS='-p no:cacheprovider' uv run --python 3.11 pytest -q tests/test_cr051_project_worktree.py tests/test_cr051_worktree_faults.py tests/test_workspace_git_sync.py tests/test_git_branch_lifecycle.py
PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 ruff check meta_flow/workspace/project_worktree.py meta_flow/workspace/worktree_capacity.py meta_flow/workspace/worktree_journal.py tests/test_cr051_project_worktree.py tests/test_cr051_worktree_faults.py
```

所有 Git mutation 仅在 `tmp_path` 下的普通 repo/worktree/local bare remote；不得指向真实 artifact remote/worktree。

## 手工验收

| Check ID | 操作 | 期望结果 | 责任方 |
|---|---|---|---|
| MAN-AW-W01 | 审查 O-AW-01 calibration report | bounded profile 0 false-safe/underestimate；失败则 auto switch disabled | CP5 reviewer |
| MAN-AW-W02 | 审查 O-AW-02 fault matrix | 14/14 预期一致，persistence fault 前 Git mutation=0，resume 幂等 | CP5 reviewer |
| MAN-AW-W03 | 真实平台兼容性（后续独立授权） | 只确认平台 worktree/fsync/remote policy；本 CR 不执行 | CP7/CP8 human |

## CP8 终验回修执行证据

- `tests/test_cr051_worktree_faults.py` 覆盖 typed proof 绑定/过期、revocation、owner 跨项目隔离、calibration 持久化、record identity 防篡改和非法 phase transition。
- `tests/test_cr051_project_worktree.py` 覆盖完整五阶段 journal、fresh before/after observation、terminal 幂等、proof 过期、persisted calibration revocation 与 intent payload 错配。
- 定向执行结果：74 项通过；跨模块回归（含 Git sync、CR lifecycle、state/current）307 项与 58 个 subtests 通过。
- 所有可变 Git fixture 均位于 `tmp_path`；真实 artifact worktree/ref/remote mutation 仍为 0。

---
status: confirmed
version: "1.2"
created_at: "2026-07-16"
owner: "host-orchestrator-inline / meta-se"
cr_ref: "CR-050"
source_requirements: "process/docs/product/REQUIREMENTS.md"
source_story_map: "process/docs/product/STORY-MAP.md"
review_gate: "CP3-R3"
supersedes: "process/archive/CR-050/design/CR050-GIT-BRANCH-BLUEPRINT.v1.0.1.md"
---

# CR-050 Governed Git Branch Lifecycle Blueprint

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-16 | host-orchestrator-inline / meta-se | 建立原生 Git 双仓 CR branch start/publish/finish 能力边界。 |
| 1.0.1 | 2026-07-16 | host-orchestrator | 用户新增显式 merge 后标记 changes-requested；正文按原 hash 归档。 |
| 1.1 | 2026-07-16 | host-orchestrator-inline / meta-se | 按 CP2 R2 新增独立 `merge` Capability/Feature、artifact→project ff-only choreography、默认分支写授权、PARTIAL/恢复与 finish gate；保持单一 HLD 和既有 source owner。 |
| 1.2 | 2026-07-16 | host-orchestrator-inline / meta-se | CP3 R3增加Paired Projection Gate，明确artifact-first为治理预写、单仓PARTIAL不可推进workflow投影，并冻结无法实现2/2门时切project-first的回退条件。 |

## 1. 蓝图适用性与目标

CR-050 同时影响 Host/CLI 生命周期、project/artifact 两仓 Git refs、默认分支写入、清理授权和过程证据，属于跨模块、高风险设计，必须有 Blueprint、Domain Map、Dependency Map、HLD 与 ADR。目标旅程为：

```text
open -> publish -> explicit merge -> finish
```

四个动作各自显式触发、各自产生 append-only attempt；相邻动作不得隐式调用下一动作。Git refs、CR/state、operation evidence 分别由既有 owner 单写，不建立第四套总状态。

## 2. 能力地图

| Capability | 用户结果 | Feature | 主要输入 | 主要输出 |
|---|---|---|---|---|
| CAP-GB-01 Governed CR Open | 两仓从各自 fresh remote default exact tip 开启同名 CR branch | FEAT-GB-01 Paired Branch Open | CR intent、workspace route、remote/default override、publication authz | base/upstream/OID、逐仓 open result |
| CAP-GB-02 Committed Ref Publish | 只发布调用前已存在的 commit | FEAT-GB-02 Safe Ref Publish | branch identity、local/remote OID、clean status | exact remote CR tip、逐仓 publish result |
| CAP-GB-03 Explicit Paired FF Merge | 以独立授权把两仓 published tip 快进到各自 remote default | FEAT-GB-04 Paired Default Fast-forward | matching publish result、fresh refs、default-write authz、remote policy | artifact→project merge attempt、default OID、PARTIAL/resume |
| CAP-GB-04 Proof-gated Finish | 重新证明合并后精确清理 CR refs | FEAT-GB-03 Merge Proof and Cleanup | current 2/2 merge evidence、fresh refs、delete authz | ancestry proof、recovery ref、remote/local cleanup result |

编号保留既有 `FEAT-GB-03`，新增 merge 使用 `FEAT-GB-04`，避免重编号破坏追溯；生命周期顺序是 01→02→04→03。

## 3. Feature 边界

### FEAT-GB-01 Paired Branch Open

- 负责：两仓 route/Git/clean/detached/remote/default/ref collision 预检；刷新默认分支；从 exact remote tip 创建同名 branch；复用现有 CR bootstrap；经独立 publication authorization 执行 `push -u`。
- 不负责：选择提交文件、隐式 commit、merge、finish、跨仓原子补偿。
- 产出：逐仓 before/after OID、upstream、terminal status 与 resume route。

### FEAT-GB-02 Safe Ref Publish

- 负责：校验 branch/upstream/clean/non-FF；只 push exact existing commit ref；核验 `remote_cr_tip == local_head`。
- 不负责：stage/commit/amend、PR、merge、force 或文件选择。
- 产出：可被 merge 消费的 matching publish attempt；失配或旧 evidence 不授权 merge。

### FEAT-GB-04 Paired Default Fast-forward

- 负责：在任何写入前对两仓完成 fresh observation、identity、published-tip、default ancestry、authorization 与 policy preflight；随后严格按 `artifact -> project` 更新 exact remote default；每仓立即 post-check。
- 唯一允许的语义：fresh default tip 是 exact published CR tip 的祖先或相等；写后 remote default 必须精确等于 expected CR tip。
- 不负责：merge commit、rebase、squash、force、自动冲突解决、branch-protection 绕过、PR/merge queue API、自动回滚成功仓。
- 产出：2/2 PASS/NO_CHANGE 才 overall PASS；artifact 成功而 project 失败为 PARTIAL，保留两仓 CR branch 并阻断 finish。
- 投影边界：artifact单仓成功只产生append-only repo outcome；只有2/2 post-check通过，Paired Projection Gate才允许CR/state/current出现paired merge PASS。PARTIAL时CR保持active，`paired_projection_advanced=false`。

### FEAT-GB-03 Merge Proof and Cleanup

- 负责：消费 current matching 2/2 merge PASS；重新 fetch/观察并独立验证 exact identity、tip drift、protected policy 与 ancestry；建立 local-only recovery ref；先完成两仓 remote delete，再 `branch -d` 本地 refs。
- 不负责：执行 merge、把 merge attempt 当作删除授权、patch-id 猜测、force-delete、自动 push/delete recovery ref。
- 产出：proof 与 cleanup attempt；证据不足或 merge PARTIAL 时 remote/local delete count=0。

## 4. 共享组件与数据归属

| 组件/对象 | owner | 职责 | 禁止承担 |
|---|---|---|---|
| Native Git Probe/Executor | `meta_flow.workspace.git_sync` 扩展边界 | argv-only、固定 cwd、allowlisted probe/mutation、有界输出 | CR 状态写入、shell、凭据解析、策略绕过 |
| Branch Lifecycle Planner/Executor | branch lifecycle workflow module | typed intent→observations→deterministic plan→per-repo outcomes | 成为 Git ref truth、跨仓事务、隐式下一阶段 |
| CR Start Coordinator | existing CR lifecycle boundary | 解决 branch clean precheck 与 CR bootstrap 写 artifact 的顺序 | 改变旧 bootstrap 默认行为、自动 commit |
| Existing CR/state/current writers | 既有 writer | formal CR/state truth 与阶段迁移 | 保存完整 Git plan/result |
| Operation Result/Ledger Adapter | existing checks/RUN/CR ledger | append-only attempt/ref/correlation | 覆盖旧 attempt、把 PARTIAL 写成 PASS |
| Paired Projection Gate | branch lifecycle workflow module | 聚合2/2 repo outcome并控制existing CR/state/current writer的终态投影 | 从artifact单仓ref推断project成功、在PARTIAL时关闭CR或开放finish |
| Git local/remote refs | 对应 repository/remote | 当前 ref truth | 被 result/state 复制替代 |
| Recovery ref | repository-local Git namespace | 删除后保持 exact tip 本地可达 | merge receipt、远端备份、release tag |

## 5. 集成契约

| 调用方 → 被调方 | 时机/触发 | 输入 | 输出/后续 | 失败/降级 | 调用方同步 |
|---|---|---|---|---|---|
| Host/CLI → Open Coordinator | 用户显式开启 CR 且具 publication authz | CR ID/slug、route、remote/default、dry-run | bootstrap refs + paired open result → requirement clarification | 任一无写 preflight失败：mutation=0；中途失败：PARTIAL/resume | CLI/help、Host route、CR writer adapter |
| Host/Developer → Publish | 两仓显式 commit 完成 | CR/branch、expected OID、dry-run | matching publish result → review/merge eligibility | dirty/wrong/non-FF：BLOCKED；不 stage/commit | CLI/docs/result schema |
| Host/Approver → Merge | publish 后，用户对本次 default write 明示授权 | publish result ref、expected tips/defaults、authz ref、dry-run | 2/2 result经Projection Gate后才产生finish eligibility | preflight failure：0 writes；policy拒绝：BLOCKED/PARTIAL；不推进paired projection；保留 branches | CLI/authz parser/result/projection/Host gate |
| Host/Approver → Finish | current 2/2 merge PASS 且另有 delete authz | merge result、fresh refs、protected policy | cleanup result → CR close evidence | reproof失败或PARTIAL：delete=0 | CLI/CR close gate/docs |

## 6. 生命周期与发布切片

```text
ST-GB-001 / SL-GB-01 open
  -> ST-GB-002 / SL-GB-02 publish
    -> ST-GB-004 / SL-GB-04 explicit paired ff merge
      -> ST-GB-003 / SL-GB-03 proof-gated finish
```

四个 Story 共享 typed intent/result、Git adapter、安全策略和端到端 release gate，且没有独立 owner/rollback/release cadence，因此保留一个 HLD、四个串行 Wave。若后续 forge adapter 需要凭据、PR/queue receipt、独立审批或平台回滚，则必须另立 CR/HLD。

## 7. 权限与 Out of Scope

- CP2/CP3/CP5 只批准产品/设计/实现范围，不等于某次真实远端写授权。
- open/publish、merge default write、finish delete 是三类不同 mutation authority；不得继承、合并或默认复用。
- 永久排除：force、reset --hard、rebase、merge commit、squash、自动冲突解决、自动 stage/commit、branch protection绕过、凭据读取/输出。
- 邻接边界：forge/PR/merge-queue adapter负责受保护平台流程与receipt；本CR的native Git merge只做可被远端策略接受的普通exact fast-forward push，拒绝即fail closed。

## 8. Gotchas

1. `git merge --ff-only` 在本地成功不等于远端 default 已更新；必须以 exact expected OID 的远端更新和 fresh post-check为终态。
2. preflight-all只能降低竞态，不能提供跨仓原子性；artifact成功/project失败必须保留为PARTIAL，不能用第二次危险写“修饰”历史。
3. remote default已等于CR tip只能算该仓NO_CHANGE；仍要验证published identity、授权对象与另一仓结果，不能直接推断整体PASS。
4. branch protection拒绝是远端真实策略，不是可重试成force的临时错误；需要PR/queue时转forge adapter CR。
5. finish必须重新证明事实；旧merge result、CR closed、remote branch absent均不能单独授权删除。
6. 当前artifact工作树包含本CR过程文件，不符合未来open clean gate；本轮只做设计证据，不做真实dogfood。
7. artifact-first的可接受性依赖2/2聚合门；若CP4/CP5不能把attempt writer与workflow projection writer隔离，必须切project-first并重开决策，不能把风险留给CP7。

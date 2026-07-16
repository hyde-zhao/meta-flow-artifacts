---
status: archived-changes-requested
version: "1.0.1"
created_at: "2026-07-16"
owner: "host-orchestrator-inline / meta-se"
cr_ref: "CR-050"
complexity: complex
decision_burden: high
review_gate: "CP3"
source_requirements: "process/docs/product/REQUIREMENTS.md"
superseded_by_scope_change: "CP2 R2 explicit merge baseline; HLD v1.1/CP3 R2 pending after approval"
---

# CR-050 Governed Git Branch Lifecycle HLD

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-16 | host-orchestrator-inline / meta-se | 将 CP2 已批准的原生 Git、双仓、verify-only、fail-closed ancestry 与 committed-ref 边界转化为可评审 HLD；新增 CR start integration、单调跨仓执行、append-only evidence/recovery ref 与风险上限决策。 |
| 1.0.1 | 2026-07-16 | host-orchestrator | 仅更新审查状态：用户撤销 verify-only 产品边界并批准纳入独立两仓 ff-only merge；v1.0 正文作为旧输入设计证据保留，CP2 R2前不生成替代架构。 |

## 1. 问题定义

当前 `meta_flow.workspace.git_sync` 只能汇总两仓状态并顺序 push；它不知道 CR identity、remote default、branch collision、non-fast-forward、merge proof、protected ref 或安全删除。Host 仍依赖手工 Git 命令，容易出现源码/过程证据不同分支、从陈旧 main 开发、dirty tree 被发布、单仓成功冒充双仓完成，或合并证明不足却删除分支。

CR-050 要提供确定性三段旅程：

- 开启：两仓从各自 fresh remote default exact tip 建立同名 CR branch/upstream；
- 发布：只 push 调用前已存在的 commit；
- 完成：外部 merge 后以 exact tip + ancestry 证明清理 remote/local branch。

成功口径为 2/2 起点/upstream、2/2 remote=local HEAD、未证明删除数 0、partial 逐仓证据 100%、dry-run mutation 数 0。

## 2. 已批准边界与不授权

CP2 已批准：原生 Git subprocess；project/artifact 成对管理但不宣称原子事务；finish 只验证外部 merge；squash/rebase 无 forge receipt 时 fail closed；publish 不隐式 stage/commit/amend。

本 HLD 不授权当前工作树执行真实 branch create/push/delete、commit、forge API、凭据读取、自动 merge、force、reset/rebase/stash，也不触碰 `process/quant-lab/**` 或 prelink backup。当前 CR-050 artifact tree 已有过程变更，不符合未来 start 的 clean precondition，不能拿本轮 CP3 当作真实 start dogfood。

## 3. Architecture Gray Areas

| ID | 问题 | 推荐 | 备选 | 切换条件 | CP3 |
|---|---|---|---|---|---|
| AGA-GB-01 | branch open 如何与“CR first”及 clean precheck 同时成立 | 新增显式 CR Start Coordinator：无写校验 intent，先准备两仓 local branch，再调用现有 bootstrap，最后 push tracking refs；旧 bootstrap 不隐式远端写 | A：Host 手工先 branch-open 后 bootstrap；B：直接让旧 bootstrap 默认写远端 | 若 coordinator 无法复用 bootstrap writer，退回显式两步且记录 bootstrap intent，不允许默认隐式写 | DQ-01 |
| AGA-GB-02 | 两仓中途失败是否自动补偿/回滚 | preflight-all + 确定顺序 + 单调执行 + observation-based resume；不撤销成功 remote fact | 自动 delete/recreate/force 回滚；单仓模式 | 只有底层提供真正跨仓 transaction 时重新评审；本轮 partial 保留 | DQ-02 |
| AGA-GB-03 | merge proof、remote auto-delete 与恢复如何持久化 | Git refs是真相；append-only attempt result保存 exact OID；删除前建 local-only namespaced recovery ref；关联现有 RUN/CR ledger | 只打印文本；新 branch state DB；仅记录 OID不建 ref | 若 recovery ref retention 产生真实维护负担，另立 retention CR；不能在 finish 内静默删 | DQ-03 |
| AGA-GB-04 | 用户禁用子 Agent时验证/发布能宣称什么 | inline fallback 可完成确定 fixture，但 CP7最高 PASS_WITH_RISK、CP8最高 READY_WITH_RISK | CP7 前解除限制并取得独立 QA；或要求 READY 时阻断 | 有可验证独立 QA evidence 后只解除该风险，远端/forge等风险独立计算 | DQ-04 |

### Advisor table-first inline synthesis

用户此前要求不使用子 Agent，本轮使用 inline fallback，不声称 custom agent/runtime receipt。advisor 视角如下：

| Lane | 判断 | 进入方案的约束 |
|---|---|---|
| product | 一个完整 CR branch journey，三 Story 串行产生用户价值 | 一个 HLD，三 release slice，不拆独立工具产品 |
| architecture | workflow policy 与低层 Git adapter 分层；Git ref 与 CR state 各自单写 | 禁止 branch state DB、Git adapter 反写 CR/state |
| security | remote delete、ref drift、命令注入和 hidden commit 是主要威胁 | argv-only、allowlist、exact ref/OID、显式 authz、fail closed |
| quality | bare remote fixtures可覆盖 ancestry/partial/idempotency，但不等于真实 forge attestation | CP7 风险上限、负例 100%、dry-run before/after refs |
| operations | partial 无法消除，恢复必须基于重新观察而非补偿猜测 | deterministic order、terminal result、resume remaining steps |

## 4. 候选方案

### Option A：In-place native Git lifecycle service + explicit CR coordinator（推荐）

扩展现有 workspace route/Git adapter，新增 branch lifecycle planner/executor 和显式 CR start coordinator；结果接入现有 checks/ledgers。旧 `workspace push` 保持兼容，不成为 branch policy owner。

优点：无新运行依赖；与 `pyproject` 零 runtime dependency、现有 bare-repo fixture 和 CR writer一致；权限/失败边界可机器测试。代价：需处理 coordinator/bootstrap 的顺序、跨仓 partial 和 append-only attempt contract。

### Option B：安装 Git Town/`gb` 并包一层 adapter

优点：stacked branch/ship 等高级能力可复用。缺点：`gb` 名义不唯一；引入 executable/version/config/sync/force 策略；普通 CR lifecycle 用不到；仍无法替代 Meta Flow 的 CR/evidence/双仓 contract。

### Option C：保留文档化的直接 Git 命令

优点：实现成本最低。缺点：无法稳定生成 machine result、阻断 protected/delete drift、证明 dry-run 零副作用或恢复 partial；当前问题继续存在。

### 适用性矩阵

| 维度 | A | B | C |
|---|---:|---:|---:|
| 无新增必需依赖 | 5 | 1 | 5 |
| CR/双仓/evidence 集成 | 5 | 2 | 1 |
| 删除安全可测试 | 5 | 3 | 1 |
| 跨平台可移植 | 4 | 2 | 3 |
| 实施/维护成本 | 3 | 2 | 4 |
| stacked branch 扩展 | 2 | 5 | 1 |
| MVP 总体 | **24** | 15 | 15 |

Option A 满足当前普通 CR 分支目标；只有真实 stacked/offline ship 场景出现且原生 adapter不足时才启动 Option B follow-up。

## 5. 推荐架构

```text
User / Host explicit action
          |
          v
CLI + CR Start Coordinator -------------------> existing CR bootstrap/state writers
          |
          v
Branch Lifecycle Planner/Executor ------------> append-only operation result
          |                                             |
    +-----+------+                                      v
    v            v                              existing RUN/CR ledgers
Workspace Route  Native Git Adapter
                       |
             project/artifact repos + remotes
```

原则：Git refs 是 ref truth；formal CR/state 是 workflow truth；operation result 是观察/执行证据。三者由只读 correlation 校验连接，不互相复制为新总真相。

## 6. 模块职责与集成契约

| 模块 | 职责 | 输入/输出 | 失败/降级 | 调用方同步 |
|---|---|---|---|---|
| CLI/Host adapter | 暴露显式 start/open、publish、finish、dry-run、JSON/output；传递退出码 | typed args → human+machine result | 参数/授权错误不调用 executor | `meta_flow/cli.py`、README、USER-MANUAL、Host rules |
| CR Start Coordinator | `intent validate → local branch prepare → existing bootstrap → remote tracking push` | CR intent + branch plan → bootstrap refs + attempt result | partial 不自动删除；输出 resume | `cr_lifecycle.py` 复用 writer，不改变旧 bootstrap 默认行为 |
| Branch Planner | observations + policy → ordered immutable plan/digest | 无 shell string | unknown/default/ref conflict → BLOCKED | 三 operation 共用 types/policy |
| Branch Executor | 按 plan 执行 allowlisted mutations并逐步 post-check | plan → per-repo outcomes | 首个不可恢复错误停止后续 destructive step | 不直接写 CR/state/ledger |
| Native Git Adapter | route repo、probe、argv-only run、有界输出 | args/cwd → typed command result | timeout/nonzero/redaction | 扩展现有 `workspace.git_sync` |
| Result/Ledger Adapter | attempt result校验、落盘、追加 ledger ref | outcome → versioned result/ref | result invalid则不能推进 | 复用 checks/RUN/CR ledger，不新建 branch DB |

## 7. 关键流程

### 7.1 Governed start/open

1. 校验 CR ID/slug/ref format、显式 publication authorization、workspace route；只读观察两仓。
2. 两仓同时通过：Git version、clean index/worktree、attached HEAD、当前 default branch、remote symbolic HEAD/override、ref collision/protected policy。
3. actual 模式依次 `fetch --prune`、默认分支 `pull --ff-only`，重新锁定 remote default exact OID；dry-run只做 read-only remote observation并列出 would-run。
4. 从 exact remote default tip 在两仓创建/切换同名 local CR branch；失败即 PARTIAL，不做 force/auto-delete。
5. coordinator 调用现有 CR bootstrap writer创建 formal CR/route/context；不复制 writer。
6. 逐仓执行 exact `push -u` 并核验 remote branch OID/upstream/base；落 result与ledger ref。

这解决“bootstrap 先写文件导致 dirty precheck 永远失败”的顺序冲突，同时避免让旧 `cr bootstrap` 突然产生远端副作用。

### 7.2 Publish committed refs

1. 重新观察两仓，要求 expected branch、exact upstream、clean tree、attached HEAD。
2. actual 模式刷新 remote-tracking refs；remote tip 必须等于或为 local HEAD 祖先。dry-run不更新 refs，freshness不足时只输出 conservative BLOCKED/plan。
3. 顺序 push exact branch refspec，不含 force；每仓立即以 read-only remote query核验 OID。
4. 后仓失败时前仓保持成功事实，overall=PARTIAL，给出重新观察/重试入口；不回退远端 commit。

### 7.3 Proof-gated finish

1. fetch/observe 两仓；known tip来源按 matching remote target、local branch、可信 start/publish result解析，来源冲突即 BLOCKED。
2. 两仓全部验证 branch identity、tip无漂移、非 protected、known tip 是 fresh remote default tip 祖先。remote absent 也不能跳过 proof。
3. 两仓建立/复用 local-only recovery ref；不同 OID collision 阻断。
4. 两仓 local default 可 ff-only 后切换/刷新；不自动 merge/rebase。
5. 逐仓删除 exact remote target并核验 absent；两个 remote phase都完成后才 `branch -d` local CR branches。
6. 保留 recovery refs和result；不自动 push/delete recovery refs。squash/rebase无 receipt时结论为 ancestry_unproven/BLOCKED。

## 8. 安全与失败设计

- 所有 Git 输入作为 argv 元素；拒绝换行、NUL、选项前缀和非法 ref；不 `shell=True`。
- mutation allowlist 只含已设计的 fetch、ff-only pull/merge、switch/create、ordinary push、exact delete、local recovery ref、`branch -d`；永久禁止 force/reset/rebase/stash/auto merge/commit。
- 输出对 remote URL/credential-like text 做安全摘要；保存完整 OID，不保存 token。
- remote write/delete 必须有本次 authorization ref；CP2/CP3 approval 只批准设计，不自动转化为执行授权。
- preflight-all 后仍可能竞态；每个 mutation后立即 post-check。已发生 mutation而整体未闭合必须 PARTIAL。

固定错误族：route、dirty、detached、default unknown、invalid/collision、diverged、wrong branch/upstream、non-FF、ref drift、protected、ancestry unproven、authorization missing、remote unavailable、partial、post-verify。

## 9. 非功能设计

| NFR | 设计 | 验收 |
|---|---|---|
| Determinism | repo顺序、plan step、error code、result key稳定；plan digest | 相同 refs/config/input 的 plan/decision/OID一致 |
| Safety | preflight-all、exact ref/OID、argv-only、fail closed、no force | 禁止命令执行数0；未证明 remote delete数0 |
| Auditability | append-only attempt、before/after OID、executed/skipped、authz/recovery refs | per-repo字段覆盖率100% |
| Idempotency | same-OID start/publish/recovery/cleanup NO_CHANGE；冲突不覆盖 | 重试不产生 force/重复错误删除 |
| Portability | stdlib + Git 2.43+；bare repo fixtures | 无 `gb`/GitPython/forge CLI依赖 |
| Dry-run | read-only probe与would-run分离；mutation counter | local/remote refs、HEAD/index/worktree变化0 |
| Compatibility | 旧 `workspace git-status/push`、`cr bootstrap` 默认行为保持 | 既有测试通过；新能力由显式入口触发 |

## 10. Use Case → Architecture Traceability

| UC/REQ | Module/Flow | ADR | Verification focus |
|---|---|---|---|
| UC-GB-001 / REQ-GB-001..004,006 | coordinator + planner + start flow | ADR-GB-001..003 | fresh default、clean/collision/divergence、2/2 base/upstream、dry-run |
| UC-GB-002 / REQ-GB-005,006,010 | publish flow + result | ADR-GB-002..003 | dirty/wrong/non-FF、remote=HEAD、partial |
| UC-GB-003 / REQ-GB-007..010 | proof/cleanup/recovery | ADR-GB-002..004 | ancestor/non-ancestor/squash/drift/protected/auto-delete |
| REQ-GB-C001..003 / NF001..002 | native adapter + authz/result | ADR-GB-001,003,004 | dependency absence、argv injection、risk ceiling |

## 11. 架构场景模拟

| ID | 场景 | 设计结果 | 路由 |
|---|---|---|---|
| SIM-GB-01 | 两仓 clean main均落后remote一个commit | PASS | fetch+ff-only，branch bases分别等于remote tips |
| SIM-GB-02 | artifact dirty或project detached | PASS | global precheck BLOCKED，new local/remote branch=0 |
| SIM-GB-03 | project start成功、artifact push网络失败 | PASS_WITH_PARTIAL | project事实保留，artifact retry；overall非PASS |
| SIM-GB-04 | publish前artifact有未提交过程文件 | PASS | 两仓push均不执行，要求显式commit |
| SIM-GB-05 | ancestry-preserving external merge | PASS | proof→recovery refs→remote deletes→local branch -d |
| SIM-GB-06 | squash merge或remote tip漂移 | PASS | ancestry/ref-drift BLOCKED，remote delete=0 |
| SIM-GB-07 | forge已自动删remote branch但local/recorded tip存在 | PASS | 仍做ancestry；证明后只做剩余local cleanup |
| SIM-GB-08 | remote/local/result tip全缺 | PASS | absence不当proof，要求未来forge receipt/follow-up |
| SIM-GB-09 | dry-run输入合法 | PASS | 完整planned steps；fetch/switch/push/update-ref/delete执行数0 |

以上仅为设计推演，不是 CP7 执行证据。

## 12. HLD 拆分评估

本 CR 只有 ST-GB-001..003、三 Feature、三 Wave；三者共享 branch identity、Git adapter、result schema、protected policy和同一最终 release gate。独立拆 HLD会复制安全不变量并制造交叉版本风险，因此保留单 HLD。

切换条件：若未来 forge receipt、stacked branches、跨仓事务服务、独立 owner/release/rollback 或 recovery retention成为正式范围，必须拆独立 CR/HLD，不在本 HLD追加隐藏平台能力。

## 13. 分阶段落地与粗估

| Wave | Story | 产出 | 前置 | 粗估 |
|---|---|---|---|---:|
| W1 | ST-GB-001 | shared types/policy/adapter + governed start/open + dry-run | CP3、全量CP5 | 3-4 dev-days |
| W2 | ST-GB-002 | committed-ref publish + non-FF/partial/idempotency | W1 | 2 dev-days |
| W3 | ST-GB-003 | proof/recovery/cleanup + docs/full regression | W2、外部 merge fixtures | 3-4 dev-days |

Story=3、Wave=3，与 Story Map/release slices一一对应。CP4/CP5 再确定 Feature Design Matrix、LLD policy、文件 owner与测试任务；CP5前不实现。

## 14. 风险、回退与发布上限

| 风险 | 状态 | 缓解 | 回退/停止 |
|---|---|---|---|
| 远端误删 | OPEN | exact identity/tip/protected/ancestry + recovery ref + delete post-check | 任一不明即BLOCKED；不force |
| 两仓partial | OPEN | preflight-all、deterministic monotonic steps、per-repo result/resume | 保留成功事实；停止剩余destructive step |
| squash/rebase不可证明 | OPEN | Git DAG only positive proof | 不删，转未来forge receipt CR |
| start/bootstrap顺序失败 | OPEN | explicit coordinator复用writer；旧bootstrap无隐式remote writes | local partial保留，result/terminal JSON给恢复 |
| recovery ref老化 | OPEN | local-only namespace、冲突fail closed、文档披露 | retention另立CR，不在finish自动清理 |
| 无独立QA | OPEN | deterministic fixtures +诚实fallback | CP7≤PASS_WITH_RISK，CP8≤READY_WITH_RISK |

若用户要求无风险 READY，则 CP7 前必须解除“不使用子 Agent”限制并取得可验证独立 QA；即使该风险关闭，remote/forge/真实双仓试运行等风险仍独立计算，不自动变 READY。

## 15. ADR 候选

- ADR-GB-001：原生 Git service + 显式 CR Start Coordinator；旧 bootstrap 不隐式远端写。
- ADR-GB-002：跨仓 preflight-all、单调执行、逐仓结果与幂等 resume；无自动补偿/原子声明。
- ADR-GB-003：Git refs 作为 truth，append-only attempt + existing ledgers作为 evidence；无 branch state DB。
- ADR-GB-004：ancestry-only cleanup、local recovery ref与叠加式验证/发布结论上限。

## 16. Gotchas

1. “先创建 formal CR 再开分支”会使 artifact tree dirty；设计必须把两者放入显式 coordinator 的有序启动流程，而不是放宽 clean gate。
2. `push --dry-run`、remote branch absence、CR closed、PR 文本都不是 merge proof。
3. local recovery ref提高可恢复性，但不是 merge证明、远端备份或可自动发布 tag。
4. 两仓按顺序执行不意味着 transaction；自动补偿远端 ref可能比partial更危险。
5. 当前 CR 的 CP2/CP3批准不授权用真实远端进行 dogfood；实现后真实试运行需单独显式授权并记录结果。

---
status: confirmed
version: "1.2"
created_at: "2026-07-16"
owner: "host-orchestrator-inline / meta-se"
cr_ref: "CR-050"
complexity: complex
decision_burden: high
review_gate: "CP3-R3"
source_requirements: "process/docs/product/REQUIREMENTS.md"
supersedes: "process/archive/CR-050/design/CR050-GIT-BRANCH-HLD.v1.0.1.md"
---

# CR-050 Governed Git Branch Lifecycle HLD

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-16 | host-orchestrator-inline / meta-se | 设计原生Git双仓open/publish/verify-only finish。 |
| 1.0.1 | 2026-07-16 | host-orchestrator | 用户新增显式merge后标记changes-requested；原正文按hash归档。 |
| 1.1 | 2026-07-16 | host-orchestrator-inline / meta-se | 落实CP2 R2：增加独立paired ff-only merge、操作级default-write authorization、artifact→project PARTIAL/恢复、current 2/2 merge→finish reproof；同步四Story/四Wave与五项CP3 R2决策。 |
| 1.2 | 2026-07-16 | host-orchestrator-inline / meta-se | CP3 R3精确化§3/5/6/7/9/12/13：把artifact-first定义为受约束的治理预写，新增2/2聚合投影门与PARTIAL不可见性不变量；若实现无法保证该门则回退project-first并重开CP2/CP3；核实UC/ST追溯并冻结01→02→04→03顺序。 |

## 1. 问题与成功标准

现有`meta_flow.workspace.git_sync`只能汇总两仓状态并顺序push，缺少CR identity、remote default、branch collision、published-tip、default fast-forward eligibility、merge授权、protected policy、finish proof与安全删除。手工Git流程会造成陈旧main开发、源码/过程证据不在同名branch、单仓成功冒充双仓完成、默认分支写入越权，或在证明不足时删branch。

目标生命周期：

```text
open -> publish -> explicit merge -> finish
```

| ID | 可量化成功标准 |
|---|---|
| SC-GB-01 | open成功时2/2 branch base等于各自fresh remote default exact OID，2/2 upstream建立；任一全局preflight失败时新local/remote ref=0。 |
| SC-GB-02 | publish成功时2/2 remote CR OID等于调用时local HEAD；隐式stage/commit/amend次数=0。 |
| SC-GB-03 | merge写入前2/2 eligibility+authz PASS；执行顺序100%为artifact→project；overall PASS时2/2 remote default等于exact published tip。 |
| SC-GB-04 | merge commit/rebase/squash/force/auto-conflict-resolution执行次数=0；branch-protection拒绝不被绕过。 |
| SC-GB-05 | artifact成功/project失败时overall=PARTIAL、paired_projection_advanced=false、cr_closed=false、2/2 CR branches retained、finish_allowed=false、automatic_rollback_count=0，逐仓terminal/resume字段覆盖率100%。 |
| SC-GB-06 | finish只在current matching 2/2 merge PASS后启动，且重新fetch验证exact identity/tip/ancestry；证明不足时remote/local delete=0。 |
| SC-GB-07 | open/publish/merge/finish dry-run的local/remote ref、HEAD、index、worktree mutation=0，计划required repo覆盖率100%。 |
| SC-GB-08 | TC-GB-001..017全部有Story/requirement/test映射；bare-remote正负/partial/resume fixture覆盖17/17场景。 |

## 2. 已批准产品边界与不授权

CP2 R2已批准：merge为独立动作；两仓先全部preflight，按artifact→project普通fast-forward；default write需要本次操作独立授权；PARTIAL保留成功事实与两仓CR branch，不自动补偿；publish/finish不隐式merge。

CP2/CP3批准均不授权当前工作树执行源码修改、stage/commit/push、真实branch create/delete、default update、真实merge、forge API、凭据或force/history rewrite。本轮不触碰`process/quant-lab/**`与prelink backup。

## 3. Architecture Gray Areas

| ID | 问题 | 推荐 | 备选 | 切换条件 | CP3 R3 |
|---|---|---|---|---|---|
| AGA-GB-R2-01 | 四阶段能力放在何处，是否新建orchestrator | 扩展同一typed branch lifecycle service；CLI/Host显式调用；open coordinator复用bootstrap；旧命令不增隐式副作用 | 外部脚本；新release orchestrator；finish隐式merge | 若后续forge/queue有独立owner/credential/release cadence，另立adapter CR；本轮不新建总编排 | DQ-01 |
| AGA-GB-R2-02 | 两仓default update如何面对无事务与中途失败 | artifact→project作为受约束治理预写：artifact先承载已批准治理基线；只有2/2 post-check后聚合投影才可写PASS/finish资格；PARTIAL只能写attempt并保持CR active | project-first；并行；自动回滚 | 若CP4/CP5无法证明`PARTIAL => paired_projection=false`，切project-first并重开CP2/CP3；只有可验证跨仓事务时才可宣称原子 | DQ-02 |
| AGA-GB-R2-03 | 如何避免普通Git push绕过授权/保护或写错OID | operation-specific typed authz绑定repo/ref/expected OID；exact ordinary ref update；remote拒绝原样BLOCKED/PARTIAL | CP gate长期授权；force-with-lease；直接接forge API | 远端强制PR/queue时native路径fail closed并启动forge-adapter CR | DQ-03 |
| AGA-GB-R2-04 | merge evidence与finish删除门如何衔接 | append-only attempt；只有current 2/2 PASS使finish eligible；finish仍fresh reproof+独立delete authz+recovery ref | merge成功直接删；新branch DB；仅stdout | future trusted receipt可作为额外proof source，但不得跳过identity/authz/protected校验 | DQ-04 |
| AGA-GB-R2-05 | no-subagent下能宣称何种验证/发布结论 | inline可做确定fixture；CP7≤PASS_WITH_RISK、CP8≤READY_WITH_RISK；该上限与real-default-write/forge receipt风险叠加独立 | CP7前解除限制取得独立QA；要求READY则阻断 | 真实独立QA只解除独立性风险，不自动解除其他风险 | DQ-05 |

### Advisor table-first inline synthesis

用户延续“不使用子Agent”；本表是Host inline synthesis，不是`meta-se-critical` runtime receipt或独立评审。

| Lane | 判断 | 形成的约束 |
|---|---|---|
| product | 四动作是一个用户旅程；merge必须显式且不能藏在finish | 四Story/四slice，01→02→04→03 |
| architecture | refs/state/result各自单写；native service足够，无需新orchestrator/DB | typed planner/executor + existing writers/ledgers |
| security | default write与delete是不同高风险authority；远端策略是约束不是错误需绕过 | operation/OID绑定authz、ordinary push、no-force、fail closed |
| quality | bare remote可证明算法/命令边界，但不证明真实保护平台与独立agent | negative command spy、partial fault injection、风险上限 |
| operations | preflight不能消除竞态；恢复必须重观测，不能反向重写成功default | deterministic order、PARTIAL terminal、resume attempt |

## 4. 候选方案

### Option A：扩展原生Git lifecycle service（推荐）

在现有workspace route/Git subprocess边界上新增typed planner/executor与显式open/publish/merge/finish handlers；open coordinator复用bootstrap；attempt接既有checks/ledgers。merge对remote default执行exact ordinary fast-forward update，不在本地制造merge commit。

优点：无新runtime依赖；权限、OID、命令与partial可fixture验证；兼容既有bootstrap/workspace push。代价：需维护typed authz、跨仓PARTIAL与resume。

### Option B：立即接forge PR/merge queue API

优点：适配受保护分支/review/receipt。缺点：引入平台差异、credential、API schema、最小权限、receipt真实性与更多失败面；当前没有授权/契约。

### Option C：安装Git Town/名义`gb`工具

优点：可能提供stacked branch/ship。缺点：`gb`无唯一可移植语义，额外version/config/sync策略，仍不能替代双仓CR/evidence/authz contract。

### Option D：文档化手工命令

实现少，但无法稳定产出机器result、证明no-force/zero-dry-run、阻断partial误报或安全恢复，不能满足目标。

| 维度（5最好） | A | B | C | D |
|---|---:|---:|---:|---:|
| 无新增必需依赖 | 5 | 2 | 1 | 5 |
| 双仓/CR/evidence集成 | 5 | 3 | 2 | 1 |
| 权限与失败可测试 | 5 | 4 | 2 | 1 |
| 受保护平台能力 | 2 | 5 | 3 | 1 |
| MVP维护成本 | 3 | 1 | 2 | 4 |
| 合计 | **20** | 15 | 10 | 12 |

选择A；出现强制PR/queue场景时不是在A内静默降级，而是启动B的独立CR。

## 5. 推荐架构

```text
User/Host explicit open|publish|merge|finish
                    |
                    v
CLI + CR lifecycle coordinator ----------> existing CR/state/current writers
                    |
                    v
Typed Branch Lifecycle Planner/Executor --> append-only attempt result
          |                 |                         |
          v                 v                         v
 Workspace Route   Authz Validator             existing ledgers
          \                 /
           v               v
             Native Git Adapter
                    |
        project/artifact repos + remotes
```

- Git refs：当前ref truth。
- CR/state：workflow truth。
- result/ledger：观察与执行evidence。
- authorization：本次动作permission evidence；不持有ref truth。
- paired merge projection：只读聚合2/2逐仓终态；仅2/2 PASS/NO_CHANGE时允许现有CR/state/current writer投影`merge=PASS`与finish eligibility。artifact单仓成功只能产生append-only PARTIAL attempt，不能让过程真相领先于源码事实。

## 6. 模块与集成契约

| 模块 | 职责 | 输入→输出 | 失败/降级 | 同步范围 |
|---|---|---|---|---|
| CLI/Host adapter | 四个独立command、dry-run、JSON/output/authz ref、退出码 | typed args→human+machine result | 参数/授权缺失不调用mutation | `meta_flow/cli.py`、docs、Host rules |
| CR Open Coordinator | no-write intent check→local branch prep→bootstrap→tracking push | CR intent→bootstrap refs+attempt | PARTIAL保留事实；旧bootstrap不变 | `cr_lifecycle.py` |
| Planner/Policy | observations+authz→deterministic immutable plan/digest | typed plan，无shell | ambiguity/non-FF/protected→BLOCKED | shared types/policy |
| Executor | 按plan顺序执行allowlisted argv与post-check | plan→per-repo outcome | mutation后失败必须PARTIAL | 不写CR/state/ledger |
| Authz Validator | 校验operation/repo/ref/OID/issuer/有效性 | authz→allow/deny reason | 不匹配fail closed | merge/delete入口 |
| Native Git Adapter | probe、ordinary exact ref update、有界输出/redaction | argv/cwd→typed command result | timeout/nonzero保留remote拒绝语义 | 扩展`workspace.git_sync` |
| Result/Ledger Adapter | schema校验、append-only落盘与correlation | outcome→result/ref/event | invalid result不能推进 | existing checks/RUN/CR ledgers |
| Paired Projection Gate | 聚合2/2 repo outcome并控制workflow投影 | validated attempt→PASS或PARTIAL projection decision | 任一仓非PASS/NO_CHANGE时不得写merge PASS、finish eligibility或CR close | existing CR/state/current writers；不持有Git truth |

## 7. 关键流程与失败路径

### 7.1 Open

无写校验intent/route/branch/authz→两仓read-only precheck→actual模式fetch/prune与default ff-only→从exact remote default创建local branches→复用bootstrap→逐仓`push -u`和OID/upstream核验。precheck失败mutation=0；local准备后失败为PARTIAL，不自动删除。

### 7.2 Publish

两仓fresh observe→clean/identity/upstream/non-FF全部通过→ordinary exact branch push→逐仓remote OID核验。只发布调用前commit；dirty/uncommitted直接BLOCKED。后一仓失败保留前仓success并给resume。

### 7.3 Explicit paired fast-forward merge

1. 读取matching publish result，但以fresh remote CR/default refs校验current事实。
2. 对2/2仓校验branch identity、exact published tip、`default_tip is-ancestor published_tip`、typed authz与remote/default目标；任一失败时default mutation=0。
3. dry-run只输出有序plan；actual按artifact→project执行普通exact remote ref update，不在本地checkout/merge，且不含force/merge commit/rebase；CP5冻结为`git push --porcelain <remote> <published_oid>:refs/heads/<default>`等价argv。
4. 每仓立即查询remote default；必须等于expected published tip。
5. 2/2 PASS/NO_CHANGE→overall PASS，并由Paired Projection Gate一次性开放workflow投影；artifact成功/project失败→PARTIAL，只追加attempt，`paired_projection_advanced=false`、CR保持active、保留2/2 CR branch、finish=false、no rollback。
6. resume生成新attempt并fresh observe；已成功仓事实仍匹配才NO_CHANGE，随后只继续未完成仓。

artifact-first不是“artifact更重要”，而是受约束的治理预写：先让低风险过程仓保存已批准的治理/证据基线，再写高风险源码默认分支；这样不会出现源码已进入default但canonical治理证据仍缺失的窗口。其安全前提是终态workflow投影严格晚于2/2 post-check；若实现层无法隔离artifact单仓attempt与paired projection，顺序必须改为project-first并回退CP2/CP3重新审批。

### 7.4 Proof-gated finish

入口要求current matching 2/2 merge PASS和独立delete authz。随后重新fetch/observe、解析known tip、检查protected/ref drift、证明tip是fresh default祖先；2/2 proof通过后建立/reuse recovery refs，再完成两仓remote delete，最后`branch -d` local refs。merge result、CR closed或remote branch absent都不能跳过reproof。

## 8. 安全、可靠性与可观测性

- 所有Git输入作为argv元素；拒绝NUL/换行/选项前缀/非法ref；禁止`shell=True`。
- allowlist只覆盖已设计probe、fetch、ff-only default refresh、switch/create、ordinary exact push、local recovery ref、exact delete、`branch -d`；禁止force/reset/rebase/stash/merge-commit/auto-commit。
- authz必须绑定当前operation/repo/remote/ref/expected OID；CP approval不自动映射为authz。
- remote URL/stderr做安全摘要，不保存token；完整保存非敏感OID与remote策略错误码。
- attempt append-only；每仓terminal、before/after OID、executed/skipped、mutation flag、resume route字段覆盖率100%。
- timeout/remote loss发生在写后时，必须fresh post-query；无法判定则PARTIAL/FAILED，不推断成功或回滚。

## 9. 场景模拟

| ID | Given/When | 判定 | 关键证据 |
|---|---|---|---|
| SIM-GB-R2-01 | clean两仓behind remote，执行open | PASS | 2/2 base=remote default、upstream exact |
| SIM-GB-R2-02 | dirty/detached/collision | BLOCKED | new refs=0 |
| SIM-GB-R2-03 | publish existing commits | PASS | 2/2 remote CR=local HEAD；commit commands=0 |
| SIM-GB-R2-04 | merge前project non-FF | BLOCKED | preflight-all使2/2 default writes=0 |
| SIM-GB-R2-05 | 2/2 eligible+authz | PASS | sequence artifact→project；defaults=published tips |
| SIM-GB-R2-06 | artifact PASS，project policy拒绝 | PARTIAL | attempt追加；paired projection/CR close=0；branches retained、finish=false、rollback=0 |
| SIM-GB-R2-07 | resume且artifact fact仍匹配 | PASS/NO_CHANGE | fresh observations；只继续project |
| SIM-GB-R2-08 | merge dry-run | PLANNED | local/remote mutations=0 |
| SIM-GB-R2-09 | merge PARTIAL后调用finish | BLOCKED | delete=0 |
| SIM-GB-R2-10 | 2/2 merge PASS后default/CR tip漂移 | BLOCKED | finish reproof失败；delete=0 |
| SIM-GB-R2-11 | ancestry-preserving current事实 | PASS | proof→recovery→remote delete→local delete |
| SIM-GB-R2-12 | squash/protected/unknown tip | BLOCKED | target refs retained |

这些是设计模拟，不是CP7执行证据。

## 10. Traceability

| Use Case | Requirements | Architecture | Stories/Tests |
|---|---|---|---|
| UC-GB-001 | REQ-GB-001..004,006 | open coordinator、route、planner、Git adapter | ST-GB-001；TC-GB-001..003,009..010 |
| UC-GB-002 | REQ-GB-005..006,010 | publish handler、attempt/result | ST-GB-002；TC-GB-004..005,009..010 |
| UC-GB-004 | REQ-GB-011..014,C004,NF003 | authz validator、paired merge choreography | ST-GB-004；TC-GB-012..017 |
| UC-GB-003 | REQ-GB-007..010,014 | finish gate、proof、recovery ref | ST-GB-003；TC-GB-006..011,015,017 |

约束REQ-GB-C001..004和NF001..003由共享policy/adapter/result fixtures覆盖。

## 11. Story/Wave与拆分评估

| Wave | Story | 交付 | 前置 |
|---:|---|---|---|
| 1 | ST-GB-001 | shared typed contracts、paired open、bootstrap integration | CP5全量批准 |
| 2 | ST-GB-002 | committed-ref publish与matching evidence | Wave 1 |
| 3 | ST-GB-004 | explicit paired ff merge、authz、PARTIAL/resume | Wave 2 |
| 4 | ST-GB-003 | current merge gate、fresh reproof与cleanup | Wave 3 |

Story数=4，Wave数=4，一一对应。虽然超过三个operation，但它们共享一个用户旅程、公共types/adapter/result、安全策略和release gate，且无独立owner/rollback，因此一个HLD更能防契约漂移。出现forge凭据/PR queue/独立release时触发子CR/HLD。

## 12. 风险与结论上限

| 风险 | 状态 | 控制/路由 |
|---|---|---|
| default write被保护策略拒绝 | OPEN | native普通push fail closed；转future forge adapter，不绕过 |
| artifact成功/project失败 | ACCEPTED-CP3-R3 | 受约束治理预写；PARTIAL只追加attempt，paired projection/CR close=0；retain 2/2 branches、block finish、fresh resume、no rollback |
| ref竞态/错误OID | OPEN | preflight+expected OID+post-check；无法判定不报PASS |
| squash/rebase无法ancestry证明 | OPEN | finish fail closed；未来trusted receipt adapter |
| no independent QA | ACCEPTANCE-PENDING-CP3 | CP7≤PASS_WITH_RISK、CP8≤READY_WITH_RISK |
| 真实平台pilot/default write未授权 | OPEN | bare fixture不冒充真实receipt；实际操作另行授权 |

风险上限是叠加式：独立QA证据只能解除独立性风险；branch protection、forge receipt、真实pilot等风险仍单独计算。

## 13. Gotchas

1. `fast-forward-only`描述的是DAG关系和exact update结果，不授权`git merge`创建任何commit。
2. `--force-with-lease`仍属于force，不能以“更安全”为由进入allowlist。
3. 全仓preflight通过后仍有竞态；因此PARTIAL是正式终态，不是日志warning。
4. old merge PASS不能长期授权finish；fresh observation与current matching attempt缺一不可。
5. authz ref存在不代表内容匹配，必须校验operation、repo、remote/ref与OID。
6. no-subagent inline设计不能宣称独立架构/QA；CP3人工批准提供架构决策，但不提供runtime attestation。
7. artifact仓出现一条default更新事实不等于配对merge完成；任何读取方若用单仓ref直接投影overall PASS，均违反2/2聚合门并必须在CP4/CP5阻断。

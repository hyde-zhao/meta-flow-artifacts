---
checkpoint_id: "CP3-CR050-GIT-BRANCH-HLD-REVIEW-R2"
checkpoint_name: "CR-050 Explicit Paired Fast-forward Merge Architecture Review R2"
type: "auto_then_manual"
status: "awaiting-user"
owner: "host-orchestrator"
created_at: "2026-07-16T14:35:24Z"
reviewed_by: null
reviewed_at: null
supersedes_checkpoint_ref: "process/checkpoints/CP3-CR050-GIT-BRANCH-HLD-REVIEW.md"
auto_check_result: "process/checks/CP3-CR050-GIT-BRANCH-HLD-CONSISTENCY-R2.result.json"
context_ref: "process/context/CP3-CR050-GIT-BRANCH-DESIGN-CONTEXT-R2.yaml"
decision_brief_profile: "compact"
target:
  phase: "solution-design"
  artifacts:
    - "process/docs/design/CR050-GIT-BRANCH-BLUEPRINT.md"
    - "process/docs/design/CR050-GIT-BRANCH-DOMAIN-MAP.md"
    - "process/docs/design/CR050-GIT-BRANCH-DEPENDENCY-MAP.md"
    - "process/docs/design/CR050-GIT-BRANCH-HLD.md"
    - "process/docs/design/CR050-GIT-BRANCH-ARCHITECTURE-DECISION.md"
---

# CP3 R2 — CR-050 显式双仓 Fast-forward Merge 架构评审

## 自动预检摘要

| 预检 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `CP3-CR050-GIT-BRANCH-HLD-CONSISTENCY-R2.result.json` | PASS | 0 | 9项全部通过；R2 supersedes旧CP3 result |
| `CP3-CR050-DISCUSSION-CHECKPOINT-R2.json` | ready-for-host-broker | 0 | 5个Architecture Gray Areas均有推荐、备选和切换条件 |
| `CP3-CR050-GIT-BRANCH-DESIGN-CONTEXT-R2.yaml` | ready-for-human-gate | 0 | compact capsule；12条read-expansion refs |
| HLD budget | PASS | 0 | 16566 bytes < 20480 bytes |

## Decision Brief

### 审批者摘要

| 字段 | 内容 |
|---|---|
| 本次确认服务的整体目标 | 冻结`open → publish → explicit merge → finish`四阶段的模块、权限、OID、双仓PARTIAL、恢复和删除证明边界。 |
| 相比CP3 v1.0 | v1.0只设计外部merge后的verify-only finish；本R2增加独立merge、artifact→project exact ff、default-write authz和2/2 merge→finish gate。旧设计/result/checkpoint已保留并由R2显式supersede。 |
| 推荐动作 | `approve`接受CP3-R2-DQ-01..05与ADR-GB-R2-001..005。 |
| approve 后会发生什么 | 自动进入Story/Feature planning，按四Story/四Wave生成CP4自动预检、全量设计证据队列并停在CP5人工门；CP5批准前不改源码。 |
| approve 不授权什么 | 源码实现、stage/commit/push、真实branch create/delete、真实default update/merge、force/history rewrite、forge API、凭据/runtime/SaaS/production write、quant-lab或prelink backup修改。 |
| 不确认会阻塞什么 | 阻塞CP4、Story设计、LLD和实现；现有Git行为不改变。 |

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule | `process/context/CP3-CR050-GIT-BRANCH-DESIGN-CONTEXT-R2.yaml` |
| read_profile | compact |
| 默认读取策略 | state/current/CP2 R2/HLD/ADR为must-read；其余只按allowed/read-if-needed消费。 |
| 全文档读取 | 本轮12次deep-review扩展已记录到READ-EXPANSION-LEDGER；archive仍deny-default。 |
| 范围 | UC-GB-001..004；REQ-GB-001..014/C001..004/NF001..003；TC-GB-001..017；ST-GB-001..004 |
| 推荐架构 | 同一typed native Git lifecycle service的四个显式handler；不新建orchestrator或branch DB |
| merge契约 | all-repo preflight；artifact→project；exact ordinary fast-forward；operation/OID-bound authz；policy拒绝fail closed |
| partial契约 | retain 2/2 CR branches、block finish、no automatic rollback、fresh-observation resume |
| finish契约 | current matching 2/2 merge PASS只建立eligibility；仍需独立delete authz与fresh reproof |
| 调度披露 | 用户要求no-subagent；Host inline产出，不是custom-agent/runtime/独立评审receipt |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| state | state/current/route plan | scanned | 1 | 0 | CP2 R2已批准；当前必停CP3 R2 |
| inline handoff | `CR050-CP2-R2-CP3-R2-META-SE.md` | scanned | 5 | 5 | 五个架构灰区全部进入DQ |
| auto checks | CP3 R2 result/context | scanned | 0 | 0 | PASS、blocker=0、waiver=0 |
| discussion | CP3 R2 discussion/checkpoint | scanned | 5 | 5 | AGA-R2-01..05→DQ-01..05 |
| product baseline | 八份`process/docs/product/**` | scanned | 4 | 0 | CP2-R2-DQ-01..04已批准，不重复提问 |
| formal design | Blueprint/Domain/Dependency/HLD/ADR v1.1 | scanned | 5 | 5 | service、PARTIAL、authz、finish gate、ceiling |
| repository evidence | existing git adapter/CLI/bare tests | scanned | 3 | 1 | 支持native扩展；具体argv留CP5冻结 |
| user input | merge、CP2 R2 approve、no-subagent | scanned | 3 | 2 | merge边界已消费；no-subagent形成风险决策 |

### 决策分层

| 分类 | 数量 | 处理 |
|---|---:|---|
| 必须用户决策 | 5 | CP3-R2-DQ-01..05 |
| 高风险策略确认 | 4 | default write、paired PARTIAL、finish deletion、verification ceiling |
| agent 默认处理 | 7 | 函数名、result字段细节、timeout、slug细节、fixture名、错误文案、help排版 |
| Deferred | 3 | forge/merge-queue adapter、stacked branch工具、recovery retention |
| 仅审计记录 | 5 | inline fallback、HLD budget、12项模拟、source clean、no real mutation |
| 明确禁止 | 8 | implicit merge、merge commit、rebase、squash、force、auto conflict、auto commit、policy bypass |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| CP3-R2-DQ-01 | architecture | 四阶段是否扩展同一native service并保持旧入口无隐式副作用？ | Option A：typed service四handler；open coordinator复用bootstrap；无新orchestrator/DB。 | A：Host显式多步；B：future forge adapter；C：新总编排器。 | 推荐复用现有Git/CR/result边界且可测试；新总编排会产生第四truth。 | 共享typed contract增加实现协调；旧入口行为必须保持。 | coordinator无法复用writer时退显式多步但保持同一intent/result；出现独立forge owner/credential/release时拆adapter CR。 |
| CP3-R2-DQ-02 | risk_acceptance | 是否接受artifact→project非原子PARTIAL与不自动补偿？ | preflight-all；artifact→project；成功fact不撤销；PARTIAL保留2/2 branches、block finish、fresh resume。 | project-first；并行；自动reset/delete/force补偿。 | 推荐最可审计且不以二次危险写放大损害。 | main可能短暂不一致，需要operator resume。 | 只有可验证跨仓transaction存在才重评；当前禁止自动补偿。 |
| CP3-R2-DQ-03 | security | native default write如何约束授权、OID与保护策略？ | 每次merge的typed authz绑定operation/repo/ref/OID；普通exact ff；remote拒绝BLOCKED/PARTIAL。 | CP gate blanket授权；force-with-lease；静默切forge API。 | 推荐权限最小且保留远端策略。 | 受保护仓可能无法由native路径完成。 | 强制PR/queue时fail closed，另立forge adapter CR并定义credential/receipt。 |
| CP3-R2-DQ-04 | architecture | merge attempt如何与finish删除门衔接？ | append-only attempt；current 2/2 PASS仅使finish eligible；finish仍要独立delete authz、fresh reproof和recovery ref。 | merge后直接删；只stdout；新branch DB；CR closed即proof。 | 推荐owner清晰且抵抗evidence过时。 | 重复fetch/proof增加少量执行成本。 | future trusted receipt可扩展proof source，但不能跳过identity/authz/protected检查。 |
| CP3-R2-DQ-05 | risk_acceptance | no-subagent时接受什么CP7/CP8上限？ | CP7≤PASS_WITH_RISK、CP8≤READY_WITH_RISK；与branch protection/forge receipt/real pilot风险叠加独立。 | CP7前取得独立QA；若要求READY则阻断。 | 推荐不把fixture冒充独立/平台证明。 | 本CR不能宣称无风险READY。 | 后续证据只解除其对应风险维度。 |

| 字段 | 内容 |
|---|---|
| 推荐决策 | 批准CP3-R2-DQ-01..05与ADR-GB-R2-001..005 |
| 备选回复 | `修改: CP3-R2-DQ-xx <具体要求>`；或`reject`保持solution-design |
| 风险接受 | DQ-02跨仓PARTIAL；DQ-05独立性结论上限 |
| 权限边界 | DQ-03只批准授权架构，不是本次真实default write授权 |
| 用户需决策事项 | `CP3-R2-DQ-01`、`CP3-R2-DQ-02`、`CP3-R2-DQ-03`、`CP3-R2-DQ-04`、`CP3-R2-DQ-05` |

### 架构摘要

- 真相：Git refs=ref facts；CR/state=workflow facts；attempt/ledger=evidence；typed authz=本次permission。
- 依赖：Host/CLI→coordinator→planner/executor→route/authz/native Git；result接existing ledgers，下层不反写state。
- 四Wave：ST-GB-001 open→ST-GB-002 publish→ST-GB-004 merge→ST-GB-003 finish。
- 十二项设计模拟覆盖正向、preflight、policy拒绝、PARTIAL/resume、dry-run、finish reproof；不是CP7 evidence。

## Entry Criteria

| 条目 | 状态 | 证据 |
|---|---|---|
| CP2 R2 approved | PASS | checkpoint+gate ledger；四项推荐已批准 |
| 产品基线ready for design | PASS | UC/REQ/TC/ST四阶段基线 |
| v1.0 changes-requested证据保留 | PASS | old checkpoint/result+`process/archive/CR-050/design/**v1.0.1.md` |
| v1.1五份设计完整 | PASS | canonical design refs |
| HLD预算 | PASS | 16566<20480 bytes |
| CP3 R2 auto result/context | PASS | R2 result/context；blocker=0 |

## Checklist

| # | 检查项 | 当前结果 | 证据/待审查 |
|---:|---|---|---|
| 1 | Option A、替代方案与切换条件 | pending-human | DQ-01 |
| 2 | 四动作显式且无新总truth/orchestrator | pending-human | DQ-01、ADR-R2-001 |
| 3 | preflight-all、artifact→project、PARTIAL/no rollback | pending-human | DQ-02、ADR-R2-002 |
| 4 | typed authz、exact ff、remote policy fail closed | pending-human | DQ-03、ADR-R2-003 |
| 5 | current 2/2 merge gate、fresh finish reproof/recovery | pending-human | DQ-04、ADR-R2-004 |
| 6 | four Story/four Wave与traceability一致 | PASS | HLD §10-11 |
| 7 | argv-only/no-force/no-auto-commit/no-implicit-merge | PASS | HLD §8、Domain invariants |
| 8 | inline fallback与叠加式结论上限诚实 | pending-human | DQ-05、ADR-R2-005 |
| 9 | source/真实Git mutation=0 | PASS | source repo clean；仅process artifacts变化 |

## Exit Criteria

| 条目 | 当前结果 | 证据 |
|---|---|---|
| 五项架构/风险决策关闭 | pending-human | CP3-R2-DQ-01..05 |
| 五份设计可作为CP4输入 | PASS | CP3 R2 result |
| Story/Feature/DAG/LLD前置明确 | PASS | HLD/Dependency Map |
| CP5前实现与真实Git写保持0 | PASS | authorization boundary |

## Deliverables

| 交付物 | 路径 | 当前结果 |
|---|---|---|
| Blueprint/Domain/Dependency | `process/docs/design/CR050-GIT-BRANCH-{BLUEPRINT,DOMAIN-MAP,DEPENDENCY-MAP}.md` | v1.1 ready |
| HLD/ADR | canonical CR050 design paths | v1.1 ready/proposed |
| Discussion | `process/discussions/CP3-CR050-GIT-BRANCH-HLD-DISCUSSION-LOG.md` | R2 complete |
| Auto result | `process/checks/CP3-CR050-GIT-BRANCH-HLD-CONSISTENCY-R2.result.json` | PASS |
| Context | `process/context/CP3-CR050-GIT-BRANCH-DESIGN-CONTEXT-R2.yaml` | ready-for-human-gate |

## 人工审查结果

- 结论：`pending`
- 审查人：待用户确认
- 审查时间：待确认
- 修改意见：待确认
- 风险接受项：CP3-R2-DQ-02、CP3-R2-DQ-05待确认；CP3-R2-DQ-03不构成真实default write授权。

## 可接受回复

- `approve`
- `修改: CP3-R2-DQ-xx <具体修改点>`
- `reject`

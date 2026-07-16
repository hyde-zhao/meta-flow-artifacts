---
discussion_id: "CP3-CR050-GIT-BRANCH-HLD-DISCUSSION"
change_id: "CR-050"
phase: "solution-design"
status: "archived-changes-requested"
owner: "host-orchestrator-inline / meta-se"
created_at: "2026-07-15T16:18:03Z"
question_broker: "host-orchestrator"
---

# CR-050 CP3 HLD Discussion Log

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-16 | host-orchestrator-inline / meta-se | 记录 CP2批准输入、Architecture Gray Areas、advisor inline synthesis、场景模拟与 CP3 DQ-01..04。 |

## 交互与证据边界

用户已批准 CP2-DQ-01..05，并延续“不使用子 Agent”约束。本阶段为 user-approved inline fallback，不声称 `meta-se-critical` custom agent、指定模型/profile或独立runtime已执行。HLD基于当前源码中的 `workspace git-status/push`、stdlib subprocess boundary和bare-repository tests核对事实；未执行真实 branch create/push/delete。

## 已继承的 CP2 决策

| 决策 | 已批准结论 | CP3 落实 |
|---|---|---|
| CP2-DQ-01 | 原生 Git subprocess，不安装 `gb`/Git Town | ADR-GB-001 |
| CP2-DQ-02 | project/artifact同名CR branch；不宣称原子事务 | ADR-GB-002 |
| CP2-DQ-03 | finish只验证外部merge后清理 | ADR-GB-004 |
| CP2-DQ-04 | squash/rebase无receipt时fail closed | ADR-GB-004 |
| CP2-DQ-05 | publish只push既有commit，不stage/commit/amend | HLD §7.2、安全边界 |

## Architecture Gray Areas

| ID | Advisor输入 | 推荐 | 可执行备选 | 当前状态 |
|---|---|---|---|---|
| AGA-GB-01 | CR-first与artifact clean precheck存在启动顺序冲突 | explicit CR Start Coordinator；local branch prepare后复用bootstrap，再push tracking | Host显式三步；旧bootstrap默认remote write（不推荐） | recommended-pending-CP3 / DQ-01 |
| AGA-GB-02 | 两仓无原子ref transaction | preflight-all、deterministic monotonic execution、observed resume | 自动补偿/force；单仓 | recommended-pending-CP3 / DQ-02 |
| AGA-GB-03 | remote auto-delete需known tip与删除恢复 | append-only attempt + existing ledgers + local-only recovery ref；无branch DB | 只stdout；仅OID；新DB | recommended-pending-CP3 / DQ-03 |
| AGA-GB-04 | inline verification与独立QA缺口 | CP7/CP8风险上限叠加；证明不足阻断 | CP7前取得独立QA；要求READY则阻断 | recommended-pending-CP3 / DQ-04 |

## Advisor table-first inline synthesis

| Lane | 关键判断 | HLD约束 |
|---|---|---|
| product | 三段旅程构成一个生命周期，Story按start/publish/finish拆 | 单HLD、三Story、三Wave |
| architecture | policy/workflow与Git adapter分层，refs/state/result各自归属 | 禁止Git adapter反写CR/state、禁止branch state DB |
| security | 删除与remote write需要exact identity/OID/authz | argv-only、protected policy、no-force、fail closed |
| quality | bare remote可验证算法，不等于真实forge/独立agent证明 | CP7 PASS_WITH_RISK ceiling、negative fixtures |
| operations | partial是预期终态之一，不能用补偿抹去 | per-repo terminal、resume-by-observation |

## 场景推演摘要

| 场景 | 设计判定 | 说明 |
|---|---|---|
| clean repos behind remote | PASS | actual fetch+ff-only后从exact tips建立同名branch |
| dirty/detached/collision | PASS | 全仓precheck阻断，branch mutation=0 |
| second repo push failure | PASS_WITH_PARTIAL | first repo fact保留，overall非PASS，resume remaining |
| dirty publish | PASS | 2/2 remote不变，不隐式commit |
| ancestry-preserving merge | PASS | proof→recovery→remote phase→local branch-d |
| squash/ref drift/protected | PASS | remote delete=0 |
| remote auto-delete | PASS | known tip仍须ancestry；缺tip BLOCKED |
| dry-run | PASS | planned steps完整，mutation count=0 |

这些是设计模拟，不是 CP7执行证据。

## 待 Host Orchestrator 收集的 CP3 决策

1. `CP3-DQ-01`：是否接受 native Git lifecycle + explicit CR Start Coordinator，并保持旧 bootstrap无隐式remote write。
2. `CP3-DQ-02`：是否接受 preflight-all、单调执行、逐仓partial与观察式resume，不做自动补偿/原子声明。
3. `CP3-DQ-03`：是否接受 append-only attempt + existing ledgers，并在finish前建立local-only namespaced recovery ref，不建branch state DB。
4. `CP3-DQ-04`：是否接受 ancestry-only cleanup与当前无独立QA时 `CP7≤PASS_WITH_RISK / CP8≤READY_WITH_RISK`。

## 当前结论

- Blueprint、Domain Map、Dependency Map、HLD、ADR v1.0 草案完成。
- 推荐 Option A；产品/设计 blocker=0；人工决策=4。
- formal CP3 approval未记录；CP3前不拆Story/LLD，不实施源码，不执行真实Git remote mutation。

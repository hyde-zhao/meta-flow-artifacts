---
status: accepted
version: "1.1"
cr_ref: "CR-047"
---

# CR-047 Workflow Truth Domain Map

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.1 | 2026-07-14 | host-orchestrator-inline / meta-se | CP3 R2：定义 Protected Historical Object Manifest、对象身份与双阶段 hash 校验规则。 |
| 1.0 | 2026-07-13 | host-orchestrator-inline / meta-se | 定义 CR-047 真相对象、状态、规则和证据语义。 |

## 术语表

| Term | 定义 | 来源 | 备注 |
|---|---|---|---|
| Machine truth | 可由 checker 直接消费并带 schema/lifecycle 的对象 | UC-WT-001, REQ-WT-001..003 | Markdown 摘要不属于 machine truth |
| Projection | 从 machine truth 重建的发现层视图 | REQ-WT-002 | `CURRENT.json` 可重生，不反向写入 State |
| Canonical route | 以 anchor + relative path 表达的 project/artifact 路由 | UC-WT-002 | 不含设备绝对路径 |
| Blocking finding | 必须使命令非零退出的 ERROR/FAIL/BLOCKED | CP2-DQ-02 | 不得被 warning 计数稀释 |
| Warning | 需计数、分类和披露，但不自动阻断 | CP2-DQ-02 | 可升级为 blocker，不能反向降级安全/证据错误 |
| Cold historical artifact | closed CR 且非当前 default/required read 的原始证据 | CP2-DQ-03 | 需 summary/index/hash/correction 或 archive ref |
| Generated wrapper | 由 installer 从 tracked rule source 生成的本机入口 | CP2-DQ-01 | 不是 clean clone canonical source |
| Protected original | CR-046 已进入正式 evidence/ref graph 且本 CR 只允许读取/引用的历史原件 | CP3-DQ-05 | 身份由 source CR + object type + canonical/provenance ref 确定，不由目录前缀确定 |

## 领域对象

| Object ID | 对象 | Owner Feature / Module | 关键属性 | 状态 | 规则来源 |
|---|---|---|---|---|---|
| OBJ-WT-01 | Workflow State Snapshot | FEAT-WT-01 / `meta_flow.state.current` | active_change, phase, pending_gate, refs, updated_at | active, awaiting-gate, blocked, delivered | REQ-WT-001, REQ-WT-NF001 |
| OBJ-WT-02 | CR Catalog Entry | FEAT-WT-01 / CR lifecycle | id, lifecycle, gate, readiness, formal ref | candidate, active, blocked, closed, cancelled, superseded | REQ-WT-003 |
| OBJ-WT-03 | Current Projection | FEAT-WT-01 / current builder | status, health, state/change/context/checkpoint refs | active, gate, idle, stale_refs | REQ-WT-002 |
| OBJ-WT-04 | Process Route | FEAT-WT-02 / workspace routing | project_name, anchors, relative paths, routing_mode | healthy, missing, conflict, broken | REQ-WT-004, REQ-WT-005 |
| OBJ-WT-05 | Artifact Budget Classification | FEAT-WT-03 | lifecycle class, read class, kind, bytes, limit, evidence refs | active-budgeted, cold-historical, warning, blocking | REQ-WT-006, CP2-DQ-03 |
| OBJ-WT-06 | Quality Finding | FEAT-WT-03 | severity, object, field/path, evidence, route | warning, blocking, resolved, legacy-unavailable | REQ-WT-007, REQ-WT-008, REQ-WT-NF002 |
| OBJ-WT-07 | Rule Source Contract | FEAT-WT-04 | canonical source, generated target, drift check, platform | tracked, generated, stale, missing | REQ-WT-009 |
| OBJ-WT-08 | Cache Artifact | FEAT-WT-04 | path, tracked, package-input, ignored | blocking, warning, absent | REQ-WT-010, REQ-WT-014 |
| OBJ-WT-09 | Run Evidence | FEAT-WT-03/04 | command, result, observed timestamp, recovery status | observed, recovered-legacy-unverified, unavailable | REQ-WT-017 |
| OBJ-WT-10 | Historical Status Projection | FEAT-WT-05 | formal CR ref, Story/CP refs, release ceiling | current, stale, ready-with-risk | REQ-WT-015, REQ-WT-016 |
| OBJ-WT-11 | Protected Historical Object Manifest | FEAT-WT-01/03/05 | source_cr, object_type, canonical_ref, provenance_ref, original_sha256, immutable, allowed_operation | captured, verified-cp6, verified-cp7, violated, indeterminate | REQ-WT-015, REQ-WT-C002, CP3-DQ-05 |

## 状态机

| State Machine ID | 对象 | 合法转换 | 转换条件 | 非法转换处理 |
|---|---|---|---|---|
| SM-WT-01 | Workflow State Snapshot | requirement-clarification → solution-design → story-planning → story-execution → documentation → delivered | route plan + CP gate 证据 | state-transition FAIL，保留上一稳定状态 |
| SM-WT-02 | CR Catalog Entry | candidate → active → closed；active ↔ blocked；active → cancelled/superseded | formal CR + status-sync + ledger | candidate 不得驱动 active state；closed 不得重新 active 而无 recovery CR |
| SM-WT-03 | Current Projection | stale/missing → rebuilt → healthy | State/refs 通过后 current-refresh | 冲突列出 stale_refs，不猜测修复 |
| SM-WT-04 | Quality Finding | observed → warning/blocking → resolved/legacy-unavailable | typed policy + evidence | 缺 evidence 不得写 resolved/PASS |
| SM-WT-05 | Cache Artifact | absent → ignored-warning 或 tracked/package-blocking → absent | Git/package classifier + preflight | 无法分类的 package candidate 按 blocking |

## 业务规则

| Rule ID | 规则 | Owner | 影响场景 | 验证入口 |
|---|---|---|---|---|
| RULE-WT-01 | Active change 必须存在于 formal CR + JSON index，且 lifecycle 未结束 | FEAT-WT-01 | TC-WT-001 | cr-tracking/state consistency |
| RULE-WT-02 | CURRENT 只从 State/refs 重建，关键 scalar 差异数必须为 0 | FEAT-WT-01 | TC-WT-001 | current-refresh + consistency fixture |
| RULE-WT-03 | Canonical CR index 只有 JSON；legacy YAML 不得参与 active check | FEAT-WT-01 | TC-WT-001 | cr-tracking |
| RULE-WT-04 | Internal docs 只有 artifact `process/docs/**` 可写副本 | FEAT-WT-02 | TC-WT-002 | routing/clean-clone fixture |
| RULE-WT-05 | Doctor 只对 blocking findings 非零，warning 必须显式统计 | FEAT-WT-03 | TC-WT-003 | doctor severity tests |
| RULE-WT-06 | Historical overage 不得被截断；cold 分类必须有 summary/index/hash/correction/ref | FEAT-WT-03 | TC-WT-003 | budget/retention tests |
| RULE-WT-07 | Future verification commands 写真实 Run event；historical recovery 只能 legacy-unverified | FEAT-WT-03 | TC-WT-003,007 | run ledger fixtures |
| RULE-WT-08 | Tracked/package cache blocking；ignored local cache warning | FEAT-WT-04 | TC-WT-004,006 | guardrail classifier fixtures |
| RULE-WT-09 | Root AGENTS wrapper 不是 tracked canonical；clean clone 校验 delivery rule + installer plan | FEAT-WT-04 | TC-WT-004 | clean archive guardrail |
| RULE-WT-10 | CR-046 只收敛 current status，不改写 recovered/post-hoc 时序不伪造 receipt/QA | FEAT-WT-05 | TC-WT-007 | evidence/status cross-check |
| RULE-WT-11 | ST-WT-007 在 CP6 pre-implementation 按对象身份生成 manifest；CP6 完成与 CP7 必须分别复算 SHA256，`protected_original_hash_changes=0` | FEAT-WT-01/03/05 | TC-WT-007 | protected-object firewall fixture |
| RULE-WT-12 | 不得用路径前缀作为唯一身份或 allowlist；身份/原 hash 不可确定，或任一 protected original 变化时，当前 Story 阻断并拆子 CR | FEAT-WT-01/03/05 | TC-WT-007 | identity ambiguity/mutation negative fixtures |
| RULE-WT-13 | 有效 inline fallback 只能支持 CP7 `PASS_WITH_RISK`；缺 fallback/独立 QA 证据则 BLOCKED；任何继承风险 OPEN 时 CP8 最高 `READY_WITH_RISK` | FEAT-WT-05 | TC-WT-007 | verdict-ceiling fixture |

## Gotchas

- `status=warning` 不是“无需处理”，它必须有类别、计数和升级条件。
- `recovered/legacy-unverified` 不能填入猜测的 original timestamp、receipt 或 actor。
- projection 失效时应重建，而不是从 projection 修改 source truth。

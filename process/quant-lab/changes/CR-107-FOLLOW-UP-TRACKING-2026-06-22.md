---
source_cr: "CR-107"
status: "closed"
created_at: "2026-06-22T14:01:49+08:00"
created_by: "host-orchestrator"
updated_at: "2026-06-22T14:13:19+08:00"
checkpoint_source: "CP8"
cr_index_path: "process/changes/CR-INDEX.yaml"
---

# CR107 后续事项跟踪台账

## 目的

本台账记录 CR107 CP8 后续候选及 `FU-CR107-001 Redesign Backlog Decision Gate` 转正式 CR108 的状态。它只覆盖 redesign backlog 决策，不授权源码修改、CR105、CR089 恢复、QMT / MiniQMT / XtQuant / gateway runtime、NAS、凭据、账户、raw log、submit / cancel / buy / sell、simulation / live、provider fetch、lake write 或 catalog publish。

## 结构化候选项

```yaml
follow_up_items:
  - id: "CR-108"
    legacy_ids:
      - "FU-CR107-001"
    title: "Redesign Backlog Decision Gate"
    kind: "requirement-change"
    lifecycle_status: "closed"
    readiness_status: "ready"
    gate_status: "closed"
    gate_profile: "compact"
    source_cr: "CR-107"
    source_decision_id: "DQ-CP8-CR107-01"
    priority: 1
    formal_cr_path: "process/changes/CR-108-REDESIGN-BACKLOG-DECISION-GATE-2026-06-22.md"
    blocked_by: []
    superseded_by: []
    impact_surface:
      - "redesign_backlog_decision"
      - "candidate_prioritization"
      - "no_code_change"
      - "no_runtime_authorization"
    conflict_keys:
      documents:
        - "docs/design/REDESIGN-BACKLOG-DECISION-CR108.md"
      stories: []
      files: []
      external_interfaces: []
      security_runtime:
        - "no_runtime_connection"
        - "no_order_write"
        - "no_nas_access"
        - "no_credential_read"
        - "no_provider_lake_publish"
      risk_acceptance: []
      source_decisions:
        - "DQ-CP8-CR107-01"
    authorization_required:
      runtime: false
      credential_read: false
      nas_access: false
      trading_write: false
    next_action: "CR108 已由用户批准并关闭为 READY；下一候选 FU-CR108-002 已启动为 CR109。"
  - id: "FU-CR107-001"
    legacy_ids: []
    title: "Redesign Backlog Decision Gate"
    kind: "requirement-change"
    lifecycle_status: "closed"
    readiness_status: "ready"
    gate_status: "closed"
    gate_profile: "compact"
    source_cr: "CR-107"
    source_decision_id: "DQ-CP8-CR107-01"
    priority: 1
    formal_cr_path: "process/changes/CR-108-REDESIGN-BACKLOG-DECISION-GATE-2026-06-22.md"
    blocked_by: []
    superseded_by: []
    impact_surface:
      - "redesign_backlog_decision"
      - "candidate_prioritization"
      - "no_code_change"
      - "no_runtime_authorization"
    conflict_keys:
      documents:
        - "docs/design/REDESIGN-BACKLOG-DECISION-CR108.md"
      stories: []
      files: []
      external_interfaces: []
      security_runtime:
        - "no_runtime_connection"
        - "no_order_write"
        - "no_nas_access"
        - "no_credential_read"
        - "no_provider_lake_publish"
      risk_acceptance: []
      source_decisions:
        - "DQ-CP8-CR107-01"
    authorization_required:
      runtime: false
      credential_read: false
      nas_access: false
      trading_write: false
    next_action: "已转 CR108 并关闭；后续由 CR108 follow-up tracking 和 CR109 接续。"
  - id: "FU-CR107-002"
    legacy_ids: []
    title: "Architecture-only Redesign HLD Gate"
    kind: "architecture-realignment"
    lifecycle_status: "candidate"
    readiness_status: "n/a"
    gate_status: "not_started"
    gate_profile: "compact"
    source_cr: "CR-107"
    source_decision_id: "DQ-CP8-CR107-01"
    priority: 2
    formal_cr_path: ""
    blocked_by:
      - "wait_for_CR108_backlog_decision"
    superseded_by: []
    impact_surface:
      - "architecture_only_redesign"
      - "module_boundary"
      - "state_contract"
      - "context_ledger_routing"
    conflict_keys:
      documents:
        - "docs/design/HLD.md"
        - "docs/design/ARCHITECTURE-DECISION.md"
      stories: []
      files: []
      external_interfaces: []
      security_runtime:
        - "no_runtime_connection"
      risk_acceptance: []
      source_decisions:
        - "DQ-CP8-CR107-01"
    authorization_required:
      runtime: false
      credential_read: false
      nas_access: false
      trading_write: false
    next_action: "仅当用户选择架构路线时启动"
```

## 分流总览

| 类别 | 数量 | 阻断当前交付 | 说明 |
|---|---:|---|---|
| 关闭范围 | 1 | 否 | CR107 scope intake 已关闭 |
| 不授权范围 | 1 | 否 | 高风险 runtime / trading / NAS / credentials / publish 不授权 |
| 风险接受项 | 1 | 否 | CR107 不执行实现 / HLD / runtime，只做 scope routing |
| 后续 CR 候选项 | 2 | 否 | FU-CR107-001 已转 CR108 并关闭；FU-CR107-002 保持 candidate |
| 取消 / deferred 项 | 1 | 否 | CR-026 / CR-027 / CR-028 延后；RA-CR097-001 / CR089 / CR105 不启动 |

## 后续 CR / Spike 候选索引

| 候选编号 | 标题 | 状态 | 类型 | 优先级 | 影响面 / 冲突键 | 正式 CR 路径 | 相关 active CR / blocked_by / superseded_by | 当前门控 | 阻塞原因 | 下一步 | 来源 |
|---|---|---|---|---:|---|---|---|---|---|---|---|
| CR-108 | Redesign Backlog Decision Gate | closed | CR | 1 | redesign_backlog_decision; no_runtime_connection; no_order_write | `process/changes/CR-108-REDESIGN-BACKLOG-DECISION-GATE-2026-06-22.md` | closed_by=CP8-CR108 | closed | 无 | 已关闭为 READY；后续由 CR109 接续 | DQ-CP8-CR107-01 |
| FU-CR107-001 | Redesign Backlog Decision Gate | closed | CR | 1 | redesign_backlog_decision; no_runtime_connection; no_order_write | `process/changes/CR-108-REDESIGN-BACKLOG-DECISION-GATE-2026-06-22.md` | formalized_by=CR-108; closed_by=CP8-CR108 | closed | 无 | 已关闭；后续由 CR108 follow-up tracking 接续 | DQ-CP8-CR107-01 |
| FU-CR107-002 | Architecture-only Redesign HLD Gate | candidate | CR | 2 | architecture_only_redesign; no_runtime_connection |  | blocked_by=CR108 backlog decision | not_started | 需要先确认 CR108 backlog decision | 仅当用户选择架构路线时启动 | DQ-CP8-CR107-01 |

## 不授权范围

| 项目 ID | 范围 | 原因 | 需要未来授权时的动作 | 来源 |
|---|---|---|---|---|
| NA-CR107-FU-01 | 源码修改、CR105、CR089 恢复、QMT / MiniQMT / XtQuant / gateway runtime、NAS、凭据、账户、raw log、submit / cancel / buy / sell、simulation / live、provider fetch、lake write、catalog publish | CR107 / CR108 都是 no-code / no-runtime 的 redesign scope 与 backlog 决策 | 必须新建独立 CR 并经人工门禁授权 | DQ-CP8-CR107-01 |

## 关闭范围

| 项目 ID | 已关闭内容 | 关闭证据 | 来源 |
|---|---|---|---|
| CLOSE-CR107-01 | CR107 Redesign Scope Intake and Candidate Prioritization | `process/checkpoints/CP8-CR107-DELIVERY-READINESS.md` | CP8 |

## 取消 / Deferred 项

| 项目 ID | 内容 | 状态 | 原因 | 可重启条件 | 来源 |
|---|---|---|---|---|---|
| DEF-CR107-01 | CR-026 / CR-027 / CR-028 延后；RA-CR097-001 / CR089 / CR105 不启动 | deferred | 不属于当前低风险 redesign backlog decision 默认路线 | 用户明确选择并完成独立门禁 | CR107 scope intake |

---
source_cr: "CR-159"
status: "active"
created_at: "2026-07-06T23:30:00+08:00"
created_by: "host-orchestrator"
updated_at: "2026-07-06T23:30:00+08:00"
checkpoint_source: "CR159-CP2-v0.3-approved-implementation"
cr_index_path: "process/changes/CR-INDEX.json"
---

# CR159 后续事项跟踪台账

## 目的

本台账记录 CR159 CP2 v0.3 实施过程中识别、但不在 CR159 B1 范围内的后续治理候选项。CR159 本体保持 `active / implementation`；本台账不重开 CR159，不授权真实 lake、NAS、provider、runtime、broker、feed、order、reconciliation、store/catalog/registry、publish、Git remote write 或交易相关操作。

## 当前事实

| 项 | 结论 | 证据 |
|---|---|---|
| CR159 CP2 decision | `approved` (v0.3, 7 decisions) | `process/checkpoints/CP2-CR159-PROCESS-HYGIENE-STATE-V2-CRINDEX-RETURN-VERIFY-SCOPE.md` |
| 子项 A 实施 | source_refs budget resolved + phase deleted + 4 residual WARN documented | `meta-flow state check --mode audit` |
| 子项 B1 实施 | CR-158 status → closed; CR-INDEX item CR-151/152/153/154 required_evidence → static_fixture_only_validation（留痕） | `process/backups/CR-INDEX.json.bak-20260706-impl` |
| 子项 C 探测 | N/A — 工具已有字段非空校验 | `meta_flow/workflow/story_evidence.py:576-636,687-703`；CR158 CP6/CP7 evidence-check OK |
| authz L2 根因 | formal CR 正文 "real lake" 关键字触发 `infer_required_evidence_from_text` 误判；cr check 用 `record_from_cr_file`（formal CR），不读 CR-INDEX item required_evidence | `meta_flow/policies/authz.py:184-203`；`meta_flow/workflow/cr_lifecycle.py:510-523` |

## 授权边界

| 类别 | 当前授权 |
|---|---|
| CR159 process hygiene within CP2-approved scope | authorized |
| FU-CR159-001 / FU-CR159-002 台账登记 | authorized as local process hygiene |
| Git remote write / push | not-authorized |
| True release execution / publish | not-authorized |
| Real lake / NAS / provider / runtime / broker / feed / order / reconciliation / store/catalog/registry write | not-authorized |
| Reopen closed CR-151/152/153/154/158 | not-authorized |
| Modify closed CR formal body (reword "real lake") | not-authorized without separate gate |
| Modify meta-flow infer logic | not-authorized in CR159 (separate tooling CR) |

## 结构化候选项

```yaml
follow_up_items:
  - id: "FU-CR159-001"
    title: "CR-INDEX/ledger global legacy normalization (incl. authz L2 infer false-positive)"
    kind: "requirement-change"
    lifecycle_status: "candidate"
    readiness_status: "not_ready"
    gate_status: "not_started"
    gate_profile: "compact"
    source_cr: "CR-159"
    source_checkpoint: "CR159-CP2-v0.3-approved-implementation"
    priority: 2
    formal_cr_path: ""
    blocked_by: ""
    impact_surface:
      - "cr_index_legacy_status_enum"
      - "cr_ledger_legacy_status_enum"
      - "missing_summary_ref_57_items"
      - "invalid_cr_type_4_items"
      - "authz_l2_infer_false_positive"
    conflict_keys:
      legacy:
        - "cr_legacy_status_normalization"
        - "closed_cr_body_infer_rewording"
        - "meta_flow_infer_logic_change"
    authorization_required:
      source_code_change: false
      closed_cr_body_change: true
      meta_flow_tool_change: false
      runtime: false
      real_lake_readonly: false
      trading_write: false
    scope_summary:
      cr_ledger_invalid_status: 63
      cr_index_invalid_status: 58
      missing_summary_ref: 57
      invalid_cr_type: 4
      authz_l2_infer_false_positive: "CR-151/152/153/154 formal CR body 'real lake' keyword in non-authorization declarations triggers infer_required_evidence_from_text; fix requires closed-CR body rewording (real lake -> real_lake / 真实数据湖) or meta-flow infer logic to distinguish authorization context"
    next_action: "Not started. Requires separate authorization to reword closed CR bodies or change meta-flow infer logic; 44 historical status values need per-CR semantic mapping to legal enum."
  - id: "FU-CR159-002"
    title: "STATE v2 schema/check allowlist extension for runtime-contract fields"
    kind: "requirement-change"
    lifecycle_status: "candidate"
    readiness_status: "not_ready"
    gate_status: "not_started"
    gate_profile: "compact"
    source_cr: "CR-159"
    source_checkpoint: "CR159-CP2-v0.3-approved-implementation"
    priority: 3
    formal_cr_path: ""
    blocked_by: ""
    impact_surface:
      - "state_v2_schema_allowlist"
      - "meta_flow_state_check"
    conflict_keys:
      tooling:
        - "state_v2_schema_extension"
        - "state_router_contract_alignment"
    authorization_required:
      source_code_change: true
      meta_flow_tool_change: true
      closed_cr_body_change: false
      runtime: false
      real_lake_readonly: false
      trading_write: false
    scope_summary:
      residual_warn_fields: ["current_agent", "last_action", "cr_tracking_ref", "cr_ledger_ref"]
      reason: "state-router/SKILL.md:104,114,290 + STATE-TEMPLATE.md:7,11,318,319 contract dependency; v2 schema does not recognize these fields, causing 4 residual WARN"
      fix: "Extend STATE v2 schema/check allowlist to legitimize these 4 runtime-contract fields, eliminating residual WARN without deleting them"
    next_action: "Not started. Requires meta-flow tooling CR to extend v2 schema allowlist; low priority since 4 residual WARN are documented and do not break state-router contract."
```

## 后续候选索引

| 候选编号 | 标题 | 状态 | 类型 | 优先级 | 影响面 / 冲突键 | 正式 CR 路径 | 当前门控 | 相关 CR | 阻塞原因 | 下一步 | 来源 |
|---|---|---|---|---:|---|---|---|---|---|---|---|
| FU-CR159-001 | CR-INDEX/ledger global legacy normalization (incl. authz L2 infer false-positive) | candidate | requirement-change | 2 | cr_legacy_status_normalization / closed_cr_body_infer_rewording / meta_flow_infer_logic_change | N/A | not_started | CR-159 | 需 closed CR 正文改写或 meta-flow infer 逻辑变更授权 | 仅当用户要求全局 cr check 0 ERROR 或 authz L2 清零时启动 | CR159 实施 |
| FU-CR159-002 | STATE v2 schema/check allowlist extension | candidate | requirement-change | 3 | state_v2_schema_extension / state_router_contract_alignment | N/A | not_started | CR-159 | 需 meta-flow 工具 CR 扩展 v2 schema | 仅当用户要求消除 4 residual WARN 时启动 | CR159 实施 |

## 风险接受与门禁规则

| 规则 | 处理 |
|---|---|
| CR159 CP8 closure | Remains `READY_WITH_RISK` (scoped); this台账 does not reopen CR159. |
| B2 legacy ERROR | Global `cr check` retains 182+ ERROR (B2); accepted as deferred to FU-CR159-001. |
| authz L2 false-positive | CR-151/152/153/154 authz L2 ERROR retained; root cause is infer logic, not data; deferred to FU-CR159-001. |
| 4 residual WARN | STATE current_agent/last_action/cr_tracking_ref/cr_ledger_ref retained for state-router contract; deferred to FU-CR159-002. |
| Closed CR modification | Rewording closed CR-151/152/153/154 formal body forbidden without separate authorization gate. |
| Git remote write / push / true release / runtime / real data | Not authorized. |

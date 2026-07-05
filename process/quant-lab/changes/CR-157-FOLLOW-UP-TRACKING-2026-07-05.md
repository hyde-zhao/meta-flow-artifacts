---
source_cr: CR-157
status: closed
created_at: '2026-07-05T15:58:00+08:00'
created_by: host-orchestrator
updated_at: '2026-07-05T19:45:00+08:00'
checkpoint_source: CP8-CR157-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE-RELEASE-READINESS
cr_index_path: process/changes/CR-INDEX.json
closed_by: CR-158
closure_ref: process/changes/CR-158-EVENT-ML-STRATEGY-ADAPTER-UNIFIED-IMPLEMENTATION-2026-07-05.md
---

# CR157 后续事项跟踪台账

## 目的

本台账记录 CR157 CP8 关闭后保留的 adapter deferred items，并记录用户已选择把 legacy deferred ids `DF-CR157-001` 和 `DF-CR157-002` 合并启动为一个正式 CR158。CR157 本体保持 `closed / READY_WITH_RISK`，本台账不重开 CR157，也不授权真实 lake、NAS、provider、credential、runtime、broker、feed、model registry、store/catalog、publish、Git remote write 或交易相关操作。

## 当前事实

| 项 | 结论 | 证据 |
|---|---|---|
| CR157 CP8 release decision | `READY_WITH_RISK` approved | `process/checkpoints/CP8-CR157-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE-RELEASE-READINESS.md` |
| Deferred event adapter | `DF-CR157-001` promoted as `FU-CR157-001` | `docs/product/USE-CASES.md#Deferred-Ideas` |
| Deferred ML adapter | `DF-CR157-002` promoted as `FU-CR157-002` | `docs/product/USE-CASES.md#Deferred-Ideas` |
| User decision | Merge both deferred items into one CR | user request on 2026-07-05 |
| Formal CR | `CR-158` closed-current-delivery / READY_WITH_RISK | `process/changes/CR-158-EVENT-ML-STRATEGY-ADAPTER-UNIFIED-IMPLEMENTATION-2026-07-05.md` |

## 授权边界

| 类别 | 当前授权 |
|---|---|
| CR158 current delivery | closed-current-delivery / READY_WITH_RISK |
| Event + ML adapter HLD / Story / LLD / implementation | completed for local/static/fixture scope under CR158; real runtime remains not authorized |
| Real event feed / live listener | not authorized |
| Real ML model training / external model service / model registry write | not authorized |
| Real lake / NAS / provider / credential / env / session access | not authorized |
| Runtime / simulation / paper / live / trading / broker operation | not authorized |
| Store / catalog / registry / prediction write | not authorized |
| External framework clone/install/run, Git remote write, publish | not authorized |

## 结构化候选项

```yaml
follow_up_items:
  - id: "CR-158"
    legacy_ids:
      - "DF-CR157-001"
      - "DF-CR157-002"
    title: "Event + ML strategy adapter unified implementation"
    kind: "requirement-change"
    lifecycle_status: "closed"
    readiness_status: "ready_with_risk"
    gate_status: "closed"
    gate_profile: "architecture-major"
    source_cr: "CR-157"
    source_checkpoint: "CP8-CR157-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE-RELEASE-READINESS"
    source_decision_id: "DQ-CP8-CR157-002"
    priority: 1
    formal_cr_path: "process/changes/CR-158-EVENT-ML-STRATEGY-ADAPTER-UNIFIED-IMPLEMENTATION-2026-07-05.md"
    related_closed_cr: "CR-158"
    closed_reason: "merged into CR158 and closed as READY_WITH_RISK at CP8"
    impact_surface:
      - "event-strategy"
      - "ml-strategy"
      - "adapter-contract"
      - "evidence-index"
      - "stage2-stage3-handoff"
      - "no-runtime-boundary"
    next_action: "No action; CR158 current delivery is closed. Future real runtime validation requires separate gate / CR."
  - id: "FU-CR157-001"
    legacy_ids:
      - "DF-CR157-001"
    title: "Event strategy adapter implementation"
    kind: "requirement-change"
    lifecycle_status: "closed"
    readiness_status: "ready_with_risk"
    gate_status: "closed"
    gate_profile: "architecture-major"
    source_cr: "CR-157"
    source_checkpoint: "CP8-CR157-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE-RELEASE-READINESS"
    source_decision_id: "DQ-CP8-CR157-002"
    priority: 1
    formal_cr_path: "process/changes/CR-158-EVENT-ML-STRATEGY-ADAPTER-UNIFIED-IMPLEMENTATION-2026-07-05.md"
    related_closed_cr: "CR-158"
    closed_under_cr: "CR-158"
    merged_with:
      - "FU-CR157-002"
      - "DF-CR157-002"
    closed_reason: "merged into CR158 and closed as READY_WITH_RISK at CP8"
    impact_surface:
      - "event-strategy"
      - "adapter-contract"
      - "evidence-index"
      - "stage2-stage3-handoff"
      - "no-runtime-boundary"
    conflict_keys:
      documents:
        - "docs/product/USE-CASES.md"
        - "docs/product/REQUIREMENTS.md"
        - "docs/product/SCENARIOS.yaml"
        - "docs/product/TEST-MATRIX.md"
      stories: []
      files: []
      external_interfaces:
        - "event strategy adapter contract"
      security_runtime:
        - "NO_RUNTIME"
        - "NO_PROVIDER_FETCH"
        - "NO_CREDENTIAL_READ"
      risk_acceptance:
        - "R-CR158-RUNTIME-OVERCLAIM"
      source_decisions:
        - "SGQ-CR157-001"
        - "DQ-CP8-CR157-002"
    authorization_required:
      runtime: false
      credential_read: false
      nas_access: false
      trading_write: false
    next_action: "No action; CR158 current delivery is closed. Future real runtime validation requires separate gate / CR."
  - id: "FU-CR157-002"
    legacy_ids:
      - "DF-CR157-002"
    title: "ML strategy adapter implementation"
    kind: "requirement-change"
    lifecycle_status: "closed"
    readiness_status: "ready_with_risk"
    gate_status: "closed"
    gate_profile: "architecture-major"
    source_cr: "CR-157"
    source_checkpoint: "CP8-CR157-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE-RELEASE-READINESS"
    source_decision_id: "DQ-CP8-CR157-002"
    priority: 1
    formal_cr_path: "process/changes/CR-158-EVENT-ML-STRATEGY-ADAPTER-UNIFIED-IMPLEMENTATION-2026-07-05.md"
    related_closed_cr: "CR-158"
    closed_under_cr: "CR-158"
    merged_with:
      - "FU-CR157-001"
      - "DF-CR157-001"
    closed_reason: "merged into CR158 and closed as READY_WITH_RISK at CP8"
    impact_surface:
      - "ml-strategy"
      - "adapter-contract"
      - "evidence-index"
      - "stage2-stage3-handoff"
      - "no-runtime-boundary"
    conflict_keys:
      documents:
        - "docs/product/USE-CASES.md"
        - "docs/product/REQUIREMENTS.md"
        - "docs/product/SCENARIOS.yaml"
        - "docs/product/TEST-MATRIX.md"
      stories: []
      files: []
      external_interfaces:
        - "ML strategy adapter contract"
      security_runtime:
        - "NO_RUNTIME"
        - "NO_PROVIDER_FETCH"
        - "NO_CREDENTIAL_READ"
      risk_acceptance:
        - "R-CR158-RUNTIME-OVERCLAIM"
      source_decisions:
        - "SGQ-CR157-001"
        - "DQ-CP8-CR157-002"
    authorization_required:
      runtime: false
      credential_read: false
      nas_access: false
      trading_write: false
    next_action: "No action; CR158 current delivery is closed. Future real runtime validation requires separate gate / CR."
```

## 后续候选索引

| 候选编号 | 标题 | 状态 | 类型 | 优先级 | 影响面 / 冲突键 | 正式 CR 路径 | 当前门控 | 相关 CR | 阻塞原因 | 下一步 | 来源 |
|---|---|---|---|---:|---|---|---|---|---|---|---|
| CR-158 | Event + ML strategy adapter unified implementation | closed-current-delivery / READY_WITH_RISK | requirement-change | 1 | event-strategy / ml-strategy / adapter-contract / evidence-index / no-runtime-boundary | `process/changes/CR-158-EVENT-ML-STRATEGY-ADAPTER-UNIFIED-IMPLEMENTATION-2026-07-05.md` | closed | related_closed_cr=CR-158; legacy_ids=DF-CR157-001,DF-CR157-002 | accepted risk `R-CR158-CP7-STATIC-FIXTURE-ONLY` | No action; future real runtime validation requires separate gate / CR | CR157 CP8 |
| FU-CR157-001 (`DF-CR157-001`) | Event strategy adapter implementation | closed under CR158 / READY_WITH_RISK | requirement-change | 1 | event-strategy / adapter-contract / evidence-index / no-runtime-boundary | `process/changes/CR-158-EVENT-ML-STRATEGY-ADAPTER-UNIFIED-IMPLEMENTATION-2026-07-05.md` | closed | related_closed_cr=CR-158; merged_with=FU-CR157-002 / DF-CR157-002 | accepted static/fixture risk | No action; future real event feed validation requires separate gate / CR | CR157 CP8 |
| FU-CR157-002 (`DF-CR157-002`) | ML strategy adapter implementation | closed under CR158 / READY_WITH_RISK | requirement-change | 1 | ml-strategy / adapter-contract / evidence-index / no-runtime-boundary | `process/changes/CR-158-EVENT-ML-STRATEGY-ADAPTER-UNIFIED-IMPLEMENTATION-2026-07-05.md` | closed | related_closed_cr=CR-158; merged_with=FU-CR157-001 / DF-CR157-001 | accepted static/fixture risk | No action; future real ML training/model registry validation requires separate gate / CR | CR157 CP8 |

## 状态索引同步

| 对象 | 路径 | 同步要求 | 当前状态 |
|---|---|---|---|
| 正式 CR | `process/changes/CR-158-EVENT-ML-STRATEGY-ADAPTER-UNIFIED-IMPLEMENTATION-2026-07-05.md` | 记录 closed CR、scope、impact surface 和 gate | closed-current-delivery / READY_WITH_RISK |
| CR 索引 | `process/changes/CR-INDEX.json` | 记录 CR158 closed | synced |
| CR ledger | `process/state/CR-LEDGER.ndjson` | 记录 CR158 closed_current_delivery_ready_with_risk | synced |
| 一致性检查 | `meta-flow check cr-tracking --project-root .` | 关闭后执行 | CR158 相关 active/closed 冲突已修正；命令仍会报告既有 CR-010/CR-018/CR-031 lifecycle_status enum debt 和历史 follow-up indexing warnings。 |

## CR158 Closure Update

- 2026-07-05T19:45:00+08:00：CR158 已关闭为 `closed-current-delivery / READY_WITH_RISK`。`FU-CR157-001` 与 `FU-CR157-002` 均视为 under CR158 closed，不再作为 active follow-up 阻塞项。
- 后续真实 event feed、ML training、registry、runtime、publish 或 trading 验证必须另起 gate / CR。

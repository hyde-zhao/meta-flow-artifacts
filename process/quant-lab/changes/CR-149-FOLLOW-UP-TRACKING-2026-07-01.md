---
source_cr: "CR-149"
status: "open"
created_at: "2026-07-01T21:26:14+08:00"
created_by: "host-orchestrator"
updated_at: "2026-07-01T21:26:14+08:00"
checkpoint_source: "RA-CR149-001-post-sync-semantics"
cr_index_path: "process/changes/CR-INDEX.yaml"
---

# CR-149 后续事项跟踪台账

## 目的

本台账记录 CR-149 关闭后由 RA-CR149-001 触发的 NAS current-truth sync 后续治理事项。CR-149 本体保持 closed，不在本台账中悄悄修改已关闭 CR 的 definition-of-done；需要改变 Phase 1 exit 判据、修改 guardrail 脚本、执行 NAS metadata normalization、读取新的 NAS 访问向量或接受 residual risk 时，必须由独立决策或正式 CR 承接。

## 当前事实

| 项 | 结论 | 证据 |
|---|---|---|
| RA-CR149-001 scoped dry-run | PASS | `process/evidence/CR149-NAS-CURRENT-TRUTH-SYNC-DRY-RUN-2026-07-01.json` |
| RA-CR149-001 scoped execute | PASS | `process/evidence/CR149-NAS-CURRENT-TRUTH-SYNC-EXECUTE-2026-07-01.json` |
| Raw post-sync strict check | BLOCKED | `process/evidence/CR149-NAS-MULTINODE-CONSISTENCY-POST-SYNC-2026-07-01.json` |
| Semantics interpretation | CONTENT PASS / METADATA BLOCKED / STRICT BLOCKED | `process/evidence/CR149-NAS-MULTINODE-CONSISTENCY-POST-SYNC-SEMANTICS-2026-07-01.json` |
| Local source-of-truth manifest | PASS | `process/evidence/CR149-LOCAL-SOURCE-TRUTH-MANIFEST-2026-07-01.json` |
| Disposition preparation | PASS_WITH_STRICT_EXIT_BLOCKED | `process/checks/RA-CR149-001-POSTSYNC-METADATA-DISPOSITION-PREP-2026-07-01.md` |

## 授权边界

| 类别 | 当前授权 |
|---|---|
| process evidence / index / tracking 写入 | authorized as no-risk audit work |
| 原始 post-sync evidence 覆盖或改写 | not-authorized |
| NAS mount / SSH / appliance shell | not-authorized |
| NAS-to-local pull | not-authorized |
| rsync write / retry / metadata normalization | not-authorized |
| chmod / chgrp | not-authorized |
| guardrail checker source-code semantics change | not-authorized without separate CR/Story |
| Phase 1 definition-of-done change | not-authorized without scope-reclassification decision |

## 结构化候选项

```yaml
follow_up_items:
  - id: "RA-CR149-001"
    title: "CR149 NAS current-truth scoped sync"
    kind: "runtime-authorization"
    lifecycle_status: "candidate"
    readiness_status: "not_ready"
    gate_status: "cp2_pending"
    gate_profile: "runtime"
    source_cr: "CR-149"
    formal_cr_path: ""
    source_tracking: "process/checkpoints/CP2-CR149-NAS-CURRENT-TRUTH-SYNC.md"
    impact_surface:
      - "NAS current-truth content replica"
      - "process/evidence/CR149-NAS-*"
      - "Phase 1 NAS multi-node consistency exit"
    blocked_by:
      - "strict_metadata_consistency_blocked_permission_group_itemization"
      - "DQ-RA-CR149-POSTSYNC-METADATA-01B_required_before_closure"
    next_action: "Do not retry or repair. Use existing rsync-daemon-only semantics evidence to prepare a second-stage decision on strict metadata parity versus content-level exit."
    evidence_refs:
      - "process/evidence/CR149-LOCAL-SOURCE-TRUTH-MANIFEST-2026-07-01.json"
      - "process/checks/RA-CR149-001-POSTSYNC-METADATA-DISPOSITION-PREP-2026-07-01.md"
  - id: "FU-CR149-002"
    title: "NAS filesystem metadata parity governance"
    kind: "requirement-change"
    lifecycle_status: "candidate"
    readiness_status: "not_ready"
    gate_status: "not_started"
    gate_profile: "standard"
    source_cr: "CR-149"
    formal_cr_path: ""
    source_tracking: "process/evidence/CR149-NAS-MULTINODE-CONSISTENCY-POST-SYNC-SEMANTICS-2026-07-01.json"
    impact_surface:
      - "NAS permission/group parity"
      - "multi-node reader usability"
      - "check_nas_multinode_consistency.py semantics"
      - "Phase 1 exit criteria"
    blocked_by:
      - "requires_scope_reclassification_or_new_CR_before_changing_phase_1_exit"
      - "requires_separate_CR_or_story_before_guardrail_checker_semantics_change"
      - "requires_runtime_authorization_before_mount_shell_pull_or_metadata_normalization"
    next_action: "Keep as candidate. If user later wants content-only exit, metadata risk acceptance, checker semantics change, or p/g normalization, start an explicit decision gate or formal CR."
```

## 分流总览

| 类别 | 数量 | 阻断当前交付 | 说明 |
|---|---:|---|---|
| 已执行 runtime gate | 1 | 是 | RA-CR149-001 scoped sync 已执行；strict metadata consistency blocked。 |
| 后续治理候选 | 1 | 不直接阻断无风险本地工作 | FU-CR149-002 承接 p/g parity，不允许从 RA-CR149-001 中无声消失。 |

## 后续候选索引

| 候选编号 | 标题 | 状态 | 类型 | 优先级 | 影响面 / 冲突键 | 正式 CR 路径 | 当前门控 | 阻塞原因 | 下一步 | 来源 |
|---|---|---|---|---:|---|---|---|---|---|---|
| RA-CR149-001 | CR149 NAS current-truth scoped sync | candidate | runtime-authorization | 1 | NAS content replica / Phase 1 NAS exit |  | cp2_pending | strict metadata p/g itemization remains unresolved; future explicit decision required | Disposition prep complete; next step is future explicit decision only when user wants closure | CP2-CR149 |
| FU-CR149-002 | NAS filesystem metadata parity governance | candidate | requirement-change | 2 | permission/group parity / checker semantics / reader usability |  | not_started | Requires scope reclassification, formal CR/Story, or runtime authorization depending on chosen route | Do not execute yet; start only after explicit user decision | RA-CR149-001 post-sync semantics |

## 风险接受与门禁规则

| 规则 | 处理 |
|---|---|
| Existing evidence interpretation and process tracking | Direct no-risk audit work |
| Raw evidence mutation | Forbidden; add interpretation evidence instead |
| Content-only Phase 1 exit reclassification | Requires explicit scope-reclassification decision; p/g parity must remain tracked by FU-CR149-002 |
| `READY_WITH_RISK` acceptance for metadata parity | Requires explicit risk acceptance; use allowed enum `READY_WITH_RISK` plus accepted risk id |
| Guardrail checker semantics change | Requires separate CR/Story with CP6/CP7 |
| Metadata normalization / chmod / chgrp / retry | Requires runtime authorization gate |
| Mount / SSH / appliance shell / NAS-to-local pull | Requires separate runtime authorization gate |

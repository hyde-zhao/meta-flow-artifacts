---
status: draft
version: "1.0"
active_change_ref: "CR-046"
source_hld: "docs/design/CR046-EVIDENCE-INTEGRITY-HLD.md"
source_adr: "docs/design/CR046-EVIDENCE-INTEGRITY-ARCHITECTURE-DECISION.md"
confirmed_by: ""
confirmed_at: ""
---

# CR-046 Feature Design Matrix

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-12 | meta-se | CP4 首版；判定四个 Feature 均需实现设计，冻结七 Story 的 full-lld、Feature refs 与三个 CP4/CP5 minor。 |

## 适用性规则

数据/状态机、跨模块公共接口、平台边界、安全审计、迁移/回滚、多 Story 共享契约任一命中即 `required`；只有单文件、无状态、无共享接口的低风险改动才可 waived。CR-046 四个 Feature 均命中多个强制条件。

## Feature 设计矩阵

| Feature ID | Feature | 来源 | 适用性 | 判定理由 | 产物目录 | 关联 Story | lld_policy |
|---|---|---|---|---|---|---|---|
| FEAT-EI-CORE | Evidence Integrity Core | Blueprint CAP-EI-01..03/08；ADR-001/002/010/011 | required | 公共 identity、状态机、平台 discovery/selector/receipt、线程复用与安全失败路径 | `docs/features/cr046-core/` | ST-EI-001..003 | full-lld |
| FEAT-EI-GOVERNANCE | Workspace and Lifecycle Governance | CAP-EI-04；ADR-003..005 | required | route/state/finalization/compaction 跨模块共享真相与迁移回滚 | `docs/features/cr046-governance/` | ST-EI-004 | full-lld |
| FEAT-EI-OBSERVABILITY | Replay, Audit and Cost Observability | CAP-EI-05/06；ADR-007/008/011 | required | provenance、双口径 replay、token 三态和 platform conformance 多消费者 schema | `docs/features/cr046-observability/` | ST-EI-005/006 | full-lld |
| FEAT-EI-CORRECTION | Correction Lifecycle and Pilot Adapter | CAP-EI-07；ADR-006/009 | required | post-close append-only mutation boundary、迁移、授权与回滚 | `docs/features/cr046-correction/` | ST-EI-007 | full-lld |

## Story 下游消费表

| Story | feature_design_refs | required_level | 触发原因 | CP5 设计证据 |
|---|---|---|---|---|
| ST-EI-001 | `cr046-core/{DESIGN,TEST-PLAN,TASKS}.md` | full-lld | chronology/state machine/public contract | `process/stories/STORY-ST-EI-001-gate-chronology-LLD.md` |
| ST-EI-002 | `cr046-core/{DESIGN,TEST-PLAN,TASKS}.md` | full-lld | external platform contract/security/thread identity | `process/stories/STORY-ST-EI-002-dispatch-attestation-LLD.md` |
| ST-EI-003 | `cr046-core/{DESIGN,TEST-PLAN,TASKS}.md` | full-lld | cross-truth correlation/shared schema | `process/stories/STORY-ST-EI-003-cp-attempt-correlation-LLD.md` |
| ST-EI-004 | `cr046-governance/{DESIGN,TEST-PLAN,TASKS}.md` | full-lld | state/routing/migration/compaction | `process/stories/STORY-ST-EI-004-governance-integrity-LLD.md` |
| ST-EI-005 | `cr046-observability/{DESIGN,TEST-PLAN,TASKS}.md` | full-lld | telemetry schema/attribution/compatibility | `process/stories/STORY-ST-EI-005-token-telemetry-LLD.md` |
| ST-EI-006 | `cr046-observability/{DESIGN,TEST-PLAN,TASKS}.md`, core DESIGN/TEST-PLAN | full-lld | replay/audit/platform conformance/A-B dogfood | `process/stories/STORY-ST-EI-006-replay-audit-LLD.md` |
| ST-EI-007 | `cr046-correction/{DESIGN,TEST-PLAN,TASKS}.md`, observability DESIGN | full-lld | post-close correction/cross-repo migration/authz | `process/stories/STORY-ST-EI-007-correction-pilot-LLD.md` |

## CP4/CP5 mandatory minor acceptance

| Minor ID | Owner | 机器可检查契约 | Fixture / Evidence |
|---|---|---|---|
| MINOR-EI-01 | ST-EI-002 + ST-EI-006 | D0 freshness 同时绑定 TTL、session_id/epoch、config hash、selector schema；expiry/reload/epoch/hash/schema 变化均触发 re-probe，旧 capability 不得用于新 spawn | `PC-18-D0-FRESHNESS-REPROBE`；core TEST-PLAN CT-CORE-08 |
| MINOR-EI-02 | ST-EI-002 + ST-EI-006 | followup 缺 reuse receipt 时不得继承原 spawn 的 `custom_agent_verified/model_attested`，即使原 spawn verified | `PC-19-FOLLOWUP-NO-REUSE-RECEIPT`；core TEST-PLAN CT-CORE-09 |
| MINOR-EI-03 | ST-EI-006 + ST-EI-007 | legacy `codex_agent_name` 在 strict replay/migration 固定分类 D3 `self-declared-unverifiable`；resolved profile/model/effort 必须为空/unavailable | `MIG-EI-03-LEGACY-CODEX-AGENT-NAME-D3`；observability/correction tests |

## 决策与豁免

提前架构决策均由 CP3-DQ-01..05 关闭。CP4 新增人工决策项=`0`；Feature waived=`0`；Story LLD 降级=`0`。平台 REQUIRED extension 仍非 CURRENT，A-baseline 风险由 CP3-DQ-05 已批准，不在 CP4 重新决策。

## 自检

| 检查项 | 结果 | 证据 |
|---|---|---|
| Feature 判定覆盖 | PASS | 4/4 required |
| required 产物覆盖 | PASS | 4 组 DESIGN/TEST-PLAN/TASKS |
| Story refs/lld_policy 覆盖 | PASS | 7/7 full-lld |
| 三个 minor machine-checkable | PASS | MINOR-EI-01..03 |

## Gotchas

- Feature DESIGN 不替代 Story LLD；七份 LLD 必须在 CP5 前由 meta-dev另行生成。
- REQUIRED platform schema 是实现目标，不是当前 Codex 能力。
- PC fixture 通过不等于真实平台 runtime attestation；Conditional-B 仍需平台边界 receipt。

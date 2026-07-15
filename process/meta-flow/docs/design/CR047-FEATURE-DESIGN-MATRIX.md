---
status: ready-for-cp4
version: "1.0"
active_change_ref: "CR-047"
source_hld: "process/docs/design/CR047-WORKFLOW-TRUTH-HLD.md"
source_adr: "process/docs/design/CR047-WORKFLOW-TRUTH-ARCHITECTURE-DECISION.md"
confirmed_by: ""
confirmed_at: ""
---

# CR-047 Feature Design Matrix

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-14 | host-orchestrator-inline / meta-se | CP3 approved 后建立 5 个 required Feature、7 个 full-lld Story 的实现设计矩阵。 |

## 适用性判定规则

| 维度 | required 触发 | waived / n/a 条件 |
|---|---|---|
| 数据与状态 | State/CR/CURRENT、finding、ledger、manifest 或 lifecycle 变化 | 纯文案且无状态/契约影响 |
| 接口与依赖 | 跨模块、共享 CLI/checker、安装/路由契约 | 单文件局部改动且无共享接口 |
| 权限与证据 | 历史不可变、receipt/QA 上限、授权边界 | 无证据或权限语义变化 |
| 运行与发布 | Doctor/guardrail/lint/test/install 组合门 | 不进入发布质量门 |
| 多 Story 复用 | 同一 Feature 被多个 Story 消费 | 单 Story 可由独立 LLD 完整覆盖 |

## Feature 设计矩阵

| Feature ID | Feature | 来源 | 适用性 | 判定理由 | 需要产物 | 关联 Story | lld_policy | 重访条件 |
|---|---|---|---|---|---|---|---|---|
| FEAT-WT-01 | Truth Consistency | CAP-WT-01；ADR-WT-001 | required | State/CR/CURRENT 三个 owner 的跨源关系和 lifecycle fail-closed | `cr047-truth/{DESIGN,TEST-PLAN,TASKS}.md` | ST-WT-001 | full-lld | 只有现有 schema 无法表达批准对象时重开 CP3 |
| FEAT-WT-02 | Artifact Routing | CAP-WT-02；ADR-WT-002 | required | 跨仓 symlink、relative metadata 与 canonical docs copy 需冻结 | `cr047-routing/{DESIGN,TEST-PLAN,TASKS}.md` | ST-WT-002 | full-lld | 正式外部契约要求根内部 docs 时重开设计 |
| FEAT-WT-03 | Quality Governance | CAP-WT-03；ADR-WT-003 | required | 双基线、生命周期分类、历史 correction 和 Doctor exit 语义跨多个 checker/policy | `cr047-quality/{DESIGN,TEST-PLAN,TASKS}.md` | ST-WT-003 | full-lld | 无法稳定分类的对象保持 blocker 并重开设计 |
| FEAT-WT-04 | Deterministic Delivery Gate | CAP-WT-04；ADR-WT-004 | required | tracked rule、cache classifier、Ruff/pytest 组合门及 clean archive 共享发布契约 | `cr047-delivery/{DESIGN,TEST-PLAN,TASKS}.md` | ST-WT-004, ST-WT-005 | full-lld | 组合层出现独立 release owner 或职责冲突时拆分 |
| FEAT-WT-05 | Operator and Status Convergence | CAP-WT-05；ADR-WT-005/006 | required | 三平台非交互入口与 CR-046 protected-original/status ceiling 共享操作者结论 | `cr047-operator-status/{DESIGN,TEST-PLAN,TASKS}.md` | ST-WT-006, ST-WT-007 | full-lld | protected original 必须改变时立即拆子 CR/HLD |

## Story 下游消费表

| Story ID | feature_design_refs | required_level | trigger_reasons | 设计证据 | CP5 审查 |
|---|---|---|---|---|---|
| ST-WT-001 | `process/docs/features/cr047-truth/{DESIGN,TEST-PLAN,TASKS}.md` | full-lld | cross-module, state-lifecycle, shared-contract | `process/stories/STORY-ST-WT-001-workflow-truth-consistency-LLD.md` | 自动 LLD 检查 + 全量人工确认 |
| ST-WT-002 | `process/docs/features/cr047-routing/{DESIGN,TEST-PLAN,TASKS}.md` | full-lld | cross-repo-routing, portability, shared-contract | `process/stories/STORY-ST-WT-002-portable-artifact-routing-LLD.md` | 同上 |
| ST-WT-003 | `process/docs/features/cr047-quality/{DESIGN,TEST-PLAN,TASKS}.md` | full-lld | policy-schema, history-integrity, cross-module | `process/stories/STORY-ST-WT-003-doctor-history-governance-LLD.md` | 同上 |
| ST-WT-004 | `process/docs/features/cr047-delivery/{DESIGN,TEST-PLAN,TASKS}.md` | full-lld | guardrail, installer-contract, package-security | `process/stories/STORY-ST-WT-004-clean-clone-guardrail-LLD.md` | 同上 |
| ST-WT-005 | `process/docs/features/cr047-delivery/{DESIGN,TEST-PLAN,TASKS}.md` | full-lld | repo-wide-lint, regression, shared-preflight | `process/stories/STORY-ST-WT-005-ruff-regression-gate-LLD.md` | 同上 |
| ST-WT-006 | `process/docs/features/cr047-operator-status/{DESIGN,TEST-PLAN,TASKS}.md` | full-lld | multi-platform, installer-cli, release-preflight | `process/stories/STORY-ST-WT-006-noninteractive-install-preflight-LLD.md` | 同上 |
| ST-WT-007 | `process/docs/features/cr047-operator-status/{DESIGN,TEST-PLAN,TASKS}.md` | full-lld | protected-history, object-identity, evidence-ceiling | `process/stories/STORY-ST-WT-007-cr046-status-firewall-LLD.md` | 同上 |

## 提前确认的关键决策

CP3-DQ-01..06 已全部 approved；本矩阵没有新增架构、安全、运行授权或风险接受决策。CP5 只确认 7 份 Story 设计证据是否可实现。任何 protected original mutation 不进入 CP5 选择题，必须 fail-closed 并拆子 CR。

## 豁免与 N/A

Feature waived=0；Story technical-note=0；Story waived=0。原因是 architecture-major 且每个 Story 均涉及共享契约、状态/历史证据或发布质量门。

## 自检

| 检查项 | 结果 | 证据 |
|---|---|---|
| Feature / Epic 判定 | PASS | 5/5 required |
| required 产物计划 | PASS | 5 组 DESIGN/TEST-PLAN/TASKS |
| Story refs 与 policy | PASS | 7/7 full-lld |
| 新增提前决策 | PASS | 0；沿用 CP3-DQ-01..06 |

## Gotchas

- Feature DESIGN 不替代 Story LLD；CP5 必须逐 Story 可定位。
- full-lld 是风险分级结论，不表示已经授权实现。
- CR-046/047 同目录对象不得用路径前缀作为身份；manifest 只能按 formal ref/provenance 建立。

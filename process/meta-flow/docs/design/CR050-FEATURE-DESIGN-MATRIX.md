---
status: ready-for-cp4
version: "1.0"
active_change_ref: "CR-050"
source_hld: "process/docs/design/CR050-GIT-BRANCH-HLD.md"
source_adr: "process/docs/design/CR050-GIT-BRANCH-ARCHITECTURE-DECISION.md"
confirmed_by: ""
confirmed_at: ""
---

# CR-050 Feature Design Matrix

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-16 | host-orchestrator-inline / meta-se | CP3 R3批准后建立4个required Feature、4个full-lld Story的实现设计矩阵；冻结001→002→004→003顺序与2/2投影门。 |

## 适用性判定规则

| 维度 | required 触发 | waived / n/a 条件 |
|---|---|---|
| 状态与证据 | Git ref、typed intent/plan/attempt、CR/state/current投影变化 | 纯文案且无状态/证据语义 |
| 接口与依赖 | CLI、CR lifecycle、workspace route、Git adapter跨模块 | 单文件局部实现且无共享接口 |
| 权限与安全 | remote ref写/删、authz、branch protection、argv注入面 | 无权限或外部写变化 |
| 可靠性 | 双仓PARTIAL、幂等resume、fresh reproof、恢复ref | 无失败恢复或并发竞态 |
| 多Story复用 | 四Story共享types/adapter/result/projection契约 | 单Story完全独立且低风险 |

## Feature 设计矩阵

| Feature ID | Feature | 来源 | 适用性 | 判定理由 | 需要产物 | 关联 Story | lld_policy | 重访条件 |
|---|---|---|---|---|---|---|---|---|
| FEAT-GB-01 | Paired Branch Open | CAP-GB-01；ADR-R2-001 | required | 跨仓route、clean/base/upstream与bootstrap writer顺序需冻结 | `cr050-open/{DESIGN,TEST-PLAN,TASKS}.md` | ST-GB-001 | full-lld | writer复用要求隐式remote副作用时回CP3 |
| FEAT-GB-02 | Safe Ref Publish | CAP-GB-02；ADR-R2-001/003 | required | 只发布existing commit、matching evidence与no-auto-commit属安全契约 | `cr050-publish/{DESIGN,TEST-PLAN,TASKS}.md` | ST-GB-002 | full-lld | 未来需要path allowlisted commit时另立CR |
| FEAT-GB-04 | Paired Default Fast-forward | CAP-GB-03；ADR-R2-002/003 | required | default写、高风险authz、非原子PARTIAL、2/2投影门与exact argv | `cr050-merge/{DESIGN,TEST-PLAN,TASKS}.md` | ST-GB-004 | full-lld | projection门不可机器保证时切project-first并重开CP2/CP3 |
| FEAT-GB-03 | Merge Proof and Cleanup | CAP-GB-04；ADR-R2-004 | required | delete authz、fresh ancestry proof、recovery ref和幂等清理 | `cr050-finish/{DESIGN,TEST-PLAN,TASKS}.md` | ST-GB-003 | full-lld | squash/rebase需trusted forge receipt时另立adapter CR |

## Story 下游消费表

| Story ID | feature_design_refs | required_level | trigger_reasons | 设计证据 | CP5审查 |
|---|---|---|---|---|---|
| ST-GB-001 | `process/docs/features/cr050-open/{DESIGN,TEST-PLAN,TASKS}.md` | full-lld | cross-repo, shared-contract, remote-write | `process/stories/STORY-ST-GB-001-paired-branch-open-LLD.md` | 自动LLD检查+全量人工确认 |
| ST-GB-002 | `process/docs/features/cr050-publish/{DESIGN,TEST-PLAN,TASKS}.md` | full-lld | remote-write, evidence, security | `process/stories/STORY-ST-GB-002-committed-ref-publish-LLD.md` | 同上 |
| ST-GB-004 | `process/docs/features/cr050-merge/{DESIGN,TEST-PLAN,TASKS}.md` | full-lld | default-write, authz, partial, projection | `process/stories/STORY-ST-GB-004-paired-default-fast-forward-LLD.md` | 同上 |
| ST-GB-003 | `process/docs/features/cr050-finish/{DESIGN,TEST-PLAN,TASKS}.md` | full-lld | delete, proof, recovery, protected-ref | `process/stories/STORY-ST-GB-003-proof-gated-finish-LLD.md` | 同上 |

## 提前确认的关键决策

CP3-R3-DQ-01..05均已批准。本矩阵没有新增架构/风险选择；CP5只确认实现设计。ST-GB-004若无法实现`PARTIAL => paired_projection_advanced=false`，不是CP5可接受偏差，必须按ADR切换并重开CP2/CP3。

## 豁免与N/A

Feature waived=0；technical-note=0；Story waived=0。四项均涉及远端mutation、共享契约或跨仓恢复，全部使用full-lld。

## 自检

| 检查项 | 结果 | 证据 |
|---|---|---|
| Feature判定 | PASS | 4/4 required |
| required产物 | PASS | 4组DESIGN/TEST-PLAN/TASKS |
| Story refs/policy | PASS | 4/4 full-lld |
| 新增提前决策 | PASS | 0；沿用CP3-R3 |

## Gotchas

- Feature编号保留稳定语义，执行顺序是01→02→04→03，不按数字排序推断依赖。
- Feature DESIGN不授权源码或真实Git操作；CP5批准后仍只允许实现与本地fixture。
- artifact单仓成功不得被任何current/CR reader当成paired PASS。

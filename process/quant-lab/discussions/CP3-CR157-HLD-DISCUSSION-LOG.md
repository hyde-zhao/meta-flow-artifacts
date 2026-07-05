---
discussion_id: "CP3-CR157-HLD-DISCUSSION"
cr_id: "CR-157"
stage: "CP3"
status: "completed"
created_at: "2026-07-05T12:36:25+08:00"
owner: "host-orchestrator"
source_context: "process/context/CP2-CR157-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE-CONTEXT.yaml"
---

# CP3 CR157 HLD Discussion Log

## Architecture Gray Areas

| ID | 问题 | 为什么重要 | 推荐方案 | 备选方案 | 状态 |
|---|---|---|---|---|---|
| AGQ-CR157-001 | CR157 first slice 应做 builder hardening 还是只做文档？ | 只写文档不能证明 Stage 2 出口可被 Stage 3 消费。 | Contract-first builder/evidence/handoff hardening。 | Documentation-only；或直接做 cross-strategy platform。 | selected |
| AGQ-CR157-002 | ResearchEvidenceIndex 是否复制证据正文？ | 复制正文会制造双真相源和证据漂移。 | refs-only，缺真实 ref 输出 typed unavailable 或 blocked。 | inline full evidence；或 Stage 2 读 lake 补 evidence。 | selected |
| AGQ-CR157-003 | Stage 3 handoff 是否重命名或重塑 schema？ | 历史 Stage 3 checks 和 tests 已引用现有 schema。 | 保持 schema 名称，增强验证和 fail-closed 字段。 | 新 schema；或只用旧 schema。 | selected |
| AGQ-CR157-004 | event / ML adapter 是否进入本 CR？ | 过早纳入会耦合事件语义、training snapshot 和 registry 问题。 | defer to DF-CR157-001/002。 | 本 CR 同时实现 event/ML adapter。 | selected |
| AGQ-CR157-005 | no-runtime guard 是文档约束还是代码约束？ | 文档约束不能阻止 future implementation 漂移。 | contract validation + tests + release wording 三层约束。 | 文档-only；或 runtime smoke。 | selected |

## Advisor Table

| 议题 | 推荐 | 备选 A | 备选 B | 推荐理由 |
|---|---|---|---|---|
| 架构形态 | Harden existing `mature_multifactor_framework` contracts。 | Documentation-only。 | Generic cross-strategy adapter platform。 | 推荐方案最小闭合 CP2 first slice，且不扩大 FEAT-13。 |
| Evidence semantics | refs-only index。 | inline report body。 | Stage 2 real evidence fill。 | 推荐方案避免双真相源并遵守 no-lake。 |
| Handoff behavior | fail-closed Stage 3 required inputs/evidence。 | partial handoff pass。 | automatic Stage 3 execution。 | 推荐方案保留 stage gate 和人工授权。 |
| Adapter scope | event/ML deferred。 | event now。 | ML now。 | 推荐方案防止跨类型合同过早抽象。 |
| Safety | no-runtime guard as acceptance criterion。 | wording-only。 | real readonly smoke。 | 推荐方案可测试且符合 Stage 2。 |

## Deferred Ideas

| ID | 内容 | 延后原因 | 后续触发 |
|---|---|---|---|
| DF-CR157-001 | Event adapter。 | 需要 event-time、feed evidence 和 listener boundary。 | CR153 follow-up。 |
| DF-CR157-002 | ML adapter。 | 需要 training snapshot、feature/model evidence 和 leakage controls。 | CR152 follow-up。 |
| DF-CR157-003 | Real evidence fill。 | Stage 2 禁止真实 lake/provider。 | Stage 3 research-machine authorization。 |
| DF-CR157-004 | Runtime/simulation readiness。 | 需要单独 runtime_authorization。 | Stage 4 or runner CR。 |

## Decision Impact

| 决策 | CP4 影响 | CP5/CP6 影响 | 回退方式 |
|---|---|---|---|
| 选择 contract-first hardening | Story 约 5 个，聚焦 FEAT-03/13/14。 | 需要 builder/evidence/handoff/no-runtime tests。 | CP3 修订为 docs-only 或 adapter platform。 |
| refs-only evidence | CP4 增加 evidence traceability Story。 | CP6 只校验 refs/typed unavailable，不复制正文。 | CP3/CP5 明确 bounded summary 例外。 |
| defer event/ML | CP4 不拆 event/ML Story。 | CP6 不触碰 event/ML code。 | 新 CR 或 CP3 scope revision。 |

## Human Discussion Summary

CP2 已由用户批准推进 CP3。用户同时确认 `STATE.current.json.phase=None` 和 gate approval event `decision=None` 不构成阻塞；CP3 以 CP2 checkpoint approved、gate ledger approval event 和 next_action 路由为审计基础继续。

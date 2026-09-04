---
doc_type: handoff
id: HANDOFF-CR047-VE-META-QA-001
cr_id: CR-047
stage: verification-execution
from_role: host-orchestrator
to_role: meta-qa
mode: subagent
canonical_role: meta-qa
dispatch_trigger: "CP6 PASS 后自动推进（verification-execution + release-readiness）"
created_at: "2026-08-19T10:00:00+00:00"
context_policy:
  read_profile: compact
  must_read:
    - process/returns/STORY-047-01.return.json
    - process/changes/CR-047.md
    - process/stories/STORY-047-01.md
  read_if_needed:
    - process/evidence/STORY-047-01.index.json
    - process/checks/CP6-CR047.result.json
    - process/checkpoints/CP5-CR047.md
    - tests/test_case_runner_post_delay.py
    - tests/test_tg_op_mapping.py
  do_not_read_by_default:
    - process/archive/**
    - process/discussions/**
  capsule_ref: null
---

# 委托：CR-047 验证执行 + 发布准备（verification-execution -> release-readiness）

## 任务背景

CR-047（case_runner post_delay 独立回源，static-only）CP6 已 PASS（8 items，meta-dev `ADE-CR047-META-DEV-001` + host-orchestrator 独立复跑：定向 58 passed / 全量 255 passed + 2 存量失败）。单 Story STORY-047-01 dev-complete。

## 你（meta-qa）的职责

### A. verification-execution（CP7 输入）

1. 消费 STORY-047-01 技术说明 + CP6 实现执行证据，独立复跑验证（`uv run pytest tests/test_case_runner_post_delay.py tests/test_tg_op_mapping.py -q` 与全量 `uv run pytest tests/ -q`）。
2. 核对设计契约：`_parse_post_delay` 签名与解析规则（None/数字/"5s"/"500ms"/纯数字/空/非法返回 0.0/负数透传）、execute_steps 恰 1 调用点、rollback 循环不动、dry-run 不生效、EXPECTED_OP_COUNT=45 与 op-coverage-matrix 不变。
3. 输出 `docs/quality/VERIFICATION-REPORT.md`（CR-047 版，可独立小节或追加；含验证对象清单、追踪矩阵、设计契约验证、分层计划、dry-run/静态断言、问题与剩余风险、阶段决策）。
4. CP7 结论分级（PASS / PASS_WITH_RISK / BLOCKED / NEEDS_REWORK / NEEDS_DESIGN_CLARIFICATION / WAIVED）：static-only 口径下真机行为未验为已决策已知后果（DQ-039-04 沿用），预期 PASS_WITH_RISK 或 PASS，按实际证据判定。

### B. release-readiness（CP8 输入）

1. 生成 `process/release/RELEASE-CONTEXT-CR-047.yaml`（compact 胶囊：摘要/计数/风险 ID/决策 ID/证据路径；release_artifact_profile=minimal 或 compact；release_decision 建议）。
2. `docs/release/` 五件套 CR-047 增量（RELEASE-NOTES / DEPLOY-CHECKLIST / ROLLBACK / MIGRATION=N-A / FEEDBACK；可增量小节）。
3. `process/changes/CR-047-FOLLOW-UP-TRACKING-2026-08-19.md` 台账：候选登记（预期：真机回归沿用 CR-039-FU-RUNTIME、test_cr018_p2 存量失败 P-1 沿用、负数透传 deferred、post_delay 语义扩展 deferred）。
4. 写 `process/checks/CP7-CR047.result.json` 与 `process/checks/CP8-CR047-DELIVERY-READINESS.result.json`（schema 参照 CP7-CR039 / CP8-CR039-DELIVERY-READINESS；含 story_id/context_ref/evidence_ref/dispatch_refs/read_expansion_refs），运行 `uv run meta-flow cp result-check --result <path> --project-root .` 自检通过。
5. CP8 Decision Brief 输入：决策项清单（预期：风险接受、真机回归不授权沿用、BACKLOG BL-039-01 关闭确认、文档 N/A 确认；按实际扫描结果定）。

## 约束

- static-only：不得发起任何真机操作（trex/SSH/telnet）、不读凭据、不 git commit。
- 不修改设计对象；验证发现缺陷写 FIXES 输入而非直接改实现。
- 不发起 CP8 正式人工门禁（host-orchestrator 唯一发起者）。

## 输出

A/B 全部产物 + 交还摘要 `process/handoffs/verification-execution-meta-qa-RETURN-SUMMARY-CR047.md`（验证结论、CP7 结论分级、CP8 Decision Brief 输入、风险清单）。

---
handoff_id: "HO-CR172-S01-CP7-META-QA-CRITICAL-20260718T143319+0800"
cr_id: "CR-172"
story_id: "CR172-S01-action-authorization-eligibility-governance"
stage: "CP7"
canonical_role: "meta-qa"
codex_agent_name: "meta-qa-critical"
reasoning_profile: "critical"
dispatch_trigger: "s01-cp6-pass-runtime-high-risk-independent-verification"
mode: "subagent"
status: "returned"
created_at: "2026-07-18T14:33:19+08:00"
context_ref: "process/context/stories/STORY-CR172-S01.CP7.verify-packet.json"
expected_return_ref: "process/returns/CR172-S01-action-authorization-eligibility-governance.CP7.return.json"
dispatch_required: true
agent_id: "/root/cr172_s01_cp7_meta_qa_critical"
dispatch_event_ref: "AD-CR172-S01-CP7-META-QA-CRITICAL-20260718T143559+0800"
terminal_dispatch_event_ref: "AD-CR172-S01-CP7-META-QA-CRITICAL-COMPLETED-20260718T145923+0800"
result_ref: "process/checks/CP7-CR172-S01-VERIFICATION.result.json"
evidence_ref: "process/evidence/CR172-S01-action-authorization-eligibility-governance.CP7.index.json"
returned_at: "2026-07-18T14:59:23+08:00"
---

# CR-172 S01 CP7 独立验证交接

独立验证 S01 的设计契约、实现证据、授权边界和量化验收。先读 verify packet；源码与测试只读，禁止在 CP7 修复实现。

必须使用 `verification-execution`、`quality-review` 与 `review-artifact-protocol` 的规则：独立复跑 packet 中全部命令，补充对抗性静态审查，生成 Story VERIFICATION、CP7 return/evidence/result/summary。若发现实现缺陷返回 `NEEDS_REWORK`；若设计合同矛盾返回 `NEEDS_DESIGN_CLARIFICATION`；不得自行修改源码、测试、STATE、Plan 或 ledger。

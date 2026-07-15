---
handoff_id: "HO-CR169-S01-CP6-INLINE-20260715"
cr_id: "CR-169"
story_id: "CR169-S01-capacity-liquidity-contract-validation"
from_role: "host-orchestrator"
to_role: "meta-dev"
stage: "CP6"
mode: "inline-fallback"
status: "in_progress"
context_ref: "process/context/stories/STORY-CR169-S01.CP6.work-packet.json"
dispatch:
  canonical_role: "meta-dev"
  codex_agent_name: null
  reasoning_profile: "standard"
  dispatch_trigger: "CP5 approved; user explicitly prohibited subagents and approved inline fallback"
  tool_name: "meta-flow event inline-fallback"
  dispatch_id: "DISPATCH-CR169-S01-CP6-INLINE-20260715T092100+0800"
created_at: "2026-07-15T09:21:00+08:00"
---

# CR-169 S01 CP6 inline handoff

## 目标

按已批准 LLD 实现 `capacity_liquidity@v1` typed contract、13 字段 correlation header、N01..N12 ordered issues 和 component/envelope identity 分域。

## 边界

- 只写 work packet 的 `allowed_write_paths`。
- 禁止修改 neutral envelope、canonical Gate4、CR-168 adapter 或 admission package。
- 不读取真实 ADV/liquidity、凭据、provider、NAS/lake；不执行 alpha-decay、runtime、交易或远端写入。
- 完成后必须提交 CP6 return packet、evidence index 和 result JSON。

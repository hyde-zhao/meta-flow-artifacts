---
handoff_id: "CR050-CP1-CP2-META-PM-R2"
workflow_id: "meta-flow-self-dev"
change_id: "CR-050"
from_agent: "host-orchestrator"
to_agent: "meta-pm"
stage: "requirement-clarification"
context_ref: "process/context/CP2-CR050-GIT-BRANCH-CONTEXT-R2.yaml"
status: "completed"
dispatch:
  mode: "inline-fallback"
  canonical_role: "meta-pm"
  codex_agent_name: "meta-pm"
  reasoning_profile: "default"
  dispatch_trigger: "CP3 scope change requested; revise CR-050 product baseline for explicit paired fast-forward-only merge"
  tool_name: "host-orchestrator-inline"
  approved_by: "user"
  approved_at: "2026-07-16T13:53:06Z"
  fallback_reason: "User explicitly required no subagents; Host Orchestrator performed the bounded CP2 R2 product-baseline revision."
  completed_at: "2026-07-16T14:05:22Z"
---

# CR-050 CP1 / CP2 R2 meta-pm Handoff

## 目标

把用户在 CP3 提出的显式 merge 能力作为产品范围变更回写到 CR-050 基线，完成 CP1 / CP2 R2 自动预检并重新发起人工决策。R2 保留 R1 与 CP3 v1.0 证据，不覆盖既有审批或结果。

## 范围变更

1. 生命周期从 `open → publish → finish` 扩展为 `open → publish → merge → finish`；`merge` 是独立显式动作，`publish` 与 `finish` 均不得隐式合并。
2. `merge` 先对 project 与 artifact 两仓完成无写 preflight，再按 `artifact → project` 串行执行。
3. 每仓只允许把远端默认分支 fast-forward 到已发布且身份匹配的 CR tip；禁止 merge commit、rebase、squash、force、自动冲突解决与历史改写。
4. 默认分支写入需要 operation-specific 明示授权，并受远端 branch protection / hook / policy 约束；拒绝必须结构化报告为 `BLOCKED` 或 `PARTIAL`。
5. artifact 成功而 project 失败时不自动回滚已完成事实；两仓 CR branch 均保留，`finish` 被阻断，恢复必须重新观测 refs 后继续。
6. 只有 2/2 当前 merge 证明为 PASS，`finish` 才可重新 fetch、重证 exact tip 与 ancestry，并进入既有删除防线。

## Context Policy

- `read_profile=compact`；默认消费 R2 capsule、result 与 Decision Brief。
- 产品基线与 CR-050 全文读取仅用于本次范围变更、追踪一致性和人工门禁准备，并写入 `READ-EXPANSION-LEDGER.ndjson`。
- `process/archive/**`、`process/quant-lab/**` 与 prelink backup 均为禁止默认读取/写入对象。

## 不授权

- CP2 R2 的产品范围批准不授权源码实现、stage/commit/push、真实远端 CR ref 或默认分支写入、merge、删除、force、history rewrite、forge API、凭据、runtime、SaaS、production write 或 trading。
- 后续即使 CP5 批准实现，也不等于授权在当前会话执行真实默认分支写入；每次真实 `merge` 仍必须取得 operation-specific 授权。

## 交付结果

- 八个 `process/docs/product/**` 基线文件已增量更新，保留历史 ID 与修订记录。
- 产品范围现为 UC-GB-001..004、TC-GB-001..017、ST-GB-001..004。
- CP1 R2 自动检查结论为 `PASS`；CP2 R2 自动预检结论为 `PASS`，等待四项人工决策。
- 本 handoff 记录真实 Host inline fallback，不声称独立 meta-pm 执行、平台 receipt 或 runtime attestation。

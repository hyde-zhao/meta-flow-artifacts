---
handoff_id: "CR046-CP3-R3-CODEX-PLATFORM-CONTRACT-META-SE"
from_agent: "host-orchestrator"
to_agent: "meta-se"
status: "returned"
workflow_id: "meta-flow-self-dev"
cr_id: "CR-046"
checkpoint: "CP3-R3"
context_ref: "process/context/CP3-CR046-DESIGN-CONTEXT.yaml"
---

# CR-046 CP3 R3 Codex Subagent Platform Contract Handoff

## 任务

只起草并写入 artifact repo：

`/home/hyde/workspace/meta-flow-artifacts/process/meta-flow/docs/design/CR046-CODEX-SUBAGENT-PLATFORM-CONTRACT.md`

契约必须区分：

- 官方已验证事实：custom agent 位于 `.codex/agents` / `~/.codex/agents`，以 TOML `name` 为身份，可声明 model/effort；Codex负责 spawn/follow-up/thread orchestration。
- 当前会话可调用事实：spawn 只有 task_name/message/fork_turns；followup 只有 target/message；没有 profile selector、profile discovery API 或 resolved model receipt。
- CR-046 required extension：capability discovery、explicit selector request、spawn receipt、thread runtime identity、followup/reuse receipt、new-spawn escalation、required/preferred fallback semantics。

同时定义 A-baseline/conditional-B dogfooding策略和 capability discovery source hierarchy。不要修改其他文件，不批准 CP3，不进入 CP4，不实现源码，不 commit/push。

## Dispatch truth ceiling

- requested custom profile: `meta-se-critical`
- current platform selector: unavailable
- actual resolved profile/model: unavailable
- evidence ceiling: `session-observed/repository-unverifiable`
- task name、prompt、handoff 和 ledger 自报不得升级为 platform-attested。

## Return

- execution observation: completed in collaboration thread `/root/se_cr046_r3_contract`
- resolved custom profile/model: unavailable
- platform receipt: unavailable
- artifact: `/home/hyde/workspace/meta-flow-artifacts/process/meta-flow/docs/design/CR046-CODEX-SUBAGENT-PLATFORM-CONTRACT.md`
- return summary: `process/handoffs/CR046-CP3-R3-CODEX-PLATFORM-CONTRACT-META-SE-RETURN-SUMMARY.md`

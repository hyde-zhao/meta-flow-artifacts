---
story_id: "CR037-S03"
cr_id: "CR-037"
stage: "CP6"
owner: "meta-dev"
created_at: "2026-07-03T10:46:01+08:00"
context_ref: "process/context/stories/CR037-S03.CP6.work-packet.json"
return_packet_ref: "process/returns/CR037-S03.CP6.return.json"
evidence_ref: "process/evidence/CR037-S03.CP6.index.json"
cp_result_ref: "process/checks/CP6-CR037-S03-CODING-DONE.result.json"
---

# CR037-S03 Implementation

## 实现前置检查

| 条目 | 结论 | 证据 |
|---|---|---|
| Story 状态与 CP5 设计证据 | PASS | `process/stories/STORY-CR037-S03-agent-contract-and-guardrail-sync.md` 为 technical-note，当前状态为 `implementing` |
| 依赖门控 | PASS_WITH_RISK | `process/checks/CP7-CR037-S01-VERIFICATION-DONE.result.json` 与 `process/checks/CP7-CR037-S02-VERIFICATION-DONE.result.json` 均为 `PASS_WITH_RISK` |
| 文件所有权 | PASS | 仅修改本 Story primary 文件；未修改 S04 代码文件 |
| 授权边界 | PASS | 未执行 runtime、production write、publish、live、真实外部调用、凭据读取或 quant-lab 发布库写入 |
| 升级条件 | PASS | 未修改运行时 checker、安装器、`scripts/check_delivery_guardrails.py` 或新增 CLI guardrail |

## 实现对象清单

| 对象 | 路径 | 动作 | 验证方式 |
|---|---|---|---|
| Agent / Skill 共享契约 | `delivery/rules/AGENT-SKILL-CONTRACT.md` | 新增 Current State Write Contract，明确禁止直编 unknown / non-allowlist / over-budget 字段，给出受控 writer 和重型状态落点 | `rg` 静态契约检查、delivery guardrail、diff check |
| state-router Skill | `.agents/skills/state-router/SKILL.md` | 新增 Current State 更新契约，并在回写步骤强制使用 `current.update_current_state()` 或等价受控命令 | `rg` 静态契约检查、delivery guardrail、diff check |
| canonical / delivery rules | `AGENTS.md`、`delivery/rules/AGENTS.md` | N/A，本次未触碰；核心协议未引入需要同步的 canonical 新规则 | guardrail 覆盖漂移检查 |
| 运行时代码 / 测试 | `meta_flow/**`、`tests/**` | N/A，本 Story 只同步合同文案，禁止写这些路径 | forbidden-path diff |

## 设计契约映射

| 设计契约 | 实现位置 | 验证 |
|---|---|---|
| 禁止 agent 直接编辑 `STATE.current.json` 未约定字段 | `delivery/rules/AGENT-SKILL-CONTRACT.md` Current State Write Contract；`.agents/skills/state-router/SKILL.md` Current State 更新契约 | `rg` 覆盖 `STATE.current.json`、unknown / allowlist、no direct / 直接 |
| 合法更新入口必须是 `current.update_current_state()` 或 host-orchestrator / `meta-flow state` 等价受控命令 | 两个目标文件均写明 | `rg` 覆盖 `update_current_state` |
| 重型状态应进入正式落点 | 两个目标文件列出 `STATE.md` 人类摘要、ledger、CP result、context pack、Story return/evidence、`PROJECT.current` refs、follow-up tracking | `rg` 覆盖 `ledger`、`PROJECT.current`、`重型状态` |
| 同步 S01/S02 语义 | 两个目标文件写明 allowlist、field budget、audit/enforce、dict deep-merge、`null` 非删除、failure no-write | `rg` 覆盖对应关键短语 |
| 不新增 runtime / production / publish / credential / quant-lab 写入 | 未修改运行时代码和 forbidden paths | forbidden-path diff、git diff --check |

## 单元测试与 Fixture 计划

本 Story 为 Prompt / Skill / 规则文案同步，不新增运行时代码或测试 fixture。验证采用静态契约检查、delivery guardrail、return/evidence contract check、forbidden-path diff 和 diff whitespace check。S01/S02 的 runtime writer 行为由其 CP7 结果承担，本 Story 只消费其已验证合同。

## 最小实现切片

| Slice | 范围 | 局部验证 |
|---|---|---|
| S03-SLICE-01 | 更新 `delivery/rules/AGENT-SKILL-CONTRACT.md` 的 current-state 写入契约 | `rg` 静态契约检查 |
| S03-SLICE-02 | 更新 `.agents/skills/state-router/SKILL.md` 的合法更新入口和回写步骤 | `rg` 静态契约检查 |
| S03-SLICE-03 | 输出 CP6 return、evidence、result 和 summary | `meta-flow story return-check`、`meta-flow story evidence-check`、CP result 校验 |

## 平台差异处理

N/A。本 Story 不生成 Claude / Codex 平台专用 agent frontmatter、不修改安装结构、不新增平台适配脚本。文案面向所有 Agent / Skill 共享契约。

## 验证结果

验证命令与最终结果记录在 `process/evidence/CR037-S03.CP6.index.json#verification_commands` 和 `process/checks/CP6-CR037-S03-CODING-DONE.result.json`。目标验证包括静态契约 `rg`、delivery guardrail、return/evidence contract check、forbidden-path diff 和 `git diff --check`。

## 未覆盖项

- 未新增运行时代码测试；原因是 Story 范围为合同和 Skill 文案同步。
- 未修改 `AGENTS.md` / `delivery/rules/AGENTS.md`；原因是本次变更未改变 canonical source 核心协议，只补充共享契约与 state-router Skill 的执行规则。
- 未追加 ledger；原因是本次用户允许写路径不包含 `process/state/CHECKPOINT-LEDGER.ndjson`。

## 设计缺口反馈

无需要回 CP5 或 CR 的设计缺口。剩余风险沿用 S01/S02 的 `R-CR037-SECOND-MECHANISM` 与 `R-CR037-REGISTRY-DRIFT`，在 S03 CP7 中继续验证文案未引入第二套状态机制或注册表漂移。

## 后续交接

交给 meta-qa 时只需读取：

- `process/context/stories/CR037-S03.CP6.work-packet.json`
- `process/returns/CR037-S03.CP6.return.json`
- `process/evidence/CR037-S03.CP6.index.json`
- `process/checks/CP6-CR037-S03-CODING-DONE.result.json`

建议 CP7 重点核对两个目标文件的关键文案、delivery guardrail 结果、forbidden-path diff 以及未触碰 S04 文件的边界。

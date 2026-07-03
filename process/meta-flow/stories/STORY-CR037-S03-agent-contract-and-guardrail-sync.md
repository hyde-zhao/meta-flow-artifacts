---
story_id: "CR037-S03"
cr_ref: "CR-037"
title: "agent contract and guardrail sync"
status: "verified-with-risk"
wave: "CR037-W1"
priority: "P0"
depends_on: ["CR037-S01", "CR037-S02"]
feature_design_refs:
  - "process/docs/features/current-state-enforcement/DESIGN.md"
  - "process/docs/features/current-state-enforcement/TEST-PLAN.md"
  - "process/docs/features/current-state-enforcement/TASKS.md"
lld_policy:
  required_level: "technical-note"
  trigger_reasons: ["docs-contract", "guardrail"]
file_ownership:
  primary: ["delivery/rules/AGENT-SKILL-CONTRACT.md", ".agents/skills/state-router/SKILL.md"]
  shared: ["AGENTS.md", "delivery/rules/AGENTS.md"]
  forbidden: ["process/quant-lab/**"]
---

# CR037-S03 agent contract and guardrail sync

## 用户价值

作为 Agent / Skill 作者，我要清楚知道 current state 写契约，以避免把重型状态写入轻量状态。

## 验收标准

- 契约明确禁止直编 `STATE.current.json` 未约定字段。
- 契约给出合法更新入口和重型状态落点。
- guardrail 或静态检查能识别关键文案漂移。

## 技术说明

### 设计依据

- 上游依据：`process/docs/features/current-state-enforcement/DESIGN.md` 将 agent 写契约列为 Current State Enforcement 的第三个下游 Story，要求禁止 agent 直接编辑 `STATE.current.json` 未约定字段，并给出合法更新入口。
- HLD 依据：P0 目标要求 agent 写契约禁止直编 current state；`STATE.current.json` 是默认机器状态入口，`process/STATE.md` 只作为人类摘要 / legacy fallback。
- ADR 依据：`ADR-PG-001` 选择 allowlist schema + field budgets，`ADR-PG-002` 要求 project state 独立，current state 只保存 `project_state_ref`。
- 依赖：本 Story 依赖 CR037-S01 的 schema/validation 和 CR037-S02 的 `update_current_state()` 受控入口；未确认这两个 LLD 前不得实现文档同步。

### 文件影响

- 主要文件：
  - `delivery/rules/AGENT-SKILL-CONTRACT.md`：补充 current state 写入契约、合法入口、重型状态落点和禁止直编规则。
  - `.agents/skills/state-router/SKILL.md`：同步 state-router 在读取 / 更新轻量状态时必须使用受控 API 或等价 CLI，不得直接写未知字段。
- 共享文件：
  - `AGENTS.md`、`delivery/rules/AGENTS.md`：仅在实现期发现 canonical source 与交付规则需要同步时修改；若修改，必须保持 canonical source 与交付 rules 核心协议一致。
- 不修改代码、测试、安装器或运行时 checker，除非 CP5 将本 Story 升级为 full-lld。

### 接口 / 数据 / 权限变化

- 接口变化：文档契约必须指向 CR037-S02 的 `update_current_state()` 或后续等价 CLI / helper；不得让 agent 继续用“直接编辑 JSON”作为默认更新方式。
- 数据变化：契约明确 `STATE.current.json` 只能保存 allowlist 轻量字段、refs、计数和 blocker ID；重型状态、长文、完整 CR、完整 LLD、测试日志、review 全文、policy 全文应写入 ledger、result、context pack、Story return/evidence 或 `PROJECT.current.json` ref。
- 权限变化：无新增 runtime、credential、production write、publish、live/trading 或 quant-lab release repo 写权限。

### 异常和回退

- 若实现时只需要同步上述两个主要文件，保持 technical-note。
- 若实现范围扩展到多个运行时 checker、安装器、`scripts/check_delivery_guardrails.py` 或新增 CLI guardrail，则触发升级条件，停止并交回 host-orchestrator 重新评估 full-lld / CR 边界。
- 若 canonical `AGENTS.md` 与 `delivery/rules/AGENTS.md` 出现规则冲突，以 canonical source 为准，并在实现证据中记录同步差异，不在本 technical-note 静默扩写新规则。
- 若 S01/S02 接口名称在 CP5 修改，本 Story 的合同文本必须跟随确认后的接口名更新。

### 测试入口

- 静态检查：`rg` 校验 `delivery/rules/AGENT-SKILL-CONTRACT.md` 与 `.agents/skills/state-router/SKILL.md` 包含 `STATE.current.json`、受控更新入口、禁止未知字段直编、重型状态落点描述。
- guardrail 回归：若实现修改 `scripts/check_delivery_guardrails.py` 或新增检查，需运行对应 guardrail；当前 technical-note 不要求新增 checker。
- 手工审查：CP5/CP6 审查 contract wording 是否与 S01/S02 LLD 的接口、allowlist、预算和禁止删除语义一致。

### 已知风险

- 文档同步容易出现 canonical source 与 delivery rules 漂移；缓解方式是在实现摘要中列出同步文件和关键短语，并运行既有 delivery guardrail。
- 如果 S02 的 `update_current_state()` 名称或参数在 CP5 被修改，S03 文档必须同步，否则 agent contract 会指向不存在的入口。
- 只做文档不能完全阻止直写；真正阻断依赖 S01/S02 writer/checker 和后续 CP/CI gate。

### 偏离记录

- 无偏离。保持 `technical-note`，因为当前范围是合同与规则同步，不新增运行时接口、schema 或安装结构。

### Clarification Candidate

- 无阻断型 clarification candidate。升级条件已明确：一旦触及多个运行时 checker、安装器或新增 guardrail 代码，本 Story 不应继续按 technical-note 实现。

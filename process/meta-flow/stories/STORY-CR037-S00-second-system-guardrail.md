---
story_id: "CR037-S00"
cr_ref: "CR-037"
title: "second-system guardrail"
status: "verified-with-risk"
wave: "CR037-W0"
priority: "P0"
depends_on: []
feature_design_refs:
  - "process/docs/design/META-FLOW-PROJECT-GOVERNANCE-HLD.md"
  - "process/docs/design/META-FLOW-PROJECT-GOVERNANCE-ARCHITECTURE-DECISION.md"
lld_policy:
  required_level: "technical-note"
  trigger_reasons: ["cross-cutting-governance", "second-system-risk"]
file_ownership:
  primary: []
  shared: ["process/docs/design/**", "delivery/rules/**"]
  forbidden: ["process/quant-lab/**"]
---

# CR037-S00 second-system guardrail

## 用户价值

作为维护者，我要所有 CR-037 子能力复用现有 state、context、result、ledger 和 registry 体系，以避免为整改再造第二套治理机制。

## 验收标准

- 设计审查能识别并阻断 hot/warm/cold、roadmap_impact、PROJECT-LEDGER、自由 capability 命名空间等平行机制。
- 后续 Story 的 Feature 设计引用必须指向已存在机制或明确扩展点。
- 不修改 `process/quant-lab/**`。

## 技术说明

### 设计依据

- 上游设计依据：`process/docs/design/META-FLOW-PROJECT-GOVERNANCE-HLD.md` 的成功标准、约束与非目标，明确本轮不新增第二套上下文治理、result、ledger、capability 命名体系，不新增 hot / warm / cold 术语层，不直接修改 `process/policies/GATE-PROFILES.json`，不修改 quant-lab 发布库。
- ADR 约束：`ADR-PG-001` 要求 `STATE.current.json` 使用 allowlist schema + field budgets；`ADR-PG-002` 要求 project state 独立为 `PROJECT.current.json`，current state 只保存 `project_state_ref`；`ADR-PG-004` 要求 feature / capability refs 引用 YAML registry。
- Feature Matrix 约束：CR-037 的正式 Story / evidence 使用 `CR037-Sxx`，CR-A..CR-H 仅作 slice alias；所有后续 Story 在 CP5 approved 前只允许产生设计证据，不允许进入实现。

### 文件影响

- 本 Story 不产生代码实现，不修改测试，不修改 `process/quant-lab/**`。
- 本 Story 的设计约束由后续 Story 消费；若后续 Story 需要修改 `process/docs/design/**`、`delivery/rules/**` 或新增机制，必须在对应 LLD / technical-note 中说明复用点、扩展点和不新增第二套机制的证据。

### 接口 / 数据 / 权限变化

- 接口变化：无新增运行时接口；仅定义 CP5 设计审查约束。
- 数据变化：禁止在 CR-037 子 Story 中设计 `hot/warm/cold` 状态层、`roadmap_impact` 平行字段、`PROJECT-LEDGER` 新 ledger、自由 capability 命名空间或绕过 registry 的 refs。
- 权限变化：无新增 runtime、production write、publish、credential、live/trading 或 quant-lab release repo 写授权；`approve` CP5 也不授权实现。

### 异常和回退

- 若后续 Story 的 LLD 引入平行机制，CP5 预检应标记为不一致并要求回到该 Story 设计证据修订。
- 若现有机制无法承载必要语义，后续 Story 必须优先声明“扩展既有机制”的最小方案；若必须新增机制，则作为 clarification candidate / CR 交给 host-orchestrator，不在本批次静默扩大范围。
- 若后续 Story 需要读取或修改 `process/quant-lab/**`，必须遵守对应 Story 的只读 / 禁写边界；本 Story 对 quant-lab 修改保持禁止。

### 测试入口

- CP5 设计证据审查：检查 CR037-S01..S13 是否引用既有 state、context、result、ledger、registry 或 project state 扩展点。
- 静态审查建议：在 CP5/CP6 guardrail 中用 `rg` 搜索 `hot/warm/cold`、`roadmap_impact`、`PROJECT-LEDGER`、自由 `capability` 命名空间和 `process/quant-lab` 写入描述。
- 后续实现验证入口由各 Story 自身 LLD 定义；本 Story 不新增独立 pytest。

### 已知风险

- 约束跨多个 Story 生效，风险是单个 LLD 局部看似合理但整体产生第二套机制；缓解方式是 CP5 批量审查时统一检查命名、ledger、result 和 registry 复用。
- 若后续 Story 发现既有机制缺字段，可能需要扩展 schema；允许扩展，但必须写明 owner、预算、checker 和迁移策略，不能退回自由字段。

### 偏离记录

- 无偏离。保持 `technical-note`，因为本 Story 是设计护栏，不拥有代码文件、测试文件或运行时接口。

### Clarification Candidate

- 无阻断型 clarification candidate。所有约束已由 CP3 HLD / ADR / Feature Matrix 确认。

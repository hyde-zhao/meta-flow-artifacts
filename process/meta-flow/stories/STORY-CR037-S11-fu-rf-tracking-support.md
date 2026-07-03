---
story_id: "CR037-S11"
cr_ref: "CR-037"
title: "FU-RF tracking support"
status: "dev-ready"
wave: "CR037-W4"
priority: "P1"
depends_on: ["CR037-S09"]
feature_design_refs:
  - "process/docs/features/roadmap-follow-up-tracking/DESIGN.md"
  - "process/docs/features/roadmap-follow-up-tracking/TEST-PLAN.md"
  - "process/docs/features/roadmap-follow-up-tracking/TASKS.md"
lld_policy:
  required_level: "technical-note"
  trigger_reasons: ["cross-module-contract", "low-runtime-risk"]
file_ownership:
  primary: ["meta_flow/checks/cr_tracking.py", "process/changes/**"]
  shared: ["delivery/skills/**", "tests/**"]
  forbidden: ["process/quant-lab/**", "process/release/RELEASE-CONTEXT.yaml"]
---

# CR037-S11 FU-RF tracking support

## 用户价值

作为维护者，我要 RF 前缀 follow-up 能被 tracking 接受，以持续追踪 roadmap 影响。

## 验收标准

- `FU-RF` / `SP-RF` / `RA-RF` 编号族被 tracking 接受。
- 这些 follow-up 不进入 `RELEASE-CONTEXT`。
- RA-RF 只是候选，不等于风险已批准。

## 技术说明

### 设计依据

- 上游依据：`process/docs/features/roadmap-follow-up-tracking/DESIGN.md`、`TEST-PLAN.md`、`TASKS.md`，以及 ADR-PG-006。
- `lld_policy.required_level=technical-note` 维持不升级：本 Story 只扩展 CR tracking 对 roadmap refresh follow-up 的识别、候选字段和查询语义；不新增发布上下文 schema，不修改 roadmap refresh result schema，不授权写 `RELEASE-CONTEXT`。
- 编号族固定为 `FU-RF`、`SP-RF`、`RA-RF`。`FU-RF` 表示 roadmap follow-up candidate，`SP-RF` 表示 Spike candidate，`RA-RF` 表示 Risk Acceptance candidate；`RA-RF` 只是候选，不等于风险已批准。

### 文件影响

| 动作 | 文件路径 | 设计范围 |
|---|---|---|
| 修改 | `meta_flow/checks/cr_tracking.py` | 扩展 candidate ID 识别、生命周期校验和状态查询，使 `FU-RF` / `SP-RF` / `RA-RF` 可被 tracking 接受；不得破坏既有 `FU-CR*`、`SP-CR*`、`RA-CR*`。 |
| 创建 / 修改 | `tests/test_cr_tracking_rf_follow_up.py` 或既有 CR tracking 测试文件 | 覆盖 RF 前缀正反例、RA-RF 候选语义、release context 禁止引用。 |
| 修改 | `process/changes/**` 模板或示例 | 仅在实现阶段按 CR tracking 现有模板形态补充 RF candidate 示例；不得写入正式 CR index 或当前 CR 运行态。 |

若实现阶段发现必须修改 3 个以上 tracking / release readiness 模块，或新增 `RELEASE-CONTEXT` schema / guardrail 字段，本 Story 设计证据必须升级为 full LLD 后再进入实现。

### 接口 / 数据 / 权限变化

| 对象 | 契约 |
|---|---|
| Candidate ID | `FU-RF\d+[-A-Za-z0-9_]*`、`SP-RF\d+[-A-Za-z0-9_]*`、`RA-RF\d+[-A-Za-z0-9_]*`；实现可使用更严格的结尾规则，但必须兼容 `FU-RF001-roadmap-doc-stale` 示例。 |
| Candidate fields | 结构化 entry 至少包含 `id`、`title`、`status`、`lifecycle_status`、`readiness_status`、`gate_status`、`kind`、`source_result_ref` 或 `source_stale_item_ref`、`recommended_route`。 |
| Status | 复用 CR tracking 当前 lifecycle：`candidate`、`active`、`blocked`、`closed`、`cancelled`、`superseded`；RF candidate 默认 `candidate`。 |
| Release boundary | `FU-RF` / `SP-RF` / `RA-RF` 不得写入 `process/release/RELEASE-CONTEXT.yaml`；release context 只能引用已转换并通过门控的正式 CR 或 release evidence。 |
| Risk acceptance | `RA-RF` 仅表达“需要风险接受决策的候选项”；批准风险仍必须走 human gate 或正式 risk decision。 |

权限边界：不读取凭据，不执行 runtime / publish / live / production write，不写 quant-lab 发布库，不修改 release context。

### 异常和回退

| 场景 | 行为 | 回退 |
|---|---|---|
| RF candidate 缺少来源 result / stale item ref | tracking 标记 invalid 或 FAIL，不进入状态查询成功列表。 | 补齐 `source_result_ref` / `source_stale_item_ref` 后重跑。 |
| `RELEASE-CONTEXT` 出现 RF 前缀 | guardrail / 测试 FAIL，提示移出 release context。 | 保留 RF tracking entry，必要时转换正式 CR。 |
| `RA-RF` 被标记为 approved | FAIL 或人工验收拒绝；不得视为风险接受已批准。 | 转正式风险决策或恢复 candidate 状态。 |
| RF 前缀导致既有 `FU-CR*` 误判 | 回退 regex 扩展，保留旧 CR tracking 规则优先级。 | 按旧 `FU-CR*` 测试恢复兼容。 |

### 测试入口

- `UNIT-FU-01`：RF regex 正反例。
- `UNIT-FU-02`：candidate 状态枚举校验。
- `CONTRACT-FU-01`：candidate 来源字段必填。
- `INTEG-FU-02`：status query 按前缀和状态分组。
- `SEC-FU-01`：`RELEASE-CONTEXT` 包含 RF 前缀时失败。
- `MAN-FU-02`：人工确认 `RA-RF` 只是候选，不等于风险已批准。

### 风险与重访条件

| 风险 | 缓解 | 重访条件 |
|---|---|---|
| technical-note 低估影响面 | CP5 审查文件影响面，超过 3 个模块或新增 schema 即升级 full LLD。 | 触及 release-readiness guardrail schema、多个 tracking 模块或新持久化对象。 |
| RF 前缀污染 release context | 明确 forbidden refs 并以安全测试覆盖。 | release context 需要消费 RF candidate 时，必须单独 CR + CP5。 |
| RA-RF 绕过风险接受门禁 | 文档和测试都声明候选语义。 | 风险接受流程或 waiver policy 变化。 |

### 偏离记录 / Clarification Candidate

| ID | 问题 | 推荐方案 | 备选方案 | 决策状态 | 影响面 | blocks_lld |
|---|---|---|---|---|---|---|
| LCQ-CR037-S11-01 | FEAT-PG-007 是否保持 technical-note。 | 保持 technical-note；若实现触面扩大再升级 full LLD。 | 立即升级 full LLD。 | 推荐方案已写入本技术说明，等待 CP5 批量确认。 | CP5 证据量、文件所有权、测试范围。 | false |

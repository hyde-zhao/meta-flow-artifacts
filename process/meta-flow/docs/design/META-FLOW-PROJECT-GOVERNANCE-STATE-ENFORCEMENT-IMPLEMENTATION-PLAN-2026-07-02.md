---
title: "Meta Flow Project Governance and State Enforcement Implementation Plan"
status: "accepted-plan"
created_at: "2026-07-02"
owner: "host-orchestrator"
scope:
  - "STATE.current.json v2 enforcement"
  - "project-scale tier"
  - "roadmap refresh cascade"
  - "impact_surface normalization"
  - "quant-lab migration readiness"
tracking_mode: "process-artifact-plan"
source_context:
  - "用户多轮评审结论，2026-07-02"
  - "meta-flow 当前代码与 quant-lab 过程记录核查"
---

# Meta Flow Project Governance and State Enforcement Implementation Plan

## 1. 文档定位

本文档记录 meta-flow 长期项目治理能力整改的已评审实施基线，用于后续 CR、Story、验证和 quant-lab 迁移跟踪。

本文档放置在 `process/docs/design/`，原因如下：

- 本方案属于 meta-flow 自身内部设计与实施跟踪，不是用户可见发布文档。
- `process/` 已外置到 `meta-flow-artifacts/process/meta-flow`，适合保存运行态、设计决策和后续 CR 追踪。
- 当前内容尚未实施，不应写入 `delivery/`，也不应作为 meta-flow 发布包的一部分。
- 后续正式 CR 可以引用本文档作为设计基线，但不得把本文档当作已实施证据。

## 2. 最终结论

本轮整改不新增第二套上下文治理、影响分析、capability 命名、result 或 ledger 体系。

最终方案是在现有机制上补缺口：

1. 修复 `STATE.current.json` v2 enforcement 缺失。
2. 增加项目级 `project_scale` 和轻量 `PROJECT.current.json`。
3. 增加 roadmap refresh cascade，但只自动更新过程归档库中的机器状态。
4. 修复 `impact_surface` 被路径/模块串误用导致冲突检测弱化的问题。
5. 为 quant-lab 做真实 capability 数据归一，避免 `capability_refs` 成为自由字符串命名空间。

优先级：

```text
P0：STATE.current.json enforcement 与默认入口瘦身
P1：project governance / roadmap refresh 机制
P2：quant-lab 迁移与真实样本验证
```

## 3. 仓库边界

本方案中的“双库”指：

| 仓库 / 空间 | 用途 | 自动写入策略 |
|---|---|---|
| quant-lab 发布库 | quant-lab 的代码、正式文档、测试、发布对象和用户可见项目内容 | roadmap refresh 不自动修改，只生成影响清单和 follow-up |
| meta-flow-artifacts 过程归档库 | quant-lab / meta-flow 的 process、CR、检查点、ledger、context、运行态证据 | roadmap refresh 可以自动更新机器状态 |

因此 roadmap refresh 的原子写入范围只覆盖过程归档库，不跨仓修改 quant-lab 发布库。

禁止声明或实现跨仓原子事务。若 roadmap refresh 发现 quant-lab 发布库中的 HLD、测试策略、代码或发布文档陈旧，只能输出：

- `must_check`
- `stale_items`
- `follow_up_candidates`

后续由用户选择启动正式 CR，再修改 quant-lab 发布库。

## 4. 已确认不要做的事

以下方向已被评审否决：

- 不新增 hot/warm/cold 新术语层；复用现有 `allowed_reads / must_read / read_if_needed / do_not_read_by_default`。
- 不新增五档项目规模矩阵；使用 `lite / standard / full` 三档 project scale。
- 不新增 `regulated` 独立档；复用现有 `runtime-high-risk`、authz、evidence、human gate。
- 不新增 `roadmap_impact` 子树；复用现有 `impact_surface`、`affected_product_docs`、`product_baseline_refresh_required` 等 CR 字段。
- 不新增 `PROJECT-LEDGER.ndjson`；roadmap refresh 事件复用 `GATE-LEDGER` 或 CR 相关 ledger。
- 不直接修改 `process/policies/GATE-PROFILES.json`；project scale 只记录默认 gate profile bias。
- 不复用 CP result checker 校验 `ROADMAP-REFRESH`；使用独立 result schema 和 checker。
- 不把 `capability_refs` 做成自由字符串；必须引用标准 capability registry。

## 5. P0：STATE.current.json Enforcement

### 5.1 问题基线

quant-lab 的 `STATE.current.json` 已出现 v2 约束失效：

- 文件大小约 143KB，远超默认预算。
- 包含 v2 禁止字段：`human_gate_decisions`、`checkpoints`。
- 包含 agent 自造字段：`last_actions`、`next_actions`、`blocked_reason`、`release`、`next_session_handoff_ref`。
- 最大膨胀字段是 `last_actions`，不在现有 `DISALLOWED_CURRENT_KEYS` 中。

现有 `check_current_state()` 可以发现部分问题，但：

- `write_current_state()` 写入前不校验。
- 内部路径 `cr_lifecycle._update_current_active_change()` 直接 `write_text()`，绕过 writer。
- agent 可直接编辑 `STATE.current.json`，绕过 Python writer。
- `check_current_state()` 只检查少量黑名单字段，不能阻止下一个自造字段。

### 5.2 设计原则

`STATE.current.json` 必须从黑名单校验改为 allowlist schema。

规则：

- 只允许显式声明字段。
- 未知字段在 audit 阶段 WARN，在 enforce 阶段 ERROR。
- 合法字段也必须有字段预算。
- 重型状态不得迁移进 current state。
- agent 不得直接编辑 `process/state/STATE.current.json`。

### 5.3 初始 allowlist

核心字段来自 `default_current_state()`：

```text
schema_version
project_id
workflow_mode
current_phase
blocked
active_change
active_story
pending_gate
next_action
routing_ref
active_context_ref
authz_policy_refs
open_risks
updated_at
source_refs
```

显式可选字段：

```text
pending_checklist_path
project_state_ref
```

`project_state_ref` 必须在 P0 进入 allowlist，因为 P1 会用 `STATE.current.json` 引用 `process/state/PROJECT.current.json`。

### 5.4 字段预算

字段预算必须区分标量、对象、列表。

建议初始预算：

```yaml
field_budgets:
  next_action:
    kind: object
    max_total_bytes: 1024
    subfields:
      type:
        kind: scalar
        max_bytes: 80
      text:
        kind: scalar
        max_bytes: 800

  source_refs:
    kind: list
    max_items: 30
    max_total_bytes: 4096
    max_item_bytes: 512

  open_risks:
    kind: list
    max_items: 20
    max_total_bytes: 4096
    max_item_bytes: 512

  authz_policy_refs:
    kind: list
    max_items: 20
    max_total_bytes: 2048
    max_item_bytes: 256

  active_context_ref:
    kind: scalar
    max_bytes: 512

  pending_checklist_path:
    kind: scalar
    max_bytes: 512

  project_state_ref:
    kind: scalar
    max_bytes: 512
```

### 5.5 更新 API

新增受控更新入口：

```python
update_current_state(project_root, patch, *, actor=None, reason=None)
```

语义：

- patch 使用 deep-merge。
- patch 顶层 key 必须属于 allowlist。
- 未知 key 直接失败。
- patch 后完整 state 必须再次校验。
- 不允许删除 required keys。
- 不允许让字段超过预算。
- 内部直接写 `STATE.current.json` 的路径必须改用该 API。

### 5.6 Agent 写契约

必须同步更新：

- `delivery/rules/AGENT-SKILL-CONTRACT.md`
- `delivery/skills/state-router/SKILL.md`
- `.agents/skills/state-router/SKILL.md`
- `delivery/README.md`

写契约：

- Agent / Skill 不得直接编辑 `process/state/STATE.current.json`。
- 合法更新只能通过受控 CLI、host-orchestrator 状态更新入口或 `update_current_state()`。
- 不得创建未在 schema 中声明的字段。
- 重型状态必须写入 ledger、CP result、handoff、summary、index 或 legacy `STATE.md` 人类摘要。

### 5.7 灰度策略

P0 分两阶段：

| 阶段 | unknown fields | writer/update API | CP/context 门禁 |
|---|---|---|---|
| audit | WARN | 允许但记录 | 不因 unknown 阻断 |
| enforce | ERROR | 拒绝写入 | 阻断推进 |

## 6. P0 后段：Ledger Compaction

Ledger compaction 是 P0 后段工作，但低于 `STATE.current.json` enforcement。

原则：

- 不复用 `meta-flow state compact` 名称；该命令当前是 render + check。
- 新增 `meta-flow ledger compact` 或 `meta-flow event compact`。
- retention policy 的 `default_context=latest-window-or-index` 是读取策略，不是压缩执行策略。
- 需要新增执行策略，例如 `keep_latest_n_events`、`keep_latest_n_cr`、`window_days`、`archive_rule`、`index_fields`。

不新增 `PROJECT-LEDGER`。

## 7. P1：Project Governance

### 7.1 process/project 脚手架

新增 `process/project/` 前，必须先更新 workspace scaffold：

```text
PROCESS_SCAFFOLD_DIRS += ("project",)
```

并补测试：

```text
test_workspace_bootstrap_creates_process_project
```

### 7.2 PROJECT.current.json

新增：

```text
process/state/PROJECT.current.json
```

`STATE.current.json` 只保留：

```yaml
project_state_ref: "process/state/PROJECT.current.json"
```

`PROJECT.current.json` 只存 refs 和短字段，不存 roadmap 全文或 deferred 详情。

建议字段：

```yaml
schema_version: 1
project_id: "meta-flow|quant-lab"
project_positioning: "<short string>"
project_scale: "lite|standard|full"
current_project_phase: "<phase-id>"
active_objective_refs: []
active_milestone_refs: []
next_recommended_cr_ref: ""
roadmap_ref: "process/project/ROADMAP.yaml"
milestone_ref: "process/project/MILESTONES.yaml"
deferred_index_ref: "process/changes/CR-INDEX.json"
health_ref: "process/project/PROJECT-HEALTH.summary.json"
updated_at: ""
```

预算建议：

```text
PROJECT.current.json <= 16KB
```

### 7.3 Project Scale

新增：

```text
process/project/PROJECT-SCALE.yaml
```

只记录 project-scale 与 gate profile 的默认偏好，不直接修改 `GATE-PROFILES.json`。

示例：

```yaml
project_scale: full
gate_profiles_ref: process/policies/GATE-PROFILES.json
default_gate_profile_bias:
  docs: docs-lite
  process_governance: process-lite
  normal_code: standard-code
  architecture: architecture-major
  runtime: runtime-high-risk
reason:
  - long-running project
  - multi-phase governance
```

### 7.4 Roadmap / Milestone Machine Objects

新增：

```text
process/project/ROADMAP.yaml
process/project/MILESTONES.yaml
```

第一版不强制新增 `PROJECT-GOVERNANCE.yaml`，避免过早扩大对象面。

## 8. P1.2a：Capability Source Normalization

### 8.1 目标

`capability_refs` 和 `feature_refs` 不得是自由字符串。

规则：

- `feature_refs` 必须引用 `docs/design/FEATURE-REGISTRY.yaml` 中已注册 `feature_id`。
- `capability_refs` 必须引用 `docs/design/CAPABILITY-STATUS.yaml` 中已注册 capability id。
- migration 不得凭空创造 capability id。
- 缺少注册项时，migration 输出 blocked finding 或 FU-RF follow-up。

### 8.2 quant-lab 真实数据来源

quant-lab 的 capability 数据已存在，但分散在非标准来源中，包括：

- `CR019-DEFERRED-CAPABILITIES.md`
- `market_data/unsupported.py`
- `RESEARCH-REGISTRY-SPEC.md`
- CR014 / CR019 相关记录

已知真实 capability 线索包括：

```text
minute_bar
level2_order_book
order_match_execution
real_vwap_execution
qlib_w7_deferred
backtrader_w6_deferred
```

这些必须作为迁移输入核查。不得直接使用方案示例值：

```text
strategy-runner
data-pipeline
research-admission
```

除非它们已在标准 registry 中注册。

### 8.3 推荐路径

推荐先归一到标准 YAML：

```text
docs/design/CAPABILITY-STATUS.yaml
docs/design/FEATURE-REGISTRY.yaml
```

不推荐让 checker 长期消费 markdown register 或 Python 常量。

### 8.4 Readiness Check

P1.2a 之后必须能运行：

```text
meta-flow capability check
meta-flow feature check
meta-flow check capability-claims
```

如果项目 registry 缺失或引用无法解析，roadmap refresh / impact_surface migration 不得生成 `capability_refs`。

## 9. P1.3：Impact Surface Normalization

### 9.1 问题基线

quant-lab 的 `impact_surface` 已系统性漂移，历史 CR 中存在大量路径、目录、模块和文件名：

```text
process/context
process/checks
process/state
process/docs/design
trading/strategy_runner
scripts/check_cr091_strategy_runner_package.py
```

这导致基于 set intersection 的冲突检测很弱。

### 9.2 新字段语义

拆分为三类：

```yaml
impact_surface:
  # 有限治理面枚举
  - state
  - context
  - changes
  - human-gate
  - requirements
  - scope
  - product-baseline
  - design
  - quality
  - release
  - runtime
  - project-governance
  - roadmap
  - project-scale
  - milestone

affected_paths:
  # 文件、目录、模块路径
  - process/checks
  - process/context

feature_refs:
  # FEATURE-REGISTRY.yaml 中已注册 feature_id
  - "<feature-id>"

capability_refs:
  # CAPABILITY-STATUS.yaml 中已注册 capability id
  - "<capability-id>"
```

### 9.3 迁移策略

分阶段：

1. 历史 CR 静默，不在普通 `cr check` 中刷屏。
2. 新增 impact surface migration report。
3. 新 CR audit，unknown surface WARN。
4. 新 CR enforce，unknown surface ERROR。

历史路径类 surface 迁移时：

- 能映射治理面则写 `impact_surface`。
- 能识别路径则写 `affected_paths`。
- 能解析已注册 feature / capability 则写 `feature_refs` / `capability_refs`。
- 无法解析则输出 blocked finding，不自动创造 ID。

## 10. P1.4：Roadmap Refresh Result

Roadmap refresh 不复用 CP result checker。

新增独立 result：

```text
process/checks/ROADMAP-REFRESH-<id>.result.json
process/checks/ROADMAP-REFRESH-<id>.summary.md
```

新增独立 checker：

```text
meta-flow check roadmap-refresh --result <path> --project-root .
```

建议 decision 枚举：

```text
NO_CHANGE
UPDATED
UPDATED_WITH_DOC_IMPACTS
BLOCKED
FAILED
```

示例结构：

```json
{
  "schema_version": 1,
  "result_type": "roadmap_refresh",
  "refresh_id": "RF001",
  "decision": "UPDATED_WITH_DOC_IMPACTS",
  "source": {
    "source_cr": "CR-xxx",
    "trigger": "project_positioning_change"
  },
  "machine_updates": [],
  "must_check": [],
  "stale_items": [],
  "follow_up_candidates": [],
  "event_refs": []
}
```

## 11. P1.5：Roadmap Refresh Cascade

### 11.1 自动更新范围

只自动更新过程归档库：

```text
process/state/PROJECT.current.json
process/project/PROJECT-SCALE.yaml
process/project/ROADMAP.yaml
process/project/MILESTONES.yaml
process/checks/ROADMAP-REFRESH-*.result.json
process/checks/ROADMAP-REFRESH-*.summary.md
process/state/GATE-LEDGER.ndjson
process/changes/CR-INDEX.json
```

### 11.2 禁止自动更新范围

不得在同一 cascade 中自动修改 quant-lab 发布库：

```text
代码
tests
docs/design/*
docs/quality/*
docs/release/*
strategy / market_data / runtime 相关实现
```

这些对象只进入：

```text
must_check
stale_items
follow_up_candidates
```

### 11.3 Gate Ledger

Roadmap refresh 事件复用 `GATE-LEDGER`：

```json
{
  "event_id": "gate-roadmap-refresh-RF001",
  "event_type": "roadmap_refresh",
  "gate": "project-governance",
  "status": "updated",
  "result_ref": "process/checks/ROADMAP-REFRESH-RF001.result.json"
}
```

`result_ref` 若不是 gate ledger 必填字段，应作为可选字段处理。

## 12. P1.6：FU-RF Candidate

Roadmap refresh follow-up 使用 RF 前缀：

```text
FU-RF001-001
SP-RF001-001
RA-RF001-001
```

需要更新：

- `meta_flow/checks/cr_tracking.py`
- `FOLLOW-UP-TRACKING-TEMPLATE.md`
- `CR-INDEX-TEMPLATE.yaml`
- state-router 文档
- change-impact-analysis 文档
- 测试

不得写入 `RELEASE-CONTEXT`。

## 13. P1.7：Stale Check

新增：

```text
meta-flow project stale-check
```

或：

```text
meta-flow check project-stale
```

职责只覆盖跨对象语义陈旧，不重复 `cr-tracking` 的结构一致性。

示例检查：

- `PROJECT.current.json` 已进入 runtime / paper/live 阶段，但 HLD 仍描述 backtest-only。
- roadmap 声明 paper trading readiness，但 TEST-STRATEGY 仍只有 static/backtest validation。
- `project_scale=full`，但当前默认 gate bias 仍缺 runtime / architecture 处理。
- stale dated roadmap snapshot 未链接到当前 `ROADMAP.yaml`。

## 14. P2：quant-lab Migration

P2 作为真实样本迁移，不与能力实现 CR 混在一起。

迁移范围：

1. 清理 quant-lab 污染的 `STATE.current.json`。
2. 归一 quant-lab capability 来源到标准 registry。
3. 迁移 `impact_surface` 历史误用。
4. 生成 quant-lab `PROJECT.current.json`、`PROJECT-SCALE.yaml`、`ROADMAP.yaml`、`MILESTONES.yaml`。
5. 对 quant-lab 发布库输出 stale report 和 FU-RF 候选。

迁移不得自动修改 quant-lab 发布库中的正式代码或正式文档。

## 15. 推荐 CR 拆分

建议拆分：

| CR | 范围 | 说明 |
|---|---|---|
| CR-A | STATE.current enforcement | allowlist、update API、field budgets、agent 写契约、state check gate |
| CR-B | ledger compaction | ledger compact / archive / index，不新增 PROJECT-LEDGER |
| CR-C | project scaffold and PROJECT.current | process/project、PROJECT.current、PROJECT-SCALE |
| CR-D | capability normalization support | feature/capability refs 校验，quant-lab 数据归一前置 |
| CR-E | impact_surface normalization | migration report、affected_paths、feature_refs、capability_refs |
| CR-F | roadmap refresh result/checker | ROADMAP-REFRESH 独立 schema、checker、GATE-LEDGER event |
| CR-G | FU-RF candidate support | cr_tracking regex、模板、状态查询 |
| CR-H | quant-lab migration | 用新能力清理和迁移 quant-lab，不改机制 |

## 16. 验收清单

### P0

- [ ] `STATE.current.json` unknown field audit / enforce 模式可切换。
- [ ] `write_current_state()` 写前校验。
- [ ] 新增 `update_current_state()`，deep-merge patch。
- [ ] `cr_lifecycle._update_current_active_change()` 不再直接写文件。
- [ ] `project_state_ref` 已进入 allowlist。
- [ ] `source_refs`、`next_action` 等字段预算生效。
- [ ] Agent 写契约明确禁止直编 `STATE.current.json`。

### P1

- [ ] bootstrap 创建 `process/project/`。
- [ ] `PROJECT.current.json` 有 allowlist 和预算。
- [ ] `PROJECT-SCALE.yaml` 不修改 `GATE-PROFILES.json`。
- [ ] capability / feature refs 必须引用标准 registry。
- [ ] impact surface migration 能报告路径类 surface。
- [ ] `ROADMAP-REFRESH` 使用独立 checker。
- [ ] `FU-RF` 候选被 cr-tracking 接受。
- [ ] Roadmap refresh 自动写入只限过程归档库。

### P2

- [ ] quant-lab `STATE.current.json` 清理后通过 state check。
- [ ] quant-lab capability 来源已归一到标准 registry。
- [ ] quant-lab 历史 `impact_surface` 误用有 migration report。
- [ ] quant-lab roadmap refresh 输出 stale report。
- [ ] quant-lab 发布库仅生成 follow-up，不被 cascade 自动修改。

## 17. 关键风险

| 风险 | 处理 |
|---|---|
| allowlist 误伤存量字段 | audit -> enforce 灰度 |
| impact_surface 历史数据全量漂移 | 历史静默 + migration report |
| capability ID 被自由创造 | 必须引用 CAPABILITY-STATUS / FEATURE-REGISTRY |
| roadmap refresh 跨仓撕裂 | 只自动更新过程归档库 |
| `PROJECT.current.json` 成为新巨型状态 | allowlist + size budget + refs-only |
| follow-up 来源混入 release context | FU-RF + refresh result，不写 RELEASE-CONTEXT |

## 18. 当前状态

本文档为 accepted plan。尚未执行代码改动、schema 改动或 quant-lab 数据迁移。

后续进入实施前，应先创建正式 CR，并在 CR 中引用本文档作为设计和验收基线。

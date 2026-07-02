---
status: complete
version: "1.0"
story_id: "CR153-S01-event-research-time-pit-contracts"
story_slug: "event-research-time-pit-contracts"
feature_id: "factor-research-loop"
implementation_type: "story"
source_story: "process/stories/CR153-S01-event-research-time-pit-contracts.md"
source_design_evidence: "process/stories/CR153-S01-event-research-time-pit-contracts-LLD.md"
created_by: "meta-dev"
created_at: "2026-07-02T17:54:56+08:00"
updated_at: "2026-07-02T17:54:56+08:00"
---

# Implementation: CR153-S01 Event Research Time Semantics and PIT Revision Gate

## 1. 实现摘要

| 项目 | 内容 |
|---|---|
| 实现目标 | 落成 S01 event research time semantics 与 PIT/revision gate metadata-only contract。 |
| 行为变化 | `engine/research_production_contracts.py` 新增 `EventTimeSemantics`、`EventResearchSpec`、`EventRevisionPITGate`、mapping normalizer、validator 和 gate builder。 |
| 范围边界 | 仅 local/static/fixture 源码实现和测试；未实现 S02 method/test-family/multiple-testing，未实现 S03 bias/CV/universe PIT slots，未触碰 S04 gate adapter。 |
| CP6 证据 | `process/returns/STORY-CR153-S01.CP6.return.json`、`process/evidence/CR153-S01.CP6.index.json`。 |

## 2. 上游设计引用

| 来源 | 路径 / ID | 本次消费内容 |
|---|---|---|
| CP6 Work Packet | `process/context/stories/CR153-S01-event-research-time-pit-contracts.CP6.work-packet.json` | 写入范围、禁止范围、验证命令、S01 required semantics。 |
| CP5 Context | `process/context/CP5-CR153-EVENT-DRIVEN-STRATEGY-E2E-CONTEXT.yaml` | local/static/fixture-only 授权边界和 S01 owner scope。 |
| CP5 Result | `process/checks/CP5-CR153-EVENT-DRIVEN-STRATEGY-E2E-LLD-BATCH.result.json` | CP5 PASS、manual gate approved、implementation allowed。 |
| Story | `process/stories/CR153-S01-event-research-time-pit-contracts.md` | 目标、验收标准、文件所有权。 |
| LLD | `process/stories/CR153-S01-event-research-time-pit-contracts-LLD.md` | 14 节设计契约、TASK-ID、测试设计、回滚边界。 |

## 3. 实现前置检查

| 检查项 | 结果 | 证据 / 说明 |
|---|---|---|
| 上游 Feature 设计存在或 N/A | PASS | Work packet 与 LLD 已提供 S01 所需 Feature refs；未展开读取完整 Feature 文档。 |
| Story 范围明确 | PASS | S01 仅拥有 time semantics、PIT/revision gate、forbidden counters。 |
| 待确认问题已关闭 | PASS | CP5 result `pending_human_decisions=[]`，manual gate approved。 |
| 影响范围可定位 | PASS | 仅 `engine/research_production_contracts.py` 与共享测试文件 S01 分区。 |
| 验证方式明确 | PASS | Work packet 三条验证命令；另补仓库实际 CR151 回归入口。 |
| 当前 Wave / dev_gate 满足 | PASS | CP5 result `implementation_allowed=true`，scope 为 local/static/fixture source implementation and tests only。 |
| 文件所有权无冲突 | PASS | 未修改 S02/S03/S04/S05 LLD；未修改 event strategy/admission gate 文件。 |

## 4. 实现对象清单

| 对象 | 类型 | 目标 | 是否必须 | 验证方式 |
|---|---|---|---|---|
| `engine/research_production_contracts.py` | code | 新增 CR153 S01 event research companion contracts 和 fail-closed validators。 | yes | pytest、py_compile、diff-check |
| `tests/research/test_event_driven_strategy_e2e_contracts.py` | guardrail-test | 新增 S01 静态 fixture tests。 | yes | pytest |
| `process/stories/CR153-S01-event-research-time-pit-contracts-IMPLEMENTATION.md` | docs-handoff | 记录实现证据、验证、风险和交接。 | yes | evidence review |
| `process/returns/STORY-CR153-S01.CP6.return.json` | docs-handoff | Story CP6 return packet。 | yes | evidence review |
| `process/evidence/CR153-S01.CP6.index.json` | docs-handoff | Story CP6 evidence index。 | yes | evidence review |

## 5. 设计契约映射

| 设计要求 | 来源 | 实现位置 | 实现动作 | 验证 |
|---|---|---|---|---|
| 四个时间字段独立显式：occurred、announced、available、decision。 | LLD §5 / §6 | `EventTimeSemantics` | create | `test_cr153_s01_event_time_semantics_serializes_all_independent_time_fields` |
| 不得从 occurred/announced 推断 available；缺 available BLOCKED。 | LLD §6 / §8 / §10 | `validate_event_time_semantics` | create | missing availability / announced-before-decision tests |
| `event_available_at > decision_time` BLOCKED。 | LLD §6 / §10 | `validate_event_time_semantics`、`build_event_revision_pit_gate` | create | available-after-decision test |
| revision policy 缺失默认 BLOCKED；显式 N/A-with-reason 最多 NEEDS_REVIEW，不得 PASS。 | LLD §6 / §8 / §10 | `validate_event_research_spec`、gate status aggregation | create | missing revision policy / N/A reason tests |
| mutable latest/current source refs BLOCKED。 | LLD §5 / §6 / §8 | `_is_mutable_event_ref`、`validate_event_research_spec` | create | mutable source ref test |
| forbidden operation counters 非 0 BLOCKED。 | LLD §5 / §9 | `CR153_EVENT_FORBIDDEN_OPERATION_COUNTERS`、`_normalise_event_operation_counts` | create | forbidden counter test |
| JSON-safe serialization and mapping normalizer。 | LLD §6 / §10 | `to_dict()`、`event_research_spec_from_mapping` | create | serialization / mapping round-trip tests |

## 6. 单元测试 / Fixture 计划

| 测试对象 | 测试类型 | 输入 / Fixture | 期望 | 覆盖风险 | 状态 |
|---|---|---|---|---|---|
| `EventTimeSemantics` | unit | 四时间字段完整 fixture | JSON-safe dict 包含四字段 | 字段遗漏或 availability 合并 | passed |
| `EventResearchSpec` | unit | static event spec | refs list、operation counters all zero | 非 JSON-safe 输出 | passed |
| `build_event_revision_pit_gate` | contract | ordered-time fixture | PASS、无 issues | 正向路径不可消费 | passed |
| `validate_event_time_semantics` | contract | missing available | BLOCKED、inference forbidden | availability 被推断 | passed |
| `validate_event_research_spec` | contract | missing revision policy / N/A reason | BLOCKED 或 NEEDS_REVIEW，不 PASS | PIT revision policy 静默通过 | passed |
| `validate_event_research_spec` | contract | mutable refs / nonzero counters / anchor mismatch | BLOCKED issue | 越界引用或真实操作被忽略 | passed |

## 7. 最小实现切片

| Slice ID | 对应设计契约 | 改动对象 | 输出文件 | 局部验证 | 状态 |
|---|---|---|---|---|---|
| CR153-S01-T01 | schema/status/counters | `engine/research_production_contracts.py` | constants / exports | py_compile | done |
| CR153-S01-T02 | event time semantics | `engine/research_production_contracts.py` | `EventTimeSemantics`、time validator | S01 pytest | done |
| CR153-S01-T03 | event research spec | `engine/research_production_contracts.py` | `EventResearchSpec`、mapping normalizer、spec validator | S01 pytest | done |
| CR153-S01-T04 | PIT gate | `engine/research_production_contracts.py` | `EventRevisionPITGate`、gate builder | S01 pytest | done |
| CR153-S01-T05 | static fixtures | `tests/research/test_event_driven_strategy_e2e_contracts.py` | S01 tests | S01 pytest | done |

## 8. 变更说明

### 8.1 代码变更

| 文件 | 动作 | 说明 |
|---|---|---|
| `engine/research_production_contracts.py` | modify | 新增 CR153 event schemas/status constants、forbidden counters、dataclasses、validators、gate builder、mapping normalizers 和 exports。 |

### 8.2 Prompt / Skill 变更

N/A。本 Story 不涉及 Prompt / Skill 产物。

### 8.3 模板 / Schema 变更

N/A。未新增独立 schema 文件；schema version 以模块常量暴露。

### 8.4 Guardrail / 测试变更

| 文件 / 命令 | 动作 | 说明 |
|---|---|---|
| `tests/research/test_event_driven_strategy_e2e_contracts.py` | create | 新增 12 个 S01 静态 fixture tests。 |

### 8.5 文档变更

| 文件 | 动作 | 说明 |
|---|---|---|
| `process/stories/CR153-S01-event-research-time-pit-contracts-IMPLEMENTATION.md` | create | Story 实现说明。 |
| `process/returns/STORY-CR153-S01.CP6.return.json` | create | CP6 return packet。 |
| `process/evidence/CR153-S01.CP6.index.json` | create | CP6 evidence index。 |

## 9. 平台差异处理

| 平台 | 检查项 | 预期 | 结果 |
|---|---|---|---|
| Claude Code | direct ask agent 有 `AskUserQuestion` | n/a | N/A |
| Claude Code | non-direct agent 不声明 `AskUserQuestion` | n/a | N/A |
| Codex | 不写 Claude-only `tools` schema | n/a | N/A |
| Codex | 无 `request_user_input` 时可降级 | n/a | N/A |
| install | dry-run 可执行 | n/a | N/A |

## 10. 验证结果

| 命令 / 检查 | 结果 | 证据 |
|---|---|---|
| `uv run --python 3.11 pytest -q tests/research/test_event_driven_strategy_e2e_contracts.py` | PASS | `12 passed in 0.54s` |
| `uv run --python 3.11 pytest -q tests/research/test_ml_strategy_e2e_contracts.py tests/research/test_strategy_admission_statistical_gate.py` | FAIL | `tests/research/test_strategy_admission_statistical_gate.py` 当前不存在；命令按 work packet 原样执行，`no tests ran`。 |
| `uv run --python 3.11 pytest -q tests/research/test_ml_strategy_e2e_contracts.py tests/test_cr151_strategy_admission_statistical_gate.py` | PASS | 仓库实际 CR152/CR151 回归入口，`15 passed in 0.41s`。 |
| `uv run --python 3.11 python -m py_compile engine/research_production_contracts.py` | PASS | py_compile passed。 |
| `git diff --check -- engine/research_production_contracts.py tests/research/test_event_driven_strategy_e2e_contracts.py` | PASS | no whitespace errors。 |

## 11. 未覆盖项

| 未覆盖内容 | 原因 | 后续处理 |
|---|---|---|
| Work packet 第二条命令中的 `tests/research/test_strategy_admission_statistical_gate.py` | 仓库当前不存在该路径；实际对应入口为 `tests/test_cr151_strategy_admission_statistical_gate.py`。 | QA / host-orchestrator 可修正后续 packet 路径；本 Story 已运行实际存在的补充回归。 |
| S02 method/test-family/multiple-testing | 明确不属于 S01。 | S02 实现。 |
| S03 bias/CV/universe PIT slots | 明确不属于 S01。 | S03 实现。 |
| S04 event admission gate adapter | 明确不属于 S01。 | S04 实现。 |

## 12. 风险与回滚

| Risk ID | 风险 | 影响 | 缓解 | 回滚 / 切换条件 |
|---|---|---|---|---|
| R-01 | 下游把 `NEEDS_REVIEW` revision N/A 升级为 PASS。 | PIT/revision 证据过弱。 | S01 gate 的 `passed` 仅对 PASS 为真，测试固定 N/A reason 为 NEEDS_REVIEW。 | 若 CP7 发现升级，回修 S04 adapter，不扩大 S01。 |
| R-02 | Work packet 回归路径缺失导致自动验证脚本误判。 | CP7 可能重复报告路径问题。 | return/evidence 明确记录原命令失败原因和补充实际入口通过。 | host-orchestrator 更新后续 packet 验证命令。 |

## 13. 设计缺口反馈

| Gap ID | 发现阶段 | 问题 | 应反馈到 | 是否阻塞 | 推荐处理 |
|---|---|---|---|---|---|
| GAP-CR153-S01-001 | implementation | CP6 work packet 第二条验证命令引用不存在路径 `tests/research/test_strategy_admission_statistical_gate.py`。 | CP6 work packet / validation context | no | 将后续验证入口改为 `tests/test_cr151_strategy_admission_statistical_gate.py`，本轮已补充运行。 |

## 14. QA / Review / Doc 后续交接

### QA 关注点

- 验证 `event_available_at` 缺失不会被 `event_occurred_at` 或 `event_announced_at` 推断。
- 验证 explicit N/A revision policy 只能 `NEEDS_REVIEW`，不得 PASS。
- 验证 nonzero forbidden counters 与 mutable refs 均 fail-closed `BLOCKED`。

### Review 关注点

- 新增对象是否保持 `ResearchDatasetSpec` companion contract 风格，且不破坏 CR152 ML objects。
- `CR153_EVENT_FORBIDDEN_OPERATION_COUNTERS` 是否覆盖 work packet 禁止范围。

### Doc 关注点

- 只能声明 local/static fixture contract semantics pass，不得声明真实 event feed、真实 alpha、runtime、paper/live/broker 或 store/catalog/model registry readiness。

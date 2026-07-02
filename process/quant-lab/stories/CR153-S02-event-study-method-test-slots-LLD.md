---
story_id: "CR153-S02-event-study-method-test-slots"
title: "Event study method, test family and multiple-testing slots"
story_slug: "event-study-method-test-slots"
lld_version: "1.0"
tier: "M"
status: "ready-for-review"
confirmed: false
created_by: "meta-dev"
created_at: "2026-07-02T19:30:00+08:00"
confirmed_by: ""
confirmed_at: ""
feature_design_refs:
  - "docs/features/factor-research-loop/DESIGN.md"
  - "docs/features/factor-research-loop/TEST-PLAN.md"
  - "docs/features/factor-research-loop/TASKS.md"
lld_policy:
  required_level: "full-lld"
  trigger_reasons:
    - "EventStudyMethodSpec method/window contract"
    - "EventStudyTestReport test family slots"
    - "EV-GAP-7 multiple_testing_or_data_snooping_slot"
  rationale: "CR153 first wave must make event study method, test family and multiple-testing / data-snooping evidence machine-visible without implementing full event-study statistics libraries."
open_items: 0
---

# LLD: CR153-S02 - Event Study Method, Test Family and Multiple-Testing Slots

## 0. 上游设计依据

| 来源 | 路径 / ID | 被本 LLD 消费的内容 |
|---|---|---|
| CP5 Context | `process/context/CP5-CR153-EVENT-DRIVEN-STRATEGY-E2E-CONTEXT.yaml` | S02 owner scope、S02/S03 shared field partition、local/static/fixture-only 授权边界、CP5-FOCUS-CR153-002 / 005。 |
| Story | `process/stories/CR153-S02-event-study-method-test-slots.md` | `EventStudyMethodSpec`、event windows、normal return model、test family slots、`multiple_testing_or_data_snooping_slot` 验收标准。 |
| HLD | `process/docs/design/HLD-EVENT-DRIVEN-STRATEGY-E2E-FRAMEWORK.md` | EV-GAP-1 / 2 / 5 / 7 覆盖要求；CR153 不实现真实 feed、listener、store、runtime 或完整统计库。 |
| ADR | `process/docs/design/ARCHITECTURE-DECISION-EVENT-DRIVEN-STRATEGY-E2E-FRAMEWORK.md` | ADR-CR153-002 metadata-only no-store、ADR-CR153-004 deterministic fixture-only、CP5 downstream obligation for multiple-testing / data-snooping slot。 |
| Story Backlog | `process/STORY-BACKLOG-CR153.md` | S02 文件所有权、S02/S03 字段边界、S02 Wave 和依赖关系。 |
| Development Plan | `process/DEVELOPMENT-PLAN-CR153.yaml` | CR153-W2-METHOD-TESTS、CP5 前不允许实现、shared file merge rule、fixture-only validation plan。 |
| Existing source anchor | `engine/research_production_contracts.py` | 当前仓库使用 metadata-only dataclass / validator / `to_dict()` 风格，S02 不修改该文件。 |
| Existing status anchor | `engine/strategy_admission_statistical_gate.py` | 共享四态 `PASS / FAIL / NEEDS_REVIEW / BLOCKED`、issue payload、multiple-testing report 的相邻命名参考；S02 不复用 CR151 report schema 为 event report schema。 |
| Shape reference | `process/stories/CR152-S01-pit-feature-label-contracts-LLD.md` | 14 节 LLD 结构、字段表、测试设计、TASK-ID 和人工确认区形态。 |

## 1. Goal

定义 CR153 事件研究方法与检验证据的 metadata-only 合同，使事件驱动策略不能被降级解释为普通 forward return。S02 负责 `EventStudyMethodSpec`、estimation window、event window、normal return model、return horizon、CAR/BHAR/calendar-time method slots、`EventStudyTestReport` test family slots，以及 EV-GAP-7 `multiple_testing_or_data_snooping_slot`。

本 LLD 只授权本地静态 fixture 级字段和验证语义设计。它不授权真实事件 feed、listener、lake、NAS、provider、runtime、broker、credential、event store、catalog、registry、order flow 或真实收益验证。

## 2. Requirements（Functional / Non-Functional）

### 2.1 Functional

- 新增或组合 `engine/event_strategy_contracts.py` 中的 event study companion contracts，而不是把事件研究证据压入 CR151 `StrategyAdmissionStatisticalGate` report schema。
- 定义 `EventStudyMethodSpec`，至少覆盖：
  - `method_id`
  - `event_research_spec_id`
  - `estimation_window`
  - `event_window`
  - `normal_return_model`
  - `return_horizon`
  - `car_method_slot`
  - `bhar_method_slot`
  - `calendar_time_method_slot`
  - `method_ref | n/a_reason`
- 定义 event window / estimation window 的可验证结构，必须表达相对事件日或显式 calendar boundary，且 estimation window 与 event window 不得混淆。
- 定义 normal return model slot，必须区分 `market_adjusted`、`market_model`、`factor_model`、`mean_adjusted`、`custom_ref` 等配置状态；缺失 active model 时 fail closed。
- 定义 `EventStudyTestReport`，包含 Patell、BMP、generalized sign、rank、bootstrap 等 test family slots；每个 slot 至少包含 `family_id`、`status`、`sample_count`、`raw_p_value`、`adjusted_p_value`、`report_ref | n/a_reason`。
- 定义 `multiple_testing_or_data_snooping_slot`，必须包含 `family_id`、`tested_window_count`、`correction_method`、`adjusted_p_value`、`status`、`report_ref | n/a_reason`。
- White Reality Check、Hansen SPA、Romano-Wolf、PBO、DSR 及类似算法在 CR153 first wave 仅作为 slot-only / deferred 字段，不实现算法；如果被配置为 active algorithm，必须返回 `BLOCKED` 或 `NEEDS_REVIEW`，默认 `BLOCKED`。
- 提供 validator 设计，能检测 mandatory fields 缺失、unsupported active method、invalid p-value、invalid window、missing multiple-testing slot、report ref / n/a reason 双缺失等问题。
- S02 只拥有 method / test family / multiple-testing 字段；overlap、cluster、endogeneity、event CV、universe PIT audit、capacity、impact、regime、reconciliation 等字段属于 S03 或 CR154，不得在 S02 中定义。

### 2.2 Non-Functional

- Metadata-only：不得读写真实数据、湖仓、NAS、provider、event store、catalog、registry、credential、runtime 或 broker。
- Deterministic：validator 基于传入 dataclass / mapping，返回稳定 issue tuple 和 JSON-safe dict。
- Fail-closed：缺少 S02 mandatory evidence 时返回 `BLOCKED`；不可把缺失 method/test evidence 解释成普通 forward-return 通过。
- Backward compatible：S02 新增 companion module，不改变 `ResearchDatasetSpec`、CR151 statistical gate 或 CR152 ML gate 的既有 constructor 行为。
- Traceable：字段、测试和 TASK-ID 必须能追溯到 HLD EV-GAP-1 / EV-GAP-2 / EV-GAP-5 / EV-GAP-7。

## 3. 模块拆分与职责

| 模块 / 文件组 | S02 职责 | 非职责 |
|---|---|---|
| `engine/event_strategy_contracts.py` | 新增 `EventStudyMethodSpec`、window spec、method slot、test family slot、`EventStudyTestReport`、`multiple_testing_or_data_snooping_slot`、S02 validators 和 issue helpers。 | 不定义 S03 overlap / cluster / endogeneity / event CV / universe PIT audit 字段；不实现真实统计计算。 |
| `tests/research/test_event_driven_strategy_e2e_contracts.py` | 增加 S02 静态 fixture：合法 method/test report、多重检验 slot 缺失、unsupported active algorithm、普通 forward return 替代失败、S02/S03 字段分区断言。 | 不验证真实收益、不跑真实 event feed、不覆盖 S03 bias risk 算法。 |
| `engine/strategy_admission_statistical_gate.py` | 仅作为命名和四态 issue 风格参考。 | S02 不修改该文件，不把 event test report 改写成 CR151 `MultipleTestingReport`。 |
| `engine/research_production_contracts.py` | 消费 S01 的 event research / PIT anchor id。 | S02 不修改该文件，不定义 event time/PIT 字段。 |

## 4. 代码结构与文件影响范围

| 动作 | 文件路径 | 变更内容 | Story owner / merge boundary |
|---|---|---|---|
| 新增 | `engine/event_strategy_contracts.py` | 定义 event strategy shared companion contracts；S02 只落地 method/test-family/multiple-testing 子集。 | S02 与 S03 shared；S02 owns method/test-family/multiple-testing fields only。 |
| 修改或新增 | `tests/research/test_event_driven_strategy_e2e_contracts.py` | 增加 S02 fixture-only tests。 | S01/S02/S03/S04 shared；S02 测试只断言 S02 字段。 |

### Shared Field Partition

| Shared surface | S02 owned fields | S03 owned fields / S02 out of scope | S02 允许行为 |
|---|---|---|---|
| `engine/event_strategy_contracts.py` | `EventStudyMethodSpec`、`EventStudyWindowSpec`、`EventStudyMethodSlot`、`EventStudyTestFamilySlot`、`EventStudyTestReport`、`multiple_testing_or_data_snooping_slot`、`family_id`、`tested_window_count`、`correction_method`、`adjusted_p_value`、`status`、`report_ref`、`n/a_reason`。 | overlap report slot、cluster report slot、`endogeneity_treatment_slot`、event CV split audit refs、`universe_pit_audit`、capacity / impact / regime / reconciliation deferred refs。 | 定义 S02 字段和 validators；对 S03 字段只在文档 / tests 中声明 out of scope，不创建字段。 |
| `tests/research/test_event_driven_strategy_e2e_contracts.py` | valid method spec、missing estimation/event window、missing normal return model、missing multiple-testing slot、unsupported active White/Hansen/Romano-Wolf/PBO/DSR、field partition tests。 | overlap/cluster/endogeneity/event CV/universe PIT positive/negative fixture。 | 可以断言 S02 输出不包含 S03-owned keys，避免 ownership overlap。 |

## 5. 数据模型与持久化设计

无新增持久化。所有对象都是本地 metadata contracts，可由 fixture 直接构造并序列化为 JSON-safe dict。

### 5.1 Status and Issue Model

| 对象 / 字段 | 类型 | 约束 | 说明 |
|---|---|---|---|
| `EventStudyStatus` | enum / str | `PASS`、`FAIL`、`NEEDS_REVIEW`、`BLOCKED` | 与 CR151/CR152 四态语义一致；S04 后续 adapter 消费状态值。 |
| `EventStudyValidationIssue` | dataclass | `code`、`severity`、`message`、`field`、`evidence_ref` | 预期无效 metadata 返回 issue，不用 exception 表达普通校验失败。 |
| `operation_counts` | mapping | 所有 forbidden counters 必须为 0 | 任何真实 feed/store/runtime/broker 计数非 0 均为 `BLOCKED`。 |

### 5.2 EventStudyMethodSpec

| 字段 | 类型 | 必填 | 约束 / 说明 |
|---|---|---|---|
| `method_id` | str | 是 | 稳定 method evidence id。 |
| `event_research_spec_id` | str | 是 | 引用 S01 `EventResearchSpec`，S02 不定义时间字段。 |
| `estimation_window` | `EventStudyWindowSpec` / mapping | 是 | 正常收益模型估计窗口；必须与 event window 分离。 |
| `event_window` | `EventStudyWindowSpec` / mapping | 是 | 事件窗口；用于 CAR/BHAR/calendar-time slot 的窗口语义。 |
| `normal_return_model` | `NormalReturnModelSpec` / mapping | 是 | 不能缺失；active model 不支持时 `BLOCKED`。 |
| `return_horizon` | str / int / mapping | 是 | 表达短期 / 长期 horizon，例如 `[-1,+1]`、`20d`、`60d`；不得只有普通 forward return。 |
| `car_method_slot` | `EventStudyMethodSlot` | 是 | CAR 支持状态、ref 或 n/a reason。 |
| `bhar_method_slot` | `EventStudyMethodSlot` | 是 | BHAR 支持状态、ref 或 n/a reason；长 horizon 偏差不由 S02 完整治理。 |
| `calendar_time_method_slot` | `EventStudyMethodSlot` | 是 | calendar-time portfolio / regression 语义 slot；算法 deferred。 |
| `method_ref` | str | 条件必填 | 有外部方法说明时填写。 |
| `n/a_reason` | str | 条件必填 | `method_ref` 不适用时必须说明。 |
| `schema_version` | str | 是 | 建议 `event_study_method_spec_v1`。 |

### 5.3 Window and Normal Return Model

| 对象 | 字段 | 约束 |
|---|---|---|
| `EventStudyWindowSpec` | `window_id`、`relative_start`、`relative_end`、`calendar_start`、`calendar_end`、`anchor`、`trading_calendar_ref` | 至少一组 relative 或 calendar boundary 完整；`relative_start <= relative_end`；`anchor` 默认 `event_available_at_or_decision_time` 由 S01 语义决定，S02 不推断 availability。 |
| `NormalReturnModelSpec` | `model_id`、`model_type`、`market_benchmark_ref`、`factor_model_ref`、`estimation_method`、`status`、`n/a_reason` | `model_type` 不得为空；`factor_model` 需要 ref；unsupported active model `BLOCKED`。 |
| `EventStudyMethodSlot` | `method_name`、`status`、`support_level`、`report_ref`、`n/a_reason`、`limitations` | `status` 必须是四态；`report_ref` 与 `n/a_reason` 至少一个非空。 |

### 5.4 EventStudyTestReport and Test Family Slots

| 字段 | 类型 | 必填 | 约束 / 说明 |
|---|---|---|---|
| `report_id` | str | 是 | 稳定 test report id。 |
| `method_id` | str | 是 | 引用 `EventStudyMethodSpec.method_id`。 |
| `test_family_slots` | tuple / mapping | 是 | 至少包含 Patell、BMP、generalized sign、rank、bootstrap 的 slot 或明确 N/A。 |
| `multiple_testing_or_data_snooping_slot` | `EventStudyMultipleTestingSlot` | 是 | EV-GAP-7 mandatory。 |
| `sample_count` | int | 是 | report-level 样本数；每个 family slot 也可覆盖。 |
| `operation_counts` | mapping | 是 | forbidden counters 必须为 0。 |
| `schema_version` | str | 是 | 建议 `event_study_test_report_v1`。 |

`EventStudyTestFamilySlot` 字段：

| 字段 | 类型 | 必填 | 约束 / 说明 |
|---|---|---|---|
| `family_id` | str | 是 | 例如 `patell`、`bmp`、`generalized_sign`、`rank`、`bootstrap`。 |
| `status` | str | 是 | `PASS / FAIL / NEEDS_REVIEW / BLOCKED`。 |
| `sample_count` | int | 是 | 大于 0，除非 slot 为明确 N/A 且有 `n/a_reason`。 |
| `raw_p_value` | float / null | 条件必填 | active test family 需要数值，范围 `[0, 1]`。 |
| `adjusted_p_value` | float / null | 条件必填 | active test family 需要数值，范围 `[0, 1]`；可来自 slot-only correction evidence。 |
| `report_ref` | str | 条件必填 | 有 report 时填写。 |
| `n/a_reason` | str | 条件必填 | 无 report 或 deferred 时填写；不得与 active claim 矛盾。 |

### 5.5 EV-GAP-7 Multiple Testing / Data Snooping Slot

| 字段 | 类型 | 必填 | 约束 / 说明 |
|---|---|---|---|
| `family_id` | str | 是 | correction family，例如 `event_window_family`、`normal_return_model_family`、`strategy_search_family`。 |
| `tested_window_count` | int | 是 | 大于 0；覆盖 event windows / return horizons / model variants 的测试数量。 |
| `correction_method` | str | 是 | 允许 `bonferroni`、`holm`、`fdr_bh`、`white_reality_check`、`hansen_spa`、`romano_wolf`、`pbo`、`dsr`、`not_applicable_with_reason` 等 slot value。 |
| `adjusted_p_value` | float / null | 条件必填 | 若 status 是 `PASS` / `FAIL`，必须为 `[0, 1]`；deferred / N/A 时可为空但必须有 `n/a_reason`。 |
| `status` | str | 是 | `PASS / FAIL / NEEDS_REVIEW / BLOCKED`。 |
| `report_ref` | str | 条件必填 | 有 correction report 时填写。 |
| `n/a_reason` | str | 条件必填 | `report_ref` 为空时必填；不能用空字符串绕过。 |

算法边界：

| Algorithm / correction | CR153 first-wave support | Active enforcement |
|---|---|---|
| `bonferroni` / `holm` / `fdr_bh` | 仅 slot evidence；可接受预先给定 adjusted p-value，不实现计算。 | 缺 `adjusted_p_value` 时 `BLOCKED`。 |
| White Reality Check | slot-only / deferred | active implementation claim 返回 `BLOCKED`。 |
| Hansen SPA | slot-only / deferred | active implementation claim 返回 `BLOCKED`。 |
| Romano-Wolf | slot-only / deferred | active implementation claim 返回 `BLOCKED`。 |
| PBO | slot-only / deferred | active implementation claim 返回 `BLOCKED`。 |
| DSR | slot-only / deferred | active implementation claim 返回 `BLOCKED`。 |

## 6. API / Interface 设计

| 接口 / 入口 | 输入 | 输出 | 调用方 | 说明 |
|---|---|---|---|---|
| `validate_event_study_method_spec(spec)` | `EventStudyMethodSpec | Mapping[str, Any] | None` | `tuple[EventStudyValidationIssue, ...]` | tests、S04 event gate | 校验 method id、S01 anchor、estimation/event window、normal return model、return horizon 和 method slots。 |
| `validate_event_study_test_report(report)` | `EventStudyTestReport | Mapping[str, Any] | None` | `tuple[EventStudyValidationIssue, ...]` | tests、S04 event gate | 校验 test family slots 和 mandatory EV-GAP-7 slot。 |
| `validate_event_multiple_testing_slot(slot)` | `EventStudyMultipleTestingSlot | Mapping[str, Any] | None` | `tuple[EventStudyValidationIssue, ...]` | `validate_event_study_test_report` | 校验 `family_id`、`tested_window_count`、`correction_method`、`adjusted_p_value`、`status`、`report_ref | n/a_reason`。 |
| `forbidden_event_operation_counts_zero(operation_counts)` | mapping | issue tuple | all validators | 本地静态边界守卫；非 0 forbidden counter -> `BLOCKED` issue。 |
| `to_dict()` | dataclass instance | JSON-safe dict | evidence/tests | 仅序列化，不做 IO。 |

错误暴露约定：

- 预期 metadata 缺陷以 `EventStudyValidationIssue` 返回，不抛异常。
- 类型不可解析或 status 不在枚举内返回 `BLOCKED` severity issue。
- validator 不读取 `.env`、文件系统、网络、provider、lake、NAS、catalog 或 registry。
- `validate_event_study_test_report()` 不计算 p-value，只验证传入 slot 的存在性、一致性和状态语义。

## 7. 核心处理流程

1. S01 提供 `EventResearchSpec` / PIT 语义的 stable id；S02 只引用 `event_research_spec_id`，不推断 event availability。
2. Caller 构造 `EventStudyMethodSpec`：
   - 选择 estimation window；
   - 选择 event window；
   - 选择 normal return model；
   - 声明 return horizon；
   - 填写 CAR / BHAR / calendar-time method slots。
3. `validate_event_study_method_spec()` 执行 mandatory field、window boundary、normal return model 和 unsupported active method 检查。
4. Caller 构造 `EventStudyTestReport`，为 Patell / BMP / generalized sign / rank / bootstrap 提供 slot evidence 或 N/A reason。
5. Caller 构造 mandatory `multiple_testing_or_data_snooping_slot`。
6. `validate_event_study_test_report()` 先检查 forbidden operation counters，再检查 family slots，最后检查 EV-GAP-7 slot。
7. 若 mandatory evidence 缺失、active unsupported algorithm 或 forbidden operation counter 非 0，返回 `BLOCKED` issue。
8. S04 后续消费 S02 method/test report 的 issue tuple 和 status，映射到 `EventStrategyAdmissionGate`；S02 不做 admission gate 决策。

### Fail-Closed Matrix

| Failure | Expected issue severity / status | Reason |
|---|---|---|
| Missing `EventStudyMethodSpec` | `blocker` / `BLOCKED` | 事件研究方法不可证明。 |
| Missing `estimation_window` | `blocker` / `BLOCKED` | EV-GAP-1 未覆盖。 |
| Missing `event_window` | `blocker` / `BLOCKED` | 不能证明 event study window。 |
| Missing `normal_return_model` | `blocker` / `BLOCKED` | 事件异常收益不能降级为普通 forward return。 |
| Missing `return_horizon` | `blocker` / `BLOCKED` | CAR/BHAR/calendar-time 语义不完整。 |
| Missing test family slots | `blocker` / `BLOCKED` | EV-GAP-2 未覆盖。 |
| Missing `multiple_testing_or_data_snooping_slot` | `blocker` / `BLOCKED` | EV-GAP-7 是 first-wave mandatory slot。 |
| Active White/Hansen/Romano-Wolf/PBO/DSR implementation claim | `blocker` / `BLOCKED` | CR153 first wave slot-only，不实现算法。 |
| `report_ref` and `n/a_reason` both empty | `blocker` / `BLOCKED` | 证据或不适用原因不可追溯。 |
| Forbidden operation counter nonzero | `blocker` / `BLOCKED` | 违反 local/static/fixture-only 授权。 |

## 8. 技术设计细节

- 遵循现有 `engine/*contracts.py` 风格，优先使用 `@dataclass(frozen=True, slots=True)`。
- `to_dict()` 输出 JSON-safe primitive；tuple 字段转换为 list，mapping 转 dict。
- Status 用 string enum 或 shared constants，值保持 `PASS / FAIL / NEEDS_REVIEW / BLOCKED`，便于 S04 adapter 消费。
- Issue code 建议前缀 `event_study_`，例如：
  - `event_study_method_missing`
  - `event_study_estimation_window_missing`
  - `event_study_event_window_missing`
  - `event_study_normal_return_model_missing`
  - `event_study_test_family_slots_missing`
  - `event_study_multiple_testing_slot_missing`
  - `event_study_multiple_testing_report_ref_or_na_missing`
  - `event_study_unsupported_algorithm_active`
  - `event_study_forbidden_operation_nonzero`
- Normal return model 和 test family 的 slot 只表达 evidence status，不执行 OLS、bootstrap、rank test、Patell/BMP 或 White/Hansen/PBO/DSR 算法。
- 若实现阶段发现 `engine/event_strategy_contracts.py` 已由 S01 或 S03 创建，S02 只追加自身字段；不得重写或删除其他 Story 字段。
- `multiple_testing_or_data_snooping_slot` 的 `adjusted_p_value` 是传入 evidence 字段，不由 S02 计算。
- 任何 `listen`、`subscribe`、`fetch`、`write`、`publish`、`register`、`promote`、`upload`、`sync`、`persist`、`mutate`、`set_current`、`submit_order`、`cancel_order`、`query_account` 行为都不属于 S02。

## 9. 安全与性能设计

| 维度 | 设计措施 | 验证方式 |
|---|---|---|
| Authorization | 所有对象只消费传入 metadata；operation counters 覆盖真实 feed、listener、lake、NAS、provider、credential、event store、catalog、registry、runtime、broker、order flow。 | S02 tests 构造非 0 counter 并断言 `BLOCKED` issue。 |
| Data safety | 不读取真实事件、真实收益、真实 provider、真实湖仓或 `.env`。 | 代码审查 + fixture-only tests；无 IO API。 |
| Scope safety | S02/S03 字段分区表写入 LLD，tests 断言 S02 fixture 不含 S03-owned keys。 | `test_s02_s03_field_partition_prevents_overlap`。 |
| Performance | 校验复杂度与 slot 数量线性相关；first wave fixture 数量小。 | 不需要 benchmark；单元测试即可。 |
| Compatibility | 新增 event companion contracts，不改变现有 CR151/CR152 objects。 | 运行 CR153 targeted tests；必要时保留 CR151/CR152 targeted tests。 |

## 10. 测试设计

| 测试场景 | 前置条件 | 操作 | 预期结果 | 验证方式 |
|---|---|---|---|---|
| Valid method spec serializes | 完整 estimation/event window、normal return model、return horizon 和 method slots | 调用 `to_dict()` | 输出包含 stable schema、窗口、normal model、CAR/BHAR/calendar-time slots | pytest |
| Missing estimation window blocked | `estimation_window` 为空 | `validate_event_study_method_spec()` | issue code `event_study_estimation_window_missing`，severity `blocker` | pytest |
| Missing event window blocked | `event_window` 为空 | `validate_event_study_method_spec()` | issue code `event_study_event_window_missing` | pytest |
| Missing normal return model blocked | `normal_return_model` 为空 | `validate_event_study_method_spec()` | issue code `event_study_normal_return_model_missing` | pytest |
| Ordinary forward return cannot substitute event study | 只给 `forward_return_days`，无 method slots | method validator | `BLOCKED`，提示 event study method evidence missing | pytest |
| Test family slots serialize | Patell/BMP/sign/rank/bootstrap slots 有 ref 或 N/A reason | report `to_dict()` | JSON-safe dict，family ids 稳定 | pytest |
| Missing test family slots blocked | `test_family_slots` 为空 | `validate_event_study_test_report()` | issue code `event_study_test_family_slots_missing` | pytest |
| Missing multiple-testing slot blocked | report 无 `multiple_testing_or_data_snooping_slot` | `validate_event_study_test_report()` | issue code `event_study_multiple_testing_slot_missing` | pytest |
| Multiple-testing mandatory fields | slot 缺 `family_id`、`tested_window_count`、`correction_method`、`status` 或 ref/N/A | `validate_event_multiple_testing_slot()` | 对应 blocker issue | pytest |
| Adjusted p-value invalid | status `PASS` 但 `adjusted_p_value` 缺失或超出 `[0,1]` | slot validator | `event_study_adjusted_p_value_invalid` | pytest |
| Unsupported active algorithms blocked | White/Hansen/Romano-Wolf/PBO/DSR 声明 active implemented | slot / method validator | `event_study_unsupported_algorithm_active` | pytest |
| Forbidden operation counter blocked | `provider_fetch=1` 或 `event_store_write=1` | report validator | `event_study_forbidden_operation_nonzero` | pytest |
| S02/S03 partition | S02 fixture 尝试包含 `cluster_report_slot` 或 `universe_pit_audit` | partition test | 测试失败或 validator 返回 out-of-scope issue | pytest |

验证入口：

```bash
uv run --python 3.11 pytest -q tests/research/test_event_driven_strategy_e2e_contracts.py
```

可选回归入口：

```bash
uv run --python 3.11 pytest -q tests/research/test_strategy_admission_statistical_gate.py tests/research/test_strategy_admission_package.py
```

## 11. 实施步骤

| TASK-ID | 动作 | 目标文件 | 详细描述 | 对应测试 |
|---|---|---|---|---|
| CR153-S02-T01 | 新增 / 修改 | `engine/event_strategy_contracts.py` | 定义 status constants、issue object、forbidden operation counters 和 S02 `to_dict()` helpers。 | forbidden counter / serialization tests |
| CR153-S02-T02 | 新增 / 修改 | `engine/event_strategy_contracts.py` | 定义 `EventStudyWindowSpec`、`NormalReturnModelSpec`、`EventStudyMethodSlot`、`EventStudyMethodSpec`。 | valid method / missing window / missing normal model |
| CR153-S02-T03 | 新增 / 修改 | `engine/event_strategy_contracts.py` | 定义 `EventStudyTestFamilySlot`、`EventStudyMultipleTestingSlot`、`EventStudyTestReport`。 | family slot serialization / multiple-testing required fields |
| CR153-S02-T04 | 新增 / 修改 | `engine/event_strategy_contracts.py` | 实现 `validate_event_study_method_spec()`、`validate_event_study_test_report()`、`validate_event_multiple_testing_slot()`。 | all negative validators |
| CR153-S02-T05 | 新增 / 修改 | `tests/research/test_event_driven_strategy_e2e_contracts.py` | 增加 S02 positive / negative fixtures，含 unsupported algorithm 和 field partition tests。 | S02 targeted pytest |

TASK 与文件影响范围映射：

- T01-T04 只触及 `engine/event_strategy_contracts.py` 的 S02-owned 字段。
- T05 只触及 shared test file 的 S02 fixture section。
- 不得修改 `engine/research_production_contracts.py`、`engine/strategy_admission_statistical_gate.py`、event store、catalog、runtime、broker 或真实数据相关模块。

## 12. 风险、难点与预研建议

### 12.1 实现灰区与取舍记录

| Clarification ID | 问题 | 推荐方案 | 决策 / 答案 | 影响面 | 证据 | 重访条件 |
|---|---|---|---|---|---|---|
| LCQ-CR153-S02-001 | White Reality Check、Hansen SPA、Romano-Wolf、PBO、DSR 是否在 CR153 first wave 实现？ | 不实现；保留 slot-only / deferred，active implementation claim 默认 `BLOCKED`。 | Resolved by HLD/ADR/CP5 context；无阻断项。 | `EventStudyMultipleTestingSlot`、validators、tests、release wording | HLD §11 EV-GAP-7；ADR Downstream Obligations；CP5-FOCUS-CR153-002 | 后续 CR 明确授权完整算法实现和真实数据验证。 |
| LCQ-CR153-S02-002 | S02 与 S03 在 shared `engine/event_strategy_contracts.py` 中如何避免字段重叠？ | S02 只定义 method/test-family/multiple-testing；S03 定义 overlap/cluster/endogeneity/CV/universe PIT。 | Resolved by CP5 context shared_field_partition；本 LLD §4 固化。 | shared module、shared tests、CP5 review | CP5-FOCUS-CR153-005；Story Backlog file ownership | CP5 人工确认要求调整 Story ownership。 |

| 风险 / 难点 | 影响 | 缓解措施 |
|---|---|---|
| Slot evidence 被误读为真实算法实现 | 虚假统计能力声明 | 字段名和 release wording 使用 `slot`、`deferred`、`n/a_reason`；unsupported active algorithms 返回 `BLOCKED`。 |
| Event study 被普通 forward return 替代 | UC-60 审计链断裂 | `normal_return_model`、estimation/event window、method slots 为 mandatory；缺失即 `BLOCKED`。 |
| S02/S03 shared 文件并行冲突 | CP5/CP6 merge 风险 | 字段分区表和 tests 明确 owner；S02 不创建 S03 字段。 |
| Multiple-testing slot 只有 p-value 无 family context | 数据窥探风险不可审计 | `family_id`、`tested_window_count`、`correction_method` 为 mandatory。 |

### OPEN / Spike 跟踪

| ID | 类型 | 问题 | 下一动作 | 责任方 |
|---|---|---|---|---|
| N/A | OPEN | 无阻断性未决项。 | N/A | N/A |

## 13. 回滚与发布策略

- Rollback：若 CP6 实现违反 S02/S03 字段分区，可回滚 `engine/event_strategy_contracts.py` 中 S02-owned classes / validators 和 shared test file 中 S02 fixture section；不得回滚其他 Story 字段。
- Compatibility：无数据库、无持久化迁移、无 event store/catalog/model registry 写入；回滚不涉及数据恢复。
- Release wording：CR153 first wave 只声明 event study method/test/multiple-testing slot semantics；不得声明 Patell/BMP/bootstrap/White/Hansen/Romano-Wolf/PBO/DSR 算法已实现，不得声明真实 feed、真实收益、生产 readiness 或 trading readiness。
- Dev gate：只有 CP5 全量设计证据人工确认后，才能按 Wave 实现；本 LLD 本身不授权源码修改。

## 14. Definition of Done

- [ ] `EventStudyMethodSpec` 可序列化，包含 estimation window、event window、normal return model、return horizon、CAR/BHAR/calendar-time slots。
- [ ] `EventStudyTestReport` 可序列化，包含 Patell、BMP、generalized sign、rank、bootstrap 等 test family slots。
- [ ] `multiple_testing_or_data_snooping_slot` 必含 `family_id`、`tested_window_count`、`correction_method`、`adjusted_p_value`、`status`、`report_ref | n/a_reason`。
- [ ] 缺少 method evidence、test family slots 或 multiple-testing slot 返回 `BLOCKED` issue。
- [ ] White Reality Check、Hansen SPA、Romano-Wolf、PBO、DSR 等算法保持 slot-only / deferred；active implementation claim 返回 `BLOCKED`。
- [ ] Shared `engine/event_strategy_contracts.py` 和 `tests/research/test_event_driven_strategy_e2e_contracts.py` 的 S02/S03 字段分区被测试或静态断言覆盖。
- [ ] 没有真实 feed、listener、lake、NAS、provider、runtime、broker、credential、event store、catalog、registry、order flow 或真实数据验证行为。

## 人工确认区

| # | 检查项 | 状态 | 证据 |
|---|---|---|---|
| 1 | LLD 覆盖 S02 acceptance criteria | pending | 第 2 / 5 / 10 / 14 节 |
| 2 | EV-GAP-7 multiple-testing / data-snooping slot mandatory | pending | 第 5.5 / 7 / 10 节 |
| 3 | White/Hansen/Romano-Wolf/PBO/DSR slot-only / deferred | pending | 第 5.5 / 8 / 12 节 |
| 4 | S02/S03 shared field partition 清晰 | pending | 第 4 节 |
| 5 | local/static/fixture-only 边界清晰 | pending | 第 1 / 9 / 13 / 14 节 |

**人工审查结果回填**：

- 结论：`pending`
- 审查人：
- 审查时间：
- 修改意见：
- 风险接受项：

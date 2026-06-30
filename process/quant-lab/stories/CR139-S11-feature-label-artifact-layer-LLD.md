---
story_id: "CR139-S11"
title: "V3 feature/label/artifact 层"
story_slug: "feature-label-artifact-layer"
lld_version: "1.0"
tier: "M"
status: "ready-for-review"
confirmed: false
created_by: "host-orchestrator-inline"
created_at: "2026-06-30T17:40:00+08:00"
confirmed_by: ""
confirmed_at: ""
shared_fragments: []
feature_design_refs:
  - "docs/design/FEATURE-DESIGN-MATRIX.md v1.13 REQ-219"
lld_policy:
  required_level: "full-lld"
  trigger_reasons: ["c 范围扩展", "feature schema"]
  rationale: "V3 是 c 类，D5 lake features/ 子层带版本"
open_items: 0
---

# LLD: CR139-S11 — V3 feature/label/artifact 层

## 0. 上游设计依据

| 来源 | 路径 / ID | 被本 LLD 消费的内容 |
|---|---|---|
| HLD | `process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md` | D5 lake features/ 子层、PIT/current truth 读侧边界、parquet 仍为事实载体。 |
| ADR | `process/docs/design/ARCHITECTURE-DECISION-STRATEGY-DATA-FOUNDATION.md` | 不引入独立 feature store；保留 DEF-139-01 切换条件。 |
| Feature Matrix | `docs/design/FEATURE-DESIGN-MATRIX.md v1.13` | REQ-219 为 FEAT-03 c 类 full-lld。 |
| Story | `process/stories/CR139-S11-feature-label-artifact-layer.md` | 文件 owner、依赖 S06、验收标准和不授权范围。 |

## 1. Goal

创建最小可验证的 versioned feature/label/artifact contract：`market_data/features/` 负责 schema、artifact identity、path preview 和安全计数，`market_data/lake_layout.py` 只新增 features/ 路径构造函数，不创建目录、不写真实 lake。

## 2. Requirements（Functional / Non-Functional）

### 2.1 Functional

- Feature artifact 必须包含 `feature_set_id`、`schema_version`、`artifact_version`、`as_of_trade_date`、`run_id`、`source_view_refs`、`lineage_checksum`。
- Label artifact 必须声明 `label_horizon`、`label_available_at_field` 和 leakage guard 所需字段。
- Artifact path 必须落在 `lake/features/feature_set=<id>/schema_version=<v>/artifact_version=<v>/as_of_trade_date=<yyyymmdd>/run_id=<run_id>` 结构下。
- DEF-139-01 独立 feature store 切换条件必须以结构化对象暴露。

### 2.2 Non-Functional

- 不写真实 lake，不创建目录，不读取 provider/credential，不启动 runtime。
- 只复用 S06 的 panel reader contract 引用，不读取未发布数据。
- 输出结构必须可序列化、可测试，并能被 S10/S12/S15 后续 Story 消费。

## 3. 模块拆分与职责

| 模块 / 文件组 | 职责 | 说明 |
|---|---|---|
| `market_data/features/__init__.py` | 暴露 feature artifact contract API。 | 新模块，独占 owner。 |
| `market_data/features/artifacts.py` | 定义 feature/label schema、artifact plan、DEF-139-01 切换条件和安全计数。 | 不依赖 pandas/pyarrow/DuckDB。 |
| `market_data/lake_layout.py` | 新增 features root/path preview helper。 | 只返回 `Path`，不 mkdir/write。 |
| `tests/test_cr139_feature_label_artifact_layer.py` | S11 fixture/static tests。 | 使用 `tmp_path`，不写真实 lake。 |

## 4. 代码结构与文件影响范围

| 动作 | 文件路径 | 变更内容 |
|---|---|---|
| 创建 | `market_data/features/__init__.py` | 导出 S11 contract 类型和 builder。 |
| 创建 | `market_data/features/artifacts.py` | 实现 feature/label artifact plan、schema validation、safety counters。 |
| 修改 | `market_data/lake_layout.py` | 添加 `features_root` 和 `feature_artifact_path()`。 |
| 创建 | `tests/test_cr139_feature_label_artifact_layer.py` | 覆盖 path/schema/label/DEF-139-01/零副作用。 |

## 5. 数据模型与持久化设计

| 对象 / 字段 | 类型 | 约束 | 说明 |
|---|---|---|---|
| `FeatureArtifactSpec.feature_set_id` | str | 非空安全分区 token | feature set 稳定 ID。 |
| `FeatureArtifactSpec.schema_version` | str | 非空安全分区 token | schema 版本。 |
| `FeatureArtifactSpec.artifact_version` | str | 非空安全分区 token | artifact 版本。 |
| `FeatureArtifactSpec.as_of_trade_date` | str | compact date | PIT/as-of 边界。 |
| `FeatureArtifactSpec.run_id` | str | CR139 run_id contract | lineage 追踪。 |
| `FeatureArtifactSpec.source_view_refs` | tuple[str] | 至少 1 个 | S06/published view 输入引用。 |
| `FeatureArtifactSpec.lineage_checksum` | str | 非空 | 下游实验 manifest 闭环。 |
| `LabelArtifactSpec.label_horizon` | int | > 0 | label horizon。 |
| `LabelArtifactSpec.label_available_at_field` | str | 默认 `label_available_at` | S15 leakage guard 预留。 |

无真实持久化写入；本 Story 只定义路径和 metadata contract。

## 6. API / Interface 设计

| 接口 / 入口 | 输入 | 输出 | 调用方 | 说明 |
|---|---|---|---|---|
| `LakeLayout.feature_artifact_path()` | feature set, schema version, artifact version, date, run_id | `Path` | feature artifact builder | 只构造 path。 |
| `build_feature_artifact_plan()` | `FeatureArtifactSpec`, `LakeLayout` | `FeatureArtifactPlan` | S10/S12/S15 | 结构化 plan；`real_write=False`。 |
| `validate_feature_artifact_spec()` | `FeatureArtifactSpec` | `FeatureArtifactValidation` | tests/downstream | 返回缺失字段与 safety counters。 |
| `feature_store_switch_policy()` | metrics mapping | `FeatureStoreSwitchPolicy` | docs/downstream | 记录 DEF-139-01。 |

## 7. 核心处理流程

1. 调用方从 S06 panel/published view 得到 `source_view_refs`。
2. 调用方构造 `FeatureArtifactSpec` / `LabelArtifactSpec`。
3. `validate_feature_artifact_spec()` 校验必填字段、version、run_id、label guard 字段。
4. `LakeLayout.feature_artifact_path()` 生成 versioned features 路径。
5. `build_feature_artifact_plan()` 返回 `real_write=False` 的 artifact plan 和零副作用计数。

## 8. 技术设计细节

- 关键规则：path token 使用 `lake_layout` 既有清洗逻辑，不接受 path separator、空值或不稳定 token。
- 依赖复用：复用 `LakeLayout`、`validate_cr139_run_id()`；不引入 pandas/pyarrow/DuckDB。
- 兼容性处理：独立 feature store 不实现，只输出 DEF-139-01 触发条件。
- 图示类型选择：不需要 Mermaid；流程小于 3 个跨模块分支。

## 9. 安全与性能设计

| 维度 | 设计措施 | 验证方式 |
|---|---|---|
| 安全 | builder 不 mkdir/write，不读 env，不读 provider/credential。 | safety counters 全 0；`tmp_path` 下无非测试写入。 |
| 性能 | 仅 dataclass/path 计算，无数据扫描。 | fixture 单测。 |

## 10. 测试设计

| 测试场景 | 前置条件 | 操作 | 预期结果 | 验证方式 |
|---|---|---|---|---|
| T-S11-01 | valid spec | build plan | versioned features path 正确，`real_write=False` | pytest |
| T-S11-02 | missing lineage/source refs | validate spec | blocked + actionable issue | pytest |
| T-S11-03 | valid label spec | build plan | label horizon/available_at contract 保留 | pytest |
| T-S11-04 | switch metrics over threshold | evaluate DEF-139-01 | recommend independent store review | pytest |

## 11. 实施步骤

| TASK-ID | 动作 | 目标文件 | 详细描述 | 对应测试 |
|---|---|---|---|---|
| TASK-S11-01 | 创建 | `market_data/features/artifacts.py` | dataclasses、validation、plan builder、switch policy。 | T-S11-01..04 |
| TASK-S11-02 | 创建 | `market_data/features/__init__.py` | 导出 public API。 | import smoke |
| TASK-S11-03 | 修改 | `market_data/lake_layout.py` | 新增 features path helper。 | T-S11-01 |
| TASK-S11-04 | 创建 | `tests/test_cr139_feature_label_artifact_layer.py` | 添加 fixture/static tests。 | T-S11-01..04 |

## 12. 风险、难点与预研建议

### 12.1 实现灰区与取舍记录

| Clarification ID | 问题 | 选项与推荐 | 决策 / 答案 | 影响面 | 证据 | 重访条件 |
|---|---|---|---|---|---|---|
| N/A | 无阻塞 clarification。 | 推荐先实现 in-lake contract，不引入独立 store。 | 已按 ADR/Story 边界处理。 | 接口 / 测试 | 本 LLD §0 | 当 DEF-139-01 指标命中时重访。 |

| 风险 / 难点 | 影响 | 缓解措施 / 预研建议 |
|---|---|---|
| 后续 S10/S12/S15 对 artifact schema 需求扩展 | 可能需要新增字段 | 使用 dataclass + validation issue 结构，保持向后兼容。 |
| 误认为本 Story 已写入真实 features lake | 越权风险 | API 和证据均标记 `real_write=False`，真实发布另走授权门。 |

### OPEN / Spike 跟踪

| ID | 类型（OPEN / Spike） | 问题 | 下一动作 | 责任方 |
|---|---|---|---|---|
| N/A | OPEN | 无开放问题。 | N/A | N/A |

## 13. 回滚与发布策略

- 发布方式：普通代码 + fixture tests；不发布数据。
- 回滚触发条件：新增 helper 破坏 `LakeLayout` 现有路径测试或 feature contract tests。
- 回滚动作：删除 `market_data/features/` 新模块和 `LakeLayout.feature_artifact_path()` 切片，保留过程证据。

## 14. Definition of Done

- [x] 14 个章节全部填写完成
- [x] 文件影响范围、接口、测试与实施步骤可直接指导编码
- [x] 实现灰区与取舍记录已显式写“无”
- [x] `confirmed=false` 时不进入实现，除非本 CR backlog 已授权无风险自动推进
- [x] OPEN / Spike 已清点

## 人工确认区

CP5 自动预检路径：`process/checks/CP5-CR139-S11-feature-label-artifact-layer-LLD-IMPLEMENTABILITY.md`。

人工审查结果回填：

- 结论：`not-required-for-no-risk-inline-backlog`
- 审查人：N/A
- 审查时间：N/A
- 修改意见：无
- 风险接受项：无

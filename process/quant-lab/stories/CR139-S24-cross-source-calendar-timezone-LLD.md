---
story_id: "CR139-S24"
title: "X2 跨源交易日历/时区一致性"
story_slug: "cross-source-calendar-timezone"
lld_version: "1.0"
tier: "M"
status: "ready-for-review"
confirmed: false
created_by: "host-orchestrator-inline"
created_at: "2026-06-30T14:45:00+08:00"
confirmed_by: ""
confirmed_at: ""
shared_fragments: []
feature_design_refs:
  - "docs/design/FEATURE-DESIGN-MATRIX.md v1.13 REQ-230"
lld_policy:
  required_level: "full-lld"
  trigger_reasons: ["d1 纯新建对象"]
  rationale: "X2 是 d1 纯新建"
open_items: 0
---

# LLD: CR139-S24 — X2 跨源交易日历/时区一致性

## 0. 上游设计依据

| 来源 | 路径 / ID | 被本 LLD 消费的内容 |
|---|---|---|
| HLD | `process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md` | FEAT-02 写侧负责跨源时点一致性；X2 独立校验，不依赖真实写湖。 |
| ADR | `process/docs/design/ARCHITECTURE-DECISION-STRATEGY-DATA-FOUNDATION.md` | 日频离线研究 + 模拟盘，默认不启动 runtime/provider。 |
| Feature Matrix | `docs/design/FEATURE-DESIGN-MATRIX.md` | REQ-230 是 d1/full-lld；输出文件 `market_data/trade_calendar.py`。 |
| Requirements | `process/REQUIREMENTS.md` | trade_calendar 支撑 open trade dates 分母；跨源日期与 `available_at` 需可审计。 |

## 1. Goal

创建 `market_data/trade_calendar.py`，提供纯内存跨源交易日历与时区归一校验：给定 tushare/jqdata/QMT 等来源的 calendar rows，统一解析 trade date、open/closed 状态和 timestamp timezone，输出一致性结果与结构化 mismatch。

## 2. Requirements（Functional / Non-Functional）

### 2.1 Functional

- 规范化 `trade_date` 为 `YYYY-MM-DD`。
- 规范化时间戳字段为 timezone-aware ISO 字符串，默认目标时区为 `Asia/Shanghai`。
- 对多个 source 的 open trade dates 做集合比较；任一 source 缺日、多日或 open/closed 冲突时 fail。
- 输出 source coverage、canonical open dates、mismatch 列表和 operation counters。

### 2.2 Non-Functional

- 纯函数实现，无 provider fetch、无 lake/catalog/manifest/pointer 写入。
- 对输入顺序稳定，输出可快照测试。
- 对缺列/非法日期 fail closed，不静默推断。

## 3. 模块拆分与职责

| 模块 / 文件组 | 职责 | 说明 |
|---|---|---|
| `market_data/trade_calendar.py` | 交易日历归一与跨源一致性校验 | S24 独占新模块。 |
| `tests/test_cr139_cross_source_calendar_timezone.py` | fixture/static 验证 | 只使用内存 rows。 |

## 4. 代码结构与文件影响范围

| 动作 | 文件路径 | 变更内容 |
|---|---|---|
| 创建 | `market_data/trade_calendar.py` | dataclass + normalize/validate helper。 |
| 创建 | `tests/test_cr139_cross_source_calendar_timezone.py` | pass/mismatch/timezone/missing-field tests。 |

## 5. 数据模型与持久化设计

| 对象 / 字段 | 类型 | 约束 | 说明 |
|---|---|---|---|
| `CalendarSourceSnapshot.source` | `str` | 非空 | 来源名，如 `tushare`、`jqdata`、`qmt`。 |
| `CalendarSourceSnapshot.open_dates` | `tuple[str, ...]` | `YYYY-MM-DD`、去重排序 | 单一来源 open dates。 |
| `CalendarConsistencyResult.passed` | `bool` | mismatch 为空才为 true | CP7 验证主断言。 |
| `CalendarConsistencyResult.mismatches` | `tuple[dict, ...]` | JSON-ready | 缺日、多日、状态冲突、字段缺失、时间不可解析。 |

无新增持久化变更；`config/lake/catalog/manifest/pointer` 均不写。

## 6. API / Interface 设计

| 接口 / 入口 | 输入 | 输出 | 调用方 | 说明 |
|---|---|---|---|---|
| `normalize_calendar_rows(source, rows, timezone="Asia/Shanghai")` | source + sequence/mapping rows | `CalendarSourceSnapshot` | tests / future writer gates | 解析 `trade_date/date`、`is_open`、可选 timestamp。 |
| `validate_cross_source_calendar(sources, timezone="Asia/Shanghai")` | source -> rows | `CalendarConsistencyResult` | tests / future quality gates | 比较所有来源 open dates。 |

## 7. 核心处理流程

1. 调用方传入 `{source_name: rows}`。
2. 每个 source 通过 `normalize_calendar_rows()` 得到 open dates 和 issues。
3. `validate_cross_source_calendar()` 以所有 source open dates 的并集为比较域。
4. 对每个 source 计算 `missing_open_dates` 与 `extra_open_dates`。
5. 若任何 issue/mismatch 存在，`passed=false`；否则输出 canonical open dates。

## 8. 技术设计细节

- 关键算法 / 规则：以 set 比较 open dates；`is_open` 缺失时默认该 row 表示 open day，但 source row 缺 `trade_date/date` 时 fail。
- 依赖选择与复用点：仅使用 Python 标准库 `zoneinfo`、`datetime` 和 pandas `to_datetime` 不引入新依赖。
- 兼容性处理：接受 `trade_date` 或 `date`；接受 `YYYYMMDD` / ISO date / timestamp。
- 图示类型选择：不需要 Mermaid，单模块纯函数无跨 3 模块流程。

## 9. 安全与性能设计

| 维度 | 设计措施 | 验证方式 |
|---|---|---|
| 安全 | 无外部 IO；operation counters 全为 0。 | return/evidence boundary check。 |
| 性能 | O(source_count * row_count) set 比较。 | fixture 单测覆盖。 |

## 10. 测试设计

| 测试场景 | 前置条件 | 操作 | 预期结果 | 验证方式 |
|---|---|---|---|---|
| 三源日历一致 | tushare/jqdata/QMT open dates 相同 | validate | `passed=true` | pytest |
| source 缺少 open date | jqdata 缺一天 | validate | `calendar_missing_open_date` | pytest |
| timezone 归一 | timestamp 带 UTC 时区 | normalize | 输出 Asia/Shanghai aware ISO | pytest |
| 缺日期字段 | row 无 `trade_date/date` | validate | `calendar_trade_date_missing` | pytest |

## 11. 实施步骤

| TASK-ID | 动作 | 目标文件 | 详细描述 | 对应测试 |
|---|---|---|---|---|
| TASK-S24-01 | 创建 | `market_data/trade_calendar.py` | 添加 dataclass、date/timestamp normalize、cross-source validate。 | all S24 tests |
| TASK-S24-02 | 创建 | `tests/test_cr139_cross_source_calendar_timezone.py` | 添加一致、缺日、timezone、缺字段测试。 | all S24 tests |

## 12. 风险、难点与预研建议

### 12.1 实现灰区与取舍记录

| Clarification ID | 问题 | 选项与推荐 | 决策 / 答案 | 影响面 | 证据 | 重访条件 |
|---|---|---|---|---|---|---|
| N/A | 无需要用户决策的实现灰区 | 推荐按 HLD 默认 Asia/Shanghai 归一 | adopted | 测试 / 合同 | 本 LLD | 若引入非 A 股交易所日历再重访 |

| 风险 / 难点 | 影响 | 缓解措施 / 预研建议 |
|---|---|---|
| 真实 source 日历口径可能不同 | 未来真实 gate 需要业务决策 | 本 Story 只提供 mismatch 检测，不自动裁决。 |

### OPEN / Spike 跟踪

| ID | 类型（OPEN / Spike） | 问题 | 下一动作 | 责任方 |
|---|---|---|---|---|
| N/A | OPEN | 无 | 无 | 无 |

## 13. 回滚与发布策略

- 发布方式：代码和 fixture/static 测试随普通 Story 合并。
- 回滚触发条件：S24 tests 或相关 import regression 失败。
- 回滚动作：移除 `market_data/trade_calendar.py` 和 S24 tests；不涉及数据回滚。

## 14. Definition of Done

- [x] 14 个章节全部填写完成
- [x] 文件影响范围、接口、测试与实施步骤可直接指导编码
- [x] 实现灰区与取舍记录显式写“无”
- [x] `confirmed=false` 时不进入实现，除非 host-orchestrator 按 no-risk rolling CP5 执行
- [x] OPEN / Spike 已清点

## 人工确认区

CP5 自动预检路径：`process/checks/CP5-CR139-S24-cross-source-calendar-timezone-LLD-IMPLEMENTABILITY.md`。

---
story_id: "CR139-S20"
title: "T4 BrokerLakeSchema 闭环实盘写+审计链"
story_slug: "broker-lake-schema-audit-chain"
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
  - "docs/design/FEATURE-DESIGN-MATRIX.md v1.13 REQ-240"
lld_policy:
  required_level: "full-lld"
  trigger_reasons: ["c 范围扩展", "实盘写+审计链"]
  rationale: "T4 是 c 类，broker lake 实盘写边界"
open_items: 0
---

# LLD: CR139-S20 — T4 BrokerLakeSchema 闭环实盘写+审计链

## 0. 上游设计依据

| 来源 | 路径 / ID | 被本 LLD 消费的内容 |
|---|---|---|
| HLD | `process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md` | broker lake 审计链、run_id 贯通、真实 broker lake 写必须独立授权。 |
| ADR | `process/docs/design/ARCHITECTURE-DECISION-STRATEGY-DATA-FOUNDATION.md` | 交易/实盘边界默认不可触发，先以 dry-run contract 验证。 |
| Feature Matrix | `docs/design/FEATURE-DESIGN-MATRIX.md v1.13` | REQ-240 为 FEAT-06 c 类 full-lld。 |
| Story | `process/stories/CR139-S20-broker-lake-schema-audit-chain.md` | 文件 owner、依赖 S22、验收标准和不授权范围。 |
| Existing contract | `trading/broker_lake.py` | 已有 CR015 broker lake schema registry 与 dry-run writer 合同。 |

## 1. Goal

补齐 CR139 级 BrokerLakeSchema 审计链验证：订单意图、broker order、fill、position 等事件必须共享 run_id/strategy/order refs，dry-run write plan 只输出结构化审计链和脱敏路径 preview，不执行真实 broker lake 写。

## 2. Requirements（Functional / Non-Functional）

### 2.1 Functional

- Broker lake schema registry 必须覆盖 8 类事件，且每类包含 `run_id` 和 partition keys。
- CR139 审计链必须证明 order intent -> broker order -> fill -> position 共享 `run_id` 和必要关联键。
- 写入计划必须 `real_write=False`，target 只能是 root label preview，不接受真实路径。
- 敏感字段必须 redacted/blocked，输出不得保留 credential/account/session 原值。

### 2.2 Non-Functional

- 不调用 QMT/broker API，不查询账户，不写 broker lake，不读取凭据。
- 不新增外部依赖。
- Fixture/static tests 必须可在本地无 broker 环境运行。

## 3. 模块拆分与职责

| 模块 / 文件组 | 职责 | 说明 |
|---|---|---|
| `trading/broker_lake.py` | 现有 schema registry、redaction、dry-run write plan；新增 CR139 audit chain helper 如测试暴露缺口。 | 独占 owner。 |
| `tests/test_cr139_broker_lake_audit_chain.py` | CR139 S20 fixture/static tests。 | 不触真实 broker/lake。 |

## 4. 代码结构与文件影响范围

| 动作 | 文件路径 | 变更内容 |
|---|---|---|
| 修改 | `trading/broker_lake.py` | 增加 `build_broker_lake_audit_chain()` / `validate_broker_lake_audit_chain()`（若现有 API 不足）。 |
| 创建 | `tests/test_cr139_broker_lake_audit_chain.py` | 覆盖 schema registry、审计链、target block、敏感值和零副作用。 |

## 5. 数据模型与持久化设计

| 对象 / 字段 | 类型 | 约束 | 说明 |
|---|---|---|---|
| `BrokerLakeAuditChain.run_id` | str | 非空 | 贯穿所有 broker events。 |
| `BrokerLakeAuditChain.strategy_id` | str | 非空 | 策略审计维度。 |
| `BrokerLakeAuditChain.order_intent_id` | str | 非空 | order/fill 链接键。 |
| `BrokerLakeAuditChain.event_plans` | tuple | 至少 3 个 | dry-run write plans。 |
| `BrokerLakeAuditChain.real_write` | bool | 必须 false | 防越权。 |

无真实持久化变更；所有 outputs 为内存态 dataclass/dict。

## 6. API / Interface 设计

| 接口 / 入口 | 输入 | 输出 | 调用方 | 说明 |
|---|---|---|---|---|
| `build_schema_audit_summary()` | none | schema summary | tests/docs | 复用现有接口。 |
| `dry_run_write_plan()` | event payload, root label | `BrokerLakeWritePlan` | audit chain helper | 复用现有接口。 |
| `build_broker_lake_audit_chain()` | event payloads, root label | `BrokerLakeAuditChain` | tests/future runtime gate | 只构造 dry-run 链。 |
| `validate_broker_lake_audit_chain()` | audit chain | validation result | CP7/tests | 校验 run_id/order refs/zero counters。 |

## 7. 核心处理流程

1. 读取 fixture event payloads，不读取 broker 或账户。
2. 对每个 event 调用 `dry_run_write_plan()`。
3. 汇总 `run_id`、`strategy_id`、`order_intent_id`，验证跨事件一致。
4. 汇总 safety counters，任何真实操作计数非 0 则 blocked。
5. 返回 evidence-only audit chain。

## 8. 技术设计细节

- 关键规则：root target 必须是 label；真实路径、`.env`、home path、account/session/password 等字段必须 redacted/blocked。
- 依赖复用：优先复用 `trading/broker_lake.py` 已有 CR015 合同，避免重复 schema。
- 兼容性处理：若事件缺失非关键链节点，validation 返回 blocked，不抛运行时副作用。
- 图示类型选择：不需要 Mermaid；流程为同步内存态 contract。

## 9. 安全与性能设计

| 维度 | 设计措施 | 验证方式 |
|---|---|---|
| 安全 | `real_write=False`、root label only、safety counters 全 0。 | fixture tests + sensitive raw value scan。 |
| 性能 | 小规模 payload/dataclass 校验，无 IO。 | fixture tests。 |

## 10. 测试设计

| 测试场景 | 前置条件 | 操作 | 预期结果 | 验证方式 |
|---|---|---|---|---|
| T-S20-01 | valid chain payloads | build audit chain | status pass, event plans planned, shared run_id | pytest |
| T-S20-02 | true target path | dry-run write plan | blocked, no real write | pytest |
| T-S20-03 | sensitive account/session values | build plan | raw value not present; redacted/blocked | pytest |
| T-S20-04 | schema summary | inspect registry | 8 event types and run_id partition coverage | pytest |

## 11. 实施步骤

| TASK-ID | 动作 | 目标文件 | 详细描述 | 对应测试 |
|---|---|---|---|---|
| TASK-S20-01 | 修改 | `trading/broker_lake.py` | Add audit chain dataclass/helper if current dry-run API is insufficient. | T-S20-01 |
| TASK-S20-02 | 创建 | `tests/test_cr139_broker_lake_audit_chain.py` | Add CR139 fixture/static tests. | T-S20-01..04 |

## 12. 风险、难点与预研建议

### 12.1 实现灰区与取舍记录

| Clarification ID | 问题 | 选项与推荐 | 决策 / 答案 | 影响面 | 证据 | 重访条件 |
|---|---|---|---|---|---|---|
| N/A | 是否执行真实 broker lake 写。 | 推荐不执行；只做 dry-run contract。 | 已按用户 no-risk 边界处理。 | 安全 / runtime | Story 不授权范围 | 真实 broker lake 写需求出现时新开 runtime gate。 |

| 风险 / 难点 | 影响 | 缓解措施 / 预研建议 |
|---|---|---|
| Story 标题含“实盘写”导致误授权 | 高风险越界 | LLD/CP6/CP7 明确本轮只验证 dry-run schema/audit chain。 |
| 敏感值泄漏 | 安全风险 | 复用 redaction gate 并加 sensitive raw output tests。 |

### OPEN / Spike 跟踪

| ID | 类型（OPEN / Spike） | 问题 | 下一动作 | 责任方 |
|---|---|---|---|---|
| N/A | OPEN | 无开放问题。 | N/A | N/A |

## 13. 回滚与发布策略

- 发布方式：普通代码 + fixture tests；不发布数据、不连接 broker。
- 回滚触发条件：broker lake 既有 CR015 tests 回归失败或 CR139 audit chain helper 输出误导为真实写。
- 回滚动作：移除 CR139 audit chain helper/tests；保留既有 CR015 schema registry。

## 14. Definition of Done

- [x] 14 个章节全部填写完成
- [x] 文件影响范围、接口、测试与实施步骤可直接指导编码
- [x] 实现灰区与取舍记录已显式写“无真实写授权”
- [x] `confirmed=false` 时不进入实现，除非本 CR backlog 已授权无风险自动推进
- [x] OPEN / Spike 已清点

## 人工确认区

CP5 自动预检路径：`process/checks/CP5-CR139-S20-broker-lake-schema-audit-chain-LLD-IMPLEMENTABILITY.md`。

人工审查结果回填：

- 结论：`not-required-for-no-risk-inline-backlog`
- 审查人：N/A
- 审查时间：N/A
- 修改意见：无
- 风险接受项：无

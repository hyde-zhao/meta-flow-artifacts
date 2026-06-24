---
checkpoint_id: "CP3-CR138-RUNNER-QMT-HLD-REVIEW"
checkpoint_name: "CR138 Runner / QMT Gateway HLD Review"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-06-24T14:45:00+08:00"
reviewed_by: "user"
reviewed_at: "2026-06-24T15:30:00+08:00"
auto_check_result: "process/checks/CP3-CR138-HLD-CONSISTENCY.md"
target:
  phase: "solution-design"
  story_id: ""
  artifacts:
    - process/docs/design/HLD-RUNNER-QMT-OPERATIONAL-CONTROL-PLANE.md
    - process/docs/design/ARCHITECTURE-DECISION-RUNNER-QMT-OPERATIONAL-CONTROL-PLANE.md
    - process/context/CP3-CR138-RUNNER-QMT-HLD-CONTEXT.yaml
---

# CP3 CR138 Runner / QMT Gateway HLD 人工审查

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---|---|
| `process/checks/CP3-CR138-HLD-CONSISTENCY.md` | PASS | 0 | HLD、ADR、Blueprint / Domain / Dependency 增量、CP3 discussion 和 context 均已生成 |

## Decision Brief

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP3-CR138-RUNNER-QMT-HLD-CONTEXT.yaml` |
| capsule 状态 | `approved` |
| read_profile | `compact` |
| 默认读取策略 | 先读 capsule；仅在缺失、冲突、字段不足、人工审计或深度评审时读取完整文档 |
| 全文档读取扩展 | 已读取两份用户指定 QMT 研究报告、既有 QMT HLD、CR046 legacy HLD 和 strategy-runner-core design；均为只读 |
| 缺失 / waived 理由 | 无 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue | `process/state/STATE.current.json` | scanned | 1 | 0 | next_action 已指向本 CP3 gate，无额外 DQ |
| 自动预检结果 | `process/checks/CP3-CR138-HLD-CONSISTENCY.md` | scanned | 5 | 5 | DQ-CP3-CR138-01..05 纳入待决策 |
| discussion log / checkpoint | `process/discussions/CP3-HLD-DISCUSSION-LOG.md` / `process/checks/CP3-DISCUSSION-CHECKPOINT.json` | scanned | 6 | 5 | AGA-138-01..06 已收敛；HLD 拆分风险进入 DQ-04 |
| 下游正式产物 | HLD / ADR / BLUEPRINT / DOMAIN-MAP / DEPENDENCY-MAP | scanned | 5 | 5 | 架构、协议、授权、查询服务和 HLD 拆分均纳入 |
| 用户显式反馈 | 当前对话 | scanned | 3 | 3 | 功能命名 / REST-only / 按需授权 / 查询服务反馈已合并到 DQ-02、DQ-03、DQ-05 |

### 推荐决策

批准 Runner / QMT Gateway Operational Control Plane HLD，进入 CP4 Story planning。推荐架构为：新增 `FEAT-11 Runner Control Plane` 与 `FEAT-12 QMT Gateway Service Layer`，通过 `RunnerCommand`、`GatewayEvent`、`OrderIntent`、`ExecutionReport`、`AuthorizationRecord`、`AuditRecord` 和幂等键衔接；CP3 approve 不自动授权任何 runtime 或真实 QMT 操作，后续必要验证可按需走独立 `runtime_authorization` gate。

### 候选方案对比

| 方案 | 摘要 | 推荐结论 | 主要优点 | 主要代价 |
|---|---|---|---|---|
| A | 独立 Runner Control Plane + QMT Gateway Service Layer | 推荐 | 覆盖 UC-33..UC-50，边界清晰，安全可分层验证 | Story 数较多，CP4 需要拆分 |
| B | 扩展 offline `strategy-runner-core` 直接接 Gateway / xtquant | 不推荐 | 表面复用已有 runner artifact | 混淆 offline pass 与 runtime，安全风险高 |
| C | Gateway 同时调度策略和触达 QMT | 不推荐 | 单机 PoC 直观 | Gateway 业务化，策略和服务层不可分 |
| D | 只做只读 Gateway + Runner 观察面 | 条件备选 | 风险低 | 无法覆盖 UC-47/48 的订单回报和恢复建模 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP3-CR138-01 | architecture | 是否批准方案 A：独立 Runner Control Plane + QMT Gateway Service Layer？ | 批准；新增 FEAT-11 / FEAT-12 | 扩展 offline runner；Gateway-centered scheduler | 推荐方案边界清晰；备选短期少文件但长期混淆 | 决定 CP4 Story owner 与模块边界 | 可缩小为 Runner-only 或 Gateway-only |
| DQ-CP3-CR138-02 | architecture | Gateway P0 协议是否采用 REST-only，SSE/WebSocket、gRPC、FIX 后置？ | 采用 REST-only P0 | REST + SSE/WebSocket；gRPC-first | 推荐方案降低 P0 复杂度，REST 可覆盖查询、命令、订阅管理、状态/事件拉取和审计；备选适合实时推送或 SDK | 不能解决亚秒级实时推送和外部流式消费 | 出现实时 UI、低延迟行情流或 OMS/EMS 接入需求时另起协议 CR |
| DQ-CP3-CR138-03 | runtime_authorization | 是否确认 CP3 approval 不自动授权 runtime，但后续必要验证可以按需申请 runtime_authorization？ | 确认；后续按 action scope、运行窗口、脱敏、回滚和审计范围单独授权 | blanket deny；CP3 直接授权只读 / 行情 / 交易 | 推荐方案兼顾安全和验证可行性；blanket deny 阻断必要验证，直接授权风险过高 | 防止 HLD approval 变运行许可，同时保留验证路径 | 任何具体运行前必须重新发起 runtime_authorization gate |
| DQ-CP3-CR138-04 | risk_acceptance | 是否接受单份 umbrella HLD，CP4 再按 Runner/Gateway/Safety/OMS 拆 Story？ | 接受 | 立即拆 HLD；只保留单边范围 | 推荐方案避免重复 ADR；代价是 HLD 较长 | CP4 必须控制 Story 和文件 owner | 若评审认为过大，拆分 HLD 后重发 CP3 |
| DQ-CP3-CR138-05 | architecture | 是否确认交易日历、佣金 / 费用模型、收益 / PnL 查询进入 FEAT-12 Reference / Account Query Service，并按本地参考 / 账户只读授权分层？ | 确认；交易日历优先本地参考数据，佣金优先配置化 FeeSchedule，账户级收益 / 佣金 / 持仓 / 成交查询必须授权并脱敏 | 放在 FEAT-11；延后到 FEAT-06 OMS/Risk | 推荐方案支撑 preflight、费用估算和盘后复盘；放 Runner 会诱导越权账户读取，延后会缺核心运营能力 | 影响 Gateway query service 和 Runner preflight/review Story | QMT 不支持时使用 unavailable_with_reason 或 estimated mode，不伪造 broker facts |

### 用户需决策事项

- DQ-CP3-CR138-01：是否接受独立 Runner Control Plane + QMT Gateway Service Layer。
- DQ-CP3-CR138-02：是否接受 Gateway P0 REST-only，实时推送 / gRPC / FIX 后置。
- DQ-CP3-CR138-03：是否接受 CP3 不自动授权 runtime，但后续必要验证可按需授权。
- DQ-CP3-CR138-04：是否接受单份 umbrella HLD，CP4 再拆 Runner / Gateway / Safety / OMS Story。
- DQ-CP3-CR138-05：是否接受交易日历、佣金 / 费用模型、收益 / PnL 查询进入 FEAT-12 Query Service。

### 推荐回复

- `approve`
- `修改: <具体修改点>`
- `reject`

## Entry Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| CP2 场景基线已确认 | PASS | `process/USE-CASES.md` v1.16 | 用户 approve |
| CP3 HLD 已生成 | PASS | `process/docs/design/HLD-RUNNER-QMT-OPERATIONAL-CONTROL-PLANE.md` | approved-cp3 |
| CP3 自动预检通过 | PASS | `process/checks/CP3-CR138-HLD-CONSISTENCY.md` | PASS |
| 待决策项已聚合 | PASS | 本文件 Decision Brief | 5 项 |

## Checklist

| # | 检查项 | 状态 | 审查提示 |
|---|---|---|---|
| 1 | 是否接受推荐架构方案 A | approved | 用户回复 `approve`，接受推荐方案 |
| 2 | 是否接受新增 FEAT-11 / FEAT-12 边界 | approved | Runner 不直连 QMT；Gateway 不决定策略 |
| 3 | 是否接受 REST-only P0，SSE/WebSocket / gRPC / FIX 后置 | approved | 接受 REST-only P0；实时推送和 SDK 协议后置 |
| 4 | 是否确认 CP3 不自动授权 runtime，但后续可按需授权验证 | approved | 接受 scoped runtime_authorization policy；CP3 不自动授权 |
| 5 | 是否接受单份 umbrella HLD 风险 | approved | 接受；CP4 必须拆 Story |
| 6 | 是否接受交易日历、佣金和收益查询进入 FEAT-12 Query Service | approved | 账户级查询必须授权并脱敏 |

## Exit Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| 用户 approve 或提出修改 | approved | 用户回复 `approve` | 进入 CP4 Story planning |
| 若修改则回写 HLD / context / checkpoint | N/A | 用户未要求修改 | 已批准推荐方案 |
| 若 reject 则回退 solution-design | N/A | 用户未 reject | 不适用 |

## Deliverables

| 交付物 | 路径 | 状态 | 说明 |
|---|---|---|---|
| HLD | `process/docs/design/HLD-RUNNER-QMT-OPERATIONAL-CONTROL-PLANE.md` | approved | CP3 评审通过 |
| ADR | `process/docs/design/ARCHITECTURE-DECISION-RUNNER-QMT-OPERATIONAL-CONTROL-PLANE.md` | approved | ADR-CR138-001..007 接受为 CP4 输入 |
| Blueprint 增量 | `process/docs/design/BLUEPRINT.md` | approved | FEAT-11 / FEAT-12 作为 CP4 Feature 边界输入 |
| Domain Map 增量 | `process/docs/design/DOMAIN-MAP.md` | approved | OBJ-44..65 作为 CP4 领域对象输入 |
| Dependency Map 增量 | `process/docs/design/DEPENDENCY-MAP.md` | approved | FD-24..31 作为 CP4 依赖边界输入 |
| CP3 context | `process/context/CP3-CR138-RUNNER-QMT-HLD-CONTEXT.yaml` | approved | compact capsule |

## 人工审查结果

| 字段 | 内容 |
|---|---|
| 审查状态 | approved |
| 审查人 | user |
| 审查时间 | 2026-06-24T15:30:00+08:00 |
| 审查结论 | approved；接受 DQ-CP3-CR138-01..05 推荐方案，进入 CP4 Story planning |
| 修改要求 | 无 |
| 风险接受项 | 接受单份 umbrella HLD 在 CP4 按 Runner / Gateway / Safety / OMS 拆 Story；CP3 approval 不自动授权 runtime，后续验证必须单独 runtime_authorization gate |

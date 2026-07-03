---
status: draft
version: "1.0"
feature_id: "FEAT-PG-006"
feature_name: "Roadmap Refresh Governance"
source_design: "process/docs/features/roadmap-refresh-governance/DESIGN.md"
confirmed_by: ""
confirmed_at: ""
---

# Feature Test Plan: Roadmap Refresh Governance

## 测试范围

| Scope ID | 覆盖内容 | 来源 Story / Scenario | 测试层级 | 自动化状态 |
|---|---|---|---|---|
| UNIT-RF-01 | ROADMAP-REFRESH result 五类 decision 的字段校验。 | CR-F-S01 / HLD §17 | unit | planned |
| UNIT-RF-02 | checker 拒绝 unknown decision、缺失 required fields、错误 result type。 | CR-F-S01 / ADR-PG-005 | unit | planned |
| INTEG-RF-01 | process-only cascade 写入 process-side project objects。 | CR-F-S02 / ADR-PG-003 | integration | planned |
| INTEG-RF-02 | Gate Ledger 追加 `roadmap_refresh` event 并引用 result。 | CR-F-S02 / Domain OBJ-PG-013 | integration / contract | planned |
| INTEG-RF-03 | 发布库只读 stale evaluation 输出 `stale_items` 和 `follow_up_candidates`。 | UC-PG-006 / SIM-PG-002 | integration / manual | planned |
| CONTRACT-RF-01 | result schema、summary、event refs 供 stale-check / FU-RF 消费。 | FEAT-PG-007 / FEAT-PG-008 | contract | planned |
| MAN-RF-01 | 人工审查 result decision 与真实 roadmap delta 是否一致。 | CP5 / CP7 | manual | planned |

## 风险驱动测试

| Risk ID | 风险 | 测试方式 | 证据 | 未覆盖原因 |
|---|---|---|---|---|
| R-RF-01 | ROADMAP-REFRESH 误复用 CP result checker。 | 构造 CP result 作为输入，期望 roadmap-refresh checker FAIL。 | checker fixture / failure finding | 无。 |
| R-RF-02 | cascade 越权写 release repo。 | 构造 `machine_updates.target_ref` 指向 release code/tests/docs，期望 reject。 | security fixture / result=`BLOCKED` | 不做真实发布库写入。 |
| R-RF-03 | Gate Ledger event 缺失或 result_ref 不一致。 | 运行 ledger contract fixture，校验 event payload。 | ledger event fixture | 无。 |
| R-RF-04 | stale/follow-up 语义误报。 | 使用 fixture + 人工抽样确认。 | manual checklist | 语义陈旧判断不能完全靠结构测试。 |

## 权限 / 安全 / 失败路径

| Case ID | 触发条件 | 期望行为 | 测试入口 |
|---|---|---|---|
| SEC-RF-01 | result 尝试声明 release repo 文件被自动修改。 | checker FAIL，并要求转 stale/FU-RF 或正式 CR。 | security fixture |
| SEC-RF-02 | cascade target 不在 process artifact boundary。 | producer 拒绝写入，result decision 为 `BLOCKED`。 | integration fixture |
| SEC-RF-03 | checker 输入为 CP result 或 checkpoint result。 | FAIL wrong result type，不调用 CP result checker。 | contract fixture |
| SEC-RF-04 | Gate Ledger append 失败。 | 不伪造成功 event，result 记录 error。 | integration fixture |
| SEC-RF-05 | refresh 需要凭据、runtime、publish 或 production write。 | 阻断并输出授权缺失 finding。 | manual / security review |

## 单元测试建议

| Test ID | 对象 | 输入 | 期望 |
|---|---|---|---|
| UT-RF-001 | decision validator | `NO_CHANGE` 且存在 machine update | FAIL 或 WARN，按 LLD 规则确定。 |
| UT-RF-002 | decision validator | `UPDATED_WITH_DOC_IMPACTS` 缺少 stale_items | FAIL。 |
| UT-RF-003 | target guard | `machine_updates[*].target_ref=process/project/ROADMAP.yaml` | PASS。 |
| UT-RF-004 | target guard | `target_ref=docs/quality/TEST-STRATEGY.md` in release repo | FAIL / BLOCKED。 |
| UT-RF-005 | checker type guard | CP result JSON | FAIL wrong result type。 |

## 集成测试建议

| Test ID | 流程 | 输入 | 期望 |
|---|---|---|---|
| IT-RF-001 | refresh no-change | project/roadmap refs 无 delta | result=`NO_CHANGE`，无 machine_updates。 |
| IT-RF-002 | process update | ROADMAP 需要过程侧更新 | result=`UPDATED`，machine_updates 只含 process refs。 |
| IT-RF-003 | doc impacts | 发布库测试策略陈旧 | result=`UPDATED_WITH_DOC_IMPACTS`，输出 stale_items/FU-RF candidate。 |
| IT-RF-004 | blocked input | registry/project refs 缺失 | result=`BLOCKED`，无 release write。 |
| IT-RF-005 | ledger event | refresh 成功后 append event | Gate Ledger 含 `roadmap_refresh` + result_ref。 |

## 契约测试建议

| Contract ID | 消费方 | 契约 | 期望 |
|---|---|---|---|
| CT-RF-001 | FEAT-PG-007 | `follow_up_candidates[*].prefix` 可为 FU-RF/SP-RF/RA-RF。 | 不进入 RELEASE-CONTEXT。 |
| CT-RF-002 | FEAT-PG-008 | `stale_items[*]` 含 object_ref、expected、observed、severity。 | stale-check 可消费。 |
| CT-RF-003 | Gate Ledger | event 含 `event_type`、`result_ref`、`decision`、`status`。 | ledger checker PASS。 |
| CT-RF-004 | checkpoint-manager | refresh result 不作为 CP result。 | CP checker 不消费该 result。 |

## 手工验收

| Check ID | 操作 | 期望结果 | 责任方 |
|---|---|---|---|
| MAN-RF-01 | 审查一个 `UPDATED_WITH_DOC_IMPACTS` fixture。 | 发布库只出现 stale/follow-up，不出现自动修改。 | human / qa |
| MAN-RF-02 | 审查 Gate Ledger event 示例。 | event 能追溯到 result_ref 和触发 gate。 | human / se |
| MAN-RF-03 | 审查 checker failure 文案。 | 明确说明使用独立 roadmap-refresh checker，不建议复用 CP result。 | human / qa |
| MAN-RF-04 | 审查 release repo target guard。 | 禁止 code/tests/docs/formal release docs 自动写入。 | human / security |

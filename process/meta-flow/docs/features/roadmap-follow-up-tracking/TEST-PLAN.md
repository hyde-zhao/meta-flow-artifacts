---
status: draft
version: "1.0"
feature_id: "FEAT-PG-007"
feature_name: "Roadmap Follow-up Tracking"
source_design: "process/docs/features/roadmap-follow-up-tracking/DESIGN.md"
confirmed_by: ""
confirmed_at: ""
---

# Feature Test Plan: Roadmap Follow-up Tracking

## 测试范围

| Scope ID | 覆盖内容 | 来源 Story / Scenario | 测试层级 | 自动化状态 |
|---|---|---|---|---|
| UNIT-FU-01 | FU-RF / SP-RF / RA-RF regex 正反例。 | CR-G-S01 / ADR-PG-006 | unit | planned |
| UNIT-FU-02 | candidate 状态枚举校验。 | Domain OBJ-PG-014 | unit | planned |
| CONTRACT-FU-01 | candidate 必须包含 source_result_ref / source_stale_item_ref。 | FEAT-PG-006 consumer contract | contract | planned |
| INTEG-FU-01 | roadmap refresh result -> tracking entry。 | FLOW-FU-01 | integration | planned |
| INTEG-FU-02 | status query 可按 open / converted / waived / closed 分组。 | CR-G-S01 | integration | planned |
| SEC-FU-01 | RELEASE-CONTEXT 禁止包含 FU-RF/SP-RF/RA-RF。 | RULE-PG-010 | security / integration | planned |
| MAN-FU-01 | 人工确认 technical-note 是否足够或需升级 full-lld。 | Feature Matrix DQ-FD-002 | manual | planned |

## 风险驱动测试

| Risk ID | 风险 | 测试方式 | 证据 | 未覆盖原因 |
|---|---|---|---|---|
| R-FU-01 | roadmap follow-up 进入 RELEASE-CONTEXT。 | 构造 RELEASE-CONTEXT fixture 含 FU-RF，期望 FAIL。 | guardrail finding | 无。 |
| R-FU-02 | candidate 无来源导致不可追溯。 | 构造缺 source_result_ref 的 candidate，期望 invalid。 | contract fixture | 无。 |
| R-FU-03 | RA-RF 被误当成风险接受批准。 | 人工验收检查 RA-RF 状态只能是候选。 | manual checklist | 风险接受语义需人工确认。 |
| R-FU-04 | technical-note 覆盖不足。 | CP5 前审查文件影响面和 schema 改动。 | CP5 design review note | 需要结合实际 LLD diff。 |

## 权限 / 安全 / 失败路径

| Case ID | 触发条件 | 期望行为 | 测试入口 |
|---|---|---|---|
| SEC-FU-01 | RELEASE-CONTEXT 包含 FU-RF/SP-RF/RA-RF。 | guardrail FAIL，提示移出 release context。 | release context fixture |
| SEC-FU-02 | follow-up candidate 推荐直接修改 release docs/code/tests。 | 状态保持 open，要求转正式 CR / human gate。 | tracking fixture |
| SEC-FU-03 | candidate 缺少 roadmap refresh result 来源。 | invalid，不进入状态查询。 | contract fixture |
| SEC-FU-04 | RA-RF 被标记为 approved。 | FAIL，RA-RF 只能是候选或已转正式风险决策。 | manual / fixture |

## 单元测试建议

| Test ID | 对象 | 输入 | 期望 |
|---|---|---|---|
| UT-FU-001 | regex | `FU-RF001-roadmap-doc-stale` | PASS。 |
| UT-FU-002 | regex | `SP-RF002-capability-spike` | PASS。 |
| UT-FU-003 | regex | `RA-RF003-risk-acceptance` | PASS。 |
| UT-FU-004 | regex | `RF-001`、`FU001`、`CR-037` | FAIL / not match。 |
| UT-FU-005 | status validator | `open`、`converted_to_cr`、`waived`、`closed` | PASS。 |

## 集成测试建议

| Test ID | 流程 | 输入 | 期望 |
|---|---|---|---|
| IT-FU-001 | refresh result consumption | result 含 `follow_up_candidates` | 生成 tracking entry，status=open。 |
| IT-FU-002 | candidate conversion | open FU-RF 转正式 CR | status=converted_to_cr，记录 cr_ref。 |
| IT-FU-003 | release context guard | RELEASE-CONTEXT 含 FU-RF | FAIL，不进入 release decision evidence。 |
| IT-FU-004 | status query | 多种前缀和状态 | 按 prefix/type/status 分组输出。 |

## 契约测试建议

| Contract ID | 消费方 | 契约 | 期望 |
|---|---|---|---|
| CT-FU-001 | FEAT-PG-006 | follow_up_candidates 含 prefix、source_result_ref、recommended_route。 | tracking 可写。 |
| CT-FU-002 | release-readiness | RELEASE-CONTEXT 不含 FU-RF/SP-RF/RA-RF。 | release guardrail PASS。 |
| CT-FU-003 | host-orchestrator | RA-RF 不等于 approved risk acceptance。 | 需要正式 decision。 |
| CT-FU-004 | story-manager | CR-G-S01 lld_policy 有升级条件。 | CP5 可审查。 |

## 手工验收

| Check ID | 操作 | 期望结果 | 责任方 |
|---|---|---|---|
| MAN-FU-01 | 审查 FU-RF/SP-RF/RA-RF 示例。 | 编号、来源和推荐 route 清楚。 | human / se |
| MAN-FU-02 | 审查 RA-RF 风险接受路径。 | RA-RF 只是候选，未绕过人工确认。 | human / security |
| MAN-FU-03 | 审查 CR-G-S01 文件影响面。 | 若触及多个 tracking 模块或新增 schema，则升级 full-lld。 | human / host-orchestrator |
| MAN-FU-04 | 审查 RELEASE-CONTEXT fixture。 | 没有 FU-RF/SP-RF/RA-RF 条目。 | human / qa |

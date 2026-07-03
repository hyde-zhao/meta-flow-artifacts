---
status: draft
version: "1.0"
feature_id: "FEAT-PG-009"
feature_name: "Quant-lab Migration Readiness"
source_design: "process/docs/features/quant-lab-migration-readiness/DESIGN.md"
confirmed_by: ""
confirmed_at: ""
---

# Feature Test Plan: Quant-lab Migration Readiness

## 测试范围

| Scope ID | 覆盖内容 | 来源 Story / Scenario | 测试层级 | 自动化状态 |
|---|---|---|---|---|
| UNIT-QL-01 | migration 默认 dry-run，不 apply。 | CR-H-S01 / RULE-PG-012 | unit | planned |
| UNIT-QL-02 | migration report schema required fields。 | OBJ-PG-016 | unit / contract | planned |
| SEC-QL-01 | release repo code/tests/docs/formal docs target 被拒绝。 | ADR-PG-003 / RULE-PG-012 | security / integration | planned |
| SEC-QL-02 | process-side target boundary guard。 | FEAT-PG-001/003 | security / integration | planned |
| SEC-QL-03 | credential / token / account read request 被拒绝。 | CR-037 authz policy | security | planned |
| INTEG-QL-01 | dry-run 生成 planned_process_changes 和 blocked_findings。 | FLOW-QL-01 | integration | planned |
| INTEG-QL-02 | process-side apply 记录 rollback refs。 | FLOW-QL-02 | integration | planned |
| INTEG-QL-03 | unresolved capability 输出 blocked finding，不自动注册。 | ADR-PG-004 | integration / contract | planned |
| INTEG-QL-04 | impact migration unknown surface 输出 report。 | FEAT-PG-005 | integration | planned |
| INTEG-QL-05 | stale formal docs 输出 stale report / FU-RF candidate。 | FEAT-PG-007/008 | integration / manual | planned |
| MAN-QL-01 | 人工审查真实迁移计划未写 quant-lab 发布库。 | CP5 / CP7 | manual | planned |

## 风险驱动测试

| Risk ID | 风险 | 测试方式 | 证据 | 未覆盖原因 |
|---|---|---|---|---|
| R-QL-01 | 误写 quant-lab 发布库。 | forbidden target fixture + manual release repo diff review。 | security finding / manual checklist | 不对真实发布库执行写入。 |
| R-QL-02 | dry-run 被绕过。 | 默认 command fixture 无 apply authorization。 | dry-run report | 无。 |
| R-QL-03 | capability 自由字符串被自动注册。 | unresolved capability fixture。 | blocked finding | 无。 |
| R-QL-04 | process-side cleanup 无回滚。 | apply fixture 校验 rollback_refs。 | rollback fixture | 无。 |
| R-QL-05 | stale report 被当成正式修复。 | manual review candidate status。 | manual checklist | 需要人工确认发布库未改。 |

## 权限 / 安全 / 失败路径

| Case ID | 触发条件 | 期望行为 | 测试入口 |
|---|---|---|---|
| SEC-QL-01 | planned change 指向 quant-lab release repo code/tests/docs。 | FAIL / blocked finding，不写入。 | forbidden target fixture |
| SEC-QL-02 | process-side target 越界。 | reject planned change。 | boundary fixture |
| SEC-QL-03 | adapter 尝试读取 credential/token/account/private key。 | FAIL。 | security fixture |
| SEC-QL-04 | command 请求 publish/live/trading/production write。 | FAIL。 | authz fixture |
| SEC-QL-05 | apply 未经 human gate。 | 只输出 dry-run report。 | command fixture |

## 单元测试建议

| Test ID | 对象 | 输入 | 期望 |
|---|---|---|---|
| UT-QL-001 | mode resolver | command without apply flag | `mode=dry_run`。 |
| UT-QL-002 | report schema | 完整 migration report | PASS。 |
| UT-QL-003 | report schema | 缺 `release_repo_readonly_refs` | FAIL。 |
| UT-QL-004 | target guard | process-side target | PASS。 |
| UT-QL-005 | target guard | release repo docs target | FAIL。 |

## 集成测试建议

| Test ID | 流程 | 输入 | 期望 |
|---|---|---|---|
| IT-QL-001 | dry-run clean | process refs + readonly release refs | report complete，无 apply。 |
| IT-QL-002 | process apply | authorized process-side target | writes process-side fixture，rollback_refs present。 |
| IT-QL-003 | unresolved capability | capability not in registry | blocked finding，不创建 ID。 |
| IT-QL-004 | impact migration | legacy impact records | normalized report / unknown report。 |
| IT-QL-005 | stale report | release doc stale observation | stale finding + FU-RF candidate。 |
| IT-QL-006 | rollback | applied process-side fixture | rollback restores before_hash。 |

## 契约测试建议

| Contract ID | 消费方 | 契约 | 期望 |
|---|---|---|---|
| CT-QL-001 | FEAT-PG-001 | process-side current state cleanup 走 state checker/writer。 | allowlist PASS。 |
| CT-QL-002 | FEAT-PG-004 | capability refs 必须 registry-resolved。 | unresolved -> blocked finding。 |
| CT-QL-003 | FEAT-PG-005 | legacy impact migration 输出 normalized report。 | unknown surface 不静默成功。 |
| CT-QL-004 | FEAT-PG-008 | stale report 可被 stale-check result 消费。 | finding refs 稳定。 |
| CT-QL-005 | FEAT-PG-007 | follow-up candidate 不进入 RELEASE-CONTEXT。 | FU-RF tracking only。 |

## 手工验收

| Check ID | 操作 | 期望结果 | 责任方 |
|---|---|---|---|
| MAN-QL-01 | 审查 dry-run migration report。 | planned changes 全部为 process-side；release repo readonly refs 仅观察。 | human / qa |
| MAN-QL-02 | 审查发布库 diff 或等价证据。 | 代码、tests、正式 docs、release docs 无自动修改。 | human / security |
| MAN-QL-03 | 审查 blocked findings。 | unresolved capability / stale docs 有明确后续 route。 | human / se |
| MAN-QL-04 | 审查 rollback refs。 | process-side apply 可回退。 | human / qa |
| MAN-QL-05 | 审查 authz evidence。 | 没有 credential read、runtime、publish、production write、trading。 | human / security |

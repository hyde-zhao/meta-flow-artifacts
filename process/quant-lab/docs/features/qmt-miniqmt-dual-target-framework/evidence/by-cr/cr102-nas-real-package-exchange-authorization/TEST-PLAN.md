---
feature_id: "CR102-NAS-REAL-PACKAGE-EXCHANGE-AUTHORIZATION"
title: "CR102 NAS Real Package Exchange Authorization Test Plan"
status: "ready-for-cp5-review"
source_cr: "CR-102"
checkpoint: "CP5"
created_at: "2026-06-21T06:32:36+08:00"
owner: "host-orchestrator"
---

# CR102 NAS Real Package Exchange Authorization Test Plan

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| v0.1 | 2026-06-21 | host-orchestrator | 初版，定义 CP5 design-only 验证策略、负向用例和后续证据验收规则。 |

## 1. 验证目标

本测试计划只验证设计材料是否 fail-closed、可审计、可作为后续授权门禁输入。不验证真实 NAS，不执行 mount/list/read/check/copy/publish/pull/delete/write。

## 2. Validation Mode

| 层级 | 方式 | 是否触碰外部系统 |
|---|---|---:|
| static_design_review | 检查 authorization matrix、execution plan shell、evidence schema | 否 |
| ledger_consistency | 检查 CR102 / CR-INDEX / STATE 状态一致 | 否 |
| redaction_contract_review | 检查 allowed / forbidden evidence fields | 否 |
| runtime_authorization_guardrail | 检查 CP5 approve 不授权 runtime/NAS | 否 |

## 3. 测试场景

| ID | 场景 | 输入 | 预期 |
|---|---|---|---|
| TP-CR102-01 | 所有 NAS 操作默认 false | DESIGN authorization matrix | access/list/read/write/copy/publish/pull/delete/mount/check 全 false |
| TP-CR102-02 | path/mount/identity 未提供 | `UNSET_BY_USER` 字段 | execution plan shell 输出 fail closed |
| TP-CR102-03 | 用户只提供 path 但未授权 check | path 非空，check=false | 仍 blocked，不进入 check |
| TP-CR102-04 | publish/delete 被请求 | publish/delete=true 请求 | 要求 high-risk gate，当前 CP5 不放行 |
| TP-CR102-05 | evidence 含 secret/account/raw log | forbidden field 出现 | blocked_redaction_failed |
| TP-CR102-06 | CR089/QMT runtime 被引入 | QMT/runtime/account 需求 | route blocked，切独立 runtime_authorization CR |
| TP-CR102-07 | CP5 approve | 用户 approve CP5 | 仅进入 future authorization readiness，不授权 NAS |

## 4. 证据验收规则

| 规则 | 验收条件 |
|---|---|
| Evidence 最小化 | 只允许 redacted status/hash/count/pass-fail |
| Sensitive field denylist | forbidden fields 出现即失败 |
| Path redaction | 未脱敏完整路径不得进入证据 |
| Runtime isolation | 证据不得来自启动 QMT/MiniQMT/XtQuant/gateway |
| Operation scope | evidence 中的 operation 必须与 future gate 明确授权一致 |

## 5. 不自动化原因

当前阶段不实现 checker 或 CLI，因为即使只读 checker 也可能被误执行为 NAS check。CP5 只允许设计审查；任何自动化 checker 或真实执行脚本必须在后续 gate 中单独确认是否可实现、是否可运行、是否需要 dry-run-only。

## 6. 回归范围

| 对象 | 原因 |
|---|---|
| CR100 fake exchange runbook | 防止真实 NAS readiness 与 fake exchange 混淆 |
| CR101 follow-up gate docs | 防止 CR102 被误读为 QMT/MiniQMT/order-write 授权 |
| CR089 blocked state | 防止 CR102 恢复更大 runtime 语义 |

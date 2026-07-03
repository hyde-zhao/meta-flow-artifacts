---
status: draft
version: "1.0"
feature_id: "FEAT-PG-001"
---

# Current State Enforcement Test Plan

## 测试范围

| Scope ID | 覆盖内容 | 来源 Story / Scenario | 测试层级 | 自动化状态 |
|---|---|---|---|---|
| TP-CS-01 | unknown top-level key audit/enforce | CR037-S01 / TC-PG-001 / TC-PG-018 | unit + CLI | planned |
| TP-CS-02 | `source_refs`、`open_risks`、`authz_policy_refs` 列表预算 | CR037-S01 / TC-PG-003 | unit | planned |
| TP-CS-03 | `next_action.text` 标量预算 | CR037-S01 / TC-PG-003 | unit | planned |
| TP-CS-04 | `update_current_state()` deep-merge 和 patch key allowlist | CR037-S02 / TC-PG-002 | unit + integration | planned |
| TP-CS-05 | CR lifecycle 不再 direct write current state | CR037-S02 / TC-PG-002 | integration | planned |
| TP-CS-06 | agent contract 禁止直编未知字段 | CR037-S03 / TC-PG-004 | static + manual | planned |

## 风险驱动测试

| Risk ID | 风险 | 测试方式 | 证据 | 未覆盖原因 |
|---|---|---|---|---|
| R-CS-01 | 黑名单漏掉新自造字段 | 构造 `action_history` / `decision_log` fixture | pytest + state check output | N/A |
| R-CS-02 | 合法列表字段承载长正文 | 构造超 max_items / max_total_bytes fixture | pytest | N/A |
| R-CS-03 | 内部路径绕过 writer | monkeypatch `write_text` 或检查 `_update_current_active_change` 行为 | integration test | N/A |
| R-CS-04 | audit/enforce 灰度误伤 | audit 模式应 WARN 不阻断，enforce 模式应 FAIL | pytest parametrized | N/A |

## 权限 / 安全 / 失败路径

| Case ID | 触发条件 | 期望行为 | 测试入口 |
|---|---|---|---|
| SEC-CS-TC-01 | current state 包含 `token` / `credential` 字段 | unknown key FAIL，且错误提示不打印敏感值 | pytest fixture |
| SEC-CS-TC-02 | agent 直接写入 disallowed 字段 | `meta-flow state check` FAIL | CLI test |
| SEC-CS-TC-03 | `project_state_ref` 缺少 optional allowlist | 测试失败，提示补 allowlist | unit test |

## 手工验收

| Check ID | 操作 | 期望结果 | 责任方 |
|---|---|---|---|
| MAN-CS-01 | 审查 agent contract 文案 | 明确禁止直编 `STATE.current.json` 未约定字段，并给出合法更新入口 | human / se |
| MAN-CS-02 | 审查 migration note | `last_actions` 类自造字段不被迁移为新语义，只清理或归入既有 ledger/ref | human / qa |

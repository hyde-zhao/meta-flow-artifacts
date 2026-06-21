---
fixes_id: "CR092-REAL-QMT-READONLY-RUNTIME-SMOKE-FIXES"
change_id: "CR-092"
status: "N/A"
created_at: "2026-06-18T17:05:00+08:00"
created_by: "host-orchestrator"
---

# CR092 Real QMT Readonly Runtime Smoke Fixes

## 回修结论

本轮 CP7 未发现需要回修的 P0 / P1 / P2 实现缺陷。

## 已处理事项

| 项目 | 处理 | 证据 |
|---|---|---|
| checker forbidden marker 误判 `forbidden_counters.*` | 已在 CP6 实现阶段修正：计数字段名允许存在，但值必须为 0 | `scripts/check_cr092_simulated_evidence.py` / `tests/test_cr092_simulated_evidence_checker.py` |

## 未回修事项

| 项目 | 原因 | 后续 |
|---|---|---|
| 真实 runtime 未验证 | 当前未授权，不是实现缺陷 | 独立逐 run runtime 授权 |
| 未读取用户实际模拟账户 evidence | 当前只提供模板，且用户未指定 evidence 文件路径 | 用户提供后运行 checker |
| CR019 / CR025 历史账本旧账 | 非 CR092 实现范围 | CR091-FU-04 ledger hygiene |

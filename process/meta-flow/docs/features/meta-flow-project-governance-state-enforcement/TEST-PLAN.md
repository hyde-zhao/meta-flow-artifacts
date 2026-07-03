---
title: "Test Plan: Meta Flow Project Governance and State Enforcement"
status: "baseline"
created_at: "2026-07-02"
owner: "host-orchestrator"
cr_ref: "CR-037"
---

# Test Plan: Meta Flow Project Governance and State Enforcement

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| v1.0 | 2026-07-02 | host-orchestrator | 建立 Feature 测试计划基线。 |

## 测试范围

| 层级 | 用例 |
|---|---|
| Unit | allowlist、budget、deep-merge patch、ROADMAP-REFRESH schema |
| Integration | CR lifecycle 更新 current state、workspace bootstrap、event ledger append |
| CLI | state check、ledger compact、project stale-check、roadmap-refresh check |
| Migration dry-run | quant-lab state cleanup、impact surface migration、capability refs readiness |
| Guardrail | agent 写契约、delivery rules 同步 |

## 核心用例

| ID | 验证点 | 期望 |
|---|---|---|
| T-01 | unknown current state key in audit | WARN |
| T-02 | unknown current state key in enforce | FAIL |
| T-03 | `source_refs` 超预算 | FAIL |
| T-04 | `update_current_state()` deep-merge | 只更新 patch 字段 |
| T-05 | CR lifecycle 不直接 `write_text()` | 测试或静态检查通过 |
| T-06 | workspace bootstrap 创建 `process/project` | PASS |
| T-07 | `PROJECT.current.json` 超 16KB | FAIL |
| T-08 | `impact_surface: ["process/checks"]` | enforce FAIL |
| T-09 | 未注册 capability ref | FAIL / blocked finding |
| T-10 | ROADMAP-REFRESH result 缺 decision | FAIL |
| T-11 | ROADMAP-REFRESH 试图写发布库 | FAIL 或转 stale item |
| T-12 | FU-RF candidate id | cr-tracking 接受 |

## 回归范围

- `tests/test_state_v2.py`
- CR lifecycle tests
- workspace routing tests
- cr-tracking tests
- product governance / capability claims tests
- CP result tests，确认未被 ROADMAP-REFRESH 污染

## 不执行项

- 不运行 live trading。
- 不读取真实凭据。
- 不对 quant-lab 发布库执行自动写入。

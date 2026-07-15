---
status: completed
version: "1.0"
story_id: "CR170-S01-na-policy-inventory-five-state-contract"
story_slug: "na-policy-inventory-five-state-contract"
feature_id: "FEAT-15"
implementation_type: "code-and-tests"
source_story: "process/stories/STORY-CR170-S01-na-policy-inventory-five-state-contract.md"
source_design_evidence: "process/stories/STORY-CR170-S01-na-policy-inventory-five-state-contract-LLD.md"
created_by: "host-orchestrator-inline-meta-dev"
created_at: "2026-07-15T15:21:00+08:00"
updated_at: "2026-07-15T15:21:00+08:00"
---

# Implementation: CR-170 S01 N/A policy inventory 与五态合同

## 1. 实现摘要

| 项目 | 内容 |
|---|---|
| 实现目标 | 建立 21-unit immutable inventory、五态分类器、4/4 boundary 校验和稳定 reason ID。 |
| 行为变化 | 新增 private repository-local 纯函数模块；尚未改变任何 canonical Gate 行为。 |
| 范围边界 | 不读取授权系统；`authorization_ref` 仅作 opaque pointer；不修改 adapters、aggregate 或 runner。 |
| CP6 证据 | `process/checks/CP6-CR170-S01-IMPLEMENTATION-DONE.result.json` |

## 2. 实现前置检查

| 检查项 | 结果 | 说明 |
|---|---|---|
| CP5 / LLD | PASS | 用户已批准，`open_items=0`。 |
| Story 依赖 | PASS | S01 无上游 Story 依赖。 |
| 文件所有权 | PASS | 只创建 S01 两个 primary files。 |
| 授权边界 | PASS | repository-local fixture/static；无外部读取或远端写入。 |

## 3. 实现对象与设计契约映射

| 对象 | 设计要求 | 实现 | 验证 |
|---|---|---|---|
| `NaPolicySpec` inventory | 21/21、6/6/1/5/3 | `NA_POLICY_SPECS` / immutable index | exact count tests |
| direction/disposition | 15/5/1、20/1 | typed enums | group assertions |
| five-state classifier | fixed precedence | `classify_na_evidence` | 5/5 state tests |
| boundary | reason/owner/scope/profile-or-auth | `_boundary_is_complete` | removal/mismatch tests |
| reason ID | stable closed categories | `build_na_reason_id` | exact string/closed enum tests |

## 4. 最小实现切片

| Slice | 文件 | 结果 |
|---|---|---|
| S01-01 | `engine/reliability_na_policy.py` typed contract | done |
| S01-02 | exact inventory 与 classifier | done |
| S01-03 | `tests/research/test_reliability_na_policy.py` | 16 passed |

## 5. 验证结果

| 检查 | 结果 |
|---|---|
| S01 targeted pytest | PASS，16 passed |
| py_compile | PASS |
| git diff --check | PASS |
| public export break | 0 |

## 6. 设计差异与未覆盖项

设计差异为 `0`。Gate-local consumer、status floor 与 admission policy 分别由 S02/S03 串行实现；本 Story 不生成 Gate status。

## 7. 风险、回滚与后续交接

- 若 inventory 数量、15/5/1 分组或 boundary precedence 回归，删除 private module即可回到历史行为；不得把 generic reason 恢复为 evidence。
- S02 必须复用 `NA_POLICY_SPECS`，不得维护第二份 inventory。
- 当前验证由 Host Orchestrator inline 执行，独立 verifier 仍是 future consumer。

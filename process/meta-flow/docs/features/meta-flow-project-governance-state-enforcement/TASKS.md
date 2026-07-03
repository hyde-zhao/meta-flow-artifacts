---
title: "Tasks: Meta Flow Project Governance and State Enforcement"
status: "baseline"
created_at: "2026-07-02"
owner: "host-orchestrator"
cr_ref: "CR-037"
---

# Tasks: Meta Flow Project Governance and State Enforcement

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| v1.1 | 2026-07-02 | host-orchestrator | 同步 CR-037 激活和 CP0 PASS 后的 Gate 状态，保留 CP2/CP3/CP5 阻断。 |
| v1.0 | 2026-07-02 | host-orchestrator | 建立 CR-037 任务基线。 |

## P0 State Enforcement

| Task | 状态 | 说明 |
|---|---|---|
| T-P0-01 | TODO | 定义 current state allowlist 和 field budget |
| T-P0-02 | TODO | 扩展 `check_current_state()` 为 allowlist + disallowed + size/budget |
| T-P0-03 | TODO | `write_current_state()` 写前校验 |
| T-P0-04 | TODO | 新增 `update_current_state()` deep-merge API |
| T-P0-05 | TODO | 收敛 `cr_lifecycle._update_current_active_change()` 直写路径 |
| T-P0-06 | TODO | 更新 agent / skill 写契约 |
| T-P0-07 | TODO | 接入 CP/state check gate |
| T-P0-08 | TODO | 在所有 per-feature 设计和实现审查中落实 ST-PG-013 / TC-PG-018 的 second-system guardrail |

## P0.5 Ledger Hygiene

| Task | 状态 | 说明 |
|---|---|---|
| T-P05-01 | TODO | 设计 ledger compact policy |
| T-P05-02 | TODO | 实现 ledger compact / archive / index 命令 |
| T-P05-03 | TODO | 补充 ledger compact 测试 |
| T-P05-04 | TODO | 在 FEAT-PG-002 per-feature 设计中落实 ST-PG-012 / TC-PG-017 的命令边界验收 |

## P1 Project Governance

| Task | 状态 | 说明 |
|---|---|---|
| T-P1-01 | TODO | `PROCESS_SCAFFOLD_DIRS` 加入 `project` |
| T-P1-02 | TODO | 定义 `PROJECT.current.json` schema/checker |
| T-P1-03 | TODO | 定义 `PROJECT-SCALE.yaml` |
| T-P1-04 | TODO | 定义 `ROADMAP.yaml` / `MILESTONES.yaml` |
| T-P1-05 | TODO | 实现 impact surface normalization audit/enforce |
| T-P1-06 | TODO | 实现 `affected_paths` / `feature_refs` / `capability_refs` 校验 |
| T-P1-07 | TODO | 实现 ROADMAP-REFRESH result schema/checker |
| T-P1-08 | TODO | 实现 roadmap cascade 边界和 GATE-LEDGER event |
| T-P1-09 | TODO | 支持 FU-RF / SP-RF / RA-RF |
| T-P1-10 | TODO | 实现 project stale-check |

## P2 Quant-lab Migration

| Task | 状态 | 说明 |
|---|---|---|
| T-P2-01 | TODO | quant-lab `STATE.current.json` cleanup dry-run |
| T-P2-02 | TODO | quant-lab capability 来源归一到标准 registry |
| T-P2-03 | TODO | quant-lab impact_surface migration report |
| T-P2-04 | TODO | quant-lab project objects 生成 |
| T-P2-05 | TODO | quant-lab roadmap stale report 和 FU-RF 输出 |

## Gate

| Gate | 状态 | 说明 |
|---|---|---|
| G-01 | DONE | 用户已授权暂停未完成的 `CR-036` 并激活 `CR-037`；`CR-036` 仍 blocked / unfinished，不可视为完成或关闭 |
| G-02 | PASS | CR-037 CP0 bootstrap readiness 已通过，见 `process/checks/CP0-CR-037-BOOTSTRAP.result.json` |
| G-03 | TODO | CP2 场景/需求确认 |
| G-04 | TODO | CP3 HLD 确认 |
| G-05 | BLOCKED | CP3 通过后按 FEAT-PG-001..009 生成 per-feature 设计证据，再进入 CP5 |

---
document_type: story-verification-report
cr_id: CR-051
story_id: ST-AW-001
checkpoint: CP7
decision: NEEDS_REWORK
verified_by: meta-qa-critical
materialized_by: host-orchestrator
verified_at: 2026-07-18T11:48:00Z
---

# CR-051 / ST-AW-001 验证执行报告

## 结论

`NEEDS_REWORK`。基础正向与既有回归全部通过，但独立负向验证发现 1 个跨项目 fail-closed 阻断缺陷和 1 个结构化错误契约缺陷。ST-AW-001 不得标记 verified，W2 runtime gate 保持关闭。

本报告固化两个真实 `meta-qa-critical` 执行 attempt 的结论；首个 attempt 已向 Host 返回可复现 finding，后续证据落盘 lane 超时，由 Host 在不改变 QA 事实、结论或严重度的前提下完成协议文件物化。源码与测试未由 QA 或 Host 在本步骤修改。

## 验证范围与对象

| 对象 | 证据 |
|---|---|
| Story / AC | `process/stories/STORY-ST-AW-001-project-first-routing.md` |
| CP6 结果 | `process/checks/CP6-CR051-ST-AW-001-project-first-routing-CODING-DONE.result.json` |
| CP6 return / evidence | `process/returns/ST-AW-001.CP6.return.json`、`process/evidence/ST-AW-001.CP6.index.json` |
| SUT | `meta_flow/workspace/project_artifact_routing.py`、`meta_flow/workspace/routing.py` |
| 定向测试 | `tests/test_cr051_project_artifact_routing.py`、`tests/test_workspace_routing.py` |

## 追踪矩阵

| 需求 / 场景 | 验证状态 | 证据 / 说明 |
|---|---|---|
| project-first docs/process 唯一路由 | PASS | 定向 pytest 正向、布局与 relocation fixtures 通过 |
| legacy/project-first 歧义 fail-closed | PASS | 既有负向 fixture 通过 |
| project identity 跨对象一致性 | FAIL | `CP7-AW-001-F01`：config/decision project 不一致仍返回 health.ok |
| portable decision/digest | PASS | 重复解析与 relocation fixtures 通过 |
| 非法路径、未知 layout、target ownership | PASS | 既有负向 fixture 通过 |
| 结构化错误契约 | FAIL | `CP7-AW-001-F02`：legacy 非字符串 `owned_paths` 泄漏裸 `TypeError` |
| TC-AW-001/002/003/010/012 适用断言 | PARTIAL | 正向与大部分负向已覆盖；上述两个边界缺口必须补测试后重验 |
| sibling / 真实 mutation = 0 | PASS | 验证仅使用临时对象与只读检查；真实 Git/worktree/link/remote mutation 为 0 |

## 执行证据

| 命令 / 检查 | 结果 |
|---|---|
| `pytest -q tests/test_cr051_project_artifact_routing.py tests/test_workspace_routing.py` | PASS，36 passed |
| `ruff check`（本 Story 4 个源码/测试文件） | PASS |
| `python -m py_compile`（两个源码文件） | PASS |
| `git diff --check` | PASS |
| `meta-flow check module-boundaries` | N/A：canonical `docs/design/MODULE-BOUNDARIES.yaml` 缺失，未伪报 PASS |
| `meta-flow check imports` | N/A：同一 manifest 前置缺失 |
| 跨项目 config/decision mismatch 负例 | FAIL：返回 `health.ok=True`，并采用 project-b target |
| malformed legacy `owned_paths` 负例 | FAIL：抛裸 `TypeError` |

## Findings

### CP7-AW-001-F01 — BLOCKER — adapter 未校验 config/decision 项目身份一致性

- 输入：`config.project_id=project-a`，`decision.project_id=project-b`，decision 为 process/PASS 且带 project-b primary target。
- 实际：`project_route_to_process_health(...)` 返回 `status=ok` / `health.ok=True`，`actual_target` 指向 project-b。
- 期望：在投影 legacy health 前 100% fail closed，输出 `route_mismatch` 和可行动错误；不得采用跨项目 target。
- 影响：下游若把 legacy health 当写入授权，可把当前项目动作路由到 sibling 项目，违反 owned-path 与多项目隔离基线。
- 修复边界：在 adapter 内显式校验 `decision.project_id == config.project_id`，并补 config/decision identity mismatch 回归测试；不改变公共 layout 或分支拓扑。

### CP7-AW-001-F02 — MAJOR — legacy `owned_paths` 类型错误未结构化

- 输入：legacy layout 的 `owned_paths` 含非字符串元素。
- 实际：`_normalize_config()` 的 legacy 分支把元素直接传入路径规范化，泄漏裸 `TypeError`。
- 期望：返回稳定 `RoutingValidationError`（`route_conflict` 或等价既有错误码），字段精确到 `owned_paths.<index>`，包含 repair route。
- 影响：CLI/health consumer 无法稳定分类和给出修复动作，破坏 fail-closed 错误契约。
- 修复边界：复用或提取 owned-path 元素类型校验，并补 legacy 非字符串负例；不得扩展 YAML 语法范围。

## 剩余风险

- canonical `MODULE-BOUNDARIES.yaml` 缺失导致两个仓库级 checker 不适用；修复后 CP7 重验仍需显式报告。
- 真实双设备 portability pilot 未获授权；当前只认可临时 device-root fixture 证据。

## 下一路由

返回 `meta-dev-debugger` 修复 `CP7-AW-001-F01/F02`，补定向回归，重新生成 CP6 return/evidence/result 后再次执行独立 CP7。无需 design delta，除非修复需要改变已批准的公共 schema/layout/错误码集合。

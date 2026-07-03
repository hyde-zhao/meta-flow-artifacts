---
story_id: "CR037-S02"
cr_id: "CR-037"
title: "controlled update API and writer refactor"
stage: "CP6"
status: "implemented"
owner: "meta-dev"
created_at: "2026-07-03T10:25:09+08:00"
context_ref: "process/context/stories/CR037-S02.CP6.work-packet.json"
lld_ref: "process/stories/STORY-CR037-S02-controlled-update-api-and-writer-refactor-LLD.md"
---

# CR037-S02 Implementation

## 实现前置检查

| 条目 | 结论 | 证据 |
|---|---|---|
| Story 状态可实现 | PASS | Story card `status=dev-ready`；work packet stage=`CP6` |
| 依赖门控 | PASS_WITH_RISK | S01 CP7 `PASS_WITH_RISK`，继续携带 `R-CR037-SECOND-MECHANISM` / `R-CR037-REGISTRY-DRIFT` |
| 写入范围 | PASS | 仅修改用户允许的代码、测试和 S02 CP6 证据文件 |
| 禁止范围 | PASS | 未写 `process/quant-lab/**`、`delivery/**`、`docs/**` 或凭据类路径 |
| 设计证据 | PASS | 消费 S02 LLD 的 patch allowlist、deep-merge、no-delete、no-write 和 lifecycle 收敛契约 |

## 实现对象清单

| 对象 | 文件 | 实现内容 | 验证方式 |
|---|---|---|---|
| 受控 state update API | `meta_flow/state/current.py` | 新增 `StateValidationError`、`validate_current_patch()`、`_deep_merge_current_state()`、`update_current_state()`；复用 S01 allowlist / budget / secret-like / full-state validation | `tests/test_state_v2.py` |
| state writer | `meta_flow/state/current.py` | 将 JSON 写入集中到 `_write_current_state_file()`，先完成 validation，再以临时文件 replace 写入 | no-write unit tests + state enforce |
| CR lifecycle active writer | `meta_flow/workflow/cr_lifecycle.py` | `_update_current_active_change()` 改调用 `current.update_current_state()`，不再直接写 `STATE.current.json` | `tests/test_cr_lifecycle.py` |
| 单元测试 | `tests/test_state_v2.py` | 覆盖 patch allowlist、deep-merge、list replacement、`None` 非删除、预算失败不落盘、missing state | pytest |
| 收敛测试 | `tests/test_cr_lifecycle.py` | 用 `wraps` 验证 lifecycle 调用受控 API，且 state 内容真实更新 | pytest |

## 设计契约映射

| LLD / AC 契约 | 实现位置 | 验证 |
|---|---|---|
| patch 顶层 key 必须 allowlist | `validate_current_patch()` | `test_update_current_state_rejects_unknown_patch_key_without_write` |
| dict deep-merge | `_deep_merge_current_state()` | `test_update_current_state_deep_merges_dict_fields` |
| list / scalar / null 整体替换 | `_deep_merge_current_state()` | `test_update_current_state_replaces_lists`、`test_update_current_state_none_is_replacement_not_delete` |
| 禁止删除语义；`None` 不是删除 | 不实现 delete sentinel；`None` 按普通值替换并交由 full-state validation | `test_update_current_state_none_is_replacement_not_delete` |
| candidate full-state enforce 失败不落盘 | `update_current_state()` 在写入前运行 `validate_current_state_payload()` | `test_update_current_state_budget_failure_does_not_write` |
| missing state 不静默创建 | `update_current_state()` 对缺失 `STATE.current.json` 抛 `FileNotFoundError` | `test_update_current_state_missing_state_is_not_created` |
| lifecycle direct write 收敛 | `_update_current_active_change()` 调用 `current.update_current_state()` | `test_update_current_active_change_uses_controlled_state_api` |
| 不新增第二套机制 | 复用 `meta_flow/state/current.py` 现有 allowlist / validation / path 常量；未新增 registry、ledger 或 patch log | rg 扫描 + diff review |

## 单元测试与 Fixture 计划

| 测试项 | 状态 | 说明 |
|---|---|---|
| target unit tests | DONE | `tests/test_state_v2.py tests/test_cr_lifecycle.py` 覆盖 S02 验收项 |
| adjacent regression | DONE | `tests/test_context_pack.py tests/test_story_context_contract.py tests/test_workspace_routing.py` |
| state enforce | DONE | `meta-flow state check --project-root . --mode enforce` |
| return/evidence/cp contracts | DONE | return-check、evidence-check、cp result-check |
| forbidden-path diff | DONE | `git diff -- process/quant-lab delivery docs` 无输出 |
| diff whitespace | DONE | `git diff --check` 通过 |

## 最小实现切片

| Slice | 内容 | 回滚点 | 结果 |
|---|---|---|---|
| S02-1 | 在现有 current-state v2 模块内实现受控 patch API 和 deep-merge | 恢复 `current.py` S02 新增函数 | PASS |
| S02-2 | 将 CR lifecycle active change writer 收敛到受控 API | 恢复 `_update_current_active_change()` 调用点 | PASS |
| S02-3 | 添加状态 API 与 lifecycle convergence 测试 | 移除新增测试方法 | PASS |
| S02-4 | 生成 CP6 return / evidence / result / summary | 删除 S02 CP6 证据文件 | PASS |

## 平台差异处理

N/A。本 Story 修改 Python 内部 state writer 与测试，不涉及 Claude / Codex / OpenClaw agent schema、安装路径、AskUserQuestion 或平台分支。

## 验证结果

| 命令 | 结果 | 摘要 |
|---|---|---|
| `PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 pytest -q tests/test_state_v2.py tests/test_cr_lifecycle.py` | PASS | `28 passed, 7 subtests passed` |
| `PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 pytest -q tests/test_context_pack.py tests/test_story_context_contract.py tests/test_workspace_routing.py` | PASS | `18 passed` |
| `PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 meta-flow state check --project-root . --mode enforce` | PASS | `State v2 Check: OK` |

## 未覆盖项

- 未实现 S03 agent contract；按 Story 明确非目标处理。
- 未新增删除 API 或 patch log；删除语义明确为不支持，`None` 是普通替换值。
- 未修改长期 Feature DESIGN / ADR / HLD，因此不生成 design delta。

## 设计缺口反馈

无阻断设计缺口。两个下游风险继续携带：

- `R-CR037-SECOND-MECHANISM`：S02 继续只使用现有 `meta_flow/state/current.py` 机制，未新增第二套 state/context/result/ledger/registry。
- `R-CR037-REGISTRY-DRIFT`：S02 未新增或修改 registry / design refs。

## 后续交接

交给 meta-qa 进行 CP7 验证时，优先检查：

- `update_current_state()` 对 patch allowlist、deep-merge、list replacement、`None` 非删除和 no-write 的行为。
- `_update_current_active_change()` 是否已消除 direct JSON write。
- CP6 return / evidence / result contract 是否通过工具校验。
- forbidden-path、runtime、production write、publish、live、trading、credential-read 边界是否保持为无授权无执行。

---
project_id: "meta-flow"
change_id: "CR-051"
story_id: "ST-AW-004"
stage: "CP7"
report_type: "test-report"
decision: "PASS_WITH_RISK"
tested_at: "2026-07-18T14:33:23Z"
tested_by: "meta-qa-critical (qa-yan)"
---

# CR-051 / ST-AW-004 测试报告

## 摘要

| 指标 | 结果 |
|---|---|
| 组合回归 | 179 passed + 9 subtests |
| aggregate 专属收集项 | 62 |
| 4×4 状态矩阵 | 16/16 PASS |
| invalid 参数分类 | 14/14 PASS |
| non-PASS 零 projector | 15/15 PASS |
| nested digest / stale targeted | 2/2 PASS |
| BLOCKING 失败 | 0 |
| 阶段建议 | `PASS_WITH_RISK` |

## 验证环境

- OS：Linux。
- Python：3.11.15。
- pytest：9.1.1。
- 依赖执行：`uv run --python 3.11`。
- 缓存/字节码：pytest 使用 `-p no:cacheprovider`，运行测试设置 `PYTHONDONTWRITEBYTECODE=1`；py_compile 输出定向到 `/tmp/cr051-staw004-cp7-pycompile`。
- 文件系统：仅 pytest 临时 fixture；未访问真实 remote/worktree 或生产目标。

## 测试设计方法

| 方法 | 应用 | 证据 |
|---|---|---|
| 决策表 | source/artifact 四状态笛卡尔积 | 16 个具名参数用例 |
| 等价分区 | raw/missing/duplicate/correlation/mode/ref/digest/key/receipt/schema/read failure | 14 个具名 invalid 参数用例 + unpublished 探针 |
| 状态转换 | persist → readback → current selector → projector | PASS、HOLD、FAILED、retryable 路径 |
| 并发/故障注入 | 同 payload、冲突 payload、selector stale、projector failure | memory/file store 并发与 fault fixture |
| 变形测试 | 只改顶层 digest、只改 nested leg digest | canonical digest targeted test |
| 回归 | leg producer/consumer、Git branch、workspace sync、state v2、CR lifecycle | 179-test 组合 suite |

## 命令与结果

### 组合回归

```text
PYTHONDONTWRITEBYTECODE=1 PYTEST_ADDOPTS='-p no:cacheprovider' uv run --python 3.11 pytest -q tests/test_artifact_leg_lifecycle.py tests/test_artifact_aggregate.py tests/test_git_branch_lifecycle.py tests/test_workspace_git_sync.py tests/test_state_v2.py tests/test_cr_lifecycle.py
```

结果：`179 passed, 9 subtests passed in 4.54s`。

### 关键参数组

| 选择器 | Selected | Passed | 关键断言 |
|---|---:|---:|---|
| `fixed_precedence_covers_all_16_status_combinations` | 16 | 16 | 固定优先级、仅 PASS/PASS eligible、每 leg 重读一次 |
| `invalid_or_unpublished_handles_are_rejected_before_aggregate` | 14 | 14 | invalid set/payload/handle 全部拒绝 |
| `non_pass_results_persist_but_never_call_projection` | 15 | 15 | `called=false`、projector calls=0 |
| nested digest + stale projection | 2 | 2 | nested digest 篡改变更 canonical digest；stale projector=0 |

### 额外运行探针

| 探针 | 结果 |
|---|---|
| `published=false` handle | `validated=false`；错误码 `unpublished-result`；未形成 aggregate |
| `readback_valid=false` receipt | `HOLD`；projector calls=0 |
| `current_selected=false` receipt | `HOLD`；projector calls=0 |
| `disposition=conflict` receipt | `HOLD`；projector calls=0 |
| `disposition=failed` receipt | `HOLD`；projector calls=0 |

### 静态检查

| 检查 | 结果 |
|---|---|
| Ruff check（aggregate/test/CR lifecycle/state current） | PASS：`All checks passed!` |
| Ruff format check（aggregate/test） | PASS：2 files formatted |
| py_compile（aggregate/CR lifecycle/state current/test） | PASS |
| aggregate forbidden dependency scan | PASS：Git/worktree/subprocess/manual sync/rollback/close/status-sync 匹配 0 |
| scoped projector call review | PASS：只进入 current completion-candidate writer 和 CR ledger append |
| Git diff | N/A：任务明确禁止执行 Git；CP6 evidence 已记录 touched-files diff check PASS |

## 状态组合结果

| source \ artifact | BLOCKED | FAIL | IN_PROGRESS | PASS |
|---|---|---|---|---|
| BLOCKED | BLOCKED / HOLD | BLOCKED / HOLD | BLOCKED / HOLD | BLOCKED / HOLD |
| FAIL | BLOCKED / HOLD | FAIL / HOLD | FAIL / HOLD | FAIL / HOLD |
| IN_PROGRESS | BLOCKED / HOLD | FAIL / HOLD | IN_PROGRESS / HOLD | IN_PROGRESS / HOLD |
| PASS | BLOCKED / HOLD | FAIL / HOLD | IN_PROGRESS / HOLD | PASS / ELIGIBLE |

实际 16 项与上表一致率 100%。只有右下角 PASS/PASS 进入 ELIGIBLE；其余 15 项调用 `project_if_eligible` 后均返回 HOLD 且 projector 调用数为 0。

## Invalid / unpublished 覆盖

已覆盖 raw payload、missing leg、duplicate leg、wrong attempt、wrong project、wrong mode、wrong result ref、wrong payload digest、wrong single-write key、wrong receipt digest、wrong handle correlation、wrong handle schema、wrong payload schema、reader failure；另以运行探针确认 `published=false` 返回 `unpublished-result`。所有路径在 aggregate persistence/projector 前失败，store/projector 副作用为 0。

## Canonical 与证据 DAG

- AggregateResult `payload_digest` 仅从顶层 canonical 对象中排除。
- `published_handle_refs.source.payload_digest` 被替换为 `0`×64 后 canonical digest 不再等于原 aggregate digest。
- AggregateResult 顶层不包含 `aggregate_ref`、write/append receipt、receipt digest、writer id、written_at。
- evidence 边保持 payload → external receipt → validated handle ref → aggregate → aggregate receipt → controlled projection，无自身回边。

## 持久化、CAS 与投影门

- same-input 重试：同一 aggregate ref、结果数 1，第二次 disposition=`idempotent-existing`。
- same aggregate ID + conflicting payload：disposition=`conflict`，原 payload 未覆盖。
- selector CAS 冲突：仅 1 个 current_selected，冲突 writer 不替换 current；last-write-wins=0。
- projection happy path：persisted/readback/current 2/2 PASS 调用 projector 1 次并 COMPLETE。
- non-PASS/stale/readback false/current false/conflict/failed：projector 调用 0。
- projector 内部失败：aggregate 保持 immutable，返回 retryable failure；不会重跑 Git。

## CLI dry-run

`test_cr_aggregate_cli_dry_run_outputs_refs_without_writes` 通过：输出 overall PASS 和 aggregate 决策，但 `write_receipt=null`、`projection_receipt=null`，临时项目中 `process/evidence/aggregates` 未创建。该结论不覆盖真实 persist/project-completion 命令授权。

## 覆盖缺口与剩余风险

| 缺口 | 影响 | 处置 |
|---|---|---|
| native Windows 跨进程 lock crash 未运行 | 不能证明 Windows crash recovery；后续 writer 预期 fail closed | CP8 披露；独立环境验证或风险接受 |
| 真实 hosted remote/worktree 未运行 | 不证明生产拓扑与远端行为 | 保持未授权；不影响本 Story 的纯 aggregate/projection gate |
| workflow eval 未运行 | Story packet 明确 `workflow_eval_required=false` | 以原生组合回归和静态证据替代；若 CP8 改变该要求需重开验证 |

## 测试结论

所有 BLOCKING 测试门通过，无失败或需要实现回修的测试缺陷。因 native Windows lock crash 与真实 remote/worktree 边界未验证，测试结论为 `PASS_WITH_RISK`。

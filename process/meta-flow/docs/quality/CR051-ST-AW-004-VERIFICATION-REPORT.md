---
project_id: "meta-flow"
change_id: "CR-051"
story_id: "ST-AW-004"
feature_id: "FEAT-AW-04"
stage: "CP7"
validation_mode: "mixed"
risk_profile: "runtime-high-risk"
decision: "PASS_WITH_RISK"
verified_at: "2026-07-18T14:33:23Z"
verified_by: "meta-qa-critical (qa-yan)"
---

# CR-051 / ST-AW-004 独立验证报告

## 1. 结论

| 项目 | 内容 |
|---|---|
| 阶段决策 | `PASS_WITH_RISK` |
| validation_mode | `mixed`（原生运行测试 + 静态边界审查 + fixture 文件系统） |
| BLOCKING 缺陷 | 0 |
| 实现回修 | 不需要 |
| 下一路由建议 | `host-orchestrator` 汇总 CP7；剩余风险进入 CP8 风险披露，不代表批准 gate |

ST-AW-004 的五项量化验收均有独立运行证据：4×4 状态矩阵 16/16 与固定优先级一致；invalid/unpublished 输入 fail closed；aggregate 自身 digest 只在顶层 omit 且嵌套 leg digest 仍被绑定；immutable append、same-input 幂等及 selector CAS 冲突均不采用 last-write-wins；只有 persisted/readback/current 的 2/2 PASS 才调用 projector，15 个 non-PASS、stale、receipt conflict/failure 路径调用数均为 0；CLI dry-run fixture 写入 aggregate store 数为 0。

结论不是 CP7 gate 批准，也不授权 Git、worktree、manual sync、close、status-sync、rollback、真实 remote 或生产写入。

## 2. 验证范围

### 2.1 范围内

- `PublishedLegResultHandle` 的 required-set、schema、correlation、mode、result ref、payload digest、receipt digest、single-write key 与重读校验。
- `BLOCKED > FAIL > IN_PROGRESS > PASS` 的 4×4 全组合纯聚合决策。
- `AggregateResult` canonical digest、嵌套 leg digest binding、无自身持久化 ref/receipt 字段。
- immutable result append、readback、same-input 幂等、conflicting payload 与 selector CAS。
- persisted/readback/current 2/2 PASS projection hard gate，以及 non-PASS、stale、conflict、failure 零 projector 调用。
- 受控 state/CR ledger completion-candidate 投影边界；保持 CR/story/phase active，不 close/status-sync/rollback。
- CR aggregate CLI handler dry-run 零 aggregate store / projection 写入。
- aggregate、producer/consumer 相邻边界、state v2、CR lifecycle、Git branch/workspace sync 相邻回归。

### 2.2 明确不在范围内

- 不修改实现、测试、验收标准、设计或状态机真相源。
- 不执行 Git/worktree/manual sync/close/status-sync/rollback。
- 不访问凭据，不执行真实托管 remote/worktree，不写生产目标。
- native Windows 跨进程锁崩溃恢复未执行；Linux fixture 只证明 fail-closed 行为。
- workflow eval：Story packet 明确 `workflow_eval_required=false`，本轮以原生组合回归、静态检查和 fixture 证据替代，未读取或运行额外 eval suite。

## 3. 验证对象清单

| 对象 | 类型 | 验证方式 | 结果 |
|---|---|---|---|
| `meta_flow/workflow/artifact_aggregate.py` | 聚合、持久化、CAS、projection guard | 全文审查、专属测试、组合回归、Ruff、py_compile、禁止依赖扫描 | PASS |
| `meta_flow/state/current.py` | controlled state writer | 全文审查、state v2 回归、projection integration fixture | PASS |
| `meta_flow/workflow/cr_lifecycle.py` | CR projector 与 aggregate CLI | 全文审查、CR lifecycle 回归、controlled projection / CLI fixture | PASS |
| `tests/test_artifact_aggregate.py` | 单元/集成/负向 fixture | 62 项完整纳入组合回归；关键参数组单独复跑 | PASS |
| Story、CP6 return/evidence/result、Feature TEST-PLAN | 需求、契约与上游证据 | capsule-first 追踪核对 | PASS |
| CP7 报告、return、evidence | 交接证据 | return/evidence checker | 见 CP7 return/evidence index |

## 4. 验证追踪矩阵

| 验收 / 契约 | 实现入口 | 测试 / 检查 | 独立结果 | 风险 |
|---|---|---|---|---|
| 4×4 共 16 状态、固定优先级、仅 PASS/PASS eligible | `validate_published_leg_results`、`compute_aggregate` | `test_fixed_precedence_covers_all_16_status_combinations`；16 项 `-vv` 复跑 | 16/16 PASS | 无 |
| invalid/raw/unpublished 输入写入前 BLOCKED | `_required_set_errors`、`_validate_one_handle`、`coordinate_aggregate` | 14 类 invalid 参数组；`published=false` 独立探针；invalid coordinate 零 store/projector | PASS；显式 unpublished code=`unpublished-result` | 无 |
| top-level-only digest omit，nested leg digest binding | `canonical_json_digest`、`compute_aggregate`、`_canonical_result_payload` | `test_aggregate_digest_omits_only_its_own_digest_and_binds_nested_leg_digests` | PASS | 无 |
| immutable store、same-input 幂等、冲突 fail closed | `InMemoryAggregateStore`、`FileAggregateStore`、`persist_aggregate` | immutable/file/concurrent same/conflicting payload tests | PASS；last-write-wins=0 | stale lock 风险见 R-AW004-LOCK-CRASH |
| selector CAS | `compare_and_set_current` | current selector CAS conflict、并发冲突测试 | PASS；冲突不替换 current | native Windows 未实测 |
| persisted/readback/current 2/2 PASS 才投影 | `project_if_eligible` | 2/2 PASS、stale、15 non-PASS、readback/current/conflict/failed receipt 探针 | PASS；允许路径调用 1，其余调用 0 | 无 |
| 不 close/status-sync/rollback/Git/worktree | `AggregateCompletionProjector`、`project_aggregate_completion` | monkeypatch forbidden calls、静态 scoped review、aggregate 禁止词扫描 | PASS；aggregate 禁止匹配 0 | 真实 remote/worktree 明确未授权 |
| CLI dry-run 零写 | `aggregate_main`、`coordinate_aggregate(dry_run=True)` | `test_cr_aggregate_cli_dry_run_outputs_refs_without_writes` | PASS；write/projection receipt 均为 null，store path 不存在 | 无 |
| 相邻回归 | aggregate/leg/state/CR/Git lifecycle/workspace sync | 6 文件组合测试命令 | 179 passed + 9 subtests | 无 |

## 5. 设计契约验证清单

| Contract | 来源 | 阻断性 | 验证结果 |
|---|---|---|---|
| 必须重读 `result_ref`，不得信任 embedded/raw payload | Story + TP-AW04-005/017 | BLOCKING | PASS：reader 调用 source/artifact 各 1；embedded payload 不影响结果 |
| required legs 固定为 `source, artifact` 且各一次 | Story + TP-AW04-007 | BLOCKING | PASS：missing/duplicate/raw 均 BLOCKED |
| canonical digest 只忽略 AggregateResult 顶层自身 digest | acceptance + CP6 修复证据 | BLOCKING | PASS：nested source digest 篡改改变 aggregate canonical digest |
| immutable append + readback 后才能 selector CAS | TP-AW04-009..011 | BLOCKING | PASS：readback mismatch/failure 不选择 current；冲突 fail closed |
| 仅 current 2/2 terminal PASS 可投影 | Story + TP-AW04-012/013 | BLOCKING | PASS：15 non-PASS、stale 与失败收据调用均为 0 |
| projection 只记录 completion candidate，不结束 CR | Story 禁止边界 | BLOCKING | PASS：active change/story/phase 保持，事件为 `aggregate_projection`，无 closed 事件 |
| dry-run 不持久化、不投影 | TP-AW04-016 | BLOCKING | PASS |
| aggregate 不拥有 Git/worktree/manual sync | Story file ownership | BLOCKING | PASS：聚合模块静态扫描匹配 0；未执行相关命令 |

## 6. 分层验证计划与执行

| 层级 | 要求 | 执行 | 结果 |
|---|---|---|---|
| 单元 / 决策表 | 16 状态组合、canonical、invalid | 专属 pytest 参数组与独立探针 | PASS |
| 持久化 / 并发 fixture | immutable append、same/conflicting writers、CAS | memory + `tmp_path` file store | PASS |
| 集成 | aggregate → state current → CR ledger projector | controlled projection integration test | PASS |
| CLI dry-run | 显式 handle 输入、零 aggregate store 写 | CLI handler fixture | PASS |
| 相邻回归 | producer/consumer、state v2、CR lifecycle | 179-test 组合命令 | PASS |
| 静态质量 | Ruff、format、py_compile | 4 个 SUT/test 文件 | PASS |
| 依赖/权限边界 | Git/worktree/subprocess/manual sync/close/status-sync/rollback | scoped source review + `rg` | PASS |
| Git diff | packet 同时禁止 Git 执行 | 本轮 N/A；复用 CP6 evidence 中已通过的 touched-files `git diff --check` | N/A，不降低本轮结论 |
| native Windows | 跨平台锁 crash 行为 | 未授权/无环境 | NOT RUN；剩余风险 |

## 7. 自动化验证结果

| 命令 / 探针 | 结果 |
|---|---|
| `pytest -q` aggregate + leg + branch + workspace sync + state v2 + CR lifecycle | `179 passed, 9 subtests passed in 4.54s` |
| 4×4 参数组单独复跑 | `16 passed, 46 deselected` |
| invalid 参数组单独复跑 | `14 passed, 48 deselected` |
| non-PASS projection 参数组单独复跑 | `15 passed, 47 deselected` |
| nested digest + stale projection 单独复跑 | `2 passed, 60 deselected` |
| `published=false` 探针 | `validated=false`，code=`unpublished-result` |
| readback/current/conflict/failed receipt 探针 | 4/4 `hold`、`called=false`、projector calls=0 |
| Ruff check | `All checks passed!` |
| Ruff format check | `2 files already formatted` |
| py_compile | exit 0 |
| aggregate forbidden dependency scan | 匹配 0（`rg` exit 1 表示无匹配） |

## 8. Fixture 与 CLI 验证

- Memory reader 只按显式 `result_ref` 返回 payload，并记录重读次数。
- Memory/File aggregate store 分别覆盖确定性、同线程/跨线程同 payload 与冲突 payload。
- `tmp_path` controlled state fixture 验证 active CR/story/phase 不被关闭或投影为 delivered。
- CLI dry-run fixture 断言 `write_receipt=null`、`projection_receipt=null`，且 `process/evidence/aggregates` 不存在。
- fixture 只写测试临时目录；本轮 QA 未写真实 artifact/worktree/ref/remote/link/migration 目标。

## 9. 平台与安全边界

- Linux / Python 3.11.15 本地 fixture：PASS。
- aggregate 模块无 Git、worktree、subprocess、manual sync、close/status-sync/rollback 依赖或调用。
- CR lifecycle 文件包含其他独立命令的 Git/close/status-sync 入口，但 `AggregateCompletionProjector` 的调用闭包只进入 `project_aggregate_completion` 与 CR ledger append；本轮没有调用其他入口。
- native Windows lock crash 未验证；已保留 fail-closed 风险，不能据此授权自动恢复或清锁。
- 真实 hosted remote/worktree 处于 `NO_RUNTIME` / `NO_REPOSITORY_PUBLICATION` 边界，不属于本 Story 通过内容。

## 10. 人工 / 语义质量审查

- 需求一致性：固定优先级、2/2 PASS、PARTIAL 仅 effect、无 overall PARTIAL 与实现一致。
- 错误路径：invalid、unpublished、read failure、digest/receipt/key/correlation/mode/schema 错误均在 aggregate persistence 前收敛为 BLOCKED。
- happy path 偏差：通过 15 个 non-PASS、stale、conflict、failure 负向路径校验，未只验证 PASS/PASS。
- 错误可行动性：validation code、receipt disposition、next route 与 retryable 字段可区分证据修复、持久化冲突和投影重试。
- 文档可交接性：Story、Feature test plan、CP6 return/evidence 与本报告形成 acceptance → implementation → command → result → risk 链路。

## 11. 问题清单

无 BLOCKER / HIGH / MEDIUM / LOW 实现缺陷；无需路由 `NEEDS_REWORK`。

## 12. 剩余风险

| Risk ID | 等级 | 状态 | Owner | 接受/关闭条件 |
|---|---|---|---|---|
| `R-AW004-LOCK-CRASH` | LOW | OPEN / fail-closed | CP8 decision owner | native 平台 crash fixture 证明恢复手册和并发 writer 不绕过 CAS，或明确接受人工诊断恢复 |
| `CR051-RISK-WINDOWS-CROSS-PROCESS-LOCK` | LOW | OPEN | CP8 decision owner | 在 native Windows 执行跨进程竞争与 crash fixture；当前不得宣称已覆盖 |
| `CR051-RISK-REMOTE-UNVERIFIED` | MEDIUM | accepted boundary | CP8 decision owner | 仅在独立 runtime authorization 后验证真实 remote/worktree；本结论不授权执行 |

## 13. 阶段决策与 CP8 输入

- 推荐阶段决策：`PASS_WITH_RISK`。
- next route：交还 host-orchestrator 汇总；不由 meta-qa 写 CP7 result，不批准 CP7/CP8 gate。
- CP8 必须披露 native Windows lock crash 未验证及真实 remote/worktree 未授权；两者不得被静默转为已验证能力。
- 无 design delta、waiver、open question 或实现回修项。

---
status: implemented
version: "1.2"
story_id: "ST-AW-004"
story_slug: "aggregate-evidence-gate"
feature_id: "FEAT-AW-04"
implementation_type: "mixed"
source_story: "process/stories/STORY-ST-AW-004-aggregate-evidence-gate.md"
source_design_evidence: "process/stories/STORY-ST-AW-004-aggregate-evidence-gate-LLD.md"
created_by: "meta-dev"
created_at: "2026-07-18T14:02:56+00:00"
updated_at: "2026-07-19T00:00:00+00:00"
---

# Implementation: ST-AW-004 — Aggregate Evidence Gate

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-18 | meta-dev | 完成 validator、纯聚合、immutable persistence、CAS、projection guard、CLI/shared controlled writer 与 CP6 证据。 |
| 1.1 | 2026-07-18 | meta-dev | 修复组合回归发现的 canonical digest 缺陷：`omit_keys` 只移除 AggregateResult 顶层自身 digest，嵌套 leg payload digests 必须参与聚合 digest；增加专门回归并复跑 6 文件组合。 |
| 1.2 | 2026-07-19 | Host Orchestrator（inline-fallback） | CP8 终验回修：实现 consumer-side target policy、canonical CLI mode、PARTIAL staged projection 与幂等 retry，补齐依赖/DAG/target-policy 回归。 |

## 1. 实现摘要

| 项目 | 内容 |
|---|---|
| 实现目标 | 只消费显式 source/artifact published handles，经 result_ref 重读与 receipt/digest/key/correlation 校验后计算 16 组合固定优先级，持久化 immutable aggregate 并用 single-writer selector CAS 控制投影资格。 |
| 行为变化 | 新增 `meta-flow cr aggregate` 显式入口；dry-run 零写，默认只持久化；仅显式 `--project-completion` 且 persisted/readback/current 的 2/2 terminal PASS 才调用 controlled CR ledger/current writer。 |
| 范围边界 | 不读取或修改 ST-AW-003 primary；aggregate 不 import/call Git、worktree、leg executor、manual sync；non-PASS/stale/conflict/readback failure 不 close、不 rollback、不自动完成 CR。 |
| CP6 证据 | 本文、`process/returns/ST-AW-004.CP6.return.json`、`process/evidence/ST-AW-004.CP6.index.json`；Host 尚未批准 CP6。 |

## 2. 上游设计引用

| 来源 | 路径 / ID | 本次消费内容 |
|---|---|---|
| Story | `process/stories/STORY-ST-AW-004-aggregate-evidence-gate.md` | AC、W3 contract gate、primary/shared owner 和禁止边界。 |
| Story 设计证据 | `process/stories/STORY-ST-AW-004-aggregate-evidence-gate-LLD.md` | schema、validator、4×4 precedence、immutable persistence、external receipt、CAS、projection 二阶段门和 CLI。 |
| Feature DESIGN | `process/docs/features/cr051-aggregate/DESIGN.md` | `BLOCKED > FAIL > IN_PROGRESS > PASS`、PARTIAL 非 overall、evidence DAG、single writer。 |
| Feature TEST-PLAN | `process/docs/features/cr051-aggregate/TEST-PLAN.md` | TP-AW04-001..020 的 unit/negative/concurrency/projection/CLI 验证义务。 |
| CP5 | `process/checkpoints/CP5-CR051-ALL-STORIES-LLD-BATCH.md` | 全量设计批次已批准；实现不等于 CP6/CP7 批准。 |

## 3. 实现前置检查

| 检查项 | 结果 | 证据 / 说明 |
|---|---|---|
| process route health | PASS | Story work packet、`CURRENT.json.health=ok` 与 Host handoff 已确认；未创建或修复 process route。 |
| 上游 Feature 设计存在 | PASS | aggregate DESIGN/TEST-PLAN 已读取。 |
| Story 范围明确 | PASS | Story 为 `dev-ready`，`lld_gate.confirmed=true`。 |
| 待确认问题已关闭 | PASS | LLD `open_items=0`；无 blocking clarification。 |
| 完整 LLD deep review | PASS | `READ-EXPANSION-LEDGER.ndjson` 事件 `RE-20260718T133253Z0000-staw004`。 |
| 当前 Wave / dev_gate 满足 | PASS | W3 contract 依赖在 CP5 R2 冻结；primary 文件与 ST-AW-003 分离。 |
| 文件所有权无冲突 | PASS | ST-AW-004 独占 aggregate/tests，并为 CR-051 CLI/shared projection merge owner。 |
| 权限边界 | PASS | 只修改 work packet `allowed_write_paths`；未执行真实 Git/worktree/ref/remote/publish。 |

## 4. 实现对象清单

| 对象 | 类型 | 目标 | 必须 | 验证方式 |
|---|---|---|---|---|
| `meta_flow/workflow/artifact_aggregate.py` | code / schema | validator、pure aggregate、immutable store、external receipt、CAS、projection guard、coordinator | yes | 定向 pytest、ruff、py_compile、import scan |
| `tests/test_artifact_aggregate.py` | guardrail-test | 16 组合、invalid/tamper、readback、idempotency、conflict、projection、CLI | yes | pytest |
| `meta_flow/state/current.py` | code / controlled writer | persisted PASS aggregate ref 的 allowlisted state projection；保留 active CR/story/phase | yes | 定向 + state v2 regression |
| `meta_flow/workflow/cr_lifecycle.py` | code / controlled writer / CLI | CR ledger receipt、state writer 组合和 `cr aggregate` 显式命令 | yes | 定向 + CR lifecycle regression |
| `meta_flow/cli.py` | CLI docs | 顶层帮助发现 aggregate 入口 | yes | help/ruff/pycompile |
| 本 IMPLEMENTATION / return / evidence | docs-handoff | CP6 可恢复、可审计交接 | yes | Story return/evidence checker |

## 5. 设计契约映射

| 设计要求 | 来源 | 实现位置 | 实现动作 | 验证 |
|---|---|---|---|---|
| 只消费 reread-validated published handles | LLD F-01..03 | `validate_published_leg_results`、`ProjectFileLegResultReader` | 拒绝 raw/missing/duplicate；逐项校验 payload/receipt/ref/key/correlation/mode | invalid matrix、embedded payload ignored |
| 16 组合固定优先级，仅 PASS/PASS 可 ELIGIBLE | LLD F-04/05 | `compute_aggregate` | pure function；PARTIAL 仅 effect | 16 参数组合 100% |
| payload 先定 digest，receipt 外置且不回写 | LLD F-06 / §5.2 | `AggregateResult`、`persist_aggregate` | aggregate payload 无自身 ref/receipt/writer/time；external receipt 独立 | deterministic/no-self-reference tests |
| immutable append + readback + selector CAS | LLD §5.2/7 | `InMemoryAggregateStore`、`FileAggregateStore` | exclusive append、canonical readback、explicit expected current ref、lock-dir CAS | same/conflicting concurrency fixtures |
| persisted/readback/current 2/2 PASS 才投影 | LLD F-07 / §8.3 | `project_if_eligible`、`AggregateCompletionProjector`、`project_aggregate_completion` | 投影前重新读取 aggregate/current selector；shared writer 再校验 PASS 字段 | positive projection + non-PASS/stale negative matrix |
| projection failure 不改 aggregate、不重跑 leg | LLD F-08 | `ProjectionReceipt` / coordinator | FAILED/PARTIAL 显式可重试，immutable aggregate 保留 | failure injection fixture |
| CLI 显式且 dry-run 零写 | LLD F-09 | `aggregate_main` / top-level help | 显式 handles/attempt/modes；projection 另需 flag + state CAS token | CLI dry-run fixture |
| 不执行 Git/worktree/manual sync/close/rollback | Story/LLD must-not | aggregate import graph + shared projector | 不 import producer/executor；controlled writer只写 CR ledger/current evidence candidate | forbidden-import scan + monkeypatched close/status-sync test |

## 6. 单元测试 / Fixture 计划

| 测试对象 | 类型 | 输入 / Fixture | 期望 | 覆盖风险 | 状态 |
|---|---|---|---|---|---|
| precedence | property/unit | source/artifact 4×4 | 16/16 与固定表一致；仅 PASS/PASS eligible | 假 PASS | passed |
| published validator | negative/contract | raw、missing、duplicate、stale、wrong identity/mode/schema/digest/ref/key/receipt | 100% BLOCKED；无 persistence/projection | 不可信证据被降级 | passed |
| immutable aggregate | unit | 相同 validated set、self-reference scan、嵌套 leg digest 篡改 | ID/digest稳定；只忽略自身顶层 digest；嵌套 leg digest 必须绑定；禁止字段为0 | evidence 自引用 / nested digest 被误排除 | passed |
| persistence/readback | fixture | memory/file stores、tamper | immutable readback，digest一致 | 写入后篡改 | passed |
| single writer | concurrency | 两个相同/冲突 writer | same input idempotent；不同 input 单 current、另一个 conflict | last-write-wins | passed |
| projection guard | unit/integration | PASS、15 个 non-PASS、stale、failure | 仅 persisted/readback/current PASS 调用；其他调用0 | non-PASS current completion | passed |
| controlled writer | integration | 临时 STATE.current + CR ledger | evidence ref/ledger receipt写入；active CR/story/phase不关闭 | 隐式 close/status-sync | passed |
| CLI | dry-run | 显式 handle JSON/ref | structured PASS；aggregate store不存在 | 隐式 mutation | passed |
| adjacent regressions | regression | state v2 + CR lifecycle | 既有 writer/CLI contract不回归 | shared 文件回归 | passed |

## 7. 最小实现切片

| Slice ID | 对应设计契约 | 改动对象 | 输出文件 | 局部验证 | 状态 |
|---|---|---|---|---|---|
| IMPL-AW004-S1 | schema/validator | typed models、canonical digest、published handle reread | aggregate + tests | 33 passed | done |
| IMPL-AW004-S2 | pure aggregate | precedence/progress/effect/deterministic identity | aggregate + tests | 16-combination matrix | done |
| IMPL-AW004-S3 | persistence/readback | immutable file/memory store、external receipt | aggregate + tests | idempotency/tamper/readback | done |
| IMPL-AW004-S4 | single writer | selector key、expected-current CAS、bounded lock | aggregate + tests | same/conflicting concurrency | done |
| IMPL-AW004-S5 | projection/CLI | persisted PASS guard、CR ledger/current writer、CLI | aggregate/shared/tests | controlled writer + CLI dry-run | done |
| IMPL-AW004-S6 | handoff | IMPLEMENTATION、return、evidence | process outputs | Story checkers | done |

## 8. 变更说明

### 8.1 代码变更

| 文件 | 动作 | 说明 |
|---|---|---|
| `meta_flow/workflow/artifact_aggregate.py` | create | 完整 aggregate schema/validation/compute/persist/project/coordinator；canonical `omit_keys` 仅作用于顶层，避免错误排除嵌套 leg evidence digests。 |
| `meta_flow/state/current.py` | modify | 增加 persisted aggregate 复核与 allowlisted completion-candidate projection。 |
| `meta_flow/workflow/cr_lifecycle.py` | modify | 增加 aggregate projector 与显式 `cr aggregate` 命令。 |
| `meta_flow/cli.py` | modify | 增加入口发现和 dry-run 示例。 |

### 8.2 Prompt / Skill 变更

N/A；本 Story 不修改 Agent、Prompt 或 Skill。

### 8.3 模板 / Schema 变更

Aggregate schema 作为 `artifact_aggregate.py` typed dataclass 落地；不复制或修改 ST-AW-003 producer schema。

### 8.4 Guardrail / 测试变更

| 文件 / 命令 | 动作 | 说明 |
|---|---|---|
| `tests/test_artifact_aggregate.py` | create | 覆盖 TP-AW04-001..020 适用 contract、failure 与 concurrency paths。 |
| forbidden import scan | run | aggregate 对 Git/worktree/subprocess/manual-sync import 数=0。 |

### 8.5 文档变更

本文与 CP6 return/evidence 只记录摘要和证据引用；不修改长期 HLD/ADR/Feature DESIGN，因此无需 Design Delta。

## 9. 平台差异处理

| 平台 | 检查项 | 预期 | 结果 |
|---|---|---|---|
| Claude / Codex / OpenClaw | Prompt/agent schema | N/A | N/A；纯 Python core/CLI。 |
| Linux / POSIX fixture | immutable append + selector lock | exclusive append、bounded lock、fail closed | PASS |
| Windows | lock directory / `os.replace` | 使用标准库跨平台原语 | 未在 Windows 实机执行；保留 CP7 平台风险 |
| runtime remote | 真实 Git/remote | 不执行 | N/A；明确禁止。 |

## 10. 验证结果

| 命令 / 检查 | 结果 | 证据 |
|---|---|---|
| `pytest -q tests/test_artifact_leg_lifecycle.py tests/test_artifact_aggregate.py tests/test_git_branch_lifecycle.py tests/test_workspace_git_sync.py tests/test_state_v2.py tests/test_cr_lifecycle.py` | PASS | 179 passed，9 subtests passed；覆盖 producer/consumer canonical 组合回归。 |
| `ruff check`（5 个 touched Python 文件） | PASS | All checks passed |
| `ruff format --check`（新建 aggregate/test 文件） | PASS | 2 files already formatted；shared 旧文件不做全文件机械重排 |
| `python -m py_compile`（5 个 touched Python 文件） | PASS | exit 0 |
| forbidden import scan | PASS | aggregate Git/worktree/subprocess/manual-sync import=0 |
| `git diff --check`（本 Story touched files） | PASS | exit 0 |
| Story Return Packet Check | PASS_WITH_WARNING | `OK`；外置 process symlink 物理路径与 packet 逻辑路径比较产生已知 path warning，逻辑输出路径与 expected ref 一致。 |
| Evidence Index Check | PASS | `Evidence Index Check: OK`。 |

## 11. 未覆盖项

| 未覆盖内容 | 原因 | 后续处理 |
|---|---|---|
| Windows 跨进程 lock crash fixture | 当前验证环境为 Linux；实现使用跨平台标准库但未实机 | CP7 作为平台剩余风险验证或保留 PASS_WITH_RISK |
| 真实托管 remote / 真实 worktree | 明确未获 runtime authorization，且 aggregate 禁止调用 | 不属于本 Story；CP8 保留 `CR051-RISK-REMOTE-UNVERIFIED` |
| CP6/CP7 gate 决策 | meta-dev 不批准自身 CP6；尚未独立 QA | Host 先执行 CP6，再调度 meta-qa CP7 |

## 12. 风险与回滚

| Risk ID | 风险 | 影响 | 缓解 | 回滚 / 切换条件 |
|---|---|---|---|---|
| R-AW004-01 | selector writer crash 留下 lock directory | 后续 writer fail closed | bounded lock、绝不覆盖 current；显式人工诊断后再恢复 | lock 不可安全判定时保持 BLOCKED，不删除 evidence |
| R-AW004-02 | current-state CAS token stale | 投影拒绝 | `expected_updated_at` 强制匹配；同 aggregate retry幂等 | 刷新 current 状态并显式重试 projection |
| R-AW004-03 | projection ledger/current 部分失败 | aggregate PASS 但 projection未完成 | aggregate immutable；retry不重跑 leg；每层外置 receipt | 停止入口，保留事实，修复 writer 后只重试 projection |
| R-AW004-04 | shared CLI/state 回归 | 既有 CR 工作流受影响 | 新命令显式 dispatch；相邻 regression suite | 回退本 Story shared 接线，不删除 aggregate/leg evidence |

## 13. 设计缺口反馈

| Gap ID | 发现阶段 | 问题 | 应反馈到 | 阻塞 | 推荐处理 |
|---|---|---|---|---|---|
| DELTA-AW004-CP8 | CP8 终验 | PARTIAL 枚举无生产路径、payload target policy 未被 consumer 独立验证 | Feature DESIGN / LLD | no（保守补强） | 已合并 `process/design-deltas/ST-AW-004.delta.json`。 |

## 14. QA / Review / Doc 后续交接

### QA 关注点

- 复跑 16 组合和 14 类 invalid handle；验证 invalid coordinate 的 store/project 调用均为0。
- 复跑 same/conflicting concurrency，确认 current selector 单写且没有 last-write-wins。
- 验证 projection shared writer 的 persisted payload、receipt、current selector、STATE.updated_at 四重门。
- 验证 non-PASS、stale、conflict、readback failure 对 current/CR ledger/close/rollback 调用均为0。

### Review 关注点

- aggregate import graph必须继续不包含 producer executor、Git/worktree/manual-sync。
- `AggregateResult` 与 leg payload 均不得吸收自身 receipt/ref/writer/time。
- shared current projection只保存轻量 ref，不新增长状态或第二 truth source。

### Doc 关注点

- 用户手册若记录该命令，必须强调默认不投影、`--project-completion` 是显式二阶段动作、真实 Git/remote 不在本命令能力内。

## 15. CP8 终验回修实施（R2）

| 回修项 | 实现结果 | 验证 |
|---|---|---|
| target policy | aggregate 重读 payload 后独立校验 canonical source/artifact mode 与 base/target/active ref | 错误 target 返回 `TARGET_POLICY_MISMATCH`，persist/project=0 |
| CLI mode | source/artifact mode 参数限定 canonical choices | 不再有 `artifact-integration` 漂移默认值 |
| staged projection | state fact 成功但 ledger/current 失败返回 `PARTIAL` 并保留 receipt | production PARTIAL 路径已覆盖 |
| retry | `retry-controlled-projection` 只补未完成投影 | aggregate/Git leg 不重跑；state fact 幂等 |
| dependency / exact DAG | 增加调用 spy 与精确顺序断言 | Git/worktree/leg/manual-sync=0；DAG 无回边 |

正式 design delta 已合并到 Feature DESIGN 1.2 与 LLD 1.2。定向组合 127 项通过，跨模块集合 307 项与 58 个 subtests 通过，全仓 697 项与 70 个 subtests 通过；Ruff、compile、delivery guardrail、whitespace 审计均通过。真实 remote/worktree/ref mutation=0。

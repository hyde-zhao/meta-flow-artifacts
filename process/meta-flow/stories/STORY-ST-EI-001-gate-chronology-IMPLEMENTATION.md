---
status: "completed"
version: "1.0"
story_id: "ST-EI-001"
story_slug: "gate-chronology"
feature_id: "FEAT-EI-CORE"
implementation_type: "mixed"
source_story: "process/stories/STORY-ST-EI-001-gate-chronology.md"
source_design_evidence: "process/stories/STORY-ST-EI-001-gate-chronology-LLD.md"
created_by: "host-orchestrator"
created_at: "2026-07-12T04:29:56Z"
updated_at: "2026-07-12T04:29:56Z"
---

# Implementation: ST-EI-001 — Gate chronology and conditional approval

## 1. 实现摘要

| 项目 | 内容 |
|---|---|
| 实现目标 | 在既有 state-transition checker 中实现只读的 typed chronology、条件式批准和 phase/gate 不变量。 |
| 行为变化 | checker 现在可拒绝无时区时间、倒序、未开门 review/approval、未满足条件的批准，以及“已批准但没有下游 transition”的状态混淆；`--chronology-events --output json` 提供稳定机器输出。 |
| 范围边界 | 不写入 canonical ledger；不改变 `cp_result.py` 或 `cli.py` shared merge-owner 文件；不实现 dispatch/final-attempt correlation。 |
| CP6 证据 | `process/checks/CP6-CR046-ST-EI-001-gate-chronology-CODING-DONE.result.json` |

## 2. 上游设计引用

| 来源 | 路径 / ID | 本次消费内容 |
|---|---|---|
| Story | `process/stories/STORY-ST-EI-001-gate-chronology.md` | AC、W1 文件所有权和禁止范围。 |
| Story LLD | `process/stories/STORY-ST-EI-001-gate-chronology-LLD.md` | typed node、偏序、条件状态机、CLI fixture 与失败关闭规则。 |
| HLD / ADR | `CR046-EVIDENCE-INTEGRITY-HLD.md` / `CR046-EI-ADR-003` | canonical evidence 只读、条件式批准不可直接提升为 final approval。 |
| Feature 设计 | `docs/features/cr046-core/{DESIGN,TEST-PLAN,TASKS}.md` | CT-CORE-01、TASK-EI-001-01 与只读 typed graph 边界。 |

## 3. 实现前置检查

| 检查项 | 结果 | 证据 / 说明 |
|---|---|---|
| CP5 全量设计确认 | PASS | `GATE-CR046-CP5-APPROVED`。 |
| Story 范围与文件 owner 明确 | PASS | Story card 与 Development Plan W1。 |
| 阻断 clarification | PASS | LLD `open_items=0`。 |
| 验证方式明确 | PASS | `pytest`、CLI JSON fixture、`git diff --check`。 |
| 平台 / 权限边界 | PASS | 无 runtime、credential、publish、quant-lab 业务源码、commit 或 push。 |
| 子 Agent 路由 | WAIVED | 用户明确要求不拉起子 Agent；`IF-CR046-ST-EI-001-CP6` 记录为 inline fallback，不能证明 custom agent/model。 |

## 4. 实现对象清单

| 对象 | 类型 | 目标 | 是否必须 | 验证方式 |
|---|---|---|---|---|
| `meta_flow/checks/state_transition.py` | code / guardrail | typed chronology、条件状态、phase/gate 分离、自动阶段路由、JSON CLI 输出 | yes | unit + CLI fixture |
| `tests/test_state_transition.py` | unit / contract fixture | 正负 chronology、conditional、phase/gate、CP5 自动执行、CLI JSON 回归 | yes | pytest |
| `process/stories/*IMPLEMENTATION.md` | docs-handoff | 记录设计映射、边界和 QA 注意项 | yes | CP6 return/evidence check |

## 5. 设计契约映射

| 设计要求 | 来源 | 实现位置 | 实现动作 | 验证 |
|---|---|---|---|---|
| aware RFC3339 偏序 | LLD §2/§5 | `ChronologyNode`, `_parse_chronology_time`, `validate_chronology` | 解析 aware datetime，按显式边比较并稳定排序 finding | timezone 与倒序 fixtures |
| 条件式批准不能直接 final | LLD §2/§7 | `derive_gate_decision` | 任一 chronology finding 或缺 `conditions-satisfied` 均返回 `pending` | conditional 正负 fixtures |
| phase work 与 future gate 分离 | LLD §2/§7 | `validate_phase_gate_state`, `_is_automatic_phase_in_progress` | 已批准 CP5 的真实 CP6/CP7 工作不伪造 CP8；无 gate-open 的 formal review/approval 被拒绝 | phase and CP5 transition fixtures |
| 稳定机器输出 | LLD §6/§10 | `state_transition.main` | 新 `--chronology-events` 和 `--output json` 输出 code/object/field/source refs | CLI JSON fixture |

## 6. 单元测试 / Fixture 计划与结果

| 测试对象 | 测试类型 | 输入 / Fixture | 期望 | 状态 |
|---|---|---|---|---|
| chronology precedence | unit | 完整顺序、每条核心边倒序 | 合法为空；倒序为 `TEMPORAL_ORDER_VIOLATION` | passed |
| conditional approval | unit | conditional→conditions→approved→dispatch；缺 conditions | 前者 approved；后者 pending + `CONDITIONS_UNSATISFIED` | passed |
| phase/gate | integration fixture | phase work、review without gate、approved without transition | 仅 phase work 合法；其余 `PHASE_GATE_CONFLATION` | passed |
| automatic route | regression | CP5 approved 后 `story-execution` | 不要求伪造 CP8；仍保留 CP3 立即推进 CP5 的既有约束 | passed |
| CLI | contract fixture | chronology JSON | stable JSON 与非零 exit | passed |

## 7. 最小实现切片

| Slice ID | 对应设计契约 | 改动对象 | 局部验证 | 状态 |
|---|---|---|---|---|
| IMPL-EI-001-A | typed chronology / conditional state | `state_transition.py` | `pytest tests/test_state_transition.py -q` | done |
| IMPL-EI-001-B | phase/gate and automatic execution | `state_transition.py`, tests | CP5 transition check + regression tests | done |
| IMPL-EI-001-C | stable CLI output | `state_transition.py`, tests | CLI JSON fixture | done |

## 8. 变更说明

| 文件 | 动作 | 说明 |
|---|---|---|
| `meta_flow/checks/state_transition.py` | modify | 增加 typed chronology API、fail-closed conditional decision、phase/gate check、CP5 automatic-phase allowance 和 JSON output。 |
| `tests/test_state_transition.py` | modify | 新增 CT-CORE-01、条件式、phase/gate、CP5 和 CLI tests；当前 27 passed / 42 subtests。 |

`cp_result.py` 与 `meta_flow/cli.py` 的 canonical-result 接线仍由 ST-EI-003 作为 merge owner 执行；本 Story 没有越过 shared owner 边界。

## 9. 平台差异处理

| 平台 | 检查项 | 结果 |
|---|---|---|
| Codex | 不把 task/profile ledger 声明写成 runtime profile receipt | PASS；本实现不读取或生成 runtime receipt。 |
| Codex | 自定义 Agent runtime proof | UNAVAILABLE；见 `CR046-CUSTOM-AGENT-CAPABILITY-PROBE.json`。 |
| Claude / 安装器 | 本 Story不涉及平台安装或 Agent frontmatter | N/A。 |

## 10. 验证结果

| 命令 / 检查 | 结果 | 证据 |
|---|---|---|
| `uv run pytest tests/test_state_transition.py -q` | PASS | 27 passed, 42 subtests passed。 |
| `uv run pytest tests/test_cp_result_event_ledger.py -q` | PASS | 36 passed, 2 subtests passed。 |
| `uv run meta-flow check state-transition --route-plan process/checks/CP0-CR046.route-plan.json --approved-gate CP5 --project-root .` | PASS | 允许真实 `story-execution` 自动阶段而不伪造 CP8。 |
| `git diff --check` | PASS | 无 whitespace error。 |

## 11. 未覆盖项

| 未覆盖内容 | 原因 | 后续处理 |
|---|---|---|
| canonical CP result/gate/state refs 自动喂给 chronology checker | shared `cp_result.py` / `cli.py` 的 merge owner 是 ST-EI-003 | ST-EI-003 接线后运行跨真相源 integration fixture。 |
| 平台 runtime custom-profile attestation | 当前工具没有 D0 selector/receipt | A-baseline；CP7/CP8 ceiling 由 ST-EI-002/006 执行。 |

## 12. 风险与回滚

| Risk ID | 风险 | 缓解 | 回滚 / 切换条件 |
|---|---|---|---|
| R-EI-001-01 | legacy callers未接入新的 chronology fixture path | 保持既有 route CLI 兼容；ST-EI-003 统一接线 | 合法 replay 回归则移除接线，不修改 canonical evidence。 |
| R-EI-001-02 | 依赖发现的 CP5 route checker 原先强迫未来 CP8 | 用 `story-execution` 专门判定，保留其他 gate 的即时推进规则 | 若产生绕过 gate 的 fixture，回退该 allowance 并重开 LLD。 |

## 13. 设计缺口反馈

| Gap ID | 问题 | 处理 |
|---|---|---|
| GAP-EI-001-01 | Story packet builder 默认把 full LLD 放入 deny-default read 集合，导致 generated packet 自检冲突。 | 作为 ST-EI-004 read-authorization / context governance 修复输入；本 Story packet 保持 capsule-first，已对实际深读写入 read-expansion ledger。 |

## 14. QA / Review / Doc 后续交接

### QA 关注点

- 验证所有 precedence edge 的矩阵，而不是仅 producer/checkpoint 一条边。
- 验证 JSON finding 的 source refs 和排序在重复运行中稳定。
- 核对 CP5 in-progress allowance 不会允许 CP7 完成后绕过 CP8。

### Review 关注点

- `INVALID_SOURCE_REF` 与 `UNKNOWN_CHRONOLOGY_KIND` 保持 fail-closed。
- ST-EI-003 接线前不应声称 CP result 已做 chronology cross-truth enforcement。

### Doc 关注点

- 用户文档无需在 W1 单独更新；最终 audit/replay CLI 文档由 ST-EI-006 汇总。

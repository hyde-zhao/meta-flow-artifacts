---
status: draft-for-cp5
version: "1.0"
feature_id: "FEAT-21"
feature_name: "Trial-lineage producer adapters"
source_hld: "docs/design/HLD-TRIAL-LINEAGE-INSTRUMENTATION.md"
source_matrix: "docs/design/FEATURE-DESIGN-MATRIX.md#cr163-cp4-增量trial-lineage-instrumentation"
related_stories: ["CR163-S03-two-producer-chain-instrumentation", "CR163-S05-integrity-recovery-permission-regression"]
lld_policy_summary: "S03/S05 full-lld"
---

# Feature Design: Trial-lineage Producer Adapters

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-11 | meta-se-critical | 冻结 2 chains / 4 mappings、session owner、identity 传递、失败和文件 ownership。 |

## 边界与目标

S03 在一个 Story 内覆盖两条去重 producer chains：public Stage3 chain 的 wrapper/orchestration/hook（CPI-001/003）与 legacy CR039 chain（CPI-002/004）。Adapter 不拥有 lineage schema/storage，不按函数调用计 trial，不引入第三条 chain。

## 现有位置与规划变更

| Mapping | Path/symbol | Owner 角色 | 规划 |
|---|---|---|---|
| CPI-001 | `scripts/research/run_multifactor_strategy_research.py::main` → `run_stage3_mature_multifactor_research` | wrapper透传；orchestration session owner | 增加显式 lineage config/session context；首 trial 前 open |
| CPI-003 | `engine/mature_multifactor_research.py::build_strategy_candidate` | 同 chain hook | 接收 stable trial/candidate context 或由 orchestration 在返回后记录 selection；只能一种，不双写 |
| CPI-002 | `scripts/legacy/research/run_multifactor_strategy_candidates.py` → `run_strategy_research` | wrapper透传；orchestration session owner | 同上 |
| CPI-004 | `engine/multifactor_strategy_candidates.py::build_strategy_candidates` | 同 chain hook | stable params+seed→trial；candidate/selection refs；不得按列表长度/调用次数替代 declared identity |

## Integration contract

| 时机 | Caller → Callee | 输入 | 输出 | 失败行为 |
|---|---|---|---|---|
| before search | orchestration → Session.open | chain id、FamilySpec、run/experiment refs | declaration receipt | missing/post-hoc blocked；不 fallback manual count |
| before each trial | orchestration/hook → session | normalized params+seed | stable trial receipt | conflict blocked |
| execution/retry | orchestration → session | attempt ordinal/run refs/status | attempt receipt | orphan/illegal state blocked |
| candidate decision | hook or orchestration（single writer）→ session | trial/candidate ref + decision/reason | selection receipt | missing trial blocked |
| family end | orchestration → session.seal | completion + artifact refs | manifest/validation | incomplete remains blocked/unavailable |

## File ownership and merge

S03 独占规划修改两个 engine 与两个 wrappers；core files只读依赖。四文件由同一个 Story owner 合并，避免两 agent 对同一 Story 分裂 session identity。S03 不修改 consumer modules。

## 失败与回退

- Adapter disabled or absent：future path `typed_unavailable`，不得 hand-fill present。
- Hook invoked without session：fail closed or orchestration records after return；LLD 必须选定明确策略。
- wrapper/hook duplicate delivery：event idempotency，raw count不变。
- producer exception：append failed/cancelled attempt/trial terminal，保留 lineage；seal completeness按 contract。
- rollback：移除/disable adapter projection，历史 append-only artifacts不删；consumer回 typed_unavailable。

## Story / downstream

S03 full-lld 冻结四 symbol signatures、session owner、trial parameter extraction、selection writer、exception/finally seal policy和4/4 fixture mapping。S05 full-lld消费本 TEST-PLAN 验证 entry coverage 与 no-double-count。

## Gotchas

- wrapper mapping 与 hook mapping 不等于四条 chain，也不等于四 trials。
- hook 返回多个 candidates 不自动等于多个 trials；trial membership 必须在 search 前声明。
- 两条 chain 可共享 core contract，但不能共享同一个 family_id，除非 CP5 LLD 明确它们确属同一 search family；默认按 producer chain/family spec 分离。
- wrapper 不得自行新建第二个 session。


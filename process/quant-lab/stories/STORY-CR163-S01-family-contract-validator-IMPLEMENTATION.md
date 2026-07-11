---
story_id: "CR163-S01-family-contract-validator"
stage: "CP6"
status: "implemented"
implemented_by: "meta-dev-debugger"
implemented_at: "2026-07-11T12:24:00+08:00"
updated_at: "2026-07-11T12:54:25+08:00"
rework_round: 2
design_evidence_ref: "process/stories/STORY-CR163-S01-family-contract-validator-LLD.md"
work_packet_ref: "process/context/stories/STORY-CR163-S01.CP6.work-packet.json"
---

# CR163-S01 Implementation — Experiment-family contract and validator

## 实现前置检查

| 检查 | 结果 | 证据 |
|---|---|---|
| Story / CP5 设计证据 | PASS | Story 为 `dev-ready`，`design_evidence_confirmed=true`，LLD 为 confirmed / approved |
| 依赖与文件所有权 | PASS | S01 无上游依赖；write scope 仅限本 Story 的 core、tests 与三项 CP6 证据 |
| 授权边界 | PASS | 仅 repository-local source、fixture/static tests；无真实 runtime/data/credential/external/statistical/backfill/release 操作 |
| 设计澄清 | PASS | open item = 0；实现未改变 Feature / HLD / ADR contract |

## 实现对象与设计契约映射

| 对象 | 实现 | 契约 / 验证 |
|---|---|---|
| 公共 core contract | `engine/experiment_family_lineage.py` | 六个 frozen/slotted persistent DTO 恰为 6/6；严格 canonical JSON 与 stable trial id |
| 状态与错误 contract | 同上 | family/trial/attempt 全表驱动合法边；unknown/illegal fail closed；冻结全部 blocked codes |
| command / façade | 同上 | 九个 typed command；公共 envelope；`FamilyLineageSession` open/submit/seal 即时转交 recorder |
| fold / validator | 同上 | parent、identity、terminal、raw count、event boundary、target binding、forbidden counter 纯验证 |
| immutable projection | 同上 | present 仅限 target-bound PASS；unavailable / N/A / blocked 不提升；effective 四字段固定 unavailable/null/empty/empty |
| contract tests | `tests/test_experiment_family_lineage_contracts.py` | 165 个静态/fixture tests（含参数矩阵）覆盖 inventory、canonical、合法/非法 transitions、commands/session、retry/count、target/error/projection/import boundary 与两轮 CP7 rework 回归 |

## 最小实现切片与结果

1. DTO / canonical / identity：完成 6/6 inventory、递归冻结与 deterministic serialization；stable id 域分离 SHA-256。
2. Commands / session / transitions：完成 9/9 typed commands、recorder protocol、receipt、immediate-submit session 和全表状态转换。
3. Fold / validation / projection：完成 deterministic fold、distinct declared stable trial count、orphan/terminal/target/effective/forbidden fail-closed checks。
4. Fixture / guardrail：完成 165 个 contract tests 与标准库-only import boundary 检查。

## CP7 NEEDS_REWORK 修复（Round 1）

| Finding | 修复 | 稳定 blocked semantics | 回归证据 |
|---|---|---|---|
| QA-CR163-S01-001 | validator 以 `sealed_last_sequence` 选择 immutable prefix，再用 `sealed_event_count`、内部 sequence coverage 与 fold boundary 双向核对；后续 correction/v2 suffix 不再污染 v1 | malformed / ambiguous / gapped boundary → `target_mismatch` | v1 before/after append、四类 malformed boundary、内部 gap |
| QA-CR163-S01-002 | sealed / superseded 状态下，在任何 membership/lifecycle mutation 前先 gate，拒绝后不修改 family/trial/attempt maps | late `DeclareTrial` → `post_hoc_declaration`；其他 late mutation → `illegal_family_transition` | sealed/superseded × 7 command kinds 参数矩阵 |
| QA-CR163-S01-003 | projection 改为正向证明 coherent PASS：present + pass + no reason + no unavailable reason + manifest + nonempty matching target + count coherence；任一不满足均 blocked | target inconsistency → `target_mismatch`；其他不一致 → existing reason 或 `required_field_missing` | availability × status × reason × target × manifest 全矩阵 |
| QA-CR163-S01-004 | fold 只接受九个 exact approved command types；public base / unknown subclass 在进入 event boundary 和状态变更前拒绝 | `schema_version_unsupported`（复用已确认 fixed enum，无 public contract change） | base + unknown subclass fold/validator fixtures |
| QA-CR163-S01-005 | 保留原 28 测试并增加上述负向矩阵 | N/A | full suite 146 passed；rework subset 118 passed |

修复没有新增 persistent object、typed command 或 blocked code，也没有改变 raw-count、effective ceiling、storage ownership或公共设计；design clarification / design delta 均不需要。

## CP7 NEEDS_REWORK 修复（Round 2）

| Finding | 修复 | 稳定 blocked semantics | 回归证据 |
|---|---|---|---|
| QA-CR163-S01-006 | `DeclareFamily` 在 family state transition 前对 nested spec 与 authoritative spec 做完整 canonical content equality；同 family ID 的 schema/producer/objective/space/run/experiment/metadata 任一漂移都不推进状态 | nested family ID mismatch → `family_identity_mismatch`；same-ID content drift → `event_identity_conflict` | 7-field content-conflict matrix；fold state/count 与 validator BLOCKED |
| QA-CR163-S01-007 | `RecordSelection` 在 parent lookup 前要求 `selection.family_id == command.family_id == spec.family_id`，并要求 declared local trial parent；拒绝后不改变 state/count | foreign family → `family_identity_mismatch`；missing local parent → `orphan_selection` | foreign-family / orphan-parent direct probes |
| QA-CR163-S01-008 | 对九个 exact typed commands 建立 nested identifier fail-closed matrix；同时覆盖 StartAttempt family、attempt/selection duplicate identity、initial seal prior、correction event ref、superseding prior completeness | 全部复用 frozen code：`event_identity_conflict`、`family_identity_mismatch`、`orphan_*`、`supersession_version_invalid`、`illegal_family_transition` | 9/9 matrix + prior QA-001..004 118-case selector + adjacent 38 tests |

Round 2 同样没有新增或改变 public DTO、command、enum、blocked code 或长期设计；无需 design clarification / design delta。

## 行为与边界

- Failed、cancelled、excluded、never-started trial 均保留在 declared membership；attempt retry 与 selection 不增加或减少 raw trial count。
- Manifest 显式冻结 `sealed_event_count` 与 `sealed_last_sequence`；validator 同时比对事件边界、trial set 和 raw count。
- Core 不访问 opaque `lineage_root`，不 import storage、producer、consumer、runtime、data、credential 或 external module。
- Effective trial count 未计算；所有 validation/projection 均强制 `typed_unavailable`、`None`、空 ref、空 method，C1 仅为 `raw_input_ready` 或 `input_blocked`。
- 未修改长期设计，因此 design delta 不需要。

## 验证结果

| 命令 | 结果 |
|---|---|
| `uv run --python 3.11 pytest -q tests/test_experiment_family_lineage_contracts.py` | PASS — 165 passed in 0.12s |
| Round 2 complete identity selector | PASS — 19 passed, 146 deselected in 0.03s |
| Same-family complete spec conflict direct probe | PASS — 7 passed, 158 deselected in 0.02s |
| Foreign-family/orphan selection direct probe | PASS — 2 passed, 163 deselected in 0.03s |
| `uv run --python 3.11 pytest -q tests/test_experiment_family_lineage_contracts.py -k 'v1_validation_replays or malformed_or_ambiguous or internal_sequence_gap or post_seal_and_post_supersession or projection_exhaustively or unknown_or_base'` | PASS — 118 passed, 47 deselected in 0.07s |
| Adjacent manifest/lineage/admission subset | PASS — 38 passed in 0.64s |
| QA-001 独立复现组：`-k 'v1_validation_replays or malformed_or_ambiguous or internal_sequence_gap'` | PASS — 6 passed |
| QA-002 独立复现组：`-k 'post_seal_and_post_supersession'` | PASS — 14 passed |
| QA-003 独立复现组：`-k 'projection_exhaustively'` | PASS — 96 passed |
| QA-004 独立复现组：`-k 'unknown_or_base'` | PASS — 2 passed |
| `uv run --python 3.11 python -m py_compile engine/experiment_family_lineage.py` | PASS |
| `git diff --check` | PASS — 无 whitespace error |
| `uv run meta-flow story return-check --packet ... --return ... --project-root .` | PASS — Story Return Packet Check: OK |
| `uv run meta-flow story evidence-check --index ... --project-root .` | PASS — Evidence Index Check: OK |

## 未运行项与剩余风险

- 未运行真实 runtime/data/credential/provider/NAS/broker/trading/external registry、统计修正、历史回填或发布操作；这些均为明确禁止项，不是验证缺口。
- S01 验证 recorder 的协议与幂等 receipt signal；持久化、canonical seal/hash、supersession resolver 由依赖本 contract 的 S02 独立实现和验证。
- QA 关注：九命令 payload 与 S02 recorder 的一向依赖、sealed event boundary、target hash binding、unknown transition fail-closed、effective ceiling。

## 实现交接

- 建议下一路由：重新执行 S01 CP6 自动检查，随后由独立 `meta-qa-critical` 重跑 CP7 static-only 验证并逐项关闭 QA-CR163-S01-001..005。
- S02-S05 应 import 本模块公开 contract，不复制 canonical/stable-id/blocked-code 算法，也不得形成 core → downstream 反向依赖。

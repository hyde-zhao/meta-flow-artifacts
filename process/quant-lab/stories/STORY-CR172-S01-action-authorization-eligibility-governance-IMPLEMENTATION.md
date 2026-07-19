---
status: complete
version: "1.1"
story_id: "CR172-S01-action-authorization-eligibility-governance"
story_slug: "action-authorization-eligibility-governance"
feature_id: "FEAT-CR172-I03"
implementation_type: "mixed"
source_story: "process/stories/STORY-CR172-S01-action-authorization-eligibility-governance.md"
source_design_evidence: "process/stories/STORY-CR172-S01-action-authorization-eligibility-governance-LLD.md"
created_by: "meta-dev"
created_at: "2026-07-18T14:17:28+08:00"
updated_at: "2026-07-18T15:15:00+08:00"
---

# Implementation: CR172-S01 PATH-I authorization and claim governance

## 1. 实现摘要

| 项目 | 内容 |
|---|---|
| 实现目标 | 创建 repository-local、无 I/O 的 PATH-I 六动作授权、执行资格 DAG、empirical-R disposition、run-path、SignalBatch 与 claim ceiling 合同。 |
| 行为变化 | 新增纯标准库 `engine.path_i_governance` API；repository fixture 可执行确定性合同判定，current-v1 `approved_ledger` 对六动作固定 `authorized=false`、`eligible_to_execute=false`。 |
| 范围边界 | 不连接 approved-ledger backend，不执行真实 data lake/runtime/NAS/R/signal/trading 操作，不修改 runner、lineage、CR173 estimator、public C1 或 admission。 |
| CP6 证据 | `process/checks/CP6-CR172-S01-IMPLEMENTATION-DONE.result.json`；`process/evidence/CR172-S01-action-authorization-eligibility-governance.CP6.index.json` |

## 2. 上游设计引用

| 来源 | 路径 / ID | 本次消费内容 |
|---|---|---|
| Story | `process/stories/STORY-CR172-S01-action-authorization-eligibility-governance.md` | Story AC、文件所有权、dev/validation gate、禁止范围。 |
| Story 设计证据 | `process/stories/STORY-CR172-S01-action-authorization-eligibility-governance-LLD.md` v1.3 | 12-field record、approved-ledger hard deny、5-edge DAG、接口、失败优先级、测试、回滚。 |
| HLD / ADR | `docs/design/HLD-TRIAL-RETURN-DEPLOYMENT-CONTRACTS.md` §6/9.2/10.4～10.5/11.1/11.5～11.7；ADR-005～009 | 六动作独立授权、typed origin/target binding、四态、REQ-013 deferred、SignalBatch 与 claim ceiling。 |
| Feature 设计 | `docs/features/path-i-authorization-claim-governance/DESIGN.md` / `TEST-PLAN.md` / `TASKS.md` | I03 合同、负向用例、CR172-S01-T02～T04。 |
| Work packet | `process/context/stories/STORY-CR172-S01.CP6.work-packet.json` | 实现目标、授权边界、读写白名单、三条强制验证命令。 |

## 3. 实现前置检查

| 检查项 | 结果 | 证据 / 说明 |
|---|---|---|
| process route health | PASS | `process/current/CURRENT.json.health=ok`，routing ref 与 STATE 一致；未创建或修复 process。 |
| 上游 Feature 设计存在 | PASS | DESIGN/TEST-PLAN/TASKS 均存在，Story `feature_design_refs=3/3`。 |
| Story 范围明确 | PASS | work packet 与 Story 共同限定两个 production/test 文件和六项非目标。 |
| 待确认问题已关闭 | PASS | LLD `open_items=0`，实现灰区/OPEN/Spike=`0/0/0`。 |
| 影响范围可定位 | PASS | primary owner=`2/2`，shared=`0`；forbidden production paths 未触碰。 |
| 验证方式明确 | PASS | work packet 提供 pytest、py_compile、git diff check 三条命令。 |
| 当前 Wave / dev_gate 满足 | PASS | W1；`design_evidence_confirmed=true`、`lld_confirmed=true`、`dependencies_satisfied=true`。 |
| 文件所有权无冲突 | PASS | 实现前目标 8 路径均无已有工作树变更；只写 allowed paths。 |

阻塞项=`0`；无需 `NEEDS_DESIGN_CLARIFICATION` 或 `NEEDS_REWORK` 路由。

## 4. 实现对象清单

| 对象 | 类型 | 目标 | 必须 | 验证方式 |
|---|---|---|---|---|
| `engine/path_i_governance.py` | contract-code / authorization-validator / claim-guard | 提供 immutable schema、单动作 evaluator、consumer guard 与 boundary validators。 | yes | pytest、py_compile、zero-operation inventory。 |
| `tests/research/test_cr172_path_i_governance.py` | guardrail-test / deterministic fixture | 覆盖 6 actions、5 edges、hard deny、revoke、四态、path、signal、claim 与静态边界。 | yes | 定向 pytest。 |
| 本 IMPLEMENTATION | docs-handoff | 留存设计契约映射、测试、范围、风险与交接。 | yes | 人工结构检查。 |
| CP6 Return / Evidence / Result / Summary | structured evidence | 为 CP6/CP7 提供 touched files、命令、边界和阶段路由。 | yes | `return-check`、`evidence-index`、`cp result-check`。 |

Prompt / Skill、安装器、平台 schema、真实 runtime fixture、design delta 均为 N/A；本 Story 未修改这些对象。

## 5. 设计契约映射

| 设计要求 | 来源 | 实现位置 | 实现动作 | 验证 |
|---|---|---|---|---|
| action/record/enforcement point=`6/6/6`，record fields=`12/12` | LLD F-01/F-11、HLD §10.4 | action enum、record dataclass、`ACTION_ENFORCEMENT_POINTS` | 创建 exact typed inventory 与 import-time 自检。 | `test_action_kind_and_dag_are_exact`、`test_approval_record_remains_exactly_twelve_fields`。 |
| current-v1 approved-ledger 固定双 false，reason 唯一稳定 | LLD F-02、ADR-005 | `evaluate_action_decision` 的最高优先级分支 | 在读取 record allow/path/predecessor 前 fail closed。 | 6-action loop，authorized/eligible=`0/0`。 |
| origin/target=`2/2`；fixture+real side effect=`0` | LLD F-06/F-12 | `ActionDecisionOriginV1`、`ActionTargetKindV1`、fixture URI binding、consumer guard | 保留 single typed truth，拒绝 cross-pair 与非 repository fixture URI。 | enum inventory、real-target/URI/port negative cases。 |
| DAG nodes/edges=`6/5`，own authorization 与 eligibility 分离 | LLD F-03～F-05 | immutable `DIRECT_PREREQUISITE`、`_evaluate_predecessor` | 只验证唯一直接前置，不做 transitive permission union。 | 5-edge parameterization、runtime-without-read 三类 negative。 |
| time/scope/path/provenance fail closed、deny-first | LLD NFR-02～05 | own-record evaluator、URI/prefix/provenance validators | 固定优先级 reason，aware UTC，deny 在 allow 之前。 | time/scope/path/revoke/context cases；3-run deterministic equality。 |
| empirical disposition exactly `4/4` 且 pre-v2 positive/C1=`0/0` | LLD F-07、ADR-006 | `EmpiricalRInputsV1` / `classify_empirical_r` | conflict→BLOCKED；fixture→declared；前置不足→unavailable；完整→empirical；全部 claim false。 | 四态集合与两个 ceiling 断言。 |
| REQ-013 只 contract ready，runtime enforcement/default switch/delivered=`0` | LLD F-08/F-13、ADR-007 | `RunPathIntentV1` / `RunPathDecisionV1` | new 仅 exact contract，legacy 仅 read；`writable=false`，delivery status 固定 deferred。 | new/legacy 与 5 类 legacy mutation negative。 |
| SignalBatch semantic slots=`8/8`，无 exchange/wire/state/ack/replay | LLD F-09、ADR-008 | `SignalBatchBoundaryV1` 两个复合槽 + validator | 物理字段精确表达八个语义槽，拒绝缺失、secret-like key 和畸形 validity。 | dataclass field count、valid/missing/credential cases。 |
| 五项高阶 claim 全 false | LLD F-10、ADR-009 | `PathIClaimCeilingV1` / `enforce_path_i_claim_ceiling` | 构造时和 guard 时双重 fail closed。 | 5 个 mutation fixture accepted=`0/5`。 |
| production/test 真实 I/O/network/env/credential/runtime 操作=`0` | LLD NFR-01、work packet authz policies | 模块只导入标准库纯值/解析能力；测试仅使用内存 fixture | 未引入 filesystem/network/env/subprocess/provider/storage/runtime adapter。 | import/public surface inventory、定向测试。 |

## 6. 单元测试 / Fixture 计划与结果

| 测试对象 | 类型 | 输入 / Fixture | 期望 | 覆盖风险 | 结果 |
|---|---|---|---|---|---|
| action/record/DAG | unit / contract | 6 actions、12 fields、5 direct predecessors | exact inventory，无 union | action/schema drift | PASS |
| origin/target/approved-ledger | negative fixture | six valid-looking real records、fixture-real cross pair | approved-ledger `0/0`；cross pair side effect=0 | forged provenance | PASS |
| own record / predecessor | parameterized unit | missing/deny/ineligible/context mismatch/time/scope/path/revoke | authorized 与 eligible 分离，reason 稳定 | runtime bypass | PASS |
| consumer guard | unit | wrong kind/context/origin、mid-operation revoke | first side effect 前抛 typed error | pointer advance | PASS |
| empirical/path/signal/claim | contract / mutation fixture | 4 dispositions、2 path modes、8 slots、5 high claims | ceiling 精确，deferred 边界不膨胀 | overclaim / scope escape | PASS |
| zero-operation surface | structure check | imported module/public symbols | backend/network/env/runtime adapter symbols=0 | 权限逃逸 | PASS |

CP6 初始 pytest 通过 `28/28`；CP6R1 在保留原用例的基础上通过 `46/46`。CP6R1 失败、跳过、xfail 均为 `0`。

## 7. 最小实现切片

| Slice ID | 对应设计契约 | 改动对象 | 输出文件 | 局部验证 | 状态 |
|---|---|---|---|---|---|
| IMPL-S1 | F-01～F-06、F-11～F-12 | record + decision + URI binding | `engine/path_i_governance.py` | module py_compile | done |
| IMPL-S2 | F-03～F-06 | immutable DAG + prerequisite + consumer guard | production/test | 6-action/5-edge/runtime-without-read tests | done |
| IMPL-S3 | F-07～F-10/F-13 | empirical/path/signal/claim facets | production/test | boundary/claim tests | done |
| IMPL-S4 | NFR-01～06 | deterministic fixtures + static boundary | test | targeted pytest + diff check | done |
| IMPL-S5 | CP6 evidence contract | implementation/return/index/result/summary | `process/*` allowed outputs | return/result checks | done |

回滚点按 S1→S5 分层；本 Story 新增模块尚无既有 consumer，因此可整体删除新增 production/test 合同回滚。

## 8. 变更说明

### 8.1 代码变更

| 文件 | 动作 | 说明 |
|---|---|---|
| `engine/path_i_governance.py` | create | 新增 frozen/slots 值对象、枚举、immutable DAG、authorization evaluator、consumer guard 和四类 boundary validator。 |

### 8.2 Prompt / Skill 变更

N/A：修改数=`0`。

### 8.3 模板 / Schema 变更

N/A：未修改共享模板或 schema；新增对象完全实现已确认 v1 合同。

### 8.4 Guardrail / 测试变更

| 文件 / 命令 | 动作 | 说明 |
|---|---|---|
| `tests/research/test_cr172_path_i_governance.py` | create / modify | 原 28 个定向测试保持通过；CP6R1 新增 18 个 finding regression pytest node。 |
| packet targeted pytest | run | CP6R1 `46 passed in 0.08s`。 |
| packet py_compile | run | production/test 两文件 exit=`0`。 |
| packet diff check | run | exit=`0`。 |

### 8.5 文档变更

| 文件 | 动作 | 说明 |
|---|---|---|
| 本 IMPLEMENTATION | create | 固化实现对象、设计映射、验证和 CP7 交接。 |
| Story 卡片 | modify | 回填 CP6 定向验证与 implementation gate 完成状态，不改变需求/LLD/文件 ownership。 |
| Return / Evidence / CP6 Result / Summary | create | 固化结构化 touched files、边界、验证和 next route。 |

## 9. 平台差异处理

本 Story 是 pure Python repository-local contract，不包含 Claude/Codex/OpenClaw agent schema、direct ask、安装路径或 dry-run；平台分支全部 N/A。代码不写 Claude-only `tools` schema，也不调用 `request_user_input`。

## 10. 验证结果

| 命令 / 检查 | 结果 | 证据 |
|---|---|---|
| `PYTHONDONTWRITEBYTECODE=1 PYTEST_ADDOPTS='-p no:cacheprovider' uv run --python 3.11 pytest -q tests/research/test_cr172_path_i_governance.py` | PASS | CP6R1 `46 passed in 0.08s`；原 28 tests 保持通过。 |
| `PYTHONPYCACHEPREFIX=/tmp/cr172-s01-pycompile uv run --python 3.11 python -m py_compile engine/path_i_governance.py tests/research/test_cr172_path_i_governance.py` | PASS | exit=`0`；cache 仅写 `/tmp`。 |
| `git diff --check -- engine/path_i_governance.py tests/research/test_cr172_path_i_governance.py` | PASS | exit=`0`。 |
| `meta-flow story return-check ...` | PASS | Return Packet 与 work packet 一致。 |
| `meta-flow cp result-check --check-consistency ...` | PASS | CP6 machine result 合法。 |
| repository-wide test / guardrail | N/A | work packet 只授权 S01 定向验证；全量独立验证由 CP7/质量 lane 执行。 |

## 11. 未覆盖项

| 未覆盖内容 | 原因 | 后续处理 |
|---|---|---|
| 可信 approved-ledger adapter / issuer envelope | current-v1 明确不存在；本 Story禁止连接真实 backend。 | 独立 runtime-high-risk CR 重开 CP3/CP5 并升级 versioned contract。 |
| 真实 multi-trial producer、runner path enforcement 与 default switch | REQ-013 当前只 contract ready。 | future native-producer CR 在 first side effect 前消费 `RunPathDecisionV1`。 |
| empirical R 计算、positive effective count、C1 | FU-CR173-001、真实 source/auth/provenance 未提供。 | 保持 false；按未来 v2/降级决策处理。 |
| Signal wire/mailbox/state/ack/replay/transport | ADR-008 明确 deferred。 | 路由 deferred signal CR，不扩张 S01。 |
| S02～S05 集成与跨 Story 质量验证 | 强制串行；S01 CP7 PASS 前不得提前触碰。 | Host Orchestrator 在独立 CP7 PASS 后调度后续 Story。 |

## 12. 风险与回滚

| Risk ID | 风险 | 影响 | 缓解 | 回滚 / 切换条件 |
|---|---|---|---|---|
| R-CR172-RUNTIME-AUTHORIZATION-GAP | 当前没有真实授权 adapter。 | 六动作真实执行保持不可用。 | approved-ledger 最高优先级 hard deny；fixture typed binding。 | 任何 real action authorized/eligible=true 立即整体回滚并 BLOCKED。 |
| R-CR172-PATH-IDENTITY-DRIFT | contract-only logical root 尚未接入 runner。 | REQ-013 runtime delivered 仍为 0。 | delivery status 固定 deferred，`writable=false`。 | 路径合同变化回设计澄清，不放宽 legacy。 |
| R-CR172-REAL-R-DOMAIN-MISMATCH | empirical 前置未闭环。 | positive effective count/C1 不可声明。 | disposition 可表达 empirical-ready，但本 v1 强制两个 claim false。 | 出现 positive/C1 true 立即回滚。 |
| R-CR172-SIGNAL-TRANSFER-AMBIGUITY | 8-slot facet 可能被误当 transport schema。 | scope 膨胀至信号交换/交易。 | 两个复合语义槽且无 wire/path/state/ack/replay API。 | 新增 slot/transport symbol 回 CP3/CP5。 |

回滚不涉及数据、授权、迁移、部署或远端操作；删除 S01 新增 production/test 合同及其本 Story CP6 证据即可。若 S02 已消费，按 S02→S01 逆依赖顺序回退。

## 13. 设计缺口反馈

设计缺口、clarification、OPEN、Spike=`0/0/0/0`。实现严格落在 LLD v1.3 与 Feature v1.2 合同内，未改变长期 HLD/ADR/Feature 设计，因此 `design_delta_required=false`。

## 14. QA / Review / Doc 后续交接

### QA 关注点

- 独立复核 `authorized` 与 `eligible_to_execute`，尤其 runtime own-record valid/read missing 的 `true/false` 组合。
- 用变异 fixture 验证 approved-ledger hard deny 不能被 caller 自报 enum、valid-looking record、real URI 或 predecessor 绕过。
- 复核 6 actions / 6 records / 6 enforcement points / 5 edges / 12 fields / 8 slots / 5 false claims 的精确计数。
- 复核 source/import surface 无 I/O、network、env、credential、runtime、R、signal exchange、trading side effect。

### Review 关注点

- `DIRECT_PREREQUISITE` 仅表达 direct edge，不允许 consumer 推导 transitive permission union。
- URI deny-first 与 repository fixture authority/port 约束；unknown enum/schema/path 保持 fail closed。
- `RunPathDecisionV1.writable=false` 与 deferred delivery status 不得被后续 Story解释为 runtime enablement。

### Doc 关注点

- CP8 只能陈述 `path_i_design_ready` 上限；五项高阶 claim、真实授权、runtime delivered 与 signal transport 必须继续写 false/deferred。
- S01 fixture PASS 不构成真实授权、真实运行或 PATH-C/A 自动恢复证据。

## 15. CP6R1 回修（F-CP7-001 / F-CP7-002）

| Finding | 根因 | 最小修复 | 回归结果 | 状态 |
|---|---|---|---|---|
| F-CP7-001 | `ActionDecisionV1` 构造期没有 invariant；consumer 只检查外层 kind/context/bool。 | 新增唯一 `_validate_action_decision_invariants`，由 `ActionDecisionV1.__post_init__` 与 `require_action_eligible` 共同调用；强制 approved-ledger hard deny、origin/target typed deny、`eligible ⇒ authorized`、唯一 reason、executable refs 非空。 | direct construction、`dataclasses.replace`、empty refs、invalid reason/origin-target、guard forged-object 与 single-validator dual-call 全通过。 | CLOSED-PENDING-CP7 |
| F-CP7-002 | decoded validity 与 raw prefix 比较不共享表示。 | 新增唯一 `_canonical_logical_uri`；request、record allow/deny 与 prefix 全部消费该表示；v1 拒绝所有 `%` encoding、非 ASCII、非小写 authority/scheme 与非规范 path。 | encoded-unreserved/separator/dot/glob/percent/authority 全拒绝；plain allowed 与 denied subtree deny-first control 通过。 | CLOSED-PENDING-CP7 |

CP6R1 未修改六动作、12-field record、5-edge DAG、四态、8-slot Signal boundary、5 项 claim ceiling 或长期设计。真实 approved-ledger、I/O、network、env、runtime、NAS、R、signal transport、trading 操作仍为 `0`。下一路由为独立 meta-qa-critical CP7 复验；本文的 CLOSED 仅表示实现回修证据完成，不替代 CP7 结论。

---
status: completed
version: "1.1"
story_id: "CR172-S04-execution-cache-materialization"
story_slug: "execution-cache-materialization"
feature_id: "FEAT-CR172-I02"
implementation_type: "mixed"
source_story: "process/stories/STORY-CR172-S04-execution-cache-materialization.md"
source_design_evidence: "process/stories/STORY-CR172-S04-execution-cache-materialization-LLD.md"
created_by: "meta-dev/dev-you"
created_at: "2026-07-18T22:28:04+08:00"
updated_at: "2026-07-18T23:22:09+08:00"
---

# Implementation: CR172-S04 Execution-local Immutable Cache

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-18 | meta-dev/dev-you | 完成 repository-local materialization contract、16 项 LLD fixture、CP6 return/evidence/result。 |
| 1.1 | 2026-07-18 | meta-dev-debugger | CP6R1 关闭 F-001/F-002/F-003：receipt exact-read、current-selection resolver、五类敏感 evidence marker fail-closed；新增 3 组负向测试与 CP6R1 机器证据。 |

## 1. 实现摘要

| 项目 | 内容 |
|---|---|
| 实现目标 | 把 S03 current distribution selection 选中的 immutable bundle 经 S02 唯一 verifier 做 bytes-level `4/4` 复验，物化为 execution-local immutable fixture cache，并用 fresh guard + CAS 生成 receipt、selection 与 local-only handle。 |
| 行为变化 | 新增纯合同模块和纯内存 fixture 测试；当前没有真实 adapter、执行机写入或 runtime consumer。 |
| 范围边界 | 不修改 S01/S02/S03；不创建 seal/digest/verifier facade；不读取 credential/environment；不执行真实 NAS/lake/network/runtime/signal/trading/deploy/Git remote 操作。 |
| CP6 证据 | `process/checks/CP6-CR172-S04-IMPLEMENTATION-DONE.result.json` |

## 2. 上游设计引用

| 来源 | 路径 / ID | 本次消费内容 |
|---|---|---|
| Story packet | `process/context/stories/STORY-CR172-S04.CP6.work-packet.json` | 唯一执行上下文、8 项 allowed write、验证命令、零操作边界。 |
| Story | `process/stories/STORY-CR172-S04-execution-cache-materialization.md` | 范围、AC、dev gate、文件所有权。 |
| full LLD v1.3 | `process/stories/STORY-CR172-S04-execution-cache-materialization-LLD.md` | F-S04-01～13、NFR-S04-01～10、接口、16 个测试 ID；全文读取登记 `RE-20260718T135844Z0000-fe3e2b3f`。 |
| S01 public contract | `engine/path_i_governance.py` | `ActionDecisionV1`、`ActionScopeContextV1`、`require_action_eligible` 与 `execution_pull_verify_materialize`。 |
| S02 public contract | `engine/trial_return_artifact.py` | typed sealed bundle/selection/verified value 与唯一 `verify_sealed_trial_return_bundle`。 |
| S03 public contract | `engine/research_artifact_replica.py` | current `DistributionSelectionV1`、`ReplicaVerificationReceiptV1`、final `read_selected_replica`。 |

## 3. 实现前置检查

| 检查项 | 结果 | 证据 / 说明 |
|---|---|---|
| 上游 Feature 设计存在 | PASS | packet `feature_design_refs` 覆盖 I01/I02/I03。 |
| Story 范围明确 | PASS | objective/non-goals、primary/forbidden paths 均明确。 |
| 待确认问题已关闭 | PASS | LLD `open_items=0`，clarification/OPEN/Spike=`0/0/0`。 |
| 影响范围可定位 | PASS | 两个 primary 工程文件 + 六个 Story/机器证据。 |
| 验证方式明确 | PASS | packet 给出 pytest、py_compile、两项 whitespace、return/result checks。 |
| 当前 Wave / dev_gate 满足 | PASS | CP5 confirmed；S01/S02/S03 dependencies satisfied；S03 CP7R1 PASS。 |
| 文件所有权无冲突 | PASS | S04 独占两个 primary；上游文件只读且 touched=`0`。 |

## 4. 实现对象清单

| 对象 | 类型 | 目标 | 必须 | 验证方式 |
|---|---|---|---|---|
| `engine/research_artifact_materialization.py` | code / contract | 落成 preflight、typed staging、4/4、immutable receipt/cache、fresh guard、CAS、local handle。 | yes | S04 pytest、S01-S04 regression、py_compile、AST boundary guard。 |
| `tests/research/test_cr172_execution_materialization.py` | guardrail / fixture | 覆盖 16 个唯一 LLD ID 与 forbidden-operation/static dependency guard。 | yes | `16 passed`、test ID duplicate=`0`。 |
| Story + IMPLEMENTATION | docs / handoff | 回写 CP6 状态、契约映射、验证与剩余风险。 | yes | Story Return boundary check、人工审查。 |
| Return + Evidence Index | machine evidence | 记录 touched files、命令、测试、风险、route。 | yes | `meta-flow story return-check`。 |
| CP6 Result + Summary | checkpoint evidence | 记录 Entry Criteria、Checklist、Exit Criteria、Deliverables 与决策。 | yes | `meta-flow cp result-check --check-consistency --correlation-profile audit`。 |
| Prompt / Skill / installer / platform adapter | N/A | Story 不涉及这些对象。 | no | N/A。 |

## 5. 设计契约映射

| 设计要求 | 来源 | 实现位置 | 实现动作 | 验证 |
|---|---|---|---|---|
| 授权在任何 read/staging 前完成且 origin/target 精确为 repository fixture | F-S04-01/12 | `validate_materialization_preflight` | 调用 S01 exact guard，未知 authority/real target fail-closed。 | T-S04-N01/N04/N07/N08。 |
| S03 current selection 是唯一数据源 | F-S04-02/13 | `MaterializationStoragePortV1.pull_to_staging` | final method 唯一调用 `ReplicaStoragePortV1.read_selected_replica`，禁止 subclass override。 | T-S04-P01/N02/N06/B03；read call=`1`。 |
| typed non-runtime staging=`3/3` | F-S04-04 | `ExecutionStagingTokenV1`、`pull_and_verify_execution_staging` | 返回 token + sealed bundle + source selection；state 固定 `non_runtime`。 | T-S04-P01/F01。 |
| S02 verifier exactly once 后计算 release/manifest/original seal/content=`4/4` | F-S04-05/13、NFR-S04-01/10 | `pull_and_verify_execution_staging` | 只有一个 verifier call site；receipt 仅 predecessor/correlation。 | T-S04-P01/N05/F03/B03。 |
| 不创建 secondary seal/digest/facade | F-S04-05/11/13 | module imports/public API | 不导入 canonical seal helper，不实现 seal helper/reseal/fallback verifier。 | T-S04-B03；canonical helper/reseal/facade=`0/0/0`。 |
| content-addressed cache 不覆盖，相同 key 仅 identical bytes 幂等复用 | F-S04-06、NFR-S04-03 | `cache_version_ref` + fixture port `persist_immutable_cache` contract | key 绑定 logical URI/release/content；existing bytes 不同即 conflict。 | T-S04-P02/F02。 |
| commit 前 fresh decision 与 S03 selection drift 重验 | F-S04-07 | `commit_execution_cache` | commit decision 必须严格晚于 preflight，context exact，replica port current selection exact。 | T-S04-N03。 |
| CAS 每次最多一次，失败保留 previous selection | F-S04-08、NFR-S04-02 | `commit_execution_cache` | immutable persist 后执行一次 `compare_and_swap_cache_selection`；失败不返回新 receipt/selection/handle。 | T-S04-F02/B03。 |
| receipt/selection canonical identity 排除 physical path/host/mount/credential | F-S04-10、NFR-S04-04/08 | `canonical_materialization_receipt_bytes` | 使用版本化 allowlist JSON；mapping/handle/self hash/time 全排除。 | T-S04-P02/B01。 |
| future runtime 只允许 selected local immutable handle | F-S04-09 | `resolve_execution_local_handle` | 只接受 `ExecutionCacheSelectionV1` + `source_kind=execution_local_immutable_cache`；没有 open/read API。 | T-S04-B02/B03。 |
| 真实操作及上游 pointer mutation 为 0 | F-S04-11/12、NFR-S04-07/09 | 模块边界、packet touched files | 无 os/pathlib/socket/subprocess/requests/open/read_bytes；不写 S01/S02/S03。 | T-S04-B03 + Return boundary。 |

## 6. 单元测试 / Fixture 计划与结果

| 测试对象 | 类型 | 输入 / Fixture | 期望 | 覆盖风险 | 状态 |
|---|---|---|---|---|---|
| 正常 + deterministic/idempotent | unit / fixture | S03 selected immutable tuple、两个授权、不同 mapping roots | `4/4`、receipt hash distinct count=`1`、cache reuse。 | identity drift / overwrite | PASSED |
| 授权 / predecessor / direct-source | contract | missing/expired/wrong action、stale receipt、real target、direct sources | before-read BLOCKED，read/staging/pointer=`0`。 | authorization escape | PASSED |
| mismatch / tampered seal | mutation | expected release/manifest/content 或 seal bytes 篡改 | verifier exactly once；cache/receipt/pointer=`0`。 | receipt-only trust / false seal | PASSED |
| interruption / revoke / selection drift | fault fixture | pull interruption、revoked fresh decision、S03 selection drift | previous selection 保留，CAS=`0`。 | partial/runtime-ready | PASSED |
| immutable / CAS conflict | recovery fixture | conflicting bytes、CAS false | previous selection=`100%` 保留，CAS increment=`1`。 | overwrite / lost update | PASSED |
| static boundary | AST / API inventory | production module source/public API | selected-read/verifier call sites=`1/1`；real-operation imports/open/read_bytes=`0`。 | bypass / runtime leakage | PASSED |

## 7. 最小实现切片

| Slice ID | 对应设计契约 | 改动对象 | 输出文件 | 局部验证 | 状态 |
|---|---|---|---|---|---|
| IMPL-S1 | F01-F05/F12-F13 | typed contract、preflight、S03/S02 dependency split | production module | py_compile + P01/N01/N02/N04/N05/N06/N07/N08/F01/F03 | done |
| IMPL-S2 | F06-F10 | immutable receipt/cache、fresh guard、CAS、local handle | production module | P02/N03/F02/B01/B02 | done |
| IMPL-S3 | §10 16 IDs、NFR safety | in-memory fixture + AST guard | test module | S04 `16/16 PASS` | done |
| IMPL-S4 | CP6 DoD | Story、IMPLEMENTATION、return/evidence/result/summary | 六项证据 | S01-S04 `124/124 PASS` + packet checks | done |

## 8. 变更说明

### 8.1 代码变更

| 文件 | 动作 | 说明 |
|---|---|---|
| `engine/research_artifact_materialization.py` | create | repository-local pure contract 与 fixture port；无真实 adapter。 |

### 8.2 Prompt / Skill 变更

N/A；本 Story 不涉及 Prompt / Skill。

### 8.3 模板 / Schema 变更

N/A；未修改共享模板或 schema。

### 8.4 Guardrail / 测试变更

| 文件 / 命令 | 动作 | 说明 |
|---|---|---|
| `tests/research/test_cr172_execution_materialization.py` | create | 16 个唯一 LLD ID、fault fixture、AST/API inventory。 |
| S01-S04 targeted pytest | run | `124 passed`，failed/skipped=`0/0`。 |

### 8.5 文档变更

| 文件 | 动作 | 说明 |
|---|---|---|
| Story | modify | 状态改为 `ready-for-verification`，回写 CP6 量化结果。 |
| IMPLEMENTATION | create | 本文件。 |
| return/evidence/result/summary | create | CP6 机器证据与人类摘要。 |

## 9. 平台差异处理

| 平台 | 检查项 | 预期 | 结果 |
|---|---|---|---|
| Claude Code / Codex / installer | agent schema、ask、install dry-run | N/A | N/A；Story 只新增 Python 纯合同与 fixture。 |
| Python | 依赖与命令通过 `uv` | yes | PASS；未新增依赖，运行统一使用 `uv run --python 3.11`。 |

## 10. 验证结果

| 命令 / 检查 | 结果 | 证据 |
|---|---|---|
| S04 scoped pytest | PASS | `16 passed in 0.51s`；failed/skipped=`0/0`。 |
| S01-S04 targeted regression | PASS | `124 passed in 0.79s`；failed/skipped=`0/0`。 |
| packet py_compile | PASS | exit=`0`。 |
| 两项 `git diff --no-index --check` | PASS | 两项均为 expected diff exit=`1`，whitespace diagnostics=`0/2`。 |
| Story Return check | PASS | `Story Return Packet Check OK`。 |
| CP6 Result check | PASS | `CP Result Check OK`。 |

## 11. 未覆盖项

| 未覆盖内容 | 原因 | 后续处理 |
|---|---|---|
| 真实 NAS pull、执行机目录、network/mount/credential adapter | packet 明确禁止且本 Story 不授权真实操作。 | 未来独立 runtime-high-risk CR + 新授权 + 真实 adapter 设计与 CP。 |
| future runtime consumer | LLD/Story 明确 out of scope；当前只输出 typed local handle。 | S05 或后续已批准 Story 只可消费 selected local handle。 |
| 真实吞吐/SLA | repository fixture 不代表生产环境。 | 激活前以真实 adapter 的性能计划和故障注入验证。 |

## 12. 风险与回滚

| Risk ID | 风险 | 影响 | 缓解 | 回滚 / 切换条件 |
|---|---|---|---|---|
| R-CR172-SYNC-REPLICA-STALE | S03 selection 在 pull/commit 间漂移 | 物化错误 replica | preflight correlation + commit current selection exact recheck。 | 任一 drift 误接受即回滚 S04 切片并返回设计/实现回修。 |
| R-CR172-RUNTIME-AUTHORIZATION-GAP | repository fixture 被误读为真实执行授权 | 越权真实 operation | origin/target/fixture URI/nominal ports 四重 guard；real operations=`0`。 | 提出真实 adapter/runtime 即 BLOCKED 并新建授权边界。 |
| R-CR172-PATH-IDENTITY-DRIFT | physical mapping 进入 receipt identity | 跨主机 hash 漂移 | canonical allowlist 排除全部 mapping/handle。 | B01 失败即阻断 CP7。 |
| R-CR172-REAL-OPERATIONS | 用户可能误认为 CP6 已部署 | 过度声明 | Story/return/result 显式记录只完成 pure contract。 | 真实 NAS/runtime 仍无授权，不提供降级 shortcut。 |

回滚只需移除 S04 两个新增 primary 文件及六项 CP6 证据；没有真实 cache、execution pointer、NAS/research pointer、runtime 或部署状态需要回滚。

## 13. 设计缺口反馈

| Gap ID | 发现阶段 | 问题 | 应反馈到 | 是否阻塞 | 推荐处理 |
|---|---|---|---|---|---|
| 无 | implementation | 已批准 LLD 与当前 S01/S02/S03 public contracts 可直接实现；长期设计变化=`0`。 | N/A | no | `design_delta_required=false`。 |

## 14. QA / Review / Doc 后续交接

### QA 关注点

- 独立确认 fresh decision 必须严格晚于 preflight，S03 current selection drift 在 immutable persist/CAS 前阻断。
- 对 tampered seal bytes 验证 S02 verifier call=`1`、seal/cache/receipt/pointer=`false/0/0/0`。
- 对 immutable/CAS conflict 复核 previous selection preservation=`100%` 与 CAS 每次最多一次。

### Review 关注点

- `pull_to_staging` 为 final nominal-port contract，bundle 只能来自 S03 `read_selected_replica`；不得引入 receipt-only/path/latest/S02-data lookup。
- receipt hash只覆盖逻辑 identity、授权 refs、4/4 与 cache version ref；mapping/hostname/mount/credential 不得进入。
- 生产模块不得增加真实 I/O、runtime read/open、seal canonicalizer/digest/reseal/facade。

### Doc 关注点

- 用户文档若提及 execution materialization，必须标注当前仅 repository-local contract ready，真实 adapter/runtime 仍未授权。
- S05 或后续 Story 只能消费 `source_kind=execution_local_immutable_cache` 的 selected handle，不得回退 direct NAS/research/staging。

## 15. CP6R1 最小回修

### 15.1 Finding 关闭映射

| Finding | 根因 | 最小回修 | 负向证据 | 状态 |
|---|---|---|---|---|
| F-001 | 相同 cache bytes 遇到 rotated receipt 时，fixture 只复用 cache tuple，selection 可能引用未存 receipt。 | port 增加按 receipt self-hash 的独立不可变持久化与读取 contract；commit 在 CAS 前要求 exact receipt equality。 | same receipt 幂等成功；rotated receipt exact-readable 后成功；receipt persistence failure、immutable conflict、CAS conflict 均零推进并 100% 保留 previous current。 | `closed_pending_cp7r1` |
| F-002 | public resolver 只消费 selection + mapping，未证明 selection 已由当前 port 提交。 | resolver 改为消费 materialization storage port，要求 `current_selection == selection` 且 selection 引用 receipt exact-readable、字段完全绑定。 | old/forged revision+hash/uncommitted accepted=`0/3`；current exact accepted=`1`；mapping/receipt-only capability 均拒绝。 | `closed_pending_cp7r1` |
| F-003 | materialization marker allowlist 缺少 `token`，且 decision refs 到 receipt 构造时才间接验证。 | marker 统一为 password/passwd/secret/token/credential；preflight 和 commit guard 在 persist/CAS 前显式验证 approval/evidence pair。 | 5 markers × 2 phases × 2 ref fields accepted=`0/20`；合法 opaque refs 继续通过；canonical receipt/selection marker count=`0`。 | `closed_pending_cp7r1` |

### 15.2 回修对象与验证

| 对象 | 动作 | 结果 |
|---|---|---|
| `engine/research_artifact_materialization.py` | modify | 新增 receipt read port contract、CAS 前 exact-read、current exact resolver、五 marker early guard；不增加真实 I/O 或 adapter。 |
| `tests/research/test_cr172_execution_materialization.py` | modify | fixture 独立保存 receipt，并新增 F-001/F-002/F-003 三组负向回归；原 16 个 T-S04 ID 保持不变。 |
| S04 scoped pytest | run | `19 passed in 0.59s`；failed/skipped=`0/0`。 |
| S01-S04 targeted pytest | run | `127 passed in 0.79s`；failed/skipped=`0/0`。 |
| packet py_compile | run | exit=`0`。 |
| production/test whitespace | run | expected diff exit=`1/1`；diagnostics=`0/2`。 |

### 15.3 边界、Design Delta 与交接

- 写路径严格为 packet 的 8 项；S01-S03/S05/docs/state/current/plan/ledgers/LLD 修改=`0`。
- 真实 NAS/execution/lake/runtime/network/credential/signal/trading/deploy/Git remote touch/execution=`0`。
- `design_delta_required=false`：回修实现了 CP7 对既有 LLD 合同的精确化验证，没有改变 Feature/HLD/ADR、数据归属或依赖方向。
- 剩余风险只保留真实 adapter/runtime 未授权、fixture 无法代表生产故障与性能；CP7R1 应独立重跑三项 semantic probe 与 S01-S04 回归。

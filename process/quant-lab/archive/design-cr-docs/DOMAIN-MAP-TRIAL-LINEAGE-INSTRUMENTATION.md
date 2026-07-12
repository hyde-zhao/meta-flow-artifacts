---
status: draft-for-cp3
version: "0.1"
cr_id: "CR-163"
---

# Domain Map: Trial Lineage Instrumentation

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 0.1 | 2026-07-11 | meta-se-critical | 冻结六对象、状态机、identity/count、seal/supersession 与 availability 规则。 |

## 1. 术语表

| Term | 定义 | 来源 | 备注 |
|---|---|---|---|
| Experiment Family | 在首次 trial 前声明的有限搜索族，包含固定 identity、search-space/policy commitment | UC-58-CR163 / REQ-001 | 不等于单次 run |
| Trial | 一组稳定参数 + seed 的声明搜索单元 | REQ-002/003 | failed/cancelled/excluded 仍计数 |
| Attempt | 同一 trial 的一次执行尝试 | REQ-002/003 | retry 新增 attempt，不增 raw count |
| Selection | 对 trial/candidate 的 selected/rejected/excluded 决策与理由 | REQ-002 | 不得删除未选 trial |
| Seal | 对规范化 spec + ordered events + version metadata 计算 hash 并生成 immutable manifest | REQ-004 | 只对完整 lineage 成功 |
| Supersession | 新版本显式替代旧 sealed version，同时保留 prior ref/hash/reason | REQ-004/007 | 非原地修改 |
| Native instrumentation | producer 在首个 trial 前使用本 contract 原生记录 | REQ-001/006 | 历史推断不属于 native |
| Availability | `present` / `typed_unavailable` / `not_applicable_with_reason` / `blocked` | CR161 baseline / REQ-006 | CR163 producer paths 通常不使用 N/A |

## 2. 公共领域对象（最终命名）

| Object ID | 最终对象 | Owner | 最小公共属性 | 状态 / 结果 | 规则来源 |
|---|---|---|---|---|---|
| OBJ-CR163-01 | `ExperimentFamilySpec` | lineage core | schema_version, family_id, producer_chain_id, declared_at, declaration_sequence, search_space, selection_policy, spec_hash | declared identity | REQ-001/005 |
| OBJ-CR163-02 | `ExperimentTrial` | lineage core | trial_id, family_id, parameter_fingerprint, seed, declared_sequence, terminal_status, run_links | declared→active→terminal | REQ-002/003 |
| OBJ-CR163-03 | `TrialAttempt` | lineage core | attempt_id, trial_id, ordinal, run_id, experiment_id, started/completed sequence, status, artifact_refs | declared→running→terminal | REQ-002/003 |
| OBJ-CR163-04 | `TrialSelection` | lineage core | selection_id, family_id, trial_id, candidate_ref, decision, reason_code, sequence | recorded immutable event | REQ-002 |
| OBJ-CR163-05 | `ExperimentFamilyManifest` | lineage core | family_id, version, spec_ref/hash, event_log_ref/hash, raw_trial_count, attempt_count, selection_count, run/artifact refs, supersedes_ref/hash, content_hash, sealed_at | sealed / superseded | REQ-004/006 |
| OBJ-CR163-06 | `FamilyLineageValidationResult` | validator | validation_id, target_ref/hash, status, check_results, blocked_reasons, computed_raw_count, availability_projection | pass / blocked / typed_unavailable | REQ-006/007/008 |

`FamilyLineageSession` 是应用 façade，不是第七个持久化领域对象。`ExperimentManifest` 和 `BacktestRunSpec` 是相邻的单次 run identity contract，不归 family lineage 所有。

## 3. Stable Identity Rules

| Identity | 规范 |
|---|---|
| family_id | producer_chain_id + pre-search family key 的稳定、显式 ID；不得从最终候选反推 |
| trial_id | `sha256(canonical({family_id, normalized_parameters, seed}))` 的 namespaced 表示；不同参数或 seed 必不同 |
| attempt_id | trial_id + monotonic attempt ordinal 的稳定 ID；retry ordinal 增加 |
| event_id | producer-generated stable command identity；相同 id+相同 canonical payload 幂等，相同 id+不同 payload 冲突 blocked |
| selection_id | family_id + decision scope + stable candidate/trial ref + sequence 的稳定 ID |
| version | 正整数从 1 连续增加；vN>1 必须带 vN-1（或明确当前合法 head）的 supersedes_ref/hash |

参数规范化必须保留类型差异，mapping key 按 Unicode code point 排序，list 顺序保留，set-like 字段在 schema 中显式声明后排序；禁止 Python `repr` 或浮点本地化字符串作为 identity 输入。

## 4. 状态机

### SM-CR163-01 Family lifecycle

| 当前状态 | 合法触发 | 下一状态 | 前置检查 | 非法转换处理 |
|---|---|---|---|---|
| absent | declare before first trial | declared | spec 完整、inventory mapping 合法 | 已发生 trial 后 declare → blocked_post_hoc_declaration |
| declared | first trial/attempt event | recording | event family_id 匹配、sequence 递增 | orphan/conflict → blocked |
| recording | append lifecycle events | recording | identity/transition 合法 | append rejection evidence；不得覆盖 |
| recording | validate + seal | sealed | completeness/ref/count/hash/tamper 全 PASS | 不生成 sealed head；返回 blocked validation |
| sealed | audit/read | sealed | hash 与 chain PASS | mismatch → availability blocked |
| sealed | append correction + seal new version | superseded（旧）+ sealed（新） | reason + prior ref/hash + valid new stream | 原地写或断链 → blocked |

`blocked` 是 validation/admission 结论，不作为可继续写入的 family lifecycle 状态；修复必须追加 correction 并产生新 version。

### SM-CR163-02 Trial lifecycle

| 状态 | 合法转换 | 计入 raw count | 规则 |
|---|---|---:|---|
| declared | active / never_started / excluded | 是 | 声明即成为 family 成员，不能事后删除 |
| active | succeeded / failed / cancelled / excluded | 是 | terminal 后不回 active；重试通过新 Attempt |
| never_started | terminal | 是 | 必须有 machine-readable terminal reason |
| succeeded / failed / cancelled / excluded | 无（immutable） | 是 | 修正走 supersession，不改事件 |

### SM-CR163-03 Attempt lifecycle

| 状态 | 合法转换 | 规则 |
|---|---|---|
| declared | running / failed / cancelled | attempt_id 唯一且 trial 存在 |
| running | succeeded / failed / cancelled | 每次 retry 新建下一 ordinal attempt |
| terminal | 无 | terminal 事件保留；不能重开同 attempt |

## 5. Count Semantics

| 指标 | 唯一定义 | 包含 | 不包含 / 禁止 |
|---|---|---|---|
| `raw_trial_count` | `count(distinct valid declared trial_id in sealed family membership)` | success, failed, cancelled, excluded, never-started-with-reason | attempts 数、wrapper/hook 调用数、selection 数 |
| `attempt_count` | `count(distinct attempt_id)` | 所有 terminal 与 active-at-seal（active 会使 completeness blocked）attempt | retry 不合并 |
| `selection_count` | `count(distinct selection_id)` | selected/rejected/excluded decision | 不改变 raw count |
| `effective_trial_count` | CR163 不定义计算 | availability=`typed_unavailable` | 不得复制 raw count；ref/method 为空 |

Manifest 声明值必须与 validator 从 canonical event stream 重算值完全相等；任一 mismatch=`blocked`。

## 6. Canonical Serialization and Seal

规范采用 UTF-8 RFC 8259 JSON 的受限 canonical profile：对象 key 递归按 Unicode code point 升序；无 insignificant whitespace；布尔/null 小写；字符串使用 JSON escaping、禁止 Unicode normalization 的隐式变换；整数十进制无前导零；非有限 float 禁止；允许的有限数值先按 schema 转为规范 decimal string。JSONL 每行是一个 canonical event object，以 `sequence,event_id` 唯一确定排序，LF 结尾。

Seal 输入域固定为：`schema_version + canonical ExperimentFamilySpec payload + canonical ordered event payloads + family_id + version + supersedes_ref/hash（若有）`。`sealed_at`、路径、mtime、validation 输出与 `content_hash` 本身不进入 hash。算法固定 `sha256`，表示为小写 64 hex，ref 使用 repo-local artifact URI/path 加 hash；相同逻辑内容 10 次 seal 必须得到一个 hash。

## 7. Availability Projection Rules

| 条件 | Availability | raw lineage fields | effective fields | Admission effect |
|---|---|---|---|---|
| 合法 native declaration + complete events + valid refs/count/hash/chain | present | ref/hash/raw count present | typed_unavailable; ref/method empty | 仅 C1 raw-input-ready；C1 仍不可计算 |
| producer/path 未接入且没有伪造 family | typed_unavailable | ref empty；reason=`native_lineage_not_instrumented` | typed_unavailable | fail closed |
| path 按产品明确 N/A（如不存在 real ML/event runner） | not_applicable_with_reason | reason + scope ref | typed_unavailable | 不进入 P0 producer 分母 |
| post-hoc、incomplete、orphan、conflict、count mismatch、tamper、broken/cyclic supersession | blocked | evidence ref/field + machine code | typed_unavailable | existing gate/package 取更差状态 |

## 8. 业务规则

| Rule ID | 规则 | 影响场景 | 验证入口 |
|---|---|---|---|
| RULE-01 | 首个 trial sequence 必须严格大于 declaration sequence | P01, N01 | ordering fixture |
| RULE-02 | 同 event id 相同 payload 幂等，不同 payload block | N02 | duplicate conflict fixture |
| RULE-03 | seed/参数差异产生 Trial；retry 产生 Attempt | B01 | recount fixture |
| RULE-04 | failed/cancelled/excluded/never-started 保留并计数 | F01 | terminal retention fixture |
| RULE-05 | sealed version immutable；correction 只走 supersession | R01, T01 | hash/chain fixture |
| RULE-06 | consumer 默认选最新合法未撤销 head，且验证整个 chain | R01, T01 | resolver fixture |
| RULE-07 | 手填 legacy count 只对账，绝不建立 present | N01, G01 | compatibility fixture |
| RULE-08 | CR155 无 native lineage，保持 blocked，不 backfill | G01 | negative regression |

## 9. Gotchas

- stable trial identity 必须由预先声明的参数和 seed 产生；若包含结果、score 或 selected 标志，就允许事后改变 identity。
- family `declared` 不代表可 present；只有 sealed manifest 经独立 validation PASS 才能投影 present。
- 幂等重复与 identity conflict 必须区分；后一种不能“last write wins”。
- supersession 的新版本可以修正事实，但旧版本永远保持可验证且默认 resolver 不得跳过断链版本。


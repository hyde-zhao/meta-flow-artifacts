---
checkpoint: CP2
checkpoint_id: CP2-CR173-REQUIREMENTS-BASELINE
cr_id: CR-173
title: Effective-Trial Offline Methodology — Requirements, Failure Semantics and Authorization Review
status: approved
gate_profile: standard-code
opened_at: "2026-07-16T12:16:00+08:00"
context_ref: process/context/CP2-CR173.context.json
cp0_result_ref: process/checks/CP0-CR173-REQUEST-INTAKE.result.json
cp1_result_ref: process/checks/CP1-CR173-USE-CASE-COMPLETENESS.result.json
discussion_ref: process/discussions/CP2-CR173-SCENARIO-DISCUSSION-LOG.md
route_plan_ref: process/checks/CP0-CR173.route-plan.json
result_ref: process/checks/CP2-CR173-REQUIREMENTS-BASELINE.result.json
approval_result: approved
reviewed_by: user
reviewed_at: "2026-07-16T13:00:53+08:00"
---

# CP2 — CR-173 Effective-Trial Offline Methodology

## 门禁修订记录

| 版本 | 时间 | 修订人 | 变更要点 | 门禁效果 |
|---|---|---|---|---|
| r1 | 2026-07-16 12:16 +08:00 | Host Orchestrator / meta-pm | 根据已批准的 CR-172 PATH-B 建立独立离线方法 CR；形成 1 UC、8 REQ、8 SC、8 matrix rows、8 DQ，以及 strategy-agnostic、non-alias、typed-unavailable、no-runtime 决策包。 | pending |
| r2 | 2026-07-16 13:00 +08:00 | Host Orchestrator | 评审确认范围/目标合理；将 projection 公共 contract 触达分类和 estimator 可识别性/Spike 回退提升为 CP3 强制设计义务；用户明确要求整改后批准并推进。 | approved |

## Decision Brief

### 审批者摘要

CR-173 是 CR-172 已批准 PATH-B 的独立方法学前置。它解决的用户问题是：`raw_trial_count` 会把相关试验误作相互独立，不能可靠表达 multiple-testing/data-snooping 暴露；当前 `effective_trial_count=typed_unavailable` 虽然诚实，但不能形成可计算 C1 方法证据。推荐方案是在不接触真实数据、不绑定具体策略的前提下，建立相关性/依赖调整后的 effective-count estimand、七字段 typed evidence、确定性 golden vectors 和 C1 consumer projection。

本门禁只冻结用户可观察行为、失败语义、schema、确定性指标、consumer claim ceiling 和 CR-172 恢复关系。具体 estimator 算法、输入类别/有效域、方法参数、假设、版本/hash 与切换条件必须在 CP3 由 meta-se 比较候选方案后冻结；CP2 `approve` 不能被解释为已选算法、允许 Story/实现或允许真实数据计算。

| 字段 | 内容 |
|---|---|
| 整体目标 | 建立 strategy-agnostic、synthetic/fixture-only、可审计、可复现且 non-alias 的 effective-trial 离线方法基础。 |
| 用户价值 | 未来 consumer 能区分 raw trials 与相关性调整后的独立试验等价量，从而支持真实 multiple-testing/overfit 评估；本 CR 不产生真实 evidence。 |
| 当前产品基线 | `1 UC / 8 P0 REQ / 8 scenarios / 8 matrix rows / 8 DQ`，trace=`100%`，blocker=`0`。 |
| 推荐批准效果 | 接受 DQ-001..008 推荐值，只解锁 CP3 solution-design；CP3 可在方法可识别性不足时转 Spike。 |
| approve 不授权什么 | 不授权算法已选定、Story、LLD、代码/测试实现、真实 lake/NAS/provider/credential、具体策略身份、真实 producer/computation、runtime/write/trading、aggregate/admission、publish/deploy 或 Git remote write。 |
| reject 效果 | CR-173 保持 cp2_pending/not_ready；effective count 继续 typed_unavailable，C1 computable 继续 false，CR-172 继续 blocked。 |

### 业务与方法边界事实表

| 事实 | 当前值 | 审批含义 |
|---|---|---|
| CR-172 路由 | CP2 approved `PATH-B`、`separate_methodology_cr` | CR-173 合法成为独立前置；不继承任何 runtime/data 授权。 |
| 策略身份 | 不需要且不可推断 | `strategy_id/strategy_name` 必填/推断数=`0`；具体身份留 CR-172 activation。 |
| 数据环境 | synthetic / repository fixture / golden vector only | 真实 lake/NAS/provider/credential 操作各=`0`。 |
| effective estimand | 待 CP2 冻结为依赖调整后的独立试验等价量 | 数值偶然等于 raw 也必须有独立方法/provenance；alias/default/fallback=`0`。 |
| 具体算法 | CP2 不选择 | CP3 必须比较候选方法并冻结输入有效域、假设、method/version/hash；不可评审时转 Spike。 |
| typed evidence | 七字段基线 `7/7` | 缺任一必要字段、版本或 provenance 时 available/present=`0`。 |
| 确定性 | 六类 vectors `6/6`，每类重复 `3/3` | 同一规范化输入+method version 只允许 1 个 canonical result。 |
| 最大 claim | `offline_method_ready`（仅 CP8 且证据通过后） | real evidence、Stage 3、admission、aggregate、CR155、CR172 auto-resume 均=`0/false`。 |

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule | `process/context/CP2-CR173.context.json`（由同源 YAML 模板派生，以 JSON 为当前 checker 可验证入口） |
| read profile | `compact`；默认只读 state/current、CR summary、CP0/CP1/CP2 result、人工 checkpoint、meta-pm return。 |
| token budget | estimated `7800` / max `16000`。 |
| 全文档扩展 | 仅缺失、冲突、人工审计或深度评审时；必须写入 `READ-EXPANSION-LEDGER.ndjson`。 |
| 默认禁止读取 | `process/STATE.md`、DEVELOPMENT-PLAN、完整 CR、docs/design、stories、archive、discussion 全文和会话 transcript。 |
| 缺失 / waiver | 具体算法和参数不是缺失 waiver，而是明确的 CP3 设计义务；CP2 blocker=`0`。 |

### Decision Collection Coverage

| 来源 | 扫描状态 | 候选数 | 纳入 DQ | 结论 |
|---|---:|---:|---:|---|
| CR-173 正式范围与量化标准 | scanned | 8 groups | 8 | estimand、input、failure、schema、identity、determinism、projection、recovery 全覆盖。 |
| UC / Scenario Gray Areas | scanned | 4 | 3 映射到 DQ；1 已由 PATH-B 解决 | strategy-agnostic 已有上游用户确认，但本门禁仍把边界纳入 DQ-005 统一批准。 |
| REQ/NFR/SC/TEST-MATRIX | scanned | 8/4/8/8 | 8 | requirement→scenario→matrix trace=`100%`。 |
| HLD / ADR / Story / LLD / implementation | N/A | 0 | 0 | CP2 前依法未生成；正式对象计数均为 0。 |
| 授权与 claim ceiling | scanned | 9 deny operations + 7 false claims | 2 组 | 进入 DQ-007/008 与固定 deny-default，不允许隐式放行。 |

### 决策分层

| 分类 | 数量 | 处理方式 |
|---|---:|---|
| 必须用户决策 | 8 | DQ-CR173-001..008；当前均 OPEN。 |
| 高风险策略确认 | 5 | estimand、failure severity、schema、projection claim、CR172 recovery。 |
| CP3 设计义务 | 1 group | 具体算法/输入合同/假设/版本/hash；CP2 只批准义务，不代选方案。 |
| agent 默认处理 | 0 | 不允许 agent 用 raw fallback、策略标签或历史输入补值。 |
| 仅审计记录 | 3 | CR172 blocked/deferred、CR032 stale index 非重叠债务、operation counts all zero。 |

### 待人工决策清单

| 决策 ID | 类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣与风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|
| `DQ-CR173-001` | product-methodology | effective-count 的产品 estimand 如何与 raw count 区分？ | 冻结为相关性/依赖调整后的独立试验等价量；严禁 alias/default/fallback。 | 保持全量 typed_unavailable，暂不设计 estimator。 | 推荐能创造方法价值，但需 CP3 审慎选型；备选最安全，却不能解除长期不可计算。 | CP3 无法证明可识别性、有效域或偏差边界时回退 typed_unavailable。 |
| `DQ-CR173-002` | architecture-obligation | CP3 必须冻结哪些输入合同与有效性前提？ | 冻结输入类别 inventory、有效域、依赖表示、method/version/hash、假设与切换条件，覆盖率 `100%`。 | 先做方法 Spike，CP3 不批准正式 HLD。 | 推荐可直接形成可评审设计；备选降低错误选型风险，但增加一轮门禁与交付时间。 | 候选方法缺稳定输入映射或 methodology owner 不接受假设时转 Spike。 |
| `DQ-CR173-003` | failure-semantics | 缺失、无效或矛盾输入如何处理？ | 缺失/不足=`typed_unavailable`；矛盾/篡改=`blocked`；永不退回 raw。 | 所有失败统一 `typed_unavailable`。 | 推荐保留安全严重度；备选 consumer 简单，但弱化篡改与矛盾信号。 | 若 CP3 证明 consumer 只支持单一失败态，可统一 unavailable，但必须保留 machine reason 且 alias=0。 |
| `DQ-CR173-004` | schema | consumer evidence schema 是否固定七字段？ | 固定 `7/7` 基线字段及 provenance，不允许隐式默认。 | CP3 可增加字段，但不得删除七个基线字段。 | 推荐使 CP2 可测并稳定 consumer；备选保留扩展性但增加 schema/迁移负担。 | 只有 CP3 证明新增字段是可审计性 blocker 时才能扩展，并记录 ADR。 |
| `DQ-CR173-005` | authorization | estimator 是否绑定具体策略身份？ | strategy-agnostic；具体身份完全留给 CR-172 activation。 | 使用不可回链真实策略的 synthetic case label。 | 推荐避免把 PATH-B 偷换成 activation；备选便于示例，但存在被误读为真实策略的风险。 | 任一标签可回链真实策略时立即删除，改用纯合成 case ID。 |
| `DQ-CR173-006` | verification | 如何验收 deterministic golden vectors？ | 六类 `6/6`、每类重复 `3/3`；合法组 canonical result=`1`，无效组 available=`0`。 | 增加 seeded property tests，但不替代固定 vectors。 | 推荐确定、可审计；备选可扩覆盖，但可能引入随机脆弱性。 | 固定 vectors 全过后，CP5 可把 seeded property tests 作为附加项。 |
| `DQ-CR173-007` | integration-claim | C1 consumer projection 可产生哪些声明？ | 只产生 offline typed evidence；real evidence、Stage3、admission、aggregate 全 false。 | 只交付 estimator，不做 projection。 | 推荐证明实际 consumer 价值；备选半径更小，但不能完整关闭 FU 的 C1 computation 目标。 | 若 CP3 发现公共 C1 contract 需跨域修改，拆 projection 为后续 CR，本 CR 退为 estimator-only。 |
| `DQ-CR173-008` | workflow-authorization | CR-173 与 CR-172 的恢复和授权关系是什么？ | CR-173 是前置；完成后不自动恢复，须 five fields + fresh precheck + 重开 CR-172 CP2。 | CR-173 完成后保持 CR-172 blocked，等待用户另行发起。 | 推荐保留可执行恢复链且不越权；备选更保守，但增加人工重启成本。 | 任一恢复前置不满足时采用备选并保持 blocked；CR-173 CP8 永不关闭 CR-172。 |

### 用户需决策事项

| 字段 | 内容 |
|---|---|
| 待人工决策 | `8/8`，当前均 `OPEN`。 |
| 推荐回复 | `approve`：接受上表八项推荐，只进入 CP3 solution-design。 |
| 修改方式 | `修改: DQ-CR173-00N=<具体值>`；仅修改指定决策，但若影响 REQ/SC/范围则由 meta-pm 增量重整并重开 CP2。 |
| reject | 保持 effective_trial_count typed_unavailable，CR-173/CR-172 均不继续设计。 |
| CP3 特别约束 | 不得把 `approve` 理解为算法已定；若方法输入映射或偏差假设不能收敛，必须在 CP3 转独立 Spike。 |

## Entry Criteria

- [x] CR-172 CP2 已批准 PATH-B 与 separate methodology；CR-172 activation blocked/deferred。
- [x] FU-CR164-004 已正式映射为 CR-173；owner=`strategy-admission methodology owner`。
- [x] fresh conflict precheck 已记录并解决显式授权的 parent/prerequisite overlap；CR-032 stale debt 非重叠。
- [x] CP0 machine result=`PASS`，blocker=`0`，route plan 保留 CP2/CP3/CP5/CP8 人工门禁。
- [x] CP1 产品基线=`PASS`：1 UC、8 REQ、8 SC、8 matrix、8 DQ。
- [x] 用户业务动机/痛点/使用价值/业务触发=`4/4`，用户可见 SGQ=`1/1`。
- [x] 六类场景=`6/6`，schema=`7/7`，golden vector=`6/6 × 3/3`，trace=`100%`。
- [x] Story/Epic/DAG/Wave/LLD/source/test-source=`0/0/0/0/0/0/0`。
- [x] 真实 lake/NAS、credential、provider、真实 computation、runtime、write、trading、Git remote write 各=`0`。

## Checklist

| ID | 审查项 | 推荐值 | 可观察后果 |
|---|---|---|---|
| DQ-001 | estimand / non-alias | dependency-adjusted independent-trial equivalent | raw alias/default/fallback=`0`；CP3 证明可识别性。 |
| DQ-002 | CP3 input/method contract | full inventory/domain/representation/method/version/hash/assumptions | 设计覆盖 100%；不收敛转 Spike。 |
| DQ-003 | failure semantics | unavailable for missing/insufficient; blocked for contradictory/tampered | 不退 raw；machine reason 保留。 |
| DQ-004 | evidence schema | seven required baseline fields | `7/7`；缺一 available/present=`0`。 |
| DQ-005 | strategy identity | strategy-agnostic | 真实策略身份 required/inferred=`0`。 |
| DQ-006 | determinism | 6/6 classes × 3/3 repeats | 每合法组 canonical result=`1`。 |
| DQ-007 | C1 projection | offline typed evidence only | real/Stage3/admission/aggregate claims=`0`。 |
| DQ-008 | CR172 recovery | no auto-resume; 5 fields + fresh precheck + reopened CP2 | CR173 auto-close/resume CR172=`0`。 |

### 不可由本次批准推导的事项

- 具体 estimator 算法、方法参数、相关性输入形态或偏差假设已确定。
- Story、Epic、DAG、Wave、LLD、源代码、测试源码或实现已授权。
- 具体 `strategy_id/strategy_name` 已选择或可从 fixture/history 推断。
- 任何真实 lake/NAS/provider/credential read/write、真实 producer/computation、runtime、QMT/broker、simulation、paper/live trading。
- C1 real evidence、CR-172 activation、C2/C3/C4、FU-006、OI-005、aggregate/FU-009、CR155 promotion 或 mature admission。
- publish、deploy、外部框架运行或 Git remote write。

## Exit Criteria

- [x] `DQ-CR173-001..008` 已由用户统一批准推荐值。
- [x] CP2 `approve` 只解锁 CP3 solution-design，算法/输入合同仍由 CP3 冻结。
- [x] failure semantics、七字段 schema、strategy-agnostic、determinism 与 projection ceiling 已统一接受。
- [x] CR-173 与 CR-172 的 no-auto-resume 恢复契约已接受。
- [x] deny-default 与 claim ceiling 保持；任何隐含 runtime/data/write 授权数=`0`。
- [x] CP3 必须完成 `DO-CR173-CP3-001/002`；不可识别方法转 Spike，跨域公共 C1 contract 修改拆分 projection。

## Deliverables

- Formal CR / conflict / route / CP0：`process/changes/CR-173.md`、`process/checks/CR173-CONFLICT-PRECHECK.json`、`process/checks/CP0-CR173.route-plan.json`、CP0 result。
- 产品基线：8 个 `docs/product/*` 增量文档。
- 自动证据：CP1 result、CP2 auto-precheck result、discussion log/checkpoint、meta-pm return summary。
- 人工门禁：本文件、`process/context/CP2-CR173.context.json`、human-gate launch message、GATE ledger opened event。

## 人工审查结果

状态：**approved**

用户于 `2026-07-16T13:00:53+08:00` 审查 CR-173、CP2 Decision Brief 与 conflict precheck 后明确指示：根据评审完成必要整改，然后批准并推进到下一个人工门禁。评审结论确认 CR-172 PATH-B、owner 分离、CR-173 范围/目标与阶段推进均合理；两项非阻塞提示按以下方式整改：

1. `consumer projection` 可能触及公共 C1 contract：已提升为 `DO-CR173-CP3-002`。CP3 必须完成 contract 触达分类和 `100%` 调用面 inventory；若属于跨 owner、跨域或非兼容修改，projection 拆为后续 CR，本 CR 收缩为 estimator-only。
2. estimator 可识别性可能不收敛：已提升为 `DO-CR173-CP3-001`。CP3 必须证明 estimand、输入映射、有效域、假设与偏差边界；无法收敛时不得硬选算法，保持 `typed_unavailable` 并转 methodology Spike。

本次批准精确接受 `DQ-CR173-001..008` 的八项推荐方案：

| 决策 ID | 已批准值 |
|---|---|
| `DQ-CR173-001` | `dependency_adjusted_independent_trial_equivalent_non_alias` |
| `DQ-CR173-002` | `freeze_full_input_method_contract_or_route_to_spike` |
| `DQ-CR173-003` | `missing_or_insufficient_typed_unavailable_contradictory_or_tampered_blocked` |
| `DQ-CR173-004` | `seven_field_baseline_schema_with_auditable_provenance` |
| `DQ-CR173-005` | `strategy_agnostic_no_real_strategy_identity` |
| `DQ-CR173-006` | `six_of_six_golden_classes_three_of_three_repeats` |
| `DQ-CR173-007` | `offline_typed_projection_only_split_if_public_contract_cross_domain` |
| `DQ-CR173-008` | `no_automatic_cr172_resume_or_close` |

批准后：

- `ready_for_design=true`（仅 CR-173 CP3 solution-design）
- `offline_method_ready=false`
- `effective_trial_count_populatable=false`
- `c1_computable=false`
- `stage3_started=false`
- `stage3_entry_ready=false`
- `real_data_or_runtime_authorized=false`
- `write=false`
- `git_remote_write=false`

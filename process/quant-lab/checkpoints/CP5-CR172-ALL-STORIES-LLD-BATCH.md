---
checkpoint_id: "CP5-CR172-ALL-STORIES-LLD-BATCH"
checkpoint_name: "CR172 PATH-I All Stories Full LLD Design Evidence"
type: "batch_auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-07-18T12:55:01+08:00"
reviewed_by: "user"
reviewed_at: "2026-07-18T13:49:27+08:00"
auto_check_result: "process/checks/CP5-CR172-LLD-DESIGN-EVIDENCE.result.json"
context_ref: "process/context/CP5-CR172-PATH-I-LLD-CONTEXT.yaml"
decision_brief_profile: "compact"
target:
  phase: "story-planning"
  story_id: "CR172-S01..S05"
  artifacts:
    - "process/stories/STORY-CR172-S01-action-authorization-eligibility-governance-LLD.md"
    - "process/stories/STORY-CR172-S02-trial-return-artifact-native-hook-LLD.md"
    - "process/stories/STORY-CR172-S03-nas-replica-verification-LLD.md"
    - "process/stories/STORY-CR172-S04-execution-cache-materialization-LLD.md"
    - "process/stories/STORY-CR172-S05-path-i-integration-claim-zero-operation-verification-LLD.md"
---

# CP5 — CR-172 PATH-I 全量 Story 设计证据人工审查

> 本门禁只审查和授权 repository-local pure contract、fixture port、测试与本地验证；不授权现有 runner 改造、真实 trial-return、真实数据湖/NAS/执行机、runtime、R、信号或交易操作。

## 自动预检摘要

| 预检 | 结论 | Blocking | Required | Optional | 说明 |
|---|---|---:|---:|---:|---|
| CP3 / CP4 | PASS | 0 | 0 | 0 | ReturnDefinition ADR owner、native producer design obligation、六动作 eligibility DAG 已冻结；Feature/Story/Wave/Task=`3/5/5/20`。 |
| Full LLD / correlation | PASS | 0 | 0 | 0 | 5 份独立 full LLD v1.2，结构检查 `5/5 OK`；REQ/scenario/outcome=`15/15,27/27,11/11`；12 个 primary path 无重复。 |
| Independent review R1-R4 | PROCEED | 0 | 0 | 1 | R1-R3 的 blocking/required 全部关闭；R4=`0/0/1`，仅 LLD authority pointer freshness 非阻塞。 |
| Authorization / operation | PASS | 0 | 0 | 0 | 实现前源码/测试/fixture、真实 lake/NAS/runtime/trial-return/R/signal/trading/deploy/remote 操作全部 `0`。 |

## Decision Brief

### 审批者摘要

| 字段 | 内容 |
|---|---|
| 本次确认服务的整体目标 | 批准 PATH-I 的 repository-local 合同与 fixture 可验证实现：六动作 deny-default、trial-return sealed artifact contract、NAS replica verification contract、执行机本地 cache materialization contract，以及零真实操作/claim ceiling 回归。 |
| 推荐动作 | 批准 S01→S02→S03→S04→S05 五个串行 Story、12 个计划路径和本地 `uv run` 验证；五份 LLD 已在实现前把 HLD/ADR 指针从 v1.3 仅刷新到 v1.4，规范变化为 0。 |
| approve 后会发生什么 | Host 按五个串行 Wave 调度 meta-dev 完成各 Story CP6，并由 meta-qa-critical 独立完成各 Story CP7；发生 FAIL、NEEDS_REWORK、NEEDS_DESIGN_CLARIFICATION、授权越界或 workflow-health 阈值时立即停止；全部通过后停在 CP8 人工门禁。 |
| approve 不授权什么 | 不授权修改现有 mature runner/lineage store，不授权真实 native multi-trial producer、真实 approval ledger adapter、真实 lake/NAS/执行机、multi-trial runtime、trial-return、empirical R、SignalBatch 传输、交易、迁移、部署或 Git remote write。 |
| 不确认会阻塞什么 | 12 个 repository-local 路径、20 个任务、全部 CP6/CP7 和 CP8 交付准备继续锁定。 |

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| Capsule | `process/context/CP5-CR172-PATH-I-LLD-CONTEXT.yaml` |
| 状态 | `ready_for_cp5_human_gate`；capsule-first / compact。 |
| read_profile | `compact`；完整 LLD/HLD 只在独立深审和人工审计时展开，扩展读取已记入 READ-EXPANSION ledger。 |
| 默认读取策略 | capsule-first；先读机器状态、CP4 result、Feature/Story 摘要与 R4 summary，仅在冲突、深审或人工审计时展开完整 LLD/HLD。 |
| 全文档读取 | 五份 LLD 已按 deep-review 读取并登记 5 个 READ-EXPANSION ref；HLD/ADR 仅按 R1-R4 finding 需要展开相关章节。 |
| Design batch | expected/completed/confirmed=`5/5/0`；人工批准后才将 confirmed 更新为 `5`。 |
| Independent review | Round 4 `proceed`；blocking/required/optional=`0/0/1`。 |
| Clarification | blocking/open=`0/0`；五份 LLD `open_items=0`。 |
| 授权边界 | CP5 前 source/test/fixture implementation=`0/0/0`，全部真实/外部/远端操作=`0`。 |

### Scope correction：为什么当前没有真实 native hook 实现

CP3 原设计义务要求 CP5 明确 native producer 插桩位置。源代码核对后确认：当前仓库没有可识别的 multi-trial portfolio-period-return producer；现有 `net_forward_return` 是 forward-label proxy，不是 trial portfolio period return，CR-163 lineage 也是 append-only metadata store。为避免伪造真实回报来源，本批次已安全收窄为 pure contract + repository fixture producer port：

- 现有 runner / lineage planned diff=`0/0`；
- `forward_label_proxy@v1` 作为 trial-return / empirical-R / effective-count 输入的接受数=`0/0/0`；
- 真实 native producer、observation interval/holdings/cost/return basis、atomic lineage/correction 和 runtime path enforcement 被列为独立 runtime-high-risk 前置；
- 因此本 CR 交付“可实现的安全合同”，不冒充“真实回报生产已接入”。

### 全量设计摘要

| Story | 主要实现路径 | 冻结合同 | 关键验收 |
|---|---|---|---|
| S01 Authorization / Claim | `engine/path_i_governance.py` + unit tests | 12-field record、六动作独立审批、5 条 eligibility edge、fixture typed origin/target、RunPathDecision contract-only | approved-ledger current-v1 authorized/eligible=`0/0` for `6/6`；runtime-without-read side effects=`0/0/0` |
| S02 Trial-return Artifact | `engine/trial_return_artifact.py` + unit tests | payload exact 2 columns、manifest/seal/hash/selection、fixture producer port、single verifier truth | timestamp/simple_return=`2/2`；runner/lineage diff=`0/0`；forward proxy misuse=`0/0/0` |
| S03 NAS Replica | `engine/research_artifact_replica.py` + unit tests | S03 selected-replica read、immutable receipt/CAS、S02 verifier consumption | selected tuple=`3/3`；new digest/verifier facade/data bypass=`0/0/0` |
| S04 Execution Cache | `engine/research_artifact_materialization.py` + unit tests | S03 唯一供数、S02 verifier-library、non-runtime staging、4/4 verify、local CAS | verifier call=`1`；tampered seal pointer=`0`；bypass/receipt-only/secondary digest=`0/0/0`；IDs=`16/16/0` |
| S05 Integrated QAC | fixture root + 3 research tests | 15 REQ/27 scenarios/11 outcomes、六动作零真实执行、五项 claim false、REQ013 split | coverage=`15/15,27/27,11/11`；六动作 real authorized/executed=`0/6,0/6`；five claims=false |

### 四轮独立评审整改结论

| Finding 组 | 最终状态 | 冻结结论 |
|---|---|---|
| R1 source fact / lineage / return semantics | CLOSED | 无真实 native producer；runner/lineage diff 0/0；forward-label 不冒充回报；partial lineage 只能 blocked audit。 |
| R2 trusted origin / seal bytes / fixture truth / REQ013 | CLOSED | caller 不能自报 approved ledger；S04 校验实际 sealed bytes；fixture provenance 单一真相；REQ013 runtime-deferred。 |
| R3 S03 verifier facade 双解释 | CLOSED | HLD/ADR v1.4 删除 facade 备选；S03 只供 selected data，S04 直接调用 S02 唯一 verifier。 |
| R4 pointer freshness | OPTIONAL | 五份 LLD 内容已与 v1.4 一致，但 §0 仍引用 v1.3；CP6 实现前必须 pointer-only refresh，normative delta=`0`。 |

### Decision Collection Coverage

| 来源 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---:|---:|---:|---|
| CP3 accepted DQs / HLD / ADR | scanned | 4 | 0 | 已批准并经 R1/R4 修订，作为不可放宽架构约束。 |
| CP4 DAG / ownership | scanned | 1 | 1 | 五个串行 Wave 与 12/12 primary owner。 |
| 五份 LLD / CP5 result | scanned | 3 | 3 | implementation contract、local authorization、claim/runtime boundary。 |
| R1-R4 independent review | scanned | 9 | 0 | blocking/required 全部关闭；optional pointer freshness转 CP6 前置。 |
| Clarification queue | scanned | 0 | 0 | blocking/open=`0/0`。 |
| 真实/外部/远程权限 | N/A | 0 | 0 | 明确禁止，不能由 CP5 隐式升级。 |

### 决策分层

| 分类 | 数量 | 处理 |
|---|---:|---|
| 必须用户决策 | 4 | 下表四项 DQ。 |
| 高风险策略确认 | 2 | repository-local implementation authorization；claim/runtime boundary。 |
| agent 默认处理 | 2 类 | private helper 名称、repository fixture 的具体 token；不得改变 schema、hash、owner、DAG、测试或授权。pointer-only refresh 已完成。 |
| 仅审计记录 | 14 类 | 3/5/5/20、15/15、27/27、11/11、12 paths、5 LLD、4 rounds、16/16/0、6-action deny、5-edge DAG、proxy 0/0/0、runner/lineage diff0、forbidden ops0、claim false。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| `DQ-CP5-CR172-IMPLEMENTATION-CONTRACT` | implementation | 是否批准五份 full LLD 作为 PATH-I 实现合同？ | 批准完成 pointer-only refresh 的 S01-S05 v1.3，normative delta=`0`。 | 指定 Story/章节修改并保持门禁。 | 推荐方案已通过四轮独立审查且 required=0；备选更细但会阻断全部串行依赖。 | 解锁实现合同；任何规范变化都可能破坏跨 Story seal/authorization 边界。 | 实现细节变化回 CP5；架构变化回 CP3；范围变化回 CP2。 |
| `DQ-CP5-CR172-WAVES-OWNERSHIP` | implementation | 是否批准五个串行 Wave 与 12/12 primary owner？ | `S01→S02→S03→S04→S05`，每 Wave 一个 Story。 | 调整 DAG/owner 后重跑 CP4。 | 串行吞吐较低，但能避免共享合同、seal、selection 和 QAC 并发漂移。 | 决定 merge 顺序和单写边界。 | 依赖、owner、primary path 变化立即停止并回 CP4。 |
| `DQ-CP5-CR172-LOCAL-AUTHORIZATION` | security | 是否只授权 12 个计划路径的 repository-local 编辑和本地验证？ | 只授权 4 个 engine 路径、7 个 test 路径/模块和 1 个 fixture root；只允许本地 `uv run`。 | 保持 design-only；额外权限另开人工门禁/CR。 | 推荐是交付 fixture 合同的最小权限；design-only 无法验证实现，扩到真实操作又没有依据。 | 不会产生真实 evidence 或 runtime 能力。 | 任何真实 lake/NAS/runtime/producer/signal/trading/remote 需求立即停止。 |
| `DQ-CP5-CR172-CLAIM-AND-RUNTIME-BOUNDARY` | risk_acceptance | 是否接受 fixture-only 验证和 PATH-I design-ready claim ceiling？ | 接受 current-v1 real actions 仍不可用、REQ013 runtime deferred、CP8 最高 `path_i_design_ready=true`。 | 暂停至真实 producer/adapter 获独立授权。 | 推荐先证明合同与 fail-closed 边界；备选独立性更强但会继续悬置基础合同。 | 不会使 C1 computable 或 Stage3 entry-ready。 | 出现真实 producer、empirical-R、public C1、runtime path switch 或 signal exchange 请求时转独立 CR。 |

### 授权说明

回复 `approve` 或“批准 CP5”仅授权以下 12 个 repository-local 路径及直接相关本地 `uv run` 验证：

1. `engine/path_i_governance.py`
2. `tests/research/test_cr172_path_i_governance.py`
3. `engine/trial_return_artifact.py`
4. `tests/research/test_cr172_trial_return_artifact.py`
5. `engine/research_artifact_replica.py`
6. `tests/research/test_cr172_nas_replica_verification.py`
7. `engine/research_artifact_materialization.py`
8. `tests/research/test_cr172_execution_materialization.py`
9. `tests/fixtures/cr172_path_i/`
10. `tests/research/test_cr172_path_i_integration.py`
11. `tests/research/test_cr172_path_i_authorization.py`
12. `tests/research/test_cr172_path_i_claim_regression.py`

批准后仍明确禁止：修改现有 `engine/mature_multifactor_research.py`、`engine/experiment_family_lineage.py`；真实数据湖/NAS/执行机操作；真实 multi-trial runtime/trial-return/R；真实 approved-ledger adapter；信号传输/ack；交易；历史迁移；部署；Git remote write；Stage3/C1/admission/aggregate 正向声明。

### 用户需决策事项 summary

| 字段 | 内容 |
|---|---|
| 本轮待人工决策项 | `DQ-CP5-CR172-IMPLEMENTATION-CONTRACT`、`DQ-CP5-CR172-WAVES-OWNERSHIP`、`DQ-CP5-CR172-LOCAL-AUTHORIZATION`、`DQ-CP5-CR172-CLAIM-AND-RUNTIME-BOUNDARY`。 |
| 推荐回复 | `approve CR-172 CP5，按 S01→S02→S03→S04→S05 串行实施并推进到下一个人工门禁`。 |
| Approve 效果 | 确认 5 份 full LLD，授权 12 个计划路径的 repository-local implementation 与 local `uv run` verification，并自动推进至 CP8 人工门禁或提前停止条件。 |
| 不表示授权 | 真实/外部/远端、现有 runner/lineage 修改、真实 trial-return/R、runtime、signal、trading、Stage3/C1、发布或部署。 |

## Entry Criteria

| 条目 | 状态 | 证据 |
|---|---|---|
| CP3 approved | PASS | `process/checkpoints/CP3-CR172-PATH-I-HLD-REVIEW.md` |
| CP4 PASS | PASS | `process/checks/CP4-CR172-PATH-I-STORY-DAG-PARALLEL-SAFETY.result.json` |
| CP5 capsule ready | PASS | `process/context/CP5-CR172-PATH-I-LLD-CONTEXT.yaml` |
| 5/5 full LLD structure ready | PASS | `process/checks/CP5-CR172-LLD-R3-CROSS-CONTRACT-CORRELATION.result.json` |
| Independent review required=0 | PASS | `process/checks/CP5-CR172-LLD-REVIEW-R4-SUMMARY.md` |
| Clarification=0 | PASS | CP5 capsule + QUESTION ledger |

## Checklist

| # | 检查项 | 待人工结果 | 证据 / 审查要点 |
|---:|---|---|---|
| 1 | 批准 S01-S05 v1.3 五份 full LLD；pointer-only refresh 已完成且 normative delta=`0` | approved | `DQ-CP5-CR172-IMPLEMENTATION-CONTRACT`；R4 blocking/required=`0/0`。 |
| 2 | 批准 5-Wave serial DAG 与 12/12 primary owner | approved | `DQ-CP5-CR172-WAVES-OWNERSHIP`。 |
| 3 | 只批准 12 个 repository-local 路径和本地 `uv run` 验证 | approved | `DQ-CP5-CR172-LOCAL-AUTHORIZATION`。 |
| 4 | 接受 fixture-only verification、REQ013 runtime-deferred 与 PATH-I claim ceiling | approved | `DQ-CP5-CR172-CLAIM-AND-RUNTIME-BOUNDARY`。 |

## Exit Criteria

| 条目 | 当前状态 | 说明 |
|---|---|---|
| 全量设计证据 ready | PASS | 五份 LLD 均 v1.3/confirmed、open_items=0。 |
| Required review finding 归零 | PASS | Round 4 blocking/required=`0/0`。 |
| CP6 最小授权显式 | PASS | 仅 12 个 repository-local 路径与本地 `uv run`。 |
| 禁止项未被隐式授权 | PASS | No real data/NAS/runtime/trial-return/R/signal/trading/remote。 |
| 人工决策已记录 | PASS | 用户批准四项 DQ；三项 optional 整改已完成。 |

## Deliverables

| 交付物 | 路径 | 状态 |
|---|---|---|
| CP4 result | `process/checks/CP4-CR172-PATH-I-STORY-DAG-PARALLEL-SAFETY.result.json` | PASS |
| CP5 context | `process/context/CP5-CR172-PATH-I-LLD-CONTEXT.yaml` | ready |
| CP5 auto result | `process/checks/CP5-CR172-LLD-DESIGN-EVIDENCE.result.json` | PASS / human pending |
| Round 4 independent review | `process/checks/CP5-CR172-LLD-REVIEW-R4-SUMMARY.md` | proceed / required 0 |
| Five full LLDs | `process/stories/STORY-CR172-S0*-*-LLD.md` | v1.3 / confirmed / normative delta 0 |

## 人工审查结果

- 结论：`approved`
- 审查人：`user`
- 审查时间：`2026-07-18T13:49:27+08:00`
- 接受的决策 ID：`DQ-CP5-CR172-IMPLEMENTATION-CONTRACT`、`DQ-CP5-CR172-WAVES-OWNERSHIP`、`DQ-CP5-CR172-LOCAL-AUTHORIZATION`、`DQ-CP5-CR172-CLAIM-AND-RUNTIME-BOUNDARY`
- 修改意见：批准前完成三项 optional 整改：五份 LLD authority pointer 刷新到 v1.4、S01 F-04 使用完整 action kind、S05 I01 TEST-PLAN 指针刷新到 v1.2；不得借机改变已审查合同，实际 normative delta=`0`。
- 风险接受项：接受 fixture-only 验证、REQ-013 `contract_ready/runtime_enforcement_deferred` 与 CP8 最高 `path_i_design_ready=true`；真实六动作、C1/Stage3 和信号/交易仍不授权。

CP5 已批准并解锁 S01 CP6。S02～S05 继续受 `S01→S02→S03→S04→S05` 串行依赖门控。

---
checkpoint_id: "CP5-CR173-ALL-STORIES-LLD-BATCH"
checkpoint_name: "CR173 All Stories Full LLD Design Evidence"
type: "batch_auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-07-16T17:36:33+08:00"
reviewed_by: "user"
reviewed_at: "2026-07-17T09:33:11+08:00"
auto_check_result: "process/checks/CP5-CR173-LLD-DESIGN-EVIDENCE.result.json"
context_ref: "process/context/CP5-CR173.context.json"
decision_brief_profile: "compact"
target:
  phase: "story-planning"
  story_id: "CR173-S01..S03"
  artifacts:
    - "process/stories/STORY-CR173-S01-contract-evidence-canonicalization-LLD.md"
    - "process/stories/STORY-CR173-S02-exact-spectral-estimator-LLD.md"
    - "process/stories/STORY-CR173-S03-golden-failure-boundary-verification-LLD.md"
---

# CP5 — CR-173 全量 Story 设计证据人工审查

> 本文把 3 份**相互独立的 full LLD**作为一次人工审查批次提交；“batch”只表示统一门禁审查，不表示使用 `batch-lld` 或合并 Story 合同。

## 自动预检摘要

| 预检 | 结论 | 阻断 | Required | Optional | 说明 |
|---|---|---:|---:|---:|---|
| CP3 / CP4 | PASS | 0 | 0 | 0 | CP3 已批准 limited participation-ratio claim 与 estimator-only split；CP4 为 1 Feature、3 Story、3 串行 Wave、12 Task。 |
| CP5 context / LLD | PASS | 0 | 0 | 0 | Context Pack Check 与 CP5 Context Check 均 OK；full LLD `3/3`，structure check `3/3 OK`，open clarification=`0`。 |
| Independent review Round 1 | REVISE | 0 | 3 | 1 | identity/audit、F03/F04、public-call 计数口径进入整改。 |
| Independent review Round 2 | REVISE | 0 | 2 | 1 | S03 bitmap/basis 时序与 operation inventory 数量进入二次整改。 |
| Independent review Round 3 | PROCEED | 0 | 0 | 1 | 所有 required finding 已关闭；仅 LLD §0 authority-version 指针滞后，为非阻断审计新鲜度项。 |
| Post-review pointer refresh | PASS | 0 | 0 | 0 | 三份 LLD §0 已刷新至 HLD/Domain/ADR v1.2 与 Feature v0.3；normative contract delta=`0`。 |
| 授权边界 | PASS | 0 | 0 | 0 | 实现、native/public 测试执行、真实数据、public C1、runtime、trading、发布和 Git remote 操作均为 0。 |

## Decision Brief

### 审批者摘要

| 字段 | 内容 |
|---|---|
| 本次确认服务的整体目标 | 批准一个 strategy-agnostic、fixture-only、exact-rational、fail-closed 的 effective-dimensionality 离线 estimator 实现合同，为未来真实 multiple-testing/overfit 评估提供相关结构有效维度基础。 |
| 推荐动作 | 已批准 S01 v1.3、S02 v1.3、S03 v1.4，批准 `S01 → S02 → S03` 三个串行 Wave，并仅授权 7 个计划路径的 repository-local 实现和本地 `uv run` 验证。 |
| approve 后会发生什么 | Host 将按 S01→S02→S03 串行调度 meta-dev 完成 CP6，并由 meta-qa-critical 独立完成各 Story CP7；遇到 FAIL、NEEDS_REWORK、NEEDS_DESIGN_CLARIFICATION、授权越界或 health 阈值立即停止，全部通过后停在 CP8 人工门禁。 |
| approve 不授权什么 | 不授权真实/empirical matrix、具体策略身份、credential/provider、lake/NAS、public C1 projection/write、Gate1 blocker removal、FWER/DSR/tail calibration、admission/Stage 3、CR172 自动恢复、runtime/trading、publish/deploy 或 Git remote write。 |
| 不确认会阻塞什么 | 7 个计划实现路径、T01-T12、全部 CP6/CP7 和 CP8 交付准备继续保持锁定；现有 public C1 effective count 继续 typed unavailable。 |

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| Capsule | `process/context/CP5-CR173.context.json` |
| 状态 | ready；capsule-first / compact；Context Pack Check 与 CP5 Context Check 均 OK。 |
| 预算 | estimated `18,790 / 22,000` tokens。 |
| read_profile | `compact`。 |
| 默认读取策略 | capsule-first；先读机器状态、CP4 result、Feature/Story 摘要与 Round 3 summary，只有人工深审、冲突或审计才展开完整 LLD/HLD。 |
| Must-read | 机器状态、CP4 result、Feature 三件套、3 张 Story 卡、Round 3 review summary。 |
| Read-if-needed | CR173 HLD/ADR/Dependency Map 与三份 full LLD；全文扩展均已记录 READ-EXPANSION ref。 |
| 全文档读取 | 三份 CR173 LLD 与 Dependency Map public inventory 已按 deep-review/audit 记录 4 个 expansion refs；DEVELOPMENT-PLAN scoped block另有 1 个 expansion ref。 |
| Design batch | expected/completed/confirmed=`3/3/3`；review rounds=`3`；unresolved blocking/required/optional=`0/0/0`。 |
| Clarification | blocking/open=`0/0`；三份 LLD `open_items=0`。 |

### 全量设计摘要

| Story | 设计证据 | 主要实现路径 | 冻结合同 | 量化验收 |
|---|---|---|---|---|
| S01 Contract / Evidence | LLD v1.3 | `engine/effective_trial_evidence.py`；`tests/research/test_effective_trial_evidence_contracts.py` | 四类 immutable contract、七字段 standalone evidence、canonical decimal/bytes/hash、stable computation ref、七项 basis、外置 attempt audit、F01-F08、recovery A/B | evidence=`7/7`；basis=`7/7`；raw alias=`0`；same basis=`1 computation + 1 evidence`；audit repeat=`3/3` |
| S02 Exact Estimator | LLD v1.3 | `engine/effective_trial_estimator.py`；`tests/research/test_effective_trial_estimator.py` | exact decimal→rational、deterministic pivot、fraction-free exact `LDLᵀ`、`q=n²/ΣRᵢⱼ²`、双 invariant、一次 half-even | PSD oracle=`4/4`；`1≤q≤n` before/after rounding=`2/2`；float/tolerance/clamp=`0/0/0` |
| S03 Golden / Boundary | LLD v1.4 | `tests/fixtures/effective_trial/golden_vectors_v1.json`；两份 CR173 verification tests | raw-only execution、actual outcome 后 finalize basis、expected oracle 旁路、golden/failure/mutation、NP 9/9、public 8+12 与六指标 | golden=`6/6×3/3`；failure=`8/8`；returned basis=`8×7/8×7`；NP=`9/9`；public=`8+12` |

### 三轮独立评审整改结论

| Finding | 最终状态 | 已冻结的结论 |
|---|---|---|
| Stable computation identity vs attempt audit | CLOSED | 七字段 computation ref 是 content-addressed stable identity；逐执行 audit 位于七字段之外，owner/linkage 明确，persistence/retention=`N/A/N/A`。 |
| F03 / F04 | CLOSED | `NaN`/`Inf` 等 non-canonical token 唯一进入 F03；F04 只处理已解析为有限 exact rational 的 matrix-domain failure。 |
| Public call scope | CLOSED | CR173 new-code edge/call/diff/write=`0/0/0/0`；既有 public regression 只属于 12/12 read-only lane，expected edits=`0`。 |
| S03 bitmap / basis / oracle | CLOSED | Bitmap 只有 identity/envelope/method 三位；attempted evidence 仅 snapshot；actual outcome 后 finalize；expected oracle 对执行输入影响=`0`；F01-F08=`8×7/8×7`。 |
| Operation inventory | CLOSED | Non-public `NP-01..09=9/9` 且各 counter=0；public 六指标独立，跨组 duplication=`0`。 |
| Authority pointer freshness | CLOSED | 三份 LLD §0 已完成 pointer-only refresh，统一引用 HLD/Domain/ADR v1.2 与 Feature v0.3；normative contract delta=`0`。 |

### 关键实现与验证合同

1. `effective_trial_computation_ref` 由七项 `EffectiveTrialAttemptBasisV1` 内容寻址；run/case/ordinal/time/worker/random/audit ref 进入 basis 的接受数为 0。
2. 唯一流水线是：raw components → S01 validation → 合法时 S02 estimator → actual outcome → finalize basis → stable ref → 七字段 evidence/hash → external audit；expected oracle 只能事后比较。
3. PSD 校验只用 exact-rational、确定性 symmetric pivot、fraction-free `LDLᵀ`；binary float、tolerance、clamp 和随机 pivot 均禁止。
4. participation ratio 只声明“二阶相关结构有效维度”，不是 Li-Ji effective independent tests，也不能直接作为 BH/FWER/Šidák/DSR/admission calibration。
5. Public C1 production 文件 `8/8` 保持只读/禁止，existing regression/authorization `12/12` 只读运行；任一路径缺失、expected relaxation 或 production diff 立即停线。
6. 回滚只删除/禁用 CR173 新增的 estimator-only 文件和 fixture；current public C1、Gate1、DSR、admission 保持 byte/semantic-equivalent，绝不回退到 raw count。

### Decision Collection Coverage

| 来源 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---:|---:|---:|---|
| CP3 accepted DQs / HLD / ADR | scanned | 2 | 0 | 已批准，作为不可放宽的算法 claim 与 estimator-only split。 |
| CP4 DAG / ownership | scanned | 1 | 1 | 三个串行 Wave 与 7/7 primary file ownership。 |
| 三份 LLD / CP5 result | scanned | 3 | 3 | implementation contract、local authorization、claim/verification boundary。 |
| Round 1-3 independent review | scanned | 6 | 0 | Required 已全部关闭；optional pointer freshness已在批准前机械刷新，未改变合同。 |
| Clarification queue | scanned | 0 | 0 | blocking/open=`0/0`。 |
| 真实/外部/远程权限 | N/A | 0 | 0 | 明确禁止，不能由 CP5 隐式升级。 |

### 决策分层

| 分类 | 数量 | 处理 |
|---|---:|---|
| 必须用户决策 | 4 | 下表四项 DQ。 |
| 高风险策略确认 | 2 | repository-local implementation authorization；claim/verification boundary。 |
| agent 默认处理 | 3 类 | private helper 名称、synthetic fixture 的具体合法 token、纯 authority-pointer refresh；不得改变 schema、算法、9+6、8+12 或 claim ceiling。 |
| 仅审计记录 | 15 类 | 1/3/3/12、7 paths、3 LLD、3 rounds、7/7、6×3、8/8、8×7、4 PSD、9/9、8+12、public six、clarification 0、forbidden ops 0。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| `DQ-CP5-CR173-IMPLEMENTATION-CONTRACT` | implementation | 是否批准三份 full LLD 作为 CR173 实现合同？ | 批准 pointer-only refresh 后的 S01 v1.3、S02 v1.3、S03 v1.4；规范合同变化数为 0。 | 指定需修改的 Story/章节并保持门禁。 | 推荐方案已通过三轮独立评审且 required=0；纯审计指针已关闭。 | 解锁设计确认；任何后续规范语义变化都可能破坏 computation identity 或验证 oracle。 | 实现细节变化回 CP5；算法/公共边界变化回 CP3；范围变化回 CP2。 |
| `DQ-CP5-CR173-WAVES-OWNERSHIP` | implementation | 是否批准三串行 Wave 与 7/7 文件 owner？ | S01 → S02 → S03，每 Wave 一个 Story，primary path `2+2+3=7` 唯一。 | 调整 DAG/owner 后重跑 CP4。 | 串行吞吐较低，但避免合同、算法、fixture 并发漂移和跨 Story 写冲突。 | 决定 merge 顺序和单写边界。 | 依赖、owner 或 primary path 变化立即停止并回 CP4。 |
| `DQ-CP5-CR173-LOCAL-AUTHORIZATION` | security | 是否只授权 7 个计划路径的 repository-local 编辑和本地验证？ | 只授权两个 engine 文件、四个 test 文件/模块和一个 synthetic golden fixture；只允许本地 `uv run`。 | 保持 design-only；额外权限另开人工门禁/CR。 | 推荐是完成离线方法的最小权限；design-only 无法交付，扩大到真实/runtime/public 没有必要且越界。 | 解锁 CP6/CP7，但不产生真实 evidence、public C1 或 Stage 3 能力。 | 任何 credential/provider/real/public/runtime/trading/remote 需求立即停止并申请新授权。 |
| `DQ-CP5-CR173-CLAIM-AND-VERIFICATION-BOUNDARY` | risk_acceptance | 是否接受 fixture-only 验证和 estimator-only claim ceiling？ | 由 meta-qa-critical 独立验证 6×3、8/8、7/7、9+6、8+12；保持 `public_effective_trial_count_populatable=false`、`c1_computable=false`。 | 暂停至 empirical matrix 或 public projection 另行获批。 | 推荐可先证明方法合同与安全边界；备选能等待更多外部证据但会把方法学风险继续悬置。 | participation ratio 仍不是 FWER/DSR 校正数；public C1 blocker 不解除。 | 出现 alpha/tail/FWER/DSR 需求转 methodology Spike；出现 public schema/producer 需求转独立 projection/activation CR。 |

### 风险与未决

| 风险 | 当前控制 | CP5 后触发 |
|---|---|---|
| 二阶模型偏差 | Claim 固定为相关结构有效维度；禁止 Li-Ji/FWER/DSR/admission 解释 | 需要多重检验校准时停止并转 Spike。 |
| Empirical matrix 尚无证据 | 仅 synthetic/fixture/golden；具体策略和真实 producer 均禁止 | 真实 activation 由 CR172 PATH-C/A 或后续 CR 单独授权。 |
| Public migration / dual truth | Public projection Feature/Story/Task=`0/0/0`，production diff/write=`0/0` | Future projection CR 必须处理 versioned schema、trust binding、migration 和 rollback。 |
| Exact integer growth | 当前 fixture 规模可控，不宣称 production SLA | 真实规模或性能目标出现时重新设计/压测。 |
| LLD authority pointer freshness | 已完成 pointer-only refresh，normative contract delta=`0` | CLOSED；后续仅允许实现已批准合同。 |
| 全局历史 checker debt | CR013 status 与 FEATURE-REGISTRY/schema 债务隔离 | 不得混入 CR173；需独立治理。 |

### 授权说明

回复 `approve` 或“批准 CP5”表示接受以上四项推荐，并授权以下最小范围：

- 创建或修改 `engine/effective_trial_evidence.py`、`engine/effective_trial_estimator.py`；
- 创建或修改 `tests/research/test_effective_trial_evidence_contracts.py`、`tests/research/test_effective_trial_estimator.py`、`tests/research/test_effective_trial_cr173_qac.py`、`tests/research/test_effective_trial_authorization.py`；
- 创建或修改 `tests/fixtures/effective_trial/golden_vectors_v1.json`；
- 通过 `uv run` 执行与上述 Story 直接相关的本地单元、QAC、授权静态检查和 12/12 既有只读回归；
- 按 `S01 → S02 → S03` 串行调度 meta-dev 和 meta-qa-critical，自动推进 CP6/CP7，全部通过后停在 CP8 人工门禁。

本批准**不授权**真实数据、具体 strategy identity、credential/provider、lake/NAS、public C1 projection/write、Gate1/DSR/admission、CR172 恢复、runtime/QMT/trading、publish/deploy 或 Git remote write。

### 用户需决策事项 summary

| 字段 | 内容 |
|---|---|
| 本轮待人工决策项 | `DQ-CP5-CR173-IMPLEMENTATION-CONTRACT`、`DQ-CP5-CR173-WAVES-OWNERSHIP`、`DQ-CP5-CR173-LOCAL-AUTHORIZATION`、`DQ-CP5-CR173-CLAIM-AND-VERIFICATION-BOUNDARY`。 |
| 推荐回复 | `approve CR-173 CP5，按 S01→S02→S03 串行实施并推进到下一个人工门禁`。 |
| Approve 效果 | 确认 3 份 full LLD，授权 7 个计划路径的 repository-local implementation 与 local `uv run` verification，并自动推进到 CP8 人工门禁或提前停止条件。 |
| 不表示授权 | 真实/外部/远端、public C1/Stage 3、FWER/DSR/admission、CR172 自动恢复或发布部署。 |

## Entry Criteria

| 条目 | 状态 | 证据 |
|---|---|---|
| CP3 approved | PASS | `process/checkpoints/CP3-CR173-HLD-REVIEW.md` |
| CP4 PASS | PASS | `process/checks/CP4-CR173-STORY-DAG-PARALLEL-SAFETY.result.json` |
| CP5 capsule ready | PASS | `process/context/CP5-CR173.context.json` |
| 3/3 LLD structure ready | PASS | `meta-flow story lld-check` 3/3 OK |
| Independent review required=0 | PASS | `process/checks/CP5-CR173-LLD-REVIEW-R3-SUMMARY.md` |
| Clarification=0 | PASS | CP5 capsule + QUESTION ledger |

## Checklist

| # | 检查项 | 待人工结果 | 证据 / 审查要点 |
|---:|---|---|---|
| 1 | 批准 S01 v1.3、S02 v1.3、S03 v1.4 三份 full LLD | approved | `DQ-CP5-CR173-IMPLEMENTATION-CONTRACT`；Round 3 blocking/required=`0/0`，pointer-only refresh delta=`0`。 |
| 2 | 批准 3-Wave serial DAG 与 7/7 primary owner | approved | `DQ-CP5-CR173-WAVES-OWNERSHIP`。 |
| 3 | 批准 7 个 repository-local 路径和本地 `uv run` 验证 | approved | `DQ-CP5-CR173-LOCAL-AUTHORIZATION`。 |
| 4 | 接受 fixture-only verification 与 estimator-only claim ceiling | approved | `DQ-CP5-CR173-CLAIM-AND-VERIFICATION-BOUNDARY`；exact integer growth 作为无 production SLA 的剩余风险接受。 |

## Exit Criteria

| 条目 | 当前状态 | 说明 |
|---|---|---|
| 全量设计证据 ready | PASS | 三份 LLD 均 approved/confirmed、open_items=0。 |
| Required review finding 归零 | PASS | Round 3 blocking/required=`0/0`。 |
| CP6 最小授权显式 | PASS | 仅 7 个计划路径与本地 `uv run`。 |
| 禁止项未被隐式授权 | PASS | Estimator-only / fixture-only / no-real / no-public-write / no-runtime。 |
| 人工决策已记录 | PASS | 用户已批准四项 DQ，并接受 fixture-only 的 exact integer growth 剩余风险。 |

## Deliverables

| 交付物 | 路径 | 状态 |
|---|---|---|
| CP4 result | `process/checks/CP4-CR173-STORY-DAG-PARALLEL-SAFETY.result.json` | PASS |
| CP5 context | `process/context/CP5-CR173.context.json` | ready / checks OK |
| CP5 auto result | `process/checks/CP5-CR173-LLD-DESIGN-EVIDENCE.result.json` | PASS / human approved |
| Round 3 independent review | `process/checks/CP5-CR173-LLD-REVIEW-R3-SUMMARY.md` | proceed / required 0 |
| S01 full LLD | `process/stories/STORY-CR173-S01-contract-evidence-canonicalization-LLD.md` | v1.3 / approved / confirmed |
| S02 full LLD | `process/stories/STORY-CR173-S02-exact-spectral-estimator-LLD.md` | v1.3 / approved / confirmed |
| S03 full LLD | `process/stories/STORY-CR173-S03-golden-failure-boundary-verification-LLD.md` | v1.4 / approved / confirmed |

## 人工审查结果

- 结论：`approved`
- 审查人：`user`
- 审查时间：`2026-07-17T09:33:11+08:00`
- 接受的决策 ID：`DQ-CP5-CR173-IMPLEMENTATION-CONTRACT`、`DQ-CP5-CR173-WAVES-OWNERSHIP`、`DQ-CP5-CR173-LOCAL-AUTHORIZATION`、`DQ-CP5-CR173-CLAIM-AND-VERIFICATION-BOUNDARY`
- 修改意见：批准前完成三份 LLD authority pointer-only refresh；不得借机改变已审查合同，实际 normative contract delta=`0`。
- 风险接受项：接受 exact-rational 在当前 synthetic/fixture（当前 golden 最大 `n=4`）上的 `O(n³)` time / `O(n²)` space 与整数增长风险；不宣称 production SLA，CR-172 PATH-C/A 或出现真实规模/性能目标前必须重新评估与压测。

CP5 已批准并解锁 S01 CP6。S02/S03 仍受串行依赖门控；禁止范围和 claim ceiling 不变。

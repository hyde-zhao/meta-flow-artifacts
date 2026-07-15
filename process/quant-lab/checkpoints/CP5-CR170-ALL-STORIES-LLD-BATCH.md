---
checkpoint_id: "CP5-CR170-ALL-STORIES-LLD-BATCH"
checkpoint_name: "CR170 All Stories LLD Design Evidence"
type: "batch_auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-07-15T14:47:00+08:00"
reviewed_by: "user"
reviewed_at: "2026-07-15T15:12:00+08:00"
auto_check_result: "process/checks/CP5-CR170-LLD-DESIGN-EVIDENCE.result.json"
decision_brief_profile: "compact"
target:
  phase: "story-planning"
  story_id: "CR170-S01..S04"
  artifacts:
    - "process/stories/STORY-CR170-S01-na-policy-inventory-five-state-contract-LLD.md"
    - "process/stories/STORY-CR170-S02-gate1-5-na-consumer-hardening-LLD.md"
    - "process/stories/STORY-CR170-S03-admission-worst-state-tier-hardening-LLD.md"
    - "process/stories/STORY-CR170-S04-compatibility-claim-closure-LLD.md"
---

# CP5 — CR170 全量 Story LLD 人工审查

## 自动预检摘要

| 预检 | 结论 | 阻断 | Waiver | 说明 |
|---|---|---:|---:|---|
| CP4 Story/DAG/Feature | PASS | 0 | 0 | FEAT-15 triplet 3/3；Story 4/4；4 serial Waves；6 typed edges；cycles/invalid refs/within-wave conflicts=0。 |
| CP5 Design Evidence | PASS | 0 | 0 | 4/4 full LLD structure OK；clarification=0；REQ 9/9、scenarios 20/20、QAC 15/15。 |
| 评审整改 | PASS | 0 | 0 | 21-unit baseline path/direction/disposition、caller contract、T3 zero-diff 已全部进入 LLD。 |
| 全局 Story plan checker | inherited FAIL | 0 个 CR170 blocker | 0 | 仅既有 CR013-S01..S04 invalid status 与历史引用 warning；CR170 scoped plan 独立 PASS。 |
| 授权边界 | PASS | 0 | 0 | CP5 前 source/test implementation、新增测试执行、真实/外部/远端操作均为 0。 |

## Decision Brief

### 审批者摘要

| 字段 | 内容 |
|---|---|
| 本次确认服务的整体目标 | 批准四份 LLD，把 21-unit N/A policy、Gate1-5 consumer、protected merge/admission tier 和 compatibility/claim guard 固化为可实现、可验证、可回退的 repository-local 合同。 |
| 推荐动作 | 批准 S01-S04、四个串行 Wave 与文件 owner；只授权批准文件的 repository-local source/test 编辑及本地 `uv run` 验证。 |
| approve 后会发生什么 | 仍不拉起子 Agent；按 S01→S02→S03→S04 串行进入 CP6/CP7 自动闭环，并在 CP8、FAIL、NEEDS_REWORK、授权边界或 workflow-health 阈值停止。 |
| approve 不授权什么 | 不授权真实数据/evidence、credential/auth system、Stage3 runner、aggregate、CR155 promotion、adapter 删除/修改、runtime/trading、publish/deploy/tag/release 或 Git remote write。 |
| 不确认会阻塞什么 | 全部 source/test implementation、CP6、CP7 与 CP8 交付准备。 |

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule | `process/context/CP5-CR170-CANONICAL-RELIABILITY-DESIGN-EVIDENCE-CONTEXT.yaml` |
| 状态 | ready；capsule-first / compact；CP5 context check OK。 |
| read_profile | `compact`。 |
| must-read | CP3 approval、CP4 result、CR170 Feature Matrix section、DEVELOPMENT-PLAN CR170 block。 |
| 默认读取策略 | capsule-first；先读 must-read，四份完整 LLD 仅在人工设计确认或冲突审计时读取。 |
| read-if-needed | CR170 HLD/ADR、FEAT-15、四份 LLD。 |
| 不默认读取 | archive、discussions、CR168/169 全部 Story/LLD/测试报告、无关设计/质量报告。 |
| 全文档读取 | scoped DEVELOPMENT-PLAN、目标 FEAT-15 与四份 CR170 LLD；均有 `full_doc_read_reason` 和 READ-EXPANSION-LEDGER 事件。 |
| clarification | blocking=`0`；四份 LLD `open_items=0`。 |

### 全量设计摘要

| Story | 可实现边界 | 主要文件 | 核心验收 |
|---|---|---|---|
| S01 | private 21-unit inventory、five-state、boundary caller | `engine/reliability_na_policy.py` + unit tests | 21/21、6/6/1/5/3、5/5、15/5/1、synthesis=0。 |
| S02 | Gate1-5 local consumer 与双向回归 | canonical Gate module + existing Gate tests | Gate 5/5、stricter/widen/preserve 15/5/1、Gate1 3/3、escape PASS=0。 |
| S03 | protected merge 与 T0-T3 admission | same canonical module + new resolver tests | merge 1/1/diff0、T0-T3 4/4、T3 1/1/diff0。 |
| S04 | public/adapters/claim closure | new CR170 regression test only | public 100%、adapter 2/2、guard deletion=0、CR155/Stage3/aggregate/real-op elevation=0。 |

### 评审问题整改结论

| 评审问题 | 整改位置 | 结果 |
|---|---|---|
| 21 unit 未区分既有 escape 与新引入 N/A | S01 exact inventory + FEAT-15 v0.2 + S02 directional matrix | `15 stricter / 5 controlled-widening / 1 preserve`；三类回归方向分别定义。 |
| `n_a_boundaries/authorization_ref` caller 不明确 | S01 §5-8、CP5 capsule caller_contract | 当前 writer 仅 fixture/test；evaluator synthesis=0；auth ref 仅 opaque audit pointer。 |
| T3 “维持现状”未显式锁定 | S03 §4/6/8/10 | existing early-return `BLOCKED+NOT_AUTHORIZED` 1/1；production diff=0。 |
| S03 把 Gate 级描述成 mandatory | S03 LLD v1.1 §8.1 + FEAT-15 v0.3 | 已精确为 unit-level applicability；resolver 只消费 S02 形成的 Gate summary status。 |
| conditional not-applicable ref status/floor 未明确 | S02 LLD v1.1 §5-8/§10 | audit-only ref status=NR、mandatory claim=0、status_floor=None、Gate status elevation=0。 |
| 缺 public evidence→admission 端到端场景 | S04 LLD v1.1 §7/§8.4/§10-14 + TEST-PLAN v0.3 | 新增 1/1 public-callable fixture：Gate NR→merge NR→T1 BLOCKED，PASS=0。 |

### 关键实现合同

1. S01 private classifier 只接收 Gate-local `evidence_present/applicable` 与 caller boundary；它不替代既有 value/shape 校验、不合成授权。
2. S02 的 5 个受控放宽 unit 只允许 complete boundary 从历史 BLOCKED 到 Gate `NEEDS_REVIEW`；generic/incomplete 非 PASS，T1/T2 后续仍 BLOCKED。
3. G1-P06 complete N/A 固定 prohibited；trial count/value/provenance 不可被 reason 替代。
4. protected merge 先测后改：回归通过则 production change=0；T3 同样 zero-diff。
5. adapters 保持 defense-in-depth；CR170 只回归，不删/改生产 guard。

### Decision Collection Coverage

| 来源 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---:|---:|---:|---|
| CP3 accepted DQs / ADR | scanned | 4 | 0 | 已批准，作为不可放宽架构约束。 |
| CP4 result / scoped plan | scanned | 1 | 1 | 四个串行 Wave 与 shared file owner。 |
| 四份 LLD / CP5 result | scanned | 3 | 3 | batch implementation contract、local authorization、inline verifier risk。 |
| clarification queue | scanned | 0 | 0 | open blocking items=0。 |
| 外部/真实/远端权限 | N/A | 0 | 0 | 明确禁止，不可由 CP5 隐式升级。 |

### 决策分层

| 分类 | 数量 | 处理 |
|---|---:|---|
| 必须用户决策 | 4 | 下表四项 DQ。 |
| 高风险策略确认 | 2 | local implementation authorization；inline verifier 风险。 |
| agent 默认处理 | 3 类 | private helper 名称、test fixture 具体值、非公共测试辅助函数；不得改变 21-unit/15-5-1/public/tier/owner 合同。 |
| 仅审计记录 | 12 类 | 9/9、20/20、15/15、21/21、5/5、15/5/1、4 Story、4 Wave、4 LLD、merge/T3 diff0、adapter 2/2、forbidden ops0。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| `DQ-CP5-CR170-IMPLEMENTATION-CONTRACT` | implementation | 是否批准四份 full LLD 作为同一实现批次？ | 批准 S01-S04。 | 指定 Story 修改并保持门禁。 | 批量批准保持跨 Story contract 一致；逐项修改更细但阻断依赖链。 | 解锁 CP6；未确认的合同变化会造成安全/返工风险。 | 实现细节变化回 CP5；架构/范围变化回 CP3/CP2。 |
| `DQ-CP5-CR170-WAVES-OWNERSHIP` | implementation | 是否批准四个串行 Wave 与 canonical 文件 owner？ | S01→S02→S03→S04；S03 在 S02 后合并 shared canonical file。 | 调整 DAG/owner 后重跑 CP4。 | 串行消除同文件冲突，代价是吞吐较低。 | 决定 merge 顺序和单写边界。 | 越 owner 或依赖变化立即停止回 CP4。 |
| `DQ-CP5-CR170-LOCAL-AUTHORIZATION` | security | 是否授权批准文件的 repository-local source/test 编辑与 `uv run` 验证？ | 只授权 LLD 文件表和本地验证；全部真实/外部/远端权限保持禁止。 | 保持 design-only；额外权限另开人工门。 | 推荐是完成 hardening 的最小权限；design-only 无法交付，扩权不必要。 | 解锁 CP6/CP7，但不产生真实 evidence、Stage3 或 runtime 能力。 | 任何真实 data/runtime/auth/remote 需求立即停止并申请新授权。 |
| `DQ-CP5-CR170-VERIFIER-RISK` | risk_acceptance | 是否接受当前不拉子 Agent下的 maintainer inline verification？ | fixture/static hardening 作为非阻断例外；CP8 强制披露，FU006 保留。 | 等待 FU006 独立 verifier 后再实现。 | 推荐遵从 no-subagent 指令并透明披露；备选独立性更强但会停止交付。 | 影响验证独立性，不降低自动断言严格度。 | 需要独立主观判断、出现高风险冲突或反复 rework 时停止申请独立 lane。 |

### 风险与未决

| 风险 | 当前控制 | CP5 后触发 |
|---|---|---|
| 双向 blast radius | 15/5/1 三组回归；widening Gate PASS=0、T1/T2 BLOCKED | 任一方向计数或结果漂移→NEEDS_REWORK。 |
| protected merge/T3 被误重构 | 1/1 regression + production diff=0 | diff 非零→NEEDS_DESIGN_CLARIFICATION。 |
| verifier independence | maintainer inline + CP8 disclosure；FU006 future | 需要独立判断→停止请求新授权。 |
| current runner 不消费 canonical | `stage3_entry_ready=false` | Stage3 接线/授权由独立 CR。 |

### 授权说明

回复 `approve` 只接受以上四项推荐方案，并授权批准文件的 repository-local source/test 编辑与本地 `uv run` 验证。它不授权子 Agent、真实数据或 evidence、credential/auth system、Stage3 runner、aggregate、CR155 promotion、adapter 删除/生产修改、runtime/trading、发布或 Git remote write。

### 用户需决策事项 summary

| 字段 | 内容 |
|---|---|
| 本轮待人工决策项 | `DQ-CP5-CR170-IMPLEMENTATION-CONTRACT`、`DQ-CP5-CR170-WAVES-OWNERSHIP`、`DQ-CP5-CR170-LOCAL-AUTHORIZATION`、`DQ-CP5-CR170-VERIFIER-RISK`。 |
| 推荐回复 | `approve CR-170 CP5，按已批准的 4 个串行 Story 实施并推进到下一个人工门禁`。 |
| approve 效果 | 确认 4 份 LLD/4 个串行 Wave，并只授权批准文件的 repository-local implementation 与 local `uv run` 验证。 |
| 不表示授权 | 子 Agent、真实/外部/远端操作、Stage3、aggregate、CR155 promotion、adapter 生产修改、发布或部署。 |

## Entry Criteria

| 条目 | 状态 | 证据 |
|---|---|---|
| CP3 approved | PASS | CP3 checkpoint + gate ledger |
| CP4 PASS | PASS | `process/checks/CP4-CR170-STORY-DAG-PARALLEL-SAFETY.result.json` |
| CP5 capsule ready | PASS | `process/context/CP5-CR170-CANONICAL-RELIABILITY-DESIGN-EVIDENCE-CONTEXT.yaml` |
| 4/4 LLD structure ready | PASS | `meta-flow story lld-check` 4/4 OK |
| clarification=0 | PASS | CP5 capsule + QUESTION ledger |

## Checklist

| # | 检查项 | 待人工结果 | 证据 / 审查要点 |
|---:|---|---|---|
| 1 | 批准四份 full LLD（含本轮三项 v1.1 补强） | approved | DQ implementation contract。 |
| 2 | 批准 4-Wave serial DAG 与 owner | approved | DQ waves/ownership。 |
| 3 | 批准 repo-local implementation/test 与 local uv validation | approved | DQ local authorization。 |
| 4 | 接受 inline verifier risk 与 CP8 disclosure | approved | DQ verifier risk。 |

## Exit Criteria

| 条目 | 当前状态 | 说明 |
|---|---|---|
| 全量设计证据 confirmed | PASS | 4/4 LLD 已回填 confirmed=true。 |
| CP6 authorization explicit | PASS | 仅 repository-local source/test 与本地 `uv run`。 |
| 禁止项未被隐式授权 | PASS | 本 gate 不扩大 CP3 范围。 |
| verifier disposition documented | PASS_WITH_RISK | CP8 强制披露。 |

## Deliverables

| 交付物 | 路径 | 状态 |
|---|---|---|
| CP4 result | `process/checks/CP4-CR170-STORY-DAG-PARALLEL-SAFETY.result.json` | PASS |
| CP5 context | `process/context/CP5-CR170-CANONICAL-RELIABILITY-DESIGN-EVIDENCE-CONTEXT.yaml` | ready |
| CP5 result | `process/checks/CP5-CR170-LLD-DESIGN-EVIDENCE.result.json` | PASS / human pending |
| Four LLDs | `process/stories/STORY-CR170-S0*-*-LLD.md` | ready-for-review / unconfirmed |

## 人工审查结果

- 结论：`approved`
- 审查人：用户
- 审查时间：2026-07-15T15:12:00+08:00
- 接受的决策 ID：`DQ-CP5-CR170-IMPLEMENTATION-CONTRACT`、`DQ-CP5-CR170-WAVES-OWNERSHIP`、`DQ-CP5-CR170-LOCAL-AUTHORIZATION`、`DQ-CP5-CR170-VERIFIER-RISK`
- 修改意见：先落实 S03 unit-level mandatory 表述、S02 conditional audit-only ref/no-floor 契约和 S04 public 端到端 fixture；已完成并通过 4/4 LLD 结构检查。
- 风险接受项：接受 fixture/static hardening 的 maintainer inline verification 非独立性风险；CP8 必须显式披露，FU-006 保持 future consumer。

用户已批准；按 post-approval route 连续推进 S01→S02→S03→S04 的 CP6/CP7，并在 CP8 人工门禁停止。

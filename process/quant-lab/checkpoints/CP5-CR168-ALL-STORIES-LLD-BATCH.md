---
checkpoint_id: "CP5-CR168-ALL-STORIES-LLD-BATCH"
checkpoint_name: "CR168 All Stories LLD Design Evidence"
type: "batch_auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-07-14T11:49:00+08:00"
reviewed_by: "user"
reviewed_at: "2026-07-14T12:17:56+08:00"
revision: 2
auto_check_result: "process/checks/CP5-CR168-LLD-DESIGN-EVIDENCE.result.json"
decision_brief_profile: "compact"
target:
  phase: "story-planning"
  story_id: "CR168-S01..S05"
  artifacts:
    - "process/stories/STORY-CR168-S01-c3-contract-identity-validation-LLD.md"
    - "process/stories/STORY-CR168-S02-deterministic-cost-producer-LLD.md"
    - "process/stories/STORY-CR168-S03-envelope-multi-strategy-compatibility-LLD.md"
    - "process/stories/STORY-CR168-S04-gate4-projection-containment-LLD.md"
    - "process/stories/STORY-CR168-S05-fixture-static-verification-LLD.md"
---

# CP5 — CR168 全量 Story LLD 人工审查

## 自动预检摘要

| 预检 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| CP4 Story/DAG/Feature | PASS | 0 | 4/4 Feature、12/12 三件套、5/5 Story、5/5 serial Wave、10 edges、cycles=0、invalid refs=0、parallel conflicts=0。 |
| CP5 Design Evidence | PASS | 0 | 5/5 full LLD structure OK；clarification=0；9/9 REQ、17/17 场景、15/15 QAC、10/10 P0 fail-closed。 |
| CR168 scoped Feature trace | PASS | 0 | 5/5 Story 的 `feature_refs` 使用工具 canonical dotted IDs；`feature_reference_ids` 保留 FEAT-168/166 人类追溯标签。 |
| 全局 Story plan checker | inherited FAIL | 0 个 CR168 阻断 | 仅已有 CR013-S01..S04 非法 legacy status 与历史索引 warning；CR168 scoped plan/DAG/ref/Wave/file checks PASS，未改写无关对象。 |
| 全局 feature/capability registry check | inherited / unauthorized | 0 个 CR168 阻断 | 基线缺 CAPABILITY-REGISTRY 与全仓 module metadata。`feature trace` 工具曾短暂生成 FEATURE-REGISTRY snapshot 以校验 CR168 card refs，已立即删除；无持久化 product/evidence/capability registry，未将治理债务伪装成 CR168 通过。 |
| CP5 定向整改复核 | PASS | 0 | A 跨 Story producer 编排、B1–B5 精确实现契约、C registry-missing N/A 测试归属、D S03 文档边界均已写回设计证据并经 5/5 LLD structure check 复核。 |

## Decision Brief

### 审批者摘要

| 字段 | 内容 |
|---|---|
| 本次确认服务的整体目标 | 确认五份 LLD 已把 CP3 批准的 C3 `economic_cost@v1`、identity 分域、Gate4 虚假 PASS 局部封闭和静态验证边界细化成可实现、可测试、可回退的 repository-local 合同。 |
| 推荐动作 | `approve`：统一批准 S01–S05、5 个串行安全 Wave、文件所有权，并只授权 repository-local source/test/synthetic fixture 实现与本地 `uv run` 验证。 |
| approve 后会发生什么 | Host Orchestrator 继续不拉起子 Agent，按 S01→S02→S03→S04→S05 串行实现；完成 CP6/CP7 自动检查后，将在 CP8 人工门禁、FAIL、NEEDS_REWORK、授权边界或健康阈值停止。 |
| approve 不授权什么 | 不授权真实数据、凭据、provider/NAS/lake、真实 TCA/calibration/capacity、C4/event producer、canonical Gate4/aggregate 修改、runtime/broker/QMT/simulation/paper/live/trading、catalog/store/registry、deploy/publish/tag/release/Git remote write、Stage3 启动或 CR155 promotion。 |
| 不确认会阻塞什么 | 所有 source/test/fixture 实现、CP6、CP7 与 CP8 交付准备。 |

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule | `process/context/CP5-CR168-ECONOMIC-COST-IMPACT-EVIDENCE-CONTEXT.yaml` |
| 状态 | `ready`；`cp5-context-check` 必须为 OK。 |
| read_profile | `compact` / capsule-first。 |
| 默认读取策略 | CP3 review、CP4 result、CR168 matrix、CR168 scoped plan；设计全文及五份 LLD 仅在冲突、人工审计或本次 design confirmation 时按需读取。 |
| 全文档读取 | `DEVELOPMENT-PLAN` 1 项 + LLD batch 1 项；原因均为 approved CP3 后的 Story truth-source / design-confirmation，不默认加载历史 archive。 |
| 设计证据 | 4/4 Feature、12/12 triplets、5/5 full LLD；open clarification=0。 |

### 全量设计摘要

| Story | 可实现边界 | 主要文件 | 核心验收 |
|---|---|---|---|
| S01 | C3 immutable contract、九族 normalization/validation、hash 分域 | `engine/economic_cost_evidence.py` | 9/9 字段族、10/10 fail-closed、identity 不污染 component hash。 |
| S02 | Decimal fee/tax/spread/slippage/static square-root calculation 与 producer | `engine/economic_cost_calculator.py`、`engine/economic_cost_evidence.py` | precision=28、raw sum 后统一量化、10→1 hash、real TCA=0。 |
| S03 | CR166 catalog 激活与 daily/ML multi-strategy attachment compatibility | `engine/strategy_evidence.py` | component 语义等价、envelope identity 不同、C1/C2 compatibility。 |
| S04 | 唯一 adapter-local C3→Gate4 containment | `engine/economic_cost_gate4_projection.py` | 4-key allowlist、8/8 denylist、C4 absent 无 reason、safe absent 非 PASS、canonical/aggregate=0。 |
| S05 | 两 fixture、QAC、权限、claim ceiling、CR155 regression 验证面 | `tests/fixtures/economic_cost/` 与 CR168 tests | 17/17 scenarios、15/15 QAC、10/10 P0、forbidden=0、CR155 promotion=0。 |

### 第二节核心发现的实施封闭

canonical Gate4 对 C4 refs 的 `absent + N/A reason` 可能不产生 missing blocked claim，形成虚假 PASS 逃逸。CR-168 不修改 canonical 逻辑；S04 在唯一 adapter 内实施以下局部闭环：

1. 只从 typed C3 component 重建 4 个 C3 字段：`impact_model_family`、`impact_model_ref`、`cost_underestimation_status`、`no_real_tca_claim`。
2. 以 key presence 拒绝精确 8 个 C4/generic N/A reason key；空值也不放行。
3. C4 三个 refs 只保持 absent，不附 reason；调用 canonical 时固定 `release_profile="candidate-release"`。
4. canonical 返回必须是非 PASS，且必须包含 3 个 C4 missing claims；异常 PASS 与非 PASS 的 postcondition 违约分为两个 reason code。
5. 测试只对 public canonical callable 使用 double；不运行时依赖私有 `_has_na_reason`；不调用 aggregate；canonical/aggregate source changes 均为 0。

### 本轮定向整改与审批依据（Revision 2）

| 评审项 | 处置 | 结论 |
|---|---|---|
| A：S01/S02 跨 Story 编排缺口 | `build_economic_cost_evidence` 固定为唯一 public entry：normalize→validate→issues short-circuit→calculate→produce；S01 result 定义为 normalized input、attachment context、ordered issues 的 typed 三元。 | 已闭环；issue 非空时 calculator calls=0。 |
| B1–B3：reason/basis/net return | Domain Map §4/§7、S01/S02 与 FEAT-168-01 三件套固定 N01..N10 精确 code、五个 basis 和 `net_return=quantized_net_pnl/performance_notional`。 | 已闭环；无隐式 basis、half-spread、FX 或 fallback。 |
| B4–B5：Gate4 claim/double | S04/FEAT-168-02 固定三个 canonical missing claim ID；double 使用 keyword-only public-callable DI，不 monkeypatch canonical module，也不依赖 private helper。 | 已闭环；安全 postcondition 可预测。 |
| C：capability registry N/A 测试归属 | S05 authorization suite 验证 registry missing → existing Feature/module refs 的 N/A-with-reason，persistent/parallel registry=0。 | 已闭环；不创建 registry。 |
| D：S03 既有 Feature 文档边界 | S03 仅可追加 revision/traceability；任何设计语义变更停止并回 CP3/新 CR。 | 已闭环；不越权改写 CR166 基线。 |

### 关键边界与剩余风险

| 项目 | 当前结论 | CP5 后处理 |
|---|---|---|
| Component vs envelope hash | A1 已在 CP3 冻结：九族校验；subject-neutral families 2–9 进入 component hash；identity/provenance/auth 进入 envelope hash。 | S01/S03 实现黄金测试，若改变域则回 CP3。 |
| Gate4 逃逸 | S04 以 adapter-local guard 封闭 CR168 path；canonical global hardening 不在当前 CR。 | 出现 adapter 外 direct caller、aggregate 计划或 canonical 修改需求时停止并路由 FU-007 / 独立 remediation CR。 |
| C4 | typed unavailable / absent-only；C4 calculator=0。 | FU-005 决定 C4 producer 与 adapter 演进；本 CR 不预设计 C4 路径。 |
| Model truth | static square-root 是 approximation，不是真实 TCA 或校准 impact。 | 新 family、formula/rounding/rebate 必须新 schema + 独立 method CR。 |
| Global workflow debt | CR013 legacy statuses、registry module metadata / capability registry 缺失；本轮 trace tool 的临时 FEATURE-REGISTRY 已删除。 | 已显式记录但不越权修复；CP7 对运行时失败逐项归因，不得 blanket waive。 |
| Verifier independence | 用户禁止子 Agent，当前为 inline fallback。 | 若 CP7 仍 inline，CP8 必须披露 `R-CR168-VERIFIER-INDEPENDENCE`；本 gate 请求用户确认该处理。 |

### Decision Collection Coverage

| 来源 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---:|---:|---:|---|
| CP3 accepted decisions / ADR | scanned | 5 | 0 | 已批准，作为不可放宽约束。 |
| CP4 result / scoped plan | scanned | 1 | 1 | 5 serial Waves 与 file owner。 |
| CP5 result / five LLDs | scanned | 3 | 3 | LLD batch、local implementation authority、verifier-independence risk。 |
| clarification queue | scanned | 0 | 0 | 五份 LLD 的 open items=0。 |
| runtime/real-data / registry authority | N/A | 0 | 0 | 明确禁止，不能由 CP5 隐式升级。 |

### 决策分层

| 分类 | 数量 | 处理方式 |
|---|---:|---|
| 必须用户决策 | 4 | 进入下方待人工决策清单。 |
| 高风险策略确认 | 2 | DQ-003 约束实现权限；DQ-004 约束独立验证风险接受。 |
| agent 默认处理 | 4 类 | helper 私有命名、小型非公共重构、测试 fixture 精确数值和格式化；不得放宽 CP3 合同。 |
| 仅审计记录 | 10 类 | 9/9 REQ、17/17 scenarios、15/15 QAC、10/10 P0、4 Features、12 triplets、5 Stories、5 Waves、5 LLD、8/8 denylist、0 external ops。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|
| `DQ-CP5-CR168-001` | implementation | 是否批准五份 full LLD 作为一个批次？ | 批准 S01–S05。 | 指定 Story 修改；暂停。 | 推荐避免已冻结的跨 Story 合同被局部绕过；备选更细粒度但会阻塞整批。 | 解锁 CP6；公共合同变化若未审查会造成返工。 | 公共合同变化回 CP5；架构/范围变化回 CP3/CP2。 |
| `DQ-CP5-CR168-002` | implementation | 是否批准 5-Wave 串行 DAG 与 file owner？ | S01→S02→S03→S04→S05，一 Wave 一 Story。 | 修改依赖/owner 后重跑 CP4；暂停。 | 串行减少共享 C3 contract、envelope 与安全 adapter 的并发冲突；代价是吞吐降低。 | 决定执行顺序与单写边界。 | 实际影响越出 owner 或依赖变化，停止并回 CP4。 |
| `DQ-CP5-CR168-003` | security | 是否授权 repo-local source/test/synthetic fixture 和本地 `uv run` 验证？ | 仅上述本地动作；所有外部/registry/Stage3 禁止保持。 | 保持 design-only；任何额外权限另立门禁。 | 推荐以最小权限交付静态 foundation；design-only 无法完成 CP6/CP7，而扩权不需要。 | 解锁 CP6/CP7，但不产生真实证据或运行能力。 | 任何 real data/runtime/external/registry 需求立即 BLOCKED 并请求新授权。 |
| `DQ-CP5-CR168-004` | risk_acceptance | 是否接受 no-subagent 下 CP7 可继续 inline，且 CP8 必须披露 verifier-independence？ | 接受该非阻断风险；CP7 仍需按分层证据执行。 | CP5 保持 design-only，待未来独立 verifier 获单独授权。 | 推荐遵循用户 no-subagent 指令并透明披露；备选独立性更强但需新授权并停止交付。 | 能继续本地实现，但独立性低于单独 QA lane。 | CP7 发现需要独立判断或高风险冲突时，停止并请求独立验证授权。 |

### 授权说明

回复 `approve` 仅表示接受以上四项推荐方案，并授权 repository-local 源码/测试/合成 fixture 编辑及本地 `uv run` 验证。它不授权子 Agent、真实数据、凭据、外部系统、真实 TCA/calibration/capacity、C4/event、canonical Gate4/aggregate、registry/write、runtime/trading、publish/deploy/Git remote write、Stage3 或 CR155 promotion。

### 用户需决策事项 summary

| 字段 | 内容 |
|---|---|
| 本轮待人工决策项 | `DQ-CP5-CR168-001`、`DQ-CP5-CR168-002`、`DQ-CP5-CR168-003`、`DQ-CP5-CR168-004`。 |
| 推荐回复 | `approve`。 |
| approve 效果 | 仅解锁 CP6/CP7 repository-local source/test/synthetic fixture 工作；仍自动停在 CP8 或其它明确停止条件。 |
| 不表示授权 | 真实数据/TCA/C4/canonical/aggregate、registry、runtime/trading、Stage3、CR155 promotion、发布或远端写入。 |

## Entry Criteria

| 条目 | 状态 | 证据 |
|---|---|---|
| CP3 approved | PASS | CP3 checkpoint + gate ledger |
| CP4 PASS | PASS | `process/checks/CP4-CR168-STORY-DAG-PARALLEL-SAFETY.result.json` |
| CP5 capsule ready | PASS | `process/context/CP5-CR168-ECONOMIC-COST-IMPACT-EVIDENCE-CONTEXT.yaml` |
| 5/5 LLD ready | PASS | structure checks=5/5 OK |
| clarification=0 | PASS | CP5 capsule + each LLD |

## Checklist

| # | 检查项 | 审查结果 | 证据 / 审查意见 |
|---:|---|---|---|
| 1 | 批准五份 full LLD | 通过 | `DQ-CP5-CR168-001`；用户指令“整改后批准”。 |
| 2 | 批准 5-Wave 串行 DAG 与文件 owner | 通过 | `DQ-CP5-CR168-002`；维持 S01→S02→S03→S04→S05。 |
| 3 | 批准 repo-local source/test/synthetic fixture 与本地验证授权 | 通过 | `DQ-CP5-CR168-003`；仍不授权任何外部/registry/runtime 边界。 |
| 4 | 接受 inline verifier-independence 风险及 CP8 披露义务 | 通过 | `DQ-CP5-CR168-004`；若 CP7 仍 inline，CP8 必须披露。 |

## Exit Criteria

| 条目 | 审查结果 | 说明 |
|---|---|---|
| 全量设计证据 confirmed | PASS | 五份 LLD 作为一个设计证据批次，Revision 2 已吸收本轮评审。 |
| CP6 authorization explicit | PASS | 仅 repository-local source/test/synthetic fixture。 |
| 禁止项未被隐式授权 | PASS | 不得由本次批准扩张范围。 |
| verifier independence disposition documented | PASS | 若 CP7 仍 inline，CP8 披露为强制前置义务。 |

## Deliverables

| 交付物 | 路径 | 状态 |
|---|---|---|
| CP4 result | `process/checks/CP4-CR168-STORY-DAG-PARALLEL-SAFETY.result.json` | PASS |
| CP5 context | `process/context/CP5-CR168-ECONOMIC-COST-IMPACT-EVIDENCE-CONTEXT.yaml` | ready |
| CP5 result | `process/checks/CP5-CR168-LLD-DESIGN-EVIDENCE.result.json` | PASS / 待人工确认 |
| Five LLDs | `process/stories/STORY-CR168-S0*-*-LLD.md` | ready-for-review |

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-07-14T12:17:56+08:00
- 接受的决策 ID：`DQ-CP5-CR168-001`、`DQ-CP5-CR168-002`、`DQ-CP5-CR168-003`、`DQ-CP5-CR168-004`。
- 修改意见：用户提交 A–D 定向评审；整改完成后以“整改后批准，继续推进到下一个人工门禁”批准。Revision 2 的具体闭环见本文件“本轮定向整改与审批依据”。
- 风险接受项：接受无子 Agent 的 inline verifier-independence 非阻断风险；CP8 必须披露 `R-CR168-VERIFIER-INDEPENDENCE`。

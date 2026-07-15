---
checkpoint: CP2
cr_id: CR-169
title: "C4 Capacity / Liquidity / ADV Evidence Producer Foundation — 需求、场景与范围基线"
status: approved
gate_profile: architecture-major
automatic_precheck: PASS
context_ref: process/context/CP2-CR169-REQUIREMENT-CONTEXT.yaml
result_ref: process/checks/CP2-CR169-REQUIREMENTS-BASELINE-PRECHECK.result.json
decision_brief_profile: compact
opened_at: "2026-07-14T17:28:00+08:00"
opened_by: host-orchestrator
revision: 2
reviewed_by: user
reviewed_at: "2026-07-14T17:45:00+08:00"
approval_source: "user-conditional-approval-after-remediation"
---

# CP2 — CR169 需求 / 场景 / 范围基线人工门禁

自动预检结论：`PASS`；评审整改后人工审查结论：`approved`。本次批准只解锁 **CP3 solution-design**；不得直接进入 Story、LLD、实现或验证。

## Decision Brief

### 审批者摘要

| 字段 | 内容 |
|---|---|
| 本次确认的整体目标 | 在 Stage 2 已记录完成、但 `stage3_entry_ready=false` 且 Stage 3 未启动的边界内，建立 fixture/static-only 的 C4 Capacity / Liquidity / ADV typed evidence foundation，并定义安全的 C3+C4 Gate 4 fixture compatibility。 |
| 推荐动作 | `approve`：接受下方 5 项推荐方案及三项评审整改，只进入 CP3 冻结方法、关联头、adapter 设计和 alpha-decay 归属。 |
| approve 后会发生什么 | 回填 CP2/gate ledger，生成 capsule-first 的 CP3 handoff；随后仅进行 HLD/ADR 与架构决策，不实现任何代码。CP8 / formal Stage 2 exit 前必须生成 `STAGE2-EXIT-VERIFICATION.result.json` 并完成 7/7 合同核验。 |
| approve 不授权什么 | 不授权真实 ADV/liquidity、真实 capacity calibration、真实 alpha-decay、canonical Gate4/global hardening、aggregate、Stage3 entry/runtime/trading、publish 或 Git remote write；FU-007a/007b 仅为提案，不随本次批准启动。 |
| 不确认会阻塞什么 | CR169 保持 active/not_ready；HLD/CP3、Story、LLD、实现和验证继续全部阻塞。 |

### 产品范围摘要

In Scope：

1. 将 `capacity_liquidity@reserved` 演进为 `1` 个 active C4 typed component / `1` 个 schema，并输出 Gate 4 消费的 `3/3` C4 refs。
2. 只使用显式 synthetic/static ADV/reference、participation、capacity curve 与 liquidity sizing 输入，形成可审计、确定性的 C4 proxy、availability、reason、lineage 和 canonical hash。
3. 冻结 C3/C4 最小 correlation header；它用于组合校验，不自动成为 component semantic hash 域。
4. 新增一个 **独立** strict C3+C4 fixture compatibility adapter，只组合已验证的 C3/C4 components，构造精确七字段 payload，并对 canonical 结果实施 fail-closed postcondition。
5. 2/2 fixture 族、12/12 P0 fail-closed、10 次→1 hash、CR168 C3-only absent-C4 回归、CR155 blocked regression。

Out of Scope：

- 真实 ADV/liquidity/quote/order/book/flow、真实 capacity calibration、真实 alpha-decay 或真实数据回填。
- 修改 `engine/economic_cost_gate4_projection.py` 的 CR168 C3-only adapter；它必须保持 C4 unavailable fail-closed。
- 修改 canonical Gate 4、`StrategyAdmissionPackage` 或 C1-C4 aggregate orchestration；这些仍归 `FU-CR161-007`。
- Stage 3、runtime、broker/QMT、simulation/paper/live/trading、catalog/store/registry/lake pointer 写入、publish/deploy/tag/release/Git remote write。
- CR155 promotion/unblock；必须保持 `BLOCKED` 与 `paper_candidate=false`。

### 场景、量化标准与关键安全语义

| 项目 | 当前值 | 说明 |
|---|---:|---|
| 新增 Use Case / Requirements / QAC | 1 / 9 / 15 | 全部为增量基线，旧文档保留 revision trace。 |
| CR169 scenarios / matrix coverage | 17 / 17/17 | P0=`16`、P1=`1`；没有 Story/implementation evidence。 |
| C4 typed component / active schema | 1 / 1 | `capacity_liquidity@v1`。 |
| Gate 4 C4 refs | 3/3 | `adv_participation_ref`、`capacity_dollars_ref`、`liquidity_sizing_refs`。 |
| C4 P0 fail-closed | 12/12 | 缺失、numeric、correlation、lineage、tamper、injection、权限。 |
| determinism | 10 次 → 1 hash | 对同一规范化 static C4 input。 |
| joint Gate4 fixture contract PASS | 1 | 只代表 fixture compatibility，不代表 aggregate/real capacity readiness。 |
| aggregate/capacity admission PASS、canonical/aggregate 修改、CR155 promotion | 0 / 0/0 / 0 | 不可因 C4 fixture 产生晋级或全局语义改变。 |
| alpha-decay calculator | 0 | CP3 决定归属前保持 0。 |
| Stage 3 entry readiness | false | Stage 2 complete 不蕴含可进入 Stage 3；其前置在下方整改记录中明确。 |

**关键整改已吸收：** CR-168 C3-only adapter 是“C4 absent 必须 fail-closed”的安全层，不能扩展为 C4-present 路径。CR-169 仅新建窄域 joint adapter；它不修复 canonical Gate4 的全局 N/A 语义，也不允许任意 caller flat mapping 或 reason escape。有效 C3+C4 static fixture 的 PASS 只能称为 `gate4_fixture_contract_pass`，不得称为 aggregate admission、capacity-scalable 或真实 capacity readiness。

### 本轮评审整改与批准回填

| Review ID | 结论 | 处置 | 状态 |
|---|---|---|---|
| RV-CR169-CP2-01 | `stage2_complete=true` 不能替代 Stage 2 exit 事实。 | CP8 / formal exit 前强制输出 `STAGE2-EXIT-VERIFICATION.result.json`，核验 `FactorSpec`、`FactorRunSpec`、factor panel、label window、evaluation、portfolio/risk、admission package/evidence index 共 `7/7`。 | resolved-as-CP8-obligation |
| RV-CR169-CP2-02 | Stage 2 完成不等于可进入 Stage 3。 | Claim ceiling 增加 `stage3_entry_ready=false`；Stage 3 仍需独立 canonical N/A hardening 决策、independent verifier 与 data/runtime authorization。 | resolved |
| RV-CR169-CP2-03 | FU-007 可考虑拆分，但不可借 CR169 静默启动。 | 仅登记 007a canonical N/A 语义硬化与 007b aggregate/CR155 regression 的非绑定提案；均需未来独立 CR/CP0/用户授权。 | recorded-not-started |

### 风险与未决问题

| 风险 | 当前控制 | 状态 |
|---|---|---|
| synthetic proxy 被误读为真实 ADV/capacity | no-real-data limitation、claim ceiling、真实能力计数=0 | OPEN，贯穿 CP7/CP8 |
| Stage 2 complete 被误读为 Stage 3 entry-ready | `stage3_entry_ready=false`、7/7 exit result、Stage 3 前置 follow-up | OPEN，CP8 / formal exit 前必须关闭为已核验或阻断 |
| C3/C4 header 过度耦合或污染 semantic hash | DQ correlation header；CP3 owner/equality/hash-domain obligation | OPEN，待 CP2/CP3 |
| joint adapter 绕过 CR168/canonical 边界 | 新 adapter 最小接口、exact 7 keys、denylist/allowlist/postcondition；canonical/aggregate 修改=0 | OPEN，待 CP2/CP3 |
| alpha-decay 被悄然塞入 C4 | DQ alpha；calculator=0 直到 CP3 明确 disposition | OPEN，待 CP2/CP3 |
| verifier 不独立 | `FU-CR161-006`；不阻塞 foundation，但 CP8 只能 READY_WITH_RISK 并披露 | OPEN-NONBLOCKING |
| C4 fixture 误提升 CR155 | CR155 BLOCKED/paper=false、promotion=0 regression | OPEN，贯穿 CP7/CP8 |
| capability registry 缺失 | existing feature/module refs + N/A-with-reason；独立治理 follow-up | ACCEPTED-NONBLOCKING |

### 待人工决策清单

| 决策 ID | 待确认问题 | 推荐方案 | 备选方案 | 影响与切换条件 |
|---|---|---|---|---|
| DQ-CP2-CR169-C4-METHOD | C4 fixture 方法、保守假设与 fail-closed 条件？ | 显式 synthetic/static ADV/reference、participation cap、capacity curve、liquidity sizing；不校准真实数据。 | 只冻结 schema，算术 proxy 延后。 | 推荐方案可验证可复算性；若 CP3 不能形成明确 numeric domain/limitations，则采用 schema-only 并重开 CP2。 |
| DQ-CP2-CR169-CORRELATION-HEADER | C3/C4 共享边界？ | 冻结最小 correlation header；C3/C4 计算 body 与 component semantic hash 域独立。 | 完全独立 header，后续承担迁移成本。 | 推荐方案支持安全组合且不预占计算语义；若 owner/equality table 无法明确，改独立 header 并重开 CP2。 |
| DQ-CP2-CR169-GATE4-COMPOSITION-OWNER | Gate4 兼容路径归谁？ | 新增 strict joint fixture adapter；CR168 adapter、canonical Gate4、aggregate 均不变。 | CR169 只输出 C4 component，联合 adapter 推迟 FU-007。 | 推荐方案能证明 C3+C4 consumer contract；若必须修改 canonical/aggregate 或无法阻断 flat/reason escape，则采用备选。 |
| DQ-CP2-CR169-ALPHA-DECAY | alpha-decay 归 C4、C2 还是独立 CR？ | CP3/meta-se 评估后决定；本 CR 默认不实现。 | 现在直接归 C4。 | 推荐方案避免预占 C2/OOS 语义；只有 CP3 证明其 input/output/risk 与 C4 不可分时才归 C4。 |
| DQ-CP2-CR169-VERIFIER | FU-006 是否为前置？ | 不阻塞 fixture foundation；若未完成，CP8 至多 READY_WITH_RISK 并强制披露。 | FU-006 前置。 | 推荐方案延续用户已批准 inline fallback；若 CP3/CP5 发现 adapter 验证无法充分独立，再切换为前置。 |

### 用户需决策事项

回复 `approve` 表示同时接受：`DQ-CP2-CR169-C4-METHOD`、`DQ-CP2-CR169-CORRELATION-HEADER`、`DQ-CP2-CR169-GATE4-COMPOSITION-OWNER`、`DQ-CP2-CR169-ALPHA-DECAY`、`DQ-CP2-CR169-VERIFIER`。

如需调整，请回复：`修改: <具体修改点>`。拒绝本轮请回复：`reject`。

## Entry Criteria

| 条目 | 状态 | 证据 |
|---|---|---|
| CP0 双仓基线 / 冲突预检 | PASS | `process/checks/CP0-CR169-BASELINE.result.json` |
| CP1 产品基线完整性 | PASS | `process/checks/CP1-CR169-USE-CASE-COMPLETENESS.result.json` |
| CP2 自动预检 | PASS | `process/checks/CP2-CR169-REQUIREMENTS-BASELINE-PRECHECK.result.json` |
| C4 Decision Brief / capsule | PASS | 本文与 `process/context/CP2-CR169-REQUIREMENT-CONTEXT.yaml` |

## Checklist

| # | 检查项 | 审查结果 | 证据 / 审查意见 |
|---:|---|---|---|
| 1 | 接受 fixture/static-only C4 foundation，不读取真实 ADV/liquidity | ☑ | DQ C4-METHOD |
| 2 | 接受 correlation header 仅作组合校验、计算/hash 域独立 | ☑ | DQ CORRELATION-HEADER |
| 3 | 接受 strict joint adapter，且 CR168/canonical/aggregate 不修改 | ☑ | DQ GATE4-COMPOSITION-OWNER |
| 4 | 接受 alpha-decay 留 CP3 disposition | ☑ | DQ ALPHA-DECAY |
| 5 | 接受 verifier 风险不阻塞、CP8 强制披露 | ☑ | DQ VERIFIER |
| 6 | 接受 9 requirements、15 QAC、17 scenarios、12/12 fail-closed 和 2/2 fixtures | ☑ | CP1 / CP2 自动结果 |
| 7 | 接受 `gate4_fixture_contract_pass` 不等于 aggregate/真实 capacity readiness | ☑ | claim ceiling |
| 8 | 确认 Stage3、真实数据/runtime、canonical/global、aggregate、CR155 promotion 仍未授权 | ☑ | 不授权项 |
| 9 | 接受 Stage 2 complete 不等于 Stage 3 entry-ready，CP8 / formal exit 必须 7/7 核验 | ☑ | RV-CR169-CP2-01 / 02 |
| 10 | 接受 FU-007 007a/007b 仅为后续提案，不启动、不修改 canonical/aggregate | ☑ | RV-CR169-CP2-03 |

## Exit Criteria

- 用户已在完成整改后 `approve` 五项 DQ 及两项非阻塞改进的落地方式。
- 本次 `approved` 仅允许进入 CP3 solution-design；其余未授权项持续生效。
- 若 `修改:`，更新基线、重新运行 CP1/CP2 自动预检并重新打开本门禁。

## Deliverables

- CP0：`process/checks/CP0-CR169-BASELINE.result.json`
- CP1：`process/checks/CP1-CR169-USE-CASE-COMPLETENESS.result.json`
- 产品证据：`process/checks/CR169-PRODUCT-BASELINE-EVIDENCE.json`
- CP2 capsule：`process/context/CP2-CR169-REQUIREMENT-CONTEXT.yaml`
- 本 checklist：`process/checkpoints/CP2-CR169-REQUIREMENTS-BASELINE.md`

## 人工审查结果

- 状态：`approved`
- 审批人：用户
- 时间：2026-07-14T17:45:00+08:00
- 已接受决策：`DQ-CP2-CR169-C4-METHOD`、`DQ-CP2-CR169-CORRELATION-HEADER`、`DQ-CP2-CR169-GATE4-COMPOSITION-OWNER`、`DQ-CP2-CR169-ALPHA-DECAY`、`DQ-CP2-CR169-VERIFIER`
- 备注：用户要求先落实 `stage3_entry_ready=false`、7/7 Stage 2 exit 核验义务及 FU-007 非绑定拆分提案，再批准 CP2；整改已完成且不扩大范围或授权。

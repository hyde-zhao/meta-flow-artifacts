---
status: "accepted-cp3-archived-baseline"
version: "0.2"
cr_id: "CR-166"
---

# Architecture Decisions：CR166 Walk-forward / OOS Evidence

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 0.1 | 2026-07-13 | host-orchestrator inline meta-se-critical | 提议 neutral primitives、静态 component catalog、fold/purge/embargo、availability/outcome、denominator、existing-consumer、event N/A 与授权决策。 |
| 0.2 | 2026-07-13 | host-orchestrator | CP3 人工批准 ADR-CR166-001..010；保持 design-only、zero-dereference 与 Stage ceiling。 |

## ADR-CR166-001 Method-neutral evidence primitives with C1 compatibility

- 状态：ACCEPTED（CP3，2026-07-13）。
- 决定：通用 canonical JSON、explicit-domain SHA-256 与 availability primitive 由 method-neutral evidence contract 拥有；C1 `statistical_evidence.py` 通过 wrapper/import/re-export 保持既有 public API、默认 domain、serialized bytes 与 golden hash 100% 不变。C2 不依赖 `StatisticalEvidenceInput`、`MethodEvidence` 或 C1 聚合语义。
- 备选：C2 直接导入 C1（错向依赖）；复制 canonical 实现（漂移）；重写 C1 hash（兼容破坏）。
- 回退：任何 C1 golden API/hash 不一致即不迁移旧实现，先保留 compatibility wrapper 并阻断 CP7。

## ADR-CR166-002 Stable envelope and static versioned component catalog

- 状态：ACCEPTED（CP3，2026-07-13）。
- 决定：`StrategyEvidenceEnvelope` 使用稳定 header，component 以 `component_type + component_schema_version + ref + hash + required` 描述。目录为静态显式表，不做动态 plugin discovery。
- 当前：`walk_forward_oos@v1` active；`economic_cost`、`capacity_liquidity` 仅 reserved，calculator 数各为 0。
- unknown decision：mandatory unknown→blocked；optional unknown 可原样保留审计，但不参与 outcome、mandatory evidence 或 PASS。
- 切换：第三方 component、多包发现或服务化成为已批准需求时另起 ADR 评审动态 registry/store。

## ADR-CR166-003 Explicit half-open temporal fold semantics

- 状态：ACCEPTED（CP3，2026-07-13）。
- 决定：时间边界为 normalized ISO-8601 date/timestamp 与半开区间 `[start,end)`。每个 common fold 必须满足：

```text
train_start < train_end
train_end <= validation_start < validation_end
validation_end <= oos_start < oos_end
```

- fold id 唯一，声明顺序稳定，cutoff 单调。允许 expanding/rolling training window；不得用简单“跨 fold 区间不重叠”规则误阻断合法 walk-forward。
- 禁止：仅检查字符串非空；从 current calendar/provider 推断边界；删除坏 fold 后重新编号或抬高分母。

## ADR-CR166-004 Explicit purge/embargo policy, no inferred sufficiency

- 状态：ACCEPTED（CP3，2026-07-13）。
- 决定：`PurgeEmbargoPolicy` 显式包含 unit、overlap applicability、label horizon、required/applied purge、required/applied embargo、policy ref。overlap applicable 时 purge 缺失或 applied<required 为 blocked；embargo one-below blocked、exact-boundary 才可继续。
- daily/ML adapter 只能映射已给事实；ML `label_horizon_days` 可约束 purge minimum，但 `MLPurgedEmbargoCVPolicy` 不能替代实际 fold boundaries。
- 不授权：真实交易日历、session resolver、label recomputation 或 event-window inference。

## ADR-CR166-005 Availability and outcome are separate

- 状态：ACCEPTED（CP3，2026-07-13）。
- 决定：availability=`present|typed_unavailable|not_applicable_with_reason|blocked`；只有 present 才有 outcome=`pass|fail|needs_review`。
- 缺失但无冲突→typed_unavailable；矛盾、非法、篡改、未授权→blocked；event 语义未冻结→not_applicable_with_reason。
- mandatory claim 保守顺序：`blocked > typed_unavailable > fail > needs_review > pass`；N/A 只表达适用性，不能替代 mandatory C2 PASS。

## ADR-CR166-006 Declared-fold denominator and metric recomputation

- 状态：ACCEPTED（CP3，2026-07-13）。
- 决定：metric policy 显式给出 mandatory metric ID、方向、threshold；producer 从 finite fold value 重算 metric/fold outcome。pass rate 使用 `passed_declared_folds / declared_fold_count`。
- missing/invalid fold 不能从分母剔除；否则会制造 survivorship-style fold selection。缺 metric、NaN、Inf 均 fail-closed，present component 中非有限值数量必须为 0。
- consumer threshold 仍由既有 gate 拥有；producer 只记录输入 policy 与结果，不创建 admission policy。

## ADR-CR166-007 Pure producer and three existing-consumer projections

- 状态：ACCEPTED（CP3，2026-07-13）。
- 决定：producer 无 I/O，只接受显式 values/refs，生成 immutable component 与 hash。三个 projection adapter 在 evidence validation 后调用：
  1. CR151 `WalkForwardValidationPlan` / pass-rate consumer；
  2. CR154 Gate 2 split/walk-forward/OOS/purge/embargo/leakage surface；
  3. `StrategyAdmissionPackage` evidence ref/availability/reason surface。
- 所有 projection 使用同一 component ref/hash/availability/reasons，采用 worse-state merge；不得创建新 gate、重算 raw fold 或提升 CR155 blocked 状态。

## ADR-CR166-008 Event compatibility is explicit N/A in CR166

- 状态：ACCEPTED（CP3，2026-07-13）。
- 决定：event-specific producer、fixture 和 calendar-fold adapter 数均为 0；applicability 输出 `not_applicable_with_reason:event_fold_semantics_unfrozen`，并记录 future owner/trigger。
- 理由：现有 `EventTimeSemantics` 只冻结 occurred/available/decision time，不定义 event window fold、事件重叠与 purge/embargo 计算。复用 daily folds 会制造假覆盖。
- 切换：未来 CR 冻结 event time vs calendar time、event window、available-at、overlap、purge/embargo 与 reference fixtures 后，可注册 event adapter。

## ADR-CR166-009 Zero dereference and deterministic local evidence

- 状态：ACCEPTED（CP3，2026-07-13）。
- 决定：ref 仅作为 opaque identity 校验；未授权 lake/NAS/provider/runtime ref 的 dereference count=0，operation counters 全为 0。canonical hash 使用显式 domain、JSON-safe finite values 与稳定 sequence normalization；同 fixture 10 次 distinct hash=1。
- tampered canonical field + old hash→blocked；semantically equivalent optional component ordering→same envelope hash。

## ADR-CR166-010 Design-only and Stage claim ceiling

- 状态：ACCEPTED（CP3，2026-07-13）。
- 决定：CP3 只批准五个 CP4 outcome 输入；CP3 前不形成正式 Story，CP5 前不修改源代码/测试。CR166 关闭后 Stage2 complete=true；Stage3 started/runtime-authorized/real-evidence-available=false。
- 不授权：真实 fold/OOS ingestion、历史重算、lake/NAS/provider/credential/external framework、runtime/broker/trading、catalog/store/registry、deploy/publish/Git remote write。

## Decision Register

| DQ | 对应 ADR | 推荐批准内容 | 备选 / 回退 |
|---|---|---|---|
| DQ-CP3-CR166-001 | 001/002 | neutral primitives + C1 compatibility + stable envelope/static catalog | C1 compatibility 不成立则暂停抽取；不采用 dynamic registry |
| DQ-CP3-CR166-002 | 003/004/005/006 | half-open fold、显式 purge/embargo、availability/outcome 分离、declared denominator | 真实 calendar/session 需求另起 Stage3 设计；不得隐式推断 |
| DQ-CP3-CR166-003 | 007/008 | 3 existing projections；event explicit N/A | event 语义独立冻结后新 CR；不创建平行 gate/空壳 |
| DQ-CP3-CR166-004 | 009/010 | deterministic zero-dereference、五个 CP4 输入、design-only 与 Stage ceiling | 修改设计或暂停；任何权限扩张必须独立人工授权 |

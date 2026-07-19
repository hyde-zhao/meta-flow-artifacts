---
status: accepted-cp3-lld-r2-remediation-synced-awaiting-r3-independent-review
version: "1.2"
change_id: CR-173
decision_scope: estimator-only
---

# Architecture Decisions：Effective-Trial Offline Methodology

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-16 | meta-se-critical | 提出方法、输入 owner、numeric/schema、public split 与 failure/recovery 五项重大 ADR。 |
| 1.1 | 2026-07-16 | meta-se-critical | CP5 Round-1 权威澄清：stable computation identity + 外置 attempt audit；F03/F04 唯一到达路径；public 新代码零集成与 12/12 只读回归分 lane。 |
| 1.2 | 2026-07-16 | meta-se-critical | CP5 Round-2 权威澄清：采用非 public deny-default operation classes `9/9` + public 双 lane 六项指标，冻结 counter owner 与禁止重复计数边界；刷新 S01/S02 LLD v1.2、S03 LLD v1.3 已同步 R2 整改并待 R3 复核的时点。 |

## ADR-CR173-001：采用 spectral participation ratio

- 状态：`ACCEPTED — CP3 approved 2026-07-16`
- 决策：method ID 固定为 `spectral_participation_ratio`；`n_eff=(tr R)^2/tr(R²)=n²/ΣRᵢⱼ²`。
- estimand：在 sealed-trial 二阶 correlation representation 下的独立维度等价量。
- 理由：给定有效 `R` 后单值可识别；PSD correlation matrix 下严格 `1≤n_eff≤n`；独立/全相关/singleton 边界可解析；无需随机积分或 eigenvalue 排序。
- 放弃：alpha-specific Gaussian-copula Šidák equivalent。它更接近 FWER，但需 tail/copula/alpha 与数值积分契约，当前输入不可识别。
- 不声明：FWER calibrated、DSR calibrated、tail-dependence equivalent、admission ready。
- 切换：用户目标变为 alpha/tail-specific，或二阶表示不被 methodology owner 接受时转 methodology Spike；期间保持 typed_unavailable。
- 后果：DO-CR173-CP3-001 在限定 claim 下 PASS；method hash 必须覆盖公式和数值合同。

## ADR-CR173-002：CR-173 不估计 correlation matrix

- 状态：`ACCEPTED — CP3 approved 2026-07-16；CP5 Round-1 clarified`
- 决策：estimator 只消费上游提供的、与 sealed ordered trial IDs 1:1 对齐的显式 correlation matrix。CR-173 v1 source mode 仅 `declared_exact` repository synthetic/fixture/golden-vector。
- 理由：在本 CR 估计矩阵会引入真实 returns、sampling error、stability、producer owner 与 runtime/data authorization，超出 PATH-B offline methodology。
- 输入有效域：n≥1、n×n、finite、symmetric、diag=1、entries∈[-1,1]、PSD、labels/hash一致。matrix token 是最多 12 位小数、无 exponent 的 canonical decimal，并按 exact base-10 rational 处理。
- PSD 判定：无 tolerance；按 canonical trial-ID 顺序执行 deterministic lexicographic symmetric-pivot、fraction-free `LDLᵀ` exact-rational 分解。负 pivot=非 PSD；零 pivot 存在非零 residual coupling=非 PSD。超过 12 位、需要 tolerance 或 empirical floating matrix=v1 unsupported/typed_unavailable，必须升级 method version 或转 Spike。
- 偏差边界：fixture exact 不外推真实无误差；future empirical matrix 必须证明 sampling/stability。若未来给定 Frobenius error `ε`，可用 HLD §4.3 的 perturbation bound；CR-173 不估真实 `ε`。
- 降级优先级：任一 non-canonical string token（含 `NaN`/`Inf`、exponent、负零等）在 exact parser 前唯一映射 F03=`unsupported_dependency_representation`；F04=`invalid_dependency_matrix_domain` 只处理所有 token 已成功解析为有限 exact rational 后的 shape/symmetry/diag/range/PSD 失败。empirical/real/unsupported representation=F03 unavailable；hash/label/tamper contradiction=`blocked`。
- 后果：真实 activation 前 `effective_trial_count_populatable=false` 仍可能保持，除非 future owner 提供合格矩阵证据。

## ADR-CR173-003：冻结七字段、数值范围与 canonical serialization

- 状态：`ACCEPTED — CP3 approved 2026-07-16；CP5 Round-1 clarified`
- 七个顶层字段：
  1. `effective_trial_count`
  2. `effective_trial_count_status`
  3. `effective_trial_method`
  4. `effective_trial_method_version`
  5. `effective_trial_method_hash`
  6. `effective_trial_input_lineage_ref`
  7. `effective_trial_computation_ref`
- status：结构化 `{state, reason_code}`，从而在不增加第八个顶层字段的情况下满足 machine-readable reason。
- count：present 时 JSON number。先以 exact rational 检查未舍入结果在 `[1,n]`；再量化到最多 12 位、round-half-even，并二次检查 `[1,n]`；任一失败=blocked，禁止 clamp。非 present 必须 null；禁止 exponent、NaN/Inf、负零、前导零与非必要尾零；整数只使用 `1` 而非 `1.0`。
- serialization：只在 evidence 边界舍入一次。numeric token 由 exact rational coefficient/scale 直接渲染，禁止 `Decimal→float`；canonical serializer 按排序键输出 UTF-8 JSON并原样嵌入 token。domain=`quant-lab.effective-trial-evidence.spectral-participation-ratio.v1`；method hash 覆盖 schema、公式、canonical grammar、F03/F04 mapping/precedence、exact-rational PSD、范围、精度、舍入、renderer、reason enum、`EffectiveTrialAttemptBasisV1` 与 stable computation-ref derivation。
- null/identity 规则：七个顶层键始终存在。unavailable/blocked 时 count=null；已验证的 method/input 元数据可以保留，不得合成。第七字段 `effective_trial_computation_ref` 是对 `quant-lab.effective-trial-attempt-basis.v1` canonical basis 内容寻址的 stable result identity，包括 F01-F08；同 basis/outcome 的 3/3 repeat 只能有 1 个 ref/evidence hash。
- attempt basis 固定包含 schema、validation stage、identity/envelope/method presence bitmap、restricted component snapshot digests/absent markers、validated refs/null markers、primary failure ID 与 outcome。verification run/case/ordinal/time/worker/random/audit ref 不进入 basis。
- 外置 `ComputationAttemptAudit`：schema owner=methodology owner，当前 lifecycle/write owner=S03 repository-local verification harness。它链接 `attempt_audit_ref`、verification run/case/ordinal、stable computation ref、evidence hash、state/reason、parent/supersedes 与 safe diagnostic codes；不进入七字段、method/evidence hash。audit ref 用 versioned audit domain + 显式 run/case/ordinal + computation/evidence + parent markers 内容寻址。
- persistence/retention：当前均 `N/A`，只在 repository-local verification evidence 的 immutable/in-memory append-only collection 中模拟；任何 production store/catalog/pointer/writer 必须独立 CR。
- 后果：current `FamilyEvidenceProjection.effective_trial_count: int|None` 与 4 aliases 不兼容，不能直接投影。

## ADR-CR173-004：Public C1 projection 拆为后续 CR

- 状态：`ACCEPTED — CP3 approved 2026-07-16；CP5 Round-1/2 clarified`
- 分类：`cross-owner + cross-domain + non-compatible`。
- 证据：
  - lineage dataclass/invariant/C1InputStatus 与 store serialization；
  - statistical gate shared consumer adapter；
  - StatisticalEvidenceSummary/project_summary 与 multiple-testing/DSR method semantics；
  - Gate-1 trial slot、forced effective blocker；
  - admission package worst-state/limitations/CR155 regression；
  - 生产路径 8/8，回归/authorization 路径 12/12。
- 决策：CR-173 只产 standalone seven-field evidence；不得修改 current public types/adapters，不解除 Gate1 blocker，不改善 admission status。
- 为什么不是 additive compatible：字段由 4→7，`effective_ref` 拆为两个 provenance refs，availability→structured status，count int→decimal number，且现有 invariant 禁止 positive truth。双写 legacy/new 会形成两个 truth source。
- future CR 必须：owner approval、versioned public schema、trust binding、old/new migration、无 dual truth、Gate1/DSR disposition、admission worst-state、8+12 全量回归、rollback。
- 回退：后续 CR 未批准时 current consumers永久保持 typed_unavailable；estimator standalone 可独立回滚。
- 计数分层：CR173 新代码必须满足 `cr173_new_code_public_dependency_edges=0`、`cr173_new_code_public_calls=0`、`public_production_diff=0`、`public_writes=0`；CP7 另行满足 `cp7_read_only_public_regression_inventory=12/12`、`existing_expected_edits=0`。既有 regression 的 current public 调用仅属于 read-only verification lane，不计入新代码 integration-call counter。
- 非 public operation-class 主选：NP-01..09 分别为 credential、real data、lake/NAS、provider/network、catalog/store/pointer、strategy runtime、QMT/trading、publish/deploy、Git remote，共 `9/9`，每类单一 owner counter=`0`。它们与前述 public 六项指标是两组正交 inventory：不共享 class ID、不复用 counter、不把 public read-only call 当作非 public operation，也不把九类 guard 命中当作 public integration metric。

## ADR-CR173-005：Failure semantics 与 append-only recovery

- 状态：`ACCEPTED — CP3 approved 2026-07-16；CP5 Round-1 clarified`
- 缺失/不足/unsupported：`typed_unavailable`。non-canonical token（含 `NaN`/`Inf`）只走 F03；有限 exact-rational matrix 的 shape/symmetry/diag/range/PSD failure 只走 F04。
- identity/hash/label contradiction、tamper、forged serialized positive truth：`blocked`。
- 任意失败：count=null、raw fallback=0、C1 PASS/available=0。
- 恢复：failure attempt A 的 stable computation ref/evidence hash 与外置 audit A 保留；修正输入或 method version 后 canonical basis 改变，创建 computation ref/evidence hash/audit B，B 的 parent/supersedes 指向 A。相同未修正 input 的重复执行保持同一 computation/evidence identity，只新增外置 audit ref；任何旧 audit 不原地覆盖。
- 公共 projection mismatch：当前 CR 视为 contract boundary stop，不生成 public evidence。

## 决策一致性与回退矩阵

| 触发 | ADR 结果 | HLD/Blueprint 对齐 | 回退 |
|---|---|---|---|
| sealed exact PSD matrix | ADR-001/002 允许 estimator | HLD §4/5；Blueprint CAP-01/02 | token grammar→F03；parsed domain→F04 |
| output present/failure | ADR-003 七字段/stable computation identity + external audit | Domain Map schema/audit | hash/version mismatch blocked；audit不改变 evidence hash |
| public C1 写入 | ADR-004 禁止本 CR | Dependency Map FD-04/05 | future CR |
| alpha/tail claim | ADR-001 不允许 | HLD risk/when-to-switch | methodology Spike |
| real matrix | ADR-002 不允许本 CR | Out of Scope/A01 | future activation |

## Gotchas

- ADR-001 的名称刻意不含 FWER、DSR、Sharpe 或 admission，避免方法名制造超范围承诺。
- ADR-002 不禁止未来 producer 估计矩阵；它只规定这不是 CR-173 estimator 的职责。
- ADR-003 的 12 位输出精度是 serialization contract，不是对真实估计精度的统计声明。
- `effective_trial_computation_ref` 与 `attempt_audit_ref` 不能互换：前者稳定且进入七字段，后者逐执行唯一、外置且不进入 evidence hash。
- `NaN`/`Inf` 不能为了获得 F04 而被 parser 接受；它们是 F03 grammar failure。
- ADR-004 的 split 不是“先双写再迁移”；后续 CR 在兼容与 owner 门禁完成前必须保持旧 consumers unavailable。
- “forbidden operation”只指非 public `9/9` inventory；public 四个 new-code zero、`12/12` read-only inventory 和 expected edits=0 必须独立报告，禁止把 public 项机械补成第十类或复用同一 probe。

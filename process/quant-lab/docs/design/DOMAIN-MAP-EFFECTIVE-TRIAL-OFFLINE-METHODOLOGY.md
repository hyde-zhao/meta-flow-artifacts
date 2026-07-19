---
status: approved-cp3-lld-r2-remediation-synced-awaiting-r3-independent-review
version: "1.2"
change_id: CR-173
---

# Domain Map：Effective-Trial Offline Methodology

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-16 | meta-se-critical | 冻结 estimator-only 领域对象、状态与规则；公共 C1 projection 标记为 deferred boundary。 |
| 1.1 | 2026-07-16 | meta-se-critical | 回写 CP5 Round-1 required findings：stable computation identity、外置 attempt audit、F03/F04 唯一分层与 public verification lane 计数。 |
| 1.2 | 2026-07-16 | meta-se-critical | 回写 CP5 Round-2 F-R2-002/003：冻结非 public deny-default operation inventory=`9/9`、public 双 lane 六项指标独立且不重复计数；记录 S01/S02 LLD v1.2、S03 LLD v1.3 已同步 R2 整改并待 R3 独立复核。 |

## 术语表

| Term | 定义 | 来源 | 备注 |
|---|---|---|---|
| raw trial count | sealed family 中被 lineage 验证的试验数量 `n` | REQ-CR173-001 | 不是 effective count 的默认值 |
| sealed-trial correlation matrix `R` | 与 ordered trial IDs 1:1 对齐、对称、单位对角、PSD 的显式二阶依赖矩阵 | REQ-CR173-002 | 由上游提供；CR-173 不估计 |
| spectral participation ratio | `n_eff=(tr R)^2/tr(R^2)=n^2/ΣᵢⱼRᵢⱼ²` | DO-CR173-CP3-001 | 等于谱有效秩；不声称 FWER/DSR 校准 |
| effective trial count | 在批准的二阶依赖表示下，试验族的独立维度等价量 | REQ-CR173-001 | canonical decimal，范围 `[1,n]` |
| typed_unavailable | 输入缺失、不足或不在 v1 有效域，不能形成 count | DQ-CR173-003 | count=null，永不回退 raw |
| blocked | 输入矛盾、篡改、hash/provenance mismatch 或不可信 serialized positive truth | DQ-CR173-003 | count=null，需要新证据版本恢复 |
| standalone evidence | estimator owner 生成的七字段 typed evidence | REQ-CR173-004 | 本 CR 不等于 public C1 projection |
| stable computation identity | 对 canonical attempt basis + outcome 内容寻址的唯一结果身份 | CP5 F-001 clarification | 写入第七字段；3/3 repeat 只产生 1 个 ref |
| computation attempt audit | 每次 repository-local verification execution 的外置 append-only 审计 | CP5 F-001 clarification | `attempt_audit_ref` 不进入七字段/evidence hash；当前 persistence/retention=N/A |
| public C1 contract | FamilyEvidenceProjection、StatisticalEvidenceSummary、Gate-1 与 admission consumers 的共享接口面 | DO-CR173-CP3-002 | 跨 owner，后续 CR 才可改变 |

## 领域对象

| Object ID | 对象 | Owner | 关键属性 | 状态 | 规则来源 |
|---|---|---|---|---|---|
| OBJ-ET-01 | SealedTrialIdentity | lineage owner | family_ref/hash、raw_trial_count、ordered_trial_ids | unavailable / present / blocked | REQ-CR173-001/005 |
| OBJ-ET-02 | DependencyMatrixEnvelope | methodology owner（fixture）；future producer（real） | schema/version、trial IDs、canonical matrix、input hash、lineage ref、source_mode | unvalidated / valid / unavailable / blocked | REQ-CR173-002/003 |
| OBJ-ET-03 | EffectiveTrialMethodSpec | methodology owner | method ID/version/hash、formula、numeric grammar、rounding、serialization domain | draft / approved / superseded | REQ-CR173-002/006 |
| OBJ-ET-04 | EffectiveTrialComputation | methodology owner | canonical attempt-basis hash、method marker、state/reason、unrounded/canonical result、stable content-addressed computation ref | present / unavailable / blocked | REQ-CR173-001/003/006；CP5 F-001 |
| OBJ-ET-05 | EffectiveTrialEvidence | methodology owner | 七个顶层字段 | present / typed_unavailable / blocked | REQ-CR173-004 |
| OBJ-ET-06 | PublicC1ProjectionContract | C1 evidence contract owner | legacy availability/count/ref/method、C1 status、consumer adapters | current-v1 / migration-required | REQ-CR173-007、DO-002 |
| OBJ-ET-07 | ComputationAttemptAudit | schema owner=methodology owner；当前 lifecycle/write owner=S03 repository-local verification harness | audit ref、verification run/case/ordinal、computation ref、evidence hash、state/reason、parent/supersedes、safe diagnostic codes | created / linked / superseded | REQ-CR173-003/006；CP5 F-001 |

## 输入合同与有效域

| 输入类别 | 必需性 | 有效域 | 无效 / 缺失行为 | 说明 |
|---|---|---|---|---|
| sealed family identity | required | ref/hash 非空；`n≥1`；trial IDs 唯一、canonical 排序、数量=n | 缺失=`typed_unavailable`；hash/数量矛盾=`blocked` | 只读输入 |
| dependency representation | required | v1 仅 `sealed_trial_correlation_matrix` | 其他 representation=`typed_unavailable` | pair list、cluster、estimated summary 均不隐式转换 |
| correlation matrix raw tokens | required | 普通 canonical decimal string，≤12 位，无 exponent/NaN/Inf/负零/前导或尾零 | 任一 non-canonical token 唯一 F03=`typed_unavailable`；不得进入 exact matrix domain validator | 原始 token tree 仅生成 restricted canonical snapshot digest；不从真实 returns 估计 |
| parsed exact-rational correlation matrix | required | token 已全部解析为有限 exact rational；n×n、对称、对角=1、元素∈[-1,1]、PSD、labels 与 trial IDs 完全一致 | shape/symmetry/diag/range/PSD 唯一 F04=`typed_unavailable`；label/hash 矛盾或篡改=`blocked` | F04 绝不包含 NaN/Inf/grammar failure |
| source mode | required | CR-173 仅 synthetic/fixture/golden-vector `declared_exact` | empirical/real=`typed_unavailable` 且越权计数=0 | 未来 activation 必须另证采样误差与稳定性 |
| method spec | required | ID/version/hash 三者与已批准 spec 一致 | 缺失=`typed_unavailable`；hash mismatch=`blocked` | 不接受默认 method |
| canonicalization config | required | decimal grammar、12 位小数、round-half-even、hash domain 全部版本化 | 缺失=`typed_unavailable`；mismatch=`blocked` | 保证 3/3 repeat 唯一 |

## 七字段 standalone evidence

| 顶层字段 | present | typed_unavailable / blocked | 规则 |
|---|---|---|---|
| `effective_trial_count` | canonical JSON number，`1≤x≤n`，最多 12 位小数 | `null` | 不得从 raw 默认补值 |
| `effective_trial_count_status` | `{state:"present", reason_code:"ok"}` | `{state:"typed_unavailable|blocked", reason_code:<enum>}` | machine reason 内嵌于 status，仍保持 7 个顶层字段 |
| `effective_trial_method` | `spectral_participation_ratio` | 已知批准方法时保留，否则 `null` | 不得写 FWER/DSR calibrated |
| `effective_trial_method_version` | 非空版本 | 已知时保留，否则 `null` | 与 method hash 联合校验 |
| `effective_trial_method_hash` | canonical spec hash | 已知时保留，否则 `null` | mismatch=blocked |
| `effective_trial_input_lineage_ref` | validation-bound ref | 可验证部分可保留，否则 `null` | orphan/mismatch=blocked |
| `effective_trial_computation_ref` | stable content-addressed computation identity | 失败也必须非空；同 canonical attempt basis + outcome 只产生 1 个 ref | 不是逐执行 attempt ID；修正输入改变 basis 后生成新 ref |

## Canonical attempt basis 与外置 audit

### EffectiveTrialAttemptBasisV1

| 字段 | canonical 值 | 约束 |
|---|---|---|
| `basis_schema` | `quant-lab.effective-trial-attempt-basis.v1` | 未知 schema 映射 F03 |
| `validation_stage` | construction / token_parse / method / integrity / matrix_domain / evidence | F03 只能在 token_parse 前后边界；F04 只能在 matrix_domain |
| `presence_bitmap` | identity/envelope/method 三个布尔位 | F01/F02/F05 必须用显式 absent，不省略键 |
| `component_snapshot_digests` | identity、dependency raw-token tree、method、attempted evidence 的 restricted canonical UTF-8 digest 或 absent marker | 只包含 repository fixture/static 内容；禁止时钟、随机数、strategy/真实数据 |
| `validated_refs` | input lineage ref、approved method hash 或 null marker | 只保留已验证片段，不合成 |
| `primary_failure_id` | none / F01..F08 | 使用唯一 precedence |
| `outcome` | state、reason code、count token/null | 与 evidence status 一致 |

`effective_trial_computation_ref` 使用 `quant-lab.effective-trial-computation.v1` domain 对整份 canonical basis 做 SHA-256 内容寻址。verification run、synthetic case、repeat/attempt ordinal、worker/time 和 `attempt_audit_ref` 不进入 basis，因此 3/3 repeat 的 computation ref/evidence hash 均为 1。

### ComputationAttemptAudit

| 字段 | 生成 / 链接规则 |
|---|---|
| `attempt_audit_ref` | `sha256(audit-domain + verification_run_ref + synthetic_case_id + attempt_ordinal + computation_ref + evidence_hash + parent/supersedes markers)` |
| `verification_run_ref` / `synthetic_case_id` / `attempt_ordinal` | 由 S03 repository-local verification harness 显式提供；同一 run/case ordinal 不得复用 |
| `effective_trial_computation_ref` / `canonical_evidence_hash` | 必须分别链接七字段 stable identity 与可重算 evidence hash |
| `state` / `reason_code` | 与 evidence 一致；F01-F08 都必须有 audit |
| `parent_attempt_audit_ref` / `supersedes_attempt_audit_ref` | recovery B 指向 A；A 不覆盖、不删除 |
| `diagnostic_codes` | 只含安全枚举，不复制 raw payload、真实数据或 secret；不进入 evidence hash |

该对象是外置第八对象，不是第八个 evidence 字段。当前 persistence=`N/A`、retention=`N/A`，只在 repository-local verification evidence 的 immutable/in-memory append-only collection 中模拟和断言；不创建 store/catalog/pointer/writer。未来持久化必须独立 CR。

## 状态机

| State Machine ID | 对象 | 合法转换 | 触发条件 | 非法转换处理 |
|---|---|---|---|---|
| SM-ET-01 | DependencyMatrixEnvelope | unvalidated→valid / typed_unavailable / blocked | 完成 schema、identity、矩阵域与 hash 检查 | 任意失败后直接标 present=blocked |
| SM-ET-02 | EffectiveTrialEvidence | attempt→present / typed_unavailable / blocked | input 与 method 均有效才 present | unavailable/blocked→present 原地覆盖禁止；必须新 computation ref |
| SM-ET-03 | MethodSpec | draft→approved→superseded | CP3/后续门禁确认；新版本替代 | 修改公式/precision 而不升级版本=blocked |
| SM-ET-04 | PublicC1ProjectionContract | current-v1→migration-required→future-versioned | 后续 CR owner approval + compatibility migration | CR-173 直接 current→positive projection 禁止 |
| SM-ET-05 | ComputationAttemptAudit | none→created→linked；可由新 audit supersede，旧记录保留 | 每次 verification execution 结束 | ref 重用、覆盖、删除、进入 evidence hash 均 blocked |

## 业务规则

| Rule ID | 规则 | Owner | 影响场景 | 验证入口 |
|---|---|---|---|---|
| RULE-ET-001 | `n_eff=n²/ΣRᵢⱼ²`；PSD correlation matrix 下 `1≤n_eff≤n` | methodology owner | P01/D01 | golden vectors |
| RULE-ET-002 | independent `R=I` 得 n；fully correlated `R=11ᵀ` 得 1；singleton 得 1 | methodology owner | D01 | 3/3 repeats |
| RULE-ET-003 | negative/positive dependency 都以平方幅度进入二阶有效维度；不解释 tail direction | methodology owner | P01 | method limitation review |
| RULE-ET-004 | v1 不估计 correlation matrix；empirical sampling error 未证明时不得 present | future activation owner | B01/A01 | authorization guard |
| RULE-ET-005 | 缺失/不足 unavailable，矛盾/篡改 blocked，任何失败 raw fallback=0；non-canonical token（含 NaN/Inf）只归 F03，F04 只归有限 exact-rational matrix domain | methodology owner | F01/N01 | negative fixtures |
| RULE-ET-006 | 七个顶层字段始终存在；只有 present 允许非 null count | methodology owner | N01 | schema validation |
| RULE-ET-007 | public projection 在 CR-173 为 0；current consumers 保持 typed_unavailable | C1 owner | C01 | inventory + regression |
| RULE-ET-008 | CR-173 CP8 不触发 CR-172、Stage 3、admission、aggregate 或 runtime | host-orchestrator | A01 | claim-ceiling checks |
| RULE-ET-009 | 同 canonical basis/outcome 的 computation ref/evidence hash 稳定；每次执行另建外置 audit ref | methodology owner + S03 verification harness | D01/F01 | 3/3 + recovery |
| RULE-ET-010 | 新代码 public dependency/call/diff/write 均为 0；12/12 existing public calls 只属 read-only CP7 lane，existing expected edits=0 | verification owner | C01/A01 | static + selected regression |
| RULE-ET-011 | 非 public deny-default operation class 固定为 NP-01..09：credential、real data、lake/NAS、provider/network、catalog/store/pointer、strategy runtime、QMT/trading、publish/deploy、Git remote 共 `9/9`，各 counter=0；每个 guard/probe 只归一个 class，且九类不与 public 六项指标重复 | verification owner | B01/C01/A01 | authorization inventory + unique-class assertion |

## Gotchas

- status 是结构化字段，reason code 不得被挤进 method/ref 字符串。
- exact fixture matrix 的“无采样误差”只描述测试输入，不可外推到真实研究矩阵。
- `FamilyEvidenceProjection.effective_trial_count: int|None` 与本方法可能产生小数，是明确的公共 contract 非兼容证据。
- append-only recovery 作用于外置 audit：同一未修正 basis 重复时 computation ref/evidence hash 不变而 audit ref 递增；修正输入后新 basis 形成新 computation ref/evidence hash/audit，旧 failure audit 保留。
- `NaN`/`Inf` 是字符串 grammar failure，只能 F03；任何测试把它送入 F04 matrix domain validator 都违反 v1 contract。
- existing 12/12 regression 可以调用既有 public surface，但不得被计入 CR173 新代码 public-call counter，也不得修改 expected。
- `9/9` 是非 public operation-class inventory，不是 public path inventory。public 四个 new-code zero、`12/12` read-only inventory 与 expected edits=0 必须单列；任何 operation probe、public metric 或回归调用不得跨两组重复计数。

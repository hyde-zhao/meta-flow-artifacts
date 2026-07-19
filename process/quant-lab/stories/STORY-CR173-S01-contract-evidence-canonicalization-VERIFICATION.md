---
story_id: "CR173-S01-contract-evidence-canonicalization"
cr_id: "CR-173"
stage: "CP7"
validation_mode: "mixed"
status: "PASS"
reviewer: "meta-qa-critical"
verified_at: "2026-07-17T10:08:19+08:00"
reverification_round: 2
reverified_at: "2026-07-17T10:59:08+08:00"
source_packet_ref: "process/context/stories/CR173-S01-contract-evidence-canonicalization.CP7.reverify-2.packet.json"
---

# CR-173 S01 CP7 独立验证报告

## 1. 阶段结论

结论：`NEEDS_REWORK`。

标准合同测试 `45/45 PASS`，语法、diff hygiene、CP6 return packet 均通过；但独立对抗审查确认 3 个阻断性合同漏洞。它们允许未批准方法、非 typed identity/lineage 或不合法 recovery chain 穿过当前构造边界，违反批准 LLD 的 trust binding、typed immutable contract 和 append-only recovery。S01 不得标记 `verified`，S02 依赖不得解锁。

## 2. 验证范围与非范围

验证范围：

- `engine/effective_trial_evidence.py` 的七字段 evidence、F01-F08、canonical identity、method trust binding、external audit 与 recovery。
- `tests/research/test_effective_trial_evidence_contracts.py` 的 45 项合同测试及缺口分析。
- CP6 return/evidence/result 与批准 LLD 的追踪一致性。
- strategy/public C1/I/O/runtime/credential/real-data/remote 边界静态检查。

非范围：S02 exact estimator、S03 golden/public read-only 回归、真实数据、public C1 projection、production persistence、性能 SLA。

全文扩展读取：因 `deep_review` 与实现证据审计，按 verify packet 的 `read_if_needed` 读取 S01 LLD v1.3 与 IMPLEMENTATION；未读取默认禁止的全局长文档。

## 3. 验证对象清单

| 对象 | 验证方式 | 结果 |
|---|---|---|
| standalone evidence schema | 45 项测试 + 构造路径审查 | 七字段正向通过；trust binding 存在漏洞 |
| F01-F08 / state / reason | 参数化测试 + precedence 审查 | 已有 8/8 正向覆盖；typed 输入负向不足 |
| canonical numeric/bytes/hash | 单测、py_compile、实现审查 | PASS |
| computation identity / method binding | repeat 测试 + 未批准 descriptor 对抗样例 | **FAIL** |
| external audit / append-only recovery | repeat/recovery 测试 + 分叉/orphan 对抗样例 | **FAIL** |
| strategy/public/I/O/runtime 边界 | AST/关键字静态扫描 + public path diff | PASS；生产实现未发现越界依赖/调用/写入 |
| CP6 过程证据 | return-check + evidence/result 追踪 | PASS（存在等价外置路径 warning，不阻断） |

## 4. 追踪矩阵

| 验收 / 风险 | 设计合同 | 实现 / 测试 | 结果 |
|---|---|---|---|
| 七字段 7/7、缺字段拒绝 | S01-FR-02 | `EffectiveTrialEvidence` + missing-key tests | PASS |
| F01-F08 8/8、非 present count=null | S01-FR-03/06 | parameterized failure tests | PASS |
| canonical token/bytes/hash 唯一 | S01-FR-04/05 | decimal/serializer/hash tests | PASS |
| 批准 method 与 basis trust binding | S01 §5.1/§6 | `build_effective_trial_evidence` | **FAIL：F-CP7-S01-001** |
| typed identity/input lineage | S01-FR-01、§5.1 | constructors + `validate_contract_bundle` | **FAIL：F-CP7-S01-002** |
| 1 computation/1 evidence/3 audits | S01-FR-07 | repeat test | PASS |
| recovery A/B 单链、orphan=0 | S01-FR-07、§5.5 | `AttemptAuditLog` | **FAIL：F-CP7-S01-003** |
| raw/strategy/public/runtime 越界=0 | S01-NFR-03/04 | AST/static/public diff | PASS |

## 5. 分层验证执行

| 层 | 命令 / 检查 | 结果 |
|---|---|---|
| 单元/合同 | `PYTHONDONTWRITEBYTECODE=1 PYTEST_ADDOPTS='-p no:cacheprovider' uv run --python 3.11 pytest -q tests/research/test_effective_trial_evidence_contracts.py` | PASS，45 passed in 0.08s |
| 语法 | `PYTHONPYCACHEPREFIX=/tmp/cr173-s01-cp7-pycompile uv run --python 3.11 python -m py_compile ...` | PASS |
| diff hygiene | `git diff --check -- engine/effective_trial_evidence.py tests/research/test_effective_trial_evidence_contracts.py` | PASS |
| CP6 return | `meta-flow story return-check ...CP6...` | PASS，1 条外置路径等价 warning |
| public production diff | 8 个 forbidden production path 的 `git diff --name-only` | PASS，输出 0 |
| 对抗：未批准方法 | 构造变更 formula、自洽 hash、手工 basis，再调用 evidence builder | FAIL，输出 `UNAPPROVED_METHOD_ACCEPTED True` |
| 对抗：非 typed contract | 使用 int family ref/trial ID/lineage 与自洽 hash 调用 validator | FAIL，输出 `UNTYPED_CONTRACT_ACCEPTED True int int` |
| 对抗：recovery 分叉 | parent=A、supersedes=B，追加到含 A/B 的日志 | FAIL，输出 `SPLIT_RECOVERY_CHAIN_ACCEPTED 3 True` |
| 对抗：initial orphan | 以不存在的 recovery ref 直接构造初始日志 | FAIL，输出 `ORPHAN_INITIAL_LOG_ACCEPTED 1 True` |

安装、平台 dry-run、workflow eval：N/A；本 Story 是 repository-local pure Python code-project，无安装器、Prompt、平台渲染或 workflow 产物。

## 6. Findings（严重度优先）

### F-CP7-S01-001 — HIGH / BLOCKING — evidence builder 接受未批准 method descriptor

- 位置：`engine/effective_trial_evidence.py:918`，尤其 `:931-938`。
- 证据：builder 只校验 method ID/version、descriptor 的自洽 hash、basis hash 相等，没有校验 `canonical_spec_descriptor == approved_method_descriptor()`；对抗样例返回 `UNAPPROVED_METHOD_ACCEPTED True`。
- 影响：调用方可手工构造带任意 formula 的 descriptor 与 basis，并产出 `present` evidence；F07 与 approved-method trust boundary 可被绕过。
- 必须修复：在可产生 evidence 的权威边界验证批准 descriptor/批准 hash；补充“自洽但未批准 descriptor 必须拒绝”的负向测试。不得用 public adapter 或默认 method 降级。

### F-CP7-S01-002 — HIGH / BLOCKING — typed identity/input 合同未落实运行时类型约束

- 位置：`engine/effective_trial_evidence.py:271-335` 与 `:600-735`。
- 证据：`SealedTrialIdentity`/`DependencyMatrixEnvelope` 构造仅 tuple 化；validator 对 ref、trial ID、lineage 多数只检查 truthiness。使用 `int` family ref、trial ID、lineage 和相应自洽 hash 可得到 `result.is_valid=True`。
- 影响：声明为 `str` 的 identity/lineage 不再是 machine-enforced typed contract，非法 provenance 可进入 S02，并在后续 basis 构造才以异常暴露，破坏稳定 F01/F02/F06 fail-closed 语义。
- 必须修复：在构造或 validator 的权威入口对所有标注为字符串/字符串序列/sha256 ref 的字段执行明确类型与非空校验，并把失败稳定映射到批准的 F01/F02/F06；补充 wrong-type、empty-element、mixed-ID negatives。

### F-CP7-S01-003 — HIGH / BLOCKING — append-only recovery 可形成分叉或 orphan 初始链

- 位置：`engine/effective_trial_evidence.py:983-1003`、`:1023-1058`、`:1067-1091`。
- 证据：parent/supersedes 仅要求同时为空或非空，未要求二者指向同一 A；append 仅分别检查已存在；`AttemptAuditLog.__post_init__` 不校验 recovery refs。分叉与 orphan 两个对抗样例均被接受。
- 影响：违反批准 LLD“B 的 parent/supersedes 均指向 A、orphan 接受数=0”，审计链可歧义或失去根，不能作为可审计恢复证据。
- 必须修复：强制 recovery pair 等值并指向同一既有 entry；`__post_init__` 对任意初始 entries 重放顺序/引用完整性校验，或禁止非空直接初始化；补充分叉、orphan、forward-ref negatives。

## 7. 人工 / 语义质量审查

- 范围：未发现 estimator 算法、public projection、真实数据、I/O、runtime、credential、trading、publish/deploy 或 Git remote 实现；无范围外生产集成。
- 过度实现：1138 行主要来自 immutable contracts、canonical serializer、basis/audit 类型和防御性验证；未发现范围外模块，但当前防御在 3 个关键 trust 边界不完整。
- 错误语义：标准 happy-path/F01-F08 测试通过，但测试使用内部 helper 生成“天然合法”对象，未覆盖敌对调用方直接使用导出 constructors/builders 的场景。
- 文档/证据：CP6 将“forged/orphan/recovery 均覆盖”写为 PASS，但当前负向仅覆盖格式明显非法的 `sha256:forged` 和 append 重复，不能支撑完整合同结论。

## 8. 剩余风险与复验范围

| 风险 | 状态 | Owner | 复验条件 |
|---|---|---|---|
| R-CR173-RAW-EFFECTIVE-ALIAS | 未发现 raw fallback；继续受控 | meta-dev/meta-qa | 原 45 项 + 新 trust negatives 全通过 |
| R-CR173-METHOD-NONDETERMINISM | S01 identity 正向受控，但 method approval 可绕过 | meta-dev | F-001 修复并复验 |
| audit recovery ambiguity | OPEN / BLOCKING | meta-dev | F-003 三类 negative 均拒绝 |
| public/runtime scope escape | 当前 PASS | Host/meta-qa | 回修 diff 仍只限两个 primary 文件 |

最小复验集：原 45 项合同测试 + 未批准 descriptor + wrong-type identity/lineage + split parent/supersedes + initial orphan/forward-ref；再跑 py_compile、diff check、public forbidden path diff。

## 9. 路由

`NEEDS_REWORK → meta-dev（同 Story） → CP6 回修证据 → meta-qa-critical 重新执行 CP7`。本轮没有设计歧义，不需要重开 CP5；修复应保持 LLD v1.3 规范合同不变，design delta 预期为 0。

## 10. 产物校验

- CP7 return-check：OK（仅外置 process 路径等价 warning）。
- CP7 evidence-check：OK。
- CP7 result-check：产物 schema/枚举项已通过；当前仅因 meta-qa dispatch 尚未由 Host 写入 terminal successful 事件、全局 `stop_reason` 尚未同步为 `needs_rework` 而返回 FAIL。两项均属于 Host 收口职责，不改变本报告的 `NEEDS_REWORK` 结论。

---

# CP7 复验 1（CP7R1）

## 11. 复验结论

结论仍为 `NEEDS_REWORK`。

CP6R1 新增 8 项负向测试与原 45 项合计 `53/53 PASS`；首轮 4 条原始对抗样例也全部由 `ACCEPTED` 转为 `REJECTED`。但独立同类旁路审查确认：F-CP7-S01-002 与 F-CP7-S01-003 只部分关闭，仍各有一个可复现缺口。此轮不创建新 finding ID，而是保持原 finding 为 OPEN。

## 12. 复验范围与对象

- 权威输入：CP7 首轮 result、CP6R1 return/evidence/result、批准 LLD v1.3。
- 工程对象：`engine/effective_trial_evidence.py`、`tests/research/test_effective_trial_evidence_contracts.py`。
- 必验合同：批准 method binding、typed identifier 的稳定失败映射、audit content-addressed single-chain integrity、原七字段/F01-F08 回归。
- 非范围：S02 estimator、public C1、真实数据、runtime、production persistence、remote。
- 全文扩展原因：`deep_review`；为检查相同 finding 的变体，读取批准 LLD。未读取默认禁止的全局长文档。

## 13. 复验执行结果

| 层 | 检查 | 结果 |
|---|---|---|
| 完整回归 | 定向 pytest | PASS，53 passed in 0.10s |
| 语法 | py_compile | PASS |
| hygiene | git diff --check | PASS |
| CP6R1 过程证据 | return-check / evidence-check | PASS / PASS |
| public/授权边界 | 8 个 public production path diff + I/O/runtime 关键字扫描 | PASS，触达数 0 |
| 原样例：unapproved method | 首轮构造原样重跑 | `UNAPPROVED_METHOD_REJECTED` |
| 原样例：int typed identity/lineage | 首轮构造原样重跑 | `UNTYPED_CONTRACT_REJECTED True F01` |
| 原样例：split recovery | 首轮构造原样重跑 | `SPLIT_RECOVERY_REJECTED` |
| 原样例：initial orphan | 首轮构造原样重跑 | `ORPHAN_INITIAL_LOG_REJECTED` |
| 同类变体：bytes family/trial ID | validator 稳定失败映射 | **FAIL：均逸出 `CanonicalizationError`** |
| 同类变体：forged content-addressed audit ref | 使用格式合法但内容错误的 sha256 ref 重建日志 | **FAIL：`FORGED_AUDIT_REF_ACCEPTED 1`** |

## 14. Finding 关闭审计

### F-CP7-S01-001 — CLOSED

- 回修：`build_effective_trial_evidence` 同时校验 approved descriptor、approved hash、自洽 hash 与 attempt-basis hash。
- 独立证据：未批准但自洽 descriptor 已被拒绝；相关 53 项测试保持全绿。
- 结论：关闭；未发现同类 approved-method 绕过。

### F-CP7-S01-002 — OPEN / BLOCKING（部分关闭）

- 已关闭部分：`int` family ref/lineage、int/空/mixed trial IDs 已稳定映射至 F01/F02/F06。
- 未关闭部分：`bytes` family ref 或 trial ID 会在 `_validation_result → canonical_component_digest → canonical_bytes` 路径逸出 `CanonicalizationError`，未返回批准的 F01/F06 typed failure。
- 影响：类型检查虽然不再接受这些值，但“所有错误 typed identifier 稳定 fail-closed”仍未兑现；调用方得到未建模异常而非 machine reason。
- 必须修复：类型失败分类必须先于不可信 snapshot canonicalization；对不可 canonicalize 的错误原值使用安全、确定的 rejected snapshot marker/digest，或在 constructor/validator 更早阻断，同时保留 F01/F02/F06。补充 bytes 与另一种不可 canonicalize 类型的回归。

### F-CP7-S01-003 — OPEN / BLOCKING（部分关闭）

- 已关闭部分：split parent/supersedes、initial orphan、forward ref、已 superseded 节点分支均拒绝。
- 未关闭部分：`ComputationAttemptAudit` 只验证 `attempt_audit_ref` 的 sha256 格式，没有重算其 content-addressed identity。将合法 audit 用 `dataclasses.replace` 改为任意 64 位 sha256 后，`AttemptAuditLog` 接受该节点。
- 影响：攻击者可伪造 audit node identity，再建立表面单链；这违反 LLD 对 `attempt_audit_ref` 的内容寻址定义，审计链不能证明节点内容未被篡改。
- 必须修复：在 `ComputationAttemptAudit.__post_init__` 或 `_validate_audit_sequence` 中按 audit domain 与 identity mapping 重算并校验 ref；补充 forged-ref root 与 forged-ref recovery 两类负向测试。

## 15. 追踪与质量判断

| 合同 | 状态 | 证据 / 风险 |
|---|---|---|
| 七字段 7/7、F01-F08、canonical numeric/hash | PASS | 原 45 项保持通过 |
| approved method trust binding | PASS | F-001 CLOSED |
| typed identifier 稳定 failure mapping | FAIL | F-002 仍有 non-canonical type 逸出 |
| audit single-chain 拓扑 | PASS | split/orphan/forward/branch 已拒绝 |
| audit node content identity | FAIL | F-003 forged sha256 ref 可接受 |
| strategy/public/I/O/runtime/real/remote 边界 | PASS | 触达数均 0 |
| design delta | N/A | 回修应继续遵循 LLD v1.3；无需重开 CP5 |

现有 53 项测试覆盖了实现方已知的 8 个样例，但仍存在“测试全绿、合同未闭环”的 happy-path 偏差。两项剩余缺陷均有可复现证据，不能降级为剩余风险或 waiver。

## 16. 路由与最小复验集

`NEEDS_REWORK → meta-dev（S01 回修 2） → CP6R2 → meta-qa-critical CP7R2`。

最小复验集：

1. 当前 53 项全部通过；
2. bytes/non-canonical family ref、trial ID、lineage 均返回稳定 F01/F02/F06，未建模异常数 0；
3. forged root/recovery audit ref 接受数 0；
4. 原 4 条对抗样例继续全部拒绝；
5. py_compile、diff check、public/授权边界继续为 PASS/0。

S01 在 F-002/F-003 真正关闭前不得标记 verified，S02 依赖不得解锁。

## 17. CP7R1 产物校验

- CP7R1 return-check：OK（仅外置 process 路径等价 warning）。
- CP7R1 evidence-check：OK。
- CP7R1 result-check：schema、枚举、finding 路由已通过；仅等待 Host 将本轮 meta-qa dispatch 写为 terminal successful，并同步全局 `stop_reason=needs_rework`。该预期收口项不改变复验结论。

---

# CP7 最终复验 2（CP7R2）

## 18. 最终结论

结论：`PASS`。

CP6R2 的原 53 项与新增 7 项合计 `60/60 PASS`。独立复验确认 F-CP7-S01-002、F-CP7-S01-003 已完整关闭：`bytes`、`bytearray`、`memoryview` 及不可哈希 trial identifier 均在不可信值进入 canonical hash、集合或排序操作前稳定映射到 F01/F02/F06，异常逸出数为 `0`；伪造 root、recovery、constructor audit ref 的接受数为 `0`。未发现同因相邻旁路。

本结论只证明 S01 standalone contract、typed fail-closed 与 external audit identity 达到已批准 LLD；不证明 S02 estimator、S03 fixture/golden harness、public C1、真实数据 activation、Stage 3 或 production SLA。

## 19. 最终复验范围与执行

- 权威上下文：`process/context/stories/CR173-S01-contract-evidence-canonicalization.CP7.reverify-2.packet.json`。
- 输入证据：CP7R1 result、CP6R2 return/evidence/result。
- 工程对象：`engine/effective_trial_evidence.py` 与 `tests/research/test_effective_trial_evidence_contracts.py`；仅审查和执行，不修改。
- 全文扩展：按 packet `deep_review` 条件读取批准 S01 LLD，用于核对 fail-closed precedence 与 audit identity mapping；未读取 archive/discussions/全局长计划。

| 层 | 检查 | 结果 |
|---|---|---|
| 完整回归 | 定向 pytest | PASS，`60 passed in 0.11s` |
| 语法 | 两个目标模块 `py_compile` | PASS，2/2 |
| hygiene | 两个目标文件 diff whitespace 检查 | PASS，错误数 0 |
| CP6R2 过程证据 | return-check / evidence-check | PASS / PASS；仅有外置 process 等价路径 warning |
| public production diff | 8 个 forbidden production path | PASS，变更路径数 0 |
| 静态边界 | I/O、network、subprocess、credential、lake、publish、trade、git push 关键字 | PASS，命中数 0 |
| 相邻 identifier 变体 | family bytearray/memoryview；lineage bytearray/memoryview；trial list/dict/set | PASS，7/7 稳定为 F01/F02/F06，异常逸出 0 |
| audit identity 变体 | constructor forged ref；绕过 constructor 的 root/recovery forged ref | PASS，3/3 拒绝，接受数 0 |

## 20. Finding 关闭审计

### F-CP7-S01-001 — CLOSED（保持）

- CP6R1 已关闭批准 method descriptor/hash/basis trust binding 绕过。
- 本轮 60 项回归保持通过，未发现回归或相邻 approved-method 旁路。

### F-CP7-S01-002 — CLOSED

- 权威类型检查先于对不可信 identifier 的集合/排序；validation result 的组件摘要经 `_safe_component_digest` fail-closed。
- 既有覆盖：family bytes/bytearray、trial bytes、lineage bytes。
- 独立相邻变体：family/lineage `memoryview`、lineage `bytearray`、trial 内含 `list`/`dict`/`set`，均无异常逸出，并分别稳定返回 F01/F02/F06。
- 量化结论：目标 machine reason 命中 `7/7`，未建模异常 `0`，旁路接受 `0`。

### F-CP7-S01-003 — CLOSED

- `ComputationAttemptAudit.__post_init__` 按冻结 domain/mapping 复算 `attempt_audit_ref`；`AttemptAuditLog` 在接纳边界再次独立复算，覆盖构造器正常路径与反序列化/篡改绕过路径。
- constructor forged ref、绕过 constructor 的 root forged ref、recovery forged ref 均被 `EvidenceContractError` 拒绝。
- 量化结论：伪造 ref 接受 `0/3`；既有 split/orphan/forward/branch negatives 与 60 项回归全部通过。

## 21. 追踪、边界与剩余风险

| 验收合同 | 最终状态 | 证据 |
|---|---|---|
| 七字段 7/7、F01-F08、canonical numeric/hash | PASS | 60 项完整回归 |
| approved method trust binding | PASS | F-001 保持 CLOSED |
| non-canonical identifier 稳定 machine reason | PASS | F-002 CLOSED；7 个相邻变体，异常逸出 0 |
| append-only recovery topology | PASS | split/orphan/forward/branch negatives |
| audit node content identity | PASS | F-003 CLOSED；forged 接受 0/3 |
| public/real/runtime/credential/remote 边界 | PASS | 8 路径 diff=0；静态命中=0 |
| normative design delta | N/A | 设计合同变化数 0 |

剩余风险均不阻断 S01：exact-integer growth 与 estimator 数值/性能属于 S02；golden vector、三次重复、external audit evidence linkage 与 12/12 public read-only regression 属于 S03；真实 activation 时仍需 CR-172 独立授权与规模评估。S01 未产生 waiver。

## 22. 路由与收口

S01 可由 Host 标记 `verified` 并解锁依赖 Story S02。下一路由为 `CR173-S02-IMPLEMENTATION`；本结论不自动启动 CR-172、不改变 public C1 `typed_unavailable`，也不提升任何 Stage 3/admission claim。

CP7R2 return/evidence/result 的 schema 与追踪校验由本轮生成；dispatch terminal、Story/STATE/current 路由和 checkpoint/event ledger 由 Host Orchestrator 收口。

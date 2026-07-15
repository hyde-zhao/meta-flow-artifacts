---
status: approved-cp3
version: "0.3"
change: CR-170
source_hld: process/archive/design-cr-docs/HLD-CANONICAL-RELIABILITY-NA-SEMANTICS-ADMISSION.md
---

# Canonical Reliability N/A Semantics and Admission ADR

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 0.1 | 2026-07-15 | host-orchestrator inline meta-se-critical | 起草 CR-170 四项 CP3 架构决策：内部五态 policy、完整 N/A 结果、merge/admission 分层、兼容与 future-consumer 边界。 |
| 0.2 | 2026-07-15 | host-orchestrator inline meta-se-critical | 根据 CP3 深度评审为 ADR-001 增加 baseline path/direction/disposition 字段，为 ADR-002 冻结 15/5/1 方向分组及 G1-P06 禁止 N/A 替代，为 ADR-003 明确 T3 现有 early-return 保持不变，并补 caller/authorization-ref 责任边界。 |
| 0.3 | 2026-07-15 | host-orchestrator inline | 回填用户对 CP3 四项 ADR 的批准，状态统一转为 ACCEPTED，并迁入设计归档。 |

## ADR-CR170-001：采用表驱动内部 N/A policy decision

- 状态：ACCEPTED（2026-07-15，CP3 用户批准）。
- 背景：单一 `_has_na_reason` bool 无法表达 policy applicability、owner、boundary 与 tier；逐 Gate 手写 21 组条件易漂移。
- 决策：使用固定 21-unit `NaPolicySpec` inventory 与内部 typed five-state decision。每行除 applicability/owner/boundary/result 外，必须记录 `baseline_na_path_type`、`hardening_direction` 与 `complete_na_disposition`。公共 validator 继续接收现有 mapping；内部 normalizer 可读取 caller 显式提供的可选 `n_a_boundaries[policy_id]`。全局 `_has_na_reason` 的布尔语义不改变，且不得作为 21 个 mandatory policy unit 的充分条件。
- 备选：逐 Gate 条件补丁；全局 helper 改义。
- 理由：表驱动既保留局部爆炸半径，又提供 21/21 可审计分母；全局 helper 改义无法区分合法 N/A 和逃逸。
- 后果：CP4/CP5 必须冻结内部对象、exact reason-id 枚举和 21/21 方向清单；当前代码基线分组为更严/受控放宽/保持 `15/5/1`；公开 API/schema 不变。
- 切换条件：只有 inventory 被证明无法由统一字段表达时，允许对个别 Gate 使用受控专用 evaluator，但仍必须返回同一 five-state decision。

## ADR-CR170-002：完整结构化 N/A 对 applicable mandatory evidence 产生 NEEDS_REVIEW

- 状态：ACCEPTED（2026-07-15，CP3 用户批准）。
- 背景：合法 N/A 必须可表达，但 reason 字符串不能代替 mandatory evidence。
- 决策：`NA_WITH_COMPLETE_BOUNDARY` 要求 reason、owner、scope、release_profile 或 authorization_ref `4/4` 完整且匹配 policy。对 `complete_na_disposition=reviewable` 的 applicable mandatory unit，结果至少为 `NEEDS_REVIEW`，不得产生无条件 PASS；对 `prohibited` unit（当前为 G1-P06 trial count/provenance），即使 boundary 完整也保持 BLOCKED，不得用 N/A 替代 value/provenance 校验。missing、generic escape、incomplete boundary 均生成 stable claim 并保持非 PASS。对明确不适用的 conditional unit，可记录完整 N/A，但不得借此覆盖其他 applicable mandatory unit。
- 备选：所有 N/A 一律 BLOCKED；complete N/A 等同 PRESENT。
- 理由：全 BLOCKED 会丢失合法 fixture/static applicability；等同 PRESENT 会重开虚假 PASS。
- 后果：LLD 必须定义 exact boundary lookup、owner/scope match、deterministic claim ordering，并按现有路径与方向分组回归：15 个更严 unit 不再 PASS，5 个受控放宽 unit 只允许 Gate-local BLOCKED->NR 且 T1/T2 仍 BLOCKED，1 个保持 unit 结果不变。
- 切换条件：若未来产品政策禁止某类 N/A，该 policy unit 可在独立 CR 改为 mandatory BLOCKED，不改变五态框架。

## ADR-CR170-003：保护 bottom-up merge，单独硬化 admission resolver

- 状态：ACCEPTED（2026-07-15，CP3 用户批准）。
- 背景：`evaluate_shared_contract` 已传播 `NEEDS_REVIEW`；问题在上游 Gate 未生成信号以及 `resolve_admission_policy` 未处理 mandatory NEEDS_REVIEW。
- 决策：先证明 `build_shared_gate_summary` NEEDS_REVIEW propagation `1/1 PASS`；通过时相关生产逻辑修改=`0`。在 `resolve_admission_policy` 最小硬化 T0/T1/T2：T0=`NEEDS_REVIEW`、T1/T2=`BLOCKED`。当前 T3 已在通用 tier 判定前 early-return status=`BLOCKED` + mode=`NOT_AUTHORIZED`，本 CR 只做 `1/1` 兼容回归，T3 生产逻辑修改=`0`。
- 备选：重写 merge；把 tier decision 下沉每个 Gate；新增 `NOT_AUTHORIZED` status enum。
- 理由：保持已有正确层，避免重复 policy；现有 mode 已能表达 T3，不需破坏公开 enum。
- 后果：Gate summary 与 admission result 的职责清晰；current Stage3 runner 接入仍为 0。
- 切换条件：只有受保护回归失败并定位到 merge 才能通过 design delta 重开；若未来公开 schema major version 演进，可再评估 status enum。

## Caller / Authorization Ref 补充契约

- 当前 `n_a_boundaries` 与 `authorization_ref` 的允许写入方仅为 CR-170 fixture/test caller；evaluator 只校验和消费，不合成、不补写、不从其他 unit 复制。
- `authorization_ref` 是指向既有决定的 opaque、非敏感审计引用，不得包含 credential/token/secret，也不构成新的授权。
- 现有 caller 不提供 mapping 时继续进入 missing/value-validation 路径；aggregate、real-evidence 与 runtime authorization caller 由 FU-009 或独立 Stage 3 CR 冻结。

## ADR-CR170-004：保持公共兼容、adapter defense-in-depth 与 future-verifier 边界

- 状态：ACCEPTED（2026-07-15，CP3 用户批准）。
- 背景：CR-168/169 adapter 已包含 typed payload 与 pre/post guards；FU-006 verifier lane 尚未实现。
- 决策：CR-170 的 public callable、result schema、CR-168/169 adapter guard 保持不变；adapter regression=`2/2 PASS`。独立验证者仅是 future consumer，本 CR 由 Gate 维护者自验证，不声明 verifier independence。adapter 简化只允许在 FU-009 中同时满足 caller 全覆盖、fail-closed 不降低、全回归 PASS、独立 ADR 四条件后评估。
- 备选：本 CR 删除 adapter；本 CR 实现 verifier lane。
- 理由：canonical 与 adapter 的防护职责不同；扩入 verifier 会改变风险、范围和 agent independence 证据。
- 后果：CP8 必须披露 verifier independence 风险；Stage3 runner/aggregate/CR155 promotion 均保持范围外。
- 切换条件：FU-006 正式交付后可将 future consumer 改为 current consumer；FU-009 满足四条件后可评估 adapter 简化。

## 决策一致性矩阵

| 决策 | HLD 落点 | Requirement / QAC | 场景 |
|---|---|---|---|
| ADR-001 | HLD §4-7 | REQ-001/002/004；QAC-01/02/04 | P01/P02/N03..N09 |
| ADR-002 | HLD §5 | REQ-002/004；QAC-05 | P02/N08/N09 |
| ADR-003 | HLD §8 | REQ-005/006；QAC-06/07/08 | B01..B05 |
| ADR-004 | HLD §2.2/2.3/9/14 | REQ-007/009；QAC-09/10/11/13/14 | R01/R02/G01/E01 |

## CP3 批准边界

批准四项 ADR 只解锁 CP4/CP5 的 Story 与详细设计，不授权实现、测试执行、真实数据、Stage 3、runtime、aggregate、CR-155 promotion 或 Git 远端写入。

---
handoff_id: "CR172-CP5-LLD-R2-ARCHITECTURE-CORRECTION"
change_id: "CR-172"
checkpoint: "CP5"
from_role: "host-orchestrator"
to_role: "meta-se"
canonical_role: "meta-se"
codex_agent_name: "meta-se-critical"
reasoning_profile: "critical"
context_ref: "process/context/CP5-CR172-PATH-I-LLD-CONTEXT.yaml"
review_findings_ref: "process/checks/CP5-CR172-LLD-REVIEW-R1-FINDINGS.md"
status: "ready"
---

# CR-172 CP5 LLD R2 架构整改交接

## 目标

根据独立评审 F-CR172-CP5-001～005，先修订 HLD、ADR、Feature / Story Plan 的架构与范围合同，使 LLD R2 有单一、可实施、fail-closed 的上游基线。

## 必须采用的整改方向

1. `F-001 lineage`：不扩展既有 append-only lineage owner 范围。删除“finish 中途失败后成功 lineage 痕迹为 0”的不可实施承诺；冻结为 `partial_lineage_blocked_audit`：任何部分成功事件均不可擦除、不可被 `fail()` 伪装回滚、不得推进 trial-return canonical selection，必须输出 machine-readable BLOCKED 审计证据。原子批次/outbox/correction-supersession 能力登记为独立 lineage-owner 前置 CR。
2. `F-002 return semantics`：不得把当前 `turnover.next_rebalance_date/net_forward_return` 映射为真实 trial portfolio period return。把它明确分类为 `forward_label_proxy@v1`，禁止进入 empirical-R/effective-count。当前仓库不存在可识别的 native multi-trial period-return producer，因此本 CR 收窄为 ReturnDefinitionV1、artifact/seal/replica/materialization、fixture producer port 与 zero-op guard；`engine/mature_multifactor_research.py` 保持不改。未来真实 producer/instrumentation 另设 runtime-high-risk 前置 CR，并必须提供真实区间端点、持仓/权重、成本和 non-overlap 语义。
3. `F-003 fixture/real`：冻结可编码的 `decision_origin` / `target_kind` binding，不得改变 HLD 已冻结的 12-field approval record。推荐在派生 `ActionDecisionV1` 中增加 `decision_origin=repository_fixture|approved_ledger`，并规定 `repository_fixture` 只能绑定 `target_kind=repository_fixture` 和 repository-confined root；fixture decision + non-fixture target accepted=`0`。
4. `F-004 seal contract`：S02 必须导出唯一 canonical seal digest 合同、verified result 和精确类型；S03 精确消费 `SealedTrialReturnBundleV1`、`ResearchCanonicalSelectionV1`、`VerifiedTrialReturnBundleV1`、`ActionDecisionV1`、`ActionScopeContextV1` 与 `verify_sealed_trial_return_bundle`。`original_seal_sha256` 只能来自 verifier 返回的同一 canonical seal bytes。
5. `F-005`：修正 S04 16 个唯一测试 ID 的重复项。

## 影响文件

- `docs/design/HLD-TRIAL-RETURN-DEPLOYMENT-CONTRACTS.md`
- `docs/design/ARCHITECTURE-DECISION-TRIAL-RETURN-DEPLOYMENT-CONTRACTS.md`
- `docs/design/FEATURE-DESIGN-MATRIX.md`
- `docs/features/trial-return-artifact-pipeline/{DESIGN,TEST-PLAN,TASKS}.md`
- 必要时同步相邻 Feature 文档、`process/DEVELOPMENT-PLAN.yaml` 与五张 Story 卡；不得修改五份 LLD、源码、测试或 fixture。

## 退出条件

- HLD/ADR/Feature/Story/Plan 内部一致，修订记录完整。
- 现有 runner hook 声明数量=`0`；`net_forward_return` 进入 trial-return/empirical-R 接受数量=`0`。
- partial lineage 被诚实标为 BLOCKED audit lane；canonical selection advance=`0`。
- fixture/real typed binding 与 S02→S03 seal public contract 可由 LLD R2 直接消费。
- 六类真实动作 authorized/executed 仍为 `0/6`,`0/6`。
- 写 return summary，列出精确修改文件、剩余 OPEN/Spike 和 LLD R2 handoff 要点。

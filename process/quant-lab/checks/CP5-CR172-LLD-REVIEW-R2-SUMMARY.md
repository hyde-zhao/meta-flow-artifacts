---
artifact: "CP5-CR172-LLD-REVIEW-R2-SUMMARY"
round: 2
status: complete
decision: revise
blocking_count: 1
required_count: 3
optional_count: 0
---

# Review Summary

## 1. 输入清单

- findings_files:
  - `process/checks/CP5-CR172-LLD-REVIEW-R2-FINDINGS.md`
- reviewed_objects:
  - `process/stories/STORY-CR172-S01-action-authorization-eligibility-governance-LLD.md`
  - `process/stories/STORY-CR172-S02-trial-return-artifact-native-hook-LLD.md`
  - `process/stories/STORY-CR172-S03-nas-replica-verification-LLD.md`
  - `process/stories/STORY-CR172-S04-execution-cache-materialization-LLD.md`
  - `process/stories/STORY-CR172-S05-path-i-integration-claim-zero-operation-verification-LLD.md`
  - `docs/design/HLD-TRIAL-RETURN-DEPLOYMENT-CONTRACTS.md`、对应 ADR/Feature v1.1/v1.2
  - `process/checks/CP5-CR172-LLD-R2-CROSS-CONTRACT-CORRELATION.result.json`

## 2. 严重度汇总

| Severity | Count | Owner |
|----------|-------|-------|
| blocking | 1 | `meta-se-critical + S03/S04 LLD owners` |
| required | 3 | `meta-se-critical + S01/S05 LLD owners` |
| optional | 0 | `N/A` |

## 3. 决策

- decision: `revise`
- rationale: `R1 的 lineage、forward-label proxy、S02→S03 seal 单一真相和 S04 test-ID 整改已成立，但尚不满足 proceed 条件。S01 的 approved_ledger 来源仍可由 caller 自报，违背真实授权 0/6；S04 没有 staged sealed bundle + 唯一 verifier 的可实施链，却承诺 bytes-level seal 复验；S05 残留未冻结 evidence_kind 别名；REQ-013 只实现 path value contract，却仍按 runtime default switch 已交付计数。`
- next_checkpoint: `停留在 CP5 准备；完成 R3 最小设计修订、五份 lld-check、跨合同 correlation 和独立复审后再发起 CP5 人工门禁。`

### CP3 Advisor Summary（适用时填写）

> 不适用。本轮没有新增架构选型或用户授权决策；采用最小、不扩权整改即可。

| 输入类型 | 来源 lane | 进入方案形成 | 进入 HLD 后评审 | 处理结果 |
|---|---|---|---|---|
| review_findings | `lane-quality` | no | yes | revise |

## 4. Decision Brief 输入

| 字段 | 内容 |
|---|---|
| 推荐决策 | `修改`：当前不能 approve CP5；先关闭 F-R2-001～004，再做独立 R3 复审。 |
| 备选方案 | `备选 A（推荐）`：当前 evaluator 固定拒绝 approved-ledger、S04 复用唯一 seal verifier、S05 删除 evidence-kind 别名、REQ-013 明确 contract-ready/runtime-deferred。`备选 B`：进一步收窄本 CR，移除 S04 materialization 正向 4/4 实现，只保留 distribution blocked 合同；该路线会降低 PATH-I 当前价值，只有 verifier dependency 无法安全闭合时采用。 |
| 影响维度 | `安全权限`：阻断自报真实授权；`完整性`：保证执行机 staging 原 seal bytes 真复验；`可验证性`：S05 只消费唯一 provenance；`交付诚实性`：REQ-013 不再把未接线 runtime default switch算作完成；`实现复杂度`：均为局部合同修订，不引入真实 adapter。 |
| 优劣分析 | A 保留五 Story 主链和 S04 4/4 价值，设计改动集中但需同步 HLD/Feature/LLD；B 最安全但只交付更窄的 contract，materialization 价值延后。两者都保持 runner/lineage diff=0/0、六真实动作 0/6 和零真实操作。 |
| 风险与回退 | 若 S04 无法在不形成第二 digest 的前提下复用唯一 verifier，则切换 B；approved-ledger adapter、真实 producer/path enforcement、NAS/execution adapter仍必须由未来独立 runtime-high-risk CR 承接。 |
| 用户需决策事项 | `无`。本轮整改不改变已批准 PATH-I 范围、不申请真实权限；完成后再由用户在 CP5 人工门禁批准实现。 |

## 5. 后续动作

1. 由 `meta-se-critical` 同步修订 HLD/ADR/Feature：当前 approved-ledger 不可用；S04 允许 verifier-library dependency 但仍禁止绕过 S03 selection；REQ-013 标记 contract-ready/runtime enforcement deferred。
2. 由 S01/S04/S05 LLD owners 落地对应 API、失败 reason、测试与 DoD；S02/S03只在 verifier facade/typed staged bundle 需要时做最小相关修订。
3. 重跑五份 `lld-check`、S01→S05 executable contract correlation、S04 tampered-seal mutation、REQ-013 claim-boundary检查和 review artifact validator。
4. 仅当 blocking=`0` 且 required=`0` 时，生成 `decision=proceed` 的 CP5 Decision Brief 输入。


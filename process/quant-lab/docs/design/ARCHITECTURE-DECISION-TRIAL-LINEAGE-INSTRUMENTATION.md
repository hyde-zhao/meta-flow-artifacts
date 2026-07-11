---
status: draft-for-cp3
version: "0.1"
cr_id: "CR-163"
source_hld: "docs/design/HLD-TRIAL-LINEAGE-INSTRUMENTATION.md"
confirmed: false
---

# ADR: CR163 Trial Lineage Instrumentation

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 0.1 | 2026-07-11 | meta-se-critical | 初始 CP3 ADR；冻结 architecture、objects、states/count、storage/seal、supersession、integration、claim ceiling 与 authorization。 |

## ADR-CR163-001: 独立 Family Lifecycle，不扩写单次-run Manifest

| 字段 | 决定 |
|---|---|
| 状态 | proposed-for-cp3 |
| 决定 | `ExperimentFamilySpec/Manifest` 为独立 lifecycle；`ExperimentManifest` 与 `BacktestRunSpec` 保持单次 run，通过 run_id/experiment_id/artifact refs 联接。 |
| 备选 | 把 family membership 嵌入每个 ExperimentManifest；用 admission package 当 lineage owner。 |
| 理由 | family 跨多个 trials/runs；单次对象无法拥有完整 membership；consumer 不应写事实。 |
| 后果 | 新 lineage core + adapters；无双向嵌套。 |
| 回退/切换 | 若未来 run 与 family 永远 1:1，也仍保留 refs 兼容，不合并历史对象。 |

## ADR-CR163-002: Session Façade + Typed Event Commands

| 字段 | 决定 |
|---|---|
| 状态 | proposed-for-cp3 |
| 决定 | Producer 使用 `FamilyLineageSession`；内部以 typed event commands append。 |
| 备选 | pure commands；close-time snapshot。 |
| 理由 | façade降低两条 producer 接入负担，commands 保留完整审计与可重放性。 |
| 后果 | façade不得缓存后一次性覆盖；command schema 是核心 contract。 |
| 切换 | 跨语言/消息 transport 出现时公开 pure command interface，保持 schema。 |

## ADR-CR163-003: 六个持久化公共对象

冻结 `ExperimentFamilySpec`、`ExperimentTrial`、`TrialAttempt`、`TrialSelection`、`ExperimentFamilyManifest`、`FamilyLineageValidationResult`。Session 仅 façade。对象 owner 唯一为 lineage core；producer 提交 commands，consumer 读取 projection。

备选是单一 untyped event dict 或把 validation 塞入 manifest。拒绝原因：弱类型使 identity/state 语义漂移；validation 必须绑定但独立于被验证 target，避免自证。

## ADR-CR163-004: Trial / Attempt / Count / Duplicate 语义

- 参数或 seed 不同 = 新 Trial；同 trial retry = 新 Attempt。
- `raw_trial_count=count(distinct stable_trial_id)`；failed/cancelled/excluded/never-started-with-reason 均计数。
- selection 不改变 family membership。
- 同 event identity + 同 canonical payload 幂等；同 identity + 不同 payload blocked。
- wrapper/hook 属同 chain，不按调用层次计数。

备选“只计 successful/selected”与“每 attempt 计 trial”均拒绝，因为允许 family shrinkage 或放大搜索次数。

## ADR-CR163-005: Local Canonical JSON/JSONL Storage

| 字段 | 决定 |
|---|---|
| 状态 | proposed-for-cp3 |
| 决定 | create-only canonical spec JSON、append-only event JSONL、immutable versioned manifest JSON、独立 validation JSON。 |
| 备选 | 单 snapshot；SQLite/MLflow/external registry。 |
| 理由 | 最小本地依赖，可 diff/fixture，满足 append-only与权限边界。 |
| 风险 | concurrent writers 受限。 |
| 切换 | 并发 writer 或索引规模超出本地原子追加时，另起 storage/migration CR 切 SQLite/registry。 |

## ADR-CR163-006: Canonical Serialization and SHA-256 Seal

UTF-8 restricted canonical JSON：递归 key 排序、无多余空白、禁止非有限数、schema-defined decimal normalization；events 按 `(sequence,event_id)`；hash 域包含 schema/spec/events/family/version/prior ref+hash，排除 clock/path/mtime/hash自身。算法 SHA-256 lowercase hex。

备选 RFC 8785 library 或普通 `json.dumps`。当前不引入新依赖，但行为必须以 golden fixtures 精确定义；若采用标准库/跨语言实现，CP5 可选择兼容 RFC 8785 的实现，前提是本 ADR 的 golden bytes/hash 不变，否则需 schema version + ADR change。

## ADR-CR163-007: Immutable Seal and Append-only Supersession

sealed path 只 create，不 update/delete。纠错生成 vN，包含 prior合法 head的 `supersedes_ref/hash` 与 reason；validator 检查单调 version、全链 hash、断链/循环。latest pointer 仅可重建 cache，不是真相源。

备选 controlled overwrite/delete-rebuild 均拒绝。任何旧 hash变化即 tamper blocked。

## ADR-CR163-008: Availability and Manual Legacy Count

- valid native sealed lineage + all checks PASS → `present`。
- 完全未 instrumented 且无伪造 → `typed_unavailable`。
- explicitly excluded/nonexistent path → `not_applicable_with_reason`。
- post-hoc/incomplete/conflict/count mismatch/tamper/broken chain → `blocked`。
- 手填 legacy count 只 reconciliation；无 sealed ref 不 present，冲突 blocked。

备选允许手填 count present 被拒绝，因为它保留 CR163 要消除的后验旁路；完全忽略手填值也被拒绝，因为会漏掉可诊断冲突。

## ADR-CR163-009: Existing Consumer Integration Only

Projection 在现有 CR151 statistical gate、CR154 reliability summary 与 StrategyAdmissionPackage attach/build 边界前提供 availability/ref/raw count/validation。不得新建 gate，status 只可保持或变差，不能因 lineage present 提升其他 blocked evidence。

CR151 的 trial_count 从 validated projection派生；CR154 effective count仍 typed_unavailable，因现有 policy可能继续 blocked，这是正确 fail-closed。Package 增加 refs/summary但不改变 runtime auth flags。

## ADR-CR163-010: C1 Raw-input-only Ceiling and CR155

`effective_trial_count` availability=`typed_unavailable`，ref/method empty；不得 raw→effective copy。CR163 只使 C1 raw-lineage input-ready，不生成 p-values、multiple testing、data snooping或 overfit proof。CR155 无 native lineage，不 backfill，保持 blocked与 `paper_candidate=false`。

## ADR-CR163-011: Frozen Producer Coverage and Story Shape

分母为 2 条去重 chains / CPI-001..004 4 mappings；S03 单 Story覆盖 4/4。五 Story outcome 不变：S01 contract/validator、S02 recorder/seal、S03 producers、S04 consumers、S05 integrity/regression。禁止因 hook 增加第三/第四 chain 或第六 Story。

## ADR-CR163-012: Design-only Authorization

本 CP3 只批准 design contracts。source/test implementation、DEVELOPMENT-PLAN、Story/LLD、quality/release、runtime/data/credential、real lake/NAS/provider、simulation/paper/live/broker/trading、external framework、statistical computation、historical backfill、Git remote/publish 均未授权。出现任何此类必要动作即 BLOCKED，由 host 路由新门禁/CR。

## CP3 Decision Register

| Decision ID | 推荐 | 备选 | Pros/Cons | 影响 | 切换条件 |
|---|---|---|---|---|---|
| DQ-CP3-CR163-001 | ADR-002 session+commands | pure commands；snapshot | 推荐兼顾易用/审计；pure commands复杂；snapshot丢历史 | producer/API | 跨语言 transport |
| DQ-CP3-CR163-002 | ADR-005..007 JSON/JSONL + deterministic immutable seal/supersession | snapshot；SQLite/registry | 推荐最小确定；DB更并发但更重 | storage/security/recovery | 并发/规模 + 新 CR |
| DQ-CP3-CR163-003 | ADR-008/009 reconciliation-only + existing consumers | accept manual present；new gate | 推荐消除 bypass且复用 gate；备选破坏目标/重复政策 | integration/risk | 独立 inferred-backfill policy |
| DQ-CP3-CR163-004 | ADR-001..012 与五 Story 输入，保持 design-only | 修改；暂停 | 推荐形成完整 CP4 输入且不扩权 | route/security | runtime/data需求需另授权 |

## Consistency Notes

| Topic | Frozen result |
|---|---|
| Chains/mappings | 2 / 4, S03 4/4 |
| Objects | 6 persistent + 1 façade |
| Trial semantics | params/seed=trial; retry=attempt; all declared terminal trials count |
| Storage | local canonical JSON/JSONL |
| Seal | deterministic SHA-256, immutable versions |
| Correction | append-only supersession chain |
| Consumer | existing CR151/CR154/package only; no new gate |
| Effective/C1 | typed unavailable/empty; raw-input-ready only, non-computable |
| CR155 | blocked, no reconstruction |

## Gotchas

- “原子替换”不能用于覆盖旧 sealed artifact，只能用于创建新 version。
- validation result 必须携带 target ref/hash，不能只写 `passed=true`。
- session close 失败不能吞掉已 append events；失败历史仍是审计事实。
- compatibility adapter 不能偷偷成为第三 producer chain。
- CP3 approval 不等于 CP5 implementation approval，也不等于 runtime/data authorization。

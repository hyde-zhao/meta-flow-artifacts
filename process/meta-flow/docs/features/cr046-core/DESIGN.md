---
status: draft
version: "1.0"
feature_id: "FEAT-EI-CORE"
related_stories: ["ST-EI-001", "ST-EI-002", "ST-EI-003"]
lld_policy_summary: "3 full-lld"
---

# Feature Design: Evidence Integrity Core

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-12 | meta-se | 冻结 chronology、typed identity、dispatch/platform adapter、thread reuse 与 CP correlation 边界。 |

## 摘要与边界

本 Feature 在既有 ledgers/results/state 之上建立只读 typed evidence graph 和结构化 finding，不建立第二真相源。它拥有 event/dispatch/attempt/thread/receipt/check-attempt identity、chronology、custom-agent request/receipt correlation 和 CP final correlation；不拥有 route metadata、report 聚合、correction 或平台真实实现。

| 相邻对象 | 本 Feature 负责 | 不负责 |
|---|---|---|
| GOVERNANCE | 输出 typed graph/findings | RouteTruth、finalization、compaction apply |
| OBSERVABILITY | 输出 validated snapshot/attestation classification | replay/report/token aggregation |
| Codex platform | 消费 D0/selector/receipt | 签发/伪造 resolved runtime facts |

## 现有位置与变更面

| 区域 | 路径 | 计划变更 |
|---|---|---|
| CP/chronology checks | `meta_flow/checks/cp_result.py`, `state_transition.py` | 引入共享 identity/chronology/correlation contract |
| Event/dispatch evidence | `meta_flow/state/event_ledger.py` + 新 core module（LLD 定名） | attempt/receipt/thread typed normalization；producer adapter |
| CLI | `meta_flow/cli.py` | proposed capability/dispatch/thread checks；名称由 LLD 冻结 |
| Tests | `tests/test_cp_result_event_ledger.py`, `tests/test_state_transition.py`, 新 CR046 fixtures | 正负 contract fixtures |

## 数据与接口

| Object / Interface | 必要字段 / 输入 | 输出 | 失败路径 |
|---|---|---|---|
| TypedIdentity | namespace + stable id + source ref | typed node/edge | collision/dangling/cross-namespace fallback BLOCKED |
| CapabilityProbe | capability_id, session/epoch, observed/expires, selector schema, config hash | fresh/stale/unsupported | 只有 D0 可 discovered |
| DispatchRequest/Receipt | dispatch/attempt, requested/resolved profile, hash/model/effort | verified/degraded/unavailable/mismatch | 缺 receipt 或 mismatch 不 verified |
| ThreadRuntimeIdentity | thread/agent, spawn receipt, profile/hash/model/effort | immutable identity | mutation/new hash => new spawn |
| ReuseReceipt | followup dispatch/attempt + inherited spawn receipt | inherited-unchanged | 缺 receipt不得继承 verification |
| CP Correlator | result/check/gate/state refs + hashes | terminal final selection | 非 final terminal、hash mismatch BLOCKED |

## D0 freshness policy

默认有效性为平台 `expires_at`；若平台只给 TTL，则以 `observed_at + ttl_seconds` 计算。以下任一触发强制 re-probe：TTL/expiry 到期；session_id 或 session_epoch 变化；current config sha256 与 probe 不同；selector/tool schema version 变化；平台 capability regression；host 明确 reload。旧 probe 只保留审计，不得用于新 spawn。无平台 freshness 字段时 strict D0=`unavailable`。

## 关键流程与失败关闭

1. chronology 先验证 causal order；2. D2 config validate/hash；3. D0 fresh probe；4. explicit selector request；5. bound receipt compare；6. freeze thread identity；7. followup 前 reuse admission；8. CP final correlator选择 terminal attempt。任一 required profile 证明缺失即 BLOCKED；preferred fallback 必须有用户 approval并保持 unattested。

## 权限、安全与回滚

- Host/ledger不能写 resolved runtime fact；平台 receipt 是唯一 resolved owner。
- task name、prompt、`codex_agent_name` 均为 D3 self-declared。
- enforce 可回 audit，但不可把 invalid 变 PASS；新字段以兼容 reader 灰度。
- 当前 CR 运行维持 A-baseline；Conditional-B 只有 D0+selector+receipts+全部 fixture 通过后启用。

## Story LLD 合同

| Story | 必须下沉设计 |
|---|---|
| ST-EI-001 | chronology rule table、conditional approval、phase/gate fixtures、文件级 owner |
| ST-EI-002 | schema、freshness/re-probe、producer/reuse/new-spawn、PC-18/19 |
| ST-EI-003 | final attempt选择、hash/cross-truth算法、supersedes与错误码 |

## Gotchas

- D2 TOML scan 绝不能提升到 D0。
- 原 spawn receipt verified 不代表无 reuse receipt 的 followup verified。
- execution completed 与 custom_agent_verified/model_attested 必须三轴记录。

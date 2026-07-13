---
discussion_id: "CP3-CR046-HLD-DISCUSSION"
change_id: "CR-046"
phase: "solution-design"
status: "draft-ready-for-host-broker"
owner: "meta-se-critical"
created_at: "2026-07-12T01:53:44Z"
question_broker: "host-orchestrator"
---

# CR-046 CP3 HLD Discussion Log

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-12 | meta-se-critical | 记录 Architecture Gray Areas、advisor table-first 输入、已知用户约束和待 host 发起的 CP3 决策。 |
| 1.1 | 2026-07-12 | meta-se / host-orchestrator | R2/R3 增补 custom-agent 平台证明、D0-D3 discovery、ThreadRuntimeIdentity 与 CR-046 A-baseline + Conditional-B 狗粮决策；五项决策均保持 pending CP3。 |

## 交互边界

本次 handoff 明确 `can_ask_user=false`、由 host-orchestrator 统一 broker。meta-se-critical 未直接向用户提问，也未把推荐项伪装成已批准。已知用户决定只有：CP3 Decision Brief 必须包含 local-directory `routing_ref` architecture decision，且不得延后到 CP8。

## Architecture Gray Areas

| ID | Advisor 输入 | 推荐 | 可执行备选 | 当前状态 |
|---|---|---|---|---|
| AGA-EI-01 | 共享 typed evidence contract vs checker-local patch vs 新 evidence DB | existing truth + typed contract | checker-local（不推荐）；只读索引 future CR | recommended-pending-CP3 |
| AGA-EI-02 | local-directory route compatibility | real portable metadata + shared resolver | explicit legacy-null；symlink migration | user-required-decision-pending |
| AGA-EI-03 | phase work vs opened human gate | 两态分离，执行中 pending_gate=null | 无；提前 gate 和 stop_reason 代替运行态均拒绝 | recommended-pending-CP3 |
| AGA-EI-04 | post-close correction/audit/replay/pilot 关系 | 通用 correction/replay/audit 先行，pilot adapter 后续独立授权 | true rerun result supersedes（仅 result 对象） | recommended-pending-CP3 |
| AGA-EI-05 | 指定 custom profile/model 的运行时证明 | 规范性 CURRENT/REQUIRED platform contract；D0 discovery + selector + spawn/reuse receipt；ThreadRuntimeIdentity 不可变 | preferred/default 经用户 waiver degraded；required critical/debugger BLOCKED | recommended-pending-CP3 |
| AGA-EI-06 | CR-046 如何在平台扩展未交付时狗粮自身机制 | A-baseline + Conditional-B；17/17 conformance 后新 spawn verified CP7 thread | pure A；pure B | recommended-pending-CP3 |

完整 pros/cons/impact/switch table 位于 HLD §2，正式长期决策位于 ADR-001..011；平台接口和 conformance fixtures 位于 `CR046-CODEX-SUBAGENT-PLATFORM-CONTRACT.md`。

## 方案形成输入

| 来源 | 事实 | 影响 |
|---|---|---|
| CP2-R2 approved baseline | 五项 scope finding 全部 required | compaction/correction/audit/dogfooding/evidence level 进入公共 contract |
| 用户 CP2 审批条件 | routing decision 必须在 CP3 | ADR-004 和 CP3-DQ-02 为 mandatory |
| Host observation | R2 checkpoint event ID duplicate | ADR-002 分离 checkpoint identity 与 check-attempt/event identity |
| Host current observation | solution-design 中间态被 state-transition checker 误拒 | ADR-003 phase-progress/open-gate temporal invariant |
| Repository fact | routing_ref 目标缺失而 workspace check local-directory OK | shared RouteTruth、dangling strict BLOCKED |

## 待 host-orchestrator 收集的 CP3 决策

1. `CP3-DQ-01`：批准 existing truth + typed evidence contract，还是要求 checker-local/新存储备选。
2. `CP3-DQ-02`：批准 local-directory real metadata 主选，还是 explicit legacy-null / symlink migration。
3. `CP3-DQ-03`：批准通用 versioned append-only correction lifecycle，还是限制为真实 rerun result supersession。
4. `CP3-DQ-04`：批准规范性 CURRENT/REQUIRED platform contract、D0-only discovery、immutable thread reuse 与 profile mismatch new-spawn。
5. `CP3-DQ-05`：批准 A-baseline + Conditional-B 狗粮策略，还是 pure A / pure B。

## 当前结论

- HLD/ADR 草案：已形成，可交 host 生成 CP3 Decision Brief。
- formal CP3 gate：R3 已打开、未批准。
- blockers：产品/设计 blocker=`0`；人工决策项=`5`。

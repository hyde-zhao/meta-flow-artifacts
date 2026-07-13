---
handoff_id: "HO-CR046-CP3-META-SE-CRITICAL-RETURN"
from_agent: "meta-se-critical"
to_agent: "host-orchestrator"
change_id: "CR-046"
phase: "solution-design"
status: "returned-design-draft"
created_at: "2026-07-12T01:53:44Z"
context_ref: "process/context/CP3-CR046-DESIGN-CONTEXT.yaml"
cp3_result_ref: "process/checks/CP3-CR046-HLD-CONSISTENCY.result.json"
formal_gate_opened: false
formal_approval_recorded: false
---

# CR-046 CP3 Meta-SE-Critical Return Summary

## 推荐 HLD

推荐 existing canonical truth + shared typed evidence contract：保留现有 ledgers/results/state 为事实源，统一 typed identity graph、RouteTruth resolver、strict chronology/correlation/consistency pipeline、compaction semantic guard，并从已校验证据生成 replay/audit/token reports。HLD：`/home/hyde/workspace/meta-flow-artifacts/process/meta-flow/docs/design/CR046-EVIDENCE-INTEGRITY-HLD.md`。

备选：checker-local patch（较快但持续 split-brain）；新 evidence DB（查询强但形成第二真相源）；仅在明确性能 SLO 无法满足时用独立 CR 评估只读索引。

## 蓝图适用性与产物

判定 `required`：七 Story 共享身份、data ownership、route/state、compaction/correction/replay contract。

- `CR046-EVIDENCE-INTEGRITY-BLUEPRINT.md`
- `CR046-EVIDENCE-INTEGRITY-DOMAIN-MAP.md`
- `CR046-EVIDENCE-INTEGRITY-DEPENDENCY-MAP.md`
- `CR046-EVIDENCE-INTEGRITY-HLD.md`
- `CR046-EVIDENCE-INTEGRITY-ARCHITECTURE-DECISION.md`

以上均位于 `/home/hyde/workspace/meta-flow-artifacts/process/meta-flow/docs/design/`，status=draft、confirmed=false，均有 revision record。

## Architecture Gray Areas

| ID | 推荐 | 备选 / switch | 状态 |
|---|---|---|---|
| AGA-EI-01 evidence model | existing truth + typed graph/pipeline | checker-local；performance breach 后另立只读索引 CR | pending CP3 |
| AGA-EI-02 routing | local-directory real portable metadata + shared resolver | explicit legacy-null；artifact root 已确认后 symlink migration | mandatory pending CP3 |
| AGA-EI-03 phase/gate | phase-in-progress 与 gate-open 分离 | 提前 pending_gate/用 stop_reason 表达运行态均拒绝 | pending CP3 |
| AGA-EI-04 correction/pilot | 通用 append-only lifecycle 先行，pilot adapter 后续授权 | true rerun result supersedes 仅用于 result 对象 | pending CP3 |

Discussion：`process/discussions/CP3-CR046-HLD-DISCUSSION-LOG.md`；checkpoint：`process/checks/CP3-CR046-DISCUSSION-CHECKPOINT.json`。因 handoff 禁止直接提问，未伪造用户选择。

## CP3 Decision Brief 必选项

### CP3-DQ-01 Typed evidence architecture

- 推荐：existing truth + typed graph/shared checker pipeline。
- 备选：checker-local patch；新 evidence store。
- 取舍：推荐统一验证且不复制事实，代价是跨模块 contract 改动；DB 方案当前过重。
- 回退：enforce -> audit；只有性能实证触发只读索引新 CR。

### CP3-DQ-02 `routing_ref` local-directory compatibility（不得延后）

- 推荐：local-directory 也必须有真实、portable-relative metadata，`routing_ref` 必须可解析；workspace/state/doctor/CP 共用一个 resolver。
- valid minimum：schema version、project name、routing mode、path format、project/process/link paths 和 anchors；通用 schema 若要求 artifact root，也必须真实、相对、可移植。
- dangling behavior：strict/compat 均不得 PASS；输出 ref/target/filesystem mode/safe route 后 FAIL/BLOCKED。
- 备选 A：`routing_ref=null` + explicit legacy-local-directory state/migration deadline。备选 B：标准 symlink migration。
- migration/rollback：candidate metadata dry-run 后原子切换；失败不应用；应用后冲突则回旧 state hash 并进入 explicit legacy-null，不得恢复 non-null dangling PASS。
- switch：当前布局可被 portable metadata 准确描述用主选；不可描述用 A；artifact root 确认且 migration checks 通过用 B。

### CP3-DQ-03 Post-close correction

- 推荐：通用 versioned append-only correction，allowlist scope + author/reason/evidence/supersedes + acyclic audit trail。
- 备选：真实 rerun 新 result supersedes（仅 result）；原位修改禁止。
- 回退：追加 superseding correction，不删除历史。

## 七 Story / Feature design 输入

DAG：`001 -> 002 -> 003 -> 004`; `002 -> 005`; `003+004+005 -> 006`; `004+006 -> 007`。四个 Feature 均触发 CP3 后的 Feature-level design（cross-module public contract、state/routing/migration、audit/telemetry schema、post-close mutation boundary）。当前未创建 FEATURE-DESIGN-MATRIX、Story 卡、LLD 或 DEVELOPMENT-PLAN。

## 模拟、风险与边界

HLD 完成 6 个设计模拟：phase/gate、dangling route、compaction restore、null provenance dogfooding、dispatch receipt ceiling、CR-163 pilot boundary，结果 6/6 PASS（设计推演，不是执行证据）。

剩余风险：legacy strict adoption、receipt/telemetry unavailable、route metadata migration、shared checker blast radius。CR-163 仍需独立执行授权；本轮未访问 runtime/credentials、未修改 quant-lab business code、未执行 pilot、未 commit/push。

## CP3 自动预检

- Result：`process/checks/CP3-CR046-HLD-CONSISTENCY.result.json`
- Summary：`process/checks/CP3-CR046-HLD-CONSISTENCY.summary.md`
- 结论：PASS；blockers=0；waivers=0；包含 checker provenance 和 input hashes。
- 下一步建议：host 验证 result 后生成 CP3 Decision Brief/checklist 并正式打开人工门；formal gate/approval 当前均为 false。

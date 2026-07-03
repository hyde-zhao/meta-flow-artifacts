---
story_id: "CR037-S10"
story_slug: "process-only-cascade-and-gate-ledger-event"
cr_ref: "CR-037"
title: "process-only cascade and Gate Ledger event"
design_evidence_type: "full-lld"
status: "draft-for-cp5"
source_story: "process/stories/STORY-CR037-S10-process-only-cascade-and-gate-ledger-event.md"
feature_design_refs:
  - "process/docs/features/roadmap-refresh-governance/DESIGN.md"
  - "process/docs/features/roadmap-refresh-governance/TEST-PLAN.md"
  - "process/docs/features/roadmap-refresh-governance/TASKS.md"
depends_on: ["CR037-S09"]
write_scope:
  primary: ["meta_flow/state/event_ledger.py", "meta_flow/project/**"]
  shared: ["process/state/GATE-LEDGER.ndjson", "tests/**"]
  forbidden: ["process/quant-lab/**"]
cp5_design_confirmed: false
---

# CR037-S10 LLD: process-only cascade and Gate Ledger event

## 1. Goal

CR037-S10 实现 ROADMAP-REFRESH 的 process-only cascade 和 Gate Ledger `roadmap_refresh` event 契约。自动写入范围只允许 process artifact / process-side project objects；发布库 code/tests/formal docs 只进入 `stale_items` 或 `follow_up_candidates`，不得直接写。Gate Ledger 必须 append `roadmap_refresh` event，记录 `result_ref`、`decision`、`gate_ref`、status 和 follow-up refs。

本 Story 消费 S09 result schema，不重新定义 result；不执行 runtime、publish、production write，不修改 `process/quant-lab/**`。

## 2. Requirements

### Functional

| ID | Requirement | Source | Verification |
|---|---|---|---|
| S10-FR-01 | cascade 自动写入范围仅限 process artifact / process-side project objects。 | ADR-PG-003 | INTEG-RF-01、SEC-RF-02 |
| S10-FR-02 | release repo stale impact 只写入 S09 result 的 stale/follow-up fields。 | Story AC | INTEG-RF-03、MAN-RF-01 |
| S10-FR-03 | Gate Ledger append `event_type=roadmap_refresh`，引用 result_ref。 | Story AC / Domain OBJ-PG-013 | INTEG-RF-02、CT-RF-003 |
| S10-FR-04 | ledger append 失败时不伪造成功 event；result 或 return status 必须 BLOCKED/FAILED。 | Feature DESIGN F-RF-04 | SEC-RF-04 |
| S10-FR-05 | rollback refs 可恢复 process-side machine updates。 | Feature DESIGN | IT-RF-002 |

### Non-Functional

| ID | Requirement | Design |
|---|---|---|
| S10-NFR-01 | 安全 | process boundary guard 在写前执行；target 越界先 reject 后记录 blocked。 |
| S10-NFR-02 | append-only | Gate Ledger 只追加，不原地修改历史 event。 |
| S10-NFR-03 | 可回滚 | 每个 applied update 记录 before_hash、after_hash、rollback_ref。 |
| S10-NFR-04 | 可审计 | event 可追溯 result、gate、actor、decision、status 和 follow-up refs。 |

## 3. 模块拆分与职责

| Module | Path | Responsibility |
|---|---|---|
| process boundary guard | `meta_flow/project/roadmap_refresh.py` 或 `meta_flow/project/__init__.py` | 校验 machine update target 是否位于允许 process artifact 边界。 |
| cascade writer | `meta_flow/project/roadmap_refresh.py` | 应用 process-side machine updates，生成 before/after hash 和 rollback payload。 |
| stale/follow-up collector | `meta_flow/project/roadmap_refresh.py` | 对 release repo refs 只生成 stale/follow-up candidates，不写 release repo。 |
| Gate Ledger appender | `meta_flow/state/event_ledger.py` | 扩展 gate ledger required/known event validation，支持 `roadmap_refresh` event。 |
| roadmap refresh command glue | `meta_flow/project/roadmap_refresh.py` + CLI dispatch | 消费 S09 result schema，串联 cascade、event append、checker。 |

当前仓库尚无 `meta_flow/project/` 包；S10 实现时可新增该包，因为 Story ownership 明确包含 `meta_flow/project/**`。

## 4. 代码结构与文件影响范围

| TASK-ID | File / Area | Change | Ownership |
|---|---|---|---|
| TASK-RF-003 | `meta_flow/project/roadmap_refresh.py` | process-only cascade target guard 和 writer。 | primary |
| TASK-RF-004 | `meta_flow/state/event_ledger.py` | Gate Ledger `roadmap_refresh` event contract validation。 | primary |
| TASK-RF-005 | `meta_flow/project/roadmap_refresh.py` | stale_items / follow_up_candidates 输出契约。 | shared |
| TASK-RF-006 | tests | no release write、no credentials/runtime/publish fixtures。 | shared |
| TASK-RF-007 | CP6 implementation evidence | 验证命令和风险说明。 | none |

不修改 S09 schema 文件时不得重新定义 result 字段；只消费并填充 S09 已定义字段。

## 5. 数据模型与持久化设计

### Process boundary

允许自动写入 target 必须满足任一条件：

- `process/project/**`
- `process/state/PROJECT.current.json`
- `process/checks/ROADMAP-REFRESH-*.result.json`
- `process/checks/ROADMAP-REFRESH-*.rollback.json`
- 未来 S06 明确授权的 process-side ROADMAP/MILESTONES refs

禁止自动写入 target 包括：

- `docs/**` 中作为发布库正式文档的路径
- `tests/**`
- `meta_flow/**` 代码路径
- 任意 `quant-lab` release repo path
- 绝对路径、URL、路径穿越、credential-like path

### Machine update

```json
{
  "target_ref": "process/project/ROADMAP.yaml",
  "operation": "update",
  "before_hash": "sha256:old",
  "after_hash": "sha256:new",
  "rollback_ref": "process/checks/ROADMAP-REFRESH-20260703T000000Z.rollback.json",
  "status": "applied"
}
```

### Gate Ledger event

```json
{
  "event_id": "GATE-ROADMAP-REFRESH-20260703T000000Z",
  "event_type": "roadmap_refresh",
  "gate": "CP5",
  "status": "UPDATED_WITH_DOC_IMPACTS",
  "result_ref": "process/checks/ROADMAP-REFRESH-20260703T000000Z.result.json",
  "decision": "UPDATED_WITH_DOC_IMPACTS",
  "actor": "host-orchestrator",
  "source_refs": ["process/checkpoints/CP5-ALL-STORIES-LLD-BATCH.md"],
  "follow_up_refs": ["FU-RF-001"],
  "created_at": "2026-07-03T00:00:00Z"
}
```

`event_ledger.LEDGER_REQUIRED_FIELDS["gate"]` 现为 `event_id`、`event_type`、`gate`、`status`。S10 需在不破坏现有 gate events 的前提下，对 `event_type=roadmap_refresh` 增加条件必填字段校验：`result_ref`、`decision`、`actor`。

## 6. API / Interface 设计

| Interface ID | Signature / CLI | Input | Output | Failure |
|---|---|---|---|---|
| S10-IF-01 | `is_process_target(project_root, target_ref) -> bool` | target_ref | allow/deny + reason | invalid path -> deny |
| S10-IF-02 | `apply_machine_updates(project_root, updates, result_id, dry_run=True)` | updates | applied/rejected updates + rollback payload | outside process -> reject |
| S10-IF-03 | `build_stale_follow_up_items(release_refs, result_context)` | release refs | stale_items/follow_up_candidates | need write -> candidate only |
| S10-IF-04 | `append_roadmap_refresh_event(project_root, result, gate_ref)` | S09 result | appended event id/ref | append failure -> error, no fake success |
| S10-IF-05 | `validate_roadmap_refresh_gate_event(event)` | event dict | errors/warnings | missing result_ref/decision |
| S10-IF-06 | `meta-flow event check --ledger process/state/GATE-LEDGER.ndjson --type gate` | ledger | OK/FAIL | invalid event exit 1 |

错误码：

| Code | Trigger | Behavior |
|---|---|---|
| `E_RF_TARGET_OUTSIDE_PROCESS` | target_ref 不在 process allowlist | reject update，result BLOCKED |
| `E_RF_FORBIDDEN_RELEASE_WRITE` | target_ref 指向 release repo code/tests/docs | reject，生成 stale/FU candidate |
| `E_RF_ROLLBACK_MISSING` | applied update 无 rollback_ref/before_hash | checker FAIL |
| `E_RF_GATE_EVENT_INVALID` | roadmap_refresh event 缺 result_ref/decision/actor | event-ledger check FAIL |
| `E_RF_LEDGER_APPEND_FAILED` | append_event 失败 | 不伪造 event，result BLOCKED/FAILED |
| `E_RF_RUNTIME_AUTH_REQUIRED` | 需要 runtime/publish/production write | blocked finding |

## 7. 核心处理流程

### Process-only cascade

1. 接收 S09 result draft 或 refresh evaluation 产生的 machine update proposals。
2. 对每个 `target_ref` 执行 `is_process_target()`。
3. 越界 target 拒绝写入，转换为 stale_item 或 follow_up_candidate。
4. 合法 process target 在 dry-run 下只计算 proposed hash；实际实现阶段默认应先支持 dry-run，写入需调用方明确授权于 process artifact 范围。
5. 写入前记录 before_hash，写入后记录 after_hash，生成 rollback payload。
6. 将 applied/rejected updates 回写到 ROADMAP-REFRESH result。

### Release repo stale path

1. refresh 发现发布库 docs/tests/code 陈旧。
2. 不调用 write API，不修改 release repo。
3. 生成 `stale_items`，必要时生成 `follow_up_candidates`，prefix 只能为 FU-RF/SP-RF/RA-RF。
4. result decision 通常为 `UPDATED_WITH_DOC_IMPACTS` 或 `BLOCKED`。

### Gate Ledger append

1. S09 result 通过 checker 后，构造 `roadmap_refresh` event。
2. 调用 `event_ledger.append_event()` 追加到 `process/state/GATE-LEDGER.ndjson`。
3. 运行 `validate_event_ledger(..., ledger_type="gate")`。
4. 失败时保留 result errors，不原地修改旧 event，不声称成功。

## 8. 技术设计细节

| Topic | Design |
|---|---|
| Project package | 新增 `meta_flow/project/` 包承载 project governance / roadmap refresh 逻辑，避免塞入 `checks/` 或 `state/`。 |
| Target normalization | 使用 `Path(target_ref)` 处理相对路径；拒绝绝对路径、`..`、URL scheme。 |
| Hash | 复用 sha256 文本/bytes hash；不存在文件 before_hash 可为 `sha256:missing` 或 `null`，需 schema 明确。 |
| Rollback | rollback payload 保存 target_ref、before_content_ref 或 inline content hash；大内容另存 process/checks rollback file。 |
| Ledger compatibility | 不改变 gate ledger 基础必填字段，只为 `event_type=roadmap_refresh` 增加条件校验。 |
| Dry-run | S10 设计建议 refresh command 默认 dry-run，只有 process artifact 写授权明确时应用写入；无论如何不得写 release repo。 |

## 9. 安全与性能设计

### Security

- process-only guard 是写前硬门禁。
- release repo path 一律转 stale/follow-up，不写。
- 不读取 credential，不执行 runtime，不 publish，不 production write。
- Gate Ledger append-only；修正旧事件只能追加 correction/blocked event，不能原地修改。
- `process/quant-lab/**` 是本 Story forbidden path。

### Performance

machine_updates 数量预计较小，逐项 hash 和写入足够。rollback payload 过大时只保存 process-side ref，避免 result 超预算。Gate Ledger append 是单行 NDJSON 写入，校验线性遍历，符合现有 event_ledger 设计。

## 10. 测试设计

| Test ID | Type | Scenario | Expected |
|---|---|---|---|
| INTEG-RF-01 | integration | process-side ROADMAP update | applied update + rollback_ref |
| INTEG-RF-02 | integration/contract | append `roadmap_refresh` event | ledger check PASS |
| INTEG-RF-03 | integration/manual | release docs stale | stale_items/FU candidates only |
| SEC-RF-02 | security | target outside process boundary | reject, result BLOCKED |
| SEC-RF-04 | security | Gate Ledger append failure | no fake success, error recorded |
| SEC-RF-05 | security/manual | refresh asks credentials/runtime/publish | blocked finding |
| IT-RF-002 | integration | process update | decision UPDATED and hashes present |
| IT-RF-003 | integration | doc impacts | UPDATED_WITH_DOC_IMPACTS, no release write |
| IT-RF-005 | integration | ledger event | event contains result_ref |
| CT-RF-003 | contract | event fields | `event_type`、`result_ref`、`decision`、`status` present |
| MAN-RF-01..04 | manual | doc impact, event example, failure wording, target guard | reviewer accepts |

接口到测试映射：S10-IF-01/02 由 SEC-RF-02、INTEG-RF-01 覆盖；S10-IF-03 由 INTEG-RF-03 覆盖；S10-IF-04/05/06 由 INTEG-RF-02、CT-RF-003、SEC-RF-04 覆盖。

## 11. 实施步骤

| Step | TASK-ID | Action | Verification |
|---|---|---|---|
| 1 | TASK-RF-003 | 新增 `meta_flow/project/roadmap_refresh.py` 和 process target guard。 | SEC-RF-02 |
| 2 | TASK-RF-003 | 实现 dry-run / apply machine updates 和 rollback payload。 | INTEG-RF-01、IT-RF-002 |
| 3 | TASK-RF-005 | 实现 release stale/follow-up collector，不写 release repo。 | INTEG-RF-03、IT-RF-003 |
| 4 | TASK-RF-004 | 扩展 `event_ledger.py` gate event 条件校验。 | CT-RF-003 |
| 5 | TASK-RF-004 | 实现 `append_roadmap_refresh_event()` 或 helper。 | INTEG-RF-02、IT-RF-005 |
| 6 | TASK-RF-006 | 添加 no credential/runtime/publish/security negative fixtures。 | SEC-RF-05 |
| 7 | TASK-RF-007 | CP6 实现证据记录 rollback、event refs、验证入口。 | CP6 self-review |

## 12. 风险、难点与预研建议

| Risk | Impact | Mitigation |
|---|---|---|
| process boundary 与发布库边界识别错误 | 越权写或误阻断 | 默认最小 allowlist，只允许 process/project 和 process/checks refresh artifacts |
| Gate Ledger 条件校验破坏旧 event | 现有 ledger check 失败 | 仅对 `event_type=roadmap_refresh` 增加条件字段，不改基础 required |
| rollback payload 过大 | result 膨胀 | 大内容写 process/checks rollback ref，result 只保存 ref/hash |
| append 失败后的状态不一致 | result 和 ledger 不匹配 | result 记录 `E_RF_LEDGER_APPEND_FAILED`，不声称完整成功 |

预研建议：实现前读取 S09 checker 的最终 result schema，S10 只消费并填充；若发现 schema 缺字段，应先回到 CP5/设计 delta，而不是在 S10 私自扩展。

## 13. 实现灰区与取舍记录

| ID | Question | Options | Recommendation | Pros / Cons | Impact Surface | blocks_lld | Status |
|---|---|---|---|---|---|---|---|
| LCQ-CR037-S10-01 | refresh command 是否默认 dry-run？ | A. 默认 dry-run，显式 `--apply-process` 才写 process artifact；B. 默认应用 process-side updates | 推荐 A | A 安全但多一步；B 自动化高但误写风险更高 | runtime authorization、process writes、tests | false | clarification candidate；默认 A |
| LCQ-CR037-S10-02 | Gate Ledger append 失败时 decision 使用 `BLOCKED` 还是 `FAILED`？ | A. 可恢复的 ledger/schema 问题用 `BLOCKED`，异常崩溃用 `FAILED`；B. 一律 `FAILED` | 推荐 A | A 语义更精确；B 简单但不利于恢复路线 | result schema、QA、ledger | false | clarification candidate；默认 A |

以上不阻断 LLD；CP5 approve 即采用推荐默认值。

## 14. 回滚与发布策略 / Definition of Done

### Rollback

- process-side update 可用 rollback payload 恢复 before content/hash。
- 若 Gate Ledger event 写错，不原地改旧 event；追加 correction/blocked event，并在新 result 中引用。
- 若 target guard 误伤，扩展 process allowlist 需后续 CR 或设计确认；不得放宽到 release repo 自动写。

### Release

- S10 属于 CR037-W4，必须在 S09 result schema/checker 完成后实现。
- 不写 release repo，不写 quant-lab，不读取凭据。
- process artifact 写入需明确在 Story execution 阶段授权；CP5 只确认设计。

### Definition of Done

- process target guard 拒绝 release repo/code/tests/docs/formal docs。
- process-side machine update 记录 before_hash、after_hash、rollback_ref。
- stale release impacts 只进入 S09 result 的 stale/follow-up fields。
- Gate Ledger `roadmap_refresh` event append 成功并通过 event-ledger check。
- ledger append failure 不伪造成 success。
- security fixtures 覆盖 no credential/runtime/publish/production write。

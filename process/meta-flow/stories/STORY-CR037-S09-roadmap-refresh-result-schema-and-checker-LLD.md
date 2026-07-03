---
story_id: "CR037-S09"
story_slug: "roadmap-refresh-result-schema-and-checker"
cr_ref: "CR-037"
title: "roadmap refresh result schema and checker"
design_evidence_type: "full-lld"
status: "draft-for-cp5"
source_story: "process/stories/STORY-CR037-S09-roadmap-refresh-result-schema-and-checker.md"
feature_design_refs:
  - "process/docs/features/roadmap-refresh-governance/DESIGN.md"
  - "process/docs/features/roadmap-refresh-governance/TEST-PLAN.md"
  - "process/docs/features/roadmap-refresh-governance/TASKS.md"
depends_on: ["CR037-S06", "CR037-S07"]
write_scope:
  primary: ["meta_flow/checks/**", "schemas/**"]
  shared: ["process/checks/**", "tests/**"]
  forbidden: ["process/quant-lab/**"]
cp5_design_confirmed: false
---

# CR037-S09 LLD: roadmap refresh result schema and checker

## 1. Goal

CR037-S09 定义 `ROADMAP-REFRESH` 独立 result schema 与 checker，明确 roadmap refresh 不复用 CP result、不调用 CP result checker、不声明发布库自动写入。checker 必须校验 refresh 专属 `decision`、`machine_updates`、`stale_items`、`follow_up_candidates`、`event_refs` 和 forbidden release write。

本 Story 只覆盖 result schema、checker、错误枚举和 CLI 入口；process-only cascade writer 与 Gate Ledger append 由 S10 设计。

## 2. Requirements

### Functional

| ID | Requirement | Source | Verification |
|---|---|---|---|
| S09-FR-01 | `ROADMAP-REFRESH-*.result.json` 使用独立 schema，不复用 CP result。 | ADR-PG-005 | SEC-RF-03、CT-RF-004 |
| S09-FR-02 | decision 至少支持 `NO_CHANGE`、`UPDATED`、`UPDATED_WITH_DOC_IMPACTS`、`BLOCKED`、`FAILED`。 | HLD success criteria | UNIT-RF-01 |
| S09-FR-03 | checker 校验 decision-specific required fields。 | Feature DESIGN | UNIT-RF-02 |
| S09-FR-04 | result 不允许声明发布库 code/tests/docs/formal docs 自动写入。 | Story AC / ADR-PG-003 | SEC-RF-01 |
| S09-FR-05 | result 中 refs 必须 registry-backed，feature/capability refs 经 S07 resolver。 | ADR-PG-004 | CONTRACT-RF-01 |

### Non-Functional

| ID | Requirement | Design |
|---|---|---|
| S09-NFR-01 | 边界清晰 | CP result checker 收到 roadmap refresh result 应失败；roadmap checker 收到 CP result 也失败。 |
| S09-NFR-02 | 可回滚 | `machine_updates` 必须包含 before/after hash 和 rollback_ref，或在 decision 中说明 N/A。 |
| S09-NFR-03 | 可消费 | S10、S11、S12 只消费 `stale_items`、`follow_up_candidates`、`event_refs` 等稳定字段。 |
| S09-NFR-04 | 安全 | checker 在结构层拒绝 release repo write claim，不依赖人工记忆。 |

## 3. 模块拆分与职责

| Module | Path | Responsibility |
|---|---|---|
| roadmap refresh schema constants | `meta_flow/checks/roadmap_refresh.py` | result type、decision enum、required fields、error codes。 |
| schema/checker | `meta_flow/checks/roadmap_refresh.py` | 校验 result JSON 与 decision-specific fields。 |
| type guard | `meta_flow/checks/roadmap_refresh.py` | 拒绝 CP result / checkpoint result / unknown result type。 |
| target guard | `meta_flow/checks/roadmap_refresh.py` | 对 result 中 machine_updates 声明的 target_ref 做 process-only 初步校验。 |
| resolver contract adapter | `meta_flow/checks/roadmap_refresh.py` | 校验 result 中 feature/capability refs 能由 S07 resolver 解析。 |
| CLI dispatch | `meta_flow/cli.py` | 增加 `meta-flow check roadmap-refresh` 或 `meta-flow roadmap refresh-check` 转发。 |

## 4. 代码结构与文件影响范围

| TASK-ID | File / Area | Change | Ownership |
|---|---|---|---|
| TASK-RF-001 | `meta_flow/checks/roadmap_refresh.py` | 新增 result schema validator 和 fixtures。 | primary |
| TASK-RF-002 | `meta_flow/checks/roadmap_refresh.py`、`meta_flow/cli.py` | 独立 checker 与 CLI 转发，wrong-result-type failure。 | primary/shared |
| TASK-RF-005 | `meta_flow/checks/roadmap_refresh.py` | `stale_items` / `follow_up_candidates` contract validation。 | shared |
| TASK-RF-006 | tests | forbidden release write、no credentials/runtime/publish fixtures。 | shared |
| TASK-RF-007 | LLD / tests | CLI 名称和验证命令固化。 | none |

若仓库后续创建 `schemas/`，可把 JSON schema 草案放入 `schemas/roadmap-refresh-result.schema.json`；当前仓库无 `schemas/` 目录，LLD 不要求 CP5 阶段创建。

## 5. 数据模型与持久化设计

### Result schema

```json
{
  "schema_version": 1,
  "result_type": "ROADMAP-REFRESH",
  "result_id": "ROADMAP-REFRESH-20260703T000000Z",
  "created_at": "2026-07-03T00:00:00Z",
  "actor": "host-orchestrator",
  "trigger": {
    "kind": "cr_change",
    "source_ref": "process/changes/CR-037.md",
    "gate_ref": "CP5"
  },
  "decision": "UPDATED_WITH_DOC_IMPACTS",
  "source_refs": ["process/project/ROADMAP.yaml"],
  "registry_refs": {
    "feature_refs": ["FEAT-PG-006"],
    "capability_refs": ["CAP-PG-ROADMAP-REFRESH"]
  },
  "machine_updates": [
    {
      "target_ref": "process/project/ROADMAP.yaml",
      "operation": "update",
      "before_hash": "sha256:...",
      "after_hash": "sha256:...",
      "rollback_ref": "process/checks/ROADMAP-REFRESH-20260703T000000Z.rollback.json"
    }
  ],
  "must_check": [
    "meta-flow check roadmap-refresh --result process/checks/ROADMAP-REFRESH-20260703T000000Z.result.json"
  ],
  "stale_items": [
    {
      "object_ref": "docs/quality/TEST-STRATEGY.md",
      "expected_semantic": "paper readiness tracked",
      "observed_semantic": "backtest-only",
      "severity": "WARN",
      "recommended_follow_up": "FU-RF"
    }
  ],
  "follow_up_candidates": [
    {
      "candidate_id": "FU-RF-001",
      "prefix": "FU-RF",
      "title": "Refresh release docs for roadmap impacts",
      "source_ref": "docs/quality/TEST-STRATEGY.md",
      "reason": "stale semantic"
    }
  ],
  "event_refs": [],
  "errors": []
}
```

### Decision-specific requirements

| Decision | Required | Forbidden / constraints |
|---|---|---|
| `NO_CHANGE` | `source_refs`、empty `machine_updates`、empty or absent `stale_items` | machine_updates 不得非空 |
| `UPDATED` | non-empty `machine_updates`、rollback info、no blocking errors | `stale_items` 可空 |
| `UPDATED_WITH_DOC_IMPACTS` | non-empty `machine_updates` or explicit no-process-update reason；non-empty `stale_items` or `follow_up_candidates` | 不得写 release repo |
| `BLOCKED` | non-empty `errors` with recovery_hint；no applied machine_updates unless marked rejected | 不得声称 success event |
| `FAILED` | non-empty `errors` with exception/failure code | 不得伪造 event success |

### Persistence

Result 默认位于 `process/checks/ROADMAP-REFRESH-<timestamp>.result.json`，summary 可选为同名前缀 `.summary.md`。checker 不修改 result，只输出验证结论。

## 6. API / Interface 设计

| Interface ID | Signature / CLI | Input | Output | Failure |
|---|---|---|---|---|
| S09-IF-01 | `load_refresh_result(path: Path) -> dict` | result path | JSON object | invalid JSON -> `E_RF_JSON_INVALID` |
| S09-IF-02 | `validate_refresh_result(project_root, result, mode="check")` | result object | `(errors, warnings)` | type/decision/field errors |
| S09-IF-03 | `validate_decision_requirements(result)` | result | findings | missing required -> `E_RF_DECISION_FIELDS` |
| S09-IF-04 | `validate_no_release_writes(project_root, result)` | machine_updates | findings | release target -> `E_RF_FORBIDDEN_RELEASE_WRITE` |
| S09-IF-05 | `validate_registry_refs(project_root, result)` | feature/cap refs | findings | S07 unresolved -> `E_REF_UNRESOLVED` |
| S09-IF-06 | `meta-flow check roadmap-refresh --result <path> --project-root .` | CLI args | `Roadmap Refresh Check: OK/FAIL` | errors exit 1 |

错误枚举：

| Code | Trigger | Severity |
|---|---|---|
| `E_RF_WRONG_RESULT_TYPE` | result_type 缺失或不是 `ROADMAP-REFRESH`；CP result 输入 | ERROR |
| `E_RF_UNKNOWN_DECISION` | decision 不在枚举 | ERROR |
| `E_RF_DECISION_FIELDS` | decision-specific required fields 缺失或 forbidden field 非空 | ERROR |
| `E_RF_FORBIDDEN_RELEASE_WRITE` | machine_updates target 指向 release repo code/tests/docs/formal docs | ERROR |
| `E_RF_EVENT_REF_INVALID` | event_refs 非列表或字段不完整 | ERROR |
| `E_RF_STALE_ITEM_INVALID` | stale item 缺 object_ref/expected/observed/severity | ERROR |
| `E_RF_FOLLOW_UP_INVALID` | candidate prefix 不是 FU-RF/SP-RF/RA-RF | ERROR |
| `E_REF_UNRESOLVED` | registry refs 未解析 | BLOCKED/ERROR |

## 7. 核心处理流程

### Checker flow

1. CLI 读取 `--result` JSON。
2. type guard 校验 `result_type == "ROADMAP-REFRESH"`，否则 `E_RF_WRONG_RESULT_TYPE`。
3. 校验顶层必填字段：`schema_version`、`result_id`、`created_at`、`decision`、`source_refs`。
4. 校验 decision-specific required/forbidden fields。
5. 校验 `machine_updates[*].target_ref` 不指向 release repo code/tests/docs/formal docs；S10 会实现更严格 process boundary guard。
6. 校验 `stale_items` 和 `follow_up_candidates` 字段。
7. 调 S07 resolver 校验 `registry_refs.feature_refs/capability_refs`。
8. 校验 `event_refs` 仅为 refs，不要求 S10 event 已存在；若 decision 表示成功且 event_refs 为空，WARN，由 S10 补齐。
9. 输出 OK/FAIL。

### Wrong type flow

1. 输入 CP result JSON。
2. checker 检测 `checkpoint` 或 `checkpoint_id` 字段但无 `result_type=ROADMAP-REFRESH`。
3. 直接 FAIL，提示使用 CP result checker 或 roadmap-refresh checker对应类型。

## 8. 技术设计细节

| Topic | Design |
|---|---|
| CLI naming | 推荐 `meta-flow check roadmap-refresh --result <path>`，与现有 `meta-flow check <validator>` 风格一致。 |
| Schema storage | 先以 Python validator 为机器真相源；若创建 `schemas/`，JSON schema 与 Python validator 必须同测。 |
| Registry refs | result 内集中放 `registry_refs`，stale/follow-up item 可引用 canonical ID 但不强制每项重复。 |
| Event refs | S09 只校验 shape；S10 负责 append event 与 ledger contract。 |
| Release target detection | 结构层拒绝 `target_ref` 指向 `docs/quality`、`docs/release`、`tests`、代码目录等发布库目标；process artifact 目标由 S10 guard 进一步限定。 |
| CP result separation | 不 import `meta_flow.checks.cp_result` 做实际校验；仅识别错误类型并报错。 |

## 9. 安全与性能设计

### Security

- result 不得声明自动修改 release repo code/tests/formal docs。
- result 不得包含 credential、token、account、private key 等字段或明显值。
- checker 不读取凭据、不执行 runtime、不触发 publish。
- `follow_up_candidates` 是候选，不等同授权创建正式 CR 或写 release context。

### Performance

result 单文件应保持轻量，建议小于 64KB。checker 线性遍历字段，无需缓存。若 stale_items 很多，应在 result 中保留摘要和 refs，详细报告另存独立 report；本 Story 不引入大型 report 分页。

## 10. 测试设计

| Test ID | Type | Scenario | Expected |
|---|---|---|---|
| UT-RF-001 | unit | `NO_CHANGE` 且 machine_updates 非空 | FAIL/WARN 按 schema 定义，推荐 FAIL |
| UT-RF-002 | unit | `UPDATED_WITH_DOC_IMPACTS` 缺 stale_items/follow_up_candidates | FAIL |
| UT-RF-003 | unit | process target `process/project/ROADMAP.yaml` | PASS |
| UT-RF-004 | security | target `docs/quality/TEST-STRATEGY.md` | FAIL/BLOCKED |
| UT-RF-005 | contract | CP result JSON 输入 | FAIL wrong result type |
| UNIT-RF-01 | unit | 五类 decision valid fixtures | PASS |
| UNIT-RF-02 | unit | unknown decision / missing fields | FAIL |
| CONTRACT-RF-01 | contract | stale/FU consumer fields | PASS |
| CT-RF-004 | contract | checkpoint-manager 不把 refresh result 当 CP result | PASS |
| SEC-RF-01 | security | release repo auto-write claim | FAIL |
| SEC-RF-03 | security | CP result checker confusion | FAIL wrong type |

接口到测试映射：S09-IF-01 由 invalid JSON fixture 覆盖；S09-IF-02/03 由 UNIT-RF-01/02 覆盖；S09-IF-04 由 UT-RF-004/SEC-RF-01 覆盖；S09-IF-05 由 registry unresolved fixture 覆盖；S09-IF-06 由 CLI smoke test 覆盖。

## 11. 实施步骤

| Step | TASK-ID | Action | Verification |
|---|---|---|---|
| 1 | TASK-RF-001 | 新增 `meta_flow/checks/roadmap_refresh.py`，定义 schema constants 和 validators。 | UNIT-RF-01 |
| 2 | TASK-RF-001 | 添加 valid/invalid fixtures 覆盖五类 decision。 | UT-RF-001..002 |
| 3 | TASK-RF-002 | 实现 wrong-result-type guard 与 CLI 转发。 | UT-RF-005、SEC-RF-03 |
| 4 | TASK-RF-002 | 增加 `meta-flow check roadmap-refresh`。 | CLI smoke |
| 5 | TASK-RF-005 | 校验 stale/follow-up 输出契约。 | CONTRACT-RF-01、CT-RF-001/002 |
| 6 | TASK-RF-006 | 补 forbidden release write 和 credential negative fixtures。 | SEC-RF-01、UT-RF-004 |
| 7 | TASK-RF-007 | 在 CP6 实现证据中记录 CLI 名称和验证命令。 | CP6 self-review |

## 12. 风险、难点与预研建议

| Risk | Impact | Mitigation |
|---|---|---|
| 独立 checker 增加 UX 成本 | 用户要记多个命令 | 顶层 `meta-flow check roadmap-refresh` 明确；未来可聚合但不合并 schema |
| release target 判断误伤 process docs | 阻断合法 process artifact | S10 进一步定义 process boundary；S09 保守校验 obvious release write claim |
| `UPDATED_WITH_DOC_IMPACTS` required fields 不清 | stale/FU 噪音 | CP5 默认要求 stale_items 或 follow_up_candidates 至少一类非空 |
| registry refs 初期缺失 | checker blocked | S09 depends_on S07；提供 recovery hint |

预研建议：实现前查看 `meta_flow/checks/cp_result.py` 的 result-check CLI 风格，仅复用打印和错误汇总习惯，不复用 CP result schema。

## 13. 实现灰区与取舍记录

| ID | Question | Options | Recommendation | Pros / Cons | Impact Surface | blocks_lld | Status |
|---|---|---|---|---|---|---|---|
| LCQ-CR037-S09-01 | roadmap-refresh checker CLI 名称是否采用 `meta-flow check roadmap-refresh`？ | A. `meta-flow check roadmap-refresh`；B. `meta-flow roadmap refresh-check`；C. `meta-flow project roadmap-refresh check` | 推荐 A | A 与现有 check dispatch 一致；B/C 子域更清楚但需新增命令层 | CLI、docs、tests | false | clarification candidate；默认 A |
| LCQ-CR037-S09-02 | `UPDATED_WITH_DOC_IMPACTS` 是否允许无 process machine update？ | A. 允许，但必须写 explicit no-process-update reason 且有 stale/follow-up；B. 必须同时有 process machine update | 推荐 A | A 覆盖只发现文档陈旧的场景；B 更严格但可能误阻断 | result schema、checker、S10 cascade | false | clarification candidate；默认 A |

以上不阻断 LLD；CP5 approve 即采用推荐默认值。

## 14. 回滚与发布策略 / Definition of Done

### Rollback

- 若 checker 误伤，先降级为 WARN 但保留 `E_RF_WRONG_RESULT_TYPE` 和 forbidden release write ERROR。
- 若 CLI 名称需要调整，保留旧入口 alias 一轮，避免 CP7 命令失效。
- result 文件可删除重建；checker 不修改 result。

### Release

- S09 属于 CR037-W4，依赖 S06 project objects 和 S07 registry resolver。
- S10 process-only cascade 必须消费 S09 result schema，不得自行定义第二套 result。
- CP5 前不实现、不写 result、不写 ledger。

### Definition of Done

- 独立 `ROADMAP-REFRESH` result validator 存在。
- 五类 decision 的 required/forbidden fields 有 fixture 覆盖。
- CP result 输入被 roadmap-refresh checker 明确拒绝。
- release repo auto-write claim 被 checker 拒绝。
- stale_items / follow_up_candidates contract 可被 S11/S12 消费。
- registry_refs 通过 S07 resolver 或输出 blocked finding。

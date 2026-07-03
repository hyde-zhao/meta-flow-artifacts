---
story_id: "CR037-S08"
story_slug: "impact-surface-field-split-and-migration-report"
cr_ref: "CR-037"
title: "impact surface field split and migration report"
design_evidence_type: "full-lld"
status: "draft-for-cp5"
source_story: "process/stories/STORY-CR037-S08-impact-surface-field-split-and-migration-report.md"
feature_design_refs:
  - "process/docs/features/impact-surface-normalization/DESIGN.md"
  - "process/docs/features/impact-surface-normalization/TEST-PLAN.md"
  - "process/docs/features/impact-surface-normalization/TASKS.md"
depends_on: ["CR037-S07"]
write_scope:
  primary: ["meta_flow/workflow/cr_lifecycle.py", "meta_flow/checks/cr_tracking.py"]
  shared: ["tests/**", "process/changes/**"]
  forbidden: ["process/quant-lab/**"]
cp5_design_confirmed: false
---

# CR037-S08 LLD: impact surface field split and migration report

## 1. Goal

CR037-S08 将 CR 的混合 `impact_surface` 拆分为四类可校验语义：`impact_surfaces[]`、`affected_paths[]`、`feature_refs[]`、`capability_refs[]`。新 CR 必须使用有限治理面枚举，路径只进入 `affected_paths`，feature/capability 只通过 S07 resolver 写 canonical refs。历史漂移不在普通 `cr check` 中刷屏，而由 migration report 聚合输出。

本 Story 不自动重写历史 CR，不修改 `process/quant-lab/**`，不把 `affected_paths` 解释为写授权。

## 2. Requirements

### Functional

| ID | Requirement | Source | Verification |
|---|---|---|---|
| S08-FR-01 | 新 CR 支持 normalized impact block：`impact_surfaces`、`affected_paths`、`feature_refs`、`capability_refs`。 | Story AC / ISN-TASK-001 | ISN-UNIT-01 |
| S08-FR-02 | `impact_surfaces` 使用有限治理面枚举，unknown 在 audit WARN、enforce ERROR。 | HLD success criteria | ISN-UNIT-02、ISN-INT-01/02 |
| S08-FR-03 | `feature_refs` / `capability_refs` 必须调用 S07 resolver。 | ADR-PG-004 | ISN-CON-01 |
| S08-FR-04 | legacy `impact_surface` 只生成 migration report，不改源 CR，不让普通检查对历史刷屏。 | HLD R-PG-005 | ISN-INT-03、ISN-SEC-04 |
| S08-FR-05 | migration report 对无法分类项输出 blocked finding 和 recovery hint。 | Feature DESIGN | ISN-MAN-01 |

### Non-Functional

| ID | Requirement | Design |
|---|---|---|
| S08-NFR-01 | 兼容性 | 旧 `impact_surface` 只读兼容；新 CR enforce 阶段禁止继续写混合字段。 |
| S08-NFR-02 | 安全 | `affected_paths` 不授予写权限；包含 approval/runtime write 语义时 security finding。 |
| S08-NFR-03 | 可审计 | migration report 含 counts、source_cr、legacy_value、suggestions、severity、source hash。 |
| S08-NFR-04 | 低噪音 | 普通 CR tracking check 不因历史漂移产生大量 blocker。 |

## 3. 模块拆分与职责

| Module | Path | Responsibility |
|---|---|---|
| CR frontmatter parser | `meta_flow/workflow/cr_lifecycle.py` | 继续读取 legacy `impact_surface`，新增 normalized field parsing。 |
| CRRecord model | `meta_flow/workflow/cr_lifecycle.py` | 增加 `impact_surfaces`、`affected_paths`、`feature_refs`、`capability_refs`。 |
| index writer | `meta_flow/workflow/cr_lifecycle.py` | CR index 同步 normalized fields；legacy `impact_surface` 标记兼容。 |
| CR tracking checker | `meta_flow/checks/cr_tracking.py` | audit/enforce mode 校验 normalized fields、enum、路径和 refs。 |
| migration scanner/report | `meta_flow/checks/cr_tracking.py` 或同模块新函数 | 扫描 legacy 字段并输出 report，不改源 CR。 |
| resolver adapter | `meta_flow/checks/cr_tracking.py` | 调用 `meta_flow.design.feature_registry.resolve_refs()`。 |

## 4. 代码结构与文件影响范围

| TASK-ID | File / Area | Change | Ownership |
|---|---|---|---|
| ISN-TASK-001 | `meta_flow/workflow/cr_lifecycle.py` | 数据模型和 summary/index 输出加入 normalized fields。 | primary |
| ISN-TASK-002 | `meta_flow/checks/cr_tracking.py` | 定义 impact surface enum 和扩展规则。 | primary |
| ISN-TASK-003 | `meta_flow/checks/cr_tracking.py` | 接入 S07 resolver 校验 refs。 | shared with S07 contract |
| ISN-TASK-004 | `meta_flow/checks/cr_tracking.py` | 新 CR audit/enforce checker。 | primary |
| ISN-TASK-005 | `meta_flow/checks/cr_tracking.py` | migration scanner/report writer；断言源 CR hash 不变。 | primary |
| ISN-TASK-006 | `meta_flow/cli.py` check dispatch / existing check alias | 暴露或复用 `meta-flow check cr-tracking` 的 mode/report 参数。 | shared |
| ISN-TASK-007 | tests / CP5 manual evidence | unclassified legacy review checklist。 | shared |
| ISN-TASK-008 | Story/CP evidence | 下游 Story 只引用本 LLD，不改 Story 卡片。 | none in this task |

## 5. 数据模型与持久化设计

### CR normalized impact block

```yaml
impact_surfaces:
  - changes
  - registry
affected_paths:
  - meta_flow/workflow/cr_lifecycle.py
feature_refs:
  - FEAT-PG-005
capability_refs:
  - CAP-PG-IMPACT-NORMALIZATION
```

`impact_surface` legacy 字段只读兼容，不作为新 CR canonical field。`record_from_cr_file()` 应同时读取新旧字段，但 `summary_from_cr_file()` / `build_index()` 优先输出新字段，并可保留 `legacy_impact_surface` 供 migration report 使用。

### Impact surface enum baseline

初始枚举至少包含：

`state`、`context`、`project-governance`、`roadmap`、`changes`、`registry`、`quality`、`release`、`security`、`runtime-authorization`、`docs`、`agent-contract`、`workflow`、`migration`。

新增枚举必须通过后续 CR 或 CP5 决策，不允许实现阶段自由扩展。

### Migration report

```json
{
  "schema_version": 1,
  "report_type": "impact-surface-migration",
  "generated_at": "ISO-8601",
  "mode": "audit",
  "source_refs": ["process/changes/CR-001.md"],
  "summary": {
    "scanned_cr_count": 1,
    "legacy_field_count": 1,
    "blocked_count": 0,
    "unclassified_count": 0
  },
  "items": [
    {
      "source_cr": "CR-001",
      "source_ref": "process/changes/CR-001.md",
      "source_hash": "sha256:...",
      "legacy_value": "meta_flow/checks",
      "suggested_impact_surfaces": ["workflow"],
      "suggested_affected_paths": ["meta_flow/checks/**"],
      "suggested_feature_refs": [],
      "suggested_capability_refs": [],
      "finding_severity": "WARN",
      "finding_code": "LEGACY_IMPACT_CLASSIFIED",
      "recovery_hint": "review and update in a future CR if needed"
    }
  ]
}
```

Report 建议输出到 `process/checks/CR-IMPACT-MIGRATION.report.json` 或调用方指定路径。普通 `cr check` 只输出 report ref / counts，不复制完整历史 CR。

## 6. API / Interface 设计

| Interface ID | Signature / CLI | Input | Output | Error model |
|---|---|---|---|---|
| S08-IF-01 | `parse_impact_fields(fields: dict[str,str])` | CR frontmatter | normalized impact object + legacy values | invalid list -> `E_IMPACT_FIELD_INVALID` |
| S08-IF-02 | `validate_impact_block(block, mode, project_root)` | normalized block | findings list | unknown enum / invalid path / unresolved ref |
| S08-IF-03 | `scan_legacy_impact(project_root, cr_paths, registry_snapshot)` | CR paths | migration report payload | unclassified -> blocked finding |
| S08-IF-04 | `write_impact_migration_report(project_root, report, output_path)` | report payload | report path | path write failure -> fail without source rewrite |
| S08-IF-05 | `meta-flow check cr-tracking --mode audit|enforce --impact-report <path>` | CLI args | summary + report | enforce errors exit 1 |

错误码：

| Code | Trigger | Behavior |
|---|---|---|
| `E_UNKNOWN_IMPACT_SURFACE` | surface 不在枚举 | audit WARN / enforce ERROR |
| `E_INVALID_AFFECTED_PATH` | 空路径、绝对危险路径、路径穿越、非字符串 | ERROR |
| `E_AFFECTED_PATH_AUTHZ_MISUSE` | 路径字段含 approval/write authorization 语义 | security finding |
| `E_REF_UNRESOLVED` | S07 resolver unresolved | blocked |
| `E_REF_DEPRECATED` | S07 resolver deprecated | audit WARN / enforce ERROR |
| `E_LEGACY_IMPACT_UNCLASSIFIED` | legacy value 无法分类 | migration report blocked |
| `E_REGISTRY_BLOCKED` | S07 registry invalid | refs validation blocked |

## 7. 核心处理流程

### 新 CR audit/enforce

1. `cr_lifecycle.record_from_cr_file()` 解析 normalized fields 和 legacy `impact_surface`。
2. `cr_tracking` 对新字段执行 schema check。
3. 校验 `impact_surfaces` 是否在 baseline enum。
4. 校验 `affected_paths` 是相对路径/glob，且不含授权语义。
5. 调 S07 resolver 校验 `feature_refs` 和 `capability_refs`。
6. audit 模式输出 WARN/BLOCKED summary；enforce 模式 unknown/unresolved 升级 ERROR。
7. 输出 findings；不修改 CR 文件。

### Legacy migration report

1. 扫描 `process/changes/CR-*.md`，排除 follow-up tracking 文档。
2. 对含 legacy `impact_surface` 且缺 normalized fields 的 CR 计算 source hash。
3. 按规则分类路径、治理面、feature/capability hints。
4. 对 refs 调 resolver；unresolved 进入 blocked finding。
5. 写 migration report；断言源 CR hash 不变。
6. 普通 check 只显示 report 路径和 counts，避免刷屏。

## 8. 技术设计细节

| Topic | Design |
|---|---|
| Mode | `audit` 默认 WARN，`enforce` 对新 CR unknown/unresolved ERROR；历史 migration report 不因旧数据直接让普通 check FAIL。 |
| New vs legacy | 若 CR 同时含 legacy 和 normalized fields，checker WARN 要求移除 legacy 或说明兼容原因。 |
| Path validation | 允许相对路径和 glob；拒绝空值、`..` 路径穿越、绝对路径、URL、含 credential 字样路径。 |
| Classification | legacy value 若包含 `/`、`.` 或 glob 形态优先进入 `affected_paths`；命中 enum alias 进入 surface；其余作为 feature/capability hint 调 resolver。 |
| Report stability | report item 排序按 `source_cr` + `legacy_value`；source hash 用 sha256 文本摘要。 |
| CLI | 优先扩展 `meta-flow check cr-tracking`，不新增过多顶层命令。 |

## 9. 安全与性能设计

### Security

- `affected_paths` 不是授权字段；出现 `approve`、`authorization`、`write allowed`、`runtime` 授权语义时输出 security finding。
- migration scanner 只读 CR 文件，写 report；不得改写历史 CR。
- unresolved refs 不写 canonical fields。
- 不读取或写入 `process/quant-lab/**`。

### Performance

扫描 CR 数量预计较小。实现可逐文件读取并计算 hash；report items 按需写一次。若历史 CR 超过数千个，可后续 CR 引入增量缓存；S08 不设计缓存。

## 10. 测试设计

| Test ID | Type | Scenario | Expected |
|---|---|---|---|
| ISN-UNIT-01 | unit | 四字段合法 block | PASS |
| ISN-UNIT-02 | unit | unknown enum | audit WARN / enforce ERROR |
| ISN-UNIT-03 | security | invalid path / 授权语义 | ERROR/security finding |
| ISN-UNIT-04 | contract | unresolved feature/capability ref | blocked finding |
| ISN-UNIT-05 | unit | legacy value 可分类为 surface + path | report suggestions |
| ISN-INT-01 | integration | 新 CR audit check | no hard fail for unknown surface |
| ISN-INT-02 | integration | 新 CR enforce check | exit 1 on unknown surface |
| ISN-INT-03 | integration/security | legacy scan | report generated, source hash unchanged |
| ISN-INT-04 | integration | registry invalid | refs blocked, surface/path findings still emitted |
| ISN-CON-01 | contract | resolver adapter used | no free-string refs written |
| ISN-CON-02 | contract | change-impact-analysis consumes normalized schema | legacy write WARN/ERROR |
| ISN-CON-03 | contract | CP checker consumes findings summary only | no full CR copy |

异常路径映射：S08-IF-02 的 enum/path/ref errors 分别由 ISN-UNIT-02/03/04 覆盖；S08-IF-03 的 unclassified legacy 由 ISN-MAN-01 和 integration fixture 覆盖；S08-IF-04 的 no-source-rewrite 由 ISN-INT-03 覆盖。

## 11. 实施步骤

| Step | TASK-ID | Action | Verification |
|---|---|---|---|
| 1 | ISN-TASK-001 | 扩展 `CRRecord` 与 parse/summary/index 输出 normalized fields。 | ISN-UNIT-01 |
| 2 | ISN-TASK-002 | 在 `cr_tracking.py` 定义 enum 和扩展策略。 | ISN-UNIT-02、ISN-MAN-02 |
| 3 | ISN-TASK-003 | 接入 S07 resolver adapter。 | ISN-CON-01、ISN-SEC-02 |
| 4 | ISN-TASK-004 | 实现 audit/enforce checker。 | ISN-INT-01、ISN-INT-02 |
| 5 | ISN-TASK-005 | 实现 legacy migration scanner/report 和 hash-stability 断言。 | ISN-INT-03、ISN-SEC-04 |
| 6 | ISN-TASK-006 | 建立 downstream contract tests。 | ISN-CON-02、ISN-CON-03 |
| 7 | ISN-TASK-007 | 输出 unclassified legacy manual review checklist。 | ISN-MAN-01 |

## 12. 风险、难点与预研建议

| Risk | Impact | Mitigation |
|---|---|---|
| 历史 legacy 值无法自动分类 | migration report blocked 项多 | 不阻断普通 check；人工回修队列处理 |
| enum 过窄 | 新 CR 无法表达合法影响面 | CP5 确认 baseline；新增 enum 走 CR |
| S07 resolver 未就绪 | refs 校验无法 enforce | S08 depends_on S07；实现时先做 contract adapter，registry invalid 时 blocked |
| `affected_paths` 被误用为授权 | 越权风险 | security fixture + 文案明确非授权 |

预研建议：实现前抽样现有 `process/changes/*.md` 的 legacy `impact_surface` 形态，仅用于设计 classifier fixtures；不要在 S08 实现中批量改写旧 CR。

## 13. 实现灰区与取舍记录

| ID | Question | Options | Recommendation | Pros / Cons | Impact Surface | blocks_lld | Status |
|---|---|---|---|---|---|---|---|
| LCQ-CR037-S08-01 | baseline enum 是否允许 CP5 后由实现者追加？ | A. 不允许，新增 enum 必须走 CR；B. 允许实现阶段追加并补测试 | 推荐 A | A 可控但可能需要 follow-up；B 快但语义漂移风险高 | CR schema、checker、docs | false | clarification candidate；默认 A |
| LCQ-CR037-S08-02 | migration report 默认输出路径是否固定为 `process/checks/CR-IMPACT-MIGRATION.report.json`？ | A. 固定默认路径并允许 CLI override；B. 每次按 timestamp 生成 | 推荐 A | A 稳定便于 CP5/CP7 引用；B 保留历史多版本但增加清理成本 | process/checks、QA evidence | false | clarification candidate；默认 A |

以上问题不阻断 LLD；CP5 approve 即采用推荐默认值。

## 14. 回滚与发布策略 / Definition of Done

### Rollback

- 如 normalized checker 误伤，可将 `mode` 降回 audit；不得恢复自由字符串作为 canonical 语义。
- 如 migration report 分类错误，删除 report 并重跑，不改源 CR。
- 如 resolver contract 不稳定，暂时只 enforce `impact_surfaces` 和 `affected_paths`，refs 校验保持 blocked/WARN 直到 S07 修复。

### Release

- S08 进入 CR037-W3，必须在 S07 resolver API 可用后实现。
- CP5 前不实现，不改 `process/changes/**` 样本。
- 不授权发布库、runtime、credentials 或 `process/quant-lab/**`。

### Definition of Done

- 新 CR normalized fields schema 可校验。
- `impact_surfaces` baseline enum 和 audit/enforce 行为有测试。
- `feature_refs` / `capability_refs` 必须经 S07 resolver。
- legacy migration report 可生成且源 CR hash 不变。
- 普通 `cr check` 不因历史 drift 刷屏 blocker。
- `affected_paths` 非授权语义有 security negative test。

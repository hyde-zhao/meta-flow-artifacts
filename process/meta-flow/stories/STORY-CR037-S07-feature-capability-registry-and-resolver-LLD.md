---
story_id: "CR037-S07"
story_slug: "feature-capability-registry-and-resolver"
cr_ref: "CR-037"
title: "feature/capability registry and resolver"
design_evidence_type: "full-lld"
status: "draft-for-cp5"
source_story: "process/stories/STORY-CR037-S07-feature-capability-registry-and-resolver.md"
feature_design_refs:
  - "process/docs/features/capability-feature-registry/DESIGN.md"
  - "process/docs/features/capability-feature-registry/TEST-PLAN.md"
  - "process/docs/features/capability-feature-registry/TASKS.md"
depends_on: ["CR037-S05"]
write_scope:
  primary: ["meta_flow/design/**", "docs/design/FEATURE-REGISTRY.yaml"]
  shared: ["tests/**"]
  forbidden: ["process/quant-lab/**"]
cp5_design_confirmed: false
---

# CR037-S07 LLD: feature/capability registry and resolver

## 1. Goal

CR037-S07 建立 registry-backed refs 的机器契约，使 `feature_refs` 和 `capability_refs` 不再依赖自由字符串、Markdown 表格或 Python 常量。实现阶段必须在现有 `meta_flow/design/feature_registry.py` 基础上扩展 feature registry，并新增 capability registry 与 resolver，使下游 S08 impact normalization、S09/S10 roadmap refresh、S12 stale-check 和 S13 migration readiness 只消费 canonical refs 或明确的 blocked finding。

本 Story 只设计 registry schema、loader/checker、resolver API、candidate report 和下游消费契约。不执行实现、不初始化真实 registry 内容、不读取或修改 `process/quant-lab/**`。

## 2. Requirements

### Functional

| ID | Requirement | Source | Verification |
|---|---|---|---|
| S07-FR-01 | `FEATURE-REGISTRY.yaml` 是 feature refs 的机器真相源，canonical ID 唯一。 | ADR-PG-004 / CFR-TASK-001 | CFR-UT-01、CFR-UT-04 |
| S07-FR-02 | 新增 `CAPABILITY-REGISTRY.yaml`，capability refs 必须引用 canonical capability ID。 | FEAT-PG-004 DESIGN | CFR-UT-02 |
| S07-FR-03 | resolver 返回 `resolved`、`unresolved`、`deprecated`、`conflict` 四类结果。 | Story AC | CFR-UT-03、CFR-UNIT-05 |
| S07-FR-04 | 未注册 ID 不自动创建 canonical ID，只进入 blocked finding 或 candidate report。 | Story AC / CFR-SEC-01 | CFR-SEC-01、CFR-SEC-04 |
| S07-FR-05 | 下游 impact、roadmap、stale、migration consumer 必须经 resolver 读取 registry-backed refs。 | Story AC / CFR-CON-01..03 | contract tests |

### Non-Functional

| ID | Requirement | Design |
|---|---|---|
| S07-NFR-01 | 可审计 | checker 输出稳定 finding code、severity、input ref、source path。 |
| S07-NFR-02 | 安全 | registry 不保存凭据、token、生产数据样本；外部样本只能生成 candidate report。 |
| S07-NFR-03 | 兼容 | 现有 `meta-flow feature build/check/trace` 入口保持可用；新增 capability 能力不破坏已有 feature tests。 |
| S07-NFR-04 | 可维护 | ID、alias、deprecated_by 和 owner/source_refs 必填或可校验。 |

## 3. 模块拆分与职责

| Module | Path | Responsibility | Notes |
|---|---|---|---|
| registry paths/constants | `meta_flow/design/feature_registry.py` | 定义 `FEATURE_REGISTRY_REL`、新增 `CAPABILITY_REGISTRY_REL`、允许状态和错误码。 | 复用现有 argparse 风格。 |
| registry loader | `meta_flow/design/feature_registry.py` | 读取 JSON-compatible YAML，返回 immutable-ish snapshot。 | 读取失败返回结构化错误，不吞掉 invalid JSON/YAML。 |
| registry checker | `meta_flow/design/feature_registry.py` | 校验 schema、唯一性、alias collision、source refs、敏感字段。 | `meta-flow feature check` 扩展。 |
| resolver API | `meta_flow/design/feature_registry.py` | 解析 feature/capability refs，输出 typed result 和 findings。 | 下游直接 import 调用。 |
| candidate report builder | `meta_flow/design/feature_registry.py` | 对 unresolved hints 生成 candidate report payload，不写 canonical registry。 | S13 可消费。 |
| CLI dispatch | `meta_flow/cli.py` | 顶层已支持 `feature` 命令；S07 仅扩展子命令。 | 不引入新 CLI 框架。 |

## 4. 代码结构与文件影响范围

| TASK-ID | File / Area | Change | Ownership |
|---|---|---|---|
| CFR-TASK-001 | `meta_flow/design/feature_registry.py` | 扩展 feature schema 校验，兼容现有 `feature_id`，新增 canonical `id` 读取。 | primary |
| CFR-TASK-002 | `meta_flow/design/feature_registry.py` | 新增 capability registry schema、loader/checker。 | primary |
| CFR-TASK-003 | `meta_flow/design/feature_registry.py` | 新增 checker finding model 与 sensitive field scan。 | primary |
| CFR-TASK-004 | `meta_flow/design/feature_registry.py` | 新增 `resolve_refs()` / `resolve_ref()` API。 | primary |
| CFR-TASK-005 | `tests/test_feature_registry.py` 或新增同域测试 | impact consumer contract fixture。 | shared |
| CFR-TASK-006 | `tests/test_feature_registry.py` 或新增同域测试 | roadmap/stale/migration consumer contract fixture。 | shared |
| CFR-TASK-007 | `meta_flow/design/feature_registry.py` | candidate report 数据构造。 | primary |
| CFR-TASK-008 | CP5 / QA 手工材料 | 初始 registry ID、owner、source refs 审查清单。 | shared |

输出 registry 默认路径延续现有代码：`docs/design/FEATURE-REGISTRY.yaml`。新增 capability registry 为 `docs/design/CAPABILITY-REGISTRY.yaml`。`process/docs/...` 是本轮设计证据位置，不改变代码默认项目路径。

## 5. 数据模型与持久化设计

### Feature Registry

```json
{
  "schema_version": 2,
  "features": [
    {
      "id": "FEAT-PG-004",
      "feature_id": "FEAT-PG-004",
      "title": "Capability / Feature Registry",
      "status": "planned",
      "owner_context": "project-governance",
      "risk_profile": "standard-code",
      "design_doc_policy": "full-design",
      "design_doc": "process/docs/features/capability-feature-registry/DESIGN.md",
      "test_plan": "process/docs/features/capability-feature-registry/TEST-PLAN.md",
      "tasks_doc": "process/docs/features/capability-feature-registry/TASKS.md",
      "module_paths": ["meta_flow/design/feature_registry.py"],
      "aliases": ["capability-feature-registry"],
      "deprecated_by": "",
      "source_refs": ["ADR-PG-004", "CR037-S07"]
    }
  ]
}
```

兼容策略：`schema_version=1` 继续按现有 feature registry 校验；`schema_version=2` 开启 `aliases`、`deprecated_by`、`source_refs` 和 capability cross-check。实现不得删除现有 v1 字段。

### Capability Registry

```json
{
  "schema_version": 1,
  "capabilities": [
    {
      "id": "CAP-PG-REGISTRY-REFS",
      "name": "Registry-backed refs",
      "domain": "project-governance",
      "status": "planned",
      "owner_context": "project-governance",
      "feature_refs": ["FEAT-PG-004", "FEAT-PG-005"],
      "aliases": ["registry refs"],
      "deprecated_by": "",
      "source_refs": ["ADR-PG-004", "HLD UC-PG-004"]
    }
  ]
}
```

### Resolver Result

```json
{
  "input_ref": "registry refs",
  "kind": "capability",
  "status": "resolved",
  "canonical_id": "CAP-PG-REGISTRY-REFS",
  "severity": "INFO",
  "code": "REF_RESOLVED",
  "message": "resolved by alias",
  "source": "alias"
}
```

状态枚举：

| Resolver status | Meaning | Default severity |
|---|---|---|
| `resolved` | canonical ID 或唯一 alias 命中 active/proposed/planned object。 | INFO |
| `unresolved` | 无 ID/alias 命中。 | BLOCKED |
| `deprecated` | 命中 deprecated object。 | WARN in audit, ERROR in enforce |
| `conflict` | alias 或 ID 映射多对象。 | ERROR |

持久化只发生在 registry YAML 和 candidate report。resolver 本身不修改 registry。

## 6. API / Interface 设计

| Interface ID | Signature / CLI | Input | Output | Failure |
|---|---|---|---|---|
| S07-IF-01 | `load_registry(project_root: Path, kind: Literal["feature","capability"])` | project root、kind | registry snapshot dict | missing -> `E_REGISTRY_MISSING` |
| S07-IF-02 | `validate_registry(project_root: Path, include_capabilities: bool = True)` | project root | `(errors, warnings)` | schema invalid、duplicate、alias collision |
| S07-IF-03 | `resolve_ref(project_root, ref, kind, mode="audit")` | ref、kind、mode | `ResolverResult` | unresolved/deprecated/conflict typed result |
| S07-IF-04 | `resolve_refs(project_root, refs, kind, mode="audit")` | refs list | result list + summary | registry invalid -> blocked result for all refs |
| S07-IF-05 | `build_candidate_report(project_root, hints, kind, source_ref)` | unresolved hints | candidate report payload/path | no canonical write |
| S07-IF-06 | `meta-flow feature check --project-root . --include-capabilities` | CLI args | human summary + exit code | errors exit 1 |
| S07-IF-07 | `meta-flow feature resolve --kind feature|capability --ref <value>` | CLI args | compact resolution line / JSON optional | unresolved exit non-zero only in enforce |

结构化错误码：

| Code | Trigger | Consumer action |
|---|---|---|
| `E_REGISTRY_MISSING` | registry 文件缺失 | 下游 blocked，不猜测 |
| `E_REGISTRY_SCHEMA_INVALID` | schema_version、字段类型或必填字段错误 | 下游 blocked |
| `E_REF_UNRESOLVED` | ref 未注册 | candidate report / blocked finding |
| `E_REF_DEPRECATED` | ref 指向 deprecated object | audit WARN，enforce ERROR |
| `E_REF_CONFLICT` | alias/ID 多重匹配 | ERROR，人工处理 |
| `E_SENSITIVE_VALUE` | registry 出现 credential-like 字段或值 | ERROR |

## 7. 核心处理流程

### Registry check

1. 定位 `docs/design/FEATURE-REGISTRY.yaml` 和 `docs/design/CAPABILITY-REGISTRY.yaml`。
2. 读取 JSON-compatible YAML。
3. 校验 schema_version、顶层列表、必填字段、状态枚举。
4. 建立 canonical ID map 和 alias map。
5. 检查 duplicate ID、alias collision、deprecated_by 指向、capability `feature_refs` 是否能解析到 feature registry。
6. 扫描 credential-like key/value。
7. 输出 errors/warnings；errors 非空时 exit 1。

### Resolver

1. 若 registry check 有 blocker，返回 `E_REGISTRY_SCHEMA_INVALID` blocked result。
2. 对每个 input ref 先按 canonical ID 查找，再按 alias 查找。
3. 无命中返回 `unresolved`；多命中返回 `conflict`。
4. 命中 deprecated object 时返回 `deprecated`，附带 `deprecated_by`。
5. mode=`enforce` 时 deprecated severity 为 ERROR；mode=`audit` 为 WARN。
6. 返回 typed results，不写任何文件。

### Candidate report

1. 接收 unresolved hints、source_ref 和 kind。
2. 去重并记录原始输入、建议 kind、source path、reason。
3. 写入下游指定 report 或返回 payload。
4. 等人工确认后，由后续 CR 明确修改 registry；S07 不自动写 canonical ID。

## 8. 技术设计细节

| Topic | Design |
|---|---|
| YAML parsing | 沿用当前 JSON-compatible YAML 策略；若未来需要完整 YAML，另行引入依赖并更新测试，不在 S07 默默切换。 |
| Backward compatibility | `feature_id` 与 `id` 都可识别；输出 canonical 使用 `id`，旧 registry 仍通过。 |
| Alias rules | alias 全 registry 唯一；同 kind 内 alias collision 为 ERROR，跨 kind alias collision 为 WARN，resolver 需指定 kind。 |
| Status rules | 推荐 statuses：`proposed`、`planned`、`active`、`implemented`、`deprecated`、`blocked`、`deferred`、`not-authorized`。实现需保留现有 `ALLOWED_FEATURE_STATUSES` 兼容。 |
| Finding shape | 先使用 dataclass / dict，不强制新增全局 schema；S08/S09/S10 通过 code/status/severity 消费。 |
| CLI UX | 现有 `meta-flow feature` 命令扩展子命令；`meta-flow check design-ownership` 继续转发到 `feature check`。 |

## 9. 安全与性能设计

### Security

- resolver 不得根据自由文本自动创建 canonical ID。
- candidate report 不得写入 canonical registry。
- registry checker 对 key/value 中的 `token`、`secret`、`password`、`api_key`、`private_key` 等 credential-like 内容报错。
- `process/quant-lab/**` 在本 Story 为 forbidden，不作为 registry 初始化输入。
- `affected_paths`、feature/capability refs 只表达影响关系，不表达写授权。

### Performance

registry 规模预计小于数百项。实现可每次加载并构建内存 map，无需缓存。若未来 registry 超过 1MB 或 resolver 成为热点，再通过独立 CR 评估缓存；当前 LLD 不引入缓存复杂度。

## 10. 测试设计

| Test ID | Type | Scenario | Expected |
|---|---|---|---|
| CFR-UNIT-01 | unit | 最小合法 feature registry v1/v2 | PASS |
| CFR-UNIT-02 | unit | 最小合法 capability registry | PASS |
| CFR-UNIT-03 | unit | unknown feature/capability ref | `E_REF_UNRESOLVED` blocked |
| CFR-UNIT-04 | unit | alias collision | checker ERROR |
| CFR-UNIT-05 | unit | deprecated ref audit/enforce | audit WARN / enforce ERROR |
| CFR-INT-01 | integration | loader + checker 读取完整 registry | PASS |
| CFR-INT-02 | contract | S08 synthetic impact consumer 调 resolver | resolved 写 canonical，unresolved blocked |
| CFR-INT-03 | contract | S09/S12 synthetic roadmap/stale consumer 调 resolver | unresolved blocks dependent check |
| CFR-SEC-01 | security | resolver 遇到自由字符串 | 不创建 ID |
| CFR-SEC-02 | security | registry 包含 credential-like 字段 | checker ERROR |
| CFR-SEC-04 | security | external sample hint | candidate report only |
| CFR-MAN-01..03 | manual | 初始 ID、owner、alias/deprecated 审查 | 人工确认 |

接口到测试映射：S07-IF-01/02 由 CFR-UNIT-01/02/04、CFR-INT-01 覆盖；S07-IF-03/04 由 CFR-UNIT-03/05、CFR-INT-02/03 覆盖；S07-IF-05 由 CFR-SEC-04 覆盖。异常路径 `E_REGISTRY_MISSING`、`E_REGISTRY_SCHEMA_INVALID` 需 negative fixture 覆盖。

## 11. 实施步骤

| Step | TASK-ID | Action | Verification |
|---|---|---|---|
| 1 | CFR-TASK-001 | 扩展 feature registry schema v2，保留 v1 兼容。 | CFR-UNIT-01、现有 `tests/test_feature_registry.py` |
| 2 | CFR-TASK-002 | 增加 capability registry path、schema 和校验。 | CFR-UNIT-02、CFR-UNIT-04 |
| 3 | CFR-TASK-003 | 统一 finding model，补 duplicate/alias/sensitive checks。 | CFR-INT-01、CFR-SEC-02 |
| 4 | CFR-TASK-004 | 实现 resolver API 和 mode-aware severity。 | CFR-UNIT-03、CFR-UNIT-05 |
| 5 | CFR-TASK-007 | 实现 candidate report builder，不写 registry。 | CFR-SEC-01、CFR-SEC-04 |
| 6 | CFR-TASK-005/006 | 增加 synthetic consumer contract tests。 | CFR-CON-01..03 |
| 7 | CFR-TASK-008 | 准备人工审查摘要。 | CFR-MAN-01..03 |

## 12. 风险、难点与预研建议

| Risk | Impact | Mitigation |
|---|---|---|
| 初始 registry 内容不完整 | 下游 migration blocked findings 增多 | candidate report + CP5 人工确认 + audit 期补齐 |
| 现有 feature registry 已使用 `feature_id` | 直接切换 `id` 可能破坏 tests | v1/v2 兼容，canonical 输出同时识别 `id`/`feature_id` |
| alias 语义过宽 | resolver conflict 多 | alias 必须全局唯一；冲突 ERROR |
| 下游绕过 resolver | refs 再次自由化 | S08/S09/S10/S12/S13 contract tests 强制消费 resolver |

预研建议：实现前检查是否需要真实 YAML parser。若 registry 文件仍保持 JSON-compatible YAML，可不新增依赖；若 CP5 决定使用标准 YAML，需评估依赖管理并通过 `uv` 同步。

## 13. 实现灰区与取舍记录

| ID | Question | Options | Recommendation | Pros / Cons | Impact Surface | blocks_lld | Status |
|---|---|---|---|---|---|---|---|
| LCQ-CR037-S07-01 | 初始 registry 是否必须在 S07 实现中填充完整 CR-037 FEAT/CAP 条目？ | A. S07 只提供 schema/resolver + 最小 fixture；B. S07 同时初始化完整 CR-037 registry；C. S07 输出 candidate report 等 S13 补齐 | 推荐 A，CP5 人工审查时确认 schema，真实内容可分批补齐 | A 风险小但下游初期 unresolved 多；B 一次性完整但语义误填风险高；C 安全但阻塞更多下游 | registry、migration、roadmap、tests | false | clarification candidate；默认 A |
| LCQ-CR037-S07-02 | capability ID 命名是否固定 `CAP-PG-*`？ | A. 使用 `CAP-PG-*`；B. 使用领域短名如 `PG-CAP-*` | 推荐 A，与 `FEAT-PG-*` 对齐 | A 一致性高；B 可读性一般且引入新前缀 | registry、docs、tests | false | clarification candidate；默认 A |

以上均不阻断 LLD；用户在 CP5 approve 时接受推荐默认值。

## 14. 回滚与发布策略 / Definition of Done

### Rollback

- 回滚代码时移除新增 resolver/capability checker 调用，保留旧 `feature check/trace` 行为。
- 若 registry v2 导致误伤，降级为 v1 校验但不得允许自由字符串写入下游；下游 refs enforcement 可暂时停在 audit。
- candidate report 可删除重建，不影响 canonical registry。

### Release

- CP5 前不实现；CP5 通过后进入 CR037-W3。
- S08 的 refs enforcement 必须等待 S07 resolver API 至少通过 unit/contract tests。
- 不发布到 quant-lab，不写 `process/quant-lab/**`。

### Definition of Done

- `FEATURE-REGISTRY.yaml` v1/v2 校验兼容通过。
- `CAPABILITY-REGISTRY.yaml` schema/checker 存在并覆盖 positive/negative fixtures。
- resolver API 覆盖 `resolved`、`unresolved`、`deprecated`、`conflict`。
- candidate report 不写 canonical registry。
- S08/S09/S10/S12/S13 的 contract tests 能消费 resolver result。
- CP6 实现证据引用本 LLD 的 TASK-ID、接口和测试映射。

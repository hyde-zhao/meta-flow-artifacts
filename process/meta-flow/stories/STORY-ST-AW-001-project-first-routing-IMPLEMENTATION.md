---
story_id: "ST-AW-001"
story_slug: "project-first-routing"
cr_id: "CR-051"
stage: "CP6"
status: "rework-complete"
implemented_by: "meta-dev / dev-yang; CP6-R2 meta-dev-debugger"
implemented_at: "2026-07-18"
design_evidence_ref: "process/stories/STORY-ST-AW-001-project-first-routing-LLD.md"
story_packet_ref: "process/context/stories/STORY-ST-AW-001.CP6-R2.work-packet.json"
---

# ST-AW-001 实现说明：Project-first Artifact Routing

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-18 | meta-dev / dev-yang | 实现 portable route schema、纯 resolver、owned-target proof、legacy health 单向投影、定向 fixture 与 CP6 交接证据。 |
| 1.1 | 2026-07-18 | meta-dev-debugger | 回修 `CP7-AW-001-F01/F02`：adapter 增加 config/decision identity guard；legacy 非字符串 `owned_paths` 统一转为带 index 的结构化错误；新增两个负例并生成 CP6-R2 证据。 |

## 实现前置检查

| 检查项 | 结果 | 证据 / 说明 |
|---|---|---|
| Workspace process route health | PASS | `meta-flow workspace check --project-root .` 返回 `process_link_health: ok`；未修复、重建或改动 link。 |
| Story 状态与 CP5 | PASS | Story `status=dev-ready`；`lld_gate.status=approved`、`confirmed=true`；`dev_gate` 四项均为 true。 |
| 依赖门 | PASS | ST-AW-001 为 CR-051 DAG root，无 runtime/contract 前置 Story。 |
| 文件所有权 | PASS | 实现仅写 packet 的 `allowed_write_paths`；`routing.py` / 旧测试由本 Story 作为 merge owner。 |
| 设计契约完整性 | PASS | Feature DESIGN/TEST-PLAN/TASKS 与 LLD v1.1 可读，blocking clarification / OPEN / Spike 为 0。 |
| 深读审计 | PASS | 读取完整 LLD 前已执行 `meta-flow context read-log ... --reason deep_review`，事件 `RE-20260718T110824Z0000-823b6869`。 |
| CP7 回修授权 | PASS | Story `status=needs-rework`；CP7 result/return/FIXES 只要求关闭 `CP7-AW-001-F01/F02`；W2 保持关闭。 |
| R2 深读审计 | PASS | 更新 IMPLEMENTATION 前已执行 `context read-log --reason deep_review`，事件 `RE-20260718T121652Z0000-87975e94`；未重新展开 LLD。 |
| 运行授权 | N/A | 本 Story 为纯 Python 解析与临时目录 fixture；不需要真实 artifact、Git、remote、link 或 migration 权限。 |

## 实现对象清单

| 类型 | 对象 / 文件 | 实现内容 | 验证入口 |
|---|---|---|---|
| 代码 / Schema | `meta_flow/workspace/project_artifact_routing.py` | frozen `PathRef`、`ProjectArtifactConfig`、`RouteTarget`、`RouteDecision`、`OwnedTargetProof`、`RoutingError` 与 13 个稳定错误码。 | schema/error/path 参数化测试；Ruff；`py_compile`。 |
| 代码 / Resolver | 同上 | 单 metadata discovery、固定 anchor DAG、project-first/legacy layout dispatch、有序 read targets、每 kind 唯一 write target、portable digest。 | TC-AW-001/002/003/012；relocation/repeat fixture。 |
| Guardrail | 同上 | project/namespace/owned path proof；absolute/dot/traversal/NUL/newline/unknown parent/cycle/control nesting fail closed；legacy/project-first `owned_paths` 均逐项结构化校验；默认 serializer 排除 runtime absolute path。 | TC-AW-010；R-TC-04..13；`test_legacy_non_string_owned_path_uses_structured_error`。 |
| Compatibility adapter | `meta_flow/workspace/routing.py` | `project_route_to_process_health()` 在采用 target 前校验 config/decision project identity；不一致时 `route_mismatch` 且 target 不进入 health；旧 `check/link/bootstrap/write` 签名和默认行为不变。 | health shape 正向测试、`test_project_route_projection_rejects_cross_project_decision`、全部旧 routing tests。 |
| 定向测试 | `tests/test_cr051_project_artifact_routing.py` | project-first、显式 legacy、无 layout、anchor/path/identity/ownership、relocation、determinism、sibling isolation 和零写入用例。 | 定向 pytest。 |
| 兼容回归 | `tests/test_workspace_routing.py` | 增加新 decision 到旧 `ProcessRouteHealth` 的投影断言；保留既有测试。 | 定向 pytest。 |
| 交接证据 | 本文、CP6 return packet、evidence index | 记录契约映射、验证、边界、风险与下一路由。 | `return-check` / `evidence-check`。 |

## 设计契约映射

| 契约 / AC | 实现位置 | 实现动作 | 测试 / 证据 |
|---|---|---|---|
| F-01 / REQ-AW-001..003：layout 必须显式 | `load_project_artifact_config()`、`SUPPORTED_LAYOUTS` | 只接受 `legacy-shared-v1` / `project-first-worktree-v1`；缺失和未知均 `layout_unsupported`，不按存在性推断。 | `test_missing_layout_fails_closed_without_write_target`、invalid config matrix。 |
| F-02：docs/process 按 kind 解析、write target 0/1 | `resolve_project_artifact_route()` | API 强制单 `target_kind`；read intent 不返回 write target，write intent 只选择 primary。 | `test_project_first_returns_only_current_project_targets[docs/process]`。 |
| F-03 / CP5-QA-R1-F01：固定 anchor DAG | `_validate_anchor_graph()`、`_runtime_anchors()` | `project_worktree` 纳入允许 anchor；先 unknown/cycle/parent 校验，再做路径和 runtime boundary。 | unknown、wrong-parent、self/indirect-cycle 参数化负例。 |
| F-04：legacy 显式兼容但不形成第二写目标 | resolver layout branch | project-first 仅把显式 legacy leaf 作为 read-only compatibility；legacy layout 只写 legacy primary。 | `test_explicit_legacy_layout_stays_the_unique_write_target`。 |
| F-05：单 metadata discovery | loader | 只读显式 `metadata_path`，否则只读 `process/.meta-flow-process.yaml`；显式缺失不 fallback。 | `test_loader_uses_explicit_or_single_default_metadata_path`、读取 spy。 |
| F-06 / IF-AW-R01..04 | 新主模块与 `routing.py` | 实现 loader、resolver、ownership、serializer 与 legacy health projection；新模块不反向 import routing/lifecycle/Git。 | API 用例、Ruff、禁止 import scan。 |
| F-07/F-08：稳定且可行动错误 | `STABLE_ERROR_CODES`、`RoutingValidationError` | 冻结 13 code；错误包含 field/candidates/repair route，message 截断 500 字符。 | error contract 与 negative matrix。 |
| NF-01 / TC-AW-012：确定性 | canonical config/decision digest | digest 只含 logical anchor-relative payload、kind/intent/target order/error code，不含 timestamp/runtime root。 | 两个设备根 × 10 次解析，digest/order 一致率 100%。 |
| NF-02：可移植 | serializer / digest | `route_decision_to_dict()` 默认排除 runtime path；诊断模式才显式标记 noncanonical observation。 | relocation payload 扫描，两个设备绝对前缀违规数 0。 |
| NF-03/04：fail closed 与隔离 | path/anchor/ownership validators | 无 target 构造前拒绝 malformed config；不 import subprocess/Git，不读 sibling 内容，不执行 file/link mutation。 | sibling read/write spy=0；禁止依赖 scan=0；全部负例通过。 |
| NF-05：旧 routing 默认行为不变 | `project_route_to_process_health()` | 只新增显式 helper；旧入口没有接线或签名变化。 | `tests/test_workspace_routing.py` 全部通过。 |
| CP7-AW-001-F01：config/decision identity 一致 | `project_route_to_process_health()` | target 选择前比较 `decision.project_id` 与 `config.project_id`；不一致时加入可行动错误并将 target 保持为 `None`。 | `test_project_route_projection_rejects_cross_project_decision`。 |
| CP7-AW-001-F02：legacy malformed `owned_paths` 结构化 | `_normalize_owned_path_items()` / `_normalize_config()` | project-first 与 legacy 共用逐项类型校验；非字符串返回既有 `route_conflict`、字段 `owned_paths.<index>` 和 repair route。 | `test_legacy_non_string_owned_path_uses_structured_error`。 |

## 单元测试与 Fixture 计划

| 测试组 | 覆盖内容 | 实际结果 |
|---|---|---|
| Schema / loader | JSON-compatible YAML、无外部 YAML 依赖的简单 YAML、显式/default discovery、schema/layout/project 校验 | PASS |
| Anchor / path negative matrix | unknown/wrong parent/self/indirect cycle、absolute、traversal、dot、NUL/newline、control nesting、leaf overlap | PASS；拒绝率 100% |
| Project-first / legacy | docs/process 两 kind、显式 compatibility read、legacy 唯一 write、无 authoritative layout fail closed | PASS |
| Ownership / isolation | current owned path、sibling/control/common-prefix/kind mismatch、sibling sentinel、file write spy | PASS；越界拒绝率 100%，sibling read/write=0 |
| Determinism / relocation | 2 个 device root，每个重复 10 次；canonical payload scan | PASS；digest/order 一致率 100%，absolute prefix=0 |
| Adapter regression | 旧 workspace routing 4 个既有测试 + 1 个 projection 测试 | PASS |
| CP7 回修负例 | config/decision 跨项目混用；legacy `owned_paths[0]` 为整数 | PASS；2/2 finding closure |

所有 fixture 使用 pytest `tmp_path`；没有访问真实 `meta-flow-artifacts`、真实 sibling 项目或修改当前 `process` link。

## 最小实现切片

| Slice | TASK-ID | 输出 | 局部验证 | 回滚点 |
|---|---|---|---|---|
| S1 Schema / validation | TASK-AW-R01 | 主模块的 value objects、错误码、metadata parser、path/anchor validator | 首轮测试先失败于模块不存在；实现后 schema/negative tests 通过。 | 删除新主模块与新测试，不影响旧 routing。 |
| S2 Resolver / ownership | TASK-AW-R02/R03 | layout dispatch、portable digest、owned proof、serializer | project-first/legacy/ownership/relocation fixture 通过。 | 保留 S1，撤销 resolver/ownership 函数。 |
| S3 Compatibility adapter | TASK-AW-R04 | `routing.py` 单向投影 helper | 旧 routing + projection tests 通过。 | 仅撤销 helper 与对应共享测试；旧入口从未切换。 |
| S4 Fixture hardening | TASK-AW-R05 | 36 个定向测试（含旧测试） | `36 passed`。 | 测试文件可独立回滚，不触碰运行路由。 |
| S5 Static / boundary evidence | TASK-AW-R06 | Ruff、py_compile、diff/import/touched-path 审计 | 除仓库缺失 boundary manifest 导致两项未进入分析外，其余通过。 | N/A；只读验证。 |
| S6 CP6-R2 最小回修 | CP7-AW-001-F01/F02 | adapter identity guard、owned-path item validator、两个回归测试 | 两个负例先红后绿；完整 `38 passed`；Ruff/py_compile/diff PASS。 | 仅撤销 R2 四个批准工程文件中的最小 diff；不涉及设计或数据回滚。 |

## 平台差异处理

N/A。实现是 Python typed domain 和本地临时目录 fixture，不生成 Claude/Codex/OpenClaw 专属 schema，不调用平台问答工具，也不涉及平台安装路径。canonical payload 使用 POSIX relative path，并额外拒绝 Windows drive/UNC 形态。

## 验证结果

| 命令 | 结果 | 说明 |
|---|---|---|
| `meta-flow workspace check --project-root .` | PASS | route health OK；未执行修复。 |
| `meta-flow context sufficiency-check --packet process/context/stories/STORY-ST-AW-001.CP6.work-packet.json` | PASS | `Context Sufficiency Check: OK`。 |
| `PYTHONDONTWRITEBYTECODE=1 PYTEST_ADDOPTS='-p no:cacheprovider' uv run --python 3.11 pytest -q tests/test_cr051_project_artifact_routing.py tests/test_workspace_routing.py` | PASS | `36 passed in 0.13s`。 |
| `uv run --python 3.11 ruff check ...`（4 个批准文件） | PASS | `All checks passed!`。 |
| `uv run --python 3.11 python -m py_compile meta_flow/workspace/project_artifact_routing.py` | PASS | exit 0。 |
| `rg -n 'subprocess\|git_sync\|meta_flow\.workflow\|artifact_leg_lifecycle\|artifact_aggregate' meta_flow/workspace/project_artifact_routing.py` | PASS | 0 matches；`rg` exit 1 表示未匹配。 |
| `git diff --check` | PASS | exit 0。 |
| `meta-flow check module-boundaries --project-root .` | NOT EXECUTED（环境前置缺失） | checker 在分析前返回 `MODULE-BOUNDARIES missing: docs/design/MODULE-BOUNDARIES.yaml`。 |
| `meta-flow check imports --project-root .` | NOT EXECUTED（环境前置缺失） | 同一固定 manifest 缺失；没有越权创建或重路由长期设计文件。 |

## 验证失败回修（CP6-R2）

### 回修范围与 finding closure

| Finding | 初始失败证据 | 最小修复 | 新增负例 | R2 结果 |
|---|---|---|---|---|
| `CP7-AW-001-F01` | project-a config + project-b PASS decision 被投影为 `health.ok=True`，并采用 project-b target | adapter 在读取 `write_target` / primary target 前校验 project identity；不一致时 `status=route_mismatch`、`actual_target=None`、`project_process_root=None`，错误给出重新 resolve 动作 | `test_project_route_projection_rejects_cross_project_decision` | CLOSED |
| `CP7-AW-001-F02` | legacy `owned_paths=[7, "process"]` 泄漏裸 `TypeError` | 提取 `_normalize_owned_path_items(value, required=...)`；legacy 与 project-first 共用元素类型校验，保留 project-first 非空/去重/重叠规则 | `test_legacy_non_string_owned_path_uses_structured_error` | CLOSED |

### 测试先行证据

1. 仅新增两个负例后运行：`2 failed in 0.09s`，分别复现裸 `TypeError` 与 `health.ok=True`。
2. 最小修复后只运行两个负例：`2 passed in 0.06s`。
3. 完整定向回归：`38 passed in 0.14s`，原 36 项全部保持通过。
4. Ruff、两个源码模块 `py_compile`、`git diff --check` 均 PASS；禁止 import 扫描匹配数为 0。

### 契约与边界复核

- `SCHEMA_VERSION` 仍为 `1`。
- 支持的 layout 仍仅为 `legacy-shared-v1` 与 `project-first-worktree-v1`。
- `STABLE_ERROR_CODES` 仍为原 13 项；F02 复用 `route_conflict`，没有新增或重命名错误码。
- 没有改变 branch topology、anchor DAG、metadata discovery、digest、write-target cardinality 或 owned proof 公共结构。
- 没有修改 LLD、Feature DESIGN、HLD/ADR 或其他 Story；Design Delta 为 N/A。
- sibling、真实 Git/worktree/ref/remote/link/migration、commit/push/publish/main-sync mutation 次数为 0。

### CP6-R2 交接

- R2 return：`process/returns/ST-AW-001.CP6-R2.return.json`。
- R2 evidence index：`process/evidence/ST-AW-001.CP6-R2.index.json`。
- 建议下一路由：Host Orchestrator 执行独立 CP6-R2 自动检查后，再次调度 meta-qa CP7；本代理不批准 CP6-R2。

## 边界与零副作用审计

- 实现源码/测试/交接文件全部位于 Story packet `allowed_write_paths`。
- `meta_flow/workflow/artifact_leg_lifecycle.py`、`artifact_aggregate.py`、worktree 模块、`.git/**`、sibling/quant-lab、真实 artifact/link/migration target 的写入次数均为 0。
- 未执行 commit、push、publish、main-sync、worktree/ref/remote mutation。
- 主 resolver 不 import `subprocess`、Git sync 或 `meta_flow.workflow.*`；不调用 mkdir/write/symlink。
- 唯一额外过程写入是 handoff 强制的 `context read-log` 对 `process/state/READ-EXPANSION-LEDGER.ndjson` 追加审计事件，不属于工程实现边界扩张。

## 偏差与设计缺口反馈

实现没有偏离 LLD 的 layout、anchor DAG、metadata discovery、唯一 write target、portable digest、owned proof 或零 mutation 契约，无需 Design Delta。

仓库级 module-boundaries/import checker 的固定输入 `docs/design/MODULE-BOUNDARIES.yaml` 当前缺失，而 packet 的设计索引也指向该路径。这是验证环境/长期设计路由前置缺失，不是本 Story 允许修复的文件；Host/QA 应决定在 CP6/CP7 前恢复该 canonical manifest，或以已有边界证据登记明确 waiver。不得由本 Story静默新建。

## 未覆盖项

1. 未执行真实跨设备手工验收 MAN-AW-R01；已用两个不同临时 device root 等价验证，真实设备验收留给 CP7/human。
2. 仓库级 module boundary/import checker 因 manifest 缺失未进入分析；代码级禁止 import scan、Ruff、py_compile 和定向回归已通过。
3. 未运行真实 artifact、Git、worktree、link 或 migration；这些均为明确禁止范围，不是测试遗漏。

## 剩余风险

| 风险 | 等级 | 处置建议 |
|---|---|---|
| `docs/design/MODULE-BOUNDARIES.yaml` 缺失导致两项架构 checker 无法执行 | Medium | Host/QA 在不改变本 Story 边界的前提下恢复 canonical manifest 或登记可审计 waiver，再执行 CP6/CP7 静态边界检查。 |
| 简单 YAML loader 只支持本 schema 所需的 mapping、scalar 和 scalar-list 子集 | Low | metadata 生成方保持 schema canonical 形态；复杂 YAML feature（anchor/merge/tag）不属于 portable contract，解析失败会 fail closed。 |
| 真实双设备人工观察尚未执行 | Low | CP7 可按 MAN-AW-R01 在受控只读环境复核；当前 relocation fixture 已证明 canonical digest 不含设备根。 |

## 后续交接

- 建议 CP6 结论：`PASS_WITH_RISK`，风险仅为仓库级 boundary manifest 缺失导致两项检查未执行；不得据此授权真实 Git/link/migration。
- QA 验证入口：定向 pytest、Ruff、py_compile、forbidden import scan、return/evidence check；恢复 manifest 后重跑 module-boundaries/imports。
- 重点复核：每 kind write target 基数、missing/unknown layout fail closed、anchor cycle/parent 错误顺序、default serializer 无 runtime path、adapter 未接管旧写入口。
- 下一路由：返回 Host Orchestrator，由 Host 生成/审批 CP6 自动结果；本实现代理不批准 CP6。

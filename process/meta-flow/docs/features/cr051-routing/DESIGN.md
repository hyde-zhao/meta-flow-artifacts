---
status: draft-for-cp4
version: "1.1"
feature_id: "FEAT-AW-01"
feature_name: "Project Context & Portable Routing"
source_blueprint: "process/docs/design/CR051-ARTIFACT-WORKTREE-BLUEPRINT.md"
source_hld: "process/docs/design/CR051-ARTIFACT-WORKTREE-HLD.md"
source_adr: "process/docs/design/CR051-ARTIFACT-WORKTREE-ARCHITECTURE-DECISION.md"
source_matrix: "process/docs/design/CR051-FEATURE-DESIGN-MATRIX.md"
related_stories: ["ST-AW-001"]
lld_policy_summary: "ST-AW-001=full-lld"
confirmed_by: ""
confirmed_at: ""
---

# Feature Design：Project Context & Portable Routing

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-18 | meta-se-critical | 冻结 project-first/legacy 路由 schema、唯一写目标、错误模型与下游只读契约 |
| 1.1 | 2026-07-18 | meta-dev | CP5 R2 关闭 `CP5-QA-R1-F01`：将 `project_worktree` 纳入 `PathRef.anchor`，冻结无环 anchor DAG、允许父子关系、拒绝矩阵与 portable digest 测试；不改变 layout/owned target/零 mutation 边界。 |

## 摘要

| 项目 | 内容 |
|---|---|
| Feature 目标 | 用显式 project identity 与 layout version 唯一解析当前项目 artifact 的 docs/process，跨设备仍保持同一语义 |
| 推荐方案 | 新增纯解析 `project_artifact_routing` 边界，以 anchor + relative path 保存 canonical metadata；现有 `routing.py` 仅作兼容适配 |
| 关键取舍 | 保留 legacy 只读兼容，但绝不根据路径存在性猜写目标；歧义一律 `BLOCKED` |
| 下游 Story | ST-AW-001 |
| LLD 策略 | `full-lld`：涉及 portable schema、跨模块契约与双布局写入安全 |

## 背景与问题

| 问题 ID | 背景 | 触发场景 | 影响 | 若不设计的风险 |
|---|---|---|---|---|
| P-AW-R01 | 当前 `routing.py` 只认识 `process` link/local-directory | UC-AW-001；REQ-AW-001..003 | 无法同时表达 docs/process、project-first 与 legacy | 写入 sibling 或形成双可写真相 |
| P-AW-R02 | 现有 runtime 路径可以是绝对路径 | REQ-AW-NF002 | 跨设备同一配置解析不稳定 | canonical metadata 绑定设备 |
| P-AW-R03 | legacy 与新目录可能同时存在 | TC-AW-002/003 | 读兼容与写安全发生冲突 | 仅凭“存在”静默切换写目标 |
| P-AW-R04 | worktree 生命周期需要可信 project context | TC-AW-010 | FEAT-AW-02/03/05 的安全门依赖路由 | identity/ownership 错配后发生 mutation |

## 上游依据与输入

| 来源 | 路径 / ID | 被本设计消费的内容 |
|---|---|---|
| Blueprint | `process/docs/design/CR051-ARTIFACT-WORKTREE-BLUEPRINT.md` | FEAT-AW-01 边界、OBJ-AW-01/02 owner、禁止依赖 lifecycle |
| HLD | `process/docs/design/CR051-ARTIFACT-WORKTREE-HLD.md` §4/6/9.1 | ProjectContext、portable paths、失败即禁止 Git |
| ADR | ADR-AW-002 | existing control checkout + configurable sibling worktree、portable registry |
| Feature Matrix | `process/docs/design/CR051-FEATURE-DESIGN-MATRIX.md` | 由 core planning lane 写入的 `required/full-lld` 判定；本 pack 不代写矩阵 |
| Product | ST-AW-001；REQ-AW-001..003/013/NF001..002 | 唯一解析、legacy dual-read、dry-run、确定性与可移植性 |
| Scenario | TC-AW-001..003、010、012 | 正向、兼容、歧义、identity mismatch 与零副作用 |

## 目标与非目标

| 类型 | 内容 | 来源 |
|---|---|---|
| Goal | 相同 canonical metadata 在不同 workspace root 下产生相同逻辑 route | REQ-AW-NF001..002 |
| Goal | project-first 与 legacy 都可显式读取，但每次只有一个 write target | REQ-AW-001..003 |
| Goal | 错误返回 project、字段、候选路径、稳定 code 与安全修复路由 | REQ-AW-NF004 |
| Non-Goal | 创建/切换/remove worktree 或修改 ref | Blueprint FEAT-AW-02 |
| Non-Goal | 自动迁移文件、创建 link 或推断用户迁移意图 | MVP OUT-AW-001/002 |
| Non-Goal | 读取 sibling 项目内容来证明当前项目 | REQ-AW-C003 |

## Feature 边界与相邻对象

| 对象 | 本 Feature 负责 | 不负责 | 相邻 Feature | 边界依据 |
|---|---|---|---|---|
| `ProjectArtifactConfig` | schema、project/layout、portable path refs、namespace/owned-path policy | worktree health、Git OID、运行授权 | FEAT-AW-02 | OBJ-AW-01 |
| `RouteDecision` | read targets、唯一 write target、conflicts、decision | 执行写入或恢复 | FEAT-AW-02/03/05 | OBJ-AW-02 |
| legacy compatibility | 显式版本下只读/写目标选择 | 文件搬迁与 link 切换 | FEAT-AW-05 | REQ-AW-002/003 |
| owned path declaration | 规范化并证明目标位于 project namespace | Git changed/staged/committed path 审计 | FEAT-AW-02/03 | ADR-AW-002 |

## 现有代码位置

| 区域 | 路径 | 当前职责 | 变更方式 |
|---|---|---|---|
| process routing | `meta_flow/workspace/routing.py` | process link/local-directory 健康检查及 metadata writer | modify：只增加兼容适配，不承载 worktree/Git |
| project artifact resolver | `meta_flow/workspace/project_artifact_routing.py` | 尚不存在 | create：本 Feature 主 owner |
| existing route tests | `tests/test_workspace_routing.py` | 旧 process route 回归 | modify：仅必要兼容回归 |
| CR-051 route tests | `tests/test_cr051_project_artifact_routing.py` | 尚不存在 | create：本 Feature 主 owner |

## 现状分析

| 维度 | 当前状态 | 缺口 | 约束 |
|---|---|---|---|
| 数据 | `.meta-flow-process.yaml` 保存 process route | 无统一 docs/process、layout/namespace schema | 不破坏旧 metadata 读取 |
| 接口 | `check_process_route()` 返回 `ProcessRouteHealth` | 无纯 `ProjectContext/RouteDecision` | resolver 不得调用 Git/lifecycle |
| 测试 | 有 process symlink/local fixture | 无双布局、双项目、移动 workspace fixture | 真实 artifact mutation=0 |
| 运维 | runtime 可显示绝对 resolved path | canonical contract 不区分 persisted/runtime | 绝对路径只能存在于临时 decision/output |

## 推荐方案

| 设计点 | 推荐做法 | 理由 | 代价 |
|---|---|---|---|
| Schema | versioned frozen dataclass/JSON-compatible payload；未知版本 fail closed | 可校验、可演进 | 需要 legacy adapter |
| Path | `PathRef(anchor, relative_path)`；仅允许已知 anchor 与规范化相对路径 | 跨设备稳定、防逃逸 | runtime 需解析步骤 |
| Layout | `legacy-shared-v1`、`project-first-worktree-v1` 两个显式枚举 | 避免基于存在性猜测 | 新 layout 必须升 schema/version |
| Write | 每个 decision 恰好 0 或 1 个 write target；多候选直接 BLOCKED | 消除双可写真相 | 部分旧环境需要先修 metadata |
| Read | 按 layout 产生 primary；只有显式 compatibility policy 才追加 legacy fallback | 保留迁移期读取 | 消费方须尊重顺序与只读标志 |

## 方案对比与决策记录

| Decision ID | 方案 | Pros | Cons | Impact Surface | Recommendation | When to switch |
|---|---|---|---|---|---|---|
| FD-AW-R01 | 独立纯 resolver + 旧 routing adapter | 单向依赖、易做无副作用 fixture | 新增一模块 | workspace routing、FEAT-AW-02/03/05 | 推荐 | resolver 稳定后可让旧 API 内部统一委托 |
| FD-AW-R01 | 直接扩写 `routing.py` | 文件少 | process link、artifact layout、Git health 继续耦合 | 旧命令/测试全量 | 备选 | 仅新模块造成不可接受的公共 API 重复时 |
| FD-AW-R02 | 显式 layout version | 写目标确定 | 迁移需先更新 config | metadata、migration | 推荐 | 不切换；新布局通过新 version 扩展 |
| FD-AW-R02 | 按目录存在性猜测 | 零配置 | 双写与跨项目风险 | 所有写入方 | 拒绝 | 无切换条件 |

## 数据模型与状态

### `PathRef`

| 字段 | 类型 / 约束 | 说明 |
|---|---|---|
| `anchor` | `project_root | artifact_control_root | sibling_root | project_worktree` | 仅允许四个稳定逻辑 anchor；`project_worktree` 是 docs/process 的合法父 anchor，不是设备绝对路径 |
| `relative_path` | 非空规范化 POSIX relative path | 禁止绝对路径、`..` 逃逸、NUL/newline |

#### Anchor DAG 与允许父子关系

```text
project_root
├── artifact_control_root
│   ├── legacy_docs
│   └── legacy_process
└── sibling_root
    └── project_worktree
        ├── docs_relative
        └── process_relative
```

| 配置字段 / 逻辑节点 | 唯一允许的 `PathRef.anchor` | 解析后成为可引用 anchor | 非法示例 |
|---|---|---|---|
| `artifact_control_root` | `project_root` | `artifact_control_root` | anchor=`artifact_control_root|sibling_root|project_worktree` |
| `sibling_root` | `project_root` | `sibling_root` | anchor=`artifact_control_root|sibling_root|project_worktree` |
| `project_worktree` | `sibling_root` | `project_worktree` | anchor=`project_root|artifact_control_root|project_worktree` |
| `docs_relative` / `process_relative`（project-first） | `project_worktree` | 否 | anchor=`project_root|artifact_control_root|sibling_root` |
| `legacy_docs` / `legacy_process` | `artifact_control_root` | 否 | anchor=`project_root|sibling_root|project_worktree` |

`project_root` 是调用方提供的唯一 runtime root，不由 metadata 中另一条 `PathRef` 定义。loader 必须先校验 anchor 枚举与上表父子关系，再对节点执行 gray/black DFS 或等价拓扑校验；未知 anchor、错误父 anchor、自环/间接环、绝对路径、`.` / `..` 段及解析后越出父 anchor 均在任何 target 构造前拒绝。解析顺序固定为 `project_root → artifact_control_root/sibling_root → project_worktree → docs/process`，legacy leaf 固定从 `artifact_control_root` 解析。

### `ProjectArtifactConfig`

| 字段 | 必需 | 约束 |
|---|---:|---|
| `schema_version` | 是 | 当前 `1`；未知版本 BLOCKED |
| `project_id` | 是 | 稳定 token；与请求/route metadata 一致 |
| `layout_version` | 是 | `legacy-shared-v1` 或 `project-first-worktree-v1` |
| `artifact_control_root` | 是 | anchor=`project_root` 的 `PathRef` |
| `sibling_root` | 新布局是 | anchor=`project_root` 的 `PathRef`，不得嵌套于 control worktree |
| `project_worktree` | 新布局是 | anchor=`sibling_root`，值必须等于规范化 project identity |
| `docs_relative` / `process_relative` | 是 | project-first 时 anchor=`project_worktree`；owned relative paths 互不重叠且不逃逸 |
| `legacy_docs` / `legacy_process` | 兼容时是 | anchor=`artifact_control_root`、只按显式 layout/policy 消费 |
| `branch_namespace` / `owned_paths` | 新布局是 | 传给下游 hard gate；resolver 本身不改 Git |

### `RouteDecision`

`decision=PASS|BLOCKED`；包含 `project_id`、`layout_version`、有序 `read_targets[{kind,path,read_only}]`、至多一个 `write_target`、`conflicts[]`、`error_code`、`config_digest`、`observed_at`。resolved absolute path 仅为 runtime observation，不得回写 canonical config。

## API / 接口设计

| Interface ID | 调用方 | 被调用方 | 输入契约 | 输出契约 | 错误模型 |
|---|---|---|---|---|---|
| IF-AW-R01 `load_project_artifact_config` | Host/CLI/adapter | resolver | project root + explicit metadata ref | validated config 或 typed error | missing/schema/project/path errors |
| IF-AW-R02 `resolve_project_artifact_route` | FEAT-AW-02/03/05 | resolver | config + requested project + operation intent(read/write) | immutable `RouteDecision` | ambiguity/escape/mismatch→BLOCKED |
| IF-AW-R03 `assert_owned_target` | mutation preflight | resolver | PASS decision + candidate path + target kind | normalized owned path proof | sibling/control/escape→BLOCKED |
| IF-AW-R04 legacy adapter | `routing.py` | resolver | existing health inputs | old `ProcessRouteHealth` projection | unsupported state keeps old blocking semantics |

稳定错误码：`config_missing`、`schema_unsupported`、`layout_unsupported`、`project_mismatch`、`anchor_unknown`、`anchor_parent_invalid`、`anchor_cycle`、`absolute_canonical_path`、`path_escape`、`control_nested_worktree`、`write_target_ambiguous`、`target_not_owned`、`route_conflict`。错误须带字段、声明 anchor、期望父 anchor和 logical candidate，不打印凭据/remote URL。

## 关键流程

| Flow ID | 触发条件 | 主流程 | 异常流程 | 输出 / 状态变化 | 观测点 |
|---|---|---|---|---|---|
| FLOW-AW-R01 | route/check | 读 config→校验 identity/version→校验 anchor enum/父子/DAG→拓扑解析 anchors→构造 targets→唯一性检查 | 未知/错误父 anchor、环、绝对/点段/越界或候选冲突→BLOCKED | 只生成 decision；零写入 | config digest、anchor edge、targets、code |
| FLOW-AW-R02 | legacy read | 显式 legacy layout→解析 legacy docs/process→标记读取属性 | 新旧并存但无 version→BLOCKED | 不移动/不链接 | layout、read order |
| FLOW-AW-R03 | downstream mutation precheck | 消费 PASS decision→校验 candidate 属于唯一 write target/owned path | sibling/control/identity 不匹配→BLOCKED | owned-path proof | project、kind、relative path |
| FLOW-AW-R04 | workspace moved | 用新 runtime project root 重解 anchors | persisted absolute/path escape→BLOCKED | 逻辑 target 不变 | device-prefix violations=0 |

## 异常、失败与降级策略

| Failure ID | 失败条件 | 系统行为 | 用户可见影响 | 恢复 / 回退 | 测试入口 |
|---|---|---|---|---|---|
| F-AW-R01 | layout/version 缺失或未知 | BLOCKED、write target=None | 无写入 | 补充/升级 metadata | TC-AW-003 |
| F-AW-R02 | project-first 与 legacy 候选冲突 | 列出两者但不给写目标 | 读取可按显式只读策略继续，写阻断 | 明确 layout version | TC-AW-002/003 |
| F-AW-R03 | project/path/namespace mismatch | BLOCKED，输出 mismatch 字段 | lifecycle 不启动 | 修正 config/选择正确项目 | TC-AW-010 |
| F-AW-R04 | absolute canonical path 或 traversal | 验证失败，零解析外写 | 配置不可用 | 改成 anchor-relative | TC-AW-001/010 |
| F-AW-R05 | 旧 metadata 只够现有 process health | 旧 API 保持兼容；新 artifact mutation 禁用 | 可只读诊断 | 生成显式新 config | legacy fixture |
| F-AW-R06 | unknown anchor、错误父 anchor 或 anchor cycle | `anchor_unknown|anchor_parent_invalid|anchor_cycle`，不解析 target | 配置不可用、mutation=0 | 按固定 DAG 修正字段 | R-TC-09..11 |

## 权限与安全

| Rule ID | 规则 | 触发条件 | 失败行为 | 测试入口 |
|---|---|---|---|---|
| SEC-AW-R01 | resolver 永不执行 Git/file/link mutation | 所有调用 | API 只返回 value object | command/write spy |
| SEC-AW-R02 | 不读取 sibling owned content | project resolve | 仅比较规范化声明与路径边界 | TC-AW-010 + sibling sentinel |
| SEC-AW-R03 | canonical path 必须相对且不逃逸 | config load | typed BLOCKED | traversal/absolute matrix |
| SEC-AW-R04 | sparse 不是授权 | owned target 检查 | 仍要求 owned-path proof | sparse negative fixture |
| SEC-AW-R05 | anchor graph 必须属于固定 allowlist 且无环 | config load | 未知/错误边/环在 target resolve 前 BLOCKED | anchor enum/parent/cycle matrix |

## 测试与验收策略

| 验收对象 | 测试层级 | 覆盖场景 | 自动化方式 | 手工入口 |
|---|---|---|---|---|
| schema/path normalization | unit | REQ-AW-001..003/NF002 | pytest 参数化 | 无 |
| project-first/legacy decisions | fixture | TC-AW-001..003 | 临时目录纯解析 | 无 |
| identity/ownership | negative fixture | TC-AW-010 | 双项目 sentinel + write/Git spy=0 | 无 |
| determinism/dry-run | contract | TC-AW-012、NF001 | 同输入重复与 workspace relocation | 跨设备仅作 CP7 补充 |
| anchor schema / DAG | unit + negative fixture | unknown anchor、错误父 anchor、自环/间接环、absolute/dot/traversal/escape | 参数化 loader contract | 无 |

## 实现顺序

| Step | 内容 | 前置条件 | 输出 | 验证入口 |
|---:|---|---|---|---|
| 1 | 冻结 schema、错误码与 path normalization | CP4/CP5 设计确认 | pure types/validators | unit |
| 2 | 实现 route decision 与 legacy policy | Step 1 | resolver | TC-AW-001..003 |
| 3 | 增加旧 routing adapter 与 owned-target proof | Step 2 | compatibility API | TC-AW-010 |
| 4 | 完成 determinism/relocation/deny-mutation fixtures | Step 3 | evidence | TC-AW-012 |

## Story 拆分建议与 LLD 策略

| Story ID | feature_design_refs | lld_policy.required_level | 触发原因 | 必须进一步设计的问题 | 可用设计证据 |
|---|---|---|---|---|---|
| ST-AW-001 | `process/docs/features/cr051-routing/{DESIGN,TEST-PLAN,TASKS}.md` | full-lld | portable schema、legacy/new conflict、跨模块输入 | 精确函数签名、metadata 文件发现顺序、兼容 adapter 的错误投影 | Story LLD |

## 下游消费契约

| 消费方 | 消费时机 | 输入契约 | 输出 / 状态要求 | 降级策略 |
|---|---|---|---|---|
| FEAT-AW-02/03/05 | mutation/preflight 前 | `RouteDecision.decision=PASS` + config digest + owned target proof | 不得重解或更改 write target | 缺失/陈旧即 BLOCKED |
| story-manager | CP4 | Story refs + full-lld policy | ST-AW-001 卡片写入 refs/policy | 缺失则 CP4 FAIL |
| lld-designer | CP5 前 | 本三件套 + Story | 字段级/函数级 LLD | schema 未冻结则 BLOCKED |
| meta-qa | CP7 | TEST-PLAN + TC refs | 路由证据与零副作用审查 | 缺口回修 |

## 风险与回退

| Risk ID | 风险 | 影响 | 缓解 | 回退 |
|---|---|---|---|---|
| R-AW-R01 | 新 resolver 与旧 process route 语义漂移 | 健康检查不一致 | compatibility fixture、单向 adapter | 保留旧 API 只读，禁用新 mutation |
| R-AW-R02 | legacy fallback 被误用为写候选 | 双写 | write target cardinality invariant | BLOCKED 并要求显式 version |
| R-AW-R03 | runtime absolute path 被持久化 | 不可移植 | canonical serializer 拒绝绝对路径 | 拒绝写 config/result canonical 字段 |
| R-AW-R04 | anchor 枚举与字段父子关系漂移 | schema 不可构造或跨 root 解析 | 单一 allowlist + parent matrix + DAG/cycle tests | loader BLOCKED，不进入 route selection |

## Gotchas

1. “两个路径都存在”不代表二者都可写；没有显式 layout version 时必须阻断。
2. runtime resolved absolute path 可用于诊断，不得反向写入 canonical metadata。
3. `process` symlink 健康不等于整个 project artifact context 合法；docs、namespace、worktree identity 仍需独立验证。
4. resolver 不能通过调用 `git status` 猜当前项目；Git/worktree observation 属于 FEAT-AW-02。
5. `project_worktree` 必须同时是 `PathRef.anchor` 的合法枚举值和 `docs_relative/process_relative` 的唯一父 anchor；只在字段说明中写“语义 anchor”而不进入类型枚举会产生不可实现 schema。

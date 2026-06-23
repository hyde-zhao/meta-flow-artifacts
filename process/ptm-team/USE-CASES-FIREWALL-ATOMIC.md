---
status: draft
version: "0.1"
confirmed_by: ""
confirmed_at: ""
engagement_mode: production
scenario_subject_type: target-artifact
scenario_subject_id: firewall-test-atomic-op-spec
target_artifact_type: mixed
governance_mode: conditional
review_policy: light
total_use_cases: 12
---

<!-- Phase 1A 治理字段判定完成于 2026-05-09 -->
<!-- 本文件独立于 process/USE-CASES.md（ptm-tde 主项目场景），仅描述"防火墙测试原子操作规范"产物的使用场景 -->

## 治理附录（Governance）

| 字段 | 当前值 | 说明 |
|---|---|---|
| `engagement_mode` | `production` | 面向目标产物的生产模式；用户未声明 meta 优化 |
| `scenario_subject_type` | `target-artifact` | 场景主体是"防火墙测试原子操作规范"这个产物，不是 ptm-tde 仓库本身 |
| `scenario_subject_id` | `firewall-test-atomic-op-spec` | 目标交付物 ID |
| `target_artifact_type` | `mixed` | 同一请求同时包含规范文档（design doc）+ 代码骨架（scaffold code）两种交付形态，落盘位置不同（doc/ vs code/），触发方式也不同（参考阅读 vs 直接复制执行），命中 mixed 硬规则 |
| `governance_mode` | `conditional` | 关键场景节点需用户确认，不全程重度评审 |
| `review_policy` | `light` | 关键对象轻量确认为主 |

**mixed 拆分说明：**
- 交付面 A（规范文档）：原子操作元数据 schema、设计原则、命名规范、字段约束文档；主要落点 `delivery/doc/`；使用方式为"阅读参考"
- 交付面 B（代码骨架）：防火墙 REST API / TG / Linux Mock / 交换机四类设备调用骨架 + YAML/Python 模板；主要落点 `delivery/scaffold/`；使用方式为"复制/抄改执行"

---

## Schema 字段集约束（已确认规则汇总）

> 本章节汇总所有已 RESOLVED 的澄清问题对 schema 设计的硬性约束。下游 HLD/LLD 必须以此为唯一事实来源，不得与下列规则矛盾。

### A. 粒度与边界（Q-08 / B2）

| 规则 ID | 字段/约束 | 内容 |
|---------|---------|------|
| B2-01 | 粒度定义 | 原子操作 = 单一可观察状态变更；内部可含必要等待/重试，不含跨目标断言 |
| B2-02 | `device` | 单值字符串，原子操作只能以单设备为操作主体 |
| B2-03 | 命名规范 | `{设备}_{动词}_{目标}`，如 `fw_ensure_acl_rule_exists` |

### B. 异常处理（Q-09 / B2 决策）

| 规则 ID | 字段/约束 | 内容 |
|---------|---------|------|
| B2-04 | `on_failure` | 枚举（`fail_fast` / `return_error` / `retry_by_caller`），默认 `return_error`，必填 |
| B2-05 | 失败返回结构 | `{status: "failed", error_type: "<类型>", detail: "<说明>"}` 为最小必填结构 |
| B2-06 | 内部重试禁止 | 原子操作内部不做自动重试，重试策略由编排层控制 |

### C. 幂等性（Q-10 / I2 决策）

| 规则 ID | 字段/约束 | 内容 |
|---------|---------|------|
| I2-01 | `idempotent` | 布尔，默认 `true`；所有原子操作应默认实现为幂等 |
| I2-02 | `side_effects` | 字符串，仅当 `idempotent: false` 时必填，描述对系统状态的持久影响 |

### D. 跨设备时序（Q-11 / E2 决策）

| 规则 ID | 字段/约束 | 内容 |
|---------|---------|------|
| E2-01 | 跨设备联动原则 | 跨设备联动不合并为复合原子操作，始终保持单设备粒度 |
| E2-02 | 测试用例 `tight_sequence` | 测试用例 schema 新增 `tight_sequence: [op_id_1, op_id_2, ...]`，声明需要紧密顺序执行的操作组；不影响原子操作本身设计 |

### E. 参数化（Q-12 / P1 决策）

| 规则 ID | 字段/约束 | 内容 |
|---------|---------|------|
| P1-01 | `parameters` | 数组，每项为独立对象 |
| P1-02 | 每参数必填字段 | `name: string`、`type: enum(string/int/float/bool/json/list)`、`required: bool`、`description: string` |
| P1-03 | 每参数可选字段 | `default: <随 type>`（有默认值时填写）；`schema_ref: string`（type 为 json 时可附带，指向复合对象定义）|
| P1-04 | 约束 | `required: true` 且无默认值时不填 `default` 字段 |

### F. 返回值规范（Q-13 / R1 决策）

| 规则 ID | 字段/约束 | 内容 |
|---------|---------|------|
| R1-01 | 统一顶层结构 | 所有原子操作返回值必须包含：`status: enum(success/failed/timeout/skipped)`、`result: object`、`metadata: object` |
| R1-02 | `metadata` 固定字段 | `duration_ms: int`、`device: string`、`timestamp: ISO8601 string` |
| R1-03 | `output_schema` | 各操作在 schema 中声明 `result` 对象的内部结构；失败时 `result` 中必含 `error_type` 和 `detail` |
| R1-04 | `metadata.device` 一致性 | 与操作级 `device` 字段保持一致，供编排层做路由和日志归因 |

### G. 版本与演进（Q-14 / V1 决策）

| 规则 ID | 字段/约束 | 内容 |
|---------|---------|------|
| V1-01 | `schema_version` | semver 字符串（如 `"1.0.0"`），每个原子操作独立版本，必填 |
| V1-02 | `deprecated` | 布尔；废弃操作必须声明 `deprecated: true` |
| V1-03 | `since` | semver；废弃起始版本，废弃时必填 |
| V1-04 | `reason` | 字符串；废弃原因说明，废弃时必填 |
| V1-05 | `replaced_by` | 操作 ID；可选，指向替代操作 |
| V1-06 | 保留周期 | 废弃操作保留至少一个大版本周期后方可删除 |
| V1-07 | `CHANGELOG.md` | 整体原子操作库维护 `CHANGELOG.md`，每次新增/修改/废弃必须追加条目 |

---

<!-- Phase 1B：待 Q-02/Q-03/Q-04/Q-07 澄清后补入画像、成功指标和场景列表 -->

## 用户画像（Personas）

| 画像 ID | 角色名称 | 典型背景 | 核心诉求 | 技术水平 |
|---|---|---|---|---|
| P-01 | 测试工程师 | 负责防火墙功能测试，熟悉 TG/CLI 工具，有编写测试脚本经验 | 快速找到或设计可复用的原子操作，保证测试步骤标准化、可读 | 中级（了解 YAML/Python，不需深入设备 SDK） |
| P-02 | LLM Agent（自动化调度者） | 由 LLM 驱动的测试编排程序，读取 schema 元数据后自动组装操作序列 | 从机器可读的 schema 推断正确调用顺序、参数约束和失败处理策略 | N/A（以 schema 字段为输入，不假设人工干预） |
| P-03 | 原子操作库维护者 | 测试基础设施负责人，管理 schema 演进、版本发布和废弃流程 | 在不破坏已有调用方的前提下扩展/升级操作库；维护 CHANGELOG | 高级（熟悉 semver、设备 API、CI 流程） |

## 成功指标（Success Metrics）

| 指标 ID | 指标名称 | 度量方式 | 目标值 |
|---|---|---|---|
| SM-01 | 操作复用率 | 新测试用例引用已有原子操作的比例 | ≥ 60% 操作无需新建 |
| SM-02 | LLM 自主编排成功率 | LLM Agent 在不人工干预下完整执行测试用例序列的比率 | ≥ 90%（初版） |
| SM-03 | 失败可诊断率 | 操作失败时 diagnostic_snapshot 提供足够信息的比率 | ≥ 95% 失败带有 error_type + detail |
| SM-04 | Schema 合规率 | 原子操作 YAML 通过 schema validator 的比率 | 100%（CI 强制门禁） |
| SM-05 | 废弃操作覆盖率 | 废弃声明包含 since / reason 字段的比率 | 100% |

## 明确排除（Out of Scope）

<!-- 已知排除项，后续可补充 -->
- 完整可运行的端到端防火墙测试套件（用户新建独立仓库后自行实现）
- 防火墙功能测试的 test case 设计（由 ptm-tde 主流程负责）
- 多厂商适配层的完整实现代码（骨架仅展示接口契约，不含真实设备驱动）

## 使用场景列表

### UC-01：测试工程师浏览原子操作清单，为新测试用例选择可复用操作

| 字段 | 内容 |
|------|------|
| **使用角色** | P-01 测试工程师 |
| **阶段** | 阶段一（用例设计） |
| **触发条件** | 工程师开始编写新防火墙特性测试用例，需要了解已有原子操作库中哪些操作可直接复用 |
| **输入** | 已有原子操作库目录（YAML 文件集），测试需求描述文字 |
| **处理逻辑** | 1. 工程师按设备类型（fw / tg / linux / switch）浏览目录结构；2. 阅读各操作 YAML 中 name、description、parameters、output_schema 字段；3. 按命名规范（`{设备}_{动词}_{目标}`）过滤匹配意图的操作；4. 确认 idempotent 标记满足用例需求；5. 记录选中的操作 ID 列表，写入测试用例 YAML 的 steps 字段 |
| **输出/结果** | 测试用例 YAML 中 steps 字段填入可复用的原子操作 ID 列表 |
| **前置条件** | 原子操作库至少已有一个厂商（TGFW）的操作 YAML；命名规范文档可读 |
| **排除情况** | 不包含跨厂商操作的搜索与比对；不包含自动推荐算法 |
| **引用规则** | B2-03（命名规范），I2-01（幂等性标记），P1-01～P1-04（参数字段结构） |

**处理流程（文字描述）：**
1. 步骤一：打开 `delivery/scaffold/` 目录，按设备前缀（fw_ / tg_ / linux_ / sw_）浏览 YAML 文件
2. 步骤二：阅读每个 YAML 的 `name`、`description`、`parameters` 字段，判断是否匹配当前测试意图
3. 步骤三：检查 `idempotent` 字段，如用例对幂等性有要求则过滤掉 `idempotent: false` 的操作
4. 步骤四：将选中操作的 `operation_id` 记录到测试用例 YAML 的 `steps` 数组
5. **异常路径**：若未找到匹配操作，转入 UC-02（新建操作）；若操作已标记 `deprecated: true`，应检查 `replaced_by` 字段找到替代操作

---

### UC-02：测试工程师为新功能设计新的原子操作（编写 schema YAML 草稿）

| 字段 | 内容 |
|------|------|
| **使用角色** | P-01 测试工程师 |
| **阶段** | 阶段一（用例设计） |
| **触发条件** | UC-01 中未找到可复用操作，需要新建一个原子操作定义 |
| **输入** | 目标设备类型、操作语义描述（自然语言）、参数列表草稿、预期返回信息 |
| **处理逻辑** | 1. 依据 B2-03 命名规范生成 `operation_id`；2. 填写 `description`、`device`、`parameters`（按 P1-01～P1-04 约束）；3. 填写 `on_failure`（默认 `return_error`）；4. 声明 `idempotent`（默认 `true`）；5. 填写 `output_schema` 描述 result 对象结构；6. 填写 `schema_version: "1.0.0"` |
| **输出/结果** | 一份符合 schema 规范的原子操作 YAML 草稿，准备进入 UC-03 自检 |
| **前置条件** | 工程师已阅读命名规范（B2-03）和字段约束规范（P1 / R1 / B2） |
| **排除情况** | 不含真实 CLI 实现代码（留给阶段二 UC-04）；不含多设备联合操作 |
| **引用规则** | B2-01～B2-06，I2-01～I2-02，P1-01～P1-04，R1-01～R1-04，V1-01 |

**处理流程（文字描述）：**
1. 步骤一：确认操作主体设备（`device` 字段），遵循 B2-02 单设备原则
2. 步骤二：按 B2-03 命名规范生成 `operation_id`（如 `fw_ensure_acl_rule_exists`）
3. 步骤三：按 P1-01～P1-04 逐个定义 `parameters` 数组条目（name / type / required / description）
4. 步骤四：声明 `on_failure`（默认 `return_error`）和 `idempotent`（默认 `true`）
5. 步骤五：填写 `output_schema` 中 result 的预期字段
6. **异常路径**：若操作涉及跨设备，拆分为多个单设备操作；若参数 type 为 json，需补充 `schema_ref`（P1-03）

---

### UC-03：测试工程师对照规范自检 schema YAML 草稿

| 字段 | 内容 |
|------|------|
| **使用角色** | P-01 测试工程师 |
| **阶段** | 阶段一（用例设计） |
| **触发条件** | 完成 UC-02 草稿后，提交到操作库前执行合规检查 |
| **输入** | UC-02 产出的原子操作 YAML 草稿，schema 校验器（如 jsonschema / custom validator） |
| **处理逻辑** | 1. 运行 schema validator 验证必填字段完整性；2. 检查 `operation_id` 命名规范符合 B2-03；3. 检查 `required: true` 参数无 `default` 字段（P1-04）；4. 若 `idempotent: false`，验证 `side_effects` 已填写（I2-02）；5. 验证 `on_failure` 枚举值合法（B2-04）；6. 校验通过则提交入库 |
| **输出/结果** | 校验通过：YAML 文件提交到操作库；校验失败：带有具体违规字段的错误报告 |
| **前置条件** | schema validator 工具可用；操作库 CI 门禁已配置（SM-04） |
| **排除情况** | 不含真实设备冒烟验证；不含 CLI 功能测试 |
| **引用规则** | B2-03～B2-06，I2-01～I2-02，P1-04，R1-01～R1-04，V1-01 |

**处理流程（文字描述）：**
1. 步骤一：执行 `uv run python scripts/validate_schema.py <yaml_path>` 进行结构校验
2. 步骤二：检查 operation_id 是否符合 `{设备}_{动词}_{目标}` 命名规范
3. 步骤三：验证所有 `required: true` 参数无 `default` 字段
4. 步骤四：若 `idempotent: false`，确认 `side_effects` 字段非空
5. **异常路径**：validator 返回 `{status: "failed", error_type: "schema_violation", detail: "<字段列表>"}` 时，工程师按 detail 修正后重新执行步骤一

---

### UC-04：测试工程师按 schema 实现原子操作 CLI 工具

| 字段 | 内容 |
|------|------|
| **使用角色** | P-01 测试工程师 |
| **阶段** | 阶段二（CLI 实现） |
| **触发条件** | UC-03 校验通过的 schema YAML 已入库，进入 CLI 实现阶段 |
| **输入** | 已合规的原子操作 YAML、对应设备骨架代码（delivery/scaffold/{device}/）、设备 API 文档 |
| **处理逻辑** | 1. 从骨架复制对应设备类型的 Python/CLI 模板；2. 按 schema `parameters` 字段生成函数签名（强类型，P1 规范）；3. 实现核心操作逻辑（调用设备 REST API 或 CLI 命令）；4. 按 R1 规范构造返回值（status / result / metadata）；5. 在 metadata 中填写 duration_ms / device / timestamp；6. 添加 `diagnostic_snapshot` 辅助调用点（详见 UC-06） |
| **输出/结果** | 可执行的 Python 函数或 CLI 入口，返回值符合 R1 规范 |
| **前置条件** | 骨架代码模板存在（delivery/scaffold/）；设备测试环境或 Mock 可用 |
| **排除情况** | 不含自动代码生成；不含设备 SDK 封装（仅调用已有接口） |
| **引用规则** | P1-01～P1-04，R1-01～R1-04，B2-04～B2-06，O1（diagnostic_snapshot） |

**处理流程（文字描述）：**
1. 步骤一：从 `delivery/scaffold/{fw|tg|linux|switch}/` 复制骨架文件到工作目录
2. 步骤二：按 schema YAML 中 `parameters` 定义，生成函数参数（强类型注解）
3. 步骤三：实现操作逻辑，调用对应设备 API；内部不做自动重试（B2-06）
4. 步骤四：构造 `{status, result, metadata}` 返回对象，确保 metadata.device 与 schema 一致（R1-04）
5. 步骤五：在失败分支中调用 `diagnostic_snapshot()` 并将结果追加到 `metadata`
6. **异常路径**：设备 API 超时时返回 `{status: "timeout", ...}` 而非抛出异常；编排层负责重试（B2-06）

---

### UC-05：测试工程师为非幂等操作补充标记与行为说明

| 字段 | 内容 |
|------|------|
| **使用角色** | P-01 测试工程师 |
| **阶段** | 阶段二（CLI 实现） |
| **触发条件** | 实现的操作具有不可逆副作用（如删除 ACL 规则、清空会话表），需要显式标记为非幂等 |
| **输入** | 已有操作 YAML 草稿，操作副作用的文字描述 |
| **处理逻辑** | 1. 将 YAML 中 `idempotent` 字段设置为 `false`（I2-01）；2. 填写 `side_effects` 字段，描述对系统持久状态的影响（I2-02）；3. 在 CLI 实现中增加操作前确认逻辑（或 dry_run 参数）；4. 在代码注释中标注"非幂等，重复调用可能导致 {具体影响}"；5. 更新操作库 CHANGELOG.md（V1-07） |
| **输出/结果** | YAML 中 `idempotent: false` + `side_effects` 已填写；CLI 实现含防重调用保护注释 |
| **前置条件** | 工程师已确认操作语义确实是非幂等的（非误判） |
| **排除情况** | 不含自动幂等化改造；不含事务回滚机制 |
| **引用规则** | I2-01，I2-02，V1-07 |

**处理流程（文字描述）：**
1. 步骤一：确认操作是否真正需要 `idempotent: false`（例如 fw_delete_all_sessions 不可安全重复执行）
2. 步骤二：在 YAML 中设置 `idempotent: false` 并填写 `side_effects` 描述
3. 步骤三：在 CLI 函数入口添加注释警告，或增加 `--confirm` / `dry_run` 参数
4. 步骤四：更新 CHANGELOG.md，记录非幂等标记新增的版本条目
5. **异常路径**：若 schema validator 检测到 `idempotent: false` 但 `side_effects` 为空，返回 `{error_type: "missing_side_effects"}` 阻止入库

---

### UC-06：测试工程师在 CLI 中实现 diagnostic_snapshot 辅助函数

| 字段 | 内容 |
|------|------|
| **使用角色** | P-01 测试工程师 |
| **阶段** | 阶段二（CLI 实现） |
| **触发条件** | 操作 CLI 实现需要在失败时采集设备当前状态快照，以支持 LLM Agent 和人工排查（O1 规范） |
| **输入** | 目标设备类型、设备 API 能力列表（可查询的状态接口） |
| **处理逻辑** | 1. 实现 `get_diagnostic_snapshot(device_id) -> list[dict]` 函数；2. 每个 dict 代表一个状态维度（如 ACL 列表、会话表条数、接口状态）；3. 每个 dict 包含 `key`、`value`、`collected_at` 字段；4. 在 UC-04 的失败分支调用该函数，将结果写入 `metadata.diagnostic_snapshot`；5. 确保快照采集不修改设备状态（读操作） |
| **输出/结果** | CLI 实现中的 `diagnostic_snapshot` 函数；操作失败时 metadata 中含有状态快照数组 |
| **前置条件** | 目标设备提供只读状态查询接口；工程师了解哪些状态维度与当前操作相关 |
| **排除情况** | 不含完整的设备状态持久化存储；快照仅用于当次调试，不写入日志系统 |
| **引用规则** | O1（diagnostic_snapshot 数组规范），R1-02（metadata 固定字段） |

**处理流程（文字描述）：**
1. 步骤一：列出与当前操作相关的设备只读查询接口（如 `GET /acl/rules`、`GET /session/count`）
2. 步骤二：实现 `get_diagnostic_snapshot(device_id)` 函数，并发或顺序调用各查询接口
3. 步骤三：将每项查询结果格式化为 `{key, value, collected_at}` dict
4. 步骤四：在 UC-04 失败分支中调用该函数，将结果赋值给 `metadata.diagnostic_snapshot`
5. **异常路径**：若快照采集接口本身失败，记录 `{key: "<接口名>", value: "collection_failed", collected_at: "<ts>"}` 而非抛出异常，保证主流程错误返回不被快照失败覆盖

---

### UC-07：LLM Agent 读取 schema 元数据，根据测试用例 YAML 自动编排原子操作调用序列

| 字段 | 内容 |
|------|------|
| **使用角色** | P-02 LLM Agent |
| **阶段** | 阶段三（LLM 自动化调度） |
| **触发条件** | LLM Agent 接收到一份测试用例 YAML，需要将其中的 steps（原子操作 ID 列表）展开为可执行调用序列 |
| **输入** | 测试用例 YAML（含 steps 数组和 tight_sequence 声明）、原子操作库 schema 目录 |
| **处理逻辑** | 1. 读取测试用例 YAML 中的 steps 数组；2. 为每个 operation_id 读取对应 schema YAML，提取 parameters、idempotent、on_failure 字段；3. 按 steps 顺序填充参数值，构造调用队列；4. 识别 tight_sequence 组，标记为不可并行组；5. 按序调用各原子操作函数，传入强类型参数；6. 收集每步返回值，写入执行报告 |
| **输出/结果** | 各原子操作按序执行完毕；执行报告包含每步 status / result / metadata |
| **前置条件** | 原子操作库 schema 机器可读（YAML/JSON）；参数类型声明明确（P1 规范） |
| **排除情况** | 不含 LLM 自行推断操作顺序（顺序由测试用例 YAML 声明）；不含跨测试用例并发调度 |
| **引用规则** | P1-01～P1-04，E2-02（tight_sequence），R1-01～R1-04，I2-01 |

**处理流程（文字描述）：**
1. 步骤一：解析测试用例 YAML，提取 `steps` 数组和可选的 `tight_sequence` 字段
2. 步骤二：逐个 operation_id 加载对应 schema YAML，验证参数类型后构造调用参数对象
3. 步骤三：识别 `tight_sequence` 中的操作组，标记为顺序约束（不可并行化）
4. 步骤四：按序调用各操作函数，等待每步完成后再继续（tight_sequence 组内串行）
5. 步骤五：汇总所有步骤返回值，写入执行报告
6. **异常路径**：任意步骤返回 `status: "failed"` 时，按该步骤 `on_failure` 字段决策（转入 UC-08）

---

### UC-08：LLM Agent 处理原子操作失败，根据 error_type 决策重试 / 终止 / 采集诊断

| 字段 | 内容 |
|------|------|
| **使用角色** | P-02 LLM Agent |
| **阶段** | 阶段三（LLM 自动化调度） |
| **触发条件** | UC-07 执行序列中某步原子操作返回 `status: "failed"` 或 `status: "timeout"` |
| **输入** | 失败原子操作的返回值（含 error_type、detail、metadata.diagnostic_snapshot） |
| **处理逻辑** | 1. 读取 `on_failure` 字段判断决策策略；2. 若 `on_failure: fail_fast`，立即终止整个序列，输出失败报告；3. 若 `on_failure: return_error`，记录错误后继续执行后续步骤（如适用）；4. 若 `on_failure: retry_by_caller`，按编排层策略重试（最多 N 次，次数由测试用例声明）；5. 提取 `metadata.diagnostic_snapshot` 输出到诊断日志；6. 更新执行报告中该步骤状态 |
| **输出/结果** | 执行报告更新当前步骤失败详情；后续操作按策略继续或终止；diagnostic_snapshot 写入诊断日志 |
| **前置条件** | 失败操作的 schema 中包含有效的 `on_failure` 枚举值；diagnostic_snapshot 已由操作 CLI 采集（UC-06） |
| **排除情况** | 不含 LLM 自动修复操作逻辑；不含多步骤回滚（单操作内部无事务） |
| **引用规则** | B2-04～B2-06，R1-01～R1-03，O1（diagnostic_snapshot） |

**处理流程（文字描述）：**
1. 步骤一：检查失败返回值中的 `error_type`（如 `device_unreachable` / `policy_conflict` / `timeout`）
2. 步骤二：读取 schema 中 `on_failure` 字段，按枚举决策：fail_fast → 中止序列，return_error → 继续，retry_by_caller → 重试
3. 步骤三：将 `metadata.diagnostic_snapshot` 数组输出到诊断日志文件
4. 步骤四：更新执行报告该步骤为 `status: "failed"` 并附 error_type + detail
5. **异常路径**：若 `on_failure` 字段缺失或枚举值非法，LLM Agent 默认按 `fail_fast` 处理并告警

---

### UC-09：LLM Agent 在 tight_sequence 约束下保证跨设备操作时序

| 字段 | 内容 |
|------|------|
| **使用角色** | P-02 LLM Agent |
| **阶段** | 阶段三（LLM 自动化调度） |
| **触发条件** | 测试用例 YAML 中声明了 `tight_sequence`，涉及跨设备（如 fw + tg）的操作组 |
| **输入** | 测试用例 YAML 中的 `tight_sequence: [op_id_1, op_id_2, ...]`，各操作 schema（含 device 字段） |
| **处理逻辑** | 1. 识别 tight_sequence 列表中每个操作对应的 device；2. 确认该组内操作为串行依赖，不可并发；3. 按声明顺序逐步发起各操作调用；4. 每步等待返回后方继续下一步；5. 任意步骤失败则触发 UC-08 失败处理；6. 记录各设备操作的完成时间戳（metadata.timestamp）供时序审计 |
| **输出/结果** | tight_sequence 组内操作按声明顺序完成；执行报告含每步时间戳 |
| **前置条件** | 测试用例 YAML 中 tight_sequence 列出的 operation_id 均已在操作库中存在 |
| **排除情况** | 不含自动推断跨设备操作的因果顺序；顺序必须由测试用例显式声明 |
| **引用规则** | E2-01（单设备粒度），E2-02（tight_sequence），R1-02（metadata.timestamp） |

**处理流程（文字描述）：**
1. 步骤一：从测试用例 YAML 读取 `tight_sequence` 数组，验证所有 operation_id 存在于操作库
2. 步骤二：标记该组为顺序约束，调度器不得并行化组内任何两个操作
3. 步骤三：按数组顺序逐步发起调用，等待每步 status 返回后继续
4. 步骤四：记录每步 `metadata.timestamp`，供事后时序审计
5. **异常路径**：若 tight_sequence 中某 operation_id 在操作库中不存在，返回 `{status: "failed", error_type: "unknown_operation_id"}` 并中止序列

---

### UC-10：维护者升级原子操作 schema 字段，标记 deprecated 并发布新版本

| 字段 | 内容 |
|------|------|
| **使用角色** | P-03 原子操作库维护者 |
| **阶段** | 演进与维护 |
| **触发条件** | 某原子操作的接口设计需要改变（参数重命名、字段新增、语义变更），无法向后兼容 |
| **输入** | 旧版操作 YAML，新版操作设计草稿，受影响调用方列表 |
| **处理逻辑** | 1. 创建新操作 YAML（新 operation_id 或新 schema_version）；2. 在旧操作 YAML 中添加 `deprecated: true`、`since`、`reason`、`replaced_by` 字段（V1-02～V1-05）；3. 新旧操作并存，直到满足 V1-06 保留周期（至少一个大版本）；4. 通知调用方（测试用例作者）迁移到新操作；5. 在 CHANGELOG.md 追加条目（V1-07）；6. CI 门禁在废弃期间允许使用旧操作，但产生 deprecation warning |
| **输出/结果** | 旧操作标记废弃；新操作入库；CHANGELOG.md 更新；调用方收到迁移通知 |
| **前置条件** | 维护者有操作库写权限；已确认新旧操作语义差异 |
| **排除情况** | 不含自动迁移测试用例 YAML（迁移由测试工程师手动执行）；不含破坏性删除（须满足保留周期） |
| **引用规则** | V1-01～V1-07，SM-05（废弃字段覆盖率 100%） |

**处理流程（文字描述）：**
1. 步骤一：创建新版操作 YAML，分配新 operation_id 或递增 schema_version
2. 步骤二：在旧操作 YAML 中设置 `deprecated: true`，填写 `since`（当前大版本）、`reason`（变更原因）、`replaced_by`（新操作 ID）
3. 步骤三：向所有引用旧操作 ID 的测试用例 YAML 发出迁移告知
4. 步骤四：在 CHANGELOG.md 追加废弃条目
5. 步骤五：等待一个大版本周期后，执行删除前再次检查是否存在仍在使用旧操作的测试用例
6. **异常路径**：若废弃操作 YAML 缺少 `since` 或 `reason` 字段，schema validator 返回 `{error_type: "incomplete_deprecation"}` 阻止发布

---

### UC-11：维护者新增设备类型扩展原子操作库

| 字段 | 内容 |
|------|------|
| **使用角色** | P-03 原子操作库维护者 |
| **阶段** | 演进与维护 |
| **触发条件** | 测试环境中引入新设备型号（如新交换机型号）或新设备类别，需要为其建立原子操作骨架 |
| **输入** | 新设备类型标识（device_prefix）、设备 API 文档、已有同类操作作为参考 |
| **处理逻辑** | 1. 在 `delivery/scaffold/` 下创建新设备目录；2. 复制最近似设备类型的骨架代码作为起点；3. 按新设备 API 调整实现细节；4. 编写最少 1 个完整原子操作 YAML（含 schema + CLI 实现）作为参考示例；5. 通过 UC-03 自检；6. 更新操作库 README，说明新设备类型的命名前缀（B2-03） |
| **输出/结果** | 新设备类型骨架目录存在于 `delivery/scaffold/`；至少一个参考原子操作 YAML 通过合规检查 |
| **前置条件** | 新设备 API 文档可获取；维护者熟悉骨架代码结构 |
| **排除情况** | 不含完整新设备原子操作库的实现（仅建立骨架和命名约定）；不含多厂商适配（当前仅 TGFW 生态） |
| **引用规则** | B2-02（device 字段单值），B2-03（命名规范），V1-01（schema_version 从 "1.0.0" 起） |

**处理流程（文字描述）：**
1. 步骤一：确认新设备前缀（如 `sw2_` 区别于已有 `sw_`），记录到命名规范文档
2. 步骤二：在 `delivery/scaffold/` 创建新设备目录，复制最近似骨架
3. 步骤三：按新设备 API 调整骨架中的 HTTP endpoint 或 CLI 命令调用
4. 步骤四：实现第一个参考操作（如 `sw2_ensure_vlan_exists`），通过 UC-03 自检
5. **异常路径**：若新设备前缀与已有前缀冲突，维护者必须在命名规范文档中记录消歧规则后方可继续

---

### UC-12：测试工程师 / LLM Agent 排查"操作返回成功但实际状态异常"问题

| 字段 | 内容 |
|------|------|
| **使用角色** | P-01 测试工程师 或 P-02 LLM Agent |
| **阶段** | 阶段三（LLM 自动化调度） / 演进与维护 |
| **触发条件** | 原子操作返回 `status: "success"`，但后续断言或设备实际状态与预期不符（虚假成功） |
| **输入** | 操作返回值（含 metadata.diagnostic_snapshot）、预期设备状态描述 |
| **处理逻辑** | 1. 读取操作返回值中 `metadata.diagnostic_snapshot` 数组；2. 对比 snapshot 中各维度实际值与测试用例预期值；3. 识别差异字段（如 ACL 规则条数不符、会话状态非预期）；4. 判断是操作 CLI 实现缺陷（未正确等待设备生效）还是操作 schema 逻辑缺陷；5. 反馈给 P-03 维护者或测试工程师修复；6. 若 diagnostic_snapshot 缺失，调用 UC-06 函数补采集 |
| **输出/结果** | 识别出差异字段；诊断结论（操作缺陷 / 设备延迟 / 测试用例断言错误）；修复建议 |
| **前置条件** | 操作 CLI 实现已包含 diagnostic_snapshot 采集逻辑（UC-06）；snapshot 字段与测试断言字段存在对应关系 |
| **排除情况** | 不含自动修复操作逻辑；诊断结论仅为人工或 LLM 决策提供依据 |
| **引用规则** | O1（diagnostic_snapshot），R1-01（status 字段），B2-05（失败返回结构），UC-06 |

**处理流程（文字描述）：**
1. 步骤一：从操作返回值提取 `metadata.diagnostic_snapshot` 数组
2. 步骤二：将每个 snapshot 条目的 `key / value` 与测试用例预期状态表逐项对比
3. 步骤三：标记差异字段，判断差异性质（设备延迟生效 / CLI 实现缺陷 / schema 逻辑问题）
4. 步骤四：若为设备延迟，建议调整 `default_timeout_s` 或 `poll_interval_s` 参数
5. 步骤五：若为 CLI 实现缺陷，提交 bug 报告给维护者（P-03），关联相关操作 operation_id
6. **异常路径**：若 diagnostic_snapshot 为空数组，说明快照未采集（操作 CLI 未实现 UC-06），需要先补实现再复现问题

---

<!-- coverage-checklist: begin -->
## 附录：覆盖自检表

| 维度 ID | 维度名称 | 状态 | 涉及场景 | 备注 |
|---|---|---|---|---|
| D1 | 用户维度 | 已覆盖 | UC-01～UC-06（P-01 测试工程师），UC-07～UC-09（P-02 LLM Agent），UC-10～UC-11（P-03 维护者），UC-12（P-01/P-02） | 三类画像 P-01/P-02/P-03 均已在场景中出现；三阶段分工明确 |
| D2 | 任务维度 | 已覆盖 | UC-01（浏览/选择），UC-02～UC-03（设计/自检），UC-04～UC-06（CLI 实现），UC-07～UC-09（自动调度），UC-10～UC-11（版本演进），UC-12（诊断排查） | 覆盖设计、实现、调度、维护、诊断五类主要任务 |
| D3 | 动机维度 | 已覆盖 | UC-01（复用提效），UC-02（标准化），UC-07（人机协作），UC-10（演进兼容），UC-12（可观测性） | 核心动机：标准化原子操作描述 + 人机协作 + 自动化调度可靠性 |
| D4 | 时间维度 | 已覆盖 | UC-01～UC-03（阶段一：用例设计），UC-04～UC-06（阶段二：CLI 实现），UC-07～UC-09（阶段三：LLM 调度），UC-10～UC-11（演进维护） | 三阶段演进路径在场景中有明确时间锚点，无时序跳跃 |
| D5 | 环境维度 | 已覆盖 | UC-04（TGFW REST API 设备），UC-05（含副作用操作的防火墙），UC-06（Linux Mock / TGFW 只读接口），UC-11（新设备类型扩展） | 单厂商 TGFW 生态；fw / tg / linux / switch 四类设备均有体现 |
| D6 | 方式维度 | 已覆盖 | UC-02～UC-03（YAML schema 编写+校验），UC-04（Python CLI），UC-07～UC-09（LLM 读取 schema 元数据后编排调用） | 覆盖 schema YAML / Python CLI / LLM 调度三种使用方式 |
| D7 | 异常维度 | 已覆盖 | UC-03（schema 校验失败），UC-05（非幂等误用），UC-07～UC-08（操作失败 error_type 决策），UC-09（unknown_operation_id），UC-10（废弃字段缺失），UC-12（虚假成功诊断） | B2 错误码（fail_fast / return_error / retry_by_caller）+ O1 诊断快照均有场景覆盖 |
| D8 | 集成维度 | 已覆盖 | UC-07～UC-09（LLM 读取操作库 schema 集成），UC-09（tight_sequence 跨设备时序集成），UC-11（新设备类型骨架集成） | 单厂商 TGFW；与测试用例 YAML 调度层的集成契约（tight_sequence）已体现；multi-device 通过 tight_sequence 而非操作内部集成 |
<!-- coverage-checklist: end -->

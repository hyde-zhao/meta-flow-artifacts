---
status: confirmed
version: "1.0"
confirmed_by: user
confirmed_at: "2026-05-09"
ready_for_design: true
source_use_cases: [UC-FWATOM-01, UC-FWATOM-02, UC-FWATOM-03, UC-FWATOM-04, UC-FWATOM-05]
source_clarification_rounds: [14, 15, 16, 17, 18]
---

# 防火墙测试原子操作规范 — 结构化需求文档

> 版本：1.0 | 日期：2026-05-09 | 状态：confirmed
> 本文档基于 CLARIFICATION-LOG.md 第 14–18 轮澄清结论生成，所有 BLOCKING/REQUIRED/OPTIONAL 问题已全部关闭。

---

## §1 业务背景与目标

### 1.1 背景

TGFW（单厂商防火墙）功能测试当前缺乏统一的原子操作描述规范。测试工程师在设计测试用例时，需要手工描述每个操作步骤，导致：

- 操作描述粒度不一致（有人写"配置 ACL"，有人写"调用 fw_create_acl_rule API"）
- 幂等性、超时、错误处理等关键属性缺乏显式声明，执行时行为不可预期
- 跨设备联动（防火墙 + 流量发生器 + Linux Mock + 交换机）的时序依赖无法在用例层表达
- 阶段三 LLM 自动调度时，缺乏机器可读的结构化元数据，无法可靠推理操作语义

### 1.2 目标

本规范旨在定义一套**防火墙测试原子操作描述规范**，覆盖：

1. 统一的原子操作元数据 schema（YAML 格式，人机双可读）
2. 4 类设备（防火墙、流量发生器 TG、Linux Mock、交换机）的原子操作骨架代码
3. 测试用例层 `tight_sequence` 模板，用于描述跨设备时序依赖
4. 工程师手册：命名规范、错误码规范、幂等性声明规则、诊断快照使用指南

### 1.3 三阶段演进模型

本规范服务于以下三阶段演进路径（Q-02 决策）：

| 阶段 | 描述 | 规范的作用 |
|------|------|-----------|
| 阶段一（用例设计） | 测试工程师读取原子操作清单，为新功能设计新的原子操作 | 提供 schema 模板和命名规范，指导工程师填写 |
| 阶段二（CLI 实现） | 测试工程师将原子操作实现为 CLI 工具，人工/半自动执行 | 骨架代码作为实现起点，schema 字段与 CLI 参数一一对应 |
| 阶段三（LLM 调度） | 所有工具实现后由 LLM 推理调度 | schema 元数据作为 LLM 的结构化输入，支持可靠推理 |

### 1.4 范围声明

- **IN-SCOPE**：原子操作 schema 定义、命名规范、错误码规范、4 类设备骨架代码、YAML 模板、工程师手册
- **OUT-OF-SCOPE**：完整端到端 CLI 实现、测试执行调度系统、缺陷管理、多厂商适配层、知识库入库

---

## §2 利益相关方与使用者

### 2.1 用户画像

| 画像 ID | 角色名称 | 典型背景 | 核心诉求 | 技术水平 |
|---------|---------|---------|---------|---------|
| P-01 | 测试工程师（用例设计） | 负责为 TGFW 新功能设计测试用例，需要查阅和新增原子操作 | 快速找到可复用的原子操作，按规范填写新操作的 schema | 中级 |
| P-02 | 测试工程师（CLI 实现） | 负责将原子操作 schema 实现为可执行的 CLI 工具 | 骨架代码可直接作为实现起点，schema 字段与 CLI 参数对应清晰 | 中级 |
| P-03 | LLM Agent（自动调度） | 阶段三中由 LLM 读取 schema 元数据，推理并编排原子操作序列 | schema 字段语义明确、机器可读，支持可靠推理和错误处理 | N/A（机器） |
| P-04 | 测试架构师 | 负责审核原子操作规范的完整性、一致性和可维护性 | 规范有明确的版本演进策略，废弃操作有追溯记录 | 高级 |

### 2.2 成功指标

| 指标 ID | 指标名称 | 度量方式 | 目标值 |
|---------|---------|---------|--------|
| SM-01 | schema 字段完整率 | 新增原子操作 schema 中必填字段的填写率 | 100% |
| SM-02 | 命名规范符合率 | 原子操作名称符合 `设备_动词_目标` 格式的比例 | >= 95% |
| SM-03 | 骨架代码可执行率 | 4 类设备骨架代码在 dry-run 模式下无语法错误的比例 | 100% |
| SM-04 | 幂等性声明覆盖率 | 所有非幂等操作显式声明 `idempotent: false` 的比例 | 100% |
| SM-05 | LLM 推理准确率 | LLM Agent 基于 schema 推理操作用法的准确率（人工抽样验证） | >= 90% |

---

## §3 交付物清单

| 交付物 ID | 名称 | 格式 | 说明 |
|----------|------|------|------|
| D-01 | 原子操作元数据 schema 定义 | YAML + Markdown | 完整字段集定义，含字段说明、类型、必填性、枚举值 |
| D-02 | 命名规范文档 | Markdown | `设备_动词_目标` 格式规则、设备前缀枚举、动词词汇表 |
| D-03 | 错误码规范文档 | Markdown | `error_type` 枚举值定义，含含义、触发条件、处理建议 |
| D-04 | 防火墙（fw）原子操作骨架 | Python | 含 3–5 个典型操作的占位桩函数，返回合规 R1 结构 |
| D-05 | 流量发生器（tg）原子操作骨架 | Python | 含 3–5 个典型操作的占位桩函数，返回合规 R1 结构 |
| D-06 | Linux Mock（mock）原子操作骨架 | Python | 含 2–3 个典型操作的占位桩函数，返回合规 R1 结构 |
| D-07 | 交换机（sw）原子操作骨架 | Python | 含 2–3 个典型操作的占位桩函数，返回合规 R1 结构 |
| D-08 | 测试用例层 YAML 模板 | YAML | 含 `tight_sequence` 字段的完整测试用例模板示例 |
| D-09 | 工程师手册 | Markdown | 覆盖：如何新增原子操作、如何声明幂等性、如何使用诊断快照、如何处理版本演进 |
| D-10 | CHANGELOG.md 初始版本 | Markdown | 原子操作库变更记录，含初始版本条目 |

---

## §4 功能需求

### FR-1：schema 字段集定义

**编号**：R-F-001
**标题**：原子操作元数据 schema 完整字段集
**描述**：每个原子操作必须以 YAML 文档描述，包含以下字段集：

```
必填字段（presence: required）：
  - op_id          : 字符串，唯一操作标识，格式 <设备>_<动词>_<目标>
  - device_type    : 枚举，[fw, tg, mock, sw]
  - description    : 字符串，单句自然语言描述，≤ 120 字符
  - parameters     : 映射，键为参数名，值含 type/required/description
  - returns        : 映射，R1 结构（status/data/error_type/diag_snapshot_ref）
  - idempotent     : 布尔，是否幂等
  - timeout_ms     : 整数，预期超时上限（毫秒）
  - error_types    : 列表，该操作可能触发的 error_type 枚举值

可选字段（presence: optional）：
  - preconditions  : 列表，前置约束描述
  - postconditions : 列表，后置断言描述
  - tags           : 列表，自由标签
  - deprecated     : 布尔，是否废弃（默认 false）
  - deprecated_by  : 字符串，替代操作的 op_id
  - since_version  : 字符串，首次引入版本号
```

**依据**：Q-04（schema 字段优先级）、Q-05（R1 统一返回结构）、Q-09（幂等声明）、Q-11（超时字段）
**验收准则**：
- Given 工程师新增原子操作 YAML 文件时，When 运行 schema 校验器，Then 所有 `required` 字段缺失均报错、所有字段类型错误均报错，通过率 = 100%
- Given schema 定义文档，When 人工核查，Then 字段名、类型、枚举值与本文档描述完全一致，无额外未声明字段

---

### FR-2：命名规范

**编号**：R-F-002
**标题**：原子操作命名规范 `设备_动词_目标`
**描述**：

所有原子操作的 `op_id` 必须严格遵循 `设备_动词_目标` 三段式格式：

- **设备前缀**：固定枚举，[fw, tg, mock, sw]
- **动词**：来自受控词汇表，初始集合包括：
  - `create`（创建资源）
  - `delete`（删除资源）
  - `update`（修改资源属性）
  - `apply`（下发/生效策略）
  - `rollback`（回滚到上一个已知状态）
  - `verify`（读取状态做断言）
  - `start`（启动流量/进程）
  - `stop`（停止流量/进程）
  - `reset`（恢复出厂/默认状态）
  - `capture`（抓取诊断快照）
- **目标**：当前设备上的资源或操作对象，使用小写下划线命名

示例合规命名：`fw_create_acl_rule`、`tg_start_traffic`、`mock_verify_route_table`、`sw_apply_vlan_config`

不合规示例：`createAclRule`（缺少设备前缀）、`fw-create-acl`（使用横线分隔）、`FW_ADD_RULE`（大写）

工程师手册须提供新词汇申请流程（PR Review 机制）。

**依据**：Q-06（命名约定：`设备_动词_目标`）
**验收准则**：
- Given 原子操作库中所有 op_id，When 运行命名校验脚本，Then 符合 `^(fw|tg|mock|sw)_[a-z]+_[a-z_]+$` 正则格式的比例 >= 95%
- Given 工程师手册，When 人工审查，Then 设备前缀枚举、动词词汇表、不合规示例三项均存在，且动词词汇表 >= 10 个词条

---

### FR-3：错误码规范

**编号**：R-F-003
**标题**：统一错误码 `error_type` 枚举集
**描述**：

所有原子操作的返回结构 R1 中，当 `status = "error"` 时，必须填写 `error_type` 字段。`error_type` 的枚举值定义如下（初始集合，可扩展）：

| error_type | 含义 | 典型触发条件 | 处理建议 |
|---|---|---|---|
| `TIMEOUT` | 操作在 `timeout_ms` 内未完成 | 设备响应慢、网络抖动 | 重试一次；记录诊断快照 |
| `DEVICE_UNREACHABLE` | 目标设备无法连接 | 网络中断、设备重启 | 终止用例；通知运维 |
| `INVALID_PARAM` | 输入参数不合规 | 参数类型错误、枚举值越界 | 修正参数后重试 |
| `RESOURCE_CONFLICT` | 目标资源已存在或被占用 | 重复创建、并发竞争 | 先删除再创建；或改用 upsert |
| `RESOURCE_NOT_FOUND` | 目标资源不存在 | 删除不存在的规则 | 检查前置条件；幂等操作可忽略 |
| `PERMISSION_DENIED` | 无权限执行该操作 | 凭证过期、权限不足 | 检查凭证；上报测试架构师 |
| `STATE_MISMATCH` | 设备当前状态不满足前置条件 | 操作依赖未就绪的资源 | 检查 `tight_sequence` 时序；先执行前置操作 |
| `INTERNAL_ERROR` | 设备/工具内部异常 | 未知错误、固件 Bug | 记录诊断快照；上报缺陷 |
| `DRY_RUN` | dry-run 模式未实际执行 | `--dry-run` 标志启用 | 仅用于验证参数合规性；不计入真实执行结果 |

规范文档须包含完整枚举表及"如何申请新增 error_type"的流程说明。

**依据**：Q-07（错误处理：原子操作必须有统一错误码）
**验收准则**：
- Given 所有原子操作骨架代码，When 以错误路径触发（如传入无效参数），Then 返回的 `error_type` 必须是枚举集合内的合法值，非合法值比例 = 0%
- Given 错误码规范文档，When 人工审查，Then 枚举表 >= 9 个条目，每条目含含义、触发条件、处理建议三列

---

### FR-4：幂等性显式声明规则

**编号**：R-F-004
**标题**：非幂等操作必须显式声明 `idempotent: false`
**描述**：

幂等性声明规则：

1. 默认值：schema 校验器不设 `idempotent` 默认值，**必须显式填写** `true` 或 `false`，缺失则报 schema 校验错误。
2. 幂等操作（`idempotent: true`）：对同一输入重复执行，结果与设备状态与首次执行相同。典型示例：`fw_verify_acl_rule`（只读操作）。
3. 非幂等操作（`idempotent: false`）：重复执行可能导致状态累积或错误。典型示例：`tg_start_traffic`（重复启动会产生多份流量流）。
4. 工程师手册中须包含一节"幂等性判定指南"，提供判定树：
   - 操作是否只读 → 若是，可声明 `true`
   - 操作是否具有"先检查后创建"逻辑 → 若是，可声明 `true`，但须在 `description` 中注明
   - 其余情况默认声明 `false`

**依据**：Q-09（幂等性：非幂等操作需显式声明 `idempotent: false`）
**验收准则**：
- Given 原子操作库中所有操作，When 运行幂等声明覆盖率检查脚本，Then `idempotent` 字段显式填写率 = 100%（无缺失）
- Given 工程师手册，When 人工审查，Then "幂等性判定指南"小节存在，含判定逻辑和 >= 2 个示例

---

### FR-5：`tight_sequence` 跨设备时序模板

**编号**：R-F-005
**标题**：测试用例层 `tight_sequence` 时序依赖声明
**描述**：

测试用例 YAML 模板中须包含 `tight_sequence` 顶层字段，用于描述跨设备原子操作的时序约束：

```yaml
tight_sequence:
  enabled: true          # 布尔，是否启用严格时序模式
  steps:
    - step: 1
      op_id: fw_apply_policy
      device: fw-01
      wait_ms: 0         # 执行前等待（毫秒），默认 0
      on_error: abort    # 枚举：[abort, skip, retry_once]
    - step: 2
      op_id: tg_start_traffic
      device: tg-01
      wait_ms: 500
      on_error: abort
    - step: 3
      op_id: fw_verify_hit_count
      device: fw-01
      wait_ms: 2000
      on_error: abort
```

字段语义：
- `enabled`：`false` 时跳过时序约束，步骤可乱序或并行执行
- `steps[].wait_ms`：前一步完成后的等待时长，用于补偿设备状态传播延迟（Q-10 决策：等待时长以 `wait_ms` 字段嵌入模板）
- `steps[].on_error`：当该步骤失败时的处理策略

**依据**：Q-03（是否保留 tight_sequence：是，保留为测试用例层描述符）、Q-10（wait_ms 字段策略）
**验收准则**：
- Given YAML 模板文件（D-08），When 人工审查，Then 含完整 `tight_sequence` 结构示例，steps 数量 >= 3，on_error 枚举值完整
- Given schema 校验器，When 验证含 `tight_sequence` 的测试用例 YAML，Then 步骤字段缺失均报错，通过率 = 100%

---

### FR-6：超时字段与处理策略

**编号**：R-F-006
**标题**：原子操作 `timeout_ms` 字段及超时处理行为
**描述**：

1. 每个原子操作 schema 中必须声明 `timeout_ms`（整数，毫秒），表示该操作的预期超时上限。
2. `timeout_ms` 参考基准（工程师手册须给出默认值参考表）：
   - 防火墙 REST API 调用：5000 ms
   - 防火墙 CLI 命令执行：10000 ms
   - 流量发生器启/停：3000 ms
   - Linux Mock 命令执行：2000 ms
   - 交换机 CLI 配置：8000 ms
3. 骨架代码须在操作执行逻辑中预留超时处理占位注释，格式为：
   ```python
   # TODO: enforce timeout_ms={timeout_ms}; raise TimeoutError on exceed
   ```
4. 当操作超时时，骨架代码返回 R1 结构中 `error_type = "TIMEOUT"`，并触发诊断快照采集（FR-7）。

**依据**：Q-11（超时字段：每个操作必须声明 timeout_ms）
**验收准则**：
- Given 所有原子操作 schema YAML，When 运行 schema 校验，Then `timeout_ms` 字段存在且为正整数的比例 = 100%
- Given 工程师手册，When 人工审查，Then 含 5 类设备超时参考基准表，且每类 >= 1 个参考值

---

### FR-7：诊断快照采集规范

**编号**：R-F-007
**标题**：`diag_snapshot_ref` 与诊断快照采集机制
**描述**：

R1 返回结构中包含 `diag_snapshot_ref` 字段（可为 null），当操作失败（`status = "error"`）或超时时，骨架代码须填写该字段。

快照采集规则：
1. 快照内容：由各设备骨架代码自定义，但须至少包含：
   - 操作名（`op_id`）
   - 执行时间戳（ISO 8601）
   - 设备标识（`device_id`）
   - 错误类型（`error_type`）
   - 原始错误消息（字符串）
2. 快照存储方式：初始版本写入本地文件（路径：`.output/diag_snapshots/<timestamp>_<op_id>.json`），供工程师手动分析。
3. 骨架代码中须提供 `capture_diag_snapshot(op_id, device_id, error_type, raw_msg)` 占位函数，函数体为 TODO 注释，返回快照文件路径字符串。
4. 工程师手册须说明：何时触发快照、快照文件格式、如何读取快照定位问题。

**依据**：Q-12（诊断快照：操作失败时自动采集诊断快照并在 R1 中返回引用）
**验收准则**：
- Given 骨架代码（D-04~D-07），When 人工审查，Then 每个骨架文件均包含 `capture_diag_snapshot` 占位函数且签名符合规范
- Given 工程师手册，When 人工审查，Then "诊断快照使用指南"小节存在，含触发条件、文件格式、读取方法三项

---

### FR-8：版本演进与废弃规则

**编号**：R-F-008
**标题**：原子操作版本演进、废弃声明与 CHANGELOG 规范
**描述**：

1. 版本字段：每个原子操作 schema 须包含 `since_version`（字符串，首次引入的语义版本号，如 `"1.0.0"`）。
2. 废弃规则：
   - 废弃操作须将 `deprecated: true` 设置，并填写 `deprecated_by`（替代操作的 op_id）。
   - 废弃操作不得立即删除，须保留至少 **1 个主版本**后方可移除，移除时在 CHANGELOG.md 中记录。
   - schema 校验器须对 `deprecated: true` 的操作发出 WARNING（非 ERROR），提示使用替代操作。
3. CHANGELOG.md 格式：
   - 每次变更追加一个版本段（语义版本 + 日期）
   - 变更条目分类：[Added] / [Changed] / [Deprecated] / [Removed] / [Fixed]
   - 初始版本条目（D-10）须包含所有初始操作的 [Added] 记录
4. 工程师手册须说明版本演进工作流：新增操作 → PR Review → 合并 → 更新 CHANGELOG → 发布新版本。

**依据**：Q-08（版本演进：schema 须有版本字段，废弃操作须有替代声明）
**验收准则**：
- Given 所有原子操作 schema，When 运行版本字段校验，Then `since_version` 字段存在且符合语义版本格式（`\d+\.\d+\.\d+`）的比例 = 100%
- Given CHANGELOG.md 初始版本（D-10），When 人工审查，Then 含至少 1 个版本段、格式含 [Added] 条目
- Given schema 校验器，When 处理 `deprecated: true` 的操作，Then 输出 WARNING 而非 ERROR

---

### FR-9：防火墙（fw）原子操作骨架代码

**编号**：R-F-009
**标题**：防火墙设备骨架代码（REST + CLI 双通道）
**描述**：

防火墙骨架代码（D-04）须提供 3–5 个典型操作的占位桩函数，覆盖 REST API 和 CLI 两种调用通道，包含：

1. `fw_create_acl_rule(rule_spec: dict, device_id: str) -> dict`：通过 REST API 创建 ACL 规则，返回 R1 结构。
2. `fw_delete_acl_rule(rule_id: str, device_id: str) -> dict`：通过 REST API 删除 ACL 规则，返回 R1 结构。
3. `fw_apply_policy(policy_name: str, device_id: str) -> dict`：通过 CLI 下发策略，返回 R1 结构。
4. `fw_verify_hit_count(rule_id: str, expected_min: int, device_id: str) -> dict`：通过 REST API 验证命中计数，返回 R1 结构。
5. `fw_rollback_last_config(device_id: str) -> dict`：通过 CLI 回滚最后一次配置，返回 R1 结构。

每个函数须包含：
- 函数签名和 docstring（含参数说明和返回值说明）
- `# TODO: implement REST call to <endpoint>` 或 `# TODO: implement CLI command: <cmd>` 注释
- 超时处理占位注释（FR-6）
- `capture_diag_snapshot` 调用占位（FR-7）
- 返回合规 R1 结构的占位代码

Q-01 决策确认：防火墙通过 REST API 和 CLI 两种方式交互，骨架代码须在函数注释中标注当前函数使用哪种通道。

**依据**：Q-01（防火墙接口：REST API + CLI 双通道）、Q-05（R1 统一返回结构）
**验收准则**：
- Given 防火墙骨架代码文件，When 运行 `python -m py_compile <file>`（dry-run 语法检查），Then 无语法错误，退出码 = 0
- Given 骨架代码，When 人工审查，Then 至少 3 个函数含完整 R1 占位返回结构，REST 和 CLI 通道各至少 1 个

---

### FR-10：流量发生器（tg）原子操作骨架代码

**编号**：R-F-010
**标题**：流量发生器设备骨架代码（专有 API 通道）
**描述**：

流量发生器骨架代码（D-05）须提供 3–5 个典型操作的占位桩函数，包含：

1. `tg_start_traffic(profile_id: str, rate_mbps: float, device_id: str) -> dict`：启动流量发送，返回 R1 结构。
2. `tg_stop_traffic(stream_id: str, device_id: str) -> dict`：停止流量发送，返回 R1 结构。
3. `tg_verify_packet_loss(stream_id: str, max_loss_pct: float, device_id: str) -> dict`：验证丢包率，返回 R1 结构。
4. `tg_capture_traffic_stats(stream_id: str, device_id: str) -> dict`：采集流量统计数据，返回 R1 结构（含 data 字段存储统计信息）。

接口通道（Q-01 补充）：流量发生器通过专有 API（如 IxNetwork REST API 或 HLTAPI）交互，骨架函数注释中须标注 `# Channel: TG proprietary API`。

非幂等声明：`tg_start_traffic` 必须声明 `idempotent: false`（FR-4）。

**依据**：Q-01（TG 接口：专有 API 通道）、Q-05、Q-09
**验收准则**：
- Given 流量发生器骨架代码文件，When 运行语法检查，Then 无语法错误，退出码 = 0
- Given `tg_start_traffic` 对应的 schema YAML，When 人工审查，Then `idempotent: false` 显式声明存在

---

### FR-11：Linux Mock（mock）原子操作骨架代码

**编号**：R-F-011
**标题**：Linux Mock 设备骨架代码（SSH CLI 通道）
**描述**：

Linux Mock 骨架代码（D-06）须提供 2–3 个典型操作的占位桩函数，包含：

1. `mock_inject_route(route_spec: dict, device_id: str) -> dict`：通过 SSH CLI 注入路由条目，返回 R1 结构。
2. `mock_verify_route_table(expected_routes: list, device_id: str) -> dict`：通过 SSH CLI 验证路由表，返回 R1 结构。
3. `mock_reset_network_config(device_id: str) -> dict`：通过 SSH CLI 恢复网络配置到初始状态，返回 R1 结构。

接口通道（Q-01 补充）：Linux Mock 通过 SSH CLI 交互，骨架函数注释中须标注 `# Channel: SSH CLI`。

**依据**：Q-01（Linux Mock 接口：SSH CLI）、Q-05
**验收准则**：
- Given Linux Mock 骨架代码文件，When 运行语法检查，Then 无语法错误，退出码 = 0
- Given 骨架代码，When 人工审查，Then >= 2 个函数，通道标注 `# Channel: SSH CLI` 存在

---

### FR-12：交换机（sw）原子操作骨架代码

**编号**：R-F-012
**标题**：交换机设备骨架代码（NETCONF / CLI 通道）
**描述**：

交换机骨架代码（D-07）须提供 2–3 个典型操作的占位桩函数，包含：

1. `sw_apply_vlan_config(vlan_id: int, ports: list, device_id: str) -> dict`：通过 NETCONF 或 CLI 配置 VLAN，返回 R1 结构。
2. `sw_verify_port_status(port_id: str, expected_state: str, device_id: str) -> dict`：通过 NETCONF 验证端口状态，返回 R1 结构。
3. `sw_reset_port_config(port_id: str, device_id: str) -> dict`：通过 CLI 恢复端口配置，返回 R1 结构。

接口通道（Q-01 补充）：交换机支持 NETCONF 和 CLI 两种通道，骨架函数注释中须标注使用哪种通道。

**依据**：Q-01（Switch 接口：NETCONF / CLI）、Q-05
**验收准则**：
- Given 交换机骨架代码文件，When 运行语法检查，Then 无语法错误，退出码 = 0
- Given 骨架代码，When 人工审查，Then >= 2 个函数，通道标注存在

---

### FR-13：测试用例层 YAML 模板

**编号**：R-F-013
**标题**：包含 `tight_sequence` 的完整测试用例 YAML 模板
**描述**：

D-08 须提供一个可直接复制使用的完整测试用例 YAML 模板，覆盖以下字段：

```yaml
# 测试用例 YAML 模板 v1.0
test_case_id: TC-FWATOM-XXX
title: <测试用例标题，≤ 80 字符>
description: <多行描述>
tags: []
preconditions:
  - <前置条件描述>
steps:
  - step: 1
    op_id: <操作ID>
    device: <设备标识>
    params: {}
    expected:
      status: success
tight_sequence:
  enabled: true
  steps:
    - step: 1
      op_id: fw_apply_policy
      device: fw-01
      wait_ms: 0
      on_error: abort
    - step: 2
      op_id: tg_start_traffic
      device: tg-01
      wait_ms: 500
      on_error: abort
    - step: 3
      op_id: fw_verify_hit_count
      device: fw-01
      wait_ms: 2000
      on_error: abort
postconditions:
  - <后置断言描述>
```

模板文件须包含完整注释说明每个字段的含义和可选枚举值。

**依据**：FR-5（tight_sequence）、Q-03、Q-10
**验收准则**：
- Given D-08 YAML 模板文件，When 使用 YAML 解析器（如 `python -c "import yaml; yaml.safe_load(open(...))"`) 解析，Then 无解析错误
- Given 模板文件，When 人工审查，Then 顶层字段包含 `test_case_id`、`tight_sequence`、`steps`、`preconditions`、`postconditions`，且每个字段含注释说明

---

### FR-14：工程师手册

**编号**：R-F-014
**标题**：原子操作工程师手册（D-09）
**描述**：

工程师手册须覆盖以下 6 个章节，每章节含实操指导和示例：

| 章节 | 标题 | 核心内容 |
|------|------|---------|
| §1 | 如何新增原子操作 | 填写 schema YAML 的完整步骤、命名规范检查清单、PR Review 流程 |
| §2 | 命名规范快速参考 | 设备前缀表、动词词汇表、合规/不合规示例对照 |
| §3 | 幂等性判定指南 | 判定树（只读/upsert/累积副作用）、典型示例 >= 3 个 |
| §4 | 超时参考基准表 | 5 类设备的 `timeout_ms` 推荐值及调整建议 |
| §5 | 诊断快照使用指南 | 触发条件、快照文件路径规范、读取和分析方法 |
| §6 | 版本演进工作流 | 新增/修改/废弃操作的完整流程、CHANGELOG 更新规范 |

手册须以 Markdown 格式交付，章节标题使用 `##` 级别，每章节包含至少 1 个具体示例。

**依据**：FR-2（命名规范）、FR-4（幂等性）、FR-6（超时）、FR-7（诊断快照）、FR-8（版本演进）
**验收准则**：
- Given D-09 工程师手册，When 人工审查，Then 6 个指定章节全部存在，每章节 >= 200 字
- Given 手册，When 人工审查，Then 每章节含 >= 1 个具体示例（代码块或表格形式）

---

## §5 非功能需求

| ID | 需求描述 | 优先级 | 验收条件 | 来源 |
|----|---------|--------|---------|------|
| R-NF-001 | schema 定义文档须以 YAML 格式交付，人机双可读；字段说明、类型、枚举值须内联注释 | P0 | Given D-01，When YAML 解析器解析，Then 无错误；When 人工阅读，Then 每字段含注释 | UC-FWATOM-01 前置条件 |
| R-NF-002 | 骨架代码须使用 Python 3.11+，遵循 PEP 8，通过 `ruff` 静态检查（零错误） | P0 | Given D-04~D-07，When `uv run ruff check <file>` 执行，Then 退出码 = 0 | UC-FWATOM-02 前置条件 |
| R-NF-003 | 所有交付 Markdown 文档（D-02、D-03、D-09）须通过 `markdownlint` 检查（允许宽松规则集） | P1 | Given 文档文件，When `markdownlint` 检查，Then 零 ERROR（WARNING 可忽略） | UC-FWATOM-03 前置条件 |
| R-NF-004 | schema 校验器须作为独立 Python 脚本交付（`validate_schema.py`），支持单文件和目录批量校验，退出码：0=通过，1=错误，2=警告 | P0 | Given `validate_schema.py`，When `python validate_schema.py <path>` 执行，Then 退出码符合约定；缺失必填字段时退出码 = 1 | UC-FWATOM-01 前置条件 |
| R-NF-005 | 所有交付物须在 `delivery/` 目录下按子目录组织，结构须与交付物清单（§3）一一对应 | P1 | Given `delivery/` 目录，When 人工核查，Then D-01~D-10 均有对应文件路径，无遗漏 | UC-FWATOM-05 前置条件 |
| R-NF-006 | 工程师手册（D-09）须适合中文阅读（主体内容为中文），技术术语允许英文原文 | P2 | Given D-09，When 人工审查，Then 正文中文比例 >= 80%，技术术语使用中英对照 | UC-FWATOM-03 前置条件 |

---

## §6 范围边界

### 6.1 IN-SCOPE（明确包含）

| 编号 | 范围项 | 说明 |
|------|--------|------|
| IS-01 | 原子操作元数据 schema 定义（YAML + 字段文档） | 完整字段集、必填性、枚举值、注释 |
| IS-02 | 命名规范（`设备_动词_目标` 格式）+ 词汇表 | 含设备前缀枚举、动词词汇表初始集合 |
| IS-03 | 错误码 `error_type` 枚举集 + 规范文档 | 9 个初始枚举值，含扩展流程 |
| IS-04 | 4 类设备（fw / tg / mock / sw）骨架代码 | Python 占位桩函数，含 R1 占位返回、TODO 注释 |
| IS-05 | 测试用例层 `tight_sequence` YAML 模板 | 含完整字段注释的可复制模板 |
| IS-06 | schema 校验器脚本（`validate_schema.py`） | 支持单文件和批量校验，含版本和幂等性检查 |
| IS-07 | 工程师手册（6 章节） | 覆盖新增、命名、幂等、超时、快照、版本演进 |
| IS-08 | CHANGELOG.md 初始版本 | 含初始操作集的 [Added] 条目 |

### 6.2 OUT-OF-SCOPE（明确排除）

| 编号 | 排除项 | 排除原因 |
|------|--------|---------|
| OOS-01 | 完整可执行的端到端 CLI 实现（非骨架） | 属于阶段二实现工作，超出本规范范围 |
| OOS-02 | 测试执行调度系统 / 编排引擎 | 属于阶段三 LLM 调度系统，超出本规范范围 |
| OOS-03 | 缺陷管理系统集成 | 超出规范范围 |
| OOS-04 | 多厂商适配层（除 TGFW 单厂商外） | Q-13 决策：仅覆盖单厂商防火墙 |
| OOS-05 | 知识库（KB）入库与检索 | 超出规范范围 |
| OOS-06 | 自动化测试框架集成（如 pytest 插件） | 超出规范范围 |
| OOS-07 | 生产环境部署脚本 | 超出规范范围 |
| OOS-08 | 图形化操作界面 | 超出规范范围 |

---

## §7 假设与依赖

### 7.1 假设

| ID | 假设内容 | 关联需求 | 风险等级 |
|----|---------|---------|---------|
| A-01 | 测试工程师具备 Python 中级水平，能读懂骨架代码的 TODO 注释 | R-F-009~R-F-012 | 低 |
| A-02 | 防火墙（TGFW）具备 REST API 和 CLI 两种接口，骨架代码中的 REST endpoint 路径由工程师在实现阶段填写 | R-F-009 | 中 |
| A-03 | 流量发生器使用专有 API（如 IxNetwork），具体 SDK 版本由工程师在实现阶段确认 | R-F-010 | 中 |
| A-04 | Linux Mock 设备可通过 SSH 访问，SSH 凭证由工程师在实现阶段注入 | R-F-011 | 低 |
| A-05 | 交换机支持 NETCONF 或 CLI 至少其中一种，具体通道由工程师在实现阶段确认 | R-F-012 | 中 |
| A-06 | YAML 格式作为 schema 和测试用例的统一描述格式，不支持 JSON 或其他格式的同等替代（Q-04 决策） | R-F-001, R-F-013 | 低 |
| A-07 | 本规范的 schema 字段集为初始版本（v1.0），后续字段扩展通过 PR Review 流程追加 | R-F-001, R-F-008 | 低 |

### 7.2 外部依赖

| ID | 依赖项 | 依赖类型 | 影响需求 |
|----|--------|---------|---------|
| DEP-01 | Python 3.11+ 运行环境 | 技术依赖 | R-NF-002, R-F-009~R-F-012 |
| DEP-02 | uv 包管理工具 | 工具依赖 | R-NF-002 |
| DEP-03 | PyYAML 或 ruamel.yaml（YAML 解析库） | 库依赖 | R-NF-004（校验器脚本） |
| DEP-04 | jsonschema 或 pydantic（schema 校验库） | 库依赖 | R-NF-004 |
| DEP-05 | ruff（Python 静态检查工具） | 工具依赖 | R-NF-002 |
| DEP-06 | TGFW 厂商 REST API 文档（用于骨架代码注释填写） | 外部文档依赖 | R-F-009 |

---

## §8 验收准则汇总

以下为所有需求验收准则的快速索引，正文各 FR/NFR 章节含完整 Given/When/Then 格式描述。

| 需求 ID | 标题 | 关键可度量指标 |
|---------|------|--------------|
| R-F-001 | schema 字段集 | 必填字段缺失报错率 = 100%；字段与文档一致 |
| R-F-002 | 命名规范 | 正则符合率 >= 95%；手册动词词汇表 >= 10 条 |
| R-F-003 | 错误码规范 | 非法 error_type 比例 = 0%；枚举表 >= 9 条目 |
| R-F-004 | 幂等性声明 | 幂等字段显式填写率 = 100% |
| R-F-005 | tight_sequence 模板 | 模板 steps >= 3；on_error 枚举完整 |
| R-F-006 | 超时字段 | timeout_ms 存在率 = 100%；手册含 5 类参考值 |
| R-F-007 | 诊断快照 | 每个骨架文件含 capture_diag_snapshot 函数 |
| R-F-008 | 版本演进 | since_version 存在率 = 100%；deprecated 触发 WARNING |
| R-F-009 | fw 骨架代码 | 语法检查零错误；REST+CLI 通道各 >= 1 个函数 |
| R-F-010 | tg 骨架代码 | 语法检查零错误；tg_start_traffic idempotent=false |
| R-F-011 | mock 骨架代码 | 语法检查零错误；SSH 通道标注存在 |
| R-F-012 | sw 骨架代码 | 语法检查零错误；通道标注存在 |
| R-F-013 | YAML 模板 | YAML 解析无错误；5 个顶层字段含注释 |
| R-F-014 | 工程师手册 | 6 章节全部存在；每章 >= 200 字 + 1 个示例 |
| R-NF-001 | YAML 可读性 | YAML 解析无错误；每字段含注释 |
| R-NF-002 | Python 代码规范 | ruff 检查退出码 = 0 |
| R-NF-003 | Markdown 规范 | markdownlint 零 ERROR |
| R-NF-004 | 校验器脚本 | 退出码约定符合；必填字段缺失时退出码 = 1 |
| R-NF-005 | 交付物目录结构 | D-01~D-10 均有对应文件路径 |
| R-NF-006 | 手册中文化 | 正文中文比例 >= 80% |

### 8.1 整体验收门控条件

在标记本需求文档对应的实现工作为"可进入设计阶段"之前，需同时满足：

- [ ] D-01~D-10 全部交付，无遗漏
- [ ] schema 校验器（`validate_schema.py`）通过自测：正确识别必填字段缺失、错误类型、废弃警告
- [ ] 4 个骨架代码文件通过 `python -m py_compile` 和 `ruff check`
- [ ] 工程师手册 6 章节完整，每章含示例
- [ ] CHANGELOG.md 初始版本含 [Added] 条目

---

## §9 风险与假设汇总

| ID | 类型 | 内容 | 关联需求 | 缓解措施 |
|----|------|------|---------|---------|
| RA-001 | RISK | 防火墙 REST API endpoint 路径在骨架代码中为 TODO，若厂商文档不可用则骨架注释无法填写具体路径 | R-F-009 | 骨架代码使用占位符 `<fw_api_base_url>`，手册说明如何获取厂商文档；不阻塞骨架代码交付 |
| RA-002 | RISK | 流量发生器专有 API 版本多样，骨架代码可能无法覆盖所有版本 | R-F-010 | 骨架函数注释中标注"适用 IxNetwork REST API"，并预留 `# TODO: adapt for other TG vendors` |
| RA-003 | RISK | schema 字段集 v1.0 后续可能因实际使用反馈需要扩展 | R-F-001, R-F-008 | 版本演进规则（FR-8）已定义扩展流程；schema 校验器设计为可扩展 |
| RA-004 | RISK | 工程师对幂等性判定存在分歧，可能导致 schema 声明不一致 | R-F-004 | 工程师手册提供判定树和 >= 3 个典型示例；PR Review 中由测试架构师（P-04）复核 |
| RA-005 | ASSUMPTION | 假设 Python 3.11+ 在测试环境中可用 | R-NF-002 | 在 REQUIREMENTS.md 中明确记录；若环境不支持需在设计阶段调整 |
| RA-006 | ASSUMPTION | 假设测试工程师能在 1 个工作日内完成新原子操作的 schema 填写（不含 PR Review） | R-F-014 | 工程师手册提供填写清单；schema 校验器提供即时反馈 |
| RA-007 | RISK | CHANGELOG.md 若无自动化工具辅助，人工维护可能遗漏条目 | R-F-008 | D-10 提供模板格式；工程师手册明确 CHANGELOG 更新为 PR 合并前置步骤 |

---

## §10 引用文档

| 引用 ID | 文档名称 | 来源 | 说明 |
|---------|---------|------|------|
| REF-01 | CLARIFICATION-LOG.md | `process/CLARIFICATION-LOG.md` | 第 14–18 轮澄清问答记录，所有 Q-xx 决策来源 |
| REF-02 | USE-CASES.md | `process/USE-CASES.md` | 已确认的 5 个使用场景（UC-FWATOM-01~05） |
| REF-03 | 三阶段演进模型（Q-02 决策） | CLARIFICATION-LOG.md §第15轮 | 规范服务的演进路径：用例设计→CLI 实现→LLM 调度 |
| REF-04 | R1 统一返回结构（Q-05 决策） | CLARIFICATION-LOG.md §第15轮 | 所有骨架函数返回 R1: {status, data, error_type, diag_snapshot_ref} |
| REF-05 | 命名规范（Q-06 决策） | CLARIFICATION-LOG.md §第15轮 | `设备_动词_目标` 三段式格式 |
| REF-06 | 接口通道决策（Q-01 决策） | CLARIFICATION-LOG.md §第14轮 | fw: REST+CLI; tg: 专有API; mock: SSH CLI; sw: NETCONF/CLI |
| REF-07 | USE-CASES-FIREWALL-ATOMIC.md | `process/USE-CASES-FIREWALL-ATOMIC.md` | 12 个已补全场景（UC-01～UC-12），覆盖三阶段演进和演进维护 |

---

## §10.1 功能需求 ↔ 使用场景映射

> 本章节记录每条功能需求对应的 UC 场景，供下游 HLD/LLD 追溯。
> UC 编号参照 `process/USE-CASES-FIREWALL-ATOMIC.md`。

| 需求 ID | 需求标题摘要 | 主要来源 UC | 说明 |
|---------|------------|------------|------|
| R-F-001 | schema 完整字段集 | UC-02, UC-03 | UC-02 场景要求工程师按字段集约束填写草稿；UC-03 场景要求 validator 按字段集自检 |
| R-F-002 | 命名规范 `{设备}_{动词}_{目标}` | UC-01, UC-02, UC-11 | UC-01 浏览时依赖命名规范过滤；UC-02 新建时按规范生成 operation_id；UC-11 新设备前缀也须遵守 |
| R-F-003 | 幂等性声明（idempotent / side_effects） | UC-02, UC-05 | UC-02 草稿阶段默认 `true`；UC-05 专门处理非幂等标记与 side_effects 填写 |
| R-F-004 | 失败行为声明（on_failure 枚举） | UC-03, UC-07, UC-08 | UC-03 自检验 on_failure 枚举合法；UC-07 编排层读取 on_failure 决策；UC-08 按 on_failure 执行 fail_fast / return_error / retry |
| R-F-005 | 返回值统一顶层结构（status/result/metadata） | UC-04, UC-07, UC-12 | UC-04 CLI 实现必须构造 R1 结构；UC-07 LLM 读取 status 决策；UC-12 从 status 判断虚假成功 |
| R-F-006 | diagnostic_snapshot 辅助函数与字段 | UC-06, UC-08, UC-12 | UC-06 定义采集函数；UC-08 提取 snapshot 写入诊断日志；UC-12 用 snapshot 对比实际与预期状态 |
| R-F-007 | 测试用例层 tight_sequence 声明 | UC-07, UC-09 | UC-07 编排时识别 tight_sequence 组；UC-09 专门处理跨设备 tight_sequence 时序保障 |
| R-F-008 | 版本演进字段（schema_version / deprecated / since / reason / replaced_by） | UC-10 | UC-10 完整演示废弃流程，所有版本字段均在该场景中被消费 |
| R-F-009 | 4 类设备骨架代码 | UC-04, UC-06, UC-11 | UC-04 从骨架复制起点；UC-06 按骨架实现 snapshot 函数；UC-11 新设备从最近似骨架派生 |
| R-F-010 | 工程师手册（命名/错误码/幂等/诊断）| UC-01, UC-02, UC-03, UC-05, UC-12 | 手册覆盖工程师在各设计与排查场景中的操作指南 |
| R-C-001 | 操作内部禁止自动重试 | UC-08 | UC-08 重试由编排层（LLM Agent）决策，操作本身只返回结果 |
| R-C-002 | 废弃操作保留至少一个大版本周期 | UC-10 | UC-10 步骤五明确先检查调用方再删除 |
| R-NF-001 | schema validator CI 门禁（SM-04：100% 合规） | UC-03 | UC-03 验收条件为 schema validator 100% 通过 |
| R-NF-002 | LLM Agent 自主编排成功率 ≥ 90%（SM-02） | UC-07, UC-08, UC-09 | 三个 LLM 场景共同保障编排可靠性 |

---

## §11 需求确认人工检查点

> 本章节为 meta-po 人工检查点，需求确认后方可进入方案设计阶段。

**检查点类型**：需求确认
**触发条件**：本文档（REQUIREMENTS-FIREWALL-ATOMIC.md）完成起草，待用户审核
**下一阶段**：solution-design（meta-se 接手，输出 HLD.md）

### 确认清单

**A. 8 维决策摘要（来自澄清轮次 14–18）**

| 决策 ID | 决策内容 | 确认？ |
|---------|---------|--------|
| Q-01 | 4 类设备接口：fw=REST+CLI, tg=专有API, mock=SSH CLI, sw=NETCONF/CLI | [ ] |
| Q-02 | 三阶段演进模型（用例设计→CLI实现→LLM调度），规范覆盖阶段一+阶段二桩代码 | [ ] |
| Q-03 | 保留 `tight_sequence` 作为测试用例层跨设备时序描述符 | [ ] |
| Q-04 | schema 格式：YAML，不使用 JSON | [ ] |
| Q-05 | R1 统一返回结构：`{status, data, error_type, diag_snapshot_ref}` | [ ] |
| Q-06 | 命名规范：`设备_动词_目标` 三段式格式 | [ ] |
| Q-07 | 统一错误码：`error_type` 枚举集（9 个初始值） | [ ] |
| Q-08 | 版本演进：`since_version` 字段 + 废弃保留 1 个主版本 | [ ] |

**B. 14 条功能需求确认**

| FR | 标题 | 确认？ |
|----|------|--------|
| R-F-001 | schema 字段集定义 | [ ] |
| R-F-002 | 命名规范 `设备_动词_目标` | [ ] |
| R-F-003 | 错误码规范 `error_type` 枚举 | [ ] |
| R-F-004 | 幂等性显式声明规则 | [ ] |
| R-F-005 | `tight_sequence` 跨设备时序模板 | [ ] |
| R-F-006 | 超时字段与处理策略 | [ ] |
| R-F-007 | 诊断快照采集规范 | [ ] |
| R-F-008 | 版本演进与废弃规则 | [ ] |
| R-F-009 | fw 骨架代码（REST+CLI） | [ ] |
| R-F-010 | tg 骨架代码（专有API） | [ ] |
| R-F-011 | mock 骨架代码（SSH CLI） | [ ] |
| R-F-012 | sw 骨架代码（NETCONF/CLI） | [ ] |
| R-F-013 | 测试用例层 YAML 模板 | [ ] |
| R-F-014 | 工程师手册（6 章节） | [ ] |

**C. 范围边界确认**

| 项目 | 内容 | 确认？ |
|------|------|--------|
| IN-SCOPE | schema + 命名 + 错误码 + 4 类骨架 + YAML 模板 + 校验器 + 手册 + CHANGELOG | [ ] |
| OUT-OF-SCOPE | 端到端 CLI 实现、调度系统、缺陷管理、多厂商适配、KB入库、pytest 插件、生产部署 | [ ] |

**D. 请选择：**

1. 确认通过 — 需求完整无歧义，`ready_for_design: true`，meta-po 推进 solution-design
2. 确认不通过 — 请指出问题所在，返回澄清循环
3. 需要补充 — 请输入需要补充或修改的内容，meta-pm 补充后再次确认

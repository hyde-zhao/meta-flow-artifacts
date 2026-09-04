---
story_id: "CR037-P1"
story_slug: "physical-pool-template"
cr_id: "CR-037"
title: "物理池模板化（config/physical_pool.yaml 改 schema+占位+指引）+ 测试造数隔离（tests/fixtures，不进目标项目）"
priority: "P0"
wave: 0
status: "lld-ready-for-review"
depends_on: []
dependency_type: "none"
assignee: null
dev_context: "ptm-team 仓库 skills/topo-config/config/physical_pool.yaml 当前为示例数据（XY 真机 7 设备），需要改为模板：schema + ${PLACEHOLDER} 占位 + 填写指引。同时创建测试造数 fixture（skills/topo-config/tests/fixtures/physical_pool.sample.yaml）隔离在测试目录，不注入目标项目。"
validation_context: "模板化后原 parser.py / topo_mapper.py 对 physical_pool.yaml 的加载逻辑不应破坏。测试 fixture 独立于模板，真实目标项目数据由 S8 install 生成不含造数的模板副本。"
acceptance_criteria:
  - "physical_pool.yaml 改为模板：schema 保留，示例值替换为 ${PLACEHOLDER} 占位符；device_type 全部 ${PLACEHOLDER} 化（如 DAS-TGFW-<MODEL> 占位粒度），模板正文不含任何具体安装型号值（Minor-3/Minor-4：case-sensitive 精确断言，A1600-HU/A1300-HU/HG3250 等特征串均不得出现于模板正文），与 S8/S10 断言一致（P-1-A）"
  - "模板包含 YAML 注释填写指引（每字段说明用途与填法）"
  - "示例 YAML 结构保留（nodes/FW1/FW2/SW1/SW2/PC1/PC2/TG1 + links）但所有 IP/密码/管理地址/device_type 改为占位符；模板示例用造数型号（如 DAS-TGFW-A1300-HU 仅作示例并统一标注「示例/占位，勿直接使用」，Minor-3）且示例块不进入模板正文/模板副本，P-1-A"
  - "池节点模板增加 hardware_platform 字段（${HARDWARE_PLATFORM} 纯占位，Minor-3 模板正文不含具体等价类值），供 S3 台账三角映射与 S4 等价类匹配消费（P-1-B，评审 D 组契约）"
  - "tests/fixtures/physical_pool.sample.yaml 造数包含 hg3250 假设设备（hg3250-51 ↔ SW1、hg3250-52 ↔ SW2）+ GE 口"
  - "造数 fixture 含 TE 端口（假设存在），供后续 S4 per-link TE 约束测试"
  - "模板与 fixture 为独立文件，安装流程不把 fixture 注入目标项目"
  - "physical_ledger.yaml 模板保持现有 list + port_status: {port: {allocated_to, allocated_vlan}} schema（P0-3），physical_pool.py 的 _build_index/get_free_ports/allocate 无需修改仍兼容"
  - "台账 model 字段使用完整 device_type（如 DAS-TGFW-<MODEL> 完整型号），与池 device_type 同值直通（P2-5）；台账模板 model 同样占位符化，不含具体 A1600-HU 型号（P-1-A）"
  - "parser.py/topo_mapper.py 加载 physical_pool.yaml 的逻辑不破坏"
output_files:
  - "skills/topo-config/config/physical_pool.yaml"
  - "skills/topo-config/config/physical_ledger.yaml"
  - "skills/topo-config/tests/fixtures/physical_pool.sample.yaml"
file_ownership:
  primary:
    - "skills/topo-config/config/physical_pool.yaml"
    - "skills/topo-config/config/physical_ledger.yaml"
    - "skills/topo-config/tests/fixtures/physical_pool.sample.yaml"
  shared: []
  merge_owner: null
  forbidden: []
feature_design_refs:
  - "F-CR037"
lld_policy:
  required_level: "technical-note"
  trigger_reasons:
    - "模板化改造，模式既有（对齐 devices.yaml.example）"
  rationale: "HLD §4 已承载边界，Story 内技术说明足够"
dev_gate: "cp5-approved"
---

# STORY-CR037-P1：物理池模板化 + 造数隔离

## 目标

将 `skills/topo-config/config/physical_pool.yaml` 从真实示例数据改造为模板（schema + `${PLACEHOLDER}` 占位 + 填写指引），对齐 `devices.yaml.example` 模式。同步创建隔离于目标项目的测试造数 fixture。

## 依赖

无。

## 输出文件

| 文件 | 说明 |
|------|------|
| `skills/topo-config/config/physical_pool.yaml` | 物理池模板（schema + 占位符 + 填写指引） |
| `skills/topo-config/config/physical_ledger.yaml` | 设备台账模板（**修订/规范化**：保持现有 list + port_status schema，与 physical_pool.py 向后兼容） |
| `skills/topo-config/tests/fixtures/physical_pool.sample.yaml` | 测试造数 fixture（含 hg3250 假设设备 + TE 端口） |

## AI 任务清单

- [ ] TASK-P1-01：改造 `physical_pool.yaml` 为模板 schema + `${PLACEHOLDER}`
- [ ] TASK-P1-02：保留节点/链路结构，所有 IP/密码/管理地址/speed_class/media_type 改为可填入的占位符或注释指引
- [ ] TASK-P1-03：为每个字段添加 YAML 注释说明用途与填写指引
- [ ] TASK-P1-04：修订 `physical_ledger.yaml` 模板（保持现有 **list + `port_status: {port: {allocated_to, allocated_vlan}}`** schema，与 `physical_pool.py` 向后兼容，P0-3）
- [ ] TASK-P1-05：创建 `tests/fixtures/physical_pool.sample.yaml` 造数 fixture
- [ ] TASK-P1-06：施工 fixture 含 hg3250-51/52 假设设备 + GE 端口 + TE 端口
- [ ] TASK-P1-07：验证 parser.py/topo_mapper.py 加载模板不破坏（parser 通过 filepath 加载时 schema 兼容）

## 技术说明

### 设计依据

CR-037 §三层文件结构明确：层1 模板/schema 层存放于 ptm-team 仓库，层3 真实数据层由目标项目用户维护。物理池从"示例数据"变为"模板"是层1标准交付物（对齐 `devices.yaml.example` 模式）。

### 文件影响

| 文件 | 操作 | 说明 |
|------|------|------|
| `skills/topo-config/config/physical_pool.yaml` | **修改** | 现有示例数据 → 模板（schema + `${PLACEHOLDER}` 占位 + YAML 注释指引），保留节点/链路结构 |
| `skills/topo-config/config/physical_ledger.yaml` | **修改（规范化）** | 设备台账模板，**保持现有 list + `port_status: {port: {allocated_to, allocated_vlan}}` schema**（P0-3），补齐占位符与填写指引；不改为 dict+ports 结构 |
| `skills/topo-config/tests/fixtures/physical_pool.sample.yaml` | **新建** | 测试造数 fixture（hg3250-51/52 假设设备 + GE/TE 端口），隔离于安装路径 |
| `skills/topo-config/src/parser.py` | **不修改** | 通过 `filepath` 参数加载 pool 文件，模板占位符为合法 YAML 字符串值，不破坏解析 |
| `skills/topo-config/src/topo_mapper.py` | **不修改** | 加载逻辑不因模板化改变 |
| `skills/topo-config/src/physical_pool.py` | **不修改** | `_build_index`/`get_free_ports`/`allocate` 依赖现有 list + `port_status` schema；本 Story 保持 schema 不变，故无需修改（P0-3） |

### 模板 Schema 设计

`physical_pool.yaml` 模板保留现有结构：

```yaml
metadata:
  schema_version: "1.0"
  pool_id: ${POOL_ID}         # 资源池唯一标识，如 lab-xy
  description: "${DESCRIPTION}" # 资源池描述
  updated_at: "${UPDATED_AT}"  # ISO 8601

nodes:
  # 每个节点（DUT/SW/PC/TG）：
  #   node_type: DUT|SW|PC|TG
  #   device_type: ${DEVICE_TYPE} —— 全部占位符化（P-1-A/Minor-3）：只允许 ${PLACEHOLDER} 占位粒度
  #               （完整 device_type 由用户填写，如 DAS-TGFW-<MODEL> 占位粒度），模板正文不得写入任何
  #               具体安装型号特征串。见 device-reference.md 硬件系列映射
  #   hardware_platform: ${HARDWARE_PLATFORM} —— P-1-B 必填（Minor-3）：按 device-reference.md 硬件系列
  #               映射填写硬件平台等价类；模板只允许 ${HARDWARE_PLATFORM} 占位，不得枚举具体等价类值，
  #               不得写入任何具体型号特征串
  #   management: { host, port, username, password } （密码用 ${PASSWORD_PLACEHOLDER}）
  #   interfaces:
  #     port<N>:
  #       physical_port: GE0_1|GigabitEthernet... （实际接线物理端口名）
  #       speed_class: GE|TE|XTE|QTE
  #       media_type: copper|fiber
  #       trex_port: "2_3" （TG 专用，可选）
  ...
  ${NODE_NAME}:
    node_type: ${NODE_TYPE}
    device_id: ${DEVICE_ID}
    device_type: ${DEVICE_TYPE}                 # P-1-A：占位粒度，如 DAS-TGFW-<MODEL>，不含具体型号
    hardware_platform: ${HARDWARE_PLATFORM}     # P-1-B/Minor-3：硬件平台等价类占位（只允许 ${HARDWARE_PLATFORM}，
                                                # 不写入具体等价类/型号值），S3 三角映射与 S4 等价类匹配消费
    management:
      host: "${MGMT_HOST}"
      port: ${MGMT_PORT}
      username: "${MGMT_USERNAME}"
      password: "${PASSWORD_PLACEHOLDER}"
    interfaces:
      port1:
        physical_port: ${PHYSICAL_PORT_1}
        speed_class: ${SPEED_CLASS}
        media_type: ${MEDIA_TYPE}

links:
  # 每根 cable：
  #   endpoints: [{node, interface}]
  #   media_type: copper|fiber
  #   description: 人读描述
  ${LINK_NAME}:
    endpoints:
      - {node: ${NODE_A}, interface: ${INTERFACE_A}}
      - {node: ${NODE_B}, interface: ${INTERFACE_B}}
    media_type: ${MEDIA_TYPE}
    description: "${DESCRIPTION}"
```

关键改动：
- **保留示例节点**（FW1/FW2/SW1/SW2/PC1/PC2/TG1，展示典型结构）但值改为占位符或保留为示意说明
- **device_type 全部 `${PLACEHOLDER}` 化（P-1-A/Minor-3）**：模板 `device_type` 使用 `${DEVICE_TYPE}` / `DAS-TGFW-<MODEL>` 占位粒度，**模板正文不得写入任何具体安装型号特征串**；若需展示填写示例，示例块统一标注「示例/占位，勿直接使用」，且示例块不进入实际模板正文 / install 生成的模板副本（与 S8/S10「不含具体型号值」断言一致，Minor-4）
- **池节点模板新增 `hardware_platform` 字段（P-1-B/Minor-3）**：每个节点模板必须包含 `hardware_platform: ${HARDWARE_PLATFORM}`（纯占位，不得枚举具体等价类/型号值），作为 S3 台账三角映射与 S4 等价类匹配的第一类字段（评审 D 组契约），不再标为可选
- **speed_class/media_type 保留**但注释说明如实填写
- **password 统一为 `${PASSWORD_PLACEHOLDER}`**（ADR-02 凭据禁止）
- **移除 trex_port 示例值**（保留字段但用占位符）

### physical_ledger.yaml 模板

台账模板记录设备级账号/端口占用状态。**schema 保持现有 `list` + `port_status: {port: {allocated_to, allocated_vlan}}`**（P0-3，向后兼容 `physical_pool.py` 的 `_build_index` / `get_free_ports` / `allocate`，不改为 dict + `ports:`）：

```yaml
metadata:
  schema_version: "1.0"
  description: "设备台账 — 记录 device_id/brand/model 与端口占用"

devices:
  - device_id: ${DEVICE_ID}
    brand: ${BRAND}                # 厂商：tgfw | ruijie | h3c | pc | ixia
    model: ${MODEL}                # 台账型号（P-1-A/P2-5/Minor-3：完整 device_type 占位粒度，如 DAS-TGFW-<MODEL>；
                                   # 模板正文不含任何具体型号值）
    version: ${VERSION}            # 固件/软件版本
    status: free                   # free | occupied | faulty
    port_status:
      port1:
        allocated_to: null         # 占用标记（topo_mapper 写回：env_name）
        allocated_vlan: null       # 占用 VLAN（topo_mapper 写回）
      port<N>:
        allocated_to: null
        allocated_vlan: null
```

端口占用闭环（HLD §4.4 H1）：`port_status[].allocated_to` / `allocated_vlan` 字段供 S4 topo_mapper 跨用例防冲突。

**P2-5 说明（model ↔ device_type 同值直通）**：台账 `model` 字段值使用**完整 device_type 占位**（如 `DAS-TGFW-<MODEL>` 占位粒度；`DAS-TGFW-A1300-HU` 仅作「示例/占位，勿直接使用」说明，模板正文不含具体型号值），与物理池节点 `device_type` 同值直通，无需额外映射表。S3 台账三角映射直接采用：`model`（完整 device_type）→ device-reference 平台等价类；若目标项目以短型号填写，S3 需加规范前缀规范化（见 S3-LLD §5.3）。

### 测试造数 Fixture（tests/fixtures/physical_pool.sample.yaml）

特征：
- **隔离于目标项目**：存放 `skills/topo-config/tests/fixtures/`，安装不走该目录
- **hg3250 假设设备（fixture 专用造数，示例/占位勿直接使用，Minor-3）**：`hg3250-51`（`device_type: DAS-TGFW-A1300-HU` 为**造数型号（仅示例/占位）**，`hardware_platform: HG3250` 等价类）↔ SW1 GE0 口连接；`hg3250-52`（`device_type: DAS-TGFW-A1300-HU` 造数型号，`hardware_platform: HG3250`）↔ SW2 GE0 口连接
- **造数型号不进入模板（Minor-3）**：fixture 是**唯一**允许含 `DAS-TGFW-A1300-HU` / `HG3250` / `hg3250` 等造数型号的位置；模板正文（config/*.yaml）与 install 生成的模板副本**不得**出现任何具体型号特征串（S10 case-sensitive 精确断言校验，Minor-4）
- **TE 端口**：hg3250-51 port3/port4 声明 `speed_class: TE, media_type: fiber`（假设存在），供 S4 per-link TE 约束测试
- **TG 节点**：含 TG1（`device_type: IXIA-C`，2 个 GE 口 `media_type: fiber`），供映射测试
- **password 用测试占位符**（`test-placeholder` 或 `<password>`）

### 接口/数据/权限变化

- **不新增接口**：模板与 fixture 仅改变 `physical_pool.yaml` 内容，不改变 parser.py/topo_mapper.py 加载接口
- **台账 schema 保持兼容（P0-3）**：`physical_ledger.yaml` 保持现有 list + `port_status` schema，`physical_pool.py` 的 `_build_index`（list 遍历）、`get_free_ports`（`port_status[port].allocated_to`）、`allocate`（写回 `allocated_to`/`allocated_vlan`）无需修改
- **不涉及权限变化**：模板内凭据占位符化（`${PASSWORD_PLACEHOLDER}`），已有 ADR-02 约束
- **不影响现有数据**：配置加载逻辑不变（parser.py 通过 `filepath` / `topo_id` 加载不依赖 `CONFIG_DIR` 内固定名）

### 异常与回退

- **parser/topo_mapper 加载失败**：若模板占位符导致 YAML 解析失败——占位符为合法 YAML 字符串值，不应破坏解析。若出现，回退到本次修改前 commit
- **fixture 数据污染目标项目**：S8 install 生成模板副本时只复制模板（不含 tests/），自然隔离

### 测试入口

- S10 单测将验证 fixture 可被 parser.py 正常加载
- S4 per-link TE 约束测试消费 fixture 中 hg3250-51 TE 端口

### 风险与重访条件

| 风险 | 级别 | 重访条件 |
|------|------|---------|
| 占位符格式与 parser/topo_mapper 预期冲突 | 低 | 加载模板时报 ValueError 或解析异常 |
| 台账 ledger 结构不足以表达占用闭环 | 中 | S4 实现时发现需扩展字段（现有 list + `port_status[].allocated_to` 已支持 per-port 占用，若需按 env 多端口聚合再做扩展 CR） |

### 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|------|------|--------|---------|
| v1.0 | 2026-08-05 | meta-dev | 初稿（technical-note，CP5 提交） |
| v1.1 | 2026-08-05 | meta-dev（CP5 NEEDS_REWORK 修订） | P-1-A：device_type 占位粒度 `${PLACEHOLDER}` 化（如 DAS-TGFW-<MODEL>），模板示例造数型号标注为占位，与 S8/S10「不含 A1600-HU」断言一致；P-1-B：池节点模板新增 `hardware_platform` 必填字段（等价类），供 S3 三角映射与 S4 等价类匹配消费（评审 D 组契约） |
| v1.2 | 2026-08-05 | meta-dev（CP5 第二轮复审 Minor-3 修订） | 模板注释残留清理：模板 schema 代码块中 `HG3250`/`A1300-HU`/`A1600-HU` 等具体型号特征串**全部清理为纯占位**（`${DEVICE_TYPE}` / `${HARDWARE_PLATFORM}` / `${MODEL}`）；示例块统一标注「示例/占位，勿直接使用」；明确「模板正文/模板副本不得含任何具体型号值，fixture 是唯一允许造数位置」，与 S10 case-sensitive 精确断言（Minor-4）对齐 |

### 偏离记录

无。对齐 HLD §4.3/4.4 与 CR-037 §前置数据任务。

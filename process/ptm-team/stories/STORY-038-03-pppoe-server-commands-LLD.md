---
doc_type: LLD
story_id: "STORY-038-03"
story_slug: "pppoe-server-commands"
cr_id: "CR-038"
tier: "A"
status: "draft"
created_at: "2026-08-15"
owner: "meta-dev"
source_hld: "docs/design/HLD-CR-038.md §10.4 + §11 + §12.3"
source_story: "process/stories/STORY-038-03.md"
source_adr: "docs/design/ARCHITECTURE-DECISION-CR-038.md ADR-CR038-03"
design_evidence_type: "full-lld"
lld_policy_required_level: "full-lld"
scope: "skills/topo-config/src/commands.py（三厂商 PPPoE 命令模板）+ skills/topo-config/src/physical_pool.py（pppoe_server 读取/校验，共享文件首次写入）+ skills/topo-config/config/physical_pool.yaml（SW 节点 pppoe_server 示例块）"
---

# STORY-038-03-LLD：PPPoE Server 命令族 + 数据源契约

> **Tier**: A（security（cipher 密文占位校验）+ external-interface（三厂商命令模板）+ data-model（pppoe_server 块 schema）三触发叠加）
> **Shared Fragments**: `skills/topo-config/src/switch_configurator.py`（STORY-038-04 消费方，复用 `_build_vlan_commands` 渲染模式，只读参考）、`skills/topo-config/config/physical_ledger.yaml`（`brand` 查询，只读）、`skills/topo-config/src/port_spec.py`（`validate_interfaces` 问题列表模式，只读参考）
> **Open Items**: Q-038-002（H3C 真机 PPPoE 命令与参考命令族差异，non-blocking-open，dry-run diff + 真机独立授权核对）；设备 cipher 密文前缀 allowlist 待真机密文样例复核（LCQ-STORY-038-03-01，non-blocking）

---

## 0. 工程依据

本 Story 是 F-CR038-E2（PPPoE 配置）的命令模板层 + 数据源层。工程依据：

- **HLD §10.4**：`commands.py` 三厂商新增 PPPoE 命令族（H3C 参考实现；ruijie/huawei 供 dry-run diff）；`switch_configurator.py` 新增 `configure_pppoe_server` 复用 `_build_*_commands` 模式（后者归 STORY-038-04）。
- **ADR-CR038-03**：PPPoE 数据源 = physical_pool SW 节点 `pppoe_server` 块；ip pool 段 + local-user（cipher 密文 `${ENV_VAR}`）存该块，保证「同源一致」（R-F-019）。
- **Feature DESIGN §2/§3**：`pppoe_server` 块 YAML 形态；三厂商命令键 `pppoe.enable_server` / `pppoe.ip_pool` / `pppoe.local_user` / `pppoe.virtual_template` / `pppoe.bind_interface`；`password_cipher` 必须 `${ENV_*}` 占位或设备 cipher 密文，禁止明文（ADR-02）；缺 `ip_pool_name`/`ip_pool_subnet` → 结构化错误 `PPPOE_SERVER_INCOMPLETE`。
- **REQUIREMENTS**：R-F-003（命令族）、R-F-004（三厂商模板）、R-F-019（数据源同源）、R-C-001（凭据 `${ENV_VAR}` 占位禁止明文）、R-C-002（local-user 密码保持设备 cipher 密文）、R-NF-001（H3C 真机验证，ruijie/huawei 仅 dry-run diff）。
- **TEST-PLAN §3**：凭据占位校验（仅接受 `${ENV_*}` 或 cipher 密文，拒绝明文）+ 密文保留（设备 cipher 密文原样透传不反解）。

**范围边界（明确）**：本 Story 只产出「命令模板定义 + 数据源 schema + 校验函数」，**不实现**命令序列渲染（`_build_pppoe_server_commands` 归 STORY-038-04 `switch_configurator.py`）、**不实现**真机下发（独立 runtime_authorization）。H3C 命令文本为参考实现（真机验证对象），ruijie/huawei 命令文本为 dry-run 模板（未经真机验证，供人工核对 diff），**本 LLD 不宣称「三厂商真机 3/3 覆盖」**。

**文档结构导览（14 段语义要点对齐）**：§1 目标与需求，§2 文件影响范围（代码结构），§3 接口设计（API），§4 数据模型，§5 核心流程与技术细节，§6 异常处理，§7 测试设计，§8 实施步骤，§9 回滚策略，§10 安全约束，§11 需求契约映射，§12 依赖与前置，§13 风险与开放项，§14 验收与交接（DoD）。

---

## 1. 背景与目标

### 1.1 Goal

1. `commands.py` 的三厂商 `COMMANDS` 字典新增 PPPoE 命令族：`pppoe.enable_server` / `pppoe.ip_pool` / `pppoe.local_user` / `pppoe.virtual_template` / `pppoe.bind_interface` 五个键。
2. `physical_pool.py` 新增 `get_pppoe_server(device_name)` 读取 SW 节点 `pppoe_server` 块 + `validate_pppoe_server(block)` 结构校验 + `_validate_password_cipher(value)` 凭据校验；数据源唯一 = physical_pool（R-F-019 同源一致）。
3. `config/physical_pool.yaml` 的 SW 节点示例新增 `pppoe_server` 块（含 `local_users[].password_cipher` 用 `${ENV_*}` 占位）。
4. `password_cipher` 仅接受 `${ENV_*}` 占位或设备 cipher 密文，拒绝明文（ADR-02 / R-C-001 / R-C-002）。
5. 承载 R-F-003（模板定义侧）/ R-F-004（三厂商模板）/ R-F-019（数据源契约）。

### 1.2 Requirements

#### Functional

| ID | 需求 | 优先级 |
|----|------|:------:|
| FR1 | `COMMANDS[vendor]['pppoe']` 存在五个族键 `enable_server` / `ip_pool` / `local_user` / `virtual_template` / `bind_interface`，三厂商（h3c/ruijie/huawei）键集一致（无缺失） | P0 |
| FR2 | H3C 五个族键为参考实现（真机验证对象），命令文本与 H3C Comware 标准一致 | P0 |
| FR3 | ruijie/huawei 五个族键存在且文本按厂商差异，但仅 dry-run diff 供人工核对（R-NF-001），不宣称真机覆盖 | P1 |
| FR4 | `get_pppoe_server(device_name) -> dict | None` 从 physical_pool 唯一真相源读取 SW 节点 `pppoe_server` 块，无 devices.yaml/独立配置回退（R-F-019） | P0 |
| FR5 | `validate_pppoe_server(block) -> list[dict]` 对 `pppoe_server` 块做全字段校验，返回问题列表（空=合法），含 `PPPOE_SERVER_INCOMPLETE` 结构化错误码 | P0 |
| FR6 | `_validate_password_cipher(value) -> tuple[bool, str]` 判定 `password_cipher` 合法性：`${ENV_*}` 占位 / 设备 cipher 密文放行，明文拒绝（ADR-02 / R-C-001 / R-C-002） | P0 |
| FR7 | 设备 cipher 密文原样透传，校验仅做前缀判定，不反解/解密（R-C-002） | P0 |

#### Non-Functional

| ID | 需求 |
|----|------|
| NFR1 | 向后兼容：`get_commands` / `COMMANDS` 既有键不变；未含 `pppoe` 键的调用方不受影响 |
| NFR2 | 确定性：同输入 `pppoe_server` 块 → 同校验结果（无时间戳/随机性） |
| NFR3 | 安全：校验/读取路径不打印 `password_cipher` 明文到日志 |
| NFR4 | 独立可测：commands.py 模板与 physical_pool.py 校验可分别单测，不依赖 STORY-038-04 |

---

## 2. 文件影响范围

### 2.1 修改文件（本 Story 写入范围）

```
skills/topo-config/src/commands.py          # 三厂商 COMMANDS 增 pppoe 族（5 键，嵌套模板）
skills/topo-config/src/physical_pool.py     # 增 get_pppoe_server / validate_pppoe_server / _validate_password_cipher（共享文件，首次写入方）
skills/topo-config/config/physical_pool.yaml # SW 节点示例增 pppoe_server 块
```

### 2.2 只读消费（不改）

```
skills/topo-config/src/switch_configurator.py  # STORY-038-04 消费方；_build_vlan_commands 渲染模式参考
skills/topo-config/config/physical_ledger.yaml # brand 查询（get_brand 既有）
skills/topo-config/src/port_spec.py            # validate_interfaces 问题列表返回模式参考
```

### 2.3 模块拆分与职责

| 模块 | 函数/键 | 职责 |
|------|---------|------|
| commands.py | `COMMANDS[vendor]['pppoe']` | 三厂商 PPPoE 命令模板（纯数据，无逻辑） |
| commands.py | `get_commands(brand)` | 既有，透传完整 `COMMANDS[brand]`（含 `pppoe`），不改签名 |
| commands.py | `has_pppoe_commands(brand) -> bool` | 能力探测：`'pppoe' in COMMANDS[brand]` 且五键齐全 |
| physical_pool.py | `get_pppoe_server(device_name)` | 从 `_device_by_name[name].get('pppoe_server')` 读取（None=缺失） |
| physical_pool.py | `validate_pppoe_server(block)` | 结构校验 → 问题列表（空=合法） |
| physical_pool.py | `_validate_password_cipher(value)` | 凭据占位/密文合法性判定（模块私有，可测试） |

---

## 3. 接口设计（`COMMANDS[vendor]['pppoe.*']` 键契约）

### 3.1 键结构契约（五族键，嵌套子命令模板）

`COMMANDS[vendor]` 为扁平字符串模板字典；`COMMANDS[vendor]['pppoe']` 为**嵌套 dict**。五个族键中 `enable_server` 与 `bind_interface` 为单命令字符串模板，`ip_pool` / `local_user` / `virtual_template` 为多段子命令 dict（因 PPPoE 各阶段是「进入视图 → 多条配置 → 退出」的多行结构，单字符串无法表达）。

```
COMMANDS[vendor]['pppoe'] = {
    'enable_server':   str | ''            # 使能命令；无全局使能的厂商为空字符串
    'ip_pool':         {'create': str, 'gateway': str, 'network': str}
    'local_user':      {'create': str, 'password': str, 'service_type': str}
    'virtual_template': {'enter': str, 'auth': str, 'pool': str, 'ip': str}
    'bind_interface':  str                  # 物理口绑定虚拟模板
}
```

五族键名与 Story 卡片 §目标完全一致（`pppoe.enable_server` / `pppoe.ip_pool` / `pppoe.local_user` / `pppoe.virtual_template` / `pppoe.bind_interface`）。渲染由 STORY-038-04 `_build_pppoe_server_commands` 按阶段顺序组装（`enable_server` → `ip_pool` → `local_user` → `virtual_template` → `bind_interface`），本 Story 只定义模板，不组装序列。

### 3.2 三厂商命令模板精确文本

> 占位符：`{pool_name}`、`{gateway}`、`{network}`、`{mask}`、`{username}`、`{cipher}`、`{auth}`、`{ip}`、`{template_id}`。`{mask}` 由 `ip_to_mask` 从 CIDR 换算；`{cipher}` 只注入 `${ENV_*}` 占位或设备密文（§10）。

#### H3C（参考实现，真机验证对象）

| 族键 | 子键 | 模板 |
|------|------|------|
| enable_server | — | `''`（H3C Comware 无全局使能，PPPoE Server 经 `bind_interface` 绑定使能） |
| ip_pool | create | `ip pool {pool_name}` |
| ip_pool | gateway | `gateway-list {gateway}` |
| ip_pool | network | `network {network} mask {mask}` |
| local_user | create | `local-user {username} class network` |
| local_user | password | `password cipher {cipher}` |
| local_user | service_type | `service-type ppp` |
| virtual_template | enter | `interface Virtual-Template{template_id}` |
| virtual_template | auth | `ppp authentication-mode {auth}` |
| virtual_template | pool | `remote address pool {pool_name}` |
| virtual_template | ip | `ip address {ip} {mask}` |
| bind_interface | — | `pppoe-server bind Virtual-Template{template_id}` |

#### Huawei（dry-run 模板，未经真机验证，供人工核对 diff）

| 族键 | 子键 | 模板 |
|------|------|------|
| enable_server | — | `''`（VRP 经 `bind_interface` 使能） |
| ip_pool | create | `ip pool {pool_name}` |
| ip_pool | gateway | `gateway-list {gateway}` |
| ip_pool | network | `network {network} mask {mask}` |
| local_user | create | `local-user {username}` |
| local_user | password | `password cipher {cipher}` |
| local_user | service_type | `service-type ppp` |
| virtual_template | enter | `interface Virtual-Template {template_id}` |
| virtual_template | auth | `ppp authentication-mode {auth}` |
| virtual_template | pool | `remote address pool {pool_name}` |
| virtual_template | ip | `ip address {ip} {mask}` |
| bind_interface | — | `pppoe-server bind virtual-template {template_id}` |

#### Ruijie（dry-run 模板，未经真机验证，供人工核对 diff）

| 族键 | 子键 | 模板 |
|------|------|------|
| enable_server | — | `''`（RGOS 经 `bind_interface` 使能） |
| ip_pool | create | `ip pool {pool_name}` |
| ip_pool | gateway | `default-gateway {gateway}` |
| ip_pool | network | `address {network} {mask}` |
| local_user | create | `username {username}` |
| local_user | password | `password cipher {cipher}` |
| local_user | service_type | `service-type ppp` |
| virtual_template | enter | `interface virtual-template {template_id}` |
| virtual_template | auth | `ppp authentication {auth}` |
| virtual_template | pool | `ppp ipcp remote-address pool {pool_name}` |
| virtual_template | ip | `ip address {ip} {mask}` |
| bind_interface | — | `pppoe-server bind virtual-template {template_id}` |

> **差异表达（关键设计点）**：三厂商差异体现在「子命令措辞 + 视图名称格式」（H3C `Virtual-Template{id}` 无空格 / `remote address pool` / `gateway-list`；Huawei `Virtual-Template {id}` 有空格 / `virtual-template` 小写；Ruijie `username` 而非 `local-user` / `default-gateway` / `address`）。H3C 为参考实现（真机验证），ruijie/huawei 文本为最佳-effort dry-run 模板，标记为「未经真机验证」，由 dry-run diff 供人工核对（Q-038-002 / RA-038-002），**不写「3/3 覆盖」**。

### 3.3 physical_pool.py 新接口

```python
def get_pppoe_server(self, device_name: str) -> dict | None:
    """读取 SW 节点 pppoe_server 块（R-F-019 唯一真相源 = physical_pool）。
    返回节点块 dict 或 None（节点不存在或节点无 pppoe_server 块）。
    不做校验（校验由 validate_pppoe_server 独立承担）。"""

def validate_pppoe_server(self, block: dict) -> list[dict]:
    """校验 pppoe_server 块，返回问题列表（空=合法）。
    问题项 schema: {'code': str, 'field': str, 'message': str}。
    校验项见 §4.3；凭据校验复用 _validate_password_cipher（§10）。"""

def _validate_password_cipher(self, value: str) -> tuple[bool, str]:
    """凭据合法性判定：返回 (ok, kind)。
    kind ∈ {'env_placeholder', 'device_cipher', 'plaintext'}。
    规则见 §10。模块私有，供 validate_pppoe_server 与单测调用。"""
```

### 3.4 能力探测

```python
def has_pppoe_commands(brand: str) -> bool:
    """brand 支持 PPPoE 命令族（'pppoe' 键存在且五族键齐全）。"""
```

- 消费方：STORY-038-04 `SwitchConfigurator` 在渲染前用 `has_pppoe_commands(self.brand)` 判定是否可生成 PPPoE 命令；`False` → 结构化 reason（`PPPOE_UNSUPPORTED_VENDOR`）。
- `get_commands(brand)` 签名与返回不变：仍返回完整 `COMMANDS[brand]`（含 `pppoe` 嵌套 dict），兼容既有调用方。

---

## 4. 数据模型（`pppoe_server` 块 schema）

### 4.1 YAML 形态

```yaml
# physical_pool.yaml 内 SW 节点扩展（示例，占位/勿直接使用）
  SW3:
    node_type: SW
    device_id: ${DEVICE_ID_SW3}
    role: pppoe-server
    management:
      host: "${MGMT_HOST_SW3}"
      port: ${MGMT_PORT_SW3}
      username: "${MGMT_USERNAME_SW3}"
      password: "${PASSWORD_PLACEHOLDER}"
    pppoe_server:
      ip_pool_name: pool_pppoe
      ip_pool_subnet: "10.10.10.0/24"
      ip_pool_gateway: "10.10.10.1"
      virtual_template: 1
      authentication: chap
      local_users:
        - username: testuser
          password_cipher: "${ENV_PPPOE_USER_PASSWORD}"   # ${ENV_*} 占位；或设备 cipher 密文；禁止明文
```

### 4.2 字段 schema（类型/必填/校验规则）

| 字段 | 类型 | 必填 | 校验规则 | 错误码 |
|------|------|:----:|---------|--------|
| `ip_pool_name` | str | 是 | 非空；`[A-Za-z0-9_-]{1,63}` | `PPPOE_FIELD_MISSING` / `PPPOE_FIELD_INVALID` |
| `ip_pool_subnet` | str | 是 | 合法 IPv4 CIDR（`ipaddress.ip_network(..., strict=False)` 可解析） | `PPPOE_SERVER_INCOMPLETE`（缺） / `PPPOE_POOL_SUBNET_INVALID` |
| `ip_pool_gateway` | str | 是 | 合法 IPv4 地址（无掩码），`ipaddress.ip_address` 可解析 | `PPPOE_FIELD_MISSING` / `PPPOE_POOL_GATEWAY_INVALID` |
| `virtual_template` | int | 是 | 正整数，`1 <= v <= 4096` | `PPPOE_VIRTUAL_TEMPLATE_INVALID` |
| `authentication` | str | 是 | 枚举 `{chap, pap}`（小写） | `PPPOE_AUTH_INVALID` |
| `local_users` | list[dict] | 是 | 非空（`len >= 1`） | `PPPOE_LOCAL_USERS_EMPTY` |
| `local_users[].username` | str | 是 | 非空；`[A-Za-z0-9_.@-]{1,64}` | `PPPOE_LOCAL_USER_USERNAME_INVALID` |
| `local_users[].password_cipher` | str | 是 | 见 §10：`${ENV_*}` 占位或设备 cipher 密文，拒绝明文 | `PPPOE_PASSWORD_PLAINTEXT_REJECTED` / `PPPOE_PASSWORD_FORMAT_INVALID` |

### 4.3 校验顺序（确定性，`validate_pppoe_server`）

1. `block` 非 dict → 返回 `[{'code': 'PPPOE_SERVER_INVALID_TYPE', 'field': 'pppoe_server', 'message': ...}]`。
2. 缺 `ip_pool_name` 或 `ip_pool_subnet` → 返回 `[{'code': 'PPPOE_SERVER_INCOMPLETE', 'field': 'pppoe_server', 'message': ...}]`（DESIGN §2 契约），**并短路返回**（缺字段不做后续字段级校验）。
3. 逐字段校验 `ip_pool_gateway` / `virtual_template` / `authentication`（按 §4.2 规则）。
4. `local_users` 为空 → `PPPOE_LOCAL_USERS_EMPTY`；非空则逐 user 校验 `username` + `_validate_password_cipher(password_cipher)`。
5. 返回全部问题列表（累计，非首错即返）。空列表 = 合法。

### 4.4 持久化设计

- 无新增持久化。`pppoe_server` 块随 `physical_pool.yaml` 静态存在，由既有 `PhysicalPool.load` / `from_pool_data` 加载进 `pool_data`，`get_pppoe_server` 只读不写。
- `from_pool_data` 不改动（`pool_data` 结构不变），避免与 STORY-038-13 的 trex_instances 合并产生结构冲突（§12）。

---

## 5. 核心流程（三厂商命令模板渲染，复用 `_build_vlan_commands` 模式）

### 5.1 模板渲染模式对齐

本 Story 不渲染（渲染归 STORY-038-04），但必须为渲染提供与 `_build_vlan_commands` 一致的「键 + `.format()`」模式契约：

- 既有模式（switch_configurator.py:287-316）：`self._cmds['system_view']` → `self._cmds['create_vlan'].format(vlan_id=...)` → ... → `self._cmds['quit']`，逐条 `.format()` 生成 `list[str]`。
- PPPoE 模式（STORY-038-04 消费）：`self._cmds['pppoe']['ip_pool']['create'].format(pool_name=...)` 等，逐子键 `.format()` 生成 `list[str]`；每个多段族键间用 `self._cmds['quit']` 退出子视图（复用既有 `quit` 键，不新增）。

**本 Story 的契约义务**：保证 `COMMANDS[vendor]['pppoe'][*]` 每个字符串模板的占位符集合，与 STORY-038-04 渲染时传入的 kwargs 键一一对应（`create/gateway/network/enter/auth/pool/ip/create/password/service_type/bind_interface` → `pool_name/gateway/network/mask/username/cipher/auth/ip/template_id`），不新增未声明占位符。

### 5.2 数据流（读取 → 校验 → 渲染）

```
switch_configurator.from_pool(sw_name)                    [STORY-038-04]
  → pool.get_pppoe_server(sw_name)                        [本 Story：physical_pool.py]
      ├─ 返回 None → 结构化 reason（PPPOE_SERVER_MISSING），终止
      └─ 返回 block
  → issues = pool.validate_pppoe_server(block)            [本 Story：physical_pool.py]
      ├─ issues 非空 → 结构化 reason（聚合 issues），终止，不生成命令
      └─ 空 → 继续
  → has_pppoe_commands(brand)                             [本 Story：commands.py]
      ├─ False → 结构化 reason（PPPOE_UNSUPPORTED_VENDOR），终止
      └─ True → 继续
  → _build_pppoe_server_commands(block)                   [STORY-038-04：渲染 + dry-run/telnet]
      # 阶段顺序：enable_server → ip_pool → local_user → virtual_template → bind_interface
```

### 5.3 技术细节

- **`ip_pool_subnet` → `network`/`mask` 换算**：渲染方用既有 `ip_to_mask(subnet)` 拆出 `(network, mask)`（commands.py:199-223，已实现），本 Story 不重复实现，仅定义模板占位符 `{network}`/`{mask}`。
- **`ip_pool_gateway` 直接注入**：`{gateway}` 无掩码（DESIGN §2 示例 `10.10.10.1`）。
- **`authentication` 注入**：`{auth}` 直接透传 `chap`/`pap`（schema 已限定枚举）。
- **`virtual_template` 注入**：`{template_id}` 取 int 直连（H3C `Virtual-Template{template_id}` 无空格；Huawei/Ruijie 有空格，已内嵌在各自模板字符串中）。

---

## 6. 异常处理

| 场景 | 触发 | 行为 | 错误码 |
|------|------|------|--------|
| 节点无 `pppoe_server` 块 | `get_pppoe_server` 返回 None（role=pppoe-server 但缺块） | 调用方（STORY-038-04）结构化 reason，终止，不生成命令 | `PPPOE_SERVER_MISSING` |
| 块非 dict | `validate_pppoe_server` | 问题列表单条 | `PPPOE_SERVER_INVALID_TYPE` |
| 缺 `ip_pool_name`/`ip_pool_subnet` | `validate_pppoe_server` 步骤 2 | 问题列表 + 短路 | `PPPOE_SERVER_INCOMPLETE` |
| CIDR 非法 | `validate_pppoe_server` | 问题列表 | `PPPOE_POOL_SUBNET_INVALID` |
| gateway 非法 | `validate_pppoe_server` | 问题列表 | `PPPOE_POOL_GATEWAY_INVALID` |
| `virtual_template` 非正整数/越界 | `validate_pppoe_server` | 问题列表 | `PPPOE_VIRTUAL_TEMPLATE_INVALID` |
| `authentication` 非 chap/pap | `validate_pppoe_server` | 问题列表 | `PPPOE_AUTH_INVALID` |
| `local_users` 空 | `validate_pppoe_server` | 问题列表 | `PPPOE_LOCAL_USERS_EMPTY` |
| `username` 空/非法 | `validate_pppoe_server` | 问题列表 | `PPPOE_LOCAL_USER_USERNAME_INVALID` |
| `password_cipher` 明文 | `_validate_password_cipher` → False | 问题列表，拒绝 | `PPPOE_PASSWORD_PLAINTEXT_REJECTED` |
| `password_cipher` 占位符格式错误（如 `${ENV}` / `$ENV_X`） | `_validate_password_cipher` → False | 问题列表，拒绝 | `PPPOE_PASSWORD_FORMAT_INVALID` |
| vendor 不支持 PPPoE 命令族 | `has_pppoe_commands` → False | 调用方结构化 reason，终止 | `PPPOE_UNSUPPORTED_VENDOR` |

**失败原则（确定性）**：所有校验失败均**不静默跳过**、**不产出部分命令序列**；问题列表以结构化 dict（`code`/`field`/`message`）返回，由调用方（STORY-038-04）聚合成 `SwitchDeployResult.errors` 或等价结构化报告（R-NF-002 风格）。本 Story 不负责真机下发异常（telnet 超时/回显差异），该层归 STORY-038-04 + telnet_client。

---

## 7. 测试设计

### 7.1 单元测试（`skills/topo-config/tests/test_commands_pppoe.py` + `test_physical_pool_pppoe.py`）

| 测试用例 | 覆盖要点 | 预期 |
|---------|---------|------|
| `test_commands_pppoe_five_keys_h3c` | H3C `pppoe` 五族键齐全 | 五键均在，无缺失 |
| `test_commands_pppoe_five_keys_ruijie` | Ruijie 五族键齐全 | 五键均在 |
| `test_commands_pppoe_five_keys_huawei` | Huawei 五族键齐全 | 五键均在 |
| `test_commands_pppoe_h3c_reference_text` | H3C 参考命令精确文本 | `remote address pool` / `gateway-list` / `Virtual-Template{template_id}`（无空格）/ `pppoe-server bind Virtual-Template{template_id}` 等逐条断言 |
| `test_commands_pppoe_three_vendor_diff` | 三厂商命令模板差异（SCN-038-POS-004） | 三厂商同一族键文本互异（如 ruijie `username` vs h3c/huawei `local-user`）；`{template_id}` 空格差异存在 |
| `test_commands_pppoe_no_3of3_claim` | ruijie/huawei 为 dry-run 模板 | 测试只断言键存在 + 文本非空，不标注真机覆盖（符合 R-NF-001） |
| `test_has_pppoe_commands_true` / `_false` | 能力探测 | 三厂商 True；未知/缺族键 False |
| `test_get_commands_backward_compat` | 既有键不变 | `get_commands('h3c')['create_vlan']` 等既有键文本不变（R-NF-004 局部回归） |
| `test_get_pppoe_server_present` | SW 节点含 `pppoe_server` 块 | 返回块 dict |
| `test_get_pppoe_server_absent` | SW 节点无块 / 节点不存在 | 返回 None |
| `test_get_pppoe_server_single_source` | R-F-019 同源一致 | 读取仅来自 physical_pool `_device_by_name`，无 devices.yaml 回退（构造无 devices 源的 fixture 断言行为） |
| `test_validate_pppoe_server_ok` | 完整合法块 | 空问题列表 |
| `test_validate_pppoe_server_incomplete` | 缺 `ip_pool_name`/`ip_pool_subnet` | 含 `PPPOE_SERVER_INCOMPLETE` |
| `test_validate_pppoe_server_subnet_invalid` | 非法 CIDR | 含 `PPPOE_POOL_SUBNET_INVALID` |
| `test_validate_pppoe_server_gateway_invalid` | 非法 gateway | 含 `PPPOE_POOL_GATEWAY_INVALID` |
| `test_validate_pppoe_server_virtual_template_invalid` | 0 / 负数 / 非 int | 含 `PPPOE_VIRTUAL_TEMPLATE_INVALID` |
| `test_validate_pppoe_server_auth_invalid` | `authentication: eap` | 含 `PPPOE_AUTH_INVALID` |
| `test_validate_pppoe_server_local_users_empty` | `local_users: []` | 含 `PPPOE_LOCAL_USERS_EMPTY` |
| `test_password_cipher_env_placeholder_ok` | `${ENV_PPPOE_USER_PASSWORD}` | `(True, 'env_placeholder')` |
| `test_password_cipher_h3c_cipher_ok` | `$c$3$...`（H3C 密文样例） | `(True, 'device_cipher')`，原样透传 |
| `test_password_cipher_huawei_cipher_ok` | `%^%#...#%^%#`（Huawei 密文样例） | `(True, 'device_cipher')` |
| `test_password_cipher_ruijie_cipher_ok` | `$1$...$...`（Ruijie 密文样例） | `(True, 'device_cipher')` |
| `test_password_cipher_plaintext_rejected` | `MyPlainPassword123` | `(False, 'plaintext')`，`validate_pppoe_server` 返回 `PPPOE_PASSWORD_PLAINTEXT_REJECTED` |
| `test_password_cipher_malformed_placeholder_rejected` | `${ENV}`（空变量名）/ `$ENV_X`（缺花括号） | `(False, 'plaintext')` → `PPPOE_PASSWORD_FORMAT_INVALID` |
| `test_password_cipher_not_reversed` | 设备密文 | 校验只做前缀判定，返回值不含任何解密/明文，密文原文透传（R-C-002） |

### 7.2 Fixture 需求

- `skills/topo-config/tests/fixtures/pool_pppoe_server.yaml`：含 SW 节点（node_type=SW，含 `pppoe_server` 完整块 + 一个 `role=pppoe-server` 无块负例节点）。
- 复用既有 `physical_ledger.yaml` fixture（brand 查询）。
- 密文样例在 fixture 内使用合成占位密文（如 `$c$3$QUJDREVGR0g=`），不落真实凭据。

### 7.3 集成/干跑

- STORY-038-04 集成：`get_pppoe_server` + `validate_pppoe_server` + `has_pppoe_commands` → `_build_pppoe_server_commands` dry-run diff（本 Story 仅提供前三个，diff 组装归 STORY-038-04）。
- SCN-038-POS-004 三厂商 diff 在 STORY-038-04 渲染后验证；本 Story 只验证三厂商模板键齐全 + 文本互异。

---

## 8. 实施步骤（最小切片）

| TASK-ID | 描述 | 预估 | 产物 |
|---------|------|:---:|------|
| TASK-E2-03-01a | physical_pool.py 增 `get_pppoe_server`（只读） | 30min | 读取接口 |
| TASK-E2-03-01b | physical_pool.py 增 `validate_pppoe_server` + `_validate_password_cipher` | 1.5h | 校验逻辑 |
| TASK-E2-03-01c | config/physical_pool.yaml SW 节点增 `pppoe_server` 示例块 | 30min | 数据源模板 |
| TASK-E2-03-02 | commands.py 三厂商 `pppoe` 五族键 + `has_pppoe_commands` | 1.5h | 命令模板 |
| TASK-E2-03-04 | 单测（§7.1 全用例 + fixture） | 2h | 测试 |
| **合计** | | **~6h** | |

**最小实现切片（按依赖顺序）**：
1. 切片 1（TASK-E2-03-01a + 01c）：只读 `get_pppoe_server` + 数据源模板，可独立验证「同源一致」（R-F-019）。
2. 切片 2（TASK-E2-03-02）：命令模板 + 能力探测，可独立验证「三厂商键齐全 + diff」（R-F-004）。
3. 切片 3（TASK-E2-03-01b）：校验逻辑 + 凭据占位校验，可独立验证安全约束（ADR-02 / R-C-001/002）。
4. 切片 4（TASK-E2-03-04）：全量单测收口。

每个切片均先写对应单测再实现（TDD），切片间无循环依赖。

---

## 9. 回滚策略

- **回滚 = 删除新增内容**：`COMMANDS[vendor]['pppoe']` 五族键、`has_pppoe_commands`、`get_pppoe_server`/`validate_pppoe_server`/`_validate_password_cipher`、`physical_pool.yaml` SW 节点 `pppoe_server` 示例块，全部为增量新增，回滚即删除对应片段，不改既有键/方法。
- **无破坏面**：`get_commands` / `get_brand` / `from_pool_data` / `_build_index` 签名与行为不变；既有 CR-037 `node2_dut1_tg1_linkN` 场景不触及（NFR1 / R-NF-004）。
- **切换条件**：若后续需 `ip pool` 段跨设备共享（ADR-CR038-03 回退），改为独立配置源时，仅替换 `get_pppoe_server` 读取源，不影响命令模板层。
- **发布**：随 `topo-config` skill 统一安装（沿用 CR-037 安装链路，本 Story 不新增安装脚本）。

---

## 10. 安全约束（`password_cipher` 校验，确定性规则）

### 10.1 校验规则（写死，`_validate_password_cipher`）

`password_cipher` 合法值判定，**按序短路**，返回 `(ok, kind)`：

1. **`${ENV_*}` 占位**：匹配正则 `^\$\{ENV_[A-Z0-9_]{1,63}\}$` → `(True, 'env_placeholder')`。
   - 只接受严格 `ENV_` 前缀 + 大写字母/数字/下划线变量名，花括号闭合。
   - 反例拒绝：`${ENV}`（空变量名）、`$ENV_X`（缺花括号）、`${env_x}`（小写前缀）、`${ENV_X}` 后跟多余字符。
2. **设备 cipher 密文**：匹配任一厂商密文前缀 allowlist → `(True, 'device_cipher')`，**原样透传，不反解**：
   - H3C Comware 可逆密文：`^\$c\$3\$[A-Za-z0-9+/=]{8,}$`
   - Huawei VRP 密文包裹：`^%^%#[A-Za-z0-9+/=_.]+#%^%#$`
   - Ruijie RGOS crypt 风格：`^\$[156]\$[A-Za-z0-9./]{1,16}\$[A-Za-z0-9./]{22,}$`
3. **其他一律判定为明文** → `(False, 'plaintext')`，拒绝（`PPPOE_PASSWORD_PLAINTEXT_REJECTED`）。

### 10.2 校验逻辑归属（确定性）

- `_validate_password_cipher(value)` 放 **`physical_pool.py`（模块私有方法）**，是唯一凭据校验实现点；`validate_pppoe_server` 在逐 user 校验时调用它，`commands.py` **不做**凭据校验（模板层无凭据逻辑，职责分离）。
- 报错结构（写入 `validate_pppoe_server` 问题列表）：

```python
{'code': 'PPPOE_PASSWORD_PLAINTEXT_REJECTED',
 'field': 'pppoe_server.local_users[0].password_cipher',
 'message': 'password_cipher 仅接受 ${ENV_*} 占位或设备 cipher 密文，拒绝明文（ADR-02 / R-C-001 / R-C-002）'}
```

- 占位符格式错误（如 `${ENV}`）→ `PPPOE_PASSWORD_FORMAT_INVALID`（区分「明文拒绝」与「占位符格式错误」两类，便于排障）。

### 10.3 安全红线（硬性）

- **拒绝明文**：任何不以 `${ENV_` 开头、且不匹配设备密文前缀的字符串，一律拒绝，不允许「兜底放行」（R-C-001）。
- **不反解设备 cipher**：`device_cipher` 只做前缀判定 + 原样透传，代码中不出现任何解密/反解逻辑（R-C-002）；密文值不出现在日志/报错/单测断言中的明文侧。
- **`${ENV_*}` 运行时解析不归本 Story**：占位符在真机下发阶段由调用方（ptm-atomic auth / 环境注入）解析，本 Story 只保证「占位合法」。
- **local-user 密码保持 cipher 密文**：`password_cipher` 注入命令模板的 `{cipher}` 位置时，只透传占位符或密文，不注入明文。
- **敏感信息不落日志**：`get_pppoe_server` / `validate_pppoe_server` 不 `logger.info` 打印 `password_cipher` 值（只打印 field 名 + code）。

---

## 11. 契约映射（R-F-003/004/019 → 实现点）

| 需求 | 本 Story 实现点 | 承接/下游 |
|------|----------------|-----------|
| R-F-003（PPPoE Server 命令族） | commands.py 三厂商 `pppoe` 五族键模板（ip pool / local-user / virtual-template / bind_interface）；「完整命令序列」由 STORY-038-04 `_build_pppoe_server_commands` 组装 | STORY-038-04 `configure_pppoe_server` |
| R-F-004（三厂商命令模板） | commands.py `COMMANDS[h3c/ruijie/huawei]['pppoe']` 五族键齐全 + 文本按厂商差异；H3C 参考、ruijie/huawei dry-run diff（R-NF-001） | STORY-038-04 渲染 + SCN-038-POS-004 |
| R-F-019（数据源同源一致） | physical_pool.py `get_pppoe_server` 唯一真相源 = `_device_by_name[name]['pppoe_server']`，无 devices.yaml/独立配置回退；`from_pool_data` 不改结构 | STORY-038-04 读取路径 |
| R-C-001（凭据 `${ENV_VAR}` 占位禁止明文） | `_validate_password_cipher` 规则 1 + 规则 3（明文拒绝） | §10 |
| R-C-002（local-user 密码保持 cipher 密文） | `_validate_password_cipher` 规则 2（设备密文透传不反解）+ 注入 `{cipher}` 原样透传 | §10 |
| ADR-CR038-03（数据源 = physical_pool `pppoe_server` 块） | `get_pppoe_server` 读取路径 + `pppoe_server` schema | §4 |

---

## 12. 依赖与前置

### 12.1 依赖

- `depends_on: []`（W1 起点，无上游 Story）。
- 只读依赖既有模块：`physical_pool.py`（`get_brand`/`get_device`/`_device_by_name` 索引）、`commands.py`（`get_commands`/`ip_to_mask`）、`physical_ledger.yaml`（brand）。
- 消费方（下游）：STORY-038-04（`switch_configurator.configure_pppoe_server` 消费命令模板 + `get_pppoe_server` + `validate_pppoe_server`）。

### 12.2 共享文件与扩展点预留（physical_pool.py）

- `physical_pool.py` 为**共享文件**：`shared: {physical_pool.py: [STORY-038-13]}`，`merge_owner: STORY-038-13`。本 Story 为**首次写入方**，只新增方法，不改既有方法体。
- **扩展点预留（关键）**：
  - 本 Story 新增的三个方法全部针对节点级键 `pppoe_server`；STORY-038-13 新增的 `get_trex_instances` / `get_trex_instance_of_port` / `get_free_ports` instance 透传全部针对 `trex_instances` / `interfaces[].instance`。二者是**同级独立节点键**，互不侵入。
  - 本 Story **不修改** `_build_index` / `from_pool_data` / `get_free_ports` / `get_interfaces`——这些是 STORY-038-13 最可能触碰的既有方法，避免合并冲突面。
  - 本 Story 新增方法以「只读查询 + 纯校验」定位，不写 `pool_data` / `ledger_data`，不与 STORY-038-13 的 trex 透传写路径竞争。
  - 合并责任：`merge_owner = STORY-038-13` 负责最终合并；本 Story 交付时只保证自身新增方法自洽、可单测、不影响既有行为。
- `config/physical_pool.yaml` 为**本 Story primary**（SW 节点 `pppoe_server` 示例块）；STORY-038-13（W3）改 TG 节点 `trex_instances`，节点不同 + Wave 串行，无同时写入冲突。

### 12.3 前置校验

- `design_evidence_confirmed=true` 且 `dev_gate=cp5-approved` 后才进入实现。
- 平台约束：本 Story 不涉及安装路径/平台目录，无需 `PLATFORM-CONTRACTS.yaml`（N/A，仅命令模板 + 数据源 schema）。

---

## 13. 风险与开放项

### 13.1 风险

| 风险 | 级别 | 缓解 |
|------|:---:|------|
| H3C 真机 PPPoE 命令与参考命令族存在差异（RA-038-002） | 高 | H3C 为参考实现；dry-run diff 供人工核对；真机下发独立 runtime_authorization；差异反馈回 `COMMANDS['h3c']` 模板（CR 回填） |
| ruijie/huawei 命令文本不准确（未经真机） | 中 | 显式标记 dry-run 模板；SCN-038-POS-004 只断言键齐全 + diff，不断言真机等价；Q-038-002 闭环前不宣称 3/3 覆盖 |
| 设备 cipher 密文前缀 allowlist 不完整（漏接合法密文 / 误放明文） | 中 | 保守规则（§10）：只放行已知前缀 + `${ENV_*}`，其余拒绝；漏接合法密文为安全方向的假阴性（可改用 `${ENV_*}` 占位绕过），误放明文为安全方向的假阳性（已用前缀 allowlist 杜绝） |
| 明文误判为密文（安全漏洞） | 高 | 前缀 allowlist 白名单 + 正则锚定（`^...$`），不「兜底放行」；单测覆盖明文拒绝 |
| `from_pool_data` 后续被 STORY-038-13 改动，与本 Story 新增方法冲突 | 低 | 本 Story 不改 `from_pool_data` / `_build_index`；新增方法独立；merge_owner STORY-038-13 负责合并 |

### 13.2 开放项

| 编号 | 内容 | 状态 | 处理 |
|------|------|------|------|
| Q-038-002 | H3C 真机 PPPoE 命令与参考命令族差异 | OPEN（non-blocking-open） | dry-run diff + 真机独立授权核对；差异经 CR 回填命令模板 |
| LCQ-STORY-038-03-01 | 设备 cipher 密文前缀 allowlist 精确性（H3C `$c$3$` / Huawei `%^%#` / Ruijie crypt） | OPEN（non-blocking） | 保守前缀 allowlist 已确定性写入 §10；真机密文样例复核后固化 |

### 13.3 实现灰区与取舍记录

**灰区 1：`pppoe.*` 命令键的粒度（单字符串 vs 嵌套 dict）**
- 问题：Story 列 5 个命令键，但 PPPoE 每阶段是多行命令（进视图 + 多配置）。
- 选项：A. `COMMANDS[vendor]['pppoe']` 嵌套 dict（5 族键 → 子命令模板）；B. 全部扁平化为 `pppoe.ip_pool_create` 等 11+ 个原子键；C. 单字符串多命令拼接（用 `\n` 分隔）。
- 决策：选 A。5 族键与 Story 卡片 §目标字面一致，子命令 dict 承载多行结构；渲染层（STORY-038-04）按 `.format()` 逐子键展开，复用 `_build_vlan_commands` 模式。C 的 `\n` 拼接破坏「每命令一个 list 元素」的 `send_commands_check` 消费契约，排除。
- 重访条件：若 STORY-038-04 渲染需要更细粒度原子键，可将子键提升为扁平键（A→B），命令文本不变。

**灰区 2：`enable_server` 空字符串语义**
- 问题：H3C/Huawei/Ruijie 无全局「pppoe enable」，使能靠 `bind_interface`。
- 决策：`enable_server` 值为空字符串 `''`，渲染层跳过空字符串（不注入空命令）；保留该键以维持三厂商键集一致（FR1）。备选「删除该键」会导致三厂商键集不一致，破坏 R-F-004 键齐全断言。
- 重访条件：若某厂商真机确认需要全局使能命令（如 `pppoe enable`），填入对应模板字符串。

**灰区 3：设备 cipher 前缀 allowlist（安全边界）**
- 问题：如何确定性区分「设备 cipher 密文」与「明文」，且不反解密文。
- 决策：前缀 allowlist 白名单（§10.2），锚定正则，其余拒绝；保守策略保证「拒绝明文」优先级高于「接纳全部密文」。已写入 LCQ-STORY-038-03-01（non-blocking）。
- 重访条件：真机密文样例出现未覆盖前缀时，追加该前缀（经 host-orchestrator 汇总确认后）并补单测。

---

## 14. 验收与交接

### 14.1 Definition of Done（DoD）

- [ ] `COMMANDS[h3c/ruijie/huawei]['pppoe']` 五族键齐全，无缺失（FR1，SCN-038-POS-004）
- [ ] H3C 五族键命令文本为参考实现，与 §3.2 表逐条一致（FR2）
- [ ] ruijie/huawei 五族键存在且文本按厂商差异，仅 dry-run diff 供人工核对，未标注「3/3 真机覆盖」（FR3 / R-NF-001）
- [ ] `get_pppoe_server` 从 physical_pool 唯一真相源读取，无 devices.yaml/独立配置回退（FR4 / R-F-019）
- [ ] `validate_pppoe_server` 全字段校验，含 `PPPOE_SERVER_INCOMPLETE` 结构化错误码（FR5）
- [ ] `_validate_password_cipher` 确定性判定 `${ENV_*}` / 设备密文放行、明文拒绝；设备密文不反解（FR6 / FR7 / R-C-001 / R-C-002）
- [ ] `physical_pool.yaml` SW 节点示例含 `pppoe_server` 块，`password_cipher` 用 `${ENV_*}` 占位
- [ ] `has_pppoe_commands` 能力探测正确
- [ ] 既有 `get_commands` 键与 CR-037 行为不变（NFR1 / R-NF-004 局部回归）
- [ ] §7.1 单测全绿（`uv run --python 3.11 pytest skills/topo-config/tests/`）

### 14.2 验收标准对齐

| Story 验收标准 | 证据 |
|---------------|------|
| SCN-038-POS-004 三厂商命令模板差异正确 | §7.1 `test_commands_pppoe_three_vendor_diff` + `test_commands_pppoe_five_keys_*` |
| R-F-019 数据源同源一致 | §7.1 `test_get_pppoe_server_single_source` |
| `password_cipher` 明文拒绝、`${ENV_*}`/密文放行（ADR-02） | §7.1 `test_password_cipher_*` 系列 + §10 |

### 14.3 交接摘要（给 STORY-038-04 / meta-qa）

- 消费接口：`COMMANDS[vendor]['pppoe']`（五族键模板）、`get_pppoe_server`、`validate_pppoe_server`、`has_pppoe_commands`、`_validate_password_cipher`（§3）。
- 验证入口：`uv run --python 3.11 pytest skills/topo-config/tests/test_commands_pppoe.py skills/topo-config/tests/test_physical_pool_pppoe.py`。
- 风险提示：ruijie/huawei 命令为 dry-run 模板未经真机验证（Q-038-002）；设备 cipher 前缀 allowlist 为保守白名单（LCQ-STORY-038-03-01）。
- 未覆盖项：命令序列组装（STORY-038-04）、真机下发（独立 runtime_authorization）、回滚闭环（DEF-038-04 延后）。

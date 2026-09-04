---
cr_id: "CR-033"
story_id: "STORY-EX-03"
title: "TG 路由参数化（op_mapper _build_exec_env 注入 TREX_API_URL + resolve_env_refs）"
wave: 1
priority: "P0"
lld_policy: "full-lld"
feature_design_refs: []
depends_on: ["ST-EX-01"]
dependency_type: "hard"
status: "lld-ready"
design_evidence_type: "full-lld"
lld_policy_required_level: "full-lld"
source_hld: "docs/design/HLD-PTM-TE-EXEC.md"
source_adr: "docs/design/ARCHITECTURE-DECISION-PTM-TE-EXEC.md"
source_requirements: "docs/product/REQUIREMENTS-PTM-TE-EXEC.md"
created_at: "2026-07-28T16:00:00+08:00"
author: "meta-dev"
---

# ST-EX-03 TG 路由参数化 LLD

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-28 | meta-dev | CR-033 ST-EX-03 full-lld 初稿：14 章节全覆盖；_build_exec_env 签名扩展（ADR-05 方案 a）；resolve_env_refs 新增（ADR-09）；execute_op 签名扩展（env_topology + tg_api_server）；9 类 ${ENV.*} 占位符；向后兼容；ptm-atomic 约束 |

## 0. 概述

本 LLD 是 ST-EX-03 TG 路由参数化的 full-lld 设计证据，属于 CR-033 ptm-te 执行引擎的 Wave 1 基础 Story，承载 op_mapper.py 环境参数化层扩展。

**需求溯源**：ST-EX-03 承载 R-F-018（TREX_API_URL 注入消除 TG_URL 硬编码）、R-F-027（环境文件驱动）、R-F-029（一次编写多环境执行）等功能需求；详细需求映射见 §1 工程依据表。

**模块拆分**：op_mapper.py 按职责拆分为 _build_exec_env（签名扩展注入 TREX_API_URL，ADR-05 方案 a）/resolve_env_refs（新增 ${ENV.*} 占位符解析，ADR-09）/execute_op（签名扩展 env_topology + tg_api_server 参数）/handle_rollback（透传 env_topology + tg_api_server）/_reconnect_and_retry（透传 tg_api_server）等函数锚点（ST-EX-03 拥有）；不新增 op_id（EXPECTED_OP_COUNT=21 不变）。

**代码结构**：详见 §2 文件影响范围。修改 skills/policy-route-execution/scripts/op_mapper.py 5 个函数锚点，新增 resolve_env_refs 约 80 行 + 签名扩展约 35 行；不触碰 case_runner.py（ST-EX-04 拥有）/install.py（ST-EX-02 拥有）/device-management（ST-EX-01 拥有）。

**技术细节**：_build_exec_env 注入 TREX_API_URL（§3.1，ADR-05 方案 a，环境文件优先 ${ENV.tg.url}，devices.yaml fallback）；resolve_env_refs 9 类 ${ENV.*} 占位符解析（§3.2，ADR-09）；向后兼容（§5，Gotcha #12，新参数默认值 None/"" 既有调用方无需修改）；ptm-atomic 约束（§12，Gotcha #13，resolve_env_refs 只做参数解析不触网）等核心技术细节在各章节展开。

**Definition of Done (DoD)**：CP6 实现完成以 §6 测试设计的 14 测试用例（TC-EX-03-01..14）全通过为准；CP7 验证采用 static-only（DQ-01 推荐），--execute 集成测试需独立 runtime_authorization。DoD 详见 §6.2/§6.4 测试矩阵与覆盖矩阵。

## 1. 工程依据

| 依据类型 | 引用 | 关联点 |
|---|---|---|
| HLD §12.2 | execute_op 流程（resolve_env_refs -> resolve_step_refs -> validate_args -> build_command -> _build_exec_env） | 核心流程顺序 |
| HLD §12.3 | 环境解析层设计（${ENV.*} 9 类占位符 + env_topology 契约 + ptm-atomic 约束） | resolve_env_refs 实现依据 |
| HLD §14 | RA-005（tg op 走旧地址）/ RA-013（${ENV.*} 路径不匹配）/ RA-014（环境文件缺失） | 风险应对 |
| HLD Gotcha | #3（TREX_API_URL 注入方案 a 定稿）/ #11（YAML 1.1 整数陷阱）/ #12（向后兼容）/ #13（ptm-atomic 约束） | 实现规避 |
| ADR-05 | TREX_API_URL 注入边界（方案 a 扩展 _build_exec_env 签名，环境文件优先，devices.yaml fallback） | _build_exec_env 签名扩展 |
| ADR-09 | 环境文件驱动（resolve_env_refs + ${ENV.*} + --env-file） | resolve_env_refs 新增 |
| 需求 | R-F-018（TREX_API_URL 注入消除 TG_URL 硬编码）/ R-F-027（环境文件驱动）/ R-F-029（一次编写多环境执行） | 成功标准 SM-EX-12 |
| 开发计划 | DEVELOPMENT-PLAN.yaml Wave 1 ST-EX-03 | file_ownership + depends_on |
| 现有代码 | `op_mapper.py` `_build_exec_env`（line 793-819）/ `execute_op`（line 1079-1270）/ `resolve_step_refs`（line 889-943） | 修改对象现状 |

**file_ownership 声明**（与 DEVELOPMENT-PLAN.yaml 一致，CP3 P1 落实）：
- `skills/policy-route-execution/scripts/op_mapper.py#_build_exec_env`
- `skills/policy-route-execution/scripts/op_mapper.py#resolve_env_refs`

**不触碰文件**：
- `op_mapper.py#resolve_step_refs`（既有函数，ST-EX-03 不修改，仅在 execute_op 中调整调用顺序）
- `op_mapper.py#build_command`（既有函数，ST-EX-03 不修改签名，resolve_env_refs 在 build_command 前执行）
- `op_mapper.py#handle_rollback`（既有函数，ST-EX-03 仅新增 env_topology/tg_api_server 参数透传）
- `skills/device-management/*`（ST-EX-01 拥有）
- `script/ptm_team/install.py`（ST-EX-02 拥有）
- `skills/case-execution/*`（ST-EX-04 拥有，本 Story 不新建 case_runner.py）

## 2. 文件影响范围

| 文件 | 归属段 | 变更类型 | 变更内容 |
|---|---|---|---|
| `skills/policy-route-execution/scripts/op_mapper.py` | `_build_exec_env` | 修改签名+函数体 | 新增 `tg_api_server: str = ""` 参数；非空时注入 `TREX_API_URL` 环境变量 |
| `skills/policy-route-execution/scripts/op_mapper.py` | `resolve_env_refs` | 新增函数 | 扫描 args 中 `${ENV.*}` 占位符，按 env_topology 解析替换 |
| `skills/policy-route-execution/scripts/op_mapper.py` | `execute_op` | 修改签名+函数体 | 新增 `env_topology: Optional[dict] = None` + `tg_api_server: str = ""` 参数；在 resolve_step_refs 前调用 resolve_env_refs；_build_exec_env 传 tg_api_server |
| `skills/policy-route-execution/scripts/op_mapper.py` | `handle_rollback` | 修改函数体 | execute_op 调用透传 env_topology + tg_api_server（回滚的 inverse_op 也需要 TREX_API_URL 注入） |
| `skills/policy-route-execution/scripts/op_mapper.py` | `_reconnect_and_retry` | 修改函数体 | _build_exec_env 调用传 tg_api_server（STATE_INVALID 重连后重试也需要 TREX_API_URL） |

**变更行数预估**：新增 resolve_env_refs 约 80 行 + _build_exec_env 修改约 10 行 + execute_op 修改约 15 行 + handle_rollback 修改约 5 行 + _reconnect_and_retry 修改约 5 行 ≈ 115 行。

## 3. 接口设计

### 3.1 _build_exec_env 签名扩展（ADR-05 方案 a 定稿）

**现有签名**：
```python
def _build_exec_env(base_url: str) -> dict:
```

**扩展后签名**：
```python
def _build_exec_env(base_url: str, tg_api_server: str = "") -> dict:
    """构建 subprocess 环境变量。

    1. 从 base_url 提取 DUT host -> NO_PROXY（既有逻辑，P2-12）
    2. tg_api_server 非空时注入 TREX_API_URL（新增，ADR-05）

    Args:
        base_url: DUT 设备 Web 管理地址
        tg_api_server: TG api_server 地址（IP:端口），由 case_runner 提供
                       （env_topology ${ENV.tg.url} 优先，devices.yaml tg.api_server fallback）。
                       空字符串时不注入 TREX_API_URL（向后兼容）。

    Returns:
        env dict（os.environ 副本 + NO_PROXY + 可选 TREX_API_URL）
    """
```

**实现要点**：
- `tg_api_server` 非空时 `env["TREX_API_URL"] = tg_api_server`
- `tg_api_server` 为空时不设置 `TREX_API_URL`（保持 os.environ 原值，向后兼容）
- 不从 env_topology 读取（op_mapper 不负责来源选择，由 case_runner 完成 ADR-05 fallback）

### 3.2 resolve_env_refs 新增函数（ADR-09）

**签名**：
```python
# ${ENV.*} 占位符正则（完整匹配）
_ENV_REF_RE = re.compile(r"^\$\{ENV\.([a-zA-Z0-9_.\[\],\s]+)\}$")

# 聚合端口数组正则：tg.ports[port1,port2]
_ENV_PORTS_AGG_RE = re.compile(r"^tg\.ports\[([a-zA-Z0-9_,\s]+)\]$")


def resolve_env_refs(args: dict, env_topology: Optional[dict]) -> dict:
    """扫描 args 值中的 ${ENV.*} 占位符，按 env_topology 解析替换。

    执行顺序（HLD §12.2）：resolve_env_refs 在 resolve_step_refs 前。
    无 env_topology 或 args 无 ${ENV.*} 占位符 -> 原样返回（向后兼容，Gotcha #12）。
    解析失败 -> 抛 ValueError（execute_op 捕获为 VALIDATION_FAILED envelope）。

    Args:
        args: ptm-tde PC 的 atomic_op.args dict，值可含 ${ENV.*} 占位符
        env_topology: 环境拓扑（port_mapping/nodes/links），由 case_runner
                      load_env_file 加载 --env-file 生成。None 时原样透传。

    Returns:
        解析后的 args dict（${ENV.*} 已替换为实际值）

    Raises:
        ValueError: ${ENV.*} 占位符无法在 env_topology 中解析
    """
```

**9 类 ${ENV.*} 占位符解析表**（HLD §12.3）：

| 占位符 | 解析目标 | 返回类型 | 说明 |
|---|---|---|---|
| `${ENV.tg.port1}` | `port_mapping.port1.tg` | str | TG 物理端口名 |
| `${ENV.tg.port1.ip}` | `nodes.tg1.interfaces.port1.ip` | str | TG 端口 IP |
| `${ENV.tg.port1.gw}` | `nodes.tg1.interfaces.port1.gw` | str | TG 端口网关 |
| `${ENV.dut.port1}` | `port_mapping.port1.dut` | str | DUT 物理端口名 |
| `${ENV.dut.port1.ip}` | `nodes.dut1.interfaces.port1.ip` | str | DUT 端口 IP |
| `${ENV.tg.url}` | `nodes.tg1.trex_api_url` | str | TG api_server URL（TREX_API_URL 来源，ADR-05） |
| `${ENV.dut.url}` | `nodes.dut1.host` | str | DUT URL |
| `${ENV.dut.next_hop}` | `nodes.dut1.next_hop` | str | DUT 下一跳 IP（policy-route next_hop_ip） |
| `${ENV.tg.ports[port1,port2]}` | `[port_mapping.port1.tg, port_mapping.port2.tg]` | list[str] | 聚合端口名数组 |

**占位符语法规则**：
- 完整匹配：值必须完全等于 `${ENV.*}` 格式（正则 `^\$\{ENV\.(.+)\}$`），不支持部分插值
- `port1`/`port2` 是逻辑端口序号，对齐 `port_mapping` 键，不随环境变化
- `tg`/`dut` 是逻辑角色名，映射到 `nodes.tg1`/`nodes.dut1`（单 TG/DUT 环境假设）
- 聚合格式 `${ENV.tg.ports[port1,port2]}` 中括号内为逻辑端口列表，逗号分隔

**解析逻辑**：
1. `env_topology` 为 None 或 args 不是 dict -> 原样返回（向后兼容）
2. 遍历 args 每个值，非字符串原样保留
3. 字符串值用 `_ENV_REF_RE` 匹配，不匹配原样保留
4. 匹配后提取路径（如 `tg.port1.ip`），按解析表查找 env_topology
5. 聚合格式 `tg.ports[...]` 用 `_ENV_PORTS_AGG_RE` 子匹配，返回端口名列表
6. 查找失败（键不存在）-> 抛 `ValueError`，含占位符和缺失键描述

### 3.3 execute_op 签名扩展

**现有签名**（节选关键参数）：
```python
def execute_op(
    op_id: str,
    args: dict,
    base_url: str,
    session_file: str,
    *,
    step_name: str = "",
    dry_run: bool = True,
    authorized: bool = False,
    timeout: int = 30,
    username: str = "admin",
    password_env: str = "FW_WEB_PASSWORD",
    exec_log_path: Optional[str] = None,
    diag_snapshot_ref: str = "",
    step_id: str = "",
    step_refs_dir: Optional[str] = None,
) -> dict:
```

**扩展后签名**（新增 2 个 keyword-only 参数）：
```python
def execute_op(
    op_id: str,
    args: dict,
    base_url: str,
    session_file: str,
    *,
    step_name: str = "",
    dry_run: bool = True,
    authorized: bool = False,
    timeout: int = 30,
    username: str = "admin",
    password_env: str = "FW_WEB_PASSWORD",
    exec_log_path: Optional[str] = None,
    diag_snapshot_ref: str = "",
    step_id: str = "",
    step_refs_dir: Optional[str] = None,
    env_topology: Optional[dict] = None,    # 新增：环境拓扑（ADR-09）
    tg_api_server: str = "",                # 新增：TG api_server（ADR-05，case_runner 完成 fallback）
) -> dict:
```

**参数职责分离**：
- `env_topology`：仅用于 `resolve_env_refs` 解析 `${ENV.*}` 占位符
- `tg_api_server`：仅用于 `_build_exec_env` 注入 `TREX_API_URL`，由 case_runner 完成 ADR-05 fallback（env_topology `${ENV.tg.url}` 优先，devices.yaml `tg.api_server` fallback）
- 两者独立：op_mapper 不从 env_topology 提取 tg_api_server（职责分离，op_mapper 不读 devices.yaml，HLD v1.2 R1）

**向后兼容**：两个新参数均有默认值（None / ""），既有调用方（op_mapper.py CLI `_cli_main`）无需修改即可工作。

### 3.4 handle_rollback 参数透传

handle_rollback 调用 execute_op 执行 inverse_op 时，需透传 env_topology + tg_api_server，确保回滚操作也能解析 `${ENV.*}` 和注入 TREX_API_URL（如 tg_start_traffic_stream 的回滚 tg_stop_traffic_stream 需要 TREX_API_URL）。

**扩展后 handle_rollback 新增参数**：
```python
def handle_rollback(
    op_id: str,
    args: dict,
    base_url: str,
    session_file: str,
    *,
    pre_snapshot: Optional[dict] = None,
    authorized: bool = False,
    timeout: int = 30,
    result_envelope: Optional[dict] = None,
    env_topology: Optional[dict] = None,    # 新增
    tg_api_server: str = "",                # 新增
) -> dict:
```

handle_rollback 内部调用 execute_op（inverse_op / restore_op）时透传这两个参数。

### 3.5 _reconnect_and_retry 参数透传

STATE_INVALID 重连后重试原命令时，_build_exec_env 需传 tg_api_server，确保重试也注入 TREX_API_URL。

**扩展后 _reconnect_and_retry 新增参数**：
```python
def _reconnect_and_retry(
    base_url: str,
    session_file: str,
    username: str,
    password_env: str,
    retry_command: List[str],
    timeout: int,
    tg_api_server: str = "",    # 新增
) -> dict:
```

## 4. 核心流程

### 4.1 execute_op 流程（HLD §12.2，扩展后）

```
execute_op(op_id, args, base_url, session_file, *, env_topology=None, tg_api_server="", ...)
  │
  ├─ [1] resolve_env_refs(args, env_topology)           ← 新增（ADR-09）
  │   ├─ env_topology=None 或无 ${ENV.*} -> 原样返回（向后兼容，Gotcha #12）
  │   ├─ 扫描 args 值中 ${ENV.*} 占位符
  │   ├─ 按 env_topology（port_mapping/nodes/links）解析替换
  │   └─ 解析失败 -> ValueError -> VALIDATION_FAILED envelope
  │
  ├─ [2] resolve_step_refs(args, step_refs_dir, ...)    ← 既有（顺序在 resolve_env_refs 后）
  │   └─ 插值 ${STEP-N.id}（既有逻辑不变）
  │
  ├─ [3] validate_args(op_id, args)                     ← 既有（resolve_env_refs/resolve_step_refs 后校验）
  │   └─ 参数合法性预检（占位符/IP/对象名，既有逻辑不变）
  │
  ├─ [4] build_command(op_id, args, base_url, ...)      ← 既有（不变）
  │   └─ 组装 ptm-atomic run <family> <action> [flags]
  │
  ├─ [5] _build_exec_env(base_url, tg_api_server)       ← 扩展（ADR-05 方案 a）
  │   ├─ 从 base_url 提取 DUT host -> NO_PROXY（既有）
  │   └─ tg_api_server 非空 -> env["TREX_API_URL"] = tg_api_server（新增）
  │
  └─ [6] subprocess.run(command, env=_build_exec_env)   ← 既有
      └─ ptm-atomic 读 TREX_API_URL 环境变量，连接 TG api_server
```

**执行顺序约束**（HLD §12.2，ADR-09）：`resolve_env_refs -> resolve_step_refs -> validate_args -> build_command`。resolve_env_refs 必须在 resolve_step_refs 前，因为 ${ENV.*} 是环境层占位符，${STEP-N.*} 是步骤间引用，先解析环境再解析步骤引用确保参数完整性。

### 4.2 resolve_env_refs 解析流程

```
resolve_env_refs(args, env_topology)
  │
  ├─ [1] 前置检查
  │   ├─ env_topology is None -> return args（原样透传，Gotcha #12）
  │   ├─ not isinstance(args, dict) -> return args
  │   └─ 提取 port_mapping = env_topology.get("port_mapping", {})
  │      提取 nodes = env_topology.get("nodes", {})
  │      提取 tg_node = nodes.get("tg1", {})
  │      提取 dut_node = nodes.get("dut1", {})
  │
  ├─ [2] 遍历 args
  │   └─ for key, val in args.items():
  │       ├─ val 非字符串 -> resolved[key] = val（原样保留）
  │       ├─ _ENV_REF_RE 不匹配 -> resolved[key] = val（原样保留，Gotcha #12）
  │       └─ 匹配 -> 提取 path，按解析表查找：
  │           ├─ tg.port{N} -> port_mapping[port{N}][tg]
  │           ├─ tg.port{N}.ip -> tg_node[interfaces][port{N}][ip]
  │           ├─ tg.port{N}.gw -> tg_node[interfaces][port{N}][gw]
  │           ├─ dut.port{N} -> port_mapping[port{N}][dut]
  │           ├─ dut.port{N}.ip -> dut_node[interfaces][port{N}][ip]
  │           ├─ tg.url -> tg_node[trex_api_url]
  │           ├─ dut.url -> dut_node[host]
  │           ├─ dut.next_hop -> dut_node[next_hop]
  │           ├─ tg.ports[p1,p2,...] -> [port_mapping[p1][tg], port_mapping[p2][tg], ...]
  │           └─ 其他路径 -> ValueError（未知占位符路径）
  │
  ├─ [3] 查找失败处理
  │   └─ 键不存在或值为 None/"" -> ValueError
  │      （含占位符原文 + 缺失的 env_topology 键路径）
  │
  └─ [4] 返回 resolved args
```

**注意**：`port{N}` 是通配，支持 port1/port2/port3 等任意逻辑端口序号，N 由占位符路径决定。

### 4.3 _build_exec_env 扩展流程

```
_build_exec_env(base_url, tg_api_server="")
  │
  ├─ [1] env = dict(os.environ)（既有）
  │
  ├─ [2] 从 base_url 提取 DUT host -> NO_PROXY（既有，P2-12）
  │
  ├─ [3] tg_api_server 非空 -> env["TREX_API_URL"] = tg_api_server（新增，ADR-05）
  │   └─ tg_api_server 为空 -> 不设置（保持 os.environ 原值，向后兼容）
  │
  └─ [4] return env
```

## 5. 异常处理

| 异常场景 | 触发条件 | 处理方式 | error_type | envelope status |
|---|---|---|---|---|
| ${ENV.*} 占位符路径未知 | resolve_env_refs 遇到不在 9 类解析表中的路径 | raise ValueError -> execute_op 捕获 | VALIDATION_FAILED | error |
| ${ENV.*} 占位符键缺失 | env_topology 中对应键不存在或值为 None/"" | raise ValueError -> execute_op 捕获 | VALIDATION_FAILED | error |
| env_topology 结构不完整 | port_mapping/nodes 缺失 | resolve_env_refs 用空 dict 兜底，占位符解析时键缺失报 VALIDATION_FAILED | VALIDATION_FAILED | error |
| env_topology 为 None | 既有调用方未传 env_topology | resolve_env_refs 原样返回 args（向后兼容，Gotcha #12） | NONE | 正常流程 |
| tg_api_server 为空 | 既有调用方未传 tg_api_server | _build_exec_env 不注入 TREX_API_URL（向后兼容） | NONE | 正常流程 |
| resolve_env_refs 在 dry-run 模式 | dry_run=True | resolve_env_refs 仍执行（验证 ${ENV.*} 解析正确性，HLD §13.3 / Gotcha #12） | NONE | 正常流程 |
| TG op 无 TREX_API_URL | tg_api_server="" 且 os.environ 无 TREX_API_URL | ptm-atomic 子进程可能连接失败 | EXEC_FAILED | error（由 ptm-atomic 返回） |

**异常捕获位置**：execute_op 的 try/except 块（line 1128-1150 既有），新增 resolve_env_refs 调用在同一 try 块内，ValueError 已被既有 except 捕获为 VALIDATION_FAILED。

**关键约束**（Gotcha #13）：resolve_env_refs 只做参数解析，不发起网络请求；TREX_API_URL 经 _build_exec_env 注入子进程环境变量，由 ptm-atomic 子进程消费；框架禁止直接调 TG REST API。

## 6. 测试设计

### 6.1 测试策略

validation_mode=static-only（CP7 采用 static review + dry-run，HLD §13.1）。不引入 pytest/robot（R-C-002），使用 op_mapper.py 既有 CLI `map`/`execute --dry-run` 子命令 + `validate` 子命令做静态校验。

### 6.2 测试用例矩阵

| 用例 ID | 测试目标 | 输入 | 预期 | 类型 |
|---|---|---|---|---|
| TC-EX-03-01 | resolve_env_refs 向后兼容（无 ${ENV.*}） | args={src_ip:"1.1.1.1"}, env_topology=None | 原样返回 args | unit |
| TC-EX-03-02 | resolve_env_refs 向后兼容（env_topology=None） | args={tx_port:"${ENV.tg.port1}"}, env_topology=None | 原样返回 args | unit |
| TC-EX-03-03 | ${ENV.tg.port1} 解析 | args={tx_port:"${ENV.tg.port1}"}, env_topology={port_mapping:{port1:{tg:"1/1/1"}}} | args={tx_port:"1/1/1"} | unit |
| TC-EX-03-04 | ${ENV.tg.url} 解析 | args={url:"${ENV.tg.url}"}, env_topology={nodes:{tg1:{trex_api_url:"10.113.52.253:8450"}}} | args={url:"10.113.52.253:8450"} | unit |
| TC-EX-03-05 | ${ENV.dut.next_hop} 解析 | args={next_hop_ip:"${ENV.dut.next_hop}"}, env_topology={nodes:{dut1:{next_hop:"192.168.102.1"}}} | args={next_hop_ip:"192.168.102.1"} | unit |
| TC-EX-03-06 | ${ENV.tg.ports[port1,port2]} 聚合 | args={ports:"${ENV.tg.ports[port1,port2]}"}, env_topology={port_mapping:{port1:{tg:"1/1/1"},port2:{tg:"1/1/2"}}} | args={ports:["1/1/1","1/1/2"]} | unit |
| TC-EX-03-07 | ${ENV.*} 键缺失 | args={tx_port:"${ENV.tg.port1}"}, env_topology={port_mapping:{}} | raise ValueError | unit |
| TC-EX-03-08 | 9 类占位符全覆盖 | 9 个 ${ENV.*} 占位符 + 完整 env_topology | 全部解析成功 | unit |
| TC-EX-03-09 | _build_exec_env 注入 TREX_API_URL | _build_exec_env("https://10.113.55.51", "10.113.52.253:8450") | env["TREX_API_URL"]=="10.113.52.253:8450" | unit |
| TC-EX-03-10 | _build_exec_env 向后兼容 | _build_exec_env("https://10.113.55.51", "") | "TREX_API_URL" 不在 env 或保持原值 | unit |
| TC-EX-03-11 | execute_op dry-run + env_topology | execute_op(tg_config_interface, args含${ENV.*}, dry_run=True, env_topology=...) | resolve_env_refs 执行，build_command 成功，不执行 mutation | integration |
| TC-EX-03-12 | execute_op dry-run + ${ENV.*} 解析失败 | execute_op(..., env_topology不完整) | envelope error_type=VALIDATION_FAILED | integration |
| TC-EX-03-13 | validate_mapping_consistency 通过 | validate 子命令 | EXPECTED_OP_COUNT=21 通过（ST-EX-03 不新增 op_id） | regression |
| TC-EX-03-14 | handle_rollback 透传 env_topology | handle_rollback(tg_start_traffic_stream, ..., env_topology=...) | inverse_op tg_stop_traffic_stream 解析 ${ENV.*} 成功 | integration |

### 6.3 dry-run 验证命令

```bash
# 1. 映射表一致性校验（回归）
python skills/policy-route-execution/scripts/op_mapper.py validate

# 2. ${ENV.*} 解析 dry-run（需 case_runner 构造 env_topology，ST-EX-04 后可用）
# ST-EX-03 范围内用 Python 内联验证：
python -c "
import sys; sys.path.insert(0, 'skills/policy-route-execution/scripts')
from op_mapper import resolve_env_refs
env = {'port_mapping': {'port1': {'tg': '1/1/1', 'dut': 'eth0'}},
       'nodes': {'tg1': {'trex_api_url': '10.113.52.253:8450',
                          'interfaces': {'port1': {'ip': '192.168.101.1', 'gw': '192.168.101.2'}}},
                 'dut1': {'host': '10.113.55.51', 'next_hop': '192.168.102.1',
                          'interfaces': {'port1': {'ip': '192.168.101.2'}}}}}
args = {'tx_port': '\${ENV.tg.port1}', 'next_hop_ip': '\${ENV.dut.next_hop}'}
print(resolve_env_refs(args, env))
# 预期: {'tx_port': '1/1/1', 'next_hop_ip': '192.168.102.1'}
"
```

### 6.4 覆盖矩阵

| 需求/成功标准 | 测试用例 | 覆盖状态 |
|---|---|---|
| SM-EX-12（${ENV.*} 9 类占位符） | TC-EX-03-03..08 | 9 类全覆盖 |
| R-F-018（TREX_API_URL 注入） | TC-EX-03-09/11 | 覆盖 |
| R-F-027（环境文件驱动） | TC-EX-03-03..08/11 | 覆盖 |
| R-F-029（一次编写多环境） | TC-EX-03-06（聚合端口） | 覆盖 |
| 向后兼容（Gotcha #12） | TC-EX-03-01/02/10 | 覆盖 |
| ptm-atomic 约束（Gotcha #13） | code review 检查无 requests/urllib 直接调 TG | 覆盖 |

## 7. 实施步骤

1. **新增 resolve_env_refs 函数**（约 80 行）
   - 位置：`op_mapper.py` resolve_step_refs 函数前（语义上 resolve_env_refs 在 resolve_step_refs 前执行）
   - 定义 `_ENV_REF_RE` 和 `_ENV_PORTS_AGG_RE` 正则常量
   - 实现 9 类占位符解析逻辑
   - 向后兼容：env_topology=None 或无 ${ENV.*} 原样返回
   - 解析失败抛 ValueError

2. **扩展 _build_exec_env 签名**（约 10 行修改）
   - 新增 `tg_api_server: str = ""` 参数
   - tg_api_server 非空时 `env["TREX_API_URL"] = tg_api_server`
   - 空时不设置（向后兼容）

3. **扩展 execute_op 签名 + 函数体**（约 15 行修改）
   - 新增 `env_topology: Optional[dict] = None` + `tg_api_server: str = ""` keyword-only 参数
   - 在 resolve_step_refs 调用前新增 resolve_env_refs 调用（同一 try 块内，ValueError 被既有 except 捕获）
   - _build_exec_env 调用传入 tg_api_server（3 处：正常执行 line 1179、超时不涉及、_reconnect_and_retry 内部）

4. **扩展 _reconnect_and_retry 签名 + 函数体**（约 5 行修改）
   - 新增 `tg_api_server: str = ""` 参数
   - _build_exec_env 调用传入 tg_api_server（2 处：login_cmd、retry_command）
   - execute_op 调用 _reconnect_and_retry 时传 tg_api_server

5. **扩展 handle_rollback 签名 + 函数体**（约 5 行修改）
   - 新增 `env_topology: Optional[dict] = None` + `tg_api_server: str = ""` 参数
   - inverse_op / restore_op 的 execute_op 调用透传 env_topology + tg_api_server

6. **更新 execute_op docstring**
   - 新增 env_topology / tg_api_server 参数说明
   - 流程描述新增 resolve_env_refs 步骤

7. **静态校验**
   - `python op_mapper.py validate`（映射表一致性回归，EXPECTED_OP_COUNT=21 不变）
   - `python op_mapper.py map --op-id tg_config_interface --args '{"interfaces":"${ENV.tg.ports[port1,port2]}"}'`（dry-run 命令构建，不执行）
   - TC-EX-03-01..14 内联验证

## 8. 回滚策略

| 场景 | 回滚方式 | 影响 |
|---|---|---|
| resolve_env_refs 实现有缺陷 | git revert ST-EX-03 commit | execute_op 恢复既有签名（无 env_topology/tg_api_server），既有调用方不受影响（向后兼容） |
| _build_exec_env 签名扩展导致调用方不兼容 | 新参数有默认值，既有调用方无需修改 | 无影响（向后兼容设计） |
| ${ENV.*} 解析逻辑错误 | git revert + 修复 | 既有用例（无 ${ENV.*}）不受影响（原样透传） |
| TREX_API_URL 注入失败 | tg_api_server="" 时 _build_exec_env 不注入，ptm-atomic 使用 os.environ 原值 | 退回既有行为（无 TREX_API_URL 注入） |

**回滚安全保证**：所有新增参数均有默认值（None/""），既有调用方（op_mapper.py CLI `_cli_main`、既有 case_runner 调用）无需修改即可工作。git revert 后 op_mapper.py 恢复为既有签名，无破坏性变更。

## 9. 平台差异

| 平台 | 适配点 | 说明 |
|---|---|---|
| ptm-team canonical | op_mapper.py 源 | skills/policy-route-execution/scripts/op_mapper.py 修改源 |
| ptm-te workspace | skill 安装目标 | install.py 安装后 workspace .claude/skills/policy-route-execution/scripts/op_mapper.py 更新（ST-EX-02 install.py 不改安装逻辑） |
| ptm-atomic CLI | 不改本体（R-C-001） | op_mapper -> subprocess -> ptm-atomic；TREX_API_URL 经环境变量传入子进程 |
| DUT（防火墙） | 不受影响 | ST-EX-03 只影响 TG op 的 TREX_API_URL 注入，DUT op 行为不变 |
| TG（trex） | TREX_API_URL 注入 | ptm-atomic 子进程读 TREX_API_URL 环境变量连接 TG api_server |

**跨仓库一致性**（RISK-CR033-CROSS-REPO）：install.py 安装 op_mapper.py 到 workspace 后，workspace 版本与 canonical 一致。ST-EX-03 不新增 op_id（EXPECTED_OP_COUNT=21 不变），`validate_mapping_consistency()` 不受影响。

## 10. 数据模型

### 10.1 env_topology 契约（HLD §12.3）

```yaml
# env_topology 结构（case_runner load_env_file 加载 --env-file 生成）
port_mapping:
  port1:
    tg: "1/1/1"       # TG 物理端口名（必须加引号，Gotcha #11 YAML 1.1 整数陷阱）
    dut: "eth0"       # DUT 物理端口名
  port2:
    tg: "1/1/2"
    dut: "eth1"

nodes:
  tg1:                  # TG 节点（键名 tg1，占位符用 tg 映射）
    trex_api_url: "10.113.52.253:8450"   # ${ENV.tg.url} 解析目标
    interfaces:
      port1:
        ip: "192.168.101.1"              # ${ENV.tg.port1.ip} 解析目标
        gw: "192.168.101.2"              # ${ENV.tg.port1.gw} 解析目标
      port2:
        ip: "192.168.102.1"
        gw: "192.168.102.2"
  dut1:                 # DUT 节点（键名 dut1，占位符用 dut 映射）
    host: "10.113.55.51"                 # ${ENV.dut.url} 解析目标
    next_hop: "192.168.102.1"            # ${ENV.dut.next_hop} 解析目标
    interfaces:
      port1:
        ip: "192.168.101.2"              # ${ENV.dut.port1.ip} 解析目标
      port2:
        ip: "192.168.102.2"

links:
  - from: "tg1.port1"
    to: "dut1.port1"
    name: "link1"
  - from: "tg1.port2"
    to: "dut1.port2"
    name: "link2"
```

**键名映射规则**：
- 占位符 `tg.*` -> nodes 键 `tg1`
- 占位符 `dut.*` -> nodes 键 `dut1`
- `port{N}` -> port_mapping / nodes.tg1.interfaces / nodes.dut1.interfaces 的键

**YAML 1.1 整数陷阱规避**（Gotcha #11）：port_mapping 中物理端口名（如 "1/1/1"）必须加引号。resolve_env_refs 不做类型校验（由 case_runner build_env_topology 校验，ST-EX-17 职责），但解析时若值为非字符串类型（如 int），原样返回（由 validate_args 后续校验）。

### 10.2 args 变换示例

**输入**（用例 case_steps，含 ${ENV.*}）：
```yaml
args:
  tx_port: "${ENV.tg.port1}"
  rx_port: "${ENV.tg.port2}"
  next_hop_ip: "${ENV.dut.next_hop}"
  src_ip: "192.168.1.100"        # 测试意图，字面值
```

**resolve_env_refs 输出**（${ENV.*} 已解析）：
```python
{
  "tx_port": "1/1/1",             # port_mapping.port1.tg
  "rx_port": "1/1/2",             # port_mapping.port2.tg
  "next_hop_ip": "192.168.102.1", # nodes.dut1.next_hop
  "src_ip": "192.168.1.100"       # 字面值原样保留
}
```

### 10.3 不变的数据模型

- `OP_ID_TO_SUBCOMMAND`：不变（不新增 op_id）
- `ARGS_TO_FLAGS`：不变（不新增 flag 映射）
- `REQUIRED_FLAGS`：不变
- `ROLLBACK_STRATEGY`：不变
- `OP_METADATA`：不变
- `EXPECTED_OP_COUNT`：不变（21，ST-EX-07 才改为 22）

## 11. 性能

| 指标 | 现有基线 | ST-EX-03 影响 | 说明 |
|---|---|---|---|
| resolve_env_refs 调用开销 | 0（不存在） | <1ms/step | 字典遍历 + 正则匹配，无 IO |
| _build_exec_env 调用开销 | ~0.1ms | ~0.1ms（新增 1 次赋值） | 可忽略 |
| execute_op 总流程 | 既有 | +<1ms/step（resolve_env_refs） | 可忽略 |
| 24 用例 × 10 step | 既有 | +<240ms 总计 | 可忽略 |

**性能结论**：resolve_env_refs 是纯内存字典操作 + 正则匹配，无网络 IO、无文件 IO，性能影响可忽略。不引入进程启动开销（ADR-01 import 模式）。

## 12. 安全

| 安全维度 | 约束 | ST-EX-03 落实 |
|---|---|---|
| NO_CREDENTIAL_READ | 不读凭据/secret | resolve_env_refs 只解析端口/IP/URL 元数据，不读 password 字段；env_topology 不含凭据 |
| NO_PRODUCTION_WRITE | 不写生产 | ST-EX-03 只修改 op_mapper.py 代码，不执行设备写操作；dry-run 默认门（ADR-02） |
| NO_EXTERNAL_PUBLISH | 不发布 | ST-EX-03 不发布任何内容 |
| ptm-atomic 约束（Gotcha #13） | 框架禁止直接调 TG REST API | resolve_env_refs 只做参数解析，不发起网络请求；TREX_API_URL 经 _build_exec_env 注入子进程环境变量 |
| TREX_API_URL 来源 | env_topology ${ENV.tg.url} 优先，devices.yaml fallback（ADR-05） | op_mapper 只接收 tg_api_server 参数，不读 devices.yaml；fallback 由 case_runner 完成 |
| dry-run 默认门（ADR-02） | dry-run 不执行 mutation op | resolve_env_refs 在 dry-run 模式仍执行（验证解析正确性），但不触发 subprocess 执行 |
| 设计通过 ≠ 运行授权 | --execute 需 runtime_authorization | ST-EX-03 代码通过不代表运行授权；execute_op 既有 authorized 检查不变 |

**安全审查要点**（code review）：
1. resolve_env_refs 无 `import requests` / `import urllib`（Gotcha #13）
2. resolve_env_refs 无 `subprocess.run` / `os.system`（只做参数解析）
3. _build_exec_env 只设置环境变量，不执行命令
4. env_topology 不含 password/secret 字段（只含端口/IP/URL/next_hop）

## 13. 依赖

### 13.1 依赖关系图

```
ST-EX-01（TG 设备建模）
  │
  └─► ST-EX-03（TG 路由参数化）  ← 本 Story
        │
        └─► ST-EX-04（case_runner 核心）  ← 消费 env_topology + tg_api_server
              │
              └─► ST-EX-17（环境解析层 + DUT 接口预配置）  ← load_env_file + build_env_topology
```

### 13.2 上游依赖

| 依赖 | 类型 | 说明 |
|---|---|---|
| ST-EX-01（TG 设备建模） | hard | devices.yaml tg 块含 api_server 字段，case_runner 据此完成 ADR-05 fallback 传入 tg_api_server |

**ST-EX-01 依赖理由**：ST-EX-03 的 tg_api_server 参数由 case_runner 提供，case_runner 的 fallback 逻辑（devices.yaml tg.api_server）依赖 ST-EX-01 定义的 tg 块 schema。ST-EX-03 本身不读 devices.yaml，但 tg_api_server 参数的语义依赖 ST-EX-01 的 tg 块存在。

### 13.3 下游消费者

| 消费者 | 消费接口 | 消费方式 |
|---|---|---|
| ST-EX-04（case_runner 核心） | execute_op(env_topology=..., tg_api_server=...) | case_runner 完成 ADR-05 fallback（env_topology ${ENV.tg.url} 优先，devices.yaml tg.api_server fallback），传入 tg_api_server |
| ST-EX-04（case_runner 核心） | resolve_env_refs(args, env_topology) | case_runner load_env_file 加载 --env-file 生成 env_topology，传入 execute_op |
| ST-EX-06（用例清理） | handle_rollback(env_topology=..., tg_api_server=...) | 回滚操作透传 env_topology + tg_api_server |
| ST-EX-07（fw_logout） | execute_op(env_topology=..., tg_api_server=...) | fw_logout 登出透传参数（TREX_API_URL 不影响 auth logout，但保持签名一致） |
| ST-EX-13（24 用例整改） | ${ENV.*} 占位符语法 | 24 用例改写为 ${ENV.*} 引用 |
| ST-EX-17（环境解析层） | env_topology 契约 | case_runner build_env_topology 校验 env_topology 完整性，传入 execute_op |

### 13.4 既有代码依赖

| 既有函数 | 依赖方式 | ST-EX-03 是否修改 |
|---|---|---|
| `resolve_step_refs` | execute_op 调用顺序在 resolve_env_refs 后 | 不修改函数体，仅调整 execute_op 中调用顺序 |
| `build_command` | execute_op 调用，resolve_env_refs 在其前 | 不修改 |
| `validate_args` | execute_op 调用，resolve_env_refs 在其前 | 不修改 |
| `_parse_atomic_output` | execute_op 调用 | 不修改 |
| `map_op_id_to_subcommand` | build_command 调用 | 不修改 |
| `map_args_to_flags` | build_command 调用 | 不修改 |

## 14. 开放项

| 问题 ID | 问题 | 状态 | 决策引用 | 阻塞 LLD |
|---|---|---|---|---|
| O-EX-03-01 | ${ENV.tg.ports[port1,port2]} 聚合数组返回纯端口名数组，但 tg_config_interface 的 --interfaces 参数需要 `[{port, ip, gateway}]` JSON 格式；resolve_env_refs 只返回端口名数组，interfaces JSON 构造由谁负责？ | OPEN（已写入 QUESTION-LEDGER LCQ-EX-03-01） | HLD §12.3 占位符表 vs 改写示例注释不一致 | 否（采用占位符表定义：纯端口名数组，interfaces JSON 构造由 case_runner/用例层处理） |
| O-EX-03-02 | HLD §12.2 execute_op 签名只写 env_topology，但 ADR-05 fallback 逻辑（devices.yaml tg.api_server）需 case_runner 完成；execute_op 是否应额外接收 tg_api_server 参数？ | OPEN（已写入 QUESTION-LEDGER LCQ-EX-03-02） | HLD §12.2 vs ADR-05 + HLD v1.2 R1（op_mapper 不读 devices.yaml） | 否（采用推荐方案：execute_op 额外接收 tg_api_server 参数，由 case_runner 完成 fallback） |
| O-02（HLD） | fw_logout op 在 ptm-atomic 安装版是否暴露 | OPEN（安装前验证） | ADR-04 / DQ-02 | 否（ST-EX-07 职责，不影响 ST-EX-03） |
| O-04（HLD） | 环境文件 schema 管理 skill/CLI（后续 CR 候选） | OPEN（后续 CR 候选） | ADR-08/09 | 否（CR-033 暂用 topology yaml 作载体） |

### 开放项说明

**O-EX-03-01（${ENV.tg.ports} 返回格式）**：HLD §12.3 占位符表定义 `${ENV.tg.ports[port1,port2]} -> [port_mapping.port1.tg, port_mapping.port2.tg]`（纯端口名数组），但改写示例注释说"框架自动构造 port/ip/gateway 数组"。本 LLD 采用占位符表定义（纯端口名数组），resolve_env_refs 只负责占位符解析。tg_config_interface 的 interfaces JSON 构造（含 port/ip/gateway）由 case_runner 更高层逻辑或用例显式构造处理（ST-EX-04/17 职责）。已写入 QUESTION-LEDGER LCQ-EX-03-01 供 host-orchestrator 确认。

**O-EX-03-02（execute_op tg_api_server 参数传递）**：HLD §12.2 execute_op 签名只写 env_topology，但 _build_exec_env 需要接收 tg_api_server。由于 op_mapper 不读 devices.yaml（HLD v1.2 R1），ADR-05 fallback 逻辑（env_topology ${ENV.tg.url} 优先，devices.yaml tg.api_server fallback）需在 case_runner 层完成。本 LLD 采用方案 B：execute_op 额外接收 tg_api_server 参数（由 case_runner 完成 fallback 后传入），env_topology 仅用于 resolve_env_refs。已写入 QUESTION-LEDGER LCQ-EX-03-02 供 host-orchestrator 确认。

## 与 HLD/ADR 追溯一致性

| HLD/ADR 条目 | 本 LLD 落实点 | 一致性 |
|---|---|---|
| HLD §12.2 execute_op 流程 | §4.1 核心流程（6 步，resolve_env_refs 在前） | 一致 |
| HLD §12.2 执行顺序 | §4.1 resolve_env_refs -> resolve_step_refs -> validate_args -> build_command | 一致 |
| HLD §12.3 ${ENV.*} 9 类占位符 | §3.2 resolve_env_refs 解析表（9 类） | 一致 |
| HLD §12.3 env_topology 契约 | §10.1 env_topology 结构（port_mapping/nodes/links） | 一致 |
| HLD §12.3 ptm-atomic 约束 | §5 异常处理 + §12 安全（resolve_env_refs 不发网络请求） | 一致 |
| HLD §13.3 dry-run resolve_env_refs 执行 | §5 异常处理（dry-run 仍执行 resolve_env_refs） | 一致 |
| HLD §14 RA-005（tg op 走旧地址） | §3.1 _build_exec_env 注入 TREX_API_URL | 一致 |
| HLD §14 RA-013（${ENV.*} 路径不匹配） | §5 异常处理（VALIDATION_FAILED） | 一致 |
| HLD §14 RA-014（环境文件缺失） | §5 异常处理（env_topology=None 原样透传） | 一致 |
| HLD Gotcha #3（方案 a 定稿） | §3.1 _build_exec_env 签名扩展 | 一致 |
| HLD Gotcha #11（YAML 1.1 整数陷阱） | §10.1 env_topology 端口名加引号说明 | 一致 |
| HLD Gotcha #12（向后兼容） | §3.2 resolve_env_refs 原样透传 + §5 异常处理 | 一致 |
| HLD Gotcha #13（ptm-atomic 约束） | §12 安全（无 requests/urllib 直接调 TG） | 一致 |
| ADR-05（TREX_API_URL 注入，方案 a） | §3.1 _build_exec_env + §3.3 execute_op tg_api_server 参数 | 一致（O-EX-03-02 记录参数传递方式不一致） |
| ADR-09（环境文件驱动） | §3.2 resolve_env_refs + §4 核心流程 | 一致 |
| SM-EX-12（${ENV.*} 9 类占位符） | §3.2 解析表 + §6.2 TC-EX-03-03..08 | 一致 |
| R-F-018/027/029 | §3.2/§4/§6 | 一致 |

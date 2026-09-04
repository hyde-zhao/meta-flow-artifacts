---
cr_id: "CR-033"
story_id: "STORY-EX-17"
story_name: "环境解析层 + DUT 接口预配置（env_topology 契约 + ${ENV.*} 解析集成 + DUT 接口自动预配置）"
story_slug: env-resolution-and-dut-preconfig
lld_policy: full-lld
evidence_type: full-lld
design_evidence_type: full-lld
lld_policy_required_level: full-lld
version: "1.0"
created_at: "2026-07-28T16:20:00+08:00"
author: "meta-dev"
hld_ref: "docs/design/HLD-PTM-TE-EXEC.md v1.6 §12.2 execute_op 流程 / §12.3 环境解析层 / §14 RA-013/014 / Gotcha #11/#12/#13"
adr_ref: "docs/design/ARCHITECTURE-DECISION-PTM-TE-EXEC.md ADR-09 环境文件驱动 / ADR-05 TREX_API_URL 环境文件优先 / ADR-08 topology yaml 用途扩展"
development_plan_ref: "process/DEVELOPMENT-PLAN.yaml ST-EX-17"
feature_design_ref: "docs/features/case-execution/DESIGN.md（FE-EX-02，CP3 后生成）"
tier: "M"
wave: "W3"
dependencies: ["ST-EX-03", "ST-EX-04"]
dependency_type: "hard"
file_owners:
  - "skills/case-execution/scripts/case_runner.py#build_env_topology"
  - "skills/case-execution/scripts/case_runner.py#preconfigure_dut_interfaces"
shared_fragments: []
open_items: ["LCQ-ST-EX-17-01", "LCQ-ST-EX-17-02"]
status: "draft"
---

# LLD - ST-EX-17 环境解析层 + DUT 接口预配置（CR-033）

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-28 | meta-dev | CR-033 ST-EX-17 full-lld 初稿：env_topology 契约 + load_env_file/build_env_topology + ${ENV.*} 9 类占位符集成 + preconfigure_dut_interfaces + 双层清理顺序 + 14 章节完整 |

## ADR / HLD 追溯

| 来源 | 章节 | 决策/约束 | 本 LLD 落点 |
|---|---|---|---|
| ADR-09 | 环境文件驱动 | resolve_env_refs + ${ENV.*} + --env-file；用例与环境解耦 | §3 接口设计 / §4 核心流程 / §10 数据模型 |
| ADR-05 | TREX_API_URL 环境文件优先 | ${ENV.tg.url} > devices.yaml tg.api_server fallback | §4 核心流程 / §11 性能 |
| ADR-08 | topology yaml 用途扩展 | env-file 载体 port_mapping/nodes/links | §10 数据模型 |
| HLD §12.2 | execute_op 流程 | resolve_env_refs -> resolve_step_refs -> validate_args -> build_command | §4 核心流程 |
| HLD §12.3 | 环境解析层 | env_topology 契约 + ${ENV.*} 9 类 + DUT 预配置 + 清理顺序 + ptm-atomic 约束 | §3/§4/§10 全文 |
| HLD §12.1[2] | 设备准备 + 环境加载 | load_env_file + build_env_topology + preconfigure_dut_interfaces | §4 核心流程 |
| HLD §14 RA-013 | ${ENV.*} 路径不匹配 | dry-run 预校验 + VALIDATION_FAILED | §5 异常处理 |
| HLD §14 RA-014 | 环境文件缺失/字段不完整 | 启动校验 DA-006/007 + fallback/报错 | §5 异常处理 |
| Gotcha #11 | YAML1.1 整数陷阱 | 端口名加引号 + 字符串类型校验 | §5 异常处理 / §10 数据模型 |
| Gotcha #12 | 向后兼容 | 无 ${ENV.*} 字面值原样透传 | §4 核心流程 |
| Gotcha #13 | ptm-atomic 约束 | 框架禁止直接调 TG REST API | §12 安全 |
| SM-EX-12 | ${ENV.*} 占位符 9 类 | resolve_env_refs 支持 9 类 | §3 接口设计 |
| R-F-027/028/029 | 环境文件驱动需求 | env-file + DUT 预配置 + 跨环境 | §1 工程依据 |

---

## 0. 概述

本 LLD 是 ST-EX-17 环境解析层 + DUT 接口预配置（env_topology 契约 + ${ENV.*} 解析集成 + DUT 接口自动预配置）的 full-lld 设计证据，属于 FE-EX-02 case-execution 引擎 Feature 的 Wave 3 环境驱动 Story，承载 case_runner.py 环境文件加载、env_topology 校验与 DUT 接口自动预配置。

**需求溯源**：ST-EX-17 承载 R-F-027（--env-file 加载 + env_topology 契约）、R-F-028（DUT 接口自动预配置 fw_update_interface）、R-F-029（一次编写多环境执行）等功能需求；详细需求映射见 §1.1 需求来源。

**模块拆分**：case_runner.py 按职责拆分为 build_env_topology（env_topology 构建与校验）/preconfigure_dut_interfaces（DUT 接口预配置 + 逆序还原）/load_env_file（环境文件加载）函数锚点（ST-EX-17 拥有）；不改 op_mapper.py（resolve_env_refs/execute_op/_build_exec_env 由 ST-EX-03 拥有，ST-EX-17 只通过 execute_op 传入 env_topology 触发 resolve_env_refs）。

**代码结构**：详见 §2 文件影响范围。case_runner.py 新增 build_env_topology/preconfigure_dut_interfaces/load_env_file 函数；op_mapper.py 由 ST-EX-03 拥有（ST-EX-17 严格分界，不修改 op_mapper）；不触碰 traffic-skill topology yaml（R-C-005）/24 用例 md（ST-EX-13 拥有）。

**技术细节**：env_topology 契约（§3.2/§10.2，port_mapping/nodes/links，ADR-09）；${ENV.*} 9 类占位符集成（§3.4，SM-EX-12，ST-EX-03 实现 resolve_env_refs，ST-EX-17 构造 env_topology 传入）；DUT 接口预配置（§3.3，--execute 模式按 nodes.dut1.interfaces 调 fw_update_interface）；双层清理顺序（§4.3，先 ST-EX-06 用例 ops 后 ST-EX-17 框架接口）；TREX_API_URL 环境文件优先（§4.2，ADR-05，${ENV.tg.url} 优先 devices.yaml fallback）；YAML 1.1 整数陷阱（§5.1，Gotcha #11）；向后兼容（§4.4，Gotcha #12）；RA-013/014 风险缓解（§5，${ENV.*} 路径不匹配报 VALIDATION_FAILED + 环境文件缺失启动校验）等核心技术细节在各章节展开。

**Definition of Done (DoD)**：CP6 实现完成以 §6 测试设计的 10 单元测试（UT-17-01..10）+ dry-run 9 类占位符解析 + 跨环境 + 向后兼容全通过为准；CP7 验证采用 static-only + dry-run-only（DQ-01 推荐），--execute 真机 DUT 预配置需独立 runtime_authorization。DoD 详见 §6.2/§6.3/§6.4/§6.5 测试分层与跨环境测试。

## 1. 工程依据

### 1.1 需求来源

ST-EX-17 承载 CR-033 CP3 评审范围扩展核心：环境文件驱动（HLD §16.1 追溯表：环境文件驱动 -> ST-EX-03/04/17/13）。

- 关联需求：R-F-027（--env-file 加载 + env_topology 契约）、R-F-028（DUT 接口自动预配置 fw_update_interface）、R-F-029（一次编写多环境执行）
- 关联场景：SCN-EX-26/27/28/29（多环境执行 + env-file 解析 + DUT 预配置）
- 关联 UC：UC-EX-11（多环境执行）

### 1.2 问题陈述

exec_v4.py 时代用例与环境耦合：端口/IP/next_hop/URL 硬编码在 case_steps 中。换环境（如 link3 -> link4，端口 2_3/2_4 -> 2_1/2_2）需改用例 md，不可复用。DUT 接口需人工预配置，框架不接管。

### 1.3 设计目标

- 环境文件驱动：--env-file 加载 port_mapping/nodes/links，解析为 env_topology
- ${ENV.*} 9 类占位符集成：用例用占位符引用环境参数，框架在 execute_op 前解析（resolve_env_refs，ST-EX-03 实现，ST-EX-17 集成调用）
- DUT 接口自动预配置：--execute 模式按 nodes.dut1.interfaces 调 fw_update_interface
- 双层清理顺序：先 ST-EX-06 用例 mutation ops 逆序清理，再 ST-EX-17 框架预配置接口逆序还原
- 一次编写多环境执行：换环境只换 --env-file，不改用例

### 1.4 量化验收

| 验收项 | 目标值 | 验证方式 |
|---|---|---|
| ${ENV.*} 占位符 9 类（SM-EX-12） | 9 类 | dry-run 解析全部 9 类占位符 |
| env-file 完整性校验（DA-006/007） | port_mapping/nodes/links 齐全 | 缺字段报 VALIDATION_FAILED |
| DUT 接口预配置 | nodes.dut1.interfaces 全配置 | dry-run 构造命令校验 |
| 跨环境执行 | 同一用例换 env-file 可执行 | dry-run link3/link4 同用例 |
| 向后兼容（Gotcha #12） | 无 ${ENV.*} 字面值原样透传 | 既有未改写用例不报错 |

### 1.5 非目标

- 不实现 resolve_env_refs 函数本体（ST-EX-03 在 op_mapper.py 实现，ST-EX-17 集成调用）
- 不实现 schema 管理 skill/CLI（O-04，后续 CR 候选）
- 不改 traffic-skill topology yaml（R-C-005，case_runner 只读）
- 不直接调 TG REST API（Gotcha #13，TG 操作经 ptm-atomic）
- 不为 devices.yaml 引入 pydantic（R-C-003）

---

## 2. 文件影响范围

### 2.1 文件所有权（DEVELOPMENT-PLAN ST-EX-17）

| 文件 | 符号 | 操作 | 说明 |
|---|---|---|---|
| skills/case-execution/scripts/case_runner.py | build_env_topology | 新增 | env_topology 构建与校验 |
| skills/case-execution/scripts/case_runner.py | preconfigure_dut_interfaces | 新增 | DUT 接口预配置 + 逆序还原 |

### 2.2 关联但不拥有的文件（ST-EX-03 拥有，ST-EX-17 依赖）

| 文件 | 符号 | 来源 Story | ST-EX-17 关系 |
|---|---|---|---|
| op_mapper.py | resolve_env_refs | ST-EX-03 | ST-EX-17 构造 env_topology 传入 execute_op，触发 resolve_env_refs |
| op_mapper.py | execute_op（env_topology 参数） | ST-EX-03 | ST-EX-17 调用时传 env_topology |
| op_mapper.py | _build_exec_env（tg_api_server 参数） | ST-EX-03 | ST-EX-17 提供 ${ENV.tg.url} 优先的 tg_api_server |

### 2.3 只读依赖

| 文件 | 来源 Story | 用途 |
|---|---|---|
| case_runner.py run 主流程 | ST-EX-04 | load_env_file 在启动校验后调用 |
| op_mapper.py execute_op | ST-EX-03/04 | 传 env_topology 参数 |
| devices.yaml tg.api_server | ST-EX-01 | TREX_API_URL fallback（ADR-05） |
| topology yaml / env-file | 用户提供 | port_mapping/nodes/links 环境契约 |

### 2.4 不触碰文件

- op_mapper.py（ST-EX-03 拥有 resolve_env_refs/execute_op/_build_exec_env 签名扩展）
- traffic-skill topology yaml（R-C-005）
- 24 用例 md（ST-EX-13 拥有 ${ENV.*} 改写）

---

## 3. 接口设计

### 3.1 load_env_file 函数签名

```python
def load_env_file(env_file_path: str) -> dict:
    """加载 --env-file（YAML）为原始 dict（HLD §12.1[2]）。

    CR-033 暂用 topology yaml（skills/traffic-skill/configs/，ADR-08）作环境文件载体，
    case_runner 只读不写。schema 管理 skill/CLI 是后续 CR 候选（O-04）。

    Args:
        env_file_path: --env-file 指定的 YAML 文件路径

    Returns:
        env_file 原始 dict（含 port_mapping/nodes/links 等字段，未经校验）

    Raises:
        FileNotFoundError: 文件不存在（RA-014）
        yaml.YAMLError: YAML 解析失败
    """
```

### 3.2 build_env_topology 函数签名

```python
def build_env_topology(env_file_dict: dict) -> dict:
    """校验 env-file 完整性并构建 env_topology（HLD §12.3，ADR-09）。

    校验维度（DA-006/007）：
    1. 顶层含 port_mapping / nodes / links 三个 key
    2. port_mapping 覆盖用例引用的全部逻辑端口（port1/port2）
       - 每个逻辑端口含 tg/dut 物理映射
       - 物理端口名为字符串类型（Gotcha #11 YAML1.1 整数陷阱）
    3. nodes 含 tg1（trex_api_url + interfaces）和 dut1（host + next_hop + interfaces）
    4. links 非空（组网拓扑）

    Args:
        env_file_dict: load_env_file 返回的原始 dict

    Returns:
        env_topology dict（结构见 §10.2）

    Raises:
        EnvTopologyError: 字段缺失/类型错误/端口名非字符串（RA-013/014）
    """
```

### 3.3 preconfigure_dut_interfaces 函数签名

```python
def preconfigure_dut_interfaces(
    env_topology: dict,
    base_url: str,
    session_file: str,
    *,
    authorized: bool = False,
    dry_run: bool = True,
    timeout: int = 45,
    username: str = "admin",
    password_env: str = "FW_WEB_PASSWORD",
    step_refs_dir: Optional[str] = None,
    env_topology_for_resolve: Optional[dict] = None,
) -> Tuple[List[dict], List[dict]]:
    """--execute 模式按 nodes.dut1.interfaces 调 fw_update_interface 预配置 DUT 接口（R-F-028）。

    dry-run 模式只构建命令不执行（与 ST-EX-05 dry-run 门一致）。

    清理配对：本函数返回 (preconfig_records, cleanup_plan)：
    - preconfig_records: 每个接口的预配置 envelope 列表（含 pre_snapshot 供逆序还原）
    - cleanup_plan: 逆序还原计划，每个条目含 {op_id, args, pre_snapshot, interface_name}
      供 run_cleanup 在 ST-EX-06 mutation ops 清理后执行

    双层清理顺序（HLD §12.3）：
    1. 先执行 ST-EX-06 用例 case_steps mutation ops 逆序清理（run_cleanup）
    2. 再执行 ST-EX-17 框架预配置接口逆序还原（本函数返回的 cleanup_plan 逆序）

    Args:
        env_topology: build_env_topology 返回的 env_topology
        base_url: DUT Web 管理地址（${ENV.dut.url} 或 devices.yaml firewall.host）
        session_file: session 文件路径
        authorized: --execute 授权标记
        dry_run: dry-run 模式（只构建命令）
        timeout: DUT op 超时（45s，Gotcha #9）
        username/password_env: STATE_INVALID 重连用
        step_refs_dir: step-refs 目录
        env_topology_for_resolve: 传给 execute_op 的 env_topology（resolve_env_refs 用）

    Returns:
        (preconfig_records, cleanup_plan)
    """
```

### 3.4 ${ENV.*} 9 类占位符解析规则（SM-EX-12）

ST-EX-17 构造 env_topology，resolve_env_refs（ST-EX-03 实现）按以下规则解析。ST-EX-17 负责保证 env_topology 结构满足解析需求。

| 占位符 | 解析目标（env_topology 路径） | 说明 |
|---|---|---|
| ${ENV.tg.port1} | port_mapping.port1.tg | TG 物理端口名 |
| ${ENV.tg.port1.ip} | nodes.tg1.interfaces.port1.ip | TG 端口 IP |
| ${ENV.tg.port1.gw} | nodes.tg1.interfaces.port1.gw | TG 端口网关 |
| ${ENV.dut.port1} | port_mapping.port1.dut | DUT 物理端口名 |
| ${ENV.dut.port1.ip} | nodes.dut1.interfaces.port1.ip | DUT 端口 IP |
| ${ENV.tg.url} | nodes.tg1.trex_api_url | TG api_server URL（TREX_API_URL 来源，ADR-05） |
| ${ENV.dut.url} | nodes.dut1.host | DUT URL |
| ${ENV.dut.next_hop} | nodes.dut1.next_hop | DUT 下一跳 IP |
| ${ENV.tg.ports[port1,port2]} | [port_mapping.port1.tg, port_mapping.port2.tg] | 聚合数组 |

### 3.5 调用契约

| 调用方 | 调用时机 | 输入 | 输出 | 后续衔接 |
|---|---|---|---|---|
| case_runner.run 启动校验后 | [2] 设备准备 + 环境加载 | --env-file 路径 | env_topology | 传给 preconfigure_dut_interfaces + 每个 execute_op |
| case_runner.run 启动校验后 | [2] DUT 接口预配置（--execute） | env_topology + base_url + session | (preconfig_records, cleanup_plan) | cleanup_plan 存入 run 级状态，用例后清理 |
| case_runner.run_case 逐 step | execute_op 调用 | env_topology 作为参数 | resolve_env_refs 在 execute_op 内解析 | ST-EX-03 实现 |
| case_runner.run 用例后清理 | 双层清理 | cleanup_plan | 逆序还原 | ST-EX-06 mutation ops 清理后 |

---

## 4. 核心流程

### 4.1 环境加载与预配置流程（HLD §12.1[2]）

```
case_runner.run --env-file <path> --execute --authorized
  │
  ├─ [1] 启动校验
  │   ├─ which ptm-atomic（DA-001）
  │   ├─ devices.yaml 含 firewall.host 和 tg.api_server（DA-003，fallback 用）
  │   └─ --env-file 存在（DA-006，若用例含 ${ENV.*}）
  │
  ├─ [2] 环境加载
  │   ├─ env_file_dict = load_env_file(--env-file)
  │   ├─ env_topology = build_env_topology(env_file_dict)
  │   │   ├─ 校验 port_mapping/nodes/links 完整（DA-006/007）
  │   │   ├─ 校验 port_mapping 覆盖 port1/port2（DA-007）
  │   │   ├─ 校验端口名为字符串类型（Gotcha #11）
  │   │   └─ 失败 -> EnvTopologyError，run 终止（RA-014）
  │   ├─ TREX_API_URL 源确定：
  │   │   ├─ 优先 env_topology.nodes.tg1.trex_api_url（${ENV.tg.url}，ADR-05）
  │   │   └─ fallback devices.yaml tg.api_server（env-file 缺失时）
  │   └─ base_url = env_topology.nodes.dut1.host（${ENV.dut.url}）或 devices.yaml firewall.host
  │
  ├─ [3] 预登录（--execute 模式，ST-EX-04/07）
  │   └─ op_mapper.execute_op(fw_login_web_management, env_topology=env_topology)
  │
  ├─ [4] DUT 接口预配置（--execute 模式，R-F-028）
  │   ├─ preconfig_records, cleanup_plan = preconfigure_dut_interfaces(
  │   │       env_topology, base_url, session_file, authorized=True, dry_run=False,
  │   │       env_topology_for_resolve=env_topology)
  │   ├─ 遍历 nodes.dut1.interfaces：
  │   │   └─ 对每个 interface（如 port1: {ip:"192.168.1.1"}）：
  │   │       ├─ 捕获 pre_snapshot（接口原值，供逆序还原）
  │   │       ├─ execute_op(fw_update_interface, args={name: ${ENV.dut.port1},
  │   │       │     ip_address: nodes.dut1.interfaces.port1.ip, ...},
  │   │       │     env_topology=env_topology)
  │   │       └─ 记录 preconfig_record + cleanup_plan 条目
  │   └─ cleanup_plan 存入 run 级状态
  │
  ├─ [5] 逐用例执行（ST-EX-04 主循环）
  │   └─ 每个 step 的 execute_op 传 env_topology=env_topology
  │       └─ resolve_env_refs 在 execute_op 内执行（ST-EX-03）：
  │           resolve_env_refs -> resolve_step_refs -> validate_args -> build_command
  │
  ├─ [6] 双层清理（用例后 + run 后）
  │   ├─ [6a] ST-EX-06 用例 mutation ops 逆序清理（run_cleanup）
  │   │   └─ 用例 case_steps 的 mutation ops 逆序 handle_rollback
  │   └─ [6b] ST-EX-17 框架预配置接口逆序还原（cleanup_plan 逆序）
  │       └─ 对 cleanup_plan 逆序遍历：
  │           ├─ 有 pre_snapshot：handle_rollback(fw_update_interface, restore_snapshot)
  │           └─ 无 pre_snapshot（接口原本不存在）：execute_op(fw_delete_interface, ...)
  │
  └─ [7] fw_logout + report（ST-EX-07/11）
```

### 4.2 resolve_env_refs 集成时序（ST-EX-03 实现，ST-EX-17 集成）

```
case_runner 传 env_topology 给 execute_op
  │
  op_mapper.execute_op(op_id, args, ..., env_topology=env_topology)
  │
  ├─ [1] resolve_env_refs(args, env_topology)        # ST-EX-03 新增
  │   ├─ 扫描 args 中 ${ENV.*} 占位符
  │   ├─ 按 §3.4 规则表解析替换
  │   ├─ 无 ${ENV.*} -> 原样返回（Gotcha #12 向后兼容）
  │   └─ 解析失败 -> VALIDATION_FAILED envelope（RA-013）
  │
  ├─ [2] resolve_step_refs(args, step_refs_dir)      # 既有
  │   └─ 插值 ${STEP-N.id}
  │
  ├─ [3] validate_args(op_id, args)                  # 既有
  │   └─ resolve 后校验参数完整性
  │
  ├─ [4] build_command(op_id, args, ...)             # 既有
  │
  └─ [5] _build_exec_env(base_url, tg_api_server)    # ST-EX-03 扩展签名
      └─ tg_api_server = env_topology.nodes.tg1.trex_api_url
         （${ENV.tg.url} 优先，devices.yaml fallback，ADR-05）
```

**执行顺序**：resolve_env_refs -> resolve_step_refs -> validate_args -> build_command（ADR-09）。未含 ${ENV.*} 字面值原样透传（Gotcha #12）。

### 4.3 双层清理顺序（HLD §12.3 关键约束）

```
用例执行完毕
  │
  ├─ [第一层] ST-EX-06 run_cleanup：用例 case_steps mutation ops 逆序清理
  │   └─ 例：STEP-005 fw_config_policy_route -> handle_rollback(fw_delete_policy_route)
  │      STEP-003 fw_config_object -> handle_rollback(fw_delete_object)
  │      （用例写的 op 先回滚）
  │
  └─ [第二层] ST-EX-17 cleanup_plan 逆序还原：框架预配置接口
      └─ 例：port2 interface -> 还原（pre_snapshot 或 delete）
         port1 interface -> 还原（pre_snapshot 或 delete）
         （框架预配置的接口后还原）
```

**顺序保证**：用例写的 mutation op（如 config policy_route 引用预配置接口）必须先回滚，否则删除预配置接口时会被引用阻止（eBeingReferenced）。先清理用例 op 解除引用，再还原预配置接口，避免引用冲突。

### 4.4 向后兼容流程（Gotcha #12）

```
用例 case_steps 不含 ${ENV.*}（既有未改写用例）
  │
  ├─ env_topology=None（未传 --env-file）或 env_topology 存在但用例不引用
  │
  ├─ execute_op(env_topology=None 或 env_topology=<dict>)
  │   └─ resolve_env_refs 扫描 args，无 ${ENV.*} 前缀
  │       └─ 原样返回 args（Gotcha #12）
  │
  └─ 既有用例行为不变（字面值透传）
```

dry-run 模式下 resolve_env_refs 仍执行（验证 ${ENV.*} 解析正确性），仅跳过 mutation op 实际执行（HLD §13.3）。

---

## 5. 异常处理

### 5.1 环境加载异常

| 异常场景 | 检测方式 | 处理行为 | run 结果 |
|---|---|---|---|
| --env-file 不存在 | load_env_file FileNotFoundError | run 终止，报"env-file 不存在"（RA-014） | ERROR |
| --env-file YAML 解析失败 | load_env_file yaml.YAMLError | run 终止，报"YAML 解析失败" | ERROR |
| port_mapping/nodes/links 缺失 | build_env_topology 字段检查 | EnvTopologyError，run 终止（RA-014） | ERROR |
| port_mapping 未覆盖 port1/port2 | build_env_topology DA-007 校验 | EnvTopologyError，报"逻辑端口未覆盖" | ERROR |
| 端口名非字符串（Gotcha #11） | build_env_topology 类型校验 | EnvTopologyError，报"端口名需字符串，检查 YAML 引号" | ERROR |
| nodes.tg1.trex_api_url 缺失 | build_env_topology | 降级 devices.yaml tg.api_server（ADR-05 fallback）；同时记 warning | 继续运行 |
| nodes.dut1.host 缺失 | build_env_topology | 降级 devices.yaml firewall.host；同时记 warning | 继续运行 |

### 5.2 resolve_env_refs 解析异常（ST-EX-03 实现，ST-EX-17 触发）

| 异常场景 | 检测方式 | 处理行为 | step 结果 |
|---|---|---|---|
| ${ENV.*} 占位符无对应 env_topology 键 | resolve_env_refs 路径查找失败 | 返回 VALIDATION_FAILED envelope（RA-013） | ERROR |
| ${ENV.tg.ports[...]} 聚合数组端口不存在 | resolve_env_refs port_mapping 查找 | VALIDATION_FAILED envelope | ERROR |
| env_topology=None 但 args 含 ${ENV.*} | resolve_env_refs 检查 | VALIDATION_FAILED envelope（env_topology 缺失） | ERROR |

### 5.3 DUT 接口预配置异常

| 异常场景 | 检测方式 | 处理行为 | run 结果 |
|---|---|---|---|
| fw_update_interface 执行失败 | envelope.status != success | 记 preconfig_failed，继续下一个接口；run 标记 preconfig_partial | 继续（部分预配置） |
| 接口 pre_snapshot 捕获失败 | verify 查询失败 | cleanup_plan 标记 no_snapshot，逆序还原时改用 delete | 继续 |
| 接口已存在相同配置 | 幂等容错（ST-EX-11） | idempotent_skip，不重复配置 | 继续 |
| preconfigure 在 dry-run 模式 | dry_run=True | 只构建命令不执行，cleanup_plan 仍生成（空 envelope） | 继续 |

### 5.4 清理异常

| 异常场景 | 处理行为 |
|---|---|
| cleanup_plan 逆序还原时接口被引用（eBeingReferenced） | 说明第一层清理未完全解除引用；记 cleanup_blocked，report.md 提示人工清理 |
| fw_delete_interface 失败 | 记 cleanup_failed，继续下一个；report.md 提示残留接口 |
| restore_snapshot 失败 | 记 restore_failed，report.md 提示接口需人工检查 |

---

## 6. 测试设计

### 6.1 测试分层

| 层级 | 范围 | 方式 | 覆盖场景 |
|---|---|---|---|
| 单元测试 | build_env_topology / preconfigure_dut_interfaces 纯函数 | Python assert | §6.2 |
| dry-run 集成 | env-file 解析 + ${ENV.*} + DUT 预配置命令构建 | case_runner dry-run | §6.3 |
| 跨环境测试 | 同一用例换 env-file | dry-run link3/link4 | R-F-029 |

### 6.2 单元测试用例

| 用例 ID | 输入 | 预期 | 覆盖 |
|---|---|---|---|
| UT-17-01 | 合法 env_file_dict（port_mapping/nodes/links 齐全） | 返回 env_topology | 正常构建 |
| UT-17-02 | env_file_dict 缺 port_mapping | EnvTopologyError | RA-014 字段缺失 |
| UT-17-03 | port_mapping.port1 缺 tg | EnvTopologyError | DA-007 端口未覆盖 |
| UT-17-04 | port_mapping.port1.tg = 111（整数，Gotcha #11） | EnvTopologyError "端口名需字符串" | YAML1.1 陷阱 |
| UT-17-05 | port_mapping.port1.tg = "1/1/1"（字符串） | 正常 | 引号规避 |
| UT-17-06 | nodes.tg1 缺 trex_api_url | 降级 warning，继续 | ADR-05 fallback |
| UT-17-07 | dry-run 模式 preconfigure | 只构建命令，cleanup_plan 生成 | dry-run 门 |
| UT-17-08 | cleanup_plan 逆序 | 逆序遍历还原 | 双层清理顺序 |
| UT-17-09 | 有 pre_snapshot 的还原 | handle_rollback restore_snapshot | 接口还原 |
| UT-17-10 | 无 pre_snapshot 的还原 | execute_op fw_delete_interface | 接口删除 |

### 6.3 dry-run 集成测试

构造含 ${ENV.*} 的 mock 用例 + 合法 env-file，dry-run 模式校验：
- resolve_env_refs 解析全部 9 类占位符（SM-EX-12）
- preconfigure_dut_interfaces 构建 fw_update_interface 命令（含 ${ENV.dut.port1} / ip）
- cleanup_plan 生成且逆序
- 双层清理顺序正确（ST-EX-06 ops 在前，ST-EX-17 接口在后）

### 6.4 跨环境测试（R-F-029）

同一用例（含 ${ENV.*}），换 env-file（link3: port1=2_3/port2=2_4 vs link4: port1=2_1/port2=2_2），dry-run 校验解析到不同物理值，命令构建正确。

### 6.5 向后兼容测试（Gotcha #12）

既有未改写用例（字面值端口，无 ${ENV.*}），不传 --env-file，dry-run 校验原样透传不报错。

---

## 7. 实施步骤

| 步骤 | 操作 | 文件 | 验证 |
|---|---|---|---|
| 1 | 实现 load_env_file 函数 | case_runner.py#load_env_file | UT + env-file 加载 |
| 2 | 实现 build_env_topology 函数 + EnvTopologyError | case_runner.py#build_env_topology | UT-17-01..06 |
| 3 | 实现 preconfigure_dut_interfaces 函数 | case_runner.py#preconfigure_dut_interfaces | UT-17-07..10 |
| 4 | run 主流程集成：[2] 环境加载 + [4] DUT 预配置 | case_runner.py#run | dry-run 集成 |
| 5 | run_case 逐 step 传 env_topology 给 execute_op | case_runner.py#run_case | ${ENV.*} 解析 |
| 6 | 双层清理集成：[6a] ST-EX-06 + [6b] cleanup_plan 逆序 | case_runner.py#run | UT-17-08 顺序 |
| 7 | TREX_API_URL 环境文件优先逻辑（与 ST-EX-03 _build_exec_env 协同） | case_runner.py#run | ADR-05 fallback |
| 8 | dry-run 9 类占位符解析 + 跨环境 + 向后兼容 | - | SM-EX-12 / R-F-029 / Gotcha #12 |

### 7.1 完成准则

- load_env_file / build_env_topology / preconfigure_dut_interfaces 实现 + 10 个单元测试通过
- dry-run 9 类 ${ENV.*} 占位符全部解析成功（SM-EX-12）
- 跨环境 dry-run 通过（R-F-029）
- 向后兼容：既有用例不报错（Gotcha #12）
- 双层清理顺序正确
- CP6 实现执行证据闭环

---

## 8. 回滚策略

### 8.1 代码回滚

ST-EX-17 新增 load_env_file / build_env_topology / preconfigure_dut_interfaces + run 集成点。回滚 = 移除函数 + 移除 run 中环境加载和 DUT 预配置调用 + run_case 中 env_topology 参数传 None。回滚后 case_runner 退化为"无环境文件驱动"，用例必须用字面值（exec_v4.py 时代），既有未改写用例不受影响（Gotcha #12）。

### 8.2 ADR-09 回退

ADR-09 回退/切换条件：无 ${ENV.*} 占位符的用例原样透传（向后兼容）；env-file 缺失时降级为 devices.yaml 直读（ADR-05 fallback）。即 ST-EX-17 回退后，不传 --env-file，用例用字面值，行为同 exec_v4.py。

### 8.3 DUT 预配置回退

若 preconfigure_dut_interfaces 导致 DUT 接口状态异常，可禁用预配置（run 中跳过 [4] 步骤），改回人工预配置 DUT 接口。cleanup_plan 不生成，双层清理退化为单层（仅 ST-EX-06）。

### 8.4 清理失败回退

cleanup_plan 逆序还原失败时，report.md 提示人工清理。运行时回滚靠人工：`ptm-atomic run --base-url <dut> interface delete --id <interface_id>` 手动删除残留接口。

---

## 9. 平台差异

| 平台 | 适配点 | 说明 |
|---|---|---|
| ptm-team canonical | skill 源 | build_env_topology/preconfigure_dut_interfaces 源在 case_runner.py |
| ptm-te workspace | skill 安装目标 | install.py 安装到 .claude/skills/case-execution/scripts/ |
| ptm-atomic CLI | 不改本体（R-C-001） | DUT 预配置经 op_mapper -> ptm-atomic run interface update；TG 经 ptm-atomic run tg trex |
| topology yaml / env-file | 不改（R-C-005） | case_runner load_env_file 只读 |
| TG（trex） | 不直接调 REST API（Gotcha #13） | TREX_API_URL 经 _build_exec_env 注入，ptm-atomic 子进程消费 |

无 Claude Code / Codex / Qoder 平台差异。

---

## 10. 数据模型

### 10.1 env-file YAML 结构（载体，ADR-08 topology yaml）

```yaml
# env-file（topology yaml 扩展，含环境契约）
port_mapping:
  port1:
    tg: "1/1/1"      # 必须引号（Gotcha #11）
    dut: "eth0"
  port2:
    tg: "1/1/2"
    dut: "eth1"

nodes:
  tg1:
    trex_api_url: "http://10.113.52.253:8450"
    interfaces:
      port1:
        ip: "192.168.101.2"
        gw: "192.168.101.1"
      port2:
        ip: "192.168.102.2"
        gw: "192.168.102.1"
  dut1:
    host: "https://10.113.55.51"
    next_hop: "192.168.102.1"
    interfaces:
      port1:
        ip: "192.168.101.1"
      port2:
        ip: "192.168.102.1"

links:
  - {from: "tg1.port1", to: "dut1.port1", name: "link1"}
  - {from: "tg1.port2", to: "dut1.port2", name: "link2"}
```

### 10.2 env_topology dict 结构（build_env_topology 输出）

```python
{
    "port_mapping": {
        "port1": {"tg": "1/1/1", "dut": "eth0"},
        "port2": {"tg": "1/1/2", "dut": "eth1"},
    },
    "nodes": {
        "tg1": {
            "trex_api_url": "http://10.113.52.253:8450",
            "interfaces": {
                "port1": {"ip": "192.168.101.2", "gw": "192.168.101.1"},
                "port2": {"ip": "192.168.102.2", "gw": "192.168.102.1"},
            },
        },
        "dut1": {
            "host": "https://10.113.55.51",
            "next_hop": "192.168.102.1",
            "interfaces": {
                "port1": {"ip": "192.168.101.1"},
                "port2": {"ip": "192.168.102.1"},
            },
        },
    },
    "links": [
        {"from": "tg1.port1", "to": "dut1.port1", "name": "link1"},
        {"from": "tg1.port2", "to": "dut1.port2", "name": "link2"},
    ],
}
```

### 10.3 cleanup_plan 条目结构

```python
{
    "interface_name": "eth0",          # ${ENV.dut.port1} 解析值
    "logical_port": "port1",
    "op_id": "fw_update_interface",
    "args": {"name": "eth0", "ip_address": "192.168.101.1", ...},
    "pre_snapshot": {...} | None,      # 接口原值（verify 查询）；None 表示原本不存在
    "preconfig_envelope": {...},       # 预配置返回的 envelope
    "has_snapshot": True | False,      # 决定逆序还原用 restore_snapshot 还是 delete
}
```

### 10.4 ${ENV.*} 解析映射表（ST-EX-03 resolve_env_refs 消费）

见 §3.4。resolve_env_refs 按 env_topology 路径查找替换。

---

## 11. 性能

### 11.1 环境加载开销

- load_env_file：YAML 解析单文件，开销 <10ms
- build_env_topology：内存遍历校验，O(n) n=端口数（典型 2-4），开销可忽略

### 11.2 DUT 接口预配置开销

- preconfigure_dut_interfaces：每个接口一次 fw_update_interface（subprocess），2 个接口约 2 × 45s = 90s（--execute 模式）
- dry-run 模式只构建命令，无 subprocess 开销
- 预配置在全量用例前一次性完成，不按用例重复

### 11.3 resolve_env_refs 开销

- 每次 execute_op 调用 resolve_env_refs 扫描 args，O(n) n=args 字段数（典型 5-10），开销可忽略
- 无 ${ENV.*} 时快速返回（Gotcha #12）

### 11.4 性能指标

HLD §13.1 目标 24 用例 < 5 分钟。DUT 预配置增加 ~90s（--execute），计入总时间。dry-run 模式无 subprocess，性能无忧。该指标需 runtime 授权后验证。

---

## 12. 安全

### 12.1 授权边界

- load_env_file / build_env_topology 是纯解析，不触网
- preconfigure_dut_interfaces 触网（fw_update_interface），复用 ST-EX-04/05 授权门：dry-run 默认不执行，--execute --authorized 才实际配置
- DUT 接口预配置是真实设备写操作（R-F-028），属 runtime_authorization 范围

### 12.2 ptm-atomic 约束（Gotcha #13）

| 约束 | 遵循 |
|---|---|
| TG 操作经 ptm-atomic run tg trex | 是，resolve_env_refs 只解析参数，TREX_API_URL 经 _build_exec_env 注入子进程环境变量 |
| 框架禁止直接调 TG REST API | 是，ST-EX-17 无 requests/urllib 调 TG 代码 |
| DUT 操作经 ptm-atomic run interface | 是，preconfigure 经 op_mapper -> ptm-atomic |

### 12.3 安全约束遵循

| 约束 | 遵循 |
|---|---|
| NO_CREDENTIAL_READ | 是，load_env_file 不读凭据；env-file 不含密码 |
| NO_PRODUCTION_WRITE | 是，DUT 预配置经授权门 + ptm-atomic |
| NO_EXTERNAL_PUBLISH | 是，env_topology 不外发 |
| R-C-001 不改 ptm-atomic | 是 |
| R-C-003 不引入 pydantic | 是，build_env_topology 手写校验 |
| R-C-005 不改 traffic-skill | 是，load_env_file 只读 |

### 12.4 env-file 安全

env-file 含设备 IP/端口/next_hop，不含密码（密码用 password_env 环境变量名）。env-file 可入库（无凭据），但建议放 ptm-te workspace 而非 canonical 仓库（环境特定）。

---

## 13. 依赖

### 13.1 Story 依赖

| 依赖 Story | 类型 | 依赖内容 | 门控 |
|---|---|---|---|
| ST-EX-03 | hard | op_mapper resolve_env_refs + execute_op(env_topology) + _build_exec_env(tg_api_server) | lld_gate=full-lld |
| ST-EX-04 | hard | case_runner.py 主框架 + run + run_case 主循环 | dev_gate=cp5-approved |
| ST-EX-06 | hard | run_cleanup 逆序清理（双层清理第一层） | lld_gate=full-lld |
| ST-EX-01 | soft | devices.yaml tg.api_server（TREX_API_URL fallback） | lld_gate=technical-note |

### 13.2 模块依赖

| 模块 | 依赖方向 | 契约 |
|---|---|---|
| op_mapper.execute_op | case_runner -> op_mapper（import，ADR-01） | env_topology 参数 + resolve_env_refs |
| op_mapper._build_exec_env | case_runner 间接（经 execute_op） | tg_api_server 参数（ADR-05） |
| op_mapper.handle_rollback | case_runner 调用（cleanup_plan 还原） | fw_update_interface restore_snapshot |
| devices.yaml | case_runner 读 | firewall.host + tg.api_server（fallback） |

### 13.3 假设

- DA-006：--env-file 存在且含 port_mapping/nodes/links 完整环境契约（启动校验）
- DA-007：--env-file 的 port_mapping 覆盖用例引用的全部逻辑端口（build_env_topology 校验）
- ST-EX-03 已实现 resolve_env_refs + execute_op env_topology 参数 + _build_exec_env tg_api_server 参数（ST-EX-17 依赖前置）

---

## 14. 开放项

| 问题 ID | 问题 | 状态 | 影响 | 处理 |
|---|---|---|---|---|
| LCQ-ST-EX-17-01 | DUT 接口预配置逆序还原：接口原本不存在时用 fw_delete_interface 删除，但 fw_delete_interface 是否在 ptm-atomic 安装版暴露？（类似 Gotcha #4 fw_logout 问题） | OPEN（已写 QUESTION-LEDGER） | 影响 cleanup_plan 无 pre_snapshot 时的还原路径 | 待 host-orchestrator 批量问用户；推荐方案：安装前验证 fw_delete_interface 暴露；未暴露时 cleanup_plan 无 snapshot 条目标记 manual_cleanup，report.md 提示人工删除 |
| LCQ-ST-EX-17-02 | env-file 与 topology yaml 职责边界：CR-033 暂用 topology yaml 作 env-file 载体，但 traffic-skill topology yaml 原有 schema 是否兼容 port_mapping/nodes/links 契约？需确认 env-file 是复用 topology yaml 还是独立文件 | OPEN（已写 QUESTION-LEDGER） | 影响 load_env_file 读取的字段结构 | 推荐：--env-file 独立文件（含完整 port_mapping/nodes/links），不复用 traffic-skill topology yaml（R-C-005 不改）；若用户希望复用，需 traffic-skill 确认 schema 兼容（超 CR-033 范围） |
| O-04 | 环境文件 schema 管理 skill/CLI | OPEN（后续 CR 候选，不阻塞 LLD） | schema 统一管理 | 后续 CR |
| O-03 | 24 用例 ${ENV.*} 改写完整性 | OPEN（ST-EX-13 整改时同步检查） | 影响跨环境执行覆盖 | ST-EX-13 整改 |

### 14.1 clarification 记录

LCQ-ST-EX-17-01 / LCQ-ST-EX-17-02 已写入 `process/state/QUESTION-LEDGER.ndjson`。LCQ-ST-EX-17-01 不阻塞 LLD（推荐方案：安装前验证 + manual_cleanup 兜底，已在 §5.4 设计）。LCQ-ST-EX-17-02 不阻塞（推荐独立 env-file，已在 §10.1 设计）。待 host-orchestrator 批量问用户确认。

### 14.2 与 ST-EX-03 边界声明

ST-EX-17 与 ST-EX-03 严格分界（避免 file_ownership 冲突）：
- ST-EX-03 拥有：op_mapper.py#resolve_env_refs + #execute_op（env_topology 参数）+ #_build_exec_env（tg_api_server 参数）
- ST-EX-17 拥有：case_runner.py#build_env_topology + #preconfigure_dut_interfaces + #load_env_file
- ST-EX-17 不修改 op_mapper.py，只通过 execute_op 调用传入 env_topology 触发 resolve_env_refs
- ST-EX-03 不修改 case_runner.py，只提供 resolve_env_refs 函数和 execute_op 参数

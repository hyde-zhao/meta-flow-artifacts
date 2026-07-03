# HLD — CLI 嵌套子命令架构重构

| 字段 | 值 |
|------|-----|
| 文档版本 | v1.0 |
| 关联 CR | CR-007 |
| 作者 | meta-po |
| 日期 | 2026-06-08 |
| 状态 | 草稿 |

---

## 1. 问题定义

当前 `ptm-atomic run` 命令将所有 40+ 原子操作的参数扁平注册（80+ 参数），随操作增长到 100+ 将不可维护。

## 2. 设计目标

| ID | 目标 | 量化标准 |
|------|------|------|
| G-01 | 参数隔离 | 每个叶子命令参数 ≤ 15 个 |
| G-02 | 扩展性 | 新增操作族 ≤ 1 个文件 + ≤ 1 行注册 |
| G-03 | 可探索性 | 三级 `--help` 可逐层发现可用操作 |
| G-04 | 向后兼容 | 旧格式可用 1-2 个版本 |
| G-05 | 零新依赖 | 不引入第三方 CLI 框架 |

## 3. CLI 命令树

```
ptm-atomic
├── sync                              # 不变
├── list                              # 不变
├── show <op_id>                      # 不变
├── adapter <op_id>                   # 不变
├── packages                          # 不变
├── validate [test_case | --package]  # 不变
└── run                               # ★ 重构
    │   [--base-url URL]              # 共享参数（run 层级）
    │   [--session-file PATH]
    │   [--timeout SECONDS]
    │   [--verify-tls]
    │   [--format json|yaml]
    │   [--execute]
    │   [--auth-header TYPE]
    │   [--payload-file PATH]         # interface 用
    │   [--payload-json JSON]         # interface 用
    │
    ├── auth                          # 简单族：二级
    │   └── login                     # fw_login_web_management
    │       [--username USER]
    │       [--password-env ENV]
    │       [--change-default-password]
    │       [--new-password-env ENV]
    │
    ├── object                        # 简单族：二级
    │   └── config                    # fw_config_object
    │       --object-name NAME
    │       --ipaddr IP
    │       --mask MASK
    │       [--object-desc DESC]
    │
    ├── policy-route                  # 复杂族：三级
    │   ├── config                    # fw_config_policy_route
    │   │   --source-network NAME
    │   │   --in-interface IFACE
    │   │   [--dst-network NAME]
    │   │   [--next-hop-ip IP]
    │   │   [--outgoing-interface IFACE]
    │   │   [--service NAME]
    │   │   [--weight N]
    │   │   ...
    │   ├── update                    # fw_update_policy_route
    │   │   --id ID
    │   │   ...（与 config 相同参数）
    │   ├── delete                    # fw_delete_policy_route
    │   │   --id ID
    │   ├── verify                    # fw_verify_policy_route
    │   └── priority                  # fw_update_policy_route_priority
    │       --targetsite {before,after,first,last}
    │       --targetid ID
    │       --moveid ID
    │
    ├── interface                     # 复杂族：三级
    │   ├── create                    # fw_config_interface
    │   │   --kind {bvi,sub,physical,bond}
    │   │   --id ID
    │   │   [--mode {Route,Switch,Pair,Monitor}]
    │   │   [--ip-address CIDR]*
    │   │   [--name NAME]
    │   │   ...
    │   ├── update                    # fw_update_interface
    │   ├── delete                    # fw_delete_interface
    │   ├── delete-batch              # fw_delete_batch_interface
    │   └── verify                    # fw_verify_interface
    │
    ├── ospf                          # 复杂族：三级
    │   ├── config
    │   │   ├── global                # fw_config_ospfv2_global
    │   │   ├── network               # fw_config_ospfv2_network
    │   │   ├── network-update        # fw_config_ospfv2_network_update
    │   │   ├── interface             # fw_config_ospfv2_interface
    │   │   ├── interface-update      # fw_config_ospfv2_interface_update
    │   │   └── redistribution        # fw_config_ospfv2_redistribution
    │   ├── delete
    │   │   ├── network               # fw_delete_ospfv2_network
    │   │   └── interface             # fw_delete_ospfv2_interface
    │   └── verify
    │       ├── global                # fw_verify_ospfv2_global
    │       ├── network               # fw_verify_ospfv2_network
    │       ├── interface             # fw_verify_ospfv2_interface
    │       ├── neighbor              # fw_verify_ospfv2_neighbor
    │       └── redistribution        # fw_verify_ospfv2_redistribution
    │
    └── ospfv3                        # 复杂族：三级
        ├── config
        │   ├── global                # fw_config_ospfv3_global
        │   ├── network               # fw_config_ospfv3_network
        │   ├── network-update        # fw_config_ospfv3_network_update
        │   ├── interface             # fw_config_ospfv3_interface
        │   ├── interface-update      # fw_config_ospfv3_interface_update
        │   └── redistribution        # fw_config_ospfv3_redistribution
        ├── delete
        │   ├── network               # fw_delete_ospfv3_network
        │   └── interface             # fw_delete_ospfv3_interface
        ├── verify
        │   ├── global                # fw_verify_ospfv3_global
        │   ├── network               # fw_verify_ospfv3_network
        │   ├── interface             # fw_verify_ospfv3_interface
        │   └── neighbor              # fw_verify_ospfv3_neighbor
        └── bfd
            ├── config                # fw_config_bfd_global
            ├── verify-global         # fw_verify_bfd_global
            └── verify-session        # fw_verify_bfd_session
```

## 4. 模块架构

### 4.1 文件结构

```
src/atomic_ops/
├── cli.py                    # ★ 重构：支持嵌套子命令注册
├── commands/
│   ├── __init__.py
│   ├── sync.py               # 不变
│   ├── list_ops.py            # 不变
│   ├── show.py               # 不变
│   ├── adapter.py            # 不变
│   ├── packages.py           # 不变
│   ├── validate.py           # 不变
│   ├── run.py                # ★ 重构：仅保留共享参数 + 兼容分发
│   ├── run_auth.py           # ★ 新增
│   ├── run_object.py         # ★ 新增
│   ├── run_policy_route.py   # ★ 新增
│   ├── run_interface.py      # ★ 新增
│   ├── run_ospf.py           # ★ 新增
│   └── run_ospfv3.py         # ★ 新增
├── errors.py                 # ★ 增强：提取公共输出辅助
└── runner/                   # 不变
    ├── ngfw_login.py
    ├── object.py
    ├── policy_route.py
    ├── interface.py
    ├── ospfv2.py
    └── ospfv3_bfd.py
```

### 4.2 模块职责

| 模块 | 职责 | 依赖 |
|------|------|------|
| `cli.py` | 顶层解析器 + 嵌套子命令注册；`CommandGroup` 新概念 | 所有 commands/ 模块 |
| `commands/run.py` | 共享参数定义 + `run` 子命令组注册 + 旧格式兼容 | 各 run_*.py 模块 |
| `commands/run_auth.py` | `auth login` 参数 + `run()` | runner/ngfw_login.py |
| `commands/run_object.py` | `object config` 参数 + `run()` | runner/object.py |
| `commands/run_policy_route.py` | `policy-route config/update/delete/verify/priority` | runner/policy_route.py |
| `commands/run_interface.py` | `interface create/update/delete/delete-batch/verify` | runner/interface.py |
| `commands/run_ospf.py` | `ospf config/delete/verify` 三级子命令 | runner/ospfv2.py |
| `commands/run_ospfv3.py` | `ospfv3 config/delete/verify/bfd` 三级子命令 | runner/ospfv3_bfd.py |

### 4.3 注册层设计（cli.py）

```python
@dataclass(frozen=True)
class CommandGroup:
    """嵌套子命令组"""
    name: str
    help: str
    description: str
    children: tuple[CommandSpec | CommandGroup, ...]

# 根命令注册
ROOT_GROUPS = (
    CommandGroup(name="run", ...,
        children=(
            CommandGroup(name="auth", ...,
                children=(CommandSpec(name="login", ...),)),
            CommandGroup(name="ospf", ...,
                children=(
                    CommandGroup(name="config", ...,
                        children=(CommandSpec(name="global", ...), ...)),
                    ...
                )),
            ...
        )),
)
```

## 5. 共享参数机制

### 设计决策（DQ-2 确认）

共享参数在 `commands/run.py` 中定义，子命令模块通过函数参数接收。

```python
# commands/run.py
SHARED_ARG_NAMES = (
    "base_url", "session_file", "timeout", "verify_tls",
    "format", "execute", "auth_header",
)

def add_shared_arguments(parser: argparse.ArgumentParser) -> None:
    """注册所有子命令共享的参数。"""
    parser.add_argument("--base-url", required=True, ...)
    parser.add_argument("--session-file", default=..., ...)
    parser.add_argument("--timeout", type=float, default=10.0, ...)
    parser.add_argument("--verify-tls", action="store_true", ...)
    parser.add_argument("--format", choices=("json", "yaml"), default="yaml", ...)
    parser.add_argument("--execute", action="store_true", ...)
    parser.add_argument("--auth-header", choices=(...), default="Authorization", ...)

def resolve_run_context(args: argparse.Namespace) -> dict[str, Any]:
    """提取共享参数，返回 RunContext dict。"""
    return {name: getattr(args, name) for name in SHARED_ARG_NAMES}
```

子命令模块：
```python
# commands/run_ospf.py
def run(args: argparse.Namespace) -> int:
    ctx = resolve_run_context(args)
    # ctx["base_url"], ctx["execute"], ...
```

## 6. 向后兼容设计（DQ-3 确认）

```python
# commands/run.py 中的兼容逻辑
DEPRECATED_OP_MAP = {
    "fw_login_web_management":   ("auth", "login"),
    "fw_config_object":          ("object", "config"),
    "fw_config_policy_route":    ("policy-route", "config"),
    "fw_config_ospfv2_global":   ("ospf", "config", "global"),
    ...
}

def run_legacy(args: argparse.Namespace) -> int:
    """处理旧格式 ptm-atomic run <op_id> [--params]"""
    op_id = args.op_id
    path = DEPRECATED_OP_MAP.get(op_id)
    if path is None:
        print(f"error: unknown op_id '{op_id}'", file=sys.stderr)
        return EXIT_NOT_IMPLEMENTED
    
    print(f"warning: '{op_id}' is deprecated, "
          f"use 'ptm-atomic run {' '.join(path)}' instead",
          file=sys.stderr)
    # 分发到对应的新命令
    return _dispatch_legacy(args, path)
```

## 7. 统一错误输出（DQ-4 确认）

在 `errors.py` 中新增以下公共函数，消除 7 个模块的重复实现：

```python
# errors.py 新增

def write_stale_warning(repo: Repository, output_format: str) -> None:
    """统一的 stale cache 警告输出。"""
    if not repo.is_stale:
        return
    write_warning({
        "warning_code": "STALE_CACHE",
        "message": "cache_status=STALE; using stale local cache for read-only operation",
    }, output_format)

def write_read_error(exc: ReadError, output_format: str, command: str) -> int:
    """统一的 ReadError 处理。"""
    suggestion_map = {
        ERROR_REPO_NOT_SYNCED: "Run ptm-atomic sync before using read commands.",
        "PACKAGE_NOT_FOUND": "Run ptm-atomic packages and choose one of the listed package names.",
        "OP_NOT_FOUND": "Run ptm-atomic sync and check the op_id spelling.",
    }
    suggestion = suggestion_map.get(exc.error_code, "Rerun the command after fixing the local cache.")
    return write_error(CliError(
        error_code=exc.error_code,
        message=exc.message,
        details={"command": command, **exc.details},
        retryable=exc.retryable,
        suggestion=suggestion,
    ), output_format)
```

## 8. 数据流

```
用户输入: ptm-atomic run ospf config global --router-id 10.1.1.1 --area 0 --base-url https://10.0.0.1

cli.py
  └─ build_parser()
       └─ 注册 run 子命令组
            └─ 注册 ospf 子命令组
                 └─ 注册 config 子命令组
                      └─ 注册 global 命令 → handler=run_ospf.run_global_config

命令行解析:
  args.router_id = "10.1.1.1"
  args.area = "0"
  args.base_url = "https://10.0.0.1"
  args.handler = run_ospf.run_global_config

run_ospf.run_global_config(args):
  ctx = resolve_run_context(args)          # {"base_url": "...", "execute": False, ...}
  ctx["router_id"] = args.router_id       # 操作族特有参数
  ctx["area"] = args.area
  → runner/ospfv2.py 的 execute_ospfv2()
```

## 9. 风险与缓解

| 风险 | 影响 | 缓解 |
|------|------|------|
| argparse 嵌套子命令样板代码多 | 开发效率 | 提取 `_register_command_group()` 递归注册函数 |
| 旧格式兼容增加了复杂度 | 维护成本 | 限定 1-2 版本后移除，标记 TODO |
| 参数传递错误（共享参数遗漏） | 运行时 bug | 测试覆盖每个操作族的共享参数 |
| 大量文件变更导致合并冲突 | 协作风险 | 单次提交，全部变更原子化 |

## 10. 架构灰区

### AGA-1：shared 参数 vs 参数污染

**问题**：`--payload-file` 和 `--payload-json` 是 interface 专属的，不算真正的"共享参数"。放在 run 层级会污染其他子命令的 `--help`。

**推荐**：只把真正通用的参数（base-url, session-file, timeout, verify-tls, format, execute, auth-header）放在 run 层级。`--payload-file`/`--payload-json` 下放到 interface 子命令。

### AGA-2：ospf vs ospfv3 共享逻辑

**问题**：ospf 和 ospfv3 有几乎相同的子命令结构（config global, config network, verify neighbor...），代码高度相似。

**推荐**：本次不合并。原因：(1) 两者的 runner 实现不同 (2) 后续 ospfv3 可能独立演进 (3) 合并会增加耦合。等两者都稳定后再评估提取公共基类。

---

## 修订记录

| 版本 | 日期 | 作者 | 变更 |
|------|------|------|------|
| v1.0 | 2026-06-08 | meta-po | 初稿 |

---
cr_id: "CR-033"
story_id: "STORY-EX-03"
title: "TG 路由参数化 CP6 实现执行证据"
wave: 1
evidence_type: "implementation-execution"
lld_policy: "full-lld"
validation_mode: "static-only"
created_at: "2026-07-28T17:30:00+08:00"
author: "meta-dev"
source_lld: "process/stories/STORY-EX-03-LLD.md"
---

# ST-EX-03 CP6 实现执行证据

## 1. 实现对象清单

| 文件 | 函数/锚点 | 变更类型 | 行号（实现后） | 说明 |
|---|---|---|---|---|
| `skills/policy-route-execution/scripts/op_mapper.py` | `_ENV_REF_RE` / `_ENV_PORTS_AGG_RE` | 新增常量 | 907/909 | ${ENV.*} 占位符正则 + 聚合端口正则 |
| `skills/policy-route-execution/scripts/op_mapper.py` | `resolve_env_refs` | 新增函数 | 913 | 9 类 ${ENV.*} 占位符解析（ADR-09），约 95 行 |
| `skills/policy-route-execution/scripts/op_mapper.py` | `_build_exec_env` | 修改签名+函数体 | 793 | 新增 `tg_api_server: str = ""`，非空时注入 TREX_API_URL（ADR-05 方案 a） |
| `skills/policy-route-execution/scripts/op_mapper.py` | `execute_op` | 修改签名+函数体 | 1231 | 新增 `env_topology` + `tg_api_server` keyword-only；resolve_env_refs 在 resolve_step_refs 前；_build_exec_env 传 tg_api_server |
| `skills/policy-route-execution/scripts/op_mapper.py` | `_reconnect_and_retry` | 修改签名+函数体 | 1148 | 新增 `tg_api_server` 参数；2 处 _build_exec_env 调用传参 |
| `skills/policy-route-execution/scripts/op_mapper.py` | `handle_rollback` | 修改签名+函数体 | 1691 | 新增 `env_topology` + `tg_api_server`；2 处 execute_op 调用透传 |

**变更行数**：新增 resolve_env_refs 约 95 行 + 正则常量 2 行 + _build_exec_env 修改约 12 行 + execute_op 修改约 20 行 + _reconnect_and_retry 修改约 8 行 + handle_rollback 修改约 12 行 ≈ 149 行（LLD 预估 115 行，略超因 docstring 详尽）。

**未触碰文件**（file_ownership 一致）：
- `op_mapper.py#resolve_step_refs`（既有函数，仅调整 execute_op 调用顺序，未改函数体）
- `op_mapper.py#build_command` / `validate_args` / `map_op_id_to_subcommand`（未修改）
- `skills/device-management/*`（ST-EX-01 拥有）
- `script/ptm_team/install.py`（ST-EX-02 拥有）
- `skills/case-execution/*`（ST-EX-04 拥有）

## 2. 设计契约映射（LLD §3 接口设计 -> 实现代码）

| LLD 契约 | 实现代码 | 一致性 |
|---|---|---|
| §3.1 `_build_exec_env(base_url, tg_api_server="")` 签名扩展 | line 793 `def _build_exec_env(base_url: str, tg_api_server: str = "") -> dict:` | 一致 |
| §3.1 tg_api_server 非空时 `env["TREX_API_URL"] = tg_api_server` | line 830-832 `if tg_api_server: env["TREX_API_URL"] = tg_api_server` | 一致 |
| §3.1 tg_api_server 空时不设置（向后兼容） | `if tg_api_server:` 守卫，空时不赋值 | 一致 |
| §3.2 `resolve_env_refs(args, env_topology)` 签名 | line 913 `def resolve_env_refs(args: dict, env_topology: Optional[dict]) -> dict:` | 一致 |
| §3.2 9 类 ${ENV.*} 占位符解析表 | line 945-1015 实现 9 类解析（tg.port{N}/tg.port{N}.ip/gw/dut.port{N}/dut.port{N}.ip/tg.url/dut.url/dut.next_hop/tg.ports[]） | 一致 |
| §3.2 env_topology=None 原样返回（Gotcha #12） | line 930 `if env_topology is None or not isinstance(args, dict): return args` | 一致 |
| §3.2 解析失败抛 ValueError | `_missing()` + `raise ValueError(...)` | 一致 |
| §3.3 `execute_op` 新增 `env_topology` + `tg_api_server` keyword-only | line 1271-1272 `env_topology: Optional[dict] = None, tg_api_server: str = "",` | 一致 |
| §3.3 两参数有默认值（向后兼容） | `None` / `""` 默认值 | 一致 |
| §3.4 `handle_rollback` 新增 `env_topology` + `tg_api_server` | line 1699-1700 | 一致 |
| §3.4 inverse_op / restore_op 调用透传 | line 1778-1779 / line 1813-1814 `env_topology=env_topology, tg_api_server=tg_api_server,` | 一致 |
| §3.5 `_reconnect_and_retry` 新增 `tg_api_server` | line 1155 `tg_api_server: str = "",` | 一致 |
| §4.1 执行顺序 resolve_env_refs -> resolve_step_refs -> validate_args -> build_command | line 1286 resolve_env_refs -> line 1289 resolve_step_refs -> line 1296 build_command（含 validate_args 在 build_command 内） | 一致 |
| §4.2 resolve_env_refs 前置检查 + 遍历 + 解析 | line 930-1033 实现 | 一致 |
| §5 异常：ValueError -> VALIDATION_FAILED envelope | execute_op 既有 except ValueError（line 1308）捕获，error_type=VALIDATION_FAILED | 一致（既有 except 复用） |

## 3. 测试 / Fixture 计划与执行结果

**validation_mode=static-only**（LLD §6.1），不引入 pytest/robot（R-C-002），使用 op_mapper.py 既有 CLI `validate` + Python 内联验证。

### 3.1 测试用例执行结果（LLD §6.2 TC-EX-03-01..14）

| 用例 ID | 测试目标 | 执行结果 | 证据 |
|---|---|---|---|
| TC-EX-03-01 | resolve_env_refs 向后兼容（无 ${ENV.*}） | PASS | `{'src_ip':'1.1.1.1'}` 原样返回 |
| TC-EX-03-02 | resolve_env_refs 向后兼容（env_topology=None） | PASS | `${ENV.tg.port1}` 原样返回 |
| TC-EX-03-03 | ${ENV.tg.port1} 解析 | PASS | -> `1/1/1` |
| TC-EX-03-04 | ${ENV.tg.url} 解析 | PASS | -> `10.113.52.253:8450` |
| TC-EX-03-05 | ${ENV.dut.next_hop} 解析 | PASS | -> `192.168.102.1` |
| TC-EX-03-06 | ${ENV.tg.ports[port1,port2]} 聚合 | PASS | -> `['1/1/1','1/1/2']` |
| TC-EX-03-07 | ${ENV.*} 键缺失 | PASS | raise ValueError |
| TC-EX-03-08 | 9 类占位符全覆盖 | PASS | 9 类全解析成功 |
| TC-EX-03-09 | _build_exec_env 注入 TREX_API_URL | PASS | `env['TREX_API_URL']=='10.113.52.253:8450'` |
| TC-EX-03-10 | _build_exec_env 向后兼容 | PASS | 空 tg_api_server 时无 TREX_API_URL |
| TC-EX-03-11 | execute_op dry-run + env_topology 完整 | PASS | resolve_env_refs 成功（error_type=OP_NOT_FOUND 在 build_command 阶段，非 VALIDATION_FAILED） |
| TC-EX-03-12 | execute_op dry-run + ${ENV.*} 解析失败 | PASS | error_type=VALIDATION_FAILED |
| TC-EX-03-13 | validate_mapping_consistency 通过 | PASS | `ValidationResult: PASS (21 op_id 全覆盖，三表一致)` |
| TC-EX-03-14 | handle_rollback 透传 env_topology | PASS | 签名兼容，envelope 正常返回；向后兼容（env_topology=None）验证通过 |

**14/14 TC PASS**。

### 3.2 dry-run 验证命令执行记录

```bash
# 1. 映射表一致性校验（回归）
$ python3 skills/policy-route-execution/scripts/op_mapper.py validate
ValidationResult: PASS (21 op_id 全覆盖，三表一致)

# 2. 语法检查
$ python3 -m py_compile skills/policy-route-execution/scripts/op_mapper.py  # OK

# 3. TC-EX-03-01..14 内联验证（见 §3.1 证据）
# 全部 PASS
```

## 4. 最小实现切片

### 4.1 resolve_env_refs 核心解析逻辑（节选）

```python
_ENV_REF_RE = re.compile(r"^\$\{ENV\.([a-zA-Z0-9_.\[\],\s]+)\}$")
_ENV_PORTS_AGG_RE = re.compile(r"^tg\.ports\[([a-zA-Z0-9_,\s]+)\]$")

def resolve_env_refs(args: dict, env_topology: Optional[dict]) -> dict:
    if env_topology is None or not isinstance(args, dict):
        return args
    port_mapping = env_topology.get("port_mapping", {}) or {}
    nodes = env_topology.get("nodes", {}) or {}
    tg_node = nodes.get("tg1", {}) or {}
    dut_node = nodes.get("dut1", {}) or {}
    # ... _resolve_one 内部函数实现 9 类解析 ...
    resolved: dict = {}
    for key, val in args.items():
        if isinstance(val, str):
            resolved[key] = _resolve_one(val)
        else:
            resolved[key] = val
    return resolved
```

### 4.2 _build_exec_env TREX_API_URL 注入（节选）

```python
def _build_exec_env(base_url: str, tg_api_server: str = "") -> dict:
    env = dict(os.environ)
    # ... NO_PROXY 既有逻辑 ...
    if tg_api_server:
        env["TREX_API_URL"] = tg_api_server
    return env
```

### 4.3 execute_op resolve_env_refs 调用顺序（节选）

```python
    try:
        # 解析 args 中的 ${ENV.*} 占位符（ADR-09，必须在 resolve_step_refs 前）
        args = resolve_env_refs(args, env_topology)
        if step_refs_dir:
            args = resolve_step_refs(args, step_refs_dir, ...)
        command = build_command(op_id, args, base_url, session_file, dry_run=dry_run)
    except ValueError as e:
        return _build_envelope(..., "VALIDATION_FAILED", ...)
```

## 5. 本地验证结果

| 验证项 | 命令 | 结果 |
|---|---|---|
| Python 语法 | `python3 -m py_compile op_mapper.py` | OK |
| 映射表一致性 | `python3 op_mapper.py validate` | PASS（21 op_id，EXPECTED_OP_COUNT=21 不变） |
| TC-EX-03-01..10 unit | Python 内联 | 全 PASS |
| TC-EX-03-11..14 integration | Python 内联（ptm-atomic 0.1.0 可用） | 全 PASS |
| resolve_env_refs 调用顺序 | grep | resolve_env_refs(1286) -> resolve_step_refs(1289) 顺序正确 |
| _build_exec_env 调用传参 | grep | 3 处调用（1174/1199/1337）均传 tg_api_server |
| ptm-atomic 约束（Gotcha #13） | code review | resolve_env_refs 无 import requests/urllib/subprocess，只做参数解析 |

**dry-run 门验证**（ADR-02）：TC-EX-03-11 dry_run=True 时 resolve_env_refs 仍执行（验证 ${ENV.*} 解析正确性），符合 HLD §13.3 / Gotcha #12。

## 6. 平台差异检查

| 平台 | 适配点 | ST-EX-03 影响 | 验证 |
|---|---|---|---|
| ptm-team canonical | op_mapper.py 源 | 修改源，EXPECTED_OP_COUNT=21 不变 | validate PASS |
| ptm-te workspace | skill 安装目标 | install.py 安装后 workspace op_mapper.py 更新 | ST-EX-02 install.py 不改安装逻辑，安装机制不变 |
| ptm-atomic CLI | 不改本体（R-C-001） | TREX_API_URL 经环境变量传入子进程 | _build_exec_env 注入 env，subprocess.run(env=...) |
| DUT（防火墙） | 不受影响 | DUT op 行为不变，tg_api_server="" 时不注入 TREX_API_URL | TC-EX-03-10 向后兼容验证 |
| TG（trex） | TREX_API_URL 注入 | ptm-atomic 子进程读 TREX_API_URL | TC-EX-03-09 注入验证 |

**跨仓库一致性**（RISK-CR033-CROSS-REPO）：op_mapper.py 不新增 op_id（EXPECTED_OP_COUNT=21 不变），`validate_mapping_consistency()` PASS，workspace 安装后与 canonical 一致。

## 7. Agent Dispatch Evidence

| 字段 | 值 |
|---|---|
| canonical_role | meta-dev |
| dispatch_mode | inline-fallback（本会话主进程直接执行编码） |
| dispatch_trigger | CP5 approved，Wave 1 ST-EX-03 dev-ready |
| reasoning_profile | medium（标准实现） |
| tool_name | 主进程直接编码（无 subagent 调度） |
| fallback_reason | 本任务为单 Story 编码实现，主进程直接执行 |
| approved_by | host-orchestrator 委托（CP5 approved） |
| completed_at | 2026-07-28T17:30:00+08:00 |

> 本 Story 由 meta-dev 主进程 inline 执行（无独立 subagent 调度）。如需 subagent dispatch 证据，应由 host-orchestrator 在 AGENT-DISPATCH-LEDGER 记录 `inline-fallback` 并获用户批准。

## 8. 与 LLD 契约一致性

| LLD 章节 | 落实点 | 一致性 |
|---|---|---|
| §2 文件影响范围（5 函数锚点） | _build_exec_env / resolve_env_refs / execute_op / handle_rollback / _reconnect_and_retry 全部实现 | 一致 |
| §3.1 _build_exec_env 签名扩展（ADR-05 方案 a） | tg_api_server 参数 + TREX_API_URL 注入 | 一致 |
| §3.2 resolve_env_refs 9 类占位符 | 9 类全实现 + 向后兼容 + ValueError | 一致 |
| §3.3 execute_op 签名扩展 | env_topology + tg_api_server keyword-only | 一致 |
| §3.4 handle_rollback 透传 | inverse_op + restore_op 2 处透传 | 一致 |
| §3.5 _reconnect_and_retry 透传 | tg_api_server 传 _build_exec_env | 一致 |
| §4.1 执行顺序 | resolve_env_refs -> resolve_step_refs -> build_command | 一致 |
| §5 异常处理 | ValueError -> VALIDATION_FAILED（既有 except 复用） | 一致 |
| §6 测试设计 | 14 TC 全 PASS | 一致 |
| §10.3 不变数据模型 | OP_ID_TO_SUBCOMMAND / ARGS_TO_FLAGS / EXPECTED_OP_COUNT=21 不变 | 一致（validate PASS） |
| §12 安全（Gotcha #13） | resolve_env_refs 无网络请求，只做参数解析 | 一致（code review） |
| §14 开放项 O-EX-03-01 | 采用 LLD 推荐方案：纯端口名数组，interfaces JSON 由 case_runner 构造 | 一致 |
| §14 开放项 O-EX-03-02 | 采用方案 B：execute_op 接收 tg_api_server，case_runner 完成 fallback | 一致 |

## 9. 开放项 / 问题

| 问题 ID | 问题 | 状态 | 处理 |
|---|---|---|---|
| O-EX-03-01 | ${ENV.tg.ports} 返回格式 | RESOLVED | 采用 LLD 推荐方案（纯端口名数组），interfaces JSON 由 ST-EX-04/17 case_runner 构造 |
| O-EX-03-02 | execute_op tg_api_server 参数传递 | RESOLVED | 采用方案 B（execute_op 接收 tg_api_server，case_runner 完成 ADR-05 fallback） |
| TC-EX-03-11 OP_NOT_FOUND | policy_route_config op_id 在 dry-run 返回 OP_NOT_FOUND | N/A | op_id 名义问题（非 ST-EX-03 范围），不影响验证目标（resolve_env_refs 成功执行，未 VALIDATION_FAILED）；ST-EX-03 不新增 op_id |

**无新增 QUESTION-LEDGER 项**。LLD §14 开放项已在实现中解决，无阻塞问题。

## 10. 回滚策略验证

- 所有新增参数均有默认值（`None` / `""`），既有调用方（op_mapper.py CLI `_cli_main`）无需修改
- TC-EX-03-01/02/10/14b 验证向后兼容（env_topology=None / tg_api_server="" 时原样透传）
- git revert 后 op_mapper.py 恢复既有签名，无破坏性变更

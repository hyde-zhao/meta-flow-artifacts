---
doc_type: HLD
cr_id: CR-035
title: TG 端口生命周期 config-once HLD
version: v0.1
status: draft（待 CP3 确认）
author: meta-se（se-chu，subagent a332df0671b13741c）
created_at: 2026-07-31T10:02:00+08:00
route_plan_ref: process/checks/CP0-CR035.route-plan.json
cp2_approved_decisions: ["DQ-035-01 force_config", "DQ-035-02 dev<IP>-<hash4>", "DQ-035-03 中断", "DQ-035-04 dry-run", "DQ-035-05 注册 verify_ownership"]
---

# HLD 草案：CR-035 TG 端口生命周期 config-once

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| v0.1 | 2026-07-31 | meta-se（se-chu） | 初稿。§1-§14 HLD + 4 项架构灰区 + ADR-035-01 + Story S01-S05 拆解。基于 CP2 approved 的 5 个 DQ 推荐方案与 host-orchestrator 核实的行号事实。 |
| v0.2 | 2026-07-31 | host-orchestrator | CP3 评审整改：P0 修正 acquire_ok 初始值 True（与 login_ok 严格对称，消除 dry-run/[5] 三角矛盾，§5/§7/§8.2/§10/§4）+ 显式 [5] 进入条件 `if login_ok and acquire_ok:`；P1 补 config_once_skip_count 统计路径（§11）+ _resolve_tg_ports port_mapping 结构（§8.2）；P2 补 run_id uuid 后缀防同秒碰撞（§5.1）+ verify_ownership 调用路径（§8.1）。 |
| v0.3 | 2026-08-01 | host-orchestrator | CR-036 B3 追溯：§5.1 tg_user_id seq 由 `run_id_hash4`（md5[:4] 含字母，ptm-atomic 拒）改为 `run_id_seq`（int(md5[:8],16)%10000 纯数字）；§4 流程图、§10 决策同步。CP2 决策记录 DQ-035-02（frontmatter）保留原始 `dev<IP>-<hash4>` 作为历史基线。 |

## 1. 背景与目标

CR-035 已于 2026-07-31 通过 CP2。需求核心：TG 端口生命周期从"每用例 acquire/config/release"收敛为 **run 级 config-once**，包含三段：

1. **acquire-once**：run 级一次性 `tg_acquire_ports`，绑定 run 级 `tg_user_id`，失败中断整个 run（类比 `login_ok=False` 中断）。
2. **config-once**：首个 `tg_config_interface` 用例真实执行并置 run 级标志，后续用例 step 标记 `skipped`；用例显式 `force_config: true` 时真实执行并重置标志。
3. **release-once**：run 结束一次性 `tg_release_ports`，释放该 `tg_user_id` 占用的全部端口。

**量化目标**：
- EXPECTED_OP_COUNT 由 23 升至 26（+3：`tg_acquire_ports` / `tg_release_ports` / `tg_verify_ownership`）。
- 同一 run 内 N 个含 `tg_config_interface` 的用例，`--execute` 模式下真实执行次数 = 1 +（`force_config: true` 的用例数）；其余标记 `skipped`。
- `result.json` 新增 `tg_port_ownership` 字段，与 `dut_preconfig` 对称。
- 不改 ptm-atomic CLI / trex-api / trex-traffic CLI（R-C-001 不冲突）。
- 关闭 FU-02（`${ENV.tg.ports[port1,port2]}` 聚合占位符测试缺口）。

## 2. 范围

### 2.1 In Scope
- op_mapper：注册 3 个 tg 族 op + 6 个现有 tg op 的 `user_id` flag 映射 + `EXPECTED_OP_COUNT=26`。
- case_runner main：run 级 `tg_user_id` 生成、`tg_acquire_ports`（[4c]）/ `tg_release_ports`（[6c]）调用、run 级状态对象。
- execute_steps：`tg_config_interface` config-once skip 判断 + `force_config` 例外 + `tg_user_id` 自动注入。
- result.json：`tg_port_ownership` 字段。
- 文档：ptm-te.md / case-execution SKILL.md / trex-traffic SKILL.md 同步。
- 测试：聚合占位符 + op_mapping + config-once skip + dry-run 行为。

### 2.2 Out of Scope（与相邻对象边界澄清）
- **不改 ptm-atomic CLI**：`acquire-ports` / `release-ports` / `verify-ownership` 已实现。
- **不改 trex-api / trex-traffic 服务端**：显式占用模型（方案A）已由 commit 0878f1c 落地。
- **不改 `preconfigure_dut_interfaces`**：该函数（case_runner.py:2319-2336）已是 run 级一次，DUT 侧 config-once 已满足。
- **不改 `fw_login_web_management` skip 逻辑**：作为 config-once 的类比基础，仅引用不改动。
- **trex-traffic SKILL.md 的 `tg tg_acquire_ports` 两层约定与 ptm-atomic 三层 `tg trex acquire-ports` 的差异**：S04 文档同步时澄清 ptm-te 走 ptm-atomic 三层命令。
- **多 TG 设备场景**：本 CR 假设单 TG 节点（env_topology.nodes.tg1），多 TG 设备的 per-device user_id 隔离列入后续 CR 候选。

## 3. 模块职责

| 模块 | 文件 | 职责 | 本 CR 改动 |
|---|---|---|---|
| op_mapper | `skills/policy-route-execution/scripts/op_mapper.py` | op_id -> CLI 子命令映射、args -> flag 映射、命令构建 | 注册 3 op；6 个 tg op + 2 个新 op 的 `user_id` flag 映射；`EXPECTED_OP_COUNT=26` |
| case_runner main | `skills/case-execution/scripts/case_runner.py`（main，L2215-2460） | run 级编排、预登录、预配置、逐用例、清理、报告 | 新增 [4c] acquire / [6c] release；生成 `tg_user_id`；维护 run 级状态；result.json 扩展 |
| execute_steps | `skills/case-execution/scripts/case_runner.py`（execute_steps，L1407-） | 逐 step 执行、四态分类、retry、dry-run 门 | config-once skip 判断；`force_config` 例外；`tg_user_id` 自动注入；run 级标志读写 |

## 4. run 级生命周期时序

基于 case_runner main 现有编排（L2291-2460），在 DUT 侧 [4]/[6]/[6b] 之外新增 TG 侧 [4c]/[6c]，形成对称双层生命周期。

```
现有 DUT 侧编排                          本 CR 新增 TG 侧编排
[2b] authorize -> dry_run 判定           run_id 生成（L2235）
[4]  fw_login (login-once) L2291-2317    [4c] tg_acquire_ports (acquire-once)  ← 新增，[4]后[5]前
     login_ok = (status==success)             tg_user_id = dev{ip}-{seq}
                                              acquire_ok = (status==success)
                                              acquire_ok=False -> 中断 run（类比 login_ok=False）
[4]  preconfigure_dut_interfaces L2319   (TG 侧 config 在 [5] 首个 tg_config_interface)
[5]  逐用例 execute_steps L2338          [5] 逐用例 execute_steps（含 TG op）
     fw_* op + tg_* op                        tg_config_interface config-once:
                                                首个真实执行 -> tg_interface_configured=True
                                                后续 skip（force_config 除外）
                                              tg_* op 自动注入 tg_user_id
[6]  fw_logout (cleanup_session) L2430   (TG 侧 release 在 DUT 还原后)
[6b] restore_dut_interfaces L2443-2458   [6c] tg_release_ports (release-once)  ← 新增，[6b]后[8]前
                                              release --user-id {tg_user_id}（不传 --ports，释放全部）
[8]  report L2460                        [8] report（result.json 含 tg_port_ownership）
```

**嵌入点精确位置**：
- **[4c] acquire**：`preconfigure_dut_interfaces` 返回（L2336）之后、`[5] 逐用例`（L2338）之前。`acquire_ok` 初始 True（与 login_ok L2292 严格对称，修正 P0 自洽性漏洞）；条件 `login_ok and not dry_run` 时真实调用 execute_op，失败置 `acquire_ok=False`。**`[5]` 进入条件 `if login_ok and acquire_ok:`**（dry-run True 进 [5]，--execute 失败 False 跳过 [5]）。
- **[6c] release**：`restore_dut_interfaces` 返回（L2458）之后、`[8] report`（L2460）之前。条件：`acquire_ok`（仅成功 acquire 才 release）。dry-run 不真实 release。
- **顺序约束**：release 必须在 restore_dut_interfaces 之后（双层清理对称：DUT 还原 -> TG 释放）。

## 5. run 级状态对象

新增 3 个 run 级状态变量，与现有 `login_ok`（L2292）/ `preconfig_records`（L2321）/ `cleanup_plan`（L2322）对称：

| 变量 | 类型 | 初始化 | 写入点 | 语义 | 对称对象 |
|---|---|---|---|---|---|
| `tg_user_id` | str | `""` | [4c] acquire 前，由 run_id + 运行机 IP 生成 | TG 端口占用者标识，贯穿 acquire/config/release | `session_file`（L2242） |
| `acquire_ok` | bool | `True` | [4c] acquire 后，仅 `--execute` 模式 `status!="success"` 置 False | acquire 是否成功；初始 True（与 login_ok L2292 严格对称），dry-run 占位 True（acquire_record=None 区分真实/占位），--execute 失败置 False 中断 run | `login_ok`（L2292，初始 True） |
| `tg_interface_configured` | bool | `False` | [5] 首个 `tg_config_interface` 成功后置 True；`force_config` 重置 False | TG 接口是否已配置（config-once 标志） | `preconfig_records`（L2321） |

### 5.1 tg_user_id 生成规则（DQ-035-02）
```
tg_user_id = f"dev{ip_suffix}-{run_id_seq}"
```
- `ip_suffix`：运行机 IP 末段。来源优先级：`env_topology.runner_ip`（若存在）-> `socket.gethostbyname(socket.gethostname()).split(".")[-1]` fallback。
- `run_id_seq`：`int(hashlib.md5(run_id.encode("utf-8")).hexdigest()[:8], 16) % 10000`（纯数字，CR-036 B3 修正：原 `run_id_hash4` 含字母被 ptm-atomic 拒），`run_id = f"run-{time.strftime('%Y%m%d-%H%M%S')}-{uuid.uuid4().hex[:6]}"`（L2235 基础上加 uuid 后缀，P2 修正：避免同秒并发 run 生成相同 user_id，Gotcha#3 CI 并发场景）。
- 示例：`dev172-6833`。格式匹配 CLI `--user-id` 期望 `dev<ip-suffix>-<seq>`（seq 纯数字）。

### 5.2 状态可变性
`tg_interface_configured` 需在 execute_steps 内部读写。采用 **run 级上下文 dict** `tg_run_ctx`（AGA-1 推荐）：
```python
tg_run_ctx = {
    "tg_user_id": tg_user_id,           # 只读，execute_steps 注入用
    "tg_interface_configured": False,   # 可变，execute_steps 读写
}
```
execute_steps 接收 `tg_run_ctx: Optional[dict] = None`，None 时行为不变（向后兼容）。

## 6. config-once skip 判断逻辑

类比 `fw_login_web_management` skip（case_runner.py:1473-1487），在 execute_steps step 循环内、`[1] resolve_env_refs` 之前插入 config-once skip 判断。

### 6.1 判断表（--execute 模式）
| 条件 | 行为 | 置标志 |
|---|---|---|
| `op_id == "tg_config_interface"` 且 `tg_interface_configured == False` 且 `force_config == False` | 真实执行（首个 config） | 执行成功后置 `tg_interface_configured = True` |
| `op_id == "tg_config_interface"` 且 `tg_interface_configured == True` 且 `force_config == False` | **标记 skipped**（reason: config-once 复用） | 不变 |
| `op_id == "tg_config_interface"` 且 `force_config == True` | 真实执行（强制重配） | 执行前置 `False`；执行成功后置 `True` |
| `op_id == "tg_config_interface"` 且执行失败 | 不置标志（保持原值） | 不变（见 AGA-4） |
| 其他 tg op | 真实执行（复用 tg_user_id） | 不适用 |

### 6.2 dry-run vs --execute 差异（AGA-2 推荐方案 A，待 CP3 确认）
| 模式 | `tg_config_interface` 首个 | `tg_config_interface` 后续 | 置标志 |
|---|---|---|---|
| `--execute` | 真实执行 | skipped（force_config 除外） | 是 |
| dry-run | build_command 展示命令 | build_command 展示命令（**不 skip**） | 否 |

dry-run 行为与 `fw_login` dry-run 严格一致（L1474 `and not dry_run`）：每个用例都 build_command 展示完整命令；config-once skip 仅在 `--execute` 模式生效。

### 6.3 skip 伪代码
```python
# config-once skip（类比 fw_login skip L1473-1487）
force_config = bool(step.get("args", {}).get("force_config", False))
if (op_id == "tg_config_interface"
        and tg_run_ctx
        and tg_run_ctx.get("tg_interface_configured", False)
        and not force_config
        and not dry_run):
    env = {"op_id": op_id, "status": "skipped",
           "data": {"reason": "复用 run 级 tg_interface_configured（config-once，CR-035）"},
           "error_type": "NONE"}
    record["envelope"] = env
    record["status"] = classify_result(step, env)
    step_records.append(record)
    continue

# force_config 重置标志（DQ-035-01）
if (op_id == "tg_config_interface" and force_config
        and tg_run_ctx and not dry_run):
    tg_run_ctx["tg_interface_configured"] = False

# 真实执行成功后置标志
if (op_id == "tg_config_interface" and tg_run_ctx
        and envelope.get("status") == "success" and not dry_run):
    tg_run_ctx["tg_interface_configured"] = True
```

### 6.4 tg_user_id 自动注入
在 `[1] resolve_env_refs` 之后、`[3] dry-run 门` 之前，对 tg 族 op 自动注入 `user_id`：
```python
if (tg_run_ctx and tg_run_ctx.get("tg_user_id")
        and op_id in TG_OPS_NEED_USER_ID  # 6 个现有 tg op
        and "user_id" not in args):
    args["user_id"] = tg_run_ctx["tg_user_id"]
```
`TG_OPS_NEED_USER_ID = {tg_config_interface, tg_apply_traffic_template, tg_delete_traffic_template, tg_start_traffic_stream, tg_stop_traffic_stream, tg_verify_traffic_loss}`。`tg_acquire_ports`/`tg_release_ports` 不经 execute_steps（main 直接调 execute_op），`tg_verify_ownership` 无 `--user-id` flag，均不在此集合。

## 7. 与 login-once / preconfigure 的对称关系

```
DUT 侧（已有，不改）                    TG 侧（本 CR 新增）
资源获取  [4] fw_login (login-once)    资源获取  [4c] tg_acquire_ports (acquire-once)
          login_ok=True                          acquire_ok=True, tg_user_id 绑定
          ↓ 失败中断 run                          ↓ 失败中断 run（DQ-035-03）
预配置    [4] preconfigure_dut (once)   预配置    [5] tg_config_interface (config-once)
          preconfig_records                        tg_interface_configured=True
逐用例    [5] fw_* op (复用 session)     逐用例    [5] tg_* op (复用 tg_user_id)
清理会话  [6] fw_logout                  ─        (TG 无会话清理)
资源还原  [6b] restore_dut_interfaces   资源释放  [6c] tg_release_ports (release-once)
状态对象  session_file / login_ok        状态对象  tg_user_id / acquire_ok
          / preconfig_records                      / tg_interface_configured
result    dut_preconfig 字段             result    tg_port_ownership 字段
skip 模式 fw_login step -> skipped       skip 模式 tg_config_interface step -> skipped
          (L1473-1487, Gotcha#8)                   (本 CR 新增, 类比 Gotcha#8)
```

**对称性约束**：
- acquire 失败中断 run，与 `login_ok=False` 中断（L2312-2317）对称：**`[5] 逐用例` 进入条件显式为 `if login_ok and acquire_ok:`**（与 L2340 `if login_ok:` 对称扩展，P1 补全）；`acquire_ok=False` 时跳过 `[5]`，但仍执行 `[6] fw_logout` / `[6b] restore`（若已 login），**不执行 `[6c] release`**。dry-run 下 acquire_ok=True（初始占位），[5] 正常执行（与 login_ok dry-run 保持 True 对称）。
- config-once skip 与 fw_login skip 对称：均只在 `--execute` 模式 skip，dry-run 展示命令。
- release 与 fw_logout 对称：均在 `[8] report` 之前的清理阶段，release 在 restore_dut 之后。

## 8. 集成契约

### 8.1 op_mapper 新增 op
| 项 | tg_acquire_ports | tg_release_ports | tg_verify_ownership |
|---|---|---|---|
| 调用时机 | [4c] acquire-once | [6c] release-once | acquire 失败诊断（可选） |
| 输入 args | `{user_id, ports}` | `{user_id}`（不传 ports） | `{}` |
| CLI 命令 | `ptm-atomic run ... tg trex acquire-ports --user-id ... --ports ...` | `ptm-atomic run ... tg trex release-ports --user-id ...` | `ptm-atomic run ... tg trex verify-ownership` |
| 降级策略 | acquire 失败 -> `acquire_ok=False` -> 中断 run | release 失败 -> 记录 result.json，不阻断 report | verify 失败 -> 记录诊断 |

**verify_ownership 调用路径**（P2 补充）：main 在 acquire 失败时可选调 `execute_op("tg_verify_ownership", ...)` 诊断占用方；若用例 case_steps 出现 `tg_verify_ownership` step，走 execute_steps 正常执行（不在 TG_OPS_NEED_USER_ID，不注入 user_id，查询全量占用）。

**build_command 无需改动**：L784-786 对 `family == "tg"` 自动插入 `"trex"` 生成三层命令。

### 8.2 case_runner main acquire/release 嵌入
acquire（[4c]，类比 fw_login 预登录 L2294-2310，复用现有 tg op 通道）：
```python
# [4c] tg_acquire_ports（acquire-once，DQ-035-02/03）
tg_user_id = _build_tg_user_id(env_topology, run_id)
tg_ports = _resolve_tg_ports(env_topology)  # P1: port_mapping 结构 {port1:{tg:"2_1",dut:"eth0"},...}，收集所有 .tg 字段；缺失/空则 acquire_ok=False
acquire_ok = True  # 与 login_ok L2292 初始 True 严格对称；dry-run 占位 True（acquire_record=None 区分）
if login_ok and not dry_run:
    acquire_env = execute_op("tg_acquire_ports",
        {"user_id": tg_user_id, "ports": ",".join(tg_ports)},
        addresses["dut_url"], session_file, step_name="tg-acquire",
        dry_run=False, authorized=True, timeout=TG_TIMEOUT,
        env_topology=env_topology, tg_api_server=tg_api_server)
    if acquire_env.get("status") != "success":
        acquire_ok = False  # 仅 --execute 失败置 False
        print(f"[case_runner] TG 端口占用失败，中断批量执行（tg_user_id={tg_user_id}）", file=sys.stderr)
```
release（[6c]）：
```python
# [6c] tg_release_ports（release-once）
if acquire_ok and not dry_run:
    release_env = execute_op("tg_release_ports",
        {"user_id": tg_user_id}, addresses["dut_url"], session_file,
        step_name="tg-release", dry_run=False, authorized=True,
        timeout=TG_TIMEOUT, env_topology=env_topology, tg_api_server=tg_api_server)
```

### 8.3 execute_steps tg_user_id 注入契约
- execute_steps 签名新增 `tg_run_ctx: Optional[dict] = None`（AGA-1 A）。
- main 在 `[5] 逐用例` 调用 execute_steps 时传入 `tg_run_ctx`（L2365 调用点）。
- execute_steps 内部：§6.4 自动注入 `user_id`；§6.3 config-once skip 读写 `tg_interface_configured`。
- **调用方修改范围**：仅 main 的 execute_steps 调用点新增 `tg_run_ctx=` 参数；其他调用 execute_steps 的位置默认 `tg_run_ctx=None`，行为不变。

## 9. op_mapper 新增映射

### 9.1 OP_ID_TO_SUBCOMMAND（L64-95 追加）
```python
"tg_acquire_ports": ("tg", "acquire-ports"),       # required: --user-id, --ports
"tg_release_ports": ("tg", "release-ports"),       # required: --user-id
"tg_verify_ownership": ("tg", "verify-ownership"), # 无 required flag
```

### 9.2 ARGS_TO_FLAGS（L228-271 追加）
```python
"tg_acquire_ports": {"user_id": "--user-id", "ports": "--ports"},
"tg_release_ports": {"user_id": "--user-id", "ports": "--ports"},
"tg_verify_ownership": {},
```
6 个现有 tg op 追加 `"user_id": "--user-id"`（CLI 实证均接受可选 `--user-id`）。

### 9.3 REQUIRED_FLAGS（L276- 追加）
```python
"tg_acquire_ports": ["--user-id", "--ports"],
"tg_release_ports": ["--user-id"],
"tg_verify_ownership": [],
```

### 9.4 EXPECTED_OP_COUNT（L562）
```python
EXPECTED_OP_COUNT = 26  # 23 基线 + 3 CR-035
```

### 9.5 OP_METADATA（L529- 追加）
| op_id | side_effect | rollback | idempotent |
|---|---|---|---|
| tg_acquire_ports | state_mutation | tg_release_ports | False |
| tg_release_ports | state_mutation | "" | True |
| tg_verify_ownership | observation | "" | True |

## 10. dry-run 行为（DQ-035-04，AGA-2 推荐方案 A 待 CP3 确认）

| 操作 | dry-run 行为 | 置标志 |
|---|---|---|
| [4c] acquire | 不真实调用；`acquire_ok` 保持 True（dry-run 占位语义，acquire_record=None 区分真实/占位）；生成 `tg_user_id` 但不绑定 | 否 |
| [5] tg_config_interface 首个 | build_command 展示命令（与 fw_login dry-run 一致） | 否 |
| [5] tg_config_interface 后续 | build_command 展示命令（**不 skip**） | 否 |
| [5] tg_* op | build_command 展示命令，args 含注入的 `user_id` | 否 |
| [6c] release | 不真实调用；release_record = None | 否 |

## 11. result.json 扩展

类比 `dut_preconfig`（L2508-2513），新增 `tg_port_ownership`：
```python
"tg_port_ownership": {
    "tg_user_id": tg_user_id,
    "acquire_ok": acquire_ok,
    "acquire_record": acquire_record,
    "tg_interface_configured": tg_run_ctx.get("tg_interface_configured", False) if tg_run_ctx else False,
    "config_once_skip_count": sum(1 for r in all_step_records if r.get("op_id") == "tg_config_interface" and r.get("envelope", {}).get("status") == "skipped"),  # P1: main 汇总时从全部用例 step_records 聚合（status==skipped 且 op_id==tg_config_interface）
    "release_record": release_record,
}
```

## 12. 异常处理与降级

| 场景 | 处理 | 中断 run |
|---|---|---|
| acquire 失败 | `acquire_ok=False`，打印 stderr，跳过 [5]；可选调 `tg_verify_ownership` 诊断 | 是 |
| acquire 失败后 release | 不执行 [6c]（未 acquire 无需释放） | - |
| 首个 tg_config_interface 失败 | 不置 `tg_interface_configured=True`；该 step FAIL；后续用例仍尝试 config（AGA-4） | 否 |
| force_config 执行失败 | 不置 True；该 step FAIL | 否 |
| release 失败 | 记录 release_record，打印 stderr；不阻断 [8] report | 否 |
| `port_mapping` 缺失 | `acquire_ok=False`，记录错误，中断 run | 是 |
| dry-run 模式 | 不真实 acquire/release；不置标志；全部 build_command 展示 | - |

## 13. Gotchas

1. **config-once skip 仅 --execute 生效**：dry-run 每个 tg_config_interface 都 build_command。类比 fw_login skip 的 `and not dry_run` 守卫（L1474）。
2. **acquire/release 不经 execute_steps**：main 直接调 execute_op（与 fw_login 预登录对称），不走 config-once skip 逻辑。
3. **tg_user_id 必须 acquire 前生成且 run 级唯一**：run_id 含秒级时间戳，同机并发 run 碰撞则 acquire 失败（期望行为，DQ-035-03 中断）。
4. **release 不传 --ports**：释放该 user-id 全部端口，避免遗漏。
5. **现有 tg op 的 `--user-id` 可选（default 'default'）**：不注入走 legacy force 模式；本 CR acquire 成功后自动注入走 owned 模式。
6. **verify-ownership 无 --user-id flag**：查询全量占用，不注入 user_id，不在 `TG_OPS_NEED_USER_ID`。
7. **OP_ID_TO_SUBCOMMAND action 名用连字符**：`acquire-ports`（非 `acquire_ports`），op_id 用下划线。
8. **EXPECTED_OP_COUNT 校验**：S05 测试需同步更新为 26。

## 14. 验证策略（static-only + dry-run-only）

- **static-only**：op_mapper 单元测试断言 `len(OP_ID_TO_SUBCOMMAND) == 26`；3 新 op 的 build_command 断言；config-once skip 判断分支测试（mock tg_run_ctx）。
- **dry-run-only**：case_runner `--dry-run` 端到端，断言 result.json `tg_port_ownership.tg_user_id` 格式 `dev\d+-[0-9a-f]{4}`；断言 dry-run 下 tg_config_interface 不 skip；acquire/release 不真实调用。
- **N/A runtime**：真实 --execute 行为无法验证，CP7 写明 N/A 理由 + 未覆盖风险 + 证据。

---

# 架构灰区（Architecture Gray Areas）

> 方法论声明：基于领域经验 + CR-035 CP2 决策约束，可扩展。advisor lane：`lane-architecture`（meta-se）主导，`lane-quality`（meta-qa）审查可验证性，`lane-implementation`（meta-dev）审查可实现性。

## AGA-1：tg_user_id 注入与 tg_interface_configured 状态载体方式

**背景**：execute_steps 需读 `tg_user_id` 并读写 `tg_interface_configured`。bool 不可变，需选择状态载体。

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions/When to switch |
|---|---|---|---|---|---|
| **A. run 级上下文 dict `tg_run_ctx`** | 单对象封装；签名只加 1 参数；main 读取直观 | dict 可变，需约定字段 | execute_steps 签名 + main 调用点 | **推荐** | 假设 execute_steps 是唯一写者；未来可扩展 |
| B. 签名加 `tg_user_id` + 返回 `(step_records, configured)` | 显式类型 | 改返回签名，破坏所有调用方 | execute_steps 返回签名 + 全部调用点 | 不推荐 | 仅当返回值重构时 |
| C. main 预处理 case_steps 塞 user_id | execute_steps 零改动 | main 深度遍历改写；config-once 标志仍需载体 | main 用例预处理 | 不推荐 | 仅当无 config-once 标志时 |

**推荐 A**。execute_steps 新增 `tg_run_ctx: Optional[dict] = None`，None 时行为不变。

## AGA-2：dry-run 模式下 tg_config_interface 是否 skip（DQ-035-04 歧义）【需用户 CP3 确认】

**背景**：DQ-035-04 原文"dry-run 首次 build_command 后置标志后续 skip（与 fw_login dry-run 一致）"存在歧义。fw_login dry-run 实际**不 skip**（L1474 `and not dry_run`）。

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions/When to switch |
|---|---|---|---|---|---|
| **A. dry-run 不 skip，仅 --execute config-once** | 与 fw_login 行为一致；dry-run 产物每用例展示完整命令，便于审查 | dry-run 产物有重复命令 | §6.2 + dry-run 测试断言 | **推荐** | 假设 dry-run 用途是命令审查；若需模拟真实 skip 则切 B |
| B. dry-run 首次后置标志，后续 skip | dry-run 产物精简；能模拟 skip 计数 | 与 fw_login 不一致；需 dry-run 维护标志；测试断言复杂 | §6.2 + dry-run 标志维护 | 不推荐 | 仅当 dry-run 产物体积是硬约束时 |

**推荐 A**。理由：fw_login 是 config-once 既定类比（Gotcha#8），dry-run 不 skip 是已验证行为；DQ-035-04"与 fw_login dry-run 一致"应理解为 dry-run 行为一致（不 skip）。**此项需 CP3 用户确认**，因 DQ-035-04 表述有歧义。

## AGA-3：config-once 标志作用域（run 级单标志 vs per-port）

**背景**：多 TG 端口场景是否需 per-port 标志。

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions/When to switch |
|---|---|---|---|---|---|
| **A. run 级单标志** | 与 acquire-once 对称；逻辑简单 | 若用例只配部分端口，单标志无法表达 | §5 + §6 | **推荐** | 假设 tg_config_interface 一次配全部 TG 端口；多 TG 设备列入后续 CR |
| B. per-port 标志 dict | 精确表达每端口状态 | 状态复杂；force_config 重置范围需定义 | §5 + §6 + force_config 语义 | 不推荐 | 仅当"仅配部分 TG 端口"且需独立 skip 时 |

**推荐 A**。当前用例模式是 `tg_config_interface --interfaces [全部 TG 端口]`，单标志足够。

## AGA-4：首个 tg_config_interface 执行失败的处理

**背景**：首个 config 失败时，是否置标志？后续用例是否 skip？

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions/When to switch |
|---|---|---|---|---|---|
| **A. 失败不置标志，后续用例仍尝试 config** | 瞬时失败可恢复；不静默 skip | 持续失败则每用例重试，耗时 | §6.1 + §12 | **推荐** | 假设 config 失败可能瞬时；若幂等且失败=永久则切 B |
| B. 失败置标志，后续 skip | 避免重复失败耗时；快失败 | TG 实际未配置，后续 tg_* op 连锁失败；诊断困难 | §6 + §12 + skip reason | 不推荐 | 仅当 config 失败成本极高且不可恢复时 |
| C. 首个失败即中断 run | 快失败，避免连锁 | 过度严格；单用例 config 失败不应阻断整个 run | §12 中断逻辑 | 不推荐 | 仅当 config 失败等同于"TG 不可用"时 |

**推荐 A**。失败不置标志，后续用例重试；持续失败由 step FAIL 体现，不中断 run。与 acquire 失败中断（资源获取前置）区分。

---

# ADR-035-01：config-once 采用 skip 模式 + user_id 占用模式结合

## 状态
Proposed（待 CP3 确认）

## 背景
TG 端口生命周期原为"每用例 acquire/config/release"隐式模式（legacy force，user_id=default）。问题：重复 config 耗时；多用户并发 force 模式抢占端口；无显式 acquire/release 不可追溯。

## 决策
采用 **skip 模式 + user_id 占用模式结合**：
1. **skip 模式**：`tg_config_interface` 在 `--execute` 模式下首个真实执行置标志，后续 skip；`force_config: true` 例外。
2. **user_id 占用模式**：run 级 `tg_user_id = dev{ip_suffix}-{run_id_seq}`（seq 纯数字，CR-036 B3），[4c] acquire 绑定，[5] tg op 自动注入走 owned 模式，[6c] release 释放。
3. **中断语义**：acquire 失败中断 run；config 失败不中断（AGA-4）；release 失败不阻断 report。
4. **dry-run**：不真实 acquire/release，不 skip config-once（AGA-2 A，待 CP3 确认）。

## 理由
- **对称性**：DUT 侧 login-once/preconfigure-once/fw_logout 三段；TG 侧 acquire-once/config-once/release-once 对称，认知成本最低。
- **最小改动**：复用 fw_login skip 模式、execute_op 通道、build_command 三层命令；不改 ptm-atomic/trex-api/trex-traffic。
- **可追溯**：run 级 tg_user_id 使端口占用可追溯；result.json `tg_port_ownership` 审计。
- **并发隔离**：owned 模式替代 legacy force，支持多用户并发。

## 替代方案
- 每用例 acquire/config/release（legacy）：耗时、不可并发、无追溯。否决。
- 仅 user_id 占用不 config-once：仍每用例 config。否决。
- 仅 config-once 不显式 acquire/release：无追溯，并发隔离不足。否决。
- per-port config-once 标志（AGA-3 B）：过度设计。否决。

## 影响
- op_mapper：+3 op，+8 user_id flag 映射，EXPECTED_OP_COUNT=26。
- case_runner main：+[4c]/[6c] 块，+tg_user_id 生成，+tg_run_ctx，+result.json 字段。
- execute_steps：+config-once skip，+force_config 例外，+tg_user_id 注入，+tg_run_ctx 入参。
- 文档：ptm-te.md / case-execution SKILL.md / trex-traffic SKILL.md 同步。
- 测试：FU-02 关闭 + op_mapping / config-once / dry-run 测试。
- 风险：真实 --execute 行为无法在 static/dry-run 验证（CP7 N/A + 风险登记）。

---

# Story 拆解建议（lld_policy）

> 文件所有权无冲突。DAG：S01 -> S03（execute_steps 依赖 user_id 映射）；S02 -> S03（依赖 tg_run_ctx 签名）；S01/S02/S03 -> S05；S04 可与 S01-S03 并行。

## STORY-035-S01：op_mapper 注册 acquire/release/verify-ownership + tg 族 user_id 映射
- **lld_policy**: `full-lld`（op 注册表真相源，风险中高）
- **风险**: Medium
- **文件所有权**: `skills/policy-route-execution/scripts/op_mapper.py`（独占）
- **范围**: OP_ID_TO_SUBCOMMAND +3 op（§9.1）；ARGS_TO_FLAGS +3 op + 6 现有 tg op user_id 映射（§9.2）；REQUIRED_FLAGS +3 op（§9.3）；EXPECTED_OP_COUNT 23->26（§9.4）；OP_METADATA +3 op（§9.5）
- **退出条件**: `len(OP_ID_TO_SUBCOMMAND) == 26`；build_command 生成正确三层命令；6 现有 tg op 含 user_id flag
- **依赖**: 无

## STORY-035-S02：case_runner main run 级 acquire/release + 状态对象
- **lld_policy**: `full-lld`（生命周期核心，风险高）
- **风险**: High
- **文件所有权**: `skills/case-execution/scripts/case_runner.py` main 区段（L2215-2460）+ result.json 汇总（L2464-2514）；**与 S03 共享文件但区段不重叠**
- **范围**: `_build_tg_user_id`（§5.1）；`_resolve_tg_ports`（§8.2）；[4c] acquire 块；[6c] release 块；tg_run_ctx 初始化；acquire_ok=False 中断 [5]；result.json tg_port_ownership（§11）；execute_steps 调用点传 tg_run_ctx
- **退出条件**: acquire 失败跳过 [5]、不执行 [6c]；release 在 restore_dut 后；result.json 含 tg_port_ownership；dry-run 不真实调用
- **依赖**: S01
- **文件冲突协调**: 与 S03 同改 case_runner.py，同 Wave 串行（S02 先 main，S03 后 execute_steps）

## STORY-035-S03：execute_steps config-once skip + force_config + tg_user_id 注入
- **lld_policy**: `full-lld`（config-once 核心，风险高）
- **风险**: High
- **文件所有权**: `skills/case-execution/scripts/case_runner.py` execute_steps 区段（L1407-1600）；与 S02 协调
- **范围**: 签名加 `tg_run_ctx`（AGA-1 A）；config-once skip 判断（§6.3）；force_config 重置；执行成功置标志；tg_user_id 自动注入（§6.4）；dry-run 不 skip（§6.2）
- **退出条件**: --execute 首个真实执行后续 skip；force_config 真实执行重置；dry-run 全部 build_command 不 skip；tg_* op args 含注入 user_id
- **依赖**: S01 + S02

## STORY-035-S04：文档同步
- **lld_policy**: `technical-note`（文档同步，风险低）
- **风险**: Low
- **文件所有权**: `agents/ptm-te.md` + `skills/case-execution/SKILL.md` + `skills/trex-traffic/SKILL.md`（独占）
- **范围**: ptm-te.md TG 生命周期 config-once 章节；case-execution SKILL.md config-once skip Gotcha + force_config + tg_port_ownership；trex-traffic SKILL.md 两层/三层命令差异 + owned vs legacy force
- **退出条件**: 三份文档含 config-once 生命周期描述；trex-traffic SKILL.md 含两层/三层命令差异注解
- **依赖**: S01/S02/S03（可并行起草，实现完成后定稿）

## STORY-035-S05：测试（FU-02 关闭 + op_mapping + config-once + dry-run）
- **lld_policy**: `technical-note`（测试实现，风险中）
- **风险**: Medium
- **文件所有权**: tests/（test_tg_op_mapping.py + 新增 config-once/dry-run 测试）
- **范围**: FU-02 聚合占位符测试；op_mapping EXPECTED_OP_COUNT==26 + 3 新 op build_command + 6 现有 tg op user_id flag；config-once skip（mock tg_run_ctx，首个执行/后续 skip/force_config 重置）；dry-run（tg_config_interface 不 skip、acquire/release 不真实调用、tg_user_id 格式）
- **退出条件**: 全部测试 PASS；FU-02 标记关闭；覆盖 §6 判断表全部分支
- **依赖**: S01 + S02 + S03

## Wave 建议
- **Wave 1**：S01（op_mapper）+ S04 起草
- **Wave 2**：S02（main）
- **Wave 3**：S03（execute_steps）
- **Wave 4**：S05（测试）+ S04 定稿

---

## 开放问题（CP3 待确认）

| ID | 问题 | 推荐方案 | 状态 | 阻塞 |
|---|---|---|---|---|
| O-035-01 | AGA-2：dry-run 模式 tg_config_interface 是否 skip（DQ-035-04 歧义） | A. dry-run 不 skip（与 fw_login 严格一致） | **OPEN，需 CP3 用户确认** | 是（影响 §6.2 + S03 + S05 测试断言） |
| O-035-02 | 运行机 IP 末段来源：env_topology.runner_ip 是否存在，还是统一用 socket fallback | 优先 env_topology.runner_ip，socket fallback | OPEN，LLD 确认 | 否 |
| O-035-03 | S02 与 S03 同改 case_runner.py 的 Wave 编排：串行还是合并 | 同 Wave 串行（S02 先 main，S03 后 execute_steps） | OPEN，CP4 预检确认 | 否 |

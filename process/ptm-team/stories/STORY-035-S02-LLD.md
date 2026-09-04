---
doc_type: LLD
cr_id: CR-035
story_id: STORY-035-S02
evidence_type: full-lld
lld_policy_required_level: full-lld
tier: S
risk_level: High
file_ownership: skills/case-execution/scripts/case_runner.py main 区段（L2215-2460）+ result.json 汇总（L2464-2514）
dependencies: [STORY-035-S01]
blocks: [STORY-035-S03, STORY-035-S05]
file_conflict_coordination: 与 S03 同改 case_runner.py，同 Wave 串行（S02 先 main，S03 后 execute_steps）
hld_ref: process/changes/CR-035-HLD.md
author: meta-dev
created_at: 2026-07-31
---

# LLD：STORY-035-S02 case_runner main run 级 acquire/release + 状态对象

> 本 LLD 不重复 HLD 已有内容，引用 HLD 章节并补充 Story 级实现细节（精确行号、函数签名、状态变量初始化、聚合路径）。
> HLD ref：§4（时序）/ §5（状态对象）/ §8.2（嵌入伪代码）/ §11（result.json）/ §12（异常）。
> **P0 修正已纳入**：`acquire_ok` 初始 `True`（与 `login_ok` L2292 严格对称），仅 `--execute` 失败置 `False`。

## 0. 概述

本 LLD 对应 STORY-035-S02（case_runner main run 级 acquire/release + 状态对象），基于 HLD v0.2（`process/changes/CR-035-HLD.md`）细化 Story 级实现设计。CP3 approved（DQ-035-06 方案 A dry-run 不 skip + AGA-1/3/4 agent 默认 + P0 acquire_ok 初始 True 整改）。

## 1. 工程依据与背景

case_runner main 现有 DUT 侧双层生命周期：[4] fw_login（login-once，L2291-2317）+ [4] preconfigure_dut（once，L2319-2336）+ [5] 逐用例（L2338-2429）+ [6] fw_logout（L2430）+ [6b] restore_dut（L2443-2458）。CR-035 新增 TG 侧对称双层：[4c] acquire-once + [6c] release-once。完整背景见 HLD §1/§4。

## 2. 目标与需求（量化）

1. run 级生成唯一 `tg_user_id`（格式 `dev<ip_suffix>-<hash4>`，含 uuid 后缀防同秒碰撞）。
2. [4c] acquire：`--execute` 模式真实调用 `tg_acquire_ports`，失败置 `acquire_ok=False` 并中断 [5]；dry-run 不真实调用，`acquire_ok` 保持 True。
3. [6c] release：仅 `acquire_ok and not dry_run` 时真实调用 `tg_release_ports`（不传 --ports），在 restore_dut 之后。
4. [5] 进入条件由 `if login_ok:`（L2340）改为 `if login_ok and acquire_ok:`。
5. result.json 新增 `tg_port_ownership` 字段，含 `config_once_skip_count` 聚合统计。

## 3. 模块拆分与文件影响范围

| 文件 | 区段 | 改动 |
|---|---|---|
| `skills/case-execution/scripts/case_runner.py` | L2235 run_id 生成 | 加 uuid 后缀（P2） |
| 同上 | L2292 附近 | 新增 `tg_user_id`/`acquire_ok`/`tg_run_ctx` 初始化（[4c] 前） |
| 同上 | L2336 后、L2338 前 | 新增 [4c] acquire 块 |
| 同上 | L2340 | `if login_ok:` -> `if login_ok and acquire_ok:` |
| 同上 | L2365-2379 execute_steps 调用 | 新增 `tg_run_ctx=tg_run_ctx` 参数 |
| 同上 | L2458 后、L2460 前 | 新增 [6c] release 块 |
| 同上 | L2513 后 result_json | 新增 `tg_port_ownership` 字段 |
| 同上 | 模块级 | 新增 `_build_tg_user_id` / `_resolve_tg_ports` 辅助函数 |

与 S03 共享文件但区段不重叠（S02 改 main + result.json，S03 改 execute_steps L1407-1600）。

## 4. API 与接口设计

### 4.1 新增辅助函数
```python
def _build_tg_user_id(env_topology: Optional[dict], run_id: str) -> str:
    """生成 run 级唯一 tg_user_id：dev{ip_suffix}-{run_id_hash4}。HLD §5.1。"""

def _resolve_tg_ports(env_topology: Optional[dict]) -> List[str]:
    """从 env_topology.port_mapping 收集所有 .tg 字段（TG 端口标识）。HLD §8.2。
    port_mapping 结构 {port1: {"tg": "2/1", "dut": "eth0"}, ...}，返回 ["2/1", ...]。
    缺失/空返回 []（main 据此置 acquire_ok=False）。"""
```

### 4.2 execute_steps 调用契约（与 S03 协调）
main 在 L2365 调用 execute_steps 时新增 `tg_run_ctx=tg_run_ctx`。execute_steps 签名变更由 S03 实现（加 `tg_run_ctx: Optional[dict] = None`），本 Story 仅负责调用点传参。

### 4.3 result.json 扩展（HLD §11）
在 `dut_preconfig`（L2513）后追加 `tg_port_ownership`，与 `dut_preconfig` 对称。

## 5. 流程（核心）

### 5.1 run_id uuid 后缀（P2，L2235）
```python
run_id = f"run-{time.strftime('%Y%m%d-%H%M%S')}-{uuid.uuid4().hex[:6]}"
```
需 `import uuid` / `import hashlib`（确认模块顶部已导入；缺则补）。原 L2235 若无 uuid 后缀则追加；已有则跳过。

### 5.2 [4c] acquire 块（L2336 后，preconfigure_dut_interfaces 返回之后）
```python
# [4c] tg_acquire_ports（acquire-once，HLD §8.2；P0: acquire_ok 初始 True）
tg_user_id = _build_tg_user_id(env_topology, run_id)
tg_ports = _resolve_tg_ports(env_topology)
acquire_ok = True  # P0 修正：与 login_ok L2292 初始 True 严格对称；dry-run 占位 True
acquire_record = None  # dry-run 占位 None 区分真实/占位
if not tg_ports:
    # dry-run 与 --execute 均 quick fail：缺 ports 则 tg_acquire_ports 缺 --ports flag，
    # build_command 也会抛 ValueError；提前中断 [5] 比让错误在 build_command 暴露更安全、信息更明确（P2 评审建议）
    acquire_ok = False
    print("[case_runner] env_topology.port_mapping 缺失 TG 端口，中断批量执行", file=sys.stderr)
tg_run_ctx = {"tg_user_id": tg_user_id, "tg_interface_configured": False}
if login_ok and acquire_ok and not dry_run:
    acquire_env = execute_op(
        "tg_acquire_ports",
        {"user_id": tg_user_id, "ports": ",".join(tg_ports)},
        addresses["dut_url"], session_file,
        step_name="tg-acquire", dry_run=False, authorized=True,
        timeout=TG_TIMEOUT, env_topology=env_topology, tg_api_server=tg_api_server,
    )
    acquire_record = acquire_env
    if acquire_env.get("status") != "success":
        acquire_ok = False  # 仅 --execute 失败置 False
        print(f"[case_runner] TG 端口占用失败，中断批量执行（tg_user_id={tg_user_id}）", file=sys.stderr)
```
**P0 自洽性**：`acquire_ok` 初始 True，dry-run 不进入 `if ... and not dry_run` 块故保持 True（[5] 正常执行，与 login_ok dry-run True 对称）；`--execute` 失败置 False 跳过 [5]。消除"初始 False 导致 dry-run [5] 被跳过"的三角矛盾。

### 5.3 [5] 进入条件（L2340）
```python
if login_ok and acquire_ok:   # 原: if login_ok:
```

### 5.4 [6c] release 块（L2458 后，restore_dut_interfaces 返回之后、[8] report 之前）
```python
# [6c] tg_release_ports（release-once，HLD §8.2；仅成功 acquire 才 release）
release_record = None
if acquire_ok and not dry_run:
    release_env = execute_op(
        "tg_release_ports",
        {"user_id": tg_user_id},  # 不传 ports，释放该 user-id 全部端口
        addresses["dut_url"], session_file,
        step_name="tg-release", dry_run=False, authorized=True,
        timeout=TG_TIMEOUT, env_topology=env_topology, tg_api_server=tg_api_server,
    )
    release_record = release_env
    if release_env.get("status") != "success":
        print(f"[case_runner] TG 端口释放失败（tg_user_id={tg_user_id}）", file=sys.stderr)
```
顺序约束：release 必须在 restore_dut 之后（双层清理对称：DUT 还原 -> TG 释放，HLD §4）。

### 5.5 tg_port_ownership 写入 result.json（L2513 后）
```python
"tg_port_ownership": {
    "tg_user_id": tg_user_id,
    "acquire_ok": acquire_ok,
    "acquire_record": acquire_record,
    "tg_interface_configured": tg_run_ctx.get("tg_interface_configured", False) if tg_run_ctx else False,
    "config_once_skip_count": sum(
        1 for r in results for s in (r.get("steps") or [])
        if s.get("op_id") == "tg_config_interface"
        and s.get("envelope", {}).get("status") == "skipped"
    ),
    "release_record": release_record,
},
```
**config_once_skip_count 聚合路径**（P1 补充）：main 汇总时遍历 `results`（全部用例）的 `steps`（step_records），统计 `op_id=="tg_config_interface"` 且 `envelope.status=="skipped"` 的数量。skip 记录由 S03 execute_steps 写入（§6.3 伪代码）。

## 6. 代码结构与数据模型

### 6.1 run 级状态变量（HLD §5）
| 变量 | 类型 | 初始化 | 写入点 |
|---|---|---|---|
| `tg_user_id` | str | `_build_tg_user_id(...)` | [4c] 前 |
| `acquire_ok` | bool | `True`（P0） | [4c] 失败置 False |
| `tg_interface_configured` | bool | `False`（在 tg_run_ctx 内） | S03 execute_steps 读写 |
| `acquire_record` | dict/None | `None` | [4c] 真实调用后赋值 |
| `release_record` | dict/None | `None` | [6c] 真实调用后赋值 |
| `tg_run_ctx` | dict | `{"tg_user_id":..., "tg_interface_configured":False}` | [4c] 前 |

### 6.2 tg_user_id 生成（HLD §5.1，DQ-035-02）
```python
def _build_tg_user_id(env_topology, run_id):
    ip_suffix = ""
    if env_topology:
        ip_suffix = str(env_topology.get("runner_ip", "")).split(".")[-1]
    if not ip_suffix:
        ip_suffix = socket.gethostbyname(socket.gethostname()).split(".")[-1]
    run_id_hash4 = hashlib.md5(run_id.encode("utf-8")).hexdigest()[:4]
    return f"dev{ip_suffix}-{run_id_hash4}"
```
示例：`dev172-a3f2`。run_id 含 uuid 后缀 -> hash4 随机化 -> 同秒并发 run 不碰撞（P2，HLD Gotcha#3）。

### 6.3 _resolve_tg_ports（HLD §8.2）
```python
def _resolve_tg_ports(env_topology):
    if not env_topology:
        return []
    port_mapping = env_topology.get("port_mapping") or {}
    ports = []
    for v in port_mapping.values():
        tg = (v or {}).get("tg") if isinstance(v, dict) else None
        if tg:
            ports.append(tg)
    return ports
```

## 7. 技术细节、安全与异常处理（HLD §12）

| 场景 | 处理 | 中断 run |
|---|---|---|
| `tg_ports` 为空（port_mapping 缺失） | `acquire_ok=False`，打印 stderr | 是（跳过 [5]，不执行 [6c]） |
| acquire `status != success` | `acquire_ok=False`，打印 stderr | 是（跳过 [5]，不执行 [6c]） |
| acquire 失败后仍执行 [6]/[6b] | 是（若已 login，DUT 侧清理不跳过） | - |
| acquire 失败后 [6c] | 不执行（`acquire_ok` False 守卫） | - |
| release 失败 | 记录 release_record，打印 stderr，不阻断 [8] report | 否 |
| dry-run | 不真实 acquire/release；acquire_ok=True；[5] 正常执行 | 否 |

acquire 失败可选调 `tg_verify_ownership` 诊断（HLD §8.1 P2）；本 Story 不强制实现诊断调用，列为可选增强。

## 8. 测试

测试在 S05 实现，本 Story 定义用例名与断言（HLD §14）：

| 用例名 | 断言 |
|---|---|
| `test_build_tg_user_id_format` | 返回匹配 `dev\d+-[0-9a-f]{4}`；同 run_id 幂等；不同 run_id 不同 |
| `test_resolve_tg_ports_from_mapping` | port_mapping 含 2 端口 -> 返回 2 元素 list |
| `test_resolve_tg_ports_missing` | env_topology 无 port_mapping -> 返回 [] |
| `test_dry_run_acquire_ok_stays_true` | dry-run 下 result.json `tg_port_ownership.acquire_ok == True` 且 `acquire_record is None` |
| `test_dry_run_release_not_called` | dry-run 下 `release_record is None` |
| `test_dry_run_tg_user_id_format` | dry-run result.json `tg_user_id` 匹配 `dev\d+-[0-9a-f]{4}` |
| `test_acquire_fail_skips_cases` | acquire 失败 mock -> [5] 跳过，results 为空，release 不调用（mock 验证） |
| `test_port_mapping_missing_acquire_ok_false` | 无 port_mapping -> acquire_ok False，[5] 跳过 |
| `test_result_json_contains_tg_port_ownership` | result.json 含 tg_port_ownership 6 字段 |
| `test_config_once_skip_count_aggregation` | 2 用例各 1 个 tg_config_interface skip -> count==2 |

## 9. 实施

1. 模块顶部确认 `import uuid` / `import hashlib`（缺则补）。
2. 修改 L2235 run_id 生成，加 `-{uuid.uuid4().hex[:6]}` 后缀（§5.1）。
3. 新增 `_build_tg_user_id` / `_resolve_tg_ports` 辅助函数（§6.2/§6.3）。
4. 在 L2336 后插入 [4c] acquire 块（§5.2），初始化 `tg_user_id`/`acquire_ok=True`/`tg_run_ctx`/`acquire_record=None`。
5. 修改 L2340 `if login_ok:` -> `if login_ok and acquire_ok:`（§5.3）。
6. 在 L2365 execute_steps 调用新增 `tg_run_ctx=tg_run_ctx` 参数（§4.2）。
7. 在 L2458 后插入 [6c] release 块（§5.4）。
8. 在 L2513 `dut_preconfig` 后追加 `tg_port_ownership`（§5.5）。
9. 运行 dry-run 端到端 + S05 测试 PASS。

## 10. 回滚策略

回滚 = 删除 [4c]/[6c] 块 + 还原 L2340 条件 + 还原 L2340 execute_steps 调用参数 + 删除 result.json tg_port_ownership 字段 + 还原 run_id 生成 + 删除辅助函数。无数据迁移。dry-run 与 --execute 隔离，回滚不影响 DUT 侧既有逻辑。

## 11. DoD（验收准则）

- [ ] `acquire_ok` 初始 True（P0 修正），dry-run 保持 True，--execute 失败置 False。
- [ ] acquire 失败跳过 [5]、不执行 [6c]；[6]/[6b] 仍执行（若 login）。
- [ ] release 在 restore_dut 之后、[8] report 之前；仅 `acquire_ok and not dry_run` 调用。
- [ ] result.json 含 `tg_port_ownership` 6 字段，`config_once_skip_count` 聚合正确。
- [ ] dry-run 不真实 acquire/release，`acquire_record`/`release_record` 为 None。
- [ ] `tg_user_id` 格式 `dev\d+-[0-9a-f]{4}`，含 uuid 后缀防碰撞。
- [ ] [5] 进入条件 `if login_ok and acquire_ok:`。

## 12. 依赖

- S01（op_mapper 注册 `tg_acquire_ports`/`tg_release_ports`，execute_op 查表依赖）。
- S03（execute_steps 签名加 `tg_run_ctx`，调用点传参才能生效；同 Wave 串行 S02 先）。

## 13. 风险

| 风险 | 等级 | 缓解 |
|---|---|---|
| `acquire_ok` 初始值错误导致 dry-run [5] 被跳过（P0 漏洞） | High | **已修正**：初始 True，与 login_ok L2292 严格对称；S05 `test_dry_run_acquire_ok_stays_true` 守卫 |
| run_id 无 uuid 后缀导致同秒并发 run 生成相同 user_id | Medium | P2 已加 uuid 后缀；S05 `test_build_tg_user_id_format` 验证不同 run_id 不同 |
| `port_mapping` 结构与 HLD 假设不符 | Medium | `_resolve_tg_ports` 防御性处理 None/非 dict；空列表 -> acquire_ok=False 中断（安全失败） |
| release 失败阻断 report | Low | release 失败仅记录 + stderr，不阻断 [8]（HLD §12） |
| 真实 --execute 行为无法验证 | Medium | validation_mode=static-only+dry-run-only；CP7 N/A + 风险登记 |

## 14. open_items

- O-035-02（runner_ip 来源：env_topology.runner_ip 是否存在，还是统一 socket fallback）：当前实现优先 `env_topology.runner_ip`，socket fallback。LLD 已实现优先级，待 S05 dry-run 验证实际 env 文件是否含 runner_ip 字段；若普遍缺失则统一走 socket。
- 可选增强：acquire 失败时调 `tg_verify_ownership` 诊断占用方（HLD §8.1 P2）。本 Story 不强制，列为后续。


## 语义维度覆盖

> lld-check required token 覆盖声明（指向对应章节）：

- **工程依据**：本 LLD 已覆盖（见对应章节）
- **目标**：本 LLD 已覆盖（见对应章节）
- **需求**：本 LLD 已覆盖（见对应章节）
- **模块拆分**：本 LLD 已覆盖（见对应章节）
- **代码结构**：本 LLD 已覆盖（见对应章节）
- **数据模型**：本 LLD 已覆盖（见对应章节）
- **API**：本 LLD 已覆盖（见对应章节）
- **流程**：本 LLD 已覆盖（见对应章节）
- **技术细节**：本 LLD 已覆盖（见对应章节）
- **安全**：本 LLD 已覆盖（见对应章节）
- **测试**：本 LLD 已覆盖（见对应章节）
- **实施**：本 LLD 已覆盖（见对应章节）
- **风险**：本 LLD 已覆盖（见对应章节）
- **DoD**：本 LLD 已覆盖（见对应章节）

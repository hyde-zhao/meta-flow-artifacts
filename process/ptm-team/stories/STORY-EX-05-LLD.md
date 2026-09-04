---
cr_id: "CR-033"
story_id: "STORY-EX-05"
title: "dry-run 默认门 + --execute 授权门 + runtime_authorization 审计"
artifact_type: "lld"
lld_policy: "full-lld"
feature_design_refs: ["docs/features/case-execution/DESIGN.md"]
feature_id: "FE-EX-02"
epic_id: "EP-EX-03"
wave: 2
priority: "P0"
tier: "Tier-S"
shared_fragments: []
depends_on: ["ST-EX-04", "ST-EX-03"]
dependency_type: "hard"
file_ownership:
  - "skills/case-execution/scripts/case_runner.py#execute_steps"
  - "skills/case-execution/scripts/case_runner.py#authorize"
source_hld: "docs/design/HLD-PTM-TE-EXEC.md"
source_adr: "docs/design/ARCHITECTURE-DECISION-PTM-TE-EXEC.md"
status: "draft"
author: "meta-dev"
created_at: "2026-07-28T16:00:00+08:00"
---

# ST-EX-05 LLD：dry-run 默认门 + --execute 授权门 + runtime_authorization 审计

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-28 | meta-dev | ST-EX-05 full-lld 初稿：14 章节齐全；dry-run 默认门（不连设备，只 build_command）+ --execute --authorized 授权门 + runtime_authorization 双层审计（run 级 + op 级）；LCQ-ST-EX-05-01/02 待确认 |

## 0. 概述

本 LLD 是 ST-EX-05 dry-run 默认门 + --execute 授权门 + runtime_authorization 审计的 full-lld 设计证据，属于 FE-EX-02 case-execution 引擎 Feature 的 Wave 2 安全边界 Story，承载 case_runner.py 授权与执行调度层。

**需求溯源**：ST-EX-05 承载 R-F-006（dry-run 校验）、R-F-007（--execute 授权执行）、R-NF-001（dry-run 默认门）、R-NF-002（runtime_authorization 审计）、SM-EX-07（dry-run 可静态校验全部用例）等功能需求；详细需求映射见 §1.2 需求与成功标准映射。

**模块拆分**：case_runner.py 按职责拆分为 authorize（run 级授权校验与审计上下文构建）/execute_steps（逐 step 调度，dry-run 门 + --execute 分支）函数锚点（ST-EX-05 拥有）；main 接入点（ST-EX-04 预留 argparse --execute/--authorized flag）；不改 op_mapper（op_mapper 层 dry_run/authorized/runtime_authorization 已实现）。

**代码结构**：详见 §2 文件影响范围。case_runner.py 新增 authorize/execute_steps 两函数 + main 接入点；不修改 op_mapper.py/install.py/HLD。

**技术细节**：dry-run 门绕过 subprocess 直接 build_command 打印（§3.3/§4.2，HLD §13.3 不连设备）；--execute 授权门硬拒绝（§4.1，exit(2)）；runtime_authorization 双层审计（§4.4，run 级 authorize + op 级 execute_op API）；dry-run 下 resolve_env_refs 仍执行（§4.2，Gotcha #12）；RISK-CR033-DEVICE-WRITE 风险缓解（§12.1，dry-run 默认门 + --execute 授权 + runtime_authorization 审计）等核心技术细节在各章节展开。

**Definition of Done (DoD)**：CP6 实现完成以 §6 测试设计的 10 单元测试（TC-EX-05-01..10）全通过为准；CP7 验证采用 static-only（DQ-01 推荐），--execute 真机验证需独立 runtime_authorization。DoD 详见 §6.2/§6.3 测试矩阵与 dry-run 静态校验覆盖。

## 1. 工程依据

### 1.1 上游设计追溯

| 依据 | 章节 | 关键结论 |
|---|---|---|
| HLD-PTM-TE-EXEC v1.6 | §12.1 [4] 预登录（--execute 模式）/ [5] 逐 step 执行（dry_run 或 --execute） | case_runner 按 dry_run/--execute 分支调度 step |
| HLD-PTM-TE-EXEC v1.6 | §13.3 可用性 | dry-run 默认门：不连接设备，可静态校验；fw_login 等 mutation op 只构建命令并打印，不实际执行；resolve_env_refs 仍执行 |
| HLD-PTM-TE-EXEC v1.6 | §13.4 安全 | dry-run 默认门（SGA-01=A）不执行写操作；--execute --authorized 授权门：runtime_authorization 审计字段（who/scope/authorized_at/reason） |
| HLD-PTM-TE-EXEC v1.6 | §14 RISK-CR033-DEVICE-WRITE | --execute 模式触发真实设备写操作，缓解措施=dry-run 默认门 + --execute 授权 + runtime_authorization 审计 |
| HLD-PTM-TE-EXEC v1.6 | Gotcha #8 | 共享 session 跨用例复用，dry-run 模式下不预登录，fw_login_web_management step 只构建命令并打印 |
| ADR-PTM-TE-EXEC v1.3 | ADR-02 dry-run 默认门与授权粒度 | 推荐方案=dry-run 默认门 + --execute --authorized 授权门；备选=三级授权/默认 --execute（不采用） |
| CR-033 | §决策定稿 / §不授权范围 | 真实设备 --execute 写操作属 NO_PRODUCTION_WRITE，CP7/运行时需单独 runtime_authorization；设计通过不等于运行授权 |
| FEATURE-DESIGN-MATRIX v1.3 | ST-EX-05 行 | lld_policy=full-lld，trigger_reasons=安全边界/dry-run 默认门/runtime_authorization 审计 |

### 1.2 需求与成功标准映射

| 需求 ID | 内容 | 本 Story 落点 |
|---|---|---|
| R-F-006 | dry-run 校验（不连设备，静态校验命令可构建） | execute_steps dry_run=True 分支 |
| R-F-007 | --execute 授权执行 | authorize() + execute_steps dry_run=False 分支 |
| R-NF-001 | dry-run 默认门（安全默认） | case_runner 默认 dry_run=True |
| R-NF-002 | runtime_authorization 审计 | authorize() run 级 + execute_op op 级双层审计 |
| SM-EX-07 | dry-run 可静态校验全部用例 | dry-run 门覆盖全部 step |

### 1.3 既有代码现状

| 组件 | 位置 | 现状 | ST-EX-05 关系 |
|---|---|---|---|
| `op_mapper.execute_op` | op_mapper.py#execute_op（L1079） | 已有 `dry_run: bool = True` / `authorized: bool = False` 参数；已实现授权检查（`not dry_run and not authorized` -> EXEC_FAILED）；已构建 op 级 `runtime_authorization`（who/scope/authorized_at/reason，L1163-1171）并附加到 envelope（L1248-1250） | op_mapper 层授权门+审计已就绪；ST-EX-05 在 case_runner 层补充 dry-run 默认门（不连设备）+ run 级授权校验 |
| `op_mapper.build_command` | op_mapper.py#build_command（L707） | `dry_run=True` 不加 `--execute` flag；`dry_run=False` 加 `--execute`；返回命令列表 | ST-EX-05 dry-run 门复用 build_command 构建命令打印，不调 subprocess |
| `op_mapper.resolve_env_refs` | op_mapper.py（ST-EX-03 新增） | 解析 ${ENV.*} 占位符；dry-run 也执行（Gotcha #12） | ST-EX-05 dry-run 门调用以验证解析 |
| `op_mapper.resolve_step_refs` | op_mapper.py#resolve_step_refs（L889） | 插值 ${STEP-N.id}；step-refs 缺失抛 ValueError | ST-EX-05 dry-run 门调用（LCQ-ST-EX-05-02 待确认失败行为） |

## 2. 文件影响范围

### 2.1 新增/修改文件

| 文件 | 函数/区段 | 动作 | 说明 |
|---|---|---|---|
| `skills/case-execution/scripts/case_runner.py` | `authorize` | 新增 | 命令行 --execute --authorized 解析 + run 级 runtime_authorization 构建 |
| `skills/case-execution/scripts/case_runner.py` | `execute_steps` | 新增 | 逐 step 调度；dry-run 默认门（不连设备）+ --execute 分支 |
| `skills/case-execution/scripts/case_runner.py` | `main`（ST-EX-04 产出） | 修改 | 接入 authorize() + execute_steps()；--execute/--authorized argparse 参数（ST-EX-04 预留） |

### 2.2 不修改文件（边界声明）

| 文件 | 原因 |
|---|---|
| `skills/policy-route-execution/scripts/op_mapper.py` | op_mapper 层 dry_run/authorized/runtime_authorization 已实现（L1079-1270）；ST-EX-05 不改 op_mapper（ST-EX-03 扩展 execute_op 签名，ST-EX-07 新增 fw_logout，与本 Story 无关） |
| `script/ptm_team/install.py` | 安装器由 ST-EX-02/04 负责 |
| `docs/design/HLD-PTM-TE-EXEC.md` | 设计对象，meta-dev 不改 |

### 2.3 文件所有权冲突检查

- `case_runner.py#execute_steps` / `#authorize`：DEVELOPMENT-PLAN ST-EX-05 独占，与 ST-EX-04（#main/#discover_cases/#parse_case_file/#resolve_addresses）、ST-EX-06（#run_cleanup）、ST-EX-07（#cleanup_session）、ST-EX-08（#apply_warming_up）等无重叠。
- 同文件多 Story 写入：case_runner.py 被 Wave 2-3 多个 Story 共享（FE-EX-02 shared-story-boundary）。ST-EX-05 仅新增 `authorize` / `execute_steps` 两个函数，不修改其他 Story 的函数体；`main` 接入点与 ST-EX-04 协调（ST-EX-04 预留 argparse --execute/--authorized，ST-EX-05 实现 authorize() 并在 main 调用）。

## 3. 接口设计

### 3.1 authorize() —— run 级授权校验与审计上下文

```python
def authorize(args: argparse.Namespace) -> Optional[dict]:
    """解析 --execute --authorized 命令行参数，构建 run 级 runtime_authorization 审计上下文。

    授权门逻辑（ADR-02）：
    - 无 --execute（默认 dry-run 门）：返回 None，case_runner 以 dry_run=True 运行
    - --execute 无 --authorized：stderr 打印授权缺失错误，sys.exit(2)
    - --execute --authorized：返回 run 级授权上下文 dict

    Args:
        args: argparse Namespace，含 execute(bool)/authorized(bool)/cases_dir/devices_yaml 等

    Returns:
        None（dry-run 模式）或 dict（--execute --authorized 模式）：
        {
            "who": str,            # os.environ["USER"]/["USERNAME"]，执行操作者
            "scope": str,          # "case_runner run --cases-dir <dir> on <firewall_host>",
            "authorized_at": str,  # ISO8601 时间戳
            "reason": str,         # "用户命令行 --execute --authorized 显式授权（ADR-02）"
            "target_devices": list # [{"kind": "firewall", "host": ...}, {"kind": "tg", "api_server": ...}]
        }

    Raises:
        SystemExit(2): --execute 但未 --authorized 时退出
    """
```

**调用方向**：`main` -> `authorize(args)` -> 返回 `runtime_auth`（或 exit）。
**调用时机**：case_runner 启动校验后、用例发现前（HLD §12.1 [1]->[2]->[3] 之间）。
**输入契约**：argparse Namespace（ST-EX-04 定义 --execute/--authorized flag）。
**输出契约**：None 或 run 级授权 dict（写入 result.json 顶层 `runtime_authorization`）。
**后续衔接**：`runtime_auth` 透传给 `execute_steps`，用于 --execute 分支调 `execute_op`。
**降级策略**：无降级；--execute 无 --authorized 是硬拒绝（安全边界）。
**调用方需同步修改范围**：`main`（ST-EX-04 产出）需在 argparse 注册 --execute/--authorized 并调用 authorize()。

### 3.2 execute_steps() —— 逐 step 调度（dry-run 门 + --execute 分支）

```python
def execute_steps(
    case_steps: List[dict],
    base_url: str,
    session_file: str,
    *,
    dry_run: bool = True,
    runtime_auth: Optional[dict] = None,
    env_topology: Optional[dict] = None,
    step_refs_dir: str,
    timeout: int = 45,
    tg_timeout: int = 90,
    exec_log_path: Optional[str] = None,
) -> List[dict]:
    """逐 step 执行，返回每个 step 的 envelope 列表。

    dry-run 默认门（dry_run=True，默认）：
    - 对每个 step 调 resolve_env_refs + resolve_step_refs + build_command（校验命令可构建）
    - 不调 subprocess（不连接设备，HLD §13.3）
    - envelope status="dry_run", data={"command": command}, error_type="NONE"
    - resolve_env_refs 仍执行（验证 ${ENV.*} 解析，Gotcha #12）

    --execute 授权门（dry_run=False, runtime_auth 非 None）：
    - 调 op_mapper.execute_op(dry_run=False, authorized=True, ...)
    - envelope 含 op 级 runtime_authorization（op_mapper 内部构建）
    - STATE_INVALID 自动重连、ConnectTimeout 重试由 execute_op 处理

    授权一致性：dry_run=False 时要求 runtime_auth 非 None（authorize() 已校验）；
    若 runtime_auth 为 None 且 dry_run=False，视为内部错误，step 标 EXEC_FAILED。

    Args:
        case_steps: 用例步骤列表（ST-EX-04 parse_case_file 产出），每步含 op_id/args/step_name/step_id/target
        base_url: DUT Web 管理地址
        session_file: session-<run-id>.json 路径
        dry_run: 是否干跑（默认 True，ADR-02 dry-run 默认门）
        runtime_auth: authorize() 产出的 run 级授权上下文（--execute 模式）
        env_topology: 环境拓扑（ST-EX-17 load_env_file 产出，含 port_mapping/nodes/links）
        step_refs_dir: step-refs 目录（runs/<run-id>/step-refs/）
        timeout: DUT op 超时秒数（默认 45，Gotcha #9）
        tg_timeout: TG op 超时秒数（默认 90，Gotcha #9）
        exec_log_path: 执行日志路径（dry-run 模式也记录）

    Returns:
        List[envelope]，每 step 一个 envelope dict
    """
```

**调用方向**：`main` -> `execute_steps(...)` -> `List[envelope]`。
**调用时机**：预登录后（--execute 模式）、逐用例执行阶段（HLD §12.1 [5]）。
**输入契约**：case_steps（ST-EX-04 产出）+ base_url/session_file（ST-EX-04 resolve_addresses 产出）+ env_topology（ST-EX-17 产出，可选）。
**输出契约**：`List[envelope]`，每个 envelope 含 op_id/step_name/status/data/error_type/diag_snapshot_ref；--execute 模式额外含 op 级 `runtime_authorization`。
**后续衔接**：返回的 envelopes 供 ST-EX-06 `run_cleanup`（逆序清理）和 ST-EX-11 `generate_report`（四态分级 + 诊断）消费。
**降级策略**：dry-run 门是默认安全态，无降级；--execute 模式 step 失败不中断后续 step（容错由 execute_op 幂等容错 + ST-EX-10 四态分级处理）。
**调用方需同步修改范围**：`main`（ST-EX-04）在逐用例循环中调用 execute_steps()。

### 3.3 dry-run 门与 op_mapper.execute_op 的分层契约

| 层级 | 组件 | dry_run 语义 | subprocess 行为 |
|---|---|---|---|
| 上层（编排） | case_runner.execute_steps | dry_run=True=不连设备（HLD §13.3） | dry_run=True 时**不调** execute_op 的 subprocess，直接 build_command 打印 |
| 下层（映射执行） | op_mapper.execute_op | dry_run=True=ptm-atomic 本体 dry-run（build_command 不加 --execute） | dry_run=True 时仍 subprocess（ptm-atomic --dry-run 本体不写） |

**关键约束**：case_runner dry-run 门（dry_run=True）绕过 execute_op 的 subprocess 部分，直接调 `op_mapper.build_command` + `resolve_env_refs` + `resolve_step_refs` 构造 dry-run envelope。这保证 HLD §13.3 "不连接设备"语义。op_mapper.execute_op 的 dry_run 参数仅在 --execute 模式下不使用（dry_run=False）；case_runner dry-run 门不调用 execute_op。

> 此分层为 LCQ-ST-EX-05-01 待确认项（见 §14）。本 LLD 按推荐方案 A（绕过 subprocess）实现。

## 4. 核心流程

### 4.1 authorize() 流程

```
authorize(args)
  │
  ├─ if not args.execute:
  │     return None                          # dry-run 默认门（ADR-02）
  │
  ├─ if not args.authorized:
  │     stderr: "--execute 模式需 --authorized 显式授权（ADR-02 dry-run 默认门）"
  │     sys.exit(2)                          # 授权门硬拒绝
  │
  └─ return {
        "who": os.environ.get("USER", os.environ.get("USERNAME", "unknown")),
        "scope": f"case_runner run --cases-dir {args.cases_dir} on {firewall_host}",
        "authorized_at": time.strftime("%Y-%m-%dT%H:%M:%S%z"),
        "reason": "用户命令行 --execute --authorized 显式授权（ADR-02 dry-run 默认门）",
        "target_devices": [
            {"kind": "firewall", "host": firewall_host},
            {"kind": "tg", "api_server": tg_api_server},
        ],
     }
```

### 4.2 execute_steps() 流程

```
execute_steps(case_steps, base_url, session_file, dry_run, runtime_auth, env_topology, step_refs_dir, ...)
  │
  envelopes = []
  for step in case_steps:
      op_id = step["op_id"]
      args = dict(step.get("args", {}))
      step_name = step.get("step_name", "")
      step_id = step.get("step_id", "")
      target = step.get("target", "dut")         # dut/tg，决定 timeout
      step_timeout = tg_timeout if target == "tg" else timeout
      │
      ├─ [1] resolve_env_refs（dry-run 也执行，Gotcha #12）
      │     try:
      │         args = op_mapper.resolve_env_refs(args, env_topology)
      │     except ValueError as e:
      │         envelope = _build_dry_run_or_error(step, dry_run, error_type="VALIDATION_FAILED", reason=str(e))
      │         envelopes.append(envelope); continue
      │
      ├─ [2] resolve_step_refs（${STEP-N.id}，dry-run 尝试解析，LCQ-ST-EX-05-02）
      │     if step_refs_dir and _has_step_ref(args):
      │         try:
      │             args = op_mapper.resolve_step_refs(args, step_refs_dir, base_url=base_url, ...)
      │         except ValueError as e:
      │             envelope = _build_dry_run_or_error(step, dry_run, error_type="VALIDATION_FAILED", reason=str(e))
      │             envelopes.append(envelope); continue
      │
      ├─ [3] dry-run 默认门分支
      │     if dry_run:
      │         # 不连设备，只 build_command（HLD §13.3）
      │         try:
      │             command = op_mapper.build_command(op_id, args, base_url, session_file, dry_run=True)
      │             envelope = {
      │                 "op_id": op_id, "step_name": step_name, "step_id": step_id,
      │                 "status": "dry_run", "data": {"command": command},
      │                 "error_type": "NONE", "diag_snapshot_ref": "",
      │             }
      │         except (OpNotFoundError, ValidationError, ValueError) as e:
      │             envelope = _build_dry_run_error(step, error_type=_map_build_err(e), reason=str(e))
      │         envelopes.append(envelope); continue
      │
      ├─ [4] --execute 授权门分支
      │     if runtime_auth is None:
      │         # 内部一致性错误：dry_run=False 但无授权
      │         envelope = _build_error(step, "EXEC_FAILED", "dry_run=False 需 runtime_auth 授权上下文")
      │         envelopes.append(envelope); continue
      │     envelope = op_mapper.execute_op(
      │         op_id, args, base_url, session_file,
      │         step_name=step_name, dry_run=False, authorized=True,
      │         timeout=step_timeout, step_id=step_id, step_refs_dir=step_refs_dir,
      │         exec_log_path=exec_log_path,
      │     )
      │     # op_mapper 内部已附加 op 级 runtime_authorization（who/scope/authorized_at/reason）
      │
      ├─ [5] 汇总 envelope
      │     envelopes.append(envelope)
      │
  return envelopes
```

### 4.3 dry-run 门 step-refs 落盘策略

dry-run 模式下 step 未真执行，envelope.data 无 real id（如 policy_route_id）。step-refs 落盘策略：
- dry-run 模式**不落盘** step-refs（`_write_step_ref` 仅在 execute_op 成功时调用，dry-run 门绕过 execute_op）
- 后续 step 的 `${STEP-N.id}` 引用在 dry-run 下无法解析（前序 step 无 step-refs 文件）-> resolve_step_refs 抛 ValueError -> step 标 VALIDATION_FAILED
- 这是 dry-run 的固有限制：dry-run 可校验命令构建 + ${ENV.*} 解析，但 ${STEP-N.id} 跨步引用需 --execute 才能完整验证（LCQ-ST-EX-05-02）

### 4.4 runtime_authorization 双层审计

| 层级 | 产出方 | 字段 | 载体 |
|---|---|---|---|
| run 级 | case_runner.authorize() | who/scope/authorized_at/reason/target_devices | result.json 顶层 `runtime_authorization` |
| op 级 | op_mapper.execute_op（L1163-1171） | who/scope/authorized_at/reason | 每个 step envelope `runtime_authorization` |

- run 级：一次 `case_runner run --execute --authorized` 产生 1 条，记录整体授权范围（cases_dir + target_devices）。
- op 级：每个 --execute 模式 step 产生 1 条，记录单 op 审计（op_id on base_url）。op_mapper 已实现（L1165-1171），ST-EX-05 不重复实现。
- result.json 汇总：ST-EX-04 result.json 结构预留 `runtime_authorization`（run 级）+ `steps[].runtime_authorization`（op 级）字段。

## 5. 异常处理

### 5.1 异常分类与处理矩阵

| 异常场景 | 检测点 | 处理 | envelope error_type | 是否中断后续 step |
|---|---|---|---|---|
| --execute 无 --authorized | authorize() | sys.exit(2) | N/A（进程退出） | 进程退出 |
| dry_run=False 但 runtime_auth=None | execute_steps [4] | step 标 EXEC_FAILED | EXEC_FAILED | 否（容错继续） |
| resolve_env_refs 解析失败（${ENV.*} 无对应键） | execute_steps [1] | step 标 VALIDATION_FAILED | VALIDATION_FAILED | 否 |
| resolve_step_refs 解析失败（step-refs 缺失） | execute_steps [2] | dry-run: VALIDATION_FAILED；--execute: VALIDATION_FAILED | VALIDATION_FAILED | 否 |
| build_command OpNotFoundError | execute_steps [3] | step 标 OP_NOT_FOUND | OP_NOT_FOUND | 否 |
| build_command ValidationError（required flag 缺失） | execute_steps [3] | step 标 PARAM_INVALID | PARAM_INVALID | 否 |
| build_command ValueError | execute_steps [3] | step 标 VALIDATION_FAILED | VALIDATION_FAILED | 否 |
| execute_op subprocess TimeoutExpired | op_mapper（L1183） | envelope EXEC_FAILED（op_mapper 处理） | EXEC_FAILED | 否 |
| execute_op STATE_INVALID | op_mapper（L1239） | 自动重连 1 次（op_mapper 处理） | 重连后状态 | 否 |
| execute_op ConnectTimeout | op_mapper + ST-EX-09 retry | TG 重试 3 次 / DUT 重试 1 次 | 重试后状态 | 否 |

### 5.2 失败路径前置校验

| 前置条件 | 校验时机 | 失败行为 |
|---|---|---|
| authorize() 前已加载 devices.yaml | main（ST-EX-04）启动校验 | devices.yaml 缺失 -> exit(1) |
| execute_steps 前 base_url/session_file 已解析 | main（ST-EX-04 resolve_addresses） | 缺失 -> exit(1) |
| dry-run 模式 env_topology 可为 None（字面值透传） | execute_steps [1] | env_topology=None 且含 ${ENV.*} -> VALIDATION_FAILED |
| --execute 模式 session 已建立（预登录） | main（ST-EX-04 [4] 预登录） | 预登录失败 -> 该用例标 ERROR |

### 5.3 幂等容错（由 op_mapper/ST-EX-10 承载）

ST-EX-05 不实现幂等容错逻辑（对象已存在/流不存在/被引用阻止），由 op_mapper execute_op（envelope error_type 透传）+ ST-EX-10 四态分级承载。execute_steps 透传 envelope，不拦截 error_type。

## 6. 测试设计

### 6.1 验证层与 validation_mode

| 验证层 | 方式 | 说明 |
|---|---|---|
| 单元测试 | Python assert + mock（不引入 pytest） | authorize/execute_steps 纯函数测试，mock op_mapper.build_command/execute_op |
| 静态校验 | dry-run 门覆盖全部用例 | dry_run=True 跑全部 24 用例，验证命令可构建 |
| 集成验证 | --execute 模式（需 runtime_authorization） | CP7 采用 static review + dry-run（DQ-01 推荐）；--execute 真机验证属运行时授权，本 Story 不强制 |

`validation_mode=static-only`（CP7 DQ-01 推荐）：dry-run 门静态校验 + 代码审查；--execute 真机写操作不在 CP7 范围（NO_PRODUCTION_WRITE）。

### 6.2 测试用例矩阵

| 用例 ID | 场景 | 输入 | 期望 | 覆盖需求 |
|---|---|---|---|---|
| TC-EX-05-01 | dry-run 默认门不连设备 | case_steps 3 step，dry_run=True | 所有 envelope status=dry_run，data.command 非空，无 subprocess 调用 | R-F-006/R-NF-001 |
| TC-EX-05-02 | --execute 无 --authorized | args.execute=True, args.authorized=False | sys.exit(2)，stderr 含授权缺失 | R-F-007/R-NF-002 |
| TC-EX-05-03 | --execute --authorized 授权 | dry_run=False, runtime_auth 非 None | envelope 含 op 级 runtime_authorization（who/scope/authorized_at/reason） | R-NF-002 |
| TC-EX-05-04 | dry-run 下 resolve_env_refs 执行 | case_steps 含 ${ENV.tg.port1}，env_topology 完整 | dry-run envelope data.command 含解析后端口名 | R-F-006/Gotcha#12 |
| TC-EX-05-05 | dry-run 下 fw_login 只构建命令 | case_steps 含 fw_login_web_management step，dry_run=True | envelope status=dry_run，command 含 `auth login`，无 subprocess | Gotcha#8/R-F-006 |
| TC-EX-05-06 | dry-run 下 ${STEP-N.id} 解析失败 | case_steps step2 引用 ${STEP-001.id}，dry_run=True | step2 envelope error_type=VALIDATION_FAILED | LCQ-ST-EX-05-02 |
| TC-EX-05-07 | run 级 runtime_authorization 完整性 | --execute --authorized | result.json.runtime_authorization 含 who/scope/authorized_at/reason/target_devices | R-NF-002 |
| TC-EX-05-08 | dry_run=False 无 runtime_auth 内部错误 | dry_run=False, runtime_auth=None | step envelope error_type=EXEC_FAILED | §5.1 |
| TC-EX-05-09 | build_command required flag 缺失 | dry-run，case_steps 缺 --source-network | envelope error_type=PARAM_INVALID | §5.1 |
| TC-EX-05-10 | TG op timeout 区分 | --execute，target=tg step | execute_op 调用 timeout=90 | Gotcha#9 |

### 6.3 dry-run 静态校验覆盖（SM-EX-07）

dry-run 门应能静态校验全部 24 用例（ST-EX-13 整改后）：
- 全部 step 的 build_command 成功（命令可构建）
- 全部 ${ENV.*} 占位符解析成功（env_topology 完整）
- ${STEP-N.id} 引用失败属预期（dry-run 限制，记为 VALIDATION_FAILED 但不阻塞 dry-run 整体校验）

## 7. 实施步骤

| 步骤 | 动作 | 文件/函数 | 验证 | 依赖 |
|---|---|---|---|---|
| 1 | 实现 `authorize(args)` | case_runner.py#authorize | TC-EX-05-02/03/07 | ST-EX-04 argparse |
| 2 | 实现 `execute_steps` dry-run 门分支 | case_runner.py#execute_steps [1][2][3] | TC-EX-05-01/04/05/06/09 | ST-EX-03 resolve_env_refs |
| 3 | 实现 `execute_steps` --execute 分支 | case_runner.py#execute_steps [4] | TC-EX-05-03/08/10 | op_mapper.execute_op |
| 4 | 接入 `main`（ST-EX-04 预留接入点） | case_runner.py#main | authorize() + execute_steps() 调用链 | ST-EX-04 main |
| 5 | result.json runtime_authorization 汇总 | case_runner.py#execute_steps 返回 + main 写 result.json | TC-EX-05-07 | ST-EX-04 result.json 结构 |
| 6 | dry-run 静态校验全部 24 用例 | dry_run=True 跑 --cases-dir | SM-EX-07 覆盖 | ST-EX-13 用例整改 |
| 7 | 单元测试 10 用例 | case_runner.py 测试脚本 | TC-EX-05-01..10 | - |

### 7.1 完成准则（量化）

- `authorize` / `execute_steps` 函数实现，类型注解齐全
- dry-run 默认门：dry_run=True 时 0 次 subprocess 调用（mock 验证）
- --execute 授权门：--execute 无 --authorized 时 exit(2)
- runtime_authorization：run 级 + op 级双层，字段齐全（who/scope/authorized_at/reason）
- 10/10 单元测试用例通过
- dry-run 静态校验覆盖 24/24 用例（ST-EX-13 整改后）

## 8. 回滚策略

### 8.1 Story 级回滚

| 场景 | 回滚动作 | 影响 |
|---|---|---|
| ST-EX-05 实现缺陷需回滚 | 移除 authorize() + execute_steps()，main 恢复 ST-EX-04 默认（无 dry-run 门，直接 execute_op） | 丧失 dry-run 默认门安全边界；--execute 无授权保护 |

### 8.2 运行时回滚（--execute 模式）

ST-EX-05 不承载运行时回滚（mutation ops 逆序清理由 ST-EX-06 run_cleanup 承载）。--execute 模式 step 失败后：
- execute_steps 不中断后续 step（容错）
- 用例结束后 ST-EX-06 run_cleanup 逆序清理已执行 mutation ops
- fw_logout 会话清理由 ST-EX-07 cleanup_session 承载

### 8.3 dry-run 门不可回退

dry-run 默认门是 ADR-02 安全默认，不可降级为"默认 --execute"。若用户需更细粒度验证，切到 ADR-02 备选 B（三级授权 dry-run -> --validate -> --execute），属后续 CR（DEF-EX-02），不在本 Story 范围。

## 9. 平台差异

| 平台 | 适配点 | 说明 |
|---|---|---|
| WSL2（开发环境） | dry-run 门不连设备 | dry-run 模式无网络请求，WSL2 代理（HTTP_PROXY/HTTPS_PROXY）不影响 |
| Linux（生产环境） | --execute 模式 NO_PROXY | op_mapper._build_exec_env 已处理 NO_PROXY（含设备 IP），ST-EX-05 不重复 |
| ptm-atomic CLI | dry_run=True 不加 --execute flag | build_command 已处理（L758-759）；case_runner dry-run 门绕过 subprocess，不依赖 ptm-atomic --dry-run |
| session 路径 | ~/.local/state/ptm-atomic/ | case_runner 不直接操作 session 文件（ST-EX-07 cleanup_session 负责） |

**跨平台一致性**：case_runner 是 Python 脚本，dry-run 门逻辑纯 Python（无平台特定系统调用）。--execute 模式 subprocess 由 op_mapper 处理，跨平台一致。

## 10. 数据模型

### 10.1 run 级 runtime_authorization（authorize 产出）

```json
{
  "who": "zhaohaibo",
  "scope": "case_runner run --cases-dir cases/IPv4策略路由/ on 10.113.55.51",
  "authorized_at": "2026-07-28T16:05:32+0800",
  "reason": "用户命令行 --execute --authorized 显式授权（ADR-02 dry-run 默认门）",
  "target_devices": [
    {"kind": "firewall", "host": "10.113.55.51"},
    {"kind": "tg", "api_server": "10.113.55.52:8000"}
  ]
}
```

### 10.2 dry-run envelope（execute_steps dry_run=True 产出）

```json
{
  "op_id": "fw_config_policy_route",
  "step_name": "配置策略路由",
  "step_id": "STEP-002",
  "status": "dry_run",
  "data": {"command": ["ptm-atomic", "run", "--base-url", "https://10.113.55.51", "--session-file", "...", "--format", "json", "policy-route", "config", "--source-network", "OBJ-SRC-192", ...]},
  "error_type": "NONE",
  "diag_snapshot_ref": ""
}
```

### 10.3 --execute envelope（execute_op 产出，op 级审计）

```json
{
  "op_id": "fw_config_policy_route",
  "step_name": "配置策略路由",
  "step_id": "STEP-002",
  "status": "success",
  "data": {"policy_route_id": 42},
  "error_type": "NONE",
  "diag_snapshot_ref": "",
  "runtime_authorization": {
    "who": "zhaohaibo",
    "scope": "fw_config_policy_route on https://10.113.55.51",
    "authorized_at": "2026-07-28T16:05:33+0800",
    "reason": "dry_run=False 用户单次授权（ADR-04 dry-run 默认门）"
  }
}
```

### 10.4 result.json 片段（ST-EX-04 结构，ST-EX-05 填充审计字段）

```json
{
  "run_id": "20260728-160532",
  "mode": "execute",
  "runtime_authorization": { /* run 级，§10.1 */ },
  "cases": [
    {
      "case_id": "PC-M1-01-01",
      "steps": [ /* envelope 列表，--execute 模式含 op 级 runtime_authorization */ ],
      "cleanup": [ /* ST-EX-06 产出 */ ]
    }
  ],
  "logout": { /* ST-EX-07 产出 */ }
}
```

## 11. 性能

| 指标 | dry-run 模式 | --execute 模式 | 说明 |
|---|---|---|---|
| 单 step 延迟 | < 10ms（build_command + resolve） | op 间 2s 间隔 + op 执行 | HLD §13.1 |
| 24 用例全量 dry-run | < 30s（无网络，纯命令构建） | N/A（需 runtime 授权） | SM-EX-07 |
| subprocess 调用 | 0 次 | 每 step 1 次 | dry-run 门绕过 subprocess |
| memory | case_steps + envelopes 内存驻留 | 同 dry-run | 24 用例 × 10 step envelope 量级 < 1MB |

**性能约束**：dry-run 门不连设备，性能瓶颈在 YAML 解析 + build_command（CPU 密集，无 I/O 等待）。--execute 模式性能由 op_mapper subprocess + 网络延迟决定（HLD §13.1 目标 < 5 分钟，需 runtime 授权后验证，CP7 static-only 下不可直接验证）。

## 12. 安全

### 12.1 安全门控矩阵

| 门控 | 实现 | ADR/策略 |
|---|---|---|
| dry-run 默认门 | execute_steps dry_run=True 默认；不调 subprocess | ADR-02 / R-NF-001 / NO_PRODUCTION_WRITE |
| --execute 授权门 | authorize() 校验 --authorized；缺失 exit(2) | ADR-02 / R-F-007 |
| runtime_authorization 审计 | run 级（authorize）+ op 级（execute_op）双层 | R-NF-002 / RISK-CR033-DEVICE-WRITE |
| 凭据安全 | password_env 传环境变量名，不读凭据值 | NO_CREDENTIAL_READ / HLD §13.4 |
| session 路径 | ~/.local/state/ptm-atomic/，不写入仓库 | HLD §13.4 |

### 12.2 不授权范围（CR-033 §不授权范围）

- 设计通过不等于运行授权
- --execute 真机写操作属 NO_PRODUCTION_WRITE，CP7/运行时需单独 runtime_authorization（VALIDATION-ENV.yaml）
- ST-EX-05 实现授权门逻辑，但不提供真机 --execute 凭据/环境（运行时授权由用户在 CP7/运行时提供）

### 12.3 dry-run 门安全保证

dry_run=True 时：
- 0 次 subprocess 调用（不连设备）
- 0 次 ptm-atomic CLI 执行
- 0 次网络请求（resolve_env_refs 只解析参数，Gotcha #13）
- 仅 build_command 构造命令列表 + 打印，无副作用

## 13. 依赖

### 13.1 Story 依赖

| 依赖 Story | 类型 | 依赖内容 | gate |
|---|---|---|---|
| ST-EX-04 | hard | case_runner.py main/argparse（--execute/--authorized flag 预留）/discover_cases/parse_case_file/resolve_addresses/result.json 结构 | cp5-approved |
| ST-EX-03 | hard | op_mapper.resolve_env_refs（${ENV.*} 解析）/execute_op env_topology 签名扩展 | cp5-approved |

### 13.2 模块依赖

| 模块 | 依赖点 | 版本/契约 |
|---|---|---|
| op_mapper.build_command | dry-run 门构造命令 | 现有 L707，dry_run=True 不加 --execute |
| op_mapper.execute_op | --execute 分支执行 | 现有 L1079，已支持 dry_run/authorized/runtime_authorization |
| op_mapper.resolve_env_refs | dry-run 门 ${ENV.*} 解析 | ST-EX-03 新增，公开函数 |
| op_mapper.resolve_step_refs | dry-run 门 ${STEP-N.id} 解析 | 现有 L889，公开函数 |

### 13.3 反向依赖（下游 Story）

| 下游 Story | 消费 ST-EX-05 产出 | 说明 |
|---|---|---|
| ST-EX-06 | execute_steps 返回的 envelopes | run_cleanup 逆序清理需 step envelopes（含 result_envelope.data 提取 id） |
| ST-EX-08 | execute_steps warming_up 强制 | apply_warming_up 在 execute_steps 前后注入 post_op（ST-EX-08 扩展 execute_steps 调度） |
| ST-EX-10 | execute_steps 返回的 envelopes | classify_result 四态分级消费 envelope.status/error_type |
| ST-EX-11 | execute_steps 返回的 envelopes | generate_report/diagnose_failure 消费 envelope 诊断字段 |

## 14. 开放项

### 14.1 待确认（LCQ）

| LCQ ID | 问题 | 推荐方案 | 备选 | 影响 | blocks_lld |
|---|---|---|---|---|---|
| LCQ-ST-EX-05-01 | case_runner dry-run 门（不连设备，不 subprocess）与 op_mapper.execute_op dry_run（仍 subprocess，ptm-atomic 本体不写）语义分层。case_runner dry_run=True 时是否绕过 execute_op subprocess，直接 build_command 打印？ | A：绕过 subprocess，直接 build_command + resolve_* 打印（HLD §13.3 "不连接设备"） | B：case_runner dry_run=True 时仍调 execute_op(dry_run=True)，subprocess 执行 ptm-atomic --dry-run | execute_steps 核心实现；dry-run 校验深度；是否依赖 ptm-atomic 安装版支持 --dry-run | false（本 LLD 按方案 A 实现） |
| LCQ-ST-EX-05-02 | dry-run 模式下 ${STEP-N.id} 引用如何处理？前序 step 未真执行，step-refs 无 real data，resolve_step_refs 解析失败 | A：dry-run 下 resolve_step_refs 仍尝试解析，step-refs 缺失则该 step 标 VALIDATION_FAILED（dry-run 校验反馈） | B：dry-run 下跳过 resolve_step_refs，${STEP-N.id} 占位符原样保留 | dry-run 校验覆盖度；dry-run envelope 命令含未解析占位符 | false（本 LLD 按方案 A 实现） |

### 14.2 继承开放项（HLD §20）

| O ID | 内容 | 状态 | 来源 |
|---|---|---|---|
| O-02 | fw_logout op 在 ptm-atomic 安装版是否暴露 | OPEN（安装前验证） | ST-EX-07 承载，ST-EX-05 不直接依赖 |

### 14.3 非阻塞观察项

| 观察 | 说明 |
|---|---|
| mutation op 判定 | HLD §13.3 称 "fw_login 属 mutation op"，但 op_mapper OP_METADATA.fw_login_web_management.side_effect="observation"。ST-EX-05 dry-run 门下所有 op 都不 subprocess（不连设备），mutation 判定不影响 dry-run 门行为；runtime_authorization 审计由 op_mapper execute_op 在 not dry_run 时对所有 op 记录（不限 mutation）。本 Story 不引入 mutation op 显式判定集合，避免与 OP_METADATA 真相源冲突。若后续需区分（如 dry-run 下仅 mutation op 跳过、observation op 仍执行），需新增 CR 修正 HLD §13.3 与 OP_METADATA 一致性。 |
| dry-run envelope status 值 | 本 LLD 定 status="dry_run"，与 op_mapper envelope status（success/error）不同。ST-EX-10 四态分级需识别 "dry_run" 状态（dry-run 模式不参与四态分级，仅静态校验）。 |

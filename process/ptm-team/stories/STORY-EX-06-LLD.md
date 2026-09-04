---
cr_id: "CR-033"
story_id: "STORY-EX-06"
title: "用例清理（逆序清理 mutation ops + step-refs 读取）"
artifact_type: "lld"
lld_policy: "full-lld"
feature_design_refs: ["docs/features/case-execution/DESIGN.md"]
feature_id: "FE-EX-02"
epic_id: "EP-EX-04"
wave: 2
priority: "P0"
tier: "Tier-S"
shared_fragments: []
depends_on: ["ST-EX-04", "ST-EX-05"]
dependency_type: "hard"
file_ownership:
  - "skills/case-execution/scripts/case_runner.py#run_cleanup"
source_hld: "docs/design/HLD-PTM-TE-EXEC.md"
source_adr: "docs/design/ARCHITECTURE-DECISION-PTM-TE-EXEC.md"
status: "draft"
author: "meta-dev"
created_at: "2026-07-28T16:10:00+08:00"
---

# ST-EX-06 LLD：用例清理（逆序清理 mutation ops + step-refs 读取）

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-28 | meta-dev | ST-EX-06 full-lld 初稿：14 章节齐全；逆序清理 mutation ops（复用 op_mapper.handle_rollback）+ step-refs 读取（step_envelopes 内存 + step_refs_dir fallback）；与 ST-EX-17 清理顺序契约；LCQ-ST-EX-06-01/02 待确认 |

## 0. 概述

本 LLD 是 ST-EX-06 用例清理（逆序清理 mutation ops + step-refs 读取）的 full-lld 设计证据，属于 FE-EX-02 case-execution 引擎 Feature 的 Wave 2 清理 Story，设计目标为承载 case_runner.py 用例后逆序清理逻辑，还原设备状态。

**需求溯源**：ST-EX-06 承载 R-F-015（用例后逆序清理还原设备状态）、R-F-017（清理失败不影响后续用例）、SM-EX-08（逆序清理覆盖全部 mutation op）等功能需求；详细需求映射见 §1.2 需求与成功标准映射。

**模块拆分**：case_runner.py 按职责拆分为 run_cleanup（逆序清理 mutation ops，复用 op_mapper.handle_rollback）函数锚点（ST-EX-06 拥有）；main 接入点（每用例 execute_steps 后调用，在 ST-EX-17 框架接口清理之前）；不改 op_mapper（handle_rollback/ROLLBACK_STRATEGY/_extract_inverse_id/_read_step_ref 已就绪）。

**代码结构**：详见 §2 文件影响范围。case_runner.py 新增 run_cleanup 函数 + main 接入点；不修改 op_mapper.py/HLD。

**技术细节**：逆序清理复用 op_mapper.handle_rollback API 按 ROLLBACK_STRATEGY.type 分派（§4.2，inverse_op/restore_snapshot/irreversible/manual_required/none/as_cleanup_skip）；step-refs 读取契约（§3.2，内存优先 + 文件 fallback）；与 ST-EX-17 清理顺序契约（§3.3，先 ST-EX-06 用例 ops 后 ST-EX-17 框架接口）；dry-run 短路（§4.1，authorized=False 返回空列表）等核心技术细节在各章节展开。

**Definition of Done (DoD)**：CP6 实现完成以 §6 测试设计的 11 单元测试（TC-EX-06-01..11）全通过为准；CP7 验证采用 static-only（DQ-01 推荐），--execute 真机清理需独立 runtime_authorization。DoD 详见 §6.2/§6.3 测试矩阵与逆序清理覆盖。

## 1. 工程依据

### 1.1 上游设计追溯

| 依据 | 章节 | 关键结论 |
|---|---|---|
| HLD-PTM-TE-EXEC v1.6 | §12.1 [5] 逐用例执行 | 每用例结束后"逆序清理 mutation ops" |
| HLD-PTM-TE-EXEC v1.6 | §12.3 环境解析层设计 | 用例后清理顺序：先 ST-EX-06 用例 case_steps mutation ops 逆序清理（run_cleanup），再 ST-EX-17 框架预配置接口逆序还原（preconfigure_dut_interfaces 的逆序清理）；确保用例写的 op 先回滚，框架预配置的接口后还原 |
| HLD-PTM-TE-EXEC v1.6 | §8 场景 2（SCN-EX-02） | 步骤 7：每用例结束后逆序清理 mutation ops |
| HLD-PTM-TE-EXEC v1.6 | §13.5 可维护性 | case_runner 用例发现/解析/执行编排分层清晰 |
| HLD-PTM-TE-EXEC v1.6 | §14 RISK-CR033-DEVICE-WRITE | --execute 模式触发真实设备写操作，缓解含逆序清理 |
| ADR-PTM-TE-EXEC v1.3 | ADR-02 dry-run 默认门 | dry-run 不执行写操作，故 dry-run 模式无需逆序清理（无 mutation op 实际执行） |
| CR-033 | §需求二 | 保留容错/重试/逆序清理/runtime_authorization |
| FEATURE-DESIGN-MATRIX v1.3 | ST-EX-06 行 | lld_policy=full-lld，trigger_reasons=逆序清理 mutation ops/step-refs 读取/回滚逻辑 |

### 1.2 需求与成功标准映射

| 需求 ID | 内容 | 本 Story 落点 |
|---|---|---|
| R-F-015 | 用例后逆序清理（还原设备状态） | run_cleanup 逆序调 handle_rollback |
| R-F-017 | 清理失败不影响后续用例 | run_cleanup 失败容错（单 op 失败不中断） |
| SM-EX-08 | 逆序清理覆盖全部 mutation op | run_cleanup 遍历全部已执行 step |

### 1.3 既有代码现状

| 组件 | 位置 | 现状 | ST-EX-06 关系 |
|---|---|---|---|
| `op_mapper.handle_rollback` | op_mapper.py#handle_rollback（L1522） | 已实现按 ROLLBACK_STRATEGY.type 回滚：inverse_op（执行 inverse_op）/restore_snapshot（按 pre_snapshot 恢复）/irreversible（不回滚）/manual_required（不回滚）/none（不回滚）；as_cleanup_skip 标记的 op 自动跳过 | ST-EX-06 run_cleanup 复用 handle_rollback，不重复实现回滚逻辑 |
| `op_mapper.ROLLBACK_STRATEGY` | op_mapper.py（L295） | 21 个 op 的回滚策略表；type=none/inverse_op/restore_snapshot/irreversible/manual_required | run_cleanup 按表调度，type=none/irreversible/manual_required/as_cleanup_skip 自动跳过 |
| `op_mapper._extract_inverse_id` | op_mapper.py（L1497） | 从 result_envelope.data 提取 id（policy_route_id/interface_id/id），供 inverse_op 构造 inverse_args | run_cleanup 传入 step_envelope 作 result_envelope |
| `op_mapper._read_step_ref` | op_mapper.py（L877） | 读取 step-refs/<step_id>.json（含 step_id/op_id/args/envelope） | run_cleanup fallback：step_envelopes 不完整时从 step_refs_dir 读取 |
| `op_mapper.execute_op` | op_mapper.py（L1079） | inverse_op 回滚通过 execute_op(dry_run=False, authorized=True) 执行 | handle_rollback 内部调用，run_cleanup 透传 authorized |

## 2. 文件影响范围

### 2.1 新增/修改文件

| 文件 | 函数/区段 | 动作 | 说明 |
|---|---|---|---|
| `skills/case-execution/scripts/case_runner.py` | `run_cleanup` | 新增 | 逆序清理本用例 mutation ops，复用 op_mapper.handle_rollback |
| `skills/case-execution/scripts/case_runner.py` | `main`（ST-EX-04 产出） | 修改 | 接入 run_cleanup()：每用例 execute_steps 后调用 |

### 2.2 不修改文件（边界声明）

| 文件 | 原因 |
|---|---|
| `skills/policy-route-execution/scripts/op_mapper.py` | handle_rollback/ROLLBACK_STRATEGY/_extract_inverse_id/_read_step_ref 已就绪；ST-EX-06 不改 op_mapper（ST-EX-07 新增 fw_logout 映射与本 Story 无关） |
| `docs/design/HLD-PTM-TE-EXEC.md` | 设计对象，meta-dev 不改 |

### 2.3 文件所有权冲突检查

- `case_runner.py#run_cleanup`：DEVELOPMENT-PLAN ST-EX-06 独占，与 ST-EX-05（#execute_steps/#authorize）、ST-EX-07（#cleanup_session）、ST-EX-08（#apply_warming_up）、ST-EX-17（#preconfigure_dut_interfaces）无重叠。
- 同文件多 Story 写入：case_runner.py 共享。ST-EX-06 仅新增 `run_cleanup` 函数 + main 接入点（每用例循环末尾调用 run_cleanup）。与 ST-EX-17 的清理顺序协调见 §3.3（ST-EX-06 先执行，ST-EX-17 后执行，两者在 main 中按序调用，无函数体冲突）。

## 3. 接口设计

### 3.1 run_cleanup() -- 逆序清理 mutation ops

```python
def run_cleanup(
    case_steps: List[dict],
    step_envelopes: List[dict],
    base_url: str,
    session_file: str,
    *,
    authorized: bool = False,
    step_refs_dir: str = "",
    timeout: int = 45,
    tg_timeout: int = 90,
) -> List[dict]:
    """逆序清理本用例已执行的 mutation ops，返回每个 op 的 rollback envelope 列表。

    清理顺序（HLD §12.3）：
    1. 本函数（ST-EX-06）：逆序清理 case_steps mutation ops
    2. ST-EX-17 preconfigure_dut_interfaces 逆序还原：框架预配置接口清理（由 ST-EX-17 承载，main 按序调用）

    逆序遍历 step_envelopes（最后执行的先清理）。对每个 step 调
    op_mapper.handle_rollback(op_id, args, base_url, session_file,
                              result_envelope=step_envelope, authorized=authorized)
    handle_rollback 按 ROLLBACK_STRATEGY.type 处理：
    - inverse_op: 执行 inverse_op（如 fw_config_policy_route -> fw_delete_policy_route）
    - restore_snapshot: 按 pre_snapshot 恢复（需 step_envelope.pre_snapshot，LCQ-ST-EX-06-02）
    - irreversible/manual_required/none/as_cleanup_skip: 自动跳过，返回 not_required/waived/skipped

    dry-run 模式（authorized=False 且上游 dry_run=True）：无 mutation op 实际执行，
    run_cleanup 直接返回空列表（无需清理）。

    Args:
        case_steps: 用例步骤列表（ST-EX-04 产出，含 op_id/args/step_id/target）
        step_envelopes: execute_steps 返回的 envelope 列表（ST-EX-05 产出，含 status/data）
        base_url: DUT Web 管理地址
        session_file: session-<run-id>.json 路径
        authorized: --execute 授权标记（dry-run 模式为 False）
        step_refs_dir: step-refs 目录（fallback：step_envelopes 不完整时从文件读取）
        timeout: DUT op 超时秒数
        tg_timeout: TG op 超时秒数

    Returns:
        List[rollback_envelope]，与 step_envelopes 逆序对应；跳过的 op 返回 not_required envelope
    """
```

**调用方向**：`main` -> `run_cleanup(...)` -> `List[rollback_envelope]`。
**调用时机**：每用例 execute_steps 完成后、下一用例开始前（HLD §12.1 [5] 末尾）。
**输入契约**：case_steps + step_envelopes（ST-EX-05 execute_steps 返回值）+ base_url/session_file + authorized（ST-EX-05 authorize 产出）。
**输出契约**：`List[rollback_envelope]`，每个含 op_id/step_name="rollback-<op_id>"/status/data/error_type；写入 result.json `cases[].cleanup`。
**后续衔接**：rollback envelopes 供 ST-EX-11 generate_report 统计清理结果 + ST-EX-10 四态分级（清理失败影响用例终态）。
**降级策略**：dry-run 模式无实际执行，返回空列表；restore_snapshot 缺 pre_snapshot 时标 cleanup_error 但不中断（LCQ-ST-EX-06-02）。
**调用方需同步修改范围**：`main`（ST-EX-04）在每用例循环末尾调用 run_cleanup()，在 ST-EX-17 preconfigure_dut_interfaces 清理之前。

### 3.2 step-refs 读取契约

| 数据源 | 用途 | 优先级 |
|---|---|---|
| `step_envelopes`（内存，execute_steps 返回值） | handle_rollback 的 result_envelope 参数（提取 inverse_op 所需 id） | 主（首选） |
| `step_refs_dir/<step_id>.json`（持久化，execute_op 落盘） | step_envelopes 不完整或跨进程恢复时 fallback | 备（fallback） |

**读取逻辑**：
1. run_cleanup 遍历 step_envelopes（内存），按 step_id 对齐 case_steps
2. 若 step_envelopes[i] 缺失或 status="error"（execute_op 未落盘 step-ref），尝试 `op_mapper._read_step_ref(step_refs_dir, step_id)` 读取持久化记录
3. 若两者均缺失，该 step 跳过清理，记录 cleanup_error="step envelope 未找到"

**id 提取**：handle_rollback 内部调 `_extract_inverse_id(result_envelope, op_id)` 从 envelope.data 提取 id（policy_route_id/interface_id/id），run_cleanup 不重复提取。

### 3.3 与 ST-EX-17 清理顺序契约

| 顺序 | 执行方 | 清理对象 | 时机 |
|---|---|---|---|
| 1 | ST-EX-06 run_cleanup | 用例 case_steps 的 mutation ops（如 fw_config_policy_route/fw_config_interface/tg_start_traffic_stream） | 每用例 execute_steps 后 |
| 2 | ST-EX-17 preconfigure_dut_interfaces 逆序还原 | 框架预配置的 DUT 接口（fw_update_interface，非用例 case_steps） | run_cleanup 后（每用例末尾或全部用例后，由 ST-EX-17 定） |

**契约约束**（HLD §12.3）：用例写的 op 先回滚（ST-EX-06），框架预配置的接口后还原（ST-EX-17）。确保用例 config 的 policy_route 先 delete，再还原框架预配置的 interface IP，避免 delete policy_route 时引用已删除的 interface。

**main 调用顺序**（ST-EX-04 main + ST-EX-06/17 接入）：
```
for case in cases:
    envelopes = execute_steps(case_steps, ...)          # ST-EX-05
    cleanup_envelopes = run_cleanup(case_steps, envelopes, ...)  # ST-EX-06（先）
    # ST-EX-17 框架预配置接口逆序还原（后，ST-EX-17 实现）
```

## 4. 核心流程

### 4.1 run_cleanup() 主流程

```
run_cleanup(case_steps, step_envelopes, base_url, session_file, authorized, step_refs_dir, ...)
  │
  ├─ [0] dry-run 模式短路
  │     if not authorized:
  │         return []                        # dry-run 无实际执行，无需清理
  │
  ├─ [1] 构造逆序清理队列
  │     # 对齐 case_steps 与 step_envelopes（按 step_id），逆序遍历
  │     cleanup_queue = []
  │     for i in range(len(case_steps) - 1, -1, -1):
  │         step = case_steps[i]
  │         envelope = step_envelopes[i] if i < len(step_envelopes) else None
  │         # fallback：envelope 缺失则读 step-refs
  │         if envelope is None and step_refs_dir:
  │             envelope = op_mapper._read_step_ref(step_refs_dir, step.get("step_id", ""))
  │             envelope = envelope.get("envelope") if envelope else None
  │         # 跳过未执行的 step（dry-run step 或 execute 失败未落盘）
  │         if envelope is None or envelope.get("status") not in ("success", "error"):
  │             continue
  │         cleanup_queue.append((step, envelope))
  │
  ├─ [2] 逐 op 逆序清理
  │     rollback_envelopes = []
  │     cleanup_errors = []
  │     for step, envelope in cleanup_queue:
  │         op_id = step["op_id"]
  │         args = step.get("args", {})
  │         step_timeout = tg_timeout if step.get("target") == "tg" else timeout
  │         try:
  │             rb_envelope = op_mapper.handle_rollback(
  │                 op_id, args, base_url, session_file,
  │                 result_envelope=envelope,
  │                 authorized=authorized,
  │                 timeout=step_timeout,
  │             )
  │             rollback_envelopes.append(rb_envelope)
  │         except Exception as e:
  │             # 失败容错：单 op 清理失败不中断后续清理
  │             err_envelope = {
  │                 "op_id": op_id, "step_name": f"rollback-{op_id}",
  │                 "status": "error", "data": {"reason": str(e)},
  │                 "error_type": "CLEANUP_FAILED",
  │             }
  │             rollback_envelopes.append(err_envelope)
  │             cleanup_errors.append({"step_id": step.get("step_id"), "op_id": op_id, "reason": str(e)})
  │
  └─ return rollback_envelopes
```

### 4.2 handle_rollback 分派逻辑（op_mapper 已实现，ST-EX-06 透传）

| ROLLBACK_STRATEGY.type | 行为 | 示例 op | rollback envelope |
|---|---|---|---|
| inverse_op | 执行 inverse_op（execute_op dry_run=False） | fw_config_policy_route -> fw_delete_policy_route | inverse_op envelope |
| restore_snapshot | 按 pre_snapshot 恢复（缺 pre_snapshot -> EXEC_FAILED） | fw_update_policy_route/fw_update_interface | restore envelope 或 EXEC_FAILED |
| irreversible | 不回滚，返回 waived | fw_reset_policy_route_hitcount | `{"rollback": "waived", "reason": ...}` |
| manual_required | 不回滚，返回 manual_required | tg_stop_traffic_stream | `{"rollback": "manual_required", ...}` |
| none | 不回滚，返回 not_required | fw_login_web_management/fw_verify_*/fw_capture_operation_log | `{"rollback": "not_required", ...}` |
| as_cleanup_skip=True | 作为清理动作不触发回滚 | fw_delete_policy_route/fw_delete_interface/tg_stop/tg_delete_template | `{"rollback": "skipped", ...}` |

### 4.3 inverse_op 的 id 解析流程

```
handle_rollback(op_id=fw_config_policy_route, result_envelope=step_envelope)
  │
  ├─ strategy = ROLLBACK_STRATEGY["fw_config_policy_route"]  # type=inverse_op, inverse_op_id=fw_delete_policy_route
  ├─ decl = _load_op_decl(op_id)                             # ptm-atomic show 声明
  ├─ inv_args = build_inverse_args(op_id, result_envelope, args, decl, ...)
  │     # 声明优先：按 rollback_strategy.id_source 4 模式构造
  │     # 无声明 -> 回退 _extract_inverse_id
  ├─ if not inv_args:
  │     rid = _extract_inverse_id(result_envelope, op_id)    # 从 data.policy_route_id 取
  │     if rid is not None: inverse_args["id"] = rid
  │     elif "id" in args: inverse_args["id"] = args["id"]
  └─ return execute_op("fw_delete_policy_route", inverse_args, ..., dry_run=False, authorized=True)
```

**step-refs 读取**：result_envelope 由 run_cleanup 从 step_envelopes（内存）传入。若 step_envelopes 缺失，run_cleanup 从 `step_refs_dir/<step_id>.json` 读取（§3.2 fallback）。

## 5. 异常处理

### 5.1 异常分类与处理矩阵

| 异常场景 | 检测点 | 处理 | rollback envelope error_type | 是否中断后续清理 |
|---|---|---|---|---|
| dry-run 模式（authorized=False） | run_cleanup [0] | 短路返回空列表 | N/A | N/A |
| step_envelope 缺失且 step-refs 文件不存在 | run_cleanup [1] | 跳过该 step，不清理 | N/A（跳过） | 否 |
| step_envelope.status 不在 (success, error) | run_cleanup [1] | 跳过（如 dry_run/envelope 未执行） | N/A | 否 |
| handle_rollback OP_NOT_FOUND（op_id 无回滚策略） | op_mapper L1554 | 返回 error envelope | OP_NOT_FOUND | 否 |
| handle_rollback restore_snapshot 缺 pre_snapshot | op_mapper L1615 | 返回 EXEC_FAILED | EXEC_FAILED | 否（LCQ-ST-EX-06-02） |
| handle_rollback inverse_op execute_op 失败 | op_mapper execute_op | 透传 execute_op envelope | 原始 error_type | 否 |
| handle_rollback 抛未预期异常 | run_cleanup [2] try/except | 构造 CLEANUP_FAILED envelope | CLEANUP_FAILED | 否 |
| fw_delete_policy_route 清理时 id 已不存在（幂等） | op_mapper execute_op | execute_op 返回 error，但视为期望状态（ST-EX-10 四态分级） | 透传 | 否 |

### 5.2 失败容错原则

- **单 op 清理失败不中断**：run_cleanup 捕获 handle_rollback 异常，记录 cleanup_errors，继续清理下一个 op（逆序）。
- **清理失败不影响后续用例**：run_cleanup 失败的 op 记入 result.json `cases[].cleanup_errors`，下一用例正常执行（R-F-017）。
- **清理失败影响用例终态**：ST-EX-10 四态分级将 cleanup 失败计入用例终态（如 step success 但 cleanup failed -> 用例可能降级为 ERROR 或 KNOWN_FAIL，由 ST-EX-10 定）。

### 5.3 前置校验

| 前置条件 | 校验时机 | 失败行为 |
|---|---|---|
| authorized 与 execute_steps 一致 | run_cleanup [0] | 不一致（execute_steps --execute 但 authorized=False）-> 视为 dry-run 短路 |
| session_file 存在（--execute 模式） | run_cleanup 调 handle_rollback 前 | session 失效 -> handle_rollback 的 execute_op STATE_INVALID 重连（op_mapper 处理） |
| step_envelopes 与 case_steps 长度对齐 | run_cleanup [1] | 不对齐 -> 按 step_id 匹配，缺失的 step 跳过 |

## 6. 测试设计

### 6.1 验证层与 validation_mode

| 验证层 | 方式 | 说明 |
|---|---|---|
| 单元测试 | Python assert + mock（不引入 pytest） | run_cleanup 纯函数测试，mock op_mapper.handle_rollback |
| 静态校验 | dry-run 模式短路验证 | dry-run 模式 run_cleanup 返回空列表（无清理） |
| 集成验证 | --execute 模式（需 runtime_authorization） | CP7 static review + dry-run；--execute 真机清理验证属运行时授权 |

`validation_mode=static-only`（CP7 DQ-01）：dry-run 短路 + 代码审查；--execute 真机清理不在 CP7 范围。

### 6.2 测试用例矩阵

| 用例 ID | 场景 | 输入 | 期望 | 覆盖需求 |
|---|---|---|---|---|
| TC-EX-06-01 | 逆序清理顺序 | 3 step：config_policy_route/config_interface/tg_start，--execute | handle_rollback 调用顺序：tg_start -> config_interface -> config_policy_route（逆序） | R-F-015 |
| TC-EX-06-02 | dry-run 模式短路 | authorized=False | 返回空列表，0 次 handle_rollback 调用 | ADR-02 |
| TC-EX-06-03 | inverse_op id 提取 | step_envelope.data.policy_route_id=42 | handle_rollback 调 fw_delete_policy_route，args.id=42 | §4.3 |
| TC-EX-06-04 | type=none 自动跳过 | step=fw_verify_policy_route（type=none） | rollback envelope `{"rollback": "not_required"}`，不调 execute_op | §4.2 |
| TC-EX-06-05 | as_cleanup_skip 跳过 | step=fw_delete_policy_route（as_cleanup_skip） | rollback envelope `{"rollback": "skipped"}` | §4.2 |
| TC-EX-06-06 | 单 op 清理失败不中断 | step2 handle_rollback 抛异常 | step2 标 CLEANUP_FAILED，step1/step3 仍清理 | R-F-017 |
| TC-EX-06-07 | step_envelope 缺失 fallback step-refs | step_envelopes[1]=None，step_refs_dir 有 STEP-002.json | 从 step-refs 读取 envelope，继续清理 | §3.2 |
| TC-EX-06-08 | step_envelope 与 step-refs 均缺失 | step_envelopes[1]=None，step_refs_dir 无文件 | 跳过该 step，记录 cleanup_error | §5.1 |
| TC-EX-06-09 | restore_snapshot 缺 pre_snapshot | step=fw_update_policy_route，envelope 无 pre_snapshot | rollback envelope error_type=EXEC_FAILED，不中断 | LCQ-ST-EX-06-02 |
| TC-EX-06-10 | TG op timeout 区分 | step=tg_start_traffic_stream（target=tg） | handle_rollback 调 execute_op timeout=90 | Gotcha#9 |
| TC-EX-06-11 | 与 ST-EX-17 清理顺序 | main 调用序列 | run_cleanup 先于 ST-EX-17 preconfigure_dut_interfaces 清理 | §3.3/HLD§12.3 |

### 6.3 逆序清理覆盖（SM-EX-08）

--execute 模式下，用例 case_steps 的全部 mutation op 应被逆序清理：
- inverse_op 类（fw_config_policy_route/fw_config_interface/tg_start_traffic_stream）：调 inverse_op 清理
- none 类（fw_login_web_management/fw_verify_*/fw_capture_operation_log/tg_config_interface/tg_apply_traffic_template）：自动跳过
- restore_snapshot 类（fw_update_policy_route/fw_update_interface）：需 pre_snapshot（LCQ-ST-EX-06-02）
- as_cleanup_skip 类（作为清理动作的 delete/stop）：跳过

## 7. 实施步骤

| 步骤 | 动作 | 文件/函数 | 验证 | 依赖 |
|---|---|---|---|---|
| 1 | 实现 `run_cleanup` dry-run 短路 + 逆序队列构造 | case_runner.py#run_cleanup [0][1] | TC-EX-06-01/02/07/08 | ST-EX-05 step_envelopes |
| 2 | 实现 `run_cleanup` handle_rollback 调用 + 失败容错 | case_runner.py#run_cleanup [2] | TC-EX-06-03/04/05/06/09/10 | op_mapper.handle_rollback |
| 3 | 接入 `main`（ST-EX-04 每用例循环末尾） | case_runner.py#main | run_cleanup 调用链 | ST-EX-04 main |
| 4 | ST-EX-17 清理顺序协调 | main 调用顺序：run_cleanup -> ST-EX-17 清理 | TC-EX-06-11 | ST-EX-17 |
| 5 | result.json cleanup 字段汇总 | main 写 result.json `cases[].cleanup` | cleanup envelopes 落盘 | ST-EX-04 result.json |
| 6 | 单元测试 11 用例 | case_runner.py 测试脚本 | TC-EX-06-01..11 | - |

### 7.1 完成准则（量化）

- `run_cleanup` 函数实现，类型注解齐全
- 逆序清理：handle_rollback 调用顺序与 case_steps 逆序一致
- dry-run 短路：authorized=False 时 0 次 handle_rollback 调用
- 失败容错：单 op 清理失败不中断，cleanup_errors 记录
- 11/11 单元测试用例通过
- 与 ST-EX-17 清理顺序契约明确（main 调用序列）

## 8. 回滚策略

### 8.1 Story 级回滚

| 场景 | 回滚动作 | 影响 |
|---|---|---|
| ST-EX-06 实现缺陷需回滚 | 移除 run_cleanup()，main 不调用清理 | 用例后无逆序清理，设备状态残留（mutation op 未还原）；--execute 模式下设备状态累积污染 |

### 8.2 运行时清理失败回滚

run_cleanup 本身是回滚机制，其失败处理：
- 单 op 清理失败 -> 记 cleanup_error，不重试（避免循环）
- 清理失败的 op 在 result.json 标记，由人工或后续用例设计处理
- 不引入"回滚的回滚"（避免无限递归）

### 8.3 dry-run 模式无需回滚

dry-run 模式无 mutation op 实际执行（ST-EX-05 dry-run 门），run_cleanup 短路返回空列表，无需回滚。

## 9. 平台差异

| 平台 | 适配点 | 说明 |
|---|---|---|
| WSL2/Linux | run_cleanup 纯 Python | 无平台特定系统调用；handle_rollback 的 subprocess 由 op_mapper 处理 |
| ptm-atomic CLI | inverse_op 通过 execute_op 调 ptm-atomic | handle_rollback 内部 execute_op(dry_run=False) 加 --execute flag |
| session 路径 | ~/.local/state/ptm-atomic/ | run_cleanup 透传 session_file，不直接操作 |

**跨平台一致性**：run_cleanup 逻辑纯 Python（逆序遍历 + handle_rollback 调用），跨平台一致。清理顺序与 ST-EX-17 契约在 main 中按序调用，无平台差异。

## 10. 数据模型

### 10.1 rollback envelope（handle_rollback 产出）

inverse_op 成功：
```json
{
  "op_id": "fw_delete_policy_route",
  "step_name": "rollback-fw_config_policy_route",
  "status": "success",
  "data": {"reason": "inverse_op 清理完成"},
  "error_type": "NONE",
  "diag_snapshot_ref": ""
}
```

type=none 跳过：
```json
{
  "op_id": "fw_verify_policy_route",
  "step_name": "rollback",
  "status": "success",
  "data": {"rollback": "not_required", "reason": "observation，只读，不回滚"},
  "error_type": "NONE",
  "diag_snapshot_ref": ""
}
```

清理失败（CLEANUP_FAILED）：
```json
{
  "op_id": "fw_config_policy_route",
  "step_name": "rollback-fw_config_policy_route",
  "status": "error",
  "data": {"reason": "handle_rollback 异常: ..."},
  "error_type": "CLEANUP_FAILED",
  "diag_snapshot_ref": ""
}
```

### 10.2 result.json cleanup 字段

```json
{
  "cases": [
    {
      "case_id": "PC-M1-01-01",
      "steps": [ /* ST-EX-05 execute_steps 产出 */ ],
      "cleanup": [
        /* ST-EX-06 run_cleanup 产出，逆序；含 inverse_op/none/skipped/failed envelope */
      ],
      "cleanup_errors": [
        {"step_id": "STEP-003", "op_id": "tg_start_traffic_stream", "reason": "..."}
      ]
    }
  ]
}
```

### 10.3 step-refs 文件结构（op_mapper._write_step_ref 产出，run_cleanup 读取）

```json
{
  "step_id": "STEP-002",
  "op_id": "fw_config_policy_route",
  "args": {"source_network": "OBJ-SRC-192", ...},
  "envelope": {
    "op_id": "fw_config_policy_route",
    "status": "success",
    "data": {"policy_route_id": 42},
    ...
  }
}
```

## 11. 性能

| 指标 | --execute 模式 | 说明 |
|---|---|---|
| 单 op 清理延迟 | inverse_op 一次 execute_op（op 间 2s 间隔） | HLD §13.1 |
| 10 step 用例清理 | < 30s（5 个 inverse_op × ~5s） | 多数 step 为 none/as_cleanup_skip（跳过，<1ms） |
| dry-run 模式 | < 1ms（短路返回） | 无 handle_rollback 调用 |
| step-refs 读取 | < 1ms/文件（本地 JSON） | 仅 fallback 时读取 |

**性能约束**：run_cleanup 性能瓶颈在 handle_rollback 的 inverse_op execute_op（subprocess + 网络）。none/as_cleanup_skip 类 op 跳过开销可忽略。24 用例全量清理（每用例 ~5 inverse_op）预估 < 2 分钟（--execute 模式，需 runtime 授权后验证）。

## 12. 安全

### 12.1 安全门控

| 门控 | 实现 | 说明 |
|---|---|---|
| dry-run 短路 | authorized=False 时 run_cleanup 返回空列表 | dry-run 无写操作，无需清理 |
| --execute 清理授权 | run_cleanup 透传 authorized 给 handle_rollback | handle_rollback 的 execute_op(dry_run=False, authorized=True) |
| 清理不引入新写风险 | inverse_op 是删除/还原（如 config -> delete） | 清理本身是写操作，但属于回滚（还原状态），authorized 已校验 |
| NO_PRODUCTION_WRITE | --execute 真机清理属运行时授权 | CP7 static-only 不执行真机清理 |

### 12.2 清理安全性

- inverse_op 清理（如 fw_delete_policy_route）是删除操作，但目的是还原用例前的设备状态，属安全回滚。
- 清理失败不导致设备状态进一步恶化（失败记录，不重试不递归）。
- 清理顺序保证用例 op 先回滚，框架预配置接口后还原（§3.3），避免引用冲突。

## 13. 依赖

### 13.1 Story 依赖

| 依赖 Story | 类型 | 依赖内容 | gate |
|---|---|---|---|
| ST-EX-04 | hard | case_runner.py main/parse_case_file（case_steps 结构）/result.json | cp5-approved |
| ST-EX-05 | hard | execute_steps 返回的 step_envelopes（含 status/data）/authorize 的 authorized | cp5-approved |

### 13.2 模块依赖

| 模块 | 依赖点 | 版本/契约 |
|---|---|---|
| op_mapper.handle_rollback | 逆序清理核心 | 现有 L1522，支持 inverse_op/restore_snapshot/irreversible/manual_required/none/as_cleanup_skip |
| op_mapper.ROLLBACK_STRATEGY | 回滚策略表 | 现有 L295，21 个 op；ST-EX-07 新增 fw_logout（type=none） |
| op_mapper._extract_inverse_id | inverse_op id 提取 | 现有 L1497 |
| op_mapper._read_step_ref | step-refs fallback 读取 | 现有 L877 |
| op_mapper.execute_op | inverse_op 执行 | 现有 L1079（handle_rollback 内部调用） |

### 13.3 反向依赖（下游 Story）

| 下游 Story | 消费 ST-EX-06 产出 | 说明 |
|---|---|---|
| ST-EX-10 | run_cleanup 返回的 rollback_envelopes | 四态分级：cleanup 失败影响用例终态（step success + cleanup failed -> ERROR/KNOWN_FAIL） |
| ST-EX-11 | rollback_envelopes + cleanup_errors | generate_report 统计清理结果，report.md 含清理记录 |
| ST-EX-17 | 清理顺序契约 | ST-EX-17 preconfigure_dut_interfaces 逆序还原在 run_cleanup 之后（main 调用顺序） |

## 14. 开放项

### 14.1 待确认（LCQ）

| LCQ ID | 问题 | 推荐方案 | 备选 | 影响 | blocks_lld |
|---|---|---|---|---|---|
| LCQ-ST-EX-06-01 | run_cleanup 清理范围：对所有已执行 op 逆序调 handle_rollback（让 type=none/irreversible/manual_required/as_cleanup_skip 自动跳过），还是只对 mutation op（side_effect != observation）调？ | A：对所有已执行 op 逆序调 handle_rollback，type=none/irreversible/manual_required/as_cleanup_skip 自动跳过（统一逻辑，复用 handle_rollback 分派） | B：只对 mutation op 调，observation op 跳过（减少调用，但需 case_runner 维护 mutation 判定） | run_cleanup 实现简洁性；handle_rollback 调用次数 | false（本 LLD 按方案 A 实现） |
| LCQ-ST-EX-06-02 | restore_snapshot 类 op（fw_update_policy_route/fw_update_interface）的 pre_snapshot 何时捕获？run_cleanup 需 pre_snapshot 才能恢复，但 pre_snapshot 应在 step 执行前捕获 | A：execute_steps（ST-EX-05）在 restore_snapshot 类 op 执行前调 fw_verify_* 捕获快照，存入 step envelope 的 pre_snapshot 字段；run_cleanup 从 step_envelope 读取 | B：run_cleanup 不支持 restore_snapshot，update 类 op 回滚由用例设计承担（记 cleanup_error） | pre_snapshot 捕获需 ST-EX-05 配合；update 类 op 回滚完整性 | false（本 LLD run_cleanup 支持 restore_snapshot 接口，pre_snapshot 捕获标 OPEN 待 ST-EX-05 协调；24 用例 case_steps 主要是 config 类 inverse_op，update 类较少） |

### 14.2 继承开放项

| O ID | 内容 | 状态 | 来源 |
|---|---|---|---|
| O-02 | fw_logout 暴露 | OPEN（安装前验证） | ST-EX-07 承载；ST-EX-06 run_cleanup 不清理 fw_logout（session 清理由 ST-EX-07 cleanup_session） |

### 14.3 非阻塞观察项

| 观察 | 说明 |
|---|---|
| 24 用例 case_steps 的 op 分布 | 24 用例 case_steps 主要是 config 类（fw_config_policy_route/fw_config_interface/tg_config_interface/tg_apply_traffic_template/tg_start_traffic_stream），inverse_op 清理完整覆盖。update 类（fw_update_policy_route/fw_update_interface）在 case_steps 中较少出现，restore_snapshot 的 pre_snapshot 缺口（LCQ-ST-EX-06-02）影响面有限。 |
| 清理与 fw_logout 的关系 | run_cleanup 清理用例 case_steps 的 mutation ops；fw_logout 是会话级清理（全部用例结束后），由 ST-EX-07 cleanup_session 承载，不在 run_cleanup 范围。main 调用顺序：全部用例 run_cleanup -> fw_logout cleanup_session。 |
| 幂等容错与清理 | handle_rollback 的 inverse_op execute_op 可能遇到"对象已不存在"（如 fw_delete_policy_route 时 id 已被前序清理删除），execute_op 返回 error 但视为期望状态（ST-EX-10 四态分级处理为 KNOWN_FAIL 或忽略）。run_cleanup 透传 envelope，不特殊处理。 |

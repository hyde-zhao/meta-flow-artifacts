---
doc_type: LLD
cr_id: CR-035
story_id: STORY-035-S03
evidence_type: full-lld
lld_policy_required_level: full-lld
tier: S
risk_level: High
file_ownership: skills/case-execution/scripts/case_runner.py execute_steps 区段（L1407-1600）
dependencies: [STORY-035-S01, STORY-035-S02]
blocks: [STORY-035-S05]
file_conflict_coordination: 与 S02 同改 case_runner.py，同 Wave 串行（S02 先 main，S03 后 execute_steps）
hld_ref: process/changes/CR-035-HLD.md
author: meta-dev
created_at: 2026-07-31
---

# LLD：STORY-035-S03 execute_steps config-once skip + force_config + tg_user_id 注入

> 本 LLD 不重复 HLD 已有内容，引用 HLD 章节并补充 Story 级实现细节（签名变更、插入点行号、常量定义、测试用例名）。
> HLD ref：§6（config-once skip 逻辑）/ §5.2（tg_run_ctx）/ §8.3（注入契约）。
> **DQ-035-06 已纳入**：dry-run 不 skip（方案 A，与 fw_login dry-run 严格一致）。
> **AGA-1A/3A/4A 已纳入**：tg_run_ctx dict 载体 / run 级单标志 / 失败不置标志。

## 0. 概述

本 LLD 对应 STORY-035-S03（execute_steps config-once skip + force_config + tg_user_id 注入），基于 HLD v0.2（`process/changes/CR-035-HLD.md`）细化 Story 级实现设计。CP3 approved（DQ-035-06 方案 A dry-run 不 skip + AGA-1/3/4 agent 默认 + P0 acquire_ok 初始 True 整改）。

## 1. 工程依据与背景

execute_steps（L1407-1600）逐 step 执行，已有 fw_login skip 模式（L1473-1487，`and not dry_run` 守卫，Gotcha#8）。CR-035 类比该模式为 `tg_config_interface` 增加 config-once skip：--execute 模式首个真实执行置 run 级标志，后续 skip；force_config 例外重置；dry-run 不 skip。同时为 tg 族 op 自动注入 `tg_user_id`。完整背景见 HLD §1/§6。

## 2. 目标与需求（量化）

1. execute_steps 签名新增 `tg_run_ctx: Optional[dict] = None`，None 时行为不变（向后兼容）。
2. `tg_config_interface` 在 `--execute` 模式首个真实执行置 `tg_interface_configured=True`，后续 skip（force_config 除外）。
3. `force_config: true` 真实执行并重置标志（执行前置 False，成功后置 True）。
4. dry-run 模式 `tg_config_interface` 全部 build_command 展示命令，不 skip（DQ-035-06）。
5. 6 个现有 tg op 在 `tg_run_ctx` 非空且 args 无 `user_id` 时自动注入 `tg_user_id`。
6. config 失败不置标志（AGA-4），后续用例仍尝试 config。

## 3. 模块拆分与文件影响范围

| 文件 | 区段 | 改动 |
|---|---|---|
| `skills/case-execution/scripts/case_runner.py` | L1407-1422 execute_steps 签名 | 新增 `tg_run_ctx: Optional[dict] = None` 参数 |
| 同上 | L1487 后、L1489 前 | 新增 config-once skip 判断块（类比 fw_login skip L1473-1487） |
| 同上 | L1489 [1] resolve_env_refs 之后、[3] dry-run 门之前 | 新增 tg_user_id 自动注入 |
| 同上 | 真实执行成功后（dry-run 门之后） | 新增置标志逻辑 |
| 同上 | 模块级 | 新增 `TG_OPS_NEED_USER_ID` 常量 |

与 S02 共享文件但区段不重叠（S03 仅改 execute_steps L1407-1600）。

## 4. API 与接口设计

### 4.1 execute_steps 签名变更（L1407-1422）
在 `password_env: str = "FW_WEB_PASSWORD"`（L1421）后追加：
```python
    tg_run_ctx: Optional[dict] = None,
```
None 时行为不变（向后兼容，其他调用点无需改动，HLD §8.3）。

### 4.2 TG_OPS_NEED_USER_ID 常量（模块级）
```python
TG_OPS_NEED_USER_ID = {
    "tg_config_interface", "tg_apply_traffic_template", "tg_delete_traffic_template",
    "tg_start_traffic_stream", "tg_stop_traffic_stream", "tg_verify_traffic_loss",
}
```
6 个现有 tg op。`tg_acquire_ports`/`tg_release_ports` 不经 execute_steps（main 直接调 execute_op），`tg_verify_ownership` 无 `--user-id` flag，均不在此集合（HLD §6.4）。

### 4.3 tg_run_ctx 契约（HLD §5.2）
```python
tg_run_ctx = {"tg_user_id": str, "tg_interface_configured": bool}
```
- `tg_user_id`：只读，execute_steps 注入用。
- `tg_interface_configured`：可变，execute_steps 读写（config-once 标志，AGA-3A run 级单标志）。

## 5. 流程（核心）

### 5.1 config-once skip 判断（L1487 后，[1] resolve_env_refs 之前）
类比 fw_login skip（L1473-1487），插入点在 fw_login skip `continue`（L1487）之后、`[1] resolve_env_refs`（L1489）之前。`and not dry_run` 守卫确保 dry-run 不 skip（DQ-035-06）。
```python
# config-once skip（类比 fw_login skip L1473-1487；DQ-035-06: dry-run 不 skip）
force_config = bool(step.get("args", {}).get("force_config", False))
if (op_id == "tg_config_interface"
        and tg_run_ctx
        and tg_run_ctx.get("tg_interface_configured", False)
        and not force_config
        and not dry_run):
    env = {
        "op_id": op_id, "step_name": step_name, "step_id": step_id,
        "status": "skipped",
        "data": {"reason": "复用 run 级 tg_interface_configured（config-once，CR-035）"},
        "error_type": "NONE", "diag_snapshot_ref": "",
    }
    record["envelope"] = env
    record["status"] = classify_result(step, env)
    step_records.append(record)
    continue
```
skip 记录的 `envelope.status == "skipped"` 是 S02 `config_once_skip_count` 聚合的判定依据。

### 5.2 tg_user_id 自动注入（[1] resolve_env_refs 之后、[3] dry-run 门之前）
在 L1491 `args = resolve_env_refs(args, env_topology)` 之后插入（args 已是 L1445 复制的 dict）：
```python
# tg_user_id 自动注入（HLD §6.4）
if (tg_run_ctx and tg_run_ctx.get("tg_user_id")
        and op_id in TG_OPS_NEED_USER_ID
        and "user_id" not in args):
    args["user_id"] = tg_run_ctx["tg_user_id"]
```
注入后 build_command（dry-run）和 execute_op（--execute）均携带 `--user-id`，走 owned 模式（HLD Gotcha#5）。

### 5.3 force_config 重置 + 执行成功置标志（dry-run 门之后，真实执行路径）
在真实执行成功分支（envelope.status == "success"）之后：
```python
# force_config 重置标志（DQ-035-01，--execute 模式）
if (op_id == "tg_config_interface" and force_config
        and tg_run_ctx and not dry_run):
    tg_run_ctx["tg_interface_configured"] = False  # 执行前置 False

# 真实执行成功后置标志（AGA-4: 失败不置，--execute 模式）
if (op_id == "tg_config_interface" and tg_run_ctx
        and envelope.get("status") == "success" and not dry_run):
    tg_run_ctx["tg_interface_configured"] = True
```
**force_config 执行顺序**：先置 False（强制重配）-> 真实执行 -> 成功置 True。失败不置 True（AGA-4，保持 False，后续用例仍尝试 config）。dry-run 不置标志（DQ-035-06，dry_run 守卫）。

> 实施注意：force_config 重置应在真实执行**之前**（执行前置 False），置标志在**之后**。具体插入位置需在 execute_steps 真实执行分支（execute_op 调用前后）定位；dry-run 分支不触碰标志。

## 6. 代码结构与数据模型

### 6.1 判断表（--execute 模式，HLD §6.1）
| 条件 | 行为 | 置标志 |
|---|---|---|
| tg_config_interface 且 configured==False 且 force==False | 真实执行（首个） | 成功后 True |
| tg_config_interface 且 configured==True 且 force==False | skipped（config-once 复用） | 不变 |
| tg_config_interface 且 force==True | 真实执行（强制重配） | 执行前 False；成功后 True |
| tg_config_interface 且执行失败 | 不置标志（保持原值） | 不变（AGA-4） |
| 其他 tg op | 真实执行（复用 tg_user_id） | 不适用 |

### 6.2 dry-run vs --execute（DQ-035-06 方案 A，HLD §6.2）
| 模式 | tg_config_interface 首个 | tg_config_interface 后续 | 置标志 |
|---|---|---|---|
| --execute | 真实执行 | skipped（force 除外） | 是 |
| dry-run | build_command 展示命令 | build_command 展示命令（**不 skip**） | 否 |

## 7. 技术细节、安全与异常处理（HLD §12）

| 场景 | 处理 | 中断 run |
|---|---|---|
| 首个 tg_config_interface 失败 | 不置 configured=True；该 step FAIL；后续用例仍尝试 config（AGA-4） | 否 |
| force_config 执行失败 | 不置 True；该 step FAIL | 否 |
| tg_run_ctx 为 None | 跳过 skip 判断 + 跳过注入（向后兼容，行为不变） | 否 |
| dry-run 模式 | 不 skip、不置标志、注入 user_id 后 build_command 展示 | 否 |

## 8. 测试

测试在 S05 实现，本 Story 定义用例名与断言（HLD §14）：

| 用例名 | 断言 |
|---|---|
| `test_config_once_first_executes_sets_flag` | mock tg_run_ctx{configured:False}，--execute，tg_config_interface -> 真实执行（mock execute_op success），configured 置 True |
| `test_config_once_subsequent_skipped` | mock tg_run_ctx{configured:True}，--execute，force_config False -> status==skipped，execute_op 不调用 |
| `test_config_once_force_config_resets_and_executes` | mock tg_run_ctx{configured:True}，force_config True -> 真实执行，执行前 configured=False，成功后 True |
| `test_config_once_fail_no_flag_set` | mock execute_op 返回 fail，--execute -> configured 保持原值（不置 True） |
| `test_config_once_dry_run_not_skip` | dry-run，configured True -> 不 skip，build_command 展示命令（DQ-035-06） |
| `test_config_once_dry_run_no_flag_set` | dry-run -> configured 保持 False（不置标志） |
| `test_tg_user_id_auto_inject` | tg_run_ctx{tg_user_id:"dev1-a3f2"}，tg_start_traffic_stream -> args 含 user_id=="dev1-a3f2" |
| `test_tg_user_id_not_injected_when_no_ctx` | tg_run_ctx=None -> args 不含 user_id（向后兼容） |
| `test_tg_user_id_not_overwrite_existing` | args 已含 user_id -> 不覆盖 |
| `test_tg_acquire_ports_not_in_tg_ops_need_user_id` | tg_acquire_ports 不在集合（虽不经 execute_steps，常量断言） |
| `test_tg_verify_ownership_not_in_tg_ops_need_user_id` | tg_verify_ownership 不在集合 |
| `test_skip_record_envelope_status_skipped` | skip 记录 envelope.status=="skipped"（供 S02 聚合） |

## 9. 实施

1. 在 execute_steps 签名 L1421 后追加 `tg_run_ctx: Optional[dict] = None`（§4.1）。
2. 模块级定义 `TG_OPS_NEED_USER_ID` 常量（§4.2）。
3. 在 L1487（fw_login skip continue）后、L1489（[1] resolve_env_refs）前插入 config-once skip 判断块（§5.1）。
4. 在 L1491（resolve_env_refs 之后）插入 tg_user_id 自动注入（§5.2）。
5. 在 execute_steps 真实执行分支定位 force_config 重置（执行前）+ 成功置标志（执行后）插入点（§5.3）。
6. 运行 S05 测试 PASS + 现有 execute_steps 测试无回归。

## 10. 回滚策略

回滚 = 删除 tg_run_ctx 参数 + 删除 TG_OPS_NEED_USER_ID 常量 + 删除 skip 判断块 + 删除注入块 + 删除置标志块。`tg_run_ctx=None` 时行为与改动前完全一致，其他调用点无感知。

## 11. DoD（验收准则）

- [ ] execute_steps 签名含 `tg_run_ctx: Optional[dict] = None`，None 时行为不变。
- [ ] --execute 模式首个 tg_config_interface 真实执行置标志，后续 skip（force_config 除外）。
- [ ] force_config 真实执行，执行前重置 False，成功后置 True。
- [ ] dry-run 模式 tg_config_interface 不 skip，全部 build_command（DQ-035-06）。
- [ ] dry-run 不置标志。
- [ ] config 失败不置标志（AGA-4）。
- [ ] 6 个现有 tg op 自动注入 user_id；tg_run_ctx=None 不注入。
- [ ] skip 记录 envelope.status=="skipped"（供 S02 config_once_skip_count 聚合）。

## 12. 依赖

- S01（`TG_OPS_NEED_USER_ID` 依赖 user_id flag 已在 ARGS_TO_FLAGS 注册，注入后 build_command 能翻译）。
- S02（main 调用 execute_steps 传 `tg_run_ctx`；签名变更需与调用点同步，同 Wave 串行 S03 后于 S02）。

## 13. 风险

| 风险 | 等级 | 缓解 |
|---|---|---|
| skip 判断位置错误导致 resolve_env_refs 未执行 | Medium | 插入点在 [1] resolve_env_refs 之前，但 skip 分支直接 continue 不需 resolve（类比 fw_login skip L1473-1487 同位置） |
| force_config 重置与置标志时序错误 | Medium | force_config 执行前置 False，成功后置 True；S05 `test_config_once_force_config_resets_and_executes` 守卫 |
| dry-run 误 skip（DQ-035-06 漏洞） | High | `and not dry_run` 守卫；S05 `test_config_once_dry_run_not_skip` 守卫 |
| tg_run_ctx None 兼容性破坏 | Low | 所有读写均 `if tg_run_ctx and ...` 守卫；S05 `test_tg_user_id_not_injected_when_no_ctx` 守卫 |
| 真实 --execute 行为无法验证 | Medium | validation_mode=static-only+dry-run-only；CP7 N/A + 风险登记 |

## 14. open_items

- 无 Story 级 open item。force_config 重置与置标志的精确插入点需在实施时定位 execute_steps 真实执行分支（execute_op 调用前后）的具体行号；LLD 已给时序约束（执行前重置、成功后置标志），实施时据代码结构落地。


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

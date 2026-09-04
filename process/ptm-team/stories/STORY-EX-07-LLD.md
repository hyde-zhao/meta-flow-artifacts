---
cr_id: "CR-033"
story_id: "STORY-EX-07"
title: "fw_logout 会话清理（op_mapper 映射 + case_runner cleanup 登出 + 降级）"
artifact_type: "lld"
lld_policy: "full-lld"
feature_design_refs: ["docs/features/case-execution/DESIGN.md"]
feature_id: "FE-EX-02"
epic_id: "EP-EX-03"
wave: 2
priority: "P0"
tier: "Tier-S"
shared_fragments: []
depends_on: ["ST-EX-04"]
dependency_type: "hard"
file_ownership:
  - "skills/policy-route-execution/scripts/op_mapper.py#OP_ID_TO_SUBCOMMAND.fw_logout"
  - "skills/policy-route-execution/scripts/op_mapper.py#ARGS_TO_FLAGS.fw_logout"
  - "skills/policy-route-execution/scripts/op_mapper.py#ROLLBACK_STRATEGY.fw_logout"
  - "skills/policy-route-execution/scripts/op_mapper.py#EXPECTED_OP_COUNT"
  - "skills/case-execution/scripts/case_runner.py#cleanup_session"
source_hld: "docs/design/HLD-PTM-TE-EXEC.md"
source_adr: "docs/design/ARCHITECTURE-DECISION-PTM-TE-EXEC.md"
status: "draft"
author: "meta-dev"
created_at: "2026-07-28T16:20:00+08:00"
---

# ST-EX-07 LLD：fw_logout 会话清理（op_mapper 映射 + case_runner cleanup 登出 + 降级）

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-28 | meta-dev | ST-EX-07 full-lld 初稿：14 章节齐全；op_mapper 新增 fw_logout 映射（四表 + OP_METADATA + EXPECTED_OP_COUNT 22）+ case_runner cleanup_session（fw_logout 登出 + 降级 session 文件清理，ADR-04）+ 安装前 show 验证（DQ-CP3-02）；LCQ-ST-EX-07-01 待确认 |

## 0. 概述

本 LLD 是 ST-EX-07 fw_logout 会话清理（op_mapper 映射 + case_runner cleanup 登出 + 降级）的 full-lld 设计证据，属于 FE-EX-02 case-execution 引擎 Feature 的 Wave 2 会话清理 Story，设计目标为承载 op_mapper fw_logout 映射新增与 case_runner session 清理，保证 session 不残留。

**需求溯源**：ST-EX-07 承载 R-F-016（fw_logout op session 登出）、R-F-017（清理失败不影响后续用例）、R-F-019（session 生命周期规则）、SM-EX-10（1 个 fw_logout op）、SM-EX-11（EXPECTED_OP_COUNT=22）等功能需求；详细需求映射见 §1.2 需求与成功标准映射。

**模块拆分**：op_mapper.py 新增 fw_logout 映射（OP_ID_TO_SUBCOMMAND/ARGS_TO_FLAGS/REQUIRED_FLAGS/ROLLBACK_STRATEGY/OP_METADATA 五表 + EXPECTED_OP_COUNT 21->22，ST-EX-07 拥有）；case_runner.py 新增 cleanup_session（fw_logout 登出 + 降级 session 文件清理）/check_fw_logout_available（安装前验证）函数锚点（ST-EX-07 拥有）。

**代码结构**：详见 §2 文件影响范围。op_mapper.py 新增 fw_logout 映射条目 + EXPECTED_OP_COUNT 改 22 + validate_mapping_consistency 同步；case_runner.py 新增 cleanup_session/check_fw_logout_available 两函数 + main 接入点。

**技术细节**：fw_logout 映射 (auth, logout) 参照 fw_login_web_management（§3.1），复用 op_mapper.execute_op API 执行；安装前 `ptm-atomic show fw_logout` 验证（§3.2/§4.1，DQ-CP3-02）；降级 os.remove(session_file)（§4.2，ADR-04，fw_logout 未暴露时兜底）；RA-004 风险缓解（§5.2，fw_logout 未暴露时降级 session 文件清理）；dry-run 短路（§4.2，authorized=False 返回 skipped）等核心技术细节在各章节展开。

**Definition of Done (DoD)**：CP6 实现完成以 §6 测试设计的 16 单元测试（TC-EX-07-01..16）全通过为准；CP7 验证采用 static-only（DQ-01 推荐），--execute 真机 fw_logout 需独立 runtime_authorization。DoD 详见 §6.2/§6.3 测试矩阵与 SM-EX-10/11 覆盖。

## 1. 工程依据

### 1.1 上游设计追溯

| 依据 | 章节 | 关键结论 |
|---|---|---|
| HLD-PTM-TE-EXEC v1.6 | §12.1 [6] fw_logout 会话清理 | op_mapper execute_op(fw_logout)；未暴露时降级清理 session 文件（DQ-02）；result.json 含 logout 状态 |
| HLD-PTM-TE-EXEC v1.6 | §12.4 install.py 规则块 | session 生命周期：fw_logout 登出 + session 文件清理 |
| HLD-PTM-TE-EXEC v1.6 | §13.4 安全 | session 路径 ~/.local/state/ptm-atomic/，不写入仓库目录 |
| HLD-PTM-TE-EXEC v1.6 | §14 RA-004 | fw_logout op 在 ptm-atomic 安装版未暴露，缓解=安装前验证 + 降级 session 文件清理（DQ-02） |
| HLD-PTM-TE-EXEC v1.6 | §成功标准 SM-EX-10 | 1 个 fw_logout op：op_mapper 含 fw_logout 映射 |
| HLD-PTM-TE-EXEC v1.6 | §成功标准 SM-EX-11 | op_id 覆盖 22 个：EXPECTED_OP_COUNT=22（21+fw_logout） |
| HLD-PTM-TE-EXEC v1.6 | Gotcha #1 | EXPECTED_OP_COUNT 必须从 21 改为 22，否则 validate_mapping_consistency 报错 |
| HLD-PTM-TE-EXEC v1.6 | Gotcha #4 | fw_logout op 可能未在 ptm-atomic 安装版暴露；安装前 `ptm-atomic show fw_logout` 验证；未暴露时降级 os.remove(session_file)，result.json 记录 logout=fallback_session_cleanup |
| HLD-PTM-TE-EXEC v1.6 | §7 UC-EX-09 | fw_logout 会话清理：op_mapper fw_logout 映射（auth logout）+ case_runner cleanup 登出 + 降级 session 文件清理 |
| ADR-PTM-TE-EXEC v1.3 | ADR-04 fw_logout op 实现与降级 | 推荐=op_mapper 映射 fw_logout -> (auth, logout) + 安装前验证 + 降级 session 文件清理；备选=强制 ptm-atomic 升级（不采用）；回退=ptm-atomic 升级暴露 fw_logout 后移除降级逻辑 |
| ADR-PTM-TE-EXEC v1.3 | DQ-CP3-02 | fw_logout op 在 ptm-atomic 安装版是否暴露：推荐安装前验证 + 降级 session 清理；回退=降级为 session 文件清理 |
| CR-033 | §决策定稿 | #4 补 op_mapper 新增 fw_logout op + case_runner cleanup 登出 |
| CR-033 | §量化成功标准 | 1 个新 fw_logout op；24/24 用例 md ARP 预热校验 |
| FEATURE-DESIGN-MATRIX v1.3 | ST-EX-07 行 | lld_policy=full-lld，trigger_reasons=op_mapper 新增 fw_logout 映射/EXPECTED_OP_COUNT 改/降级逻辑 |

### 1.2 需求与成功标准映射

| 需求 ID | 内容 | 本 Story 落点 |
|---|---|---|
| R-F-016 | fw_logout op（session 登出） | op_mapper OP_ID_TO_SUBCOMMAND 新增 fw_logout -> (auth, logout) |
| R-F-017 | 清理失败不影响后续用例 | cleanup_session 失败容错（降级 + 记录） |
| R-F-019 | session 生命周期规则（fw_logout 登出 + session 文件清理） | cleanup_session + 降级 os.remove |
| SM-EX-10 | 1 个 fw_logout op | op_mapper 含 fw_logout 映射 |
| SM-EX-11 | EXPECTED_OP_COUNT=22 | 21 + fw_logout |

### 1.3 既有代码现状（fw_login_web_management 映射作为 fw_logout 参照）

| 组件 | 位置 | 现状（fw_login_web_management） | ST-EX-07 fw_logout 参照 |
|---|---|---|---|
| `OP_ID_TO_SUBCOMMAND` | op_mapper.py L63 | `"fw_login_web_management": ("auth", "login")` | 新增 `"fw_logout": ("auth", "logout")` |
| `ARGS_TO_FLAGS` | op_mapper.py L98 | `{"username": "--username", "password_env": "--password-env"}` | 新增 `{}`（logout 无 args，session 由 --session-file 传递） |
| `REQUIRED_FLAGS` | op_mapper.py L264 | `[]` | 新增 `[]` |
| `ROLLBACK_STRATEGY` | op_mapper.py L296 | `{"type": "none", "reason": "observation，只读，建立 session，不回滚"}` | 新增 `{"type": "none", "reason": "会话清理 op，本身是清理动作，不回滚"}` |
| `OP_METADATA` | op_mapper.py L447 | `{"side_effect": "observation", "rollback": "", "idempotent": True}` | 新增 `{"side_effect": "observation", "rollback": "", "idempotent": True}` |
| `EXPECTED_OP_COUNT` | op_mapper.py L525 | `21` | 改为 `22` |
| `validate_mapping_consistency` | op_mapper.py L1675 | 校验三表一致 + OP_METADATA + auth 族子命令 | auth 族校验新增 logout action |
| `build_command` | op_mapper.py L707 | family="auth" -> subtree=["auth", action]；--session-file 已含 | fw_logout -> ["auth", "logout"]，无需改 build_command |
| `execute_op` | op_mapper.py L1079 | 通用执行，dry_run/authorized/runtime_authorization 已实现 | fw_logout 复用，无需改 execute_op |

## 2. 文件影响范围

### 2.1 新增/修改文件

| 文件 | 函数/区段 | 动作 | 说明 |
|---|---|---|---|
| `skills/policy-route-execution/scripts/op_mapper.py` | `OP_ID_TO_SUBCOMMAND` | 修改 | 新增 `"fw_logout": ("auth", "logout")` |
| `skills/policy-route-execution/scripts/op_mapper.py` | `ARGS_TO_FLAGS` | 修改 | 新增 `"fw_logout": {}` |
| `skills/policy-route-execution/scripts/op_mapper.py` | `REQUIRED_FLAGS` | 修改 | 新增 `"fw_logout": []` |
| `skills/policy-route-execution/scripts/op_mapper.py` | `ROLLBACK_STRATEGY` | 修改 | 新增 `"fw_logout": {"type": "none", ...}` |
| `skills/policy-route-execution/scripts/op_mapper.py` | `OP_METADATA` | 修改 | 新增 `"fw_logout": {"side_effect": "observation", ...}` |
| `skills/policy-route-execution/scripts/op_mapper.py` | `EXPECTED_OP_COUNT` | 修改 | 21 -> 22 |
| `skills/policy-route-execution/scripts/op_mapper.py` | `validate_mapping_consistency` | 修改 | auth 族子命令校验新增 logout（若有 auth 族硬编码校验） |
| `skills/case-execution/scripts/case_runner.py` | `cleanup_session` | 新增 | fw_logout 登出 + 降级 session 文件清理 |
| `skills/case-execution/scripts/case_runner.py` | `check_fw_logout_available` | 新增 | 安装前 `ptm-atomic show fw_logout` 验证（DQ-CP3-02） |
| `skills/case-execution/scripts/case_runner.py` | `main`（ST-EX-04 产出） | 修改 | 接入 cleanup_session()：全部用例执行后调用 |

### 2.2 不修改文件（边界声明）

| 文件 | 原因 |
|---|---|
| `docs/design/HLD-PTM-TE-EXEC.md` | 设计对象，meta-dev 不改 |
| `docs/design/ARCHITECTURE-DECISION-PTM-TE-EXEC.md` | ADR 已定稿 ADR-04，meta-dev 不改 |
| `script/ptm_team/install.py` | 安装器由 ST-EX-02/04 负责（session 生命周期规则块由 ST-EX-02 承载） |

### 2.3 文件所有权冲突检查

- `op_mapper.py#OP_ID_TO_SUBCOMMAND.fw_logout` / `#ARGS_TO_FLAGS.fw_logout` / `#ROLLBACK_STRATEGY.fw_logout` / `#EXPECTED_OP_COUNT`：DEVELOPMENT-PLAN ST-EX-07 独占。与 ST-EX-03（#_build_exec_env/#resolve_env_refs，不同函数）、既有 21 op 映射（不同 key）无冲突。
- `case_runner.py#cleanup_session`：DEVELOPMENT-PLAN ST-EX-07 独占，与 ST-EX-05（#execute_steps/#authorize）、ST-EX-06（#run_cleanup）、ST-EX-08（#apply_warming_up）无重叠。
- 同文件多 Story 写入：op_mapper.py 由 ST-EX-03（_build_exec_env/resolve_env_refs）+ ST-EX-07（fw_logout 映射）共享。ST-EX-07 仅新增 fw_logout 相关常量条目 + EXPECTED_OP_COUNT 值修改，不修改 ST-EX-03 的函数体。case_runner.py 由 Wave 2-3 多 Story 共享，ST-EX-07 仅新增 cleanup_session/check_fw_logout_available 两个函数 + main 接入点（全部用例循环末尾）。

## 3. 接口设计

### 3.1 op_mapper fw_logout 映射（四表 + OP_METADATA + EXPECTED_OP_COUNT）

```python
# op_mapper.py 修改（新增条目）

# OP_ID_TO_SUBCOMMAND（L63，auth 族新增）
"fw_logout": ("auth", "logout"),

# ARGS_TO_FLAGS（L97，新增；logout 无 args，session 由 build_command 的 --session-file 传递）
"fw_logout": {},

# REQUIRED_FLAGS（L264，新增）
"fw_logout": [],

# ROLLBACK_STRATEGY（L295，新增；fw_logout 是清理动作，不回滚）
"fw_logout": {
    "type": "none",
    "reason": "会话清理 op，本身是清理动作，不回滚",
},

# OP_METADATA（L411，新增；参照 fw_login_web_management）
"fw_logout": {
    "side_effect": "observation",
    "rollback": "",
    "idempotent": True,
},

# EXPECTED_OP_COUNT（L525）
EXPECTED_OP_COUNT = 22  # 21 + fw_logout（Gotcha #1 同步）
```

**映射契约**：
- `map_op_id_to_subcommand("fw_logout")` -> `("auth", "logout")`
- `build_command("fw_logout", {}, base_url, session_file, dry_run=False)` -> `["ptm-atomic", "run", "--base-url", <url>, "--session-file", <path>, "--format", "json", "auth", "logout", "--execute"]`
- `handle_rollback("fw_logout", ...)` -> `{"rollback": "not_required", "reason": "会话清理 op..."}`（type=none）
- `validate_mapping_consistency()` -> PASS（四表 + OP_METADATA 一致，22 个 op）

### 3.2 check_fw_logout_available() -- 安装前验证（DQ-CP3-02）

```python
def check_fw_logout_available() -> bool:
    """安装前验证 ptm-atomic 是否暴露 fw_logout op（DQ-CP3-02）。

    执行 `ptm-atomic show fw_logout`：
    - returncode == 0 且 stdout 含 "fw_logout" -> True（暴露）
    - 否则 -> False（未暴露，降级）

    Returns:
        True（fw_logout 可用）/ False（未暴露，需降级）
    """
```

**调用方向**：`main`（启动校验阶段）-> `check_fw_logout_available()` -> bool。
**调用时机**：case_runner 启动校验（HLD §12.1 [1]），which ptm-atomic 后。
**输入契约**：无参数（调用 ptm-atomic CLI）。
**输出契约**：bool，写入 result.json `fw_logout_available` 字段。
**降级策略**：ptm-atomic 命令不存在 / 超时 -> False（降级）。
**调用方需同步修改范围**：`main`（ST-EX-04）启动校验调用 check_fw_logout_available()，结果透传 cleanup_session()。

### 3.3 cleanup_session() -- fw_logout 登出 + 降级 session 文件清理

```python
def cleanup_session(
    base_url: str,
    session_file: str,
    *,
    authorized: bool = False,
    fw_logout_available: Optional[bool] = None,
    timeout: int = 30,
) -> dict:
    """全部用例执行后清理 session：fw_logout 登出 + 降级 session 文件清理（ADR-04）。

    降级策略（ADR-04 / DQ-CP3-02）：
    - fw_logout_available=True（安装前 show 验证暴露）：调 op_mapper.execute_op(fw_logout)
    - fw_logout_available=False（安装前 show 验证未暴露）：降级 os.remove(session_file)
    - fw_logout_available=None（未预验证）：尝试 execute_op(fw_logout)，
      OP_NOT_FOUND/EXEC_FAILED 时降级 os.remove(session_file)

    dry-run 模式（authorized=False）：未预登录，无 session 文件，跳过返回 skipped。

    Args:
        base_url: DUT Web 管理地址
        session_file: session-<run-id>.json 路径
        authorized: --execute 授权标记（dry-run 为 False）
        fw_logout_available: check_fw_logout_available() 结果（None=未预验证）
        timeout: fw_logout 超时秒数

    Returns:
        dict: {"logout": "success"|"fallback_session_cleanup"|"failed"|"skipped",
               "envelope": <fw_logout envelope or None>,
               "reason": str}
    """
```

**调用方向**：`main` -> `cleanup_session(...)` -> dict（写入 result.json `logout`）。
**调用时机**：全部用例执行 + run_cleanup 完成后（HLD §12.1 [6]，最后一步）。
**输入契约**：base_url/session_file（ST-EX-04 resolve_addresses 产出）+ authorized（ST-EX-05 authorize 产出）+ fw_logout_available（check_fw_logout_available 产出）。
**输出契约**：dict，含 logout 状态 + envelope + reason；写入 result.json 顶层 `logout`。
**后续衔接**：result.json `logout` 字段供 ST-EX-11 generate_report 统计会话清理结果。
**降级策略**：fw_logout 未暴露 -> os.remove(session_file)；os.remove 失败 -> logout=failed。
**调用方需同步修改范围**：`main`（ST-EX-04）在全部用例循环后调用 cleanup_session()。

## 4. 核心流程

### 4.1 check_fw_logout_available() 流程

```
check_fw_logout_available()
  │
  ├─ try:
  │     proc = subprocess.run(["ptm-atomic", "show", "fw_logout"],
  │                           capture_output=True, text=True, timeout=10)
  │     if proc.returncode == 0 and "fw_logout" in proc.stdout:
  │         return True                     # fw_logout 暴露
  │     else:
  │         return False                    # 未暴露（降级）
  ├─ except subprocess.TimeoutExpired:
  │     return False                        # 超时降级
  └─ except FileNotFoundError:
       return False                         # ptm-atomic 不存在降级
```

### 4.2 cleanup_session() 主流程

```
cleanup_session(base_url, session_file, authorized, fw_logout_available, timeout)
  │
  ├─ [0] dry-run 模式短路
  │     if not authorized:
  │         return {"logout": "skipped",
  │                 "envelope": None,
  │                 "reason": "dry-run 模式未预登录，无 session 文件"}
  │
  ├─ [1] fw_logout 可用性分支
  │     if fw_logout_available is False:
  │         # 安装前验证未暴露，直接降级
  │         return _fallback_session_cleanup(session_file, None,
  │                                          "安装前验证 fw_logout 未暴露（DQ-CP3-02）")
  │
  │     # fw_logout_available is True 或 None（未预验证）
  │     envelope = op_mapper.execute_op(
  │         "fw_logout", {}, base_url, session_file,
  │         step_name="cleanup_session",
  │         dry_run=False, authorized=True,
  │         timeout=timeout,
  │     )
  │
  ├─ [2] 判定 execute_op 结果
  │     if envelope.get("status") == "success":
  │         return {"logout": "success", "envelope": envelope,
  │                 "reason": "fw_logout 登出成功"}
  │
  │     elif envelope.get("error_type") in ("OP_NOT_FOUND", "EXEC_FAILED"):
  │         # fw_logout 未暴露或执行失败 -> 降级 session 文件清理（ADR-04）
  │         return _fallback_session_cleanup(
  │             session_file, envelope,
  │             f"fw_logout 执行失败（{envelope.get('error_type')}），降级清理 session 文件")
  │
  │     else:
  │         # 其他错误（如 STATE_INVALID 重连失败）
  │         return {"logout": "failed", "envelope": envelope,
  │                 "reason": f"fw_logout 失败: {envelope.get('error_type')}"}
  │
  └─ _fallback_session_cleanup(session_file, original_envelope, reason):
        try:
            if session_file and os.path.exists(session_file):
                os.remove(session_file)
            return {"logout": "fallback_session_cleanup",
                    "envelope": original_envelope,
                    "reason": reason}
        except OSError as e:
            return {"logout": "failed", "envelope": original_envelope,
                    "reason": f"session 文件清理失败: {e}"}
```

### 4.3 fw_logout 命令构建（build_command 复用）

```
build_command("fw_logout", {}, base_url, session_file, dry_run=False)
  │
  ├─ family, action = map_op_id_to_subcommand("fw_logout")  # ("auth", "logout")
  ├─ flags = map_args_to_flags("fw_logout", {})             # []（空 args）
  ├─ _check_required_flags("fw_logout", [])                 # PASS（required=[]）
  ├─ validate_args("fw_logout", {})                         # PASS
  ├─ subtree = ["auth", "logout"]                           # family != "tg"
  └─ command = ["ptm-atomic", "run", "--base-url", base_url,
                "--session-file", session_file, "--format", "json",
                "auth", "logout", "--execute"]              # dry_run=False 加 --execute
```

### 4.4 main 调用顺序（ST-EX-04 main + ST-EX-07 接入）

```
main(args)
  │
  ├─ [1] 启动校验
  │     ├─ which ptm-atomic
  │     ├─ devices.yaml 校验
  │     └─ fw_logout_available = check_fw_logout_available()   # ST-EX-07 新增
  │
  ├─ [2] authorize(args) -> runtime_auth（ST-EX-05）
  │
  ├─ [3] 预登录（--execute 模式，ST-EX-05）
  │
  ├─ [4] 逐用例执行
  │     for case in cases:
  │         envelopes = execute_steps(...)                      # ST-EX-05
  │         run_cleanup(...)                                    # ST-EX-06
  │         # ST-EX-17 框架预配置接口清理（若 --execute）
  │
  └─ [5] fw_logout 会话清理
        logout_result = cleanup_session(base_url, session_file,
                                        authorized=runtime_auth is not None,
                                        fw_logout_available=fw_logout_available)  # ST-EX-07
        result.json["logout"] = logout_result
```

## 5. 异常处理

### 5.1 异常分类与处理矩阵

| 异常场景 | 检测点 | 处理 | logout 状态 | 是否中断 |
|---|---|---|---|---|
| dry-run 模式（authorized=False） | cleanup_session [0] | 短路返回 skipped | skipped | N/A |
| 安装前验证 fw_logout 未暴露（fw_logout_available=False） | cleanup_session [1] | 直接降级 os.remove | fallback_session_cleanup | 否 |
| fw_logout execute_op 成功 | cleanup_session [2] | 返回 success | success | N/A |
| fw_logout execute_op OP_NOT_FOUND | cleanup_session [2] | 降级 os.remove | fallback_session_cleanup | 否 |
| fw_logout execute_op EXEC_FAILED（ptm-atomic logout 未暴露） | cleanup_session [2] | 降级 os.remove | fallback_session_cleanup | 否 |
| fw_logout execute_op STATE_INVALID（session 失效） | op_mapper 自动重连 1 次 | 重连后重试 | 重连后状态 | 否 |
| fw_logout execute_op 其他错误 | cleanup_session [2] else | 返回 failed | failed | 否 |
| os.remove session 文件不存在 | _fallback_session_cleanup | 不报错（已清理） | fallback_session_cleanup | 否 |
| os.remove 权限不足/OSError | _fallback_session_cleanup | 返回 failed | failed | 否 |
| check_fw_logout_available 超时 | check_fw_logout_available | 返回 False（降级） | N/A（影响 [1]） | 否 |
| check_fw_logout_available ptm-atomic 不存在 | check_fw_logout_available | 返回 False（降级） | N/A | 否 |

### 5.2 降级链路（ADR-04）

```
fw_logout 可用？
  ├─ 是（安装前 show 验证）-> execute_op(fw_logout)
  │     ├─ success -> logout=success
  │     ├─ OP_NOT_FOUND/EXEC_FAILED -> 降级 os.remove
  │     └─ 其他错误 -> logout=failed
  ├─ 否（安装前 show 验证未暴露）-> 降级 os.remove
  └─ 未预验证（None）-> execute_op(fw_logout) 运行时探测
        ├─ success -> logout=success
        └─ OP_NOT_FOUND/EXEC_FAILED -> 降级 os.remove
```

**降级保证**：无论 fw_logout 是否暴露，session 文件最终被清理（os.remove 兜底），避免 session 残留。

### 5.3 前置校验

| 前置条件 | 校验时机 | 失败行为 |
|---|---|---|
| ptm-atomic 已安装（which ptm-atomic） | main [1] 启动校验 | 缺失 -> exit(1)（ST-EX-04 承载） |
| session_file 路径有效（--execute 模式） | cleanup_session 调用前 | 预登录失败 -> session 未建立，cleanup_session 仍尝试 os.remove（文件可能不存在，不报错） |
| base_url 有效 | main [1] | 缺失 -> exit(1)（ST-EX-04） |

## 6. 测试设计

### 6.1 验证层与 validation_mode

| 验证层 | 方式 | 说明 |
|---|---|---|
| 单元测试 | Python assert + mock（不引入 pytest） | check_fw_logout_available/cleanup_session 纯函数测试，mock op_mapper.execute_op/subprocess |
| 映射一致性 | op_mapper.validate_mapping_consistency() | 四表 + OP_METADATA 一致，EXPECTED_OP_COUNT=22 |
| 静态校验 | build_command("fw_logout") 命令构建 | 命令含 `auth logout`，dry_run 不加 --execute |
| 集成验证 | --execute 模式（需 runtime_authorization） | CP7 static review + dry-run；--execute 真机 fw_logout 属运行时授权 |

`validation_mode=static-only`（CP7 DQ-01）：映射一致性 + 命令构建 + 降级逻辑代码审查；--execute 真机 fw_logout 不在 CP7 范围。

### 6.2 测试用例矩阵

| 用例 ID | 场景 | 输入 | 期望 | 覆盖需求 |
|---|---|---|---|---|
| TC-EX-07-01 | fw_logout 四表映射 | OP_ID_TO_SUBCOMMAND/ARGS_TO_FLAGS/REQUIRED_FLAGS/ROLLBACK_STRATEGY | fw_logout 在四表均存在，值符合 §3.1 | R-F-016/SM-EX-10 |
| TC-EX-07-02 | OP_METADATA 一致 | OP_METADATA["fw_logout"] | side_effect=observation/rollback=""/idempotent=True | §3.1 |
| TC-EX-07-03 | EXPECTED_OP_COUNT=22 | EXPECTED_OP_COUNT | 值为 22；validate_mapping_consistency PASS | SM-EX-11/Gotcha#1 |
| TC-EX-07-04 | build_command fw_logout dry_run | dry_run=True | 命令含 `auth logout`，无 --execute | §4.3 |
| TC-EX-07-05 | build_command fw_logout --execute | dry_run=False | 命令含 `auth logout --execute` | §4.3 |
| TC-EX-07-06 | check_fw_logout_available 暴露 | mock subprocess returncode=0, stdout 含 fw_logout | 返回 True | DQ-CP3-02 |
| TC-EX-07-07 | check_fw_logout_available 未暴露 | mock subprocess returncode=1 | 返回 False | DQ-CP3-02 |
| TC-EX-07-08 | check_fw_logout_available 超时 | mock TimeoutExpired | 返回 False（降级） | §5.1 |
| TC-EX-07-09 | cleanup_session fw_logout 成功 | fw_logout_available=True, execute_op success | logout=success，envelope 非空 | R-F-016 |
| TC-EX-07-10 | cleanup_session 安装前降级 | fw_logout_available=False | logout=fallback_session_cleanup，os.remove 调用 | ADR-04 |
| TC-EX-07-11 | cleanup_session 运行时 OP_NOT_FOUND 降级 | fw_logout_available=None, execute_op OP_NOT_FOUND | logout=fallback_session_cleanup | ADR-04/Gotcha#4 |
| TC-EX-07-12 | cleanup_session dry-run 短路 | authorized=False | logout=skipped，0 次 execute_op | ADR-02 |
| TC-EX-07-13 | cleanup_session os.remove 失败 | session 文件权限不足 | logout=failed，reason 含清理失败 | §5.1 |
| TC-EX-07-14 | cleanup_session os.remove 文件不存在 | session_file 不存在 | logout=fallback_session_cleanup（不报错） | §5.1 |
| TC-EX-07-15 | handle_rollback fw_logout 跳过 | handle_rollback("fw_logout", ...) | 返回 not_required（type=none） | §3.1 |
| TC-EX-07-16 | validate_mapping_consistency 全表 | 22 个 op | PASS（四表 + OP_METADATA 一致） | Gotcha#1 |

### 6.3 SM-EX-10/11 覆盖

- SM-EX-10：op_mapper 含 fw_logout 映射（TC-EX-07-01 验证）
- SM-EX-11：EXPECTED_OP_COUNT=22（TC-EX-07-03/16 验证）

## 7. 实施步骤

| 步骤 | 动作 | 文件/函数 | 验证 | 依赖 |
|---|---|---|---|---|
| 1 | op_mapper 新增 fw_logout 映射（四表 + OP_METADATA） | op_mapper.py L63/L97/L264/L295/L411 | TC-EX-07-01/02 | - |
| 2 | EXPECTED_OP_COUNT 21 -> 22 | op_mapper.py L525 | TC-EX-07-03 | 步骤 1 |
| 3 | validate_mapping_consistency auth 族校验同步 | op_mapper.py L1675 | TC-EX-07-16 | 步骤 1/2 |
| 4 | 实现 `check_fw_logout_available` | case_runner.py#check_fw_logout_available | TC-EX-07-06/07/08 | - |
| 5 | 实现 `cleanup_session` + `_fallback_session_cleanup` | case_runner.py#cleanup_session | TC-EX-07-09/10/11/12/13/14 | op_mapper.execute_op |
| 6 | 接入 `main`（启动校验 check + 全部用例后 cleanup_session） | case_runner.py#main | TC-EX-07 调用链 | ST-EX-04 main |
| 7 | result.json logout 字段汇总 | main 写 result.json | logout_result 落盘 | ST-EX-04 result.json |
| 8 | 单元测试 16 用例 | case_runner.py + op_mapper.py 测试脚本 | TC-EX-07-01..16 | - |

### 7.1 完成准则（量化）

- fw_logout 在 OP_ID_TO_SUBCOMMAND/ARGS_TO_FLAGS/REQUIRED_FLAGS/ROLLBACK_STRATEGY/OP_METADATA 五处均存在，值符合 §3.1
- EXPECTED_OP_COUNT=22，validate_mapping_consistency PASS
- build_command("fw_logout") 生成正确命令（`auth logout` + --execute 可选）
- cleanup_session 三条路径覆盖：fw_logout 成功 / 降级 os.remove / dry-run 短路
- check_fw_logout_available 安装前验证（DQ-CP3-02）
- 16/16 单元测试用例通过

## 8. 回滚策略

### 8.1 Story 级回滚

| 场景 | 回滚动作 | 影响 |
|---|---|---|
| ST-EX-07 实现缺陷需回滚 | 移除 fw_logout 映射（四表 + OP_METADATA + EXPECTED_OP_COUNT 恢复 21）+ cleanup_session/check_fw_logout_available | 丧失 fw_logout op；session 清理无降级兜底（session 文件残留）；validate_mapping_consistency 恢复 21 |

### 8.2 降级路径不可回退

ADR-04 降级策略（os.remove session 文件）是兜底保证，不可移除。即使 fw_logout 映射回滚，session 文件清理仍应保留（避免 session 残留）。若 ST-EX-07 整体回滚，需在后续 CR 补充独立 session 清理逻辑。

### 8.3 ADR-04 回退条件

ADR-04 回退/切换条件：ptm-atomic 升级暴露 fw_logout 后，移除降级逻辑（os.remove fallback），仅保留 execute_op(fw_logout) 路径。此回退属后续 CR（ptm-atomic 升级后），不在 ST-EX-07 范围。

## 9. 平台差异

| 平台 | 适配点 | 说明 |
|---|---|---|
| WSL2/Linux | os.remove session 文件 | 跨平台一致；session 路径 ~/.local/state/ptm-atomic/ |
| ptm-atomic CLI | `ptm-atomic show fw_logout` 验证 | DQ-CP3-02；ptm-atomic 0.1.0 可能未暴露 logout |
| ptm-atomic CLI | `ptm-atomic run ... auth logout --execute` | build_command 复用现有 family="auth" 逻辑 |
| session 路径 | ~/.local/state/ptm-atomic/session-<run-id>.json | HLD §13.4；不写入仓库目录 |

**跨平台一致性**：fw_logout 映射是数据常量（无平台差异）；cleanup_session 逻辑纯 Python（subprocess + os.remove），跨平台一致。check_fw_logout_available 调用 ptm-atomic CLI，依赖 ptm-atomic 安装（which ptm-atomic 校验）。

## 10. 数据模型

### 10.1 fw_logout envelope（execute_op 产出）

成功：
```json
{
  "op_id": "fw_logout",
  "step_name": "cleanup_session",
  "status": "success",
  "data": {"reason": "logout 成功"},
  "error_type": "NONE",
  "diag_snapshot_ref": "",
  "runtime_authorization": {
    "who": "zhaohaibo",
    "scope": "fw_logout on https://10.113.55.51",
    "authorized_at": "2026-07-28T16:30:00+0800",
    "reason": "dry_run=False 用户单次授权（ADR-04 dry-run 默认门）"
  }
}
```

未暴露（OP_NOT_FOUND）：
```json
{
  "op_id": "fw_logout",
  "step_name": "cleanup_session",
  "status": "error",
  "data": {"reason": "未识别的 op_id: fw_logout，当前映射表覆盖 22 个 op_id"},
  "error_type": "OP_NOT_FOUND",
  "diag_snapshot_ref": ""
}
```

### 10.2 cleanup_session result（写入 result.json `logout`）

fw_logout 成功：
```json
{"logout": "success", "envelope": {/* §10.1 成功 */}, "reason": "fw_logout 登出成功"}
```

降级 session 文件清理：
```json
{"logout": "fallback_session_cleanup", "envelope": {/* §10.1 未暴露 */},
 "reason": "fw_logout 执行失败（OP_NOT_FOUND），降级清理 session 文件"}
```

dry-run 短路：
```json
{"logout": "skipped", "envelope": null, "reason": "dry-run 模式未预登录，无 session 文件"}
```

os.remove 失败：
```json
{"logout": "failed", "envelope": null, "reason": "session 文件清理失败: [Errno 13] Permission denied"}
```

### 10.3 result.json logout 字段

```json
{
  "run_id": "20260728-160532",
  "mode": "execute",
  "fw_logout_available": true,
  "runtime_authorization": {/* run 级 */},
  "cases": [/* ... */],
  "logout": {/* §10.2 */}
}
```

## 11. 性能

| 指标 | 场景 | 说明 |
|---|---|---|
| check_fw_logout_available | < 10s（subprocess 超时上限） | 启动校验阶段，单次 |
| fw_logout execute_op | < 5s（auth logout 单次） | --execute 模式，单次 |
| 降级 os.remove | < 1ms | 本地文件删除 |
| dry-run 短路 | < 1ms | authorized=False 直接返回 |
| validate_mapping_consistency | < 10ms | 22 个 op 校验 |

**性能约束**：cleanup_session 是单次操作（全部用例后），性能影响可忽略。check_fw_logout_available 在启动校验阶段执行（< 10s），不阻塞用例执行。fw_logout execute_op 的 subprocess + 网络延迟 < 5s（auth logout 轻量）。

## 12. 安全

### 12.1 安全门控

| 门控 | 实现 | 说明 |
|---|---|---|
| dry-run 短路 | authorized=False 时 cleanup_session 返回 skipped | dry-run 未预登录，无 session 文件 |
| --execute 授权 | cleanup_session 透传 authorized=True 给 execute_op | execute_op 内部校验 authorized |
| session 文件清理 | 降级 os.remove(session_file) | 避免 session 残留（含 token/cookie） |
| NO_CREDENTIAL_READ | cleanup_session 不读 session 文件内容 | 仅 os.remove，不读取/解析 session |
| session 路径 | ~/.local/state/ptm-atomic/ | 不写入仓库目录（HLD §13.4） |
| NO_PRODUCTION_WRITE | --execute 真机 fw_logout 属运行时授权 | CP7 static-only 不执行真机 fw_logout |

### 12.2 session 安全

- session 文件含登录 token/cookie，cleanup_session 通过 fw_logout 登出（服务端失效）+ os.remove（本地删除）双重清理。
- 降级路径（os.remove）保证 session 文件不残留，即使 fw_logout 未暴露。
- cleanup_session 不读取/打印 session 文件内容（NO_CREDENTIAL_READ）。

### 12.3 降级安全性

降级 os.remove(session_file) 是本地文件操作，不涉及网络/设备：
- 不连设备（不调 ptm-atomic CLI）
- 不读 session 内容（仅删除文件）
- 跨用户安全（session 文件在用户目录 ~/.local/state/ptm-atomic/）

## 13. 依赖

### 13.1 Story 依赖

| 依赖 Story | 类型 | 依赖内容 | gate |
|---|---|---|---|
| ST-EX-04 | hard | case_runner.py main/result.json/session_file 路径 | cp5-approved |

### 13.2 模块依赖

| 模块 | 依赖点 | 版本/契约 |
|---|---|---|
| op_mapper.OP_ID_TO_SUBCOMMAND | fw_logout 映射 | ST-EX-07 新增 |
| op_mapper.execute_op | fw_logout 执行 | 现有 L1079，dry_run/authorized/runtime_authorization 已实现 |
| op_mapper.build_command | fw_logout 命令构建 | 现有 L707，family="auth" 复用 |
| op_mapper.validate_mapping_consistency | 四表一致性校验 | 现有 L1675，EXPECTED_OP_COUNT=22 |
| op_mapper.handle_rollback | fw_logout 回滚（type=none） | 现有 L1522，自动跳过 |

### 13.3 反向依赖（下游 Story）

| 下游 Story | 消费 ST-EX-07 产出 | 说明 |
|---|---|---|
| ST-EX-11 | result.json `logout` 字段 | generate_report 统计会话清理结果，report.md 含 logout 状态 |
| ST-EX-02 | session 生命周期规则块 | install.py 规则块引用 fw_logout 登出 + session 文件清理（ST-EX-02 规则文本，ST-EX-07 实现） |

## 14. 开放项

### 14.1 待确认（LCQ）

| LCQ ID | 问题 | 推荐方案 | 备选 | 影响 | blocks_lld |
|---|---|---|---|---|---|
| LCQ-ST-EX-07-01 | fw_logout 未暴露检测时机：安装前 `ptm-atomic show fw_logout` 验证（DQ-CP3-02）vs 运行时 execute_op OP_NOT_FOUND 触发降级？ | A：双重保障--安装前 check_fw_logout_available 预验证（结果透传 cleanup_session）+ 运行时 execute_op OP_NOT_FOUND/EXEC_FAILED 兜底降级 | B：仅运行时探测（不预验证），execute_op OP_NOT_FOUND 触发降级 | check_fw_logout_available 实现；result.json fw_logout_available 字段；降级响应速度 | false（本 LLD 按方案 A 双重保障实现） |

### 14.2 继承开放项

| O ID | 内容 | 状态 | 来源 |
|---|---|---|---|
| O-02 | fw_logout op 在 ptm-atomic 安装版是否暴露 | OPEN（安装前验证） | ST-EX-07 承载；check_fw_logout_available + 降级兜底 |

### 14.3 非阻塞观察项

| 观察 | 说明 |
|---|---|
| op_id 命名不对称 | fw_logout（无 _web_management 后缀）vs fw_login_web_management（带后缀）。CR-033 决策定稿用 "fw_logout"（SM-EX-10/SM-EX-11/CR-033 §决策定稿一致）。命名不对称属 ADR-07（命名）范畴，本 Story 按决策实现 "fw_logout"。若后续需统一命名（如 fw_logout_web_management），属后续 CR，不影响 ST-EX-07 功能。 |
| fw_logout 无 args | fw_logout 不需 username/password_env（session 已建立，--session-file 传递 session）。ARGS_TO_FLAGS["fw_logout"]={}，build_command 不加额外 flag。与 fw_login_web_management（需 username/password_env）不同，符合语义。 |
| 降级与 ST-EX-06 run_cleanup 的关系 | run_cleanup 清理用例 case_steps 的 mutation ops（每用例后）；cleanup_session 清理 session（全部用例后）。两者独立，main 调用顺序：全部用例 run_cleanup -> cleanup_session。fw_logout 不在 run_cleanup 范围（fw_logout ROLLBACK_STRATEGY.type=none，且 fw_logout 由 cleanup_session 主动调用，非 case_steps）。 |
| ADR-04 回退条件 | ptm-atomic 升级暴露 fw_logout 后，可移除降级 os.remove 逻辑（仅保留 execute_op 路径）。此回退属后续 CR（ptm-atomic 升级后），ST-EX-07 保留降级逻辑保证当前可用性。 |

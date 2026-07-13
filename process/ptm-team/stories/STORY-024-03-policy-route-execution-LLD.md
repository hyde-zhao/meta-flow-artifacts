---
change_id: CR-024-ptm-te-agent
workflow_id: WF-PTM-TEAM-20260520-001
story_id: STORY-024-03
story_name: policy-route-execution skill + op_mapper
story_slug: policy-route-execution
lld_policy: full-lld
version: 1.0
hld_ref: process/HLD-CR-024.md
development_plan_ref: process/DEVELOPMENT-PLAN-CR-024.yaml
tier: M
wave: W2
dependencies: [STORY-024-01]
file_owners:
  - skills/policy-route-execution/SKILL.md
  - skills/policy-route-execution/scripts/op_mapper.py
evidence_type: full-lld
design_evidence_type: full-lld
lld_policy_required_level: full-lld
created_at: "2026-07-10T00:00:00+08:00"
author: host-orchestrator（inline-fallback，基于 HLD-CR-024 v1.1）
---

# LLD - STORY-024-03 policy-route-execution skill + op_mapper（CR-024）

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|------|------|--------|---------|
| v1.0 | 2026-07-10 | host-orchestrator | 初版。基于 HLD-CR-024 v1.1 §4 三层映射 + §9 回滚策略产出。锁定 8 个 op_id->子命令映射 + 7 个 op args->flag 映射 + 8 个 op rollback 策略；op_mapper.py 函数级设计（map_op_id_to_subcommand / map_args_to_flags / build_command / execute_op / handle_rollback / validate_mapping_consistency 等 10 个函数）；三层映射一致性校验三处真相源（run_policy_route.py + op yaml + CLI --help） |

---

## 0. 工程依据

本 LLD 的工程依据为 `process/HLD-CR-024.md`（v1.1）。架构决策与映射表真相源锁定于 HLD §4（op_id/args 三层映射）、§5（skill 边界）、§6（连接模型）、§7（凭据管理）、§8（login-once-reuse-session）、§9（inverse_op 回滚策略）。外部真相源：`ptm-atomic list`（rollback/side_effect 实测，2026-07-10）、`run_policy_route.py`（op_id->子命令硬编码）、op yaml `inputs.params`（参数名）、`ptm-atomic run --help`（flag）。CP2 决策（DQ-01 dry-run 默认门 / DQ-02 hg3250-51 / DQ-03 v1 不注入 rule block / DQ-04 手写最小 PC）已批准，CP3 HLD v1.1 + 5 ADR 已 approved。

## 1. Goal

本 Story 新建 `skills/policy-route-execution/` 目录，交付两个文件：

1. **`SKILL.md`**：策略路由执行 Skill 定义文档，包含双层映射表说明、干跑/执行/verify 三阶段流程、inverse_op 清理策略、错误表和 Gotchas，供 ptm-te agent 编排流程 [4] 调用。
2. **`scripts/op_mapper.py`**：核心 Python 脚本，承载 op_id->CLI 子命令（第一层）+ args->CLI flag（第二层）的双层映射翻译，提供 `execute_op`（执行单条原子操作）和 `handle_rollback`（回滚清理）两个主入口。

### 量化目标

| # | 目标 | 度量值 | 真相源 |
|---|------|--------|--------|
| G-1 | op_id->子命令映射覆盖 | 8 个 op_id 全覆盖 | HLD §4.3 + `run_policy_route.py` `build_subtree()` |
| G-2 | args->flag 映射覆盖 | 7 个 op 的 flag 映射全覆盖（config/update/delete/verify/priority/reset-hitcount/verify-hitcount）+ login | HLD §4.4 + `run_policy_route.py` `_add_*_args()` + op yaml `inputs.params` + CLI `--help` |
| G-3 | 三层映射一致性校验 | static 校验 PASS：op_mapper 映射表 vs `run_policy_route.py` + op yaml + CLI `--help` 三处真相源一致 | CP7 static |
| G-4 | dry-run 默认门 | `execute_op` 默认 `dry_run=True`；`dry_run=False`（`--execute`）需显式传入授权标记 | HLD §3.3 ADR-04 |
| G-5 | STATE_INVALID 重连 | 检测 `error_type=STATE_INVALID` 自动重新 `auth login` 并重试当前 op 1 次 | HLD §8.1 |
| G-6 | inverse_op 清理覆盖 | config->delete 清理覆盖；update->restore_snapshot；irreversible 豁免注明 | HLD §9.1 |
| G-7 | 未识别 op_id 阻塞 | 未在映射表中的 op_id 返回 `error_type=OP_NOT_FOUND`，不静默跳过 | HLD §3.1 降级策略 |

### Out of Scope（本 Story 不含）

- `agents/ptm-te.md` 编排流程实现（STORY-024-01）
- device-management / device-connection skill 实现（STORY-024-02）
- 安装器集成（STORY-024-04）
- `fw_config_batch_policy_route` package 级编排（T-05 candidate，不走 `run` 单条命令）
- 其他 op 族（interface/ospf 等）的映射（首期 N/A）
- 修改 ptm-tde 已交付基线 args 命名（不授权项）

---

## 2. Requirements

### 2.1 Functional Requirements

| # | 需求 ID | 需求描述 | 验收条件 | 真相源 |
|---|---------|---------|---------|--------|
| FR-1 | op_id 映射 | `op_mapper.py` 必须覆盖 8 个 op_id 到 CLI `family action` 子命令的映射 | `map_op_id_to_subcommand("fw_config_policy_route")` 返回 `("policy-route", "config")`；全部 8 个 op_id 映射与 `run_policy_route.py` `build_subtree()` 一致 | HLD §4.3 |
| FR-2 | args 映射 | `op_mapper.py` 必须覆盖 7 个 op 的 args->CLI flag 映射 + login | `map_args_to_flags("fw_config_policy_route", {"src_addr": "10.0.0.0/24", ...})` 输出含 `["--source-network", "10.0.0.0/24", ...]`；映射与 op yaml `inputs.params` + CLI `--help` 一致 | HLD §4.4 |
| FR-3 | 三层命名翻译 | ptm-tde PC args（`src_addr`）-> op yaml params（`source_network`）-> CLI flag（`--source-network`）的三层翻译由 op_mapper 承载 | ptm-tde args 命名不变；翻译表 centralize 在 `ARGS_TO_FLAGS` dict | HLD §4.2 ADR-01 |
| FR-4 | 命令构建 | `build_command` 组装完整的 `ptm-atomic run --base-url <url> <family> <action> [flags]` 命令列表 | 命令格式为嵌套子命令（非扁平 `ptm-atomic run <op_id>`）；共享参数 `--base-url`/`--session-file` 在 `run` 层级 | HLD §4.3 + 计划 §3.1 |
| FR-5 | 干跑默认 | `execute_op` 默认 `dry_run=True`（不加 `--execute`）；`dry_run=False` 时加 `--execute` | 默认输出 dry-run envelope；`--execute` 需显式授权标记 `authorized=True` | HLD §3.3 ADR-04 |
| FR-6 | envelope 封装 | 每条 op 执行结果封装为 envelope（op_id/step_name/status/data/error_type/diag_snapshot_ref） | envelope 字段完整；`error_type=NONE` 表示成功 | HLD §3.3 |
| FR-7 | STATE_INVALID 重连 | 检测 `error_type=STATE_INVALID` 自动重新 `auth login` 并重试当前 op（最多 1 次） | 重连后重试成功则 envelope status=success；重试仍失败则 status=error 且 error_type=STATE_INVALID | HLD §8.1 |
| FR-8 | 未识别 op_id | op_id 不在 `OP_ID_TO_SUBCOMMAND` 中时返回 `error_type=OP_NOT_FOUND` | 不静默跳过；envelope status=error；提示工具缺失反馈 ptm-tae | HLD §3.1 |
| FR-9 | inverse_op 清理 | `handle_rollback` 按 op 的 rollback 策略执行清理 | config -> 执行 delete（inverse_op）；update -> restore_snapshot 恢复；reset-hitcount -> irreversible 豁免注明不回滚 | HLD §9.1 |
| FR-10 | session 复用 | login 后 session 写入 `--session-file`，后续 op 复用同一 session | session 路径由调用方传入；op_mapper 不自建 session 管理逻辑 | HLD §8.1 |
| FR-11 | SKILL.md 文档 | SKILL.md 含映射表、干跑/执行/verify 流程、错误表、Gotchas、回滚策略 | 覆盖 HLD §16 全部 13 条 Gotchas 中与 policy-route 相关的 9 条（#1/#2/#3/#4/#5/#6/#7/#10/#11） | HLD §16 |

### 2.2 Non-Functional Requirements

| # | 需求 ID | 需求描述 | 验收条件 |
|---|---------|---------|---------|
| NFR-1 | Python 版本 | op_mapper.py 兼容 Python >=3.9（与 device-connection telnetlib 约束一致） | `python3.9 -c "import op_mapper"` 不报错；无 3.10+ 专有语法 |
| NFR-2 | 无外部依赖 | op_mapper.py 仅使用标准库（subprocess/json/sys/typing），不引入第三方包 | `import` 行无第三方包；`pyproject.toml` 无新增依赖 |
| NFR-3 | 映射表单点维护 | 全部映射逻辑集中在模块级 dict 常量，漂移时单点修正 | `OP_ID_TO_SUBCOMMAND` / `ARGS_TO_FLAGS` / `ROLLBACK_STRATEGY` 为模块级常量 |
| NFR-4 | 执行超时 | 单条 op 执行默认超时 30 秒（与 op yaml `timeout_ms: 30000` 一致） | `execute_op` 默认 `timeout=30`；超时返回 `error_type=EXEC_FAILED` |
| NFR-5 | 可测试性 | 映射函数纯函数化，`build_command` 不执行命令，`execute_op` 封装 subprocess 调用 | 映射函数可独立单元测试；`build_command` 返回 `List[str]` 不含副作用 |
| NFR-6 | 安全性 | 禁止命令行明文密码；密码经 `--password-env` 环境变量传递 | login 命令含 `--password-env FW_WEB_PASSWORD`，不含明文密码值 |
| NFR-7 | 错误可追溯 | 每条 op 执行记录写入 exec-log.jsonl，含命令、退出码、envelope | exec-log.jsonl 每行一条记录；字段完整 |
| NFR-8 | 幂等安全 | config op 的 idempotent=true，op_mapper 不重复执行已成功的 config | 由 ptm-atomic CLI 层幂等保证；op_mapper 不自建幂等逻辑 |

---

## 3. 模块拆分与职责

### 3.1 文件级模块

```
skills/policy-route-execution/
├── SKILL.md                    # Skill 定义文档（人类可读 + agent 消费）
└── scripts/
    └── op_mapper.py            # 核心映射脚本（双层映射 + 执行 + 回滚）
```

### 3.2 op_mapper.py 内部模块职责

| 模块区 | 职责 | 函数/常量 | 输入 | 输出 |
|--------|------|----------|------|------|
| 映射表常量 | 三层映射的真相源 | `OP_ID_TO_SUBCOMMAND` / `ARGS_TO_FLAGS` / `ROLLBACK_STRATEGY` / `OP_METADATA` | - | dict |
| 第一层映射 | op_id -> 子命令 | `map_op_id_to_subcommand()` | op_id: str | (family, action) tuple |
| 第二层映射 | args -> CLI flag | `map_args_to_flags()` | op_id: str, args: dict | List[str] |
| 命令构建 | 组装完整命令 | `build_command()` | op_id, args, base_url, session_file, dry_run | List[str] |
| 执行 | subprocess 调用 ptm-atomic | `execute_op()` | op_id, args, base_url, session_file, dry_run, authorized, timeout | dict (envelope) |
| session 重连 | STATE_INVALID 自动重连 | `_reconnect_and_retry()` | base_url, session_file, username, password_env, retry_fn | dict (envelope) |
| 回滚清理 | 按 rollback 策略清理 | `handle_rollback()` | op_id, args, base_url, session_file, pre_snapshot, authorized | dict (envelope) |
| 校验 | 映射表一致性 | `validate_mapping_consistency()` | - | ValidationResult |
| 日志 | 执行日志写入 | `_append_exec_log()` | log_path, record | None |
| 工具 | envelope 构建 | `_build_envelope()` | op_id, step_name, status, data, error_type, diag_ref | dict |

### 3.3 职责边界（相邻对象边界澄清）

| 职责 | 归属 | 差异界定 |
|------|------|---------|
| op_id/args 双层映射翻译 | **op_mapper.py**（本 Story） | centralize 三层命名翻译，其他模块不重复映射逻辑 |
| 设备 SSH/Telnet 连接 | device-connection（STORY-024-02） | op_mapper 不建立 SSH/Telnet 连接，只调用 ptm-atomic CLI（Web HTTPS） |
| 设备清单管理 | device-management（STORY-024-02） | op_mapper 不维护 devices.yaml，base_url/session_file 由调用方传入 |
| 用例解析 | agents/ptm-te.md（STORY-024-01） | op_mapper 不解析 PC 文件，op_id/args 由调用方从 case_steps 提取后传入 |
| 执行门控 | agents/ptm-te.md（STORY-024-01） | op_mapper 不实现环境就绪/关键判定门控，只提供执行和回滚能力 |
| 快照采集 | device-connection（STORY-024-02） | op_mapper 的 `diag_snapshot_ref` 引用由 device-connection 采集的快照路径 |
| ptm-atomic CLI 本身 | ptm-atomic 仓库 | op_mapper 是 CLI 的消费者，不修改 ptm-atomic |

---

## 4. 代码结构与文件影响范围

### 4.1 文件影响范围

| 文件 | 操作 | 行数估计 | 说明 |
|------|------|---------|------|
| `skills/policy-route-execution/SKILL.md` | 新建 | ~250 行 | Skill 定义：frontmatter + 目标 + 双层映射表 + 干跑/执行/verify 流程 + 错误表 + Gotchas + 回滚策略 |
| `skills/policy-route-execution/scripts/op_mapper.py` | 新建 | ~450 行 | 核心脚本：映射表常量 + 10 个函数 + `__main__` CLI 入口 |

**文件总数：2 个（均为新建）。不修改任何现有文件。**

### 4.2 SKILL.md 章节结构

```markdown
---
name: policy-route-execution
description: 通过 ptm-atomic CLI 执行策略路由原子操作...
argument-hint: "<op_id> [--dry-run | --execute]"
user-invokable: true
status: active
---

# policy-route-execution

## 目标
## 前置条件
  - ptm-atomic 已安装且已 sync
  - auth login 已完成，session.json 可用
## 双层映射表
  ### 第一层：op_id -> CLI 子命令（8 个）
  ### 第二层：args -> CLI flag（7 个 op + login）
## 执行流程（干跑 -> 执行 -> verify）
## 回滚策略（inverse_op / restore_snapshot / irreversible 豁免）
## 错误表
## Gotchas
## 参数说明
```

### 4.3 op_mapper.py 代码结构

```python
#!/usr/bin/env python3
"""op_mapper.py - 策略路由双层映射 + 执行 + 回滚。

真相源锁定三处：
1. run_policy_route.py build_subtree() - 子命令名
2. op yaml inputs.params - 参数名
3. ptm-atomic run ... --help - CLI flag
"""

# ===== 映射表常量 =====
OP_ID_TO_SUBCOMMAND: dict[str, tuple[str, str]]  # 8 个
ARGS_TO_FLAGS: dict[str, dict[str, str]]          # 8 个 op
ROLLBACK_STRATEGY: dict[str, dict]                # 8 个 op
OP_METADATA: dict[str, dict]                      # side_effect / idempotent / rollback

# ===== 第一层映射 =====
def map_op_id_to_subcommand(op_id: str) -> tuple[str, str]: ...

# ===== 第二层映射 =====
def map_args_to_flags(op_id: str, args: dict) -> list[str]: ...

# ===== 命令构建 =====
def build_command(op_id, args, base_url, session_file, dry_run=True) -> list[str]: ...

# ===== 执行 =====
def execute_op(op_id, args, base_url, session_file, *, step_name="",
               dry_run=True, authorized=False, timeout=30,
               username="admin", password_env="FW_WEB_PASSWORD",
               exec_log_path=None) -> dict: ...

# ===== session 重连 =====
def _reconnect_and_retry(base_url, session_file, username, password_env,
                         retry_command, timeout) -> dict: ...

# ===== 回滚清理 =====
def handle_rollback(op_id, args, base_url, session_file, *,
                    pre_snapshot=None, authorized=False, timeout=30) -> dict: ...

# ===== 校验 =====
def validate_mapping_consistency() -> "ValidationResult": ...

# ===== 工具函数 =====
def _build_envelope(op_id, step_name, status, data, error_type, diag_ref) -> dict: ...
def _append_exec_log(log_path, record) -> None: ...
def _parse_atomic_output(stdout: str) -> dict: ...

# ===== CLI 入口 =====
if __name__ == "__main__": ...
```

### 4.4 不涉及的文件

- `agents/ptm-te.md`（STORY-024-01）
- `skills/device-management/**`（STORY-024-02）
- `skills/device-connection/**`（STORY-024-02）
- `script/ptm_team/install.py`（STORY-024-04）
- `skills/README.md`（STORY-024-04）
- ptm-atomic 仓库任何文件（外部依赖，不修改）

---

## 5. 数据模型与持久化设计

### 5.1 映射表数据结构（模块级常量，op_mapper.py 真相源）

#### 5.1.1 第一层映射：OP_ID_TO_SUBCOMMAND

```python
OP_ID_TO_SUBCOMMAND = {
    # auth 族
    "fw_login_web_management":        ("auth",         "login"),
    # policy-route 族（7 个，与 run_policy_route.py build_subtree() 一致）
    "fw_config_policy_route":         ("policy-route", "config"),
    "fw_update_policy_route":         ("policy-route", "update"),
    "fw_delete_policy_route":         ("policy-route", "delete"),
    "fw_verify_policy_route":         ("policy-route", "verify"),
    "fw_update_policy_route_priority":("policy-route", "priority"),
    "fw_reset_policy_route_hitcount": ("policy-route", "reset-hitcount"),
    "fw_verify_policy_route_hitcount":("policy-route", "verify-hitcount"),
}
```

真相源校验：`run_policy_route.py` `build_subtree()` 返回的 7 个 `CommandSpec(name=...)` + `auth login`（`run_auth.py`）。共 8 个，op_mapper 必须全覆盖。

#### 5.1.2 第二层映射：ARGS_TO_FLAGS

```python
ARGS_TO_FLAGS = {
    "fw_login_web_management": {
        "username":     "--username",
        "password_env": "--password-env",   # 默认值 "FW_WEB_PASSWORD"
    },
    "fw_config_policy_route": {
        "src_addr":     "--source-network",   # ptm-tde args -> op yaml source_network -> CLI --source-network
        "dst_addr":     "--dst-network",      # ptm-tde args -> op yaml dst_network -> CLI --dst-network
        "next_hop":     "--next-hop-ip",      # ptm-tde args -> op yaml next_hop_ip -> CLI --next-hop-ip
        "in_interface": "--in-interface",     # 三层同名（in_interface）
        "type":         "--policy-route-type",# ptm-tde args -> op yaml type -> CLI --policy-route-type
    },
    "fw_update_policy_route": {
        # 与 config 相同的 5 个 flag
        "src_addr":     "--source-network",
        "dst_addr":     "--dst-network",
        "next_hop":     "--next-hop-ip",
        "in_interface": "--in-interface",
        "type":         "--policy-route-type",
        # update 额外带 --id（从 verify 查询获取目标策略路由 id）
        "id":           "--id",
    },
    "fw_delete_policy_route": {
        "id":   "--id",
        "type": "--policy-route-type",
    },
    "fw_verify_policy_route": {
        "type": "--policy-route-type",
        "page": "--page",
        "size": "--size",
    },
    "fw_update_policy_route_priority": {
        "type":       "--policy-route-type",
        "moveid":     "--moveid",
        "targetid":   "--targetid",
        "targetsite": "--targetsite",
    },
    "fw_reset_policy_route_hitcount": {
        "id":   "--id",
        "type": "--policy-route-type",
    },
    "fw_verify_policy_route_hitcount": {
        "type": "--policy-route-type",
        "page": "--page",
        "size": "--size",
    },
}
```

三层命名不一致对照（真相源：HLD §4.1 + op yaml `inputs.params`）：

| ptm-tde PC `args` | op yaml `inputs.params` | CLI flag | 三层是否一致 |
|---|---|---|---|
| `src_addr` | `source_network` | `--source-network` | 否（三层各不同） |
| `dst_addr` | `dst_network` | `--dst-network` | 否（第 1 层不同） |
| `next_hop` | `next_hop_ip` | `--next-hop-ip` | 否（第 1 层不同） |
| `in_interface` | `in_interface` | `--in-interface` | 否（第 3 层用连字符） |
| `type` | `type` | `--policy-route-type` | 否（第 3 层加前缀） |
| `id` | `id` | `--id` | 是 |
| `page` | - | `--page` | 是（op yaml 无此 param，CLI 分页参数） |
| `size` | - | `--size` | 是（同上） |
| `moveid` | - | `--moveid` | 是（op yaml params 在 `build_policy_route_priority_payload` 消费） |
| `targetid` | - | `--targetid` | 是 |
| `targetsite` | - | `--targetsite` | 是 |

#### 5.1.3 回滚策略：ROLLBACK_STRATEGY

```python
ROLLBACK_STRATEGY = {
    "fw_login_web_management": {
        "type": "none",
        "reason": "observation，只读，建立 session，不回滚",
    },
    "fw_config_policy_route": {
        "type": "inverse_op",
        "inverse_op_id": "fw_delete_policy_route",
        "inverse_args_key": "id",          # 从 config 返回 data.policy_route_id 取 id
        "snapshot_required": False,
    },
    "fw_update_policy_route": {
        "type": "restore_snapshot",
        "snapshot_source": "pre_snapshot.full_config",
        "restore_op_id": "fw_update_policy_route",
        "snapshot_required": True,
    },
    "fw_delete_policy_route": {
        "type": "restore_snapshot",
        "snapshot_source": "pre_snapshot.full_config",
        "restore_op_id": "fw_config_policy_route",
        "snapshot_required": True,
        "as_cleanup_skip": True,   # 作为 config 清理动作时不触发回滚
    },
    "fw_verify_policy_route": {
        "type": "none",
        "reason": "observation，只读，不回滚",
    },
    "fw_update_policy_route_priority": {
        "type": "none",
        "reason": "无 rollback 元数据，由用例设计决定是否恢复原优先级",
    },
    "fw_reset_policy_route_hitcount": {
        "type": "irreversible",
        "reason": "命中计数清零不可恢复，不回滚",
    },
    "fw_verify_policy_route_hitcount": {
        "type": "none",
        "reason": "observation，只读，不回滚",
    },
}
```

真相源校验（`ptm-atomic list` 2026-07-10 实测）：

| op_id | side_effect | rollback | idempotent | op_mapper ROLLBACK_STRATEGY.type |
|---|---|---|---|---|
| fw_config_policy_route | state_mutation | `inverse_op:fw_delete_policy_route` | true | inverse_op |
| fw_update_policy_route | state_mutation | `restore_snapshot` | true | restore_snapshot |
| fw_delete_policy_route | destructive | `restore_snapshot` | false | restore_snapshot（as_cleanup_skip） |
| fw_verify_policy_route | observation | （空） | true | none |
| fw_update_policy_route_priority | （空） | （空） | true | none |
| fw_reset_policy_route_hitcount | state_mutation | `irreversible` | true | irreversible |
| fw_verify_policy_route_hitcount | observation | （空） | true | none |
| fw_login_web_management | observation | （空） | true | none |

#### 5.1.4 OP_METADATA（元数据缓存，来源 ptm-atomic list）

```python
OP_METADATA = {
    "fw_config_policy_route":         {"side_effect": "state_mutation", "rollback": "inverse_op:fw_delete_policy_route", "idempotent": True},
    "fw_update_policy_route":         {"side_effect": "state_mutation", "rollback": "restore_snapshot",             "idempotent": True},
    "fw_delete_policy_route":         {"side_effect": "destructive",    "rollback": "restore_snapshot",             "idempotent": False},
    "fw_verify_policy_route":         {"side_effect": "observation",    "rollback": "",                             "idempotent": True},
    "fw_update_policy_route_priority":{"side_effect": "",               "rollback": "",                             "idempotent": True},
    "fw_reset_policy_route_hitcount": {"side_effect": "state_mutation", "rollback": "irreversible",                 "idempotent": True},
    "fw_verify_policy_route_hitcount":{"side_effect": "observation",    "rollback": "",                             "idempotent": True},
    "fw_login_web_management":        {"side_effect": "observation",    "rollback": "",                             "idempotent": True},
}
```

### 5.2 envelope schema

每条 op 执行结果封装为 envelope（JSON dict）：

```json
{
  "op_id": "fw_config_policy_route",
  "step_name": "配置策略路由",
  "status": "success",
  "data": {
    "policy_route_id": "<id>",
    "config_result": { "status": "succeeded", "config_domain": "policy_route" }
  },
  "error_type": "NONE",
  "diag_snapshot_ref": "runs/<run-id>/snapshot-before/<step>.json"
}
```

| 字段 | 类型 | 取值 | 说明 |
|------|------|------|------|
| `op_id` | string | 8 个 op_id 之一 | 当前执行的原子操作 ID |
| `step_name` | string | 任意 | 用例步骤名（从 case_steps.step_name 传入） |
| `status` | string | `success` / `error` | 执行结果 |
| `data` | object | ptm-atomic 返回的 data 字段 | 成功时含操作结果；失败时为空 dict |
| `error_type` | string | 见下表 | 错误分类 |
| `diag_snapshot_ref` | string | 路径 | 诊断快照引用（由调用方传入） |

error_type 取值表：

| error_type | 含义 | op_mapper 处理 |
|------------|------|---------------|
| `NONE` | 成功 | 无错误 |
| `STATE_INVALID` | session 失效 | 自动重连 + 重试 1 次 |
| `OP_NOT_FOUND` | op_id 未在映射表中 | 阻塞，提示工具缺失 |
| `EXEC_FAILED` | subprocess 执行失败 / 超时 | 返回错误 envelope |
| `VALIDATION_FAILED` | 参数校验失败 | 返回错误 envelope |
| `CONFIG_REJECTED` | 设备拒绝配置（如 ePolicyRouteInIfModeError） | 返回错误 envelope，SKILL 注明需人工 Web 改 |
| `DEVICE_UNREACHABLE` | 设备不可达 | 返回错误 envelope |
| `AUTH_FAILED` | 登录失败 | 终止，不重试 |
| `UNKNOWN_ERROR` | 未知错误 | 返回错误 envelope |

### 5.3 exec-log.jsonl schema

每条 op 执行记录写入 `runs/<run-id>/exec-log.jsonl`，每行一个 JSON 对象：

```json
{
  "step_index": 1,
  "step_name": "配置策略路由",
  "op_id": "fw_config_policy_route",
  "mode": "dry-run",
  "command": ["ptm-atomic", "run", "--base-url", "https://10.113.55.51", "--session-file", "/path/session.json", "policy-route", "config", "--source-network", "10.0.0.0/24", "--in-interface", "GE0_12"],
  "exit_code": 0,
  "envelope": { "op_id": "...", "status": "success", ... },
  "api_status_code": 200,
  "timestamp": "2026-07-10T12:00:00+08:00",
  "duration_ms": 1234
}
```

### 5.4 持久化

op_mapper.py 自身不持久化状态。所有持久化由调用方（ptm-te agent 编排流程）负责：

- `session.json`：由 ptm-atomic `--session-file` 自动管理（默认 `~/.local/state/ptm-atomic/ngfw/session.json`）
- `exec-log.jsonl`：op_mapper 的 `_append_exec_log` 按 `exec_log_path` 参数写入，路径由调用方指定
- `result.json`：由调用方汇总各条 envelope 后写入
- 映射表常量：编译期固定在 op_mapper.py 源码中，不动态加载

---

## 6. API / Interface 设计

### 6.1 公开 API（op_mapper.py 对外接口）

#### 6.1.1 map_op_id_to_subcommand

```python
def map_op_id_to_subcommand(op_id: str) -> tuple[str, str]:
    """第一层映射：op_id -> (family, action) CLI 子命令。

    Args:
        op_id: ptm-tde PC 中的 atomic_op.op_id，如 "fw_config_policy_route"

    Returns:
        (family, action) 元组，如 ("policy-route", "config")

    Raises:
        OpNotFoundError: op_id 不在 OP_ID_TO_SUBCOMMAND 中时抛出
    """
```

#### 6.1.2 map_args_to_flags

```python
def map_args_to_flags(op_id: str, args: dict) -> list[str]:
    """第二层映射：args dict -> CLI flag 列表。

    将 ptm-tde PC 的 args 字段名翻译为 ptm-atomic CLI flag。
    三层命名翻译在此 centralize：args key -> flag name 取自 ARGS_TO_FLAGS[op_id]。

    Args:
        op_id: 原子操作 ID
        args: ptm-tde PC 的 atomic_op.args dict，如 {"src_addr": "10.0.0.0/24", "in_interface": "GE0_12"}

    Returns:
        CLI flag 列表，如 ["--source-network", "10.0.0.0/24", "--in-interface", "GE0_12"]

    Raises:
        OpNotFoundError: op_id 不在 ARGS_TO_FLAGS 中
    """
```

特殊处理：
- `fw_login_web_management`：`password_env` 默认 `"FW_WEB_PASSWORD"`，若 args 中未提供则使用默认值
- `fw_config_policy_route` / `fw_update_policy_route`：`--source-network` 和 `--in-interface` 在 CLI 中为 required（见 `run_policy_route.py` `_add_common_args`），args 中缺失时 `map_args_to_flags` 仍生成 flag 但值为 None，由 `build_command` 校验 required flag
- 只映射 `ARGS_TO_FLAGS[op_id]` 中定义的 key；args 中多余 key 忽略并记录 warning

#### 6.1.3 build_command

```python
def build_command(
    op_id: str,
    args: dict,
    base_url: str,
    session_file: str,
    *,
    dry_run: bool = True,
) -> list[str]:
    """组装完整的 ptm-atomic run 命令列表。

    命令格式（嵌套子命令，非扁平）：
        ptm-atomic run --base-url <url> [--session-file <path>] <family> <action> [flags]
    dry_run=True 时不加 --execute（默认干跑）。
    dry_run=False 时加 --execute。

    Args:
        op_id: 原子操作 ID
        args: 参数 dict
        base_url: 设备 Web 管理地址，如 "https://10.113.55.51"
        session_file: session.json 路径
        dry_run: 是否干跑模式（默认 True）

    Returns:
        命令列表，如 ["ptm-atomic", "run", "--base-url", "https://...", "--session-file", "/path", "policy-route", "config", "--source-network", "..."]

    Raises:
        OpNotFoundError: op_id 未识别
        ValueError: required flag 缺失（--source-network / --in-interface / --id 等）
    """
```

#### 6.1.4 execute_op

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
    exec_log_path: str | None = None,
    diag_snapshot_ref: str = "",
) -> dict:
    """执行单条原子操作，返回 envelope dict。

    流程：
    1. build_command 组装命令
    2. dry_run=False 且 authorized=False 时拒绝执行（返回 error_type=EXEC_FAILED）
    3. subprocess 调用 ptm-atomic CLI
    4. 解析输出（JSON/YAML）为 envelope
    5. 检测 STATE_INVALID -> _reconnect_and_retry（最多 1 次）
    6. 写入 exec-log.jsonl（若 exec_log_path 提供）

    Args:
        op_id: 原子操作 ID
        args: 参数 dict
        base_url: 设备 Web 管理地址
        session_file: session.json 路径
        step_name: 用例步骤名（写入 envelope）
        dry_run: 是否干跑（默认 True）
        authorized: --execute 写操作授权标记（dry_run=False 时必须为 True）
        timeout: 超时秒数（默认 30，与 op yaml timeout_ms 一致）
        username: 登录用户名（STATE_INVALID 重连用）
        password_env: 密码环境变量名（STATE_INVALID 重连用）
        exec_log_path: 执行日志路径（None 则不写日志）
        diag_snapshot_ref: 诊断快照引用路径

    Returns:
        envelope dict，含 op_id/step_name/status/data/error_type/diag_snapshot_ref
    """
```

#### 6.1.5 handle_rollback

```python
def handle_rollback(
    op_id: str,
    args: dict,
    base_url: str,
    session_file: str,
    *,
    pre_snapshot: dict | None = None,
    authorized: bool = False,
    timeout: int = 30,
) -> dict:
    """按 op 的 rollback 策略执行回滚清理。

    策略路由：
    - inverse_op: 执行 inverse_op 清理（如 config -> delete）
    - restore_snapshot: 按 pre_snapshot 恢复（如 update -> 恢复原值）
    - irreversible: 不回滚，返回豁免说明 envelope
    - none: 不回滚，返回无需回滚 envelope

    Args:
        op_id: 原子操作 ID
        args: 原操作参数（用于 inverse_op 时提取 id 等清理参数）
        base_url: 设备 Web 管理地址
        session_file: session.json 路径
        pre_snapshot: 操作前快照（restore_snapshot 类必需）
        authorized: --execute 授权标记
        timeout: 超时秒数

    Returns:
        回滚结果 envelope dict
    """
```

#### 6.1.6 validate_mapping_consistency

```python
def validate_mapping_consistency() -> "ValidationResult":
    """校验映射表与三处真相源的一致性（CP7 static 校验入口）。

    校验维度：
    1. OP_ID_TO_SUBCOMMAND 的 7 个 policy-route op 与 run_policy_route.py build_subtree() 一致
    2. ARGS_TO_FLAGS 的 flag 名与 run_policy_route.py _add_*_args() 定义的 argparse flag 一致
    3. ARGS_TO_FLAGS 的 args key 与 op yaml inputs.params 参数名对应一致（三层翻译覆盖）
    4. ROLLBACK_STRATEGY 与 OP_METADATA 的 rollback 字段一致
    5. 8 个 op_id 全覆盖（OP_ID_TO_SUBCOMMAND / ARGS_TO_FLAGS / ROLLBACK_STRATEGY 三表一致）

    Returns:
        ValidationResult: passed=True/False, mismatches=list[str]
    """
```

### 6.2 私有 API（op_mapper.py 内部函数）

```python
def _reconnect_and_retry(
    base_url: str,
    session_file: str,
    username: str,
    password_env: str,
    retry_command: list[str],
    timeout: int,
) -> dict:
    """STATE_INVALID 自动重连：重新 auth login 后重试原命令。

    1. 执行 auth login（--username --password-env --session-file）
    2. 重试原 retry_command
    3. 返回重试后的 envelope
    重试仍失败则返回 error_type=STATE_INVALID 的 envelope。
    """

def _build_envelope(
    op_id: str,
    step_name: str,
    status: str,
    data: dict,
    error_type: str,
    diag_snapshot_ref: str = "",
) -> dict:
    """构建标准 envelope dict。"""

def _append_exec_log(log_path: str, record: dict) -> None:
    """向 exec-log.jsonl 追加一条记录（JSON Lines 格式）。"""

def _parse_atomic_output(stdout: str) -> dict:
    """解析 ptm-atomic CLI 输出（JSON 或 YAML）为 dict。

    ptm-atomic 默认输出 YAML（--format yaml）；op_mapper 统一使用 --format json 便于解析。
    """

def _check_required_flags(op_id: str, flag_list: list[str]) -> None:
    """校验 required flag 是否存在（如 config 的 --source-network / --in-interface）。

    Raises:
        ValueError: required flag 缺失
    """
```

### 6.3 CLI 入口（__main__）

```python
# op_mapper.py 可作为 CLI 直接调用（便于测试和手动验证）
# 用法：
#   python op_mapper.py execute --op-id fw_config_policy_route \
#       --base-url https://10.113.55.51 \
#       --session-file /path/session.json \
#       --args '{"src_addr":"10.0.0.0/24","in_interface":"GE0_12"}' \
#       --dry-run
#   python op_mapper.py validate          # 映射表一致性校验
#   python op_mapper.py map --op-id fw_config_policy_route --args '{...}'  # 打印映射结果
```

### 6.4 集成契约（与 ptm-te agent 编排流程 [4] 的契约）

| 维度 | 契约 |
|------|------|
| 调用方向 | ptm-te agent 编排流程 [4] -> op_mapper.execute_op / handle_rollback（单向） |
| 调用时机 | ptm-te 逐条执行 case_steps 时 |
| 触发方式 | ptm-te agent 读取 PC case_steps，提取 op_id + args，调用 execute_op |
| 输入契约 | op_id: str, args: dict, base_url: str, session_file: str, dry_run: bool, authorized: bool |
| 输出契约 | envelope dict（op_id/step_name/status/data/error_type/diag_snapshot_ref） |
| 后续衔接 | ptm-te agent 根据 envelope.status 判定 Check 点；失败时根据 error_type 决定反馈 ptm-tae（OP_NOT_FOUND）或重试（STATE_INVALID） |
| 降级策略 | OP_NOT_FOUND -> 阻塞提示工具缺失；STATE_INVALID -> 自动重连重试；DEVICE_UNREACHABLE -> 降级 dry-run-only |
| 调用方需同步修改范围 | 无。op_mapper 是纯消费方，不要求上游修改 |

---

## 7. 核心处理流程

### 7.1 execute_op 主流程（干跑 -> 执行 -> STATE_INVALID 重连）

```
execute_op(op_id, args, base_url, session_file, dry_run=True, authorized=False)
  │
  ├─[1] map_op_id_to_subcommand(op_id)
  │     └─ op_id 不在映射表 -> 返回 envelope{status=error, error_type=OP_NOT_FOUND}
  │
  ├─[2] map_args_to_flags(op_id, args)
  │     └─ 生成 flag 列表
  │
  ├─[3] build_command(op_id, args, base_url, session_file, dry_run)
  │     └─ 组装 ["ptm-atomic", "run", "--base-url", ..., "--session-file", ..., family, action, flags...]
  │     └─ dry_run=True: 不加 --execute
  │     └─ dry_run=False: 加 --execute
  │
  ├─[4] 授权检查
  │     └─ dry_run=False 且 authorized=False -> 返回 envelope{status=error, error_type=EXEC_FAILED, data={"reason":"unauthorized"}}
  │
  ├─[5] subprocess.run(command, timeout=timeout)
  │     └─ 超时 -> 返回 envelope{status=error, error_type=EXEC_FAILED}
  │     └─ exit_code != 0 -> 解析错误输出
  │
  ├─[6] _parse_atomic_output(stdout)
  │     └─ 解析为 envelope dict
  │
  ├─[7] STATE_INVALID 检测
  │     └─ envelope.error_type == STATE_INVALID 且 dry_run=False:
  │         ├─ _reconnect_and_retry(base_url, session_file, username, password_env, command, timeout)
  │         │   ├─ 执行 auth login: ptm-atomic run --base-url <url> auth login --username admin --password-env FW_WEB_PASSWORD --session-file <path> --execute
  │         │   ├─ 重试原 command
  │         │   └─ 返回重试 envelope（成功或失败）
  │         └─ 最多重试 1 次
  │
  ├─[8] _append_exec_log(exec_log_path, record)（若 exec_log_path 提供）
  │     └─ 写入 exec-log.jsonl 一行
  │
  └─[9] 返回 envelope
```

### 7.2 三阶段执行流程（干跑 -> 执行 -> verify）

ptm-te agent 编排流程 [4] 对每条 state_mutation 类 op 的三阶段执行：

```
[阶段 1] 干跑（dry-run）
  └─ execute_op(op_id, args, base_url, session_file, dry_run=True)
      └─ 验证参数路由和 session 有效性
      └─ envelope.status == success -> 进入阶段 2
      └─ envelope.status == error -> 阻塞，不进入执行

[阶段 2] 执行（execute，需 authorized=True）
  └─ execute_op(op_id, args, base_url, session_file, dry_run=False, authorized=True)
      └─ 真实写操作，设备策略变更
      └─ STATE_INVALID -> 自动重连重试
      └─ envelope.status == success -> 进入阶段 3
      └─ envelope.status == error -> 记录错误，准备回滚

[阶段 3] 验证（verify）
  └─ execute_op("fw_verify_policy_route", {type, page, size}, base_url, session_file, dry_run=False, authorized=True)
      └─ 查询设备当前策略路由状态
      └─ 与 expected_result 比对判定 Check 点
```

observation 类 op（verify / verify-hitcount）和 login 只执行阶段 1 或直接 execute（只读无副作用）。

### 7.3 inverse_op 清理流程

用例执行后，ptm-te agent 调用 `handle_rollback` 清理：

```
handle_rollback(op_id, args, base_url, session_file, pre_snapshot, authorized)
  │
  ├─ 查询 ROLLBACK_STRATEGY[op_id]
  │
  ├─[A] type == "inverse_op"
  │    └─ config 的 inverse_op = fw_delete_policy_route
  │    └─ 从 config 返回的 data.policy_route_id 提取 id
  │    └─ execute_op("fw_delete_policy_route", {"id": policy_route_id, "type": args["type"]},
  │                  base_url, session_file, dry_run=False, authorized=True)
  │    └─ delete 作为清理动作本身不再触发回滚（as_cleanup_skip）
  │
  ├─[B] type == "restore_snapshot"
  │    └─ update 的 restore_snapshot
  │    └─ 从 pre_snapshot.full_config 恢复原始策略路由完整配置
  │    └─ execute_op("fw_update_policy_route", pre_snapshot.full_config,
  │                  base_url, session_file, dry_run=False, authorized=True)
  │    └─ delete 的 rollback 也是 restore_snapshot，但作为 config 清理动作时不触发
  │
  ├─[C] type == "irreversible"
  │    └─ reset-hitcount：不回滚
  │    └─ 返回 envelope{status=success, data={"rollback":"waived","reason":"irreversible"}}
  │    └─ SKILL 注明：用例设计者需接受该副作用或通过用例顺序规避
  │
  └─[D] type == "none"
       └─ verify / verify-hitcount / login / priority：不回滚
       └─ 返回 envelope{status=success, data={"rollback":"not_required"}}
```

### 7.4 STATE_INVALID 重连流程

```
execute_op 检测到 envelope.error_type == STATE_INVALID
  │
  ├─[1] _reconnect_and_retry(base_url, session_file, username, password_env, original_command, timeout)
  │
  ├─[2] 执行 auth login:
  │      ptm-atomic run --base-url <url> auth login
  │        --username admin --password-env FW_WEB_PASSWORD
  │        --session-file <path> --execute --format json
  │    └─ login 成功 -> session.json 更新
  │    └─ login 失败 -> 返回 envelope{error_type=AUTH_FAILED}，不重试原命令
  │
  ├─[3] 重试原 original_command（带 --execute）
  │    └─ 重试成功 -> 返回 envelope{status=success}
  │    └─ 重试仍 STATE_INVALID -> 返回 envelope{error_type=STATE_INVALID}（不再重试）
  │    └─ 重试其他错误 -> 返回对应 error_type envelope
  │
  └─[4] 重试上限：1 次（避免无限循环）
```

### 7.5 未识别 op_id 处理流程

```
execute_op(op_id="fw_some_unknown_op", ...)
  │
  ├─ map_op_id_to_subcommand("fw_some_unknown_op")
  │    └─ op_id 不在 OP_ID_TO_SUBCOMMAND -> 抛出 OpNotFoundError
  │
  ├─ 捕获 OpNotFoundError
  │    └─ 返回 envelope{
  │         status=error,
  │         error_type=OP_NOT_FOUND,
  │         data={"op_id":"fw_some_unknown_op", "suggestion":"反馈 ptm-tae 检查工具覆盖"}
  │       }
  │
  └─ ptm-te agent 收到 OP_NOT_FOUND -> 阻塞该 step，不静默跳过
```

---

## 8. 技术设计细节

> 本节阐述技术细节（函数级设计、数据结构、算法实现）。

### 8.1 op_mapper.py 函数级设计

#### 8.1.1 map_op_id_to_subcommand

```python
def map_op_id_to_subcommand(op_id: str) -> tuple[str, str]:
    """第一层映射：op_id -> (family, action)。

    Args:
        op_id: 原子操作 ID（如 "fw_config_policy_route"）

    Returns:
        (family, action) 元组（如 ("policy-route", "config")）

    Raises:
        OpNotFoundError: op_id 不在 OP_ID_TO_SUBCOMMAND 中
    """
    if op_id not in OP_ID_TO_SUBCOMMAND:
        raise OpNotFoundError(
            f"未识别的 op_id: {op_id}，当前映射表覆盖 {len(OP_ID_TO_SUBCOMMAND)} 个 op_id。"
            f"请反馈 ptm-tae 检查工具覆盖。"
        )
    return OP_ID_TO_SUBCOMMAND[op_id]
```

#### 8.1.2 map_args_to_flags

```python
def map_args_to_flags(op_id: str, args: dict) -> list[str]:
    """第二层映射：args dict -> CLI flag 列表。

    三层命名翻译 centralize 在此：args key -> flag name 取自 ARGS_TO_FLAGS[op_id]。
    args 中多余 key 忽略并记录 warning 到 stderr。
    """
    if op_id not in ARGS_TO_FLAGS:
        raise OpNotFoundError(f"op_id {op_id} 无 args->flag 映射表")
    flag_map = ARGS_TO_FLAGS[op_id]
    result: list[str] = []
    for args_key, cli_flag in flag_map.items():
        if args_key in args and args[args_key] is not None:
            value = args[args_key]
            # password_env 特殊处理：默认 FW_WEB_PASSWORD
            if args_key == "password_env" and not value:
                value = "FW_WEB_PASSWORD"
            result.append(cli_flag)
            result.append(str(value))
        # login 的 password_env 默认值处理
        elif args_key == "password_env" and op_id == "fw_login_web_management":
            result.append(cli_flag)
            result.append("FW_WEB_PASSWORD")
    # 检查多余 key
    for key in args:
        if key not in flag_map:
            print(f"[op_mapper] WARNING: args key '{key}' 不在 {op_id} 映射表中，已忽略", file=sys.stderr)
    return result
```

#### 8.1.3 build_command

```python
def build_command(
    op_id: str,
    args: dict,
    base_url: str,
    session_file: str,
    *,
    dry_run: bool = True,
) -> list[str]:
    """组装完整的 ptm-atomic run 嵌套子命令列表。"""
    family, action = map_op_id_to_subcommand(op_id)
    flags = map_args_to_flags(op_id, args)
    # required flag 校验
    _check_required_flags(op_id, flags)
    command = [
        "ptm-atomic", "run",
        "--base-url", base_url,
        "--session-file", session_file,
        "--format", "json",        # 统一 JSON 输出便于解析
        family, action,
        *flags,
    ]
    if not dry_run:
        command.append("--execute")
    return command
```

required flag 校验规则（来源 `run_policy_route.py` `_add_*_args`）：

| op_id | required flags |
|-------|---------------|
| fw_config_policy_route | `--source-network`, `--in-interface` |
| fw_update_policy_route | `--source-network`, `--in-interface`, `--id` |
| fw_delete_policy_route | `--id` |
| fw_reset_policy_route_hitcount | `--id` |
| fw_update_policy_route_priority | `--targetsite`, `--targetid`, `--moveid` |
| fw_verify_policy_route | 无 required |
| fw_verify_policy_route_hitcount | 无 required |
| fw_login_web_management | 无 required（`--username` / `--password-env` 有默认值或由 op_mapper 补全） |

#### 8.1.4 execute_op

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
    exec_log_path: str | None = None,
    diag_snapshot_ref: str = "",
) -> dict:
    """执行单条原子操作，返回 envelope dict。"""
    import subprocess, time, json

    # [1] 构建命令
    try:
        command = build_command(op_id, args, base_url, session_file, dry_run=dry_run)
    except OpNotFoundError as e:
        return _build_envelope(op_id, step_name, "error", {"reason": str(e)}, "OP_NOT_FOUND", diag_snapshot_ref)
    except ValueError as e:
        return _build_envelope(op_id, step_name, "error", {"reason": str(e)}, "VALIDATION_FAILED", diag_snapshot_ref)

    # [2] 授权检查
    if not dry_run and not authorized:
        return _build_envelope(op_id, step_name, "error",
            {"reason": "dry_run=False 需要 authorized=True 授权标记"}, "EXEC_FAILED", diag_snapshot_ref)

    # [3] 执行
    start_time = time.time()
    try:
        proc = subprocess.run(command, capture_output=True, text=True, timeout=timeout)
        exit_code = proc.returncode
        stdout, stderr = proc.stdout, proc.stderr
    except subprocess.TimeoutExpired:
        return _build_envelope(op_id, step_name, "error",
            {"reason": f"执行超时 ({timeout}s)"}, "EXEC_FAILED", diag_snapshot_ref)

    duration_ms = int((time.time() - start_time) * 1000)

    # [4] 解析输出
    envelope = _parse_atomic_output(stdout)
    if envelope is None:
        envelope = _build_envelope(op_id, step_name, "error",
            {"reason": "无法解析 ptm-atomic 输出", "stdout": stdout[:500], "stderr": stderr[:500]},
            "UNKNOWN_ERROR", diag_snapshot_ref)
    else:
        envelope["op_id"] = op_id
        envelope["step_name"] = step_name
        if "diag_snapshot_ref" not in envelope:
            envelope["diag_snapshot_ref"] = diag_snapshot_ref

    # [5] STATE_INVALID 重连
    if envelope.get("error_type") == "STATE_INVALID" and not dry_run:
        envelope = _reconnect_and_retry(
            base_url, session_file, username, password_env, command, timeout
        )
        envelope["op_id"] = op_id
        envelope["step_name"] = step_name

    # [6] 写入 exec-log
    if exec_log_path:
        record = {
            "step_name": step_name,
            "op_id": op_id,
            "mode": "dry-run" if dry_run else "execute",
            "command": command,
            "exit_code": exit_code,
            "envelope": envelope,
            "timestamp": time.strftime("%Y-%m-%dT%H:%M:%S%z"),
            "duration_ms": duration_ms,
        }
        _append_exec_log(exec_log_path, record)

    return envelope
```

#### 8.1.5 _reconnect_and_retry

```python
def _reconnect_and_retry(
    base_url: str,
    session_file: str,
    username: str,
    password_env: str,
    retry_command: list[str],
    timeout: int,
) -> dict:
    """STATE_INVALID 自动重连：重新 auth login 后重试原命令。"""
    import subprocess

    # [1] 重新 login
    login_cmd = [
        "ptm-atomic", "run",
        "--base-url", base_url,
        "--session-file", session_file,
        "--format", "json",
        "auth", "login",
        "--username", username,
        "--password-env", password_env,
        "--execute",
    ]
    try:
        login_proc = subprocess.run(login_cmd, capture_output=True, text=True, timeout=timeout)
        login_envelope = _parse_atomic_output(login_proc.stdout)
        if login_envelope is None or login_envelope.get("status") != "success":
            return _build_envelope("", "", "error",
                {"reason": "重连登录失败", "login_output": login_proc.stdout[:500]},
                "AUTH_FAILED", "")
    except subprocess.TimeoutExpired:
        return _build_envelope("", "", "error", {"reason": "重连登录超时"}, "AUTH_FAILED", "")

    # [2] 重试原命令
    try:
        proc = subprocess.run(retry_command, capture_output=True, text=True, timeout=timeout)
        envelope = _parse_atomic_output(proc.stdout)
        if envelope is None:
            return _build_envelope("", "", "error",
                {"reason": "重试后仍无法解析输出"}, "UNKNOWN_ERROR", "")
        return envelope
    except subprocess.TimeoutExpired:
        return _build_envelope("", "", "error",
            {"reason": "重试执行超时"}, "EXEC_FAILED", "")
```

#### 8.1.6 handle_rollback

```python
def handle_rollback(
    op_id: str,
    args: dict,
    base_url: str,
    session_file: str,
    *,
    pre_snapshot: dict | None = None,
    authorized: bool = False,
    timeout: int = 30,
) -> dict:
    """按 op 的 rollback 策略执行回滚清理。"""
    if op_id not in ROLLBACK_STRATEGY:
        return _build_envelope(op_id, "rollback", "error",
            {"reason": f"op_id {op_id} 无回滚策略"}, "OP_NOT_FOUND", "")

    strategy = ROLLBACK_STRATEGY[op_id]
    rtype = strategy["type"]

    if rtype == "inverse_op":
        # config -> delete 清理
        inverse_op_id = strategy["inverse_op_id"]
        # 从原操作返回结果提取 id（config 返回 data.policy_route_id）
        inverse_args = {}
        if "id" in args:
            inverse_args["id"] = args["id"]
        if "type" in args:
            inverse_args["type"] = args["type"]
        return execute_op(inverse_op_id, inverse_args, base_url, session_file,
            step_name=f"rollback-{op_id}", dry_run=False, authorized=authorized, timeout=timeout)

    elif rtype == "restore_snapshot":
        # update / delete -> 按快照恢复
        if strategy.get("as_cleanup_skip"):
            # delete 作为 config 清理动作时不触发回滚
            return _build_envelope(op_id, "rollback", "success",
                {"rollback": "skipped", "reason": "作为清理动作不触发回滚"}, "NONE", "")
        if pre_snapshot is None:
            return _build_envelope(op_id, "rollback", "error",
                {"reason": "restore_snapshot 需要 pre_snapshot"}, "EXEC_FAILED", "")
        restore_op_id = strategy["restore_op_id"]
        restore_args = pre_snapshot.get(strategy["snapshot_source"], pre_snapshot)
        return execute_op(restore_op_id, restore_args, base_url, session_file,
            step_name=f"rollback-{op_id}", dry_run=False, authorized=authorized, timeout=timeout)

    elif rtype == "irreversible":
        # reset-hitcount：不回滚
        return _build_envelope(op_id, "rollback", "success",
            {"rollback": "waived", "reason": strategy["reason"]}, "NONE", "")

    else:  # none
        return _build_envelope(op_id, "rollback", "success",
            {"rollback": "not_required", "reason": strategy.get("reason", "")}, "NONE", "")
```

#### 8.1.7 validate_mapping_consistency

```python
def validate_mapping_consistency() -> "ValidationResult":
    """校验映射表与三处真相源的一致性。

    三处真相源：
    1. run_policy_route.py build_subtree() - 7 个 policy-route 子命令名
    2. op yaml inputs.params - 参数名（source_network/dst_network/next_hop_ip/in_interface/type/id）
    3. ptm-atomic run ... --help - CLI flag 名（--source-network 等）

    校验维度：
    - 8 个 op_id 在 OP_ID_TO_SUBCOMMAND / ARGS_TO_FLAGS / ROLLBACK_STRATEGY 三表全覆盖
    - 7 个 policy-route 子命令名与 build_subtree() 一致
    - flag 名格式正确（-- 前缀）
    - ROLLBACK_STRATEGY.type 与 OP_METADATA.rollback 一致
    """
    mismatches: list[str] = []

    # [1] 三表 op_id 一致性
    ops_in_sub = set(OP_ID_TO_SUBCOMMAND.keys())
    ops_in_args = set(ARGS_TO_FLAGS.keys())
    ops_in_rollback = set(ROLLBACK_STRATEGY.keys())
    if ops_in_sub != ops_in_args:
        mismatches.append(f"OP_ID_TO_SUBCOMMAND 与 ARGS_TO_FLAGS 的 op_id 集合不一致: "
                          f"差集={ops_in_sub.symmetric_difference(ops_in_args)}")
    if ops_in_sub != ops_in_rollback:
        mismatches.append(f"OP_ID_TO_SUBCOMMAND 与 ROLLBACK_STRATEGY 的 op_id 集合不一致: "
                          f"差集={ops_in_sub.symmetric_difference(ops_in_rollback)}")

    # [2] op_id 数量校验
    if len(ops_in_sub) != 8:
        mismatches.append(f"OP_ID_TO_SUBCOMMAND 应覆盖 8 个 op_id，实际 {len(ops_in_sub)} 个")

    # [3] flag 格式校验
    for op_id, flag_map in ARGS_TO_FLAGS.items():
        for args_key, cli_flag in flag_map.items():
            if not cli_flag.startswith("--"):
                mismatches.append(f"{op_id}.{args_key} 的 flag '{cli_flag}' 缺少 -- 前缀")

    # [4] ROLLBACK_STRATEGY 与 OP_METADATA 一致性
    for op_id, meta in OP_METADATA.items():
        strategy = ROLLBACK_STRATEGY.get(op_id, {})
        meta_rollback = meta.get("rollback", "")
        strategy_type = strategy.get("type", "")
        # 交叉校验
        if meta_rollback.startswith("inverse_op:") and strategy_type != "inverse_op":
            mismatches.append(f"{op_id}: OP_METADATA.rollback='{meta_rollback}' 但 ROLLBACK_STRATEGY.type='{strategy_type}'")
        if meta_rollback == "restore_snapshot" and strategy_type != "restore_snapshot":
            mismatches.append(f"{op_id}: OP_METADATA.rollback='restore_snapshot' 但 ROLLBACK_STRATEGY.type='{strategy_type}'")
        if meta_rollback == "irreversible" and strategy_type != "irreversible":
            mismatches.append(f"{op_id}: OP_METADATA.rollback='irreversible' 但 ROLLBACK_STRATEGY.type='{strategy_type}'")
        if meta_rollback == "" and strategy_type not in ("none",):
            mismatches.append(f"{op_id}: OP_METADATA.rollback 为空但 ROLLBACK_STRATEGY.type='{strategy_type}'")

    passed = len(mismatches) == 0
    return ValidationResult(passed=passed, mismatches=mismatches)
```

#### 8.1.8 辅助类型定义

```python
class OpNotFoundError(Exception):
    """op_id 未在映射表中。"""

class ValidationResult:
    """映射表一致性校验结果。"""
    def __init__(self, passed: bool, mismatches: list[str]):
        self.passed = passed
        self.mismatches = mismatches
    def __str__(self):
        if self.passed:
            return "ValidationResult: PASS (8 op_id 全覆盖，三表一致)"
        return f"ValidationResult: FAIL\n" + "\n".join(f"  - {m}" for m in self.mismatches)
```

### 8.2 subprocess 调用细节

```python
# ptm-atomic CLI 输出格式统一为 JSON（--format json）
# 成功输出示例（config dry-run）：
# {
#   "status": "success",
#   "data": { "policy_route_id": "...", "config_result": {...} },
#   "error_type": "NONE",
#   "diag_snapshot_ref": "..."
# }

# 失败输出示例（STATE_INVALID）：
# {
#   "status": "error",
#   "data": {},
#   "error_type": "STATE_INVALID",
#   "diag_snapshot_ref": "..."
# }

# _parse_atomic_output 解析逻辑：
# 1. 尝试 json.loads(stdout)
# 2. 失败则尝试 yaml.safe_load(stdout)
# 3. 均失败返回 None（调用方构建 UNKNOWN_ERROR envelope）
```

### 8.3 CLI flag 与 argparse 定义对照（run_policy_route.py 真相源）

| _add_*_args 函数 | 对应 op | required flags | optional flags（带默认值） |
|---|---|---|---|
| `_add_common_args` | config, update | `--source-network`, `--in-interface` | `--dst-network`(any), `--next-hop-ip`, `--outgoing-interface`(""), `--service`(any), `--weight`(100), `--time-obj`(""), `--policy-route-type`(ipv4), `--policy-route-active`(True), `--policy-route-action`(1), `--config-source`(0) |
| `_add_delete_args` | delete, reset-hitcount | `--id` | `--policy-route-type`(ipv4) |
| `_add_verify_args` | verify, verify-hitcount | 无 | `--policy-route-type`(ipv4), `--page`(1), `--size`(40) |
| `_add_priority_args` | priority | `--targetsite`, `--targetid`, `--moveid` | `--policy-route-type`(ipv4) |

> 注意：`run_policy_route.py` 中 update 使用 `_add_common_args`，但 `_run_update` 调用 `_require_arg(args, "id")`。当前 `_add_common_args` 未定义 `--id` argparse 参数（疑似 ptm-atomic 侧待修复）。op_mapper 按 HLD §4.4 生成 `--id` flag。Gotcha #10 已记录此约束。

### 8.4 命令示例（build_command 输出）

**config dry-run**：
```bash
ptm-atomic run --base-url https://10.113.55.51 --session-file ~/.local/state/ptm-atomic/ngfw/session.json --format json policy-route config --source-network 10.0.0.0/24 --dst-network any --next-hop-ip 18.18.2.2 --in-interface GE0_12 --policy-route-type ipv4
```

**config execute**：
```bash
ptm-atomic run --base-url https://10.113.55.51 --session-file ~/.local/state/ptm-atomic/ngfw/session.json --format json policy-route config --source-network 10.0.0.0/24 --dst-network any --next-hop-ip 18.18.2.2 --in-interface GE0_12 --policy-route-type ipv4 --execute
```

**delete execute（inverse_op 清理）**：
```bash
ptm-atomic run --base-url https://10.113.55.51 --session-file ~/.local/state/ptm-atomic/ngfw/session.json --format json policy-route delete --id 15 --policy-route-type ipv4 --execute
```

**auth login（重连）**：
```bash
ptm-atomic run --base-url https://10.113.55.51 --session-file ~/.local/state/ptm-atomic/ngfw/session.json --format json auth login --username admin --password-env FW_WEB_PASSWORD --execute
```

---

## 9. 安全与性能设计

### 9.1 安全设计

| # | 安全项 | 设计 | 真相源 |
|---|--------|------|--------|
| S-1 | 禁止明文密码 | login 命令使用 `--password-env FW_WEB_PASSWORD`，op_mapper 不接受明文密码参数；密码从环境变量读取 | HLD §7 |
| S-2 | dry-run 默认门 | `execute_op` 默认 `dry_run=True`；`dry_run=False` 需 `authorized=True` 显式授权标记，否则返回 `EXEC_FAILED` | HLD ADR-04 |
| S-3 | session 文件安全 | session.json 路径由调用方传入，op_mapper 不硬编码路径；session 文件不入库（.gitignore） | HLD §8 |
| S-4 | 命令注入防护 | `subprocess.run(command, ...)` 使用列表参数（非 shell=True），args 值作为独立参数传递，不经过 shell 解析 | NFR-5 |
| S-5 | TLS 验证 | ptm-atomic 默认 `--verify-tls`（设备自签名证书时由调用方决定是否 `--no-verify-tls`）；op_mapper 不自行为 TLS 配置做决定 | HLD §6 |
| S-6 | 超时保护 | 单条 op 默认 30 秒超时（与 op yaml `timeout_ms: 30000` 一致），防止挂起 | NFR-4 |
| S-7 | 未识别 op_id 阻塞 | 不静默跳过未知 op_id，返回 `OP_NOT_FOUND` 阻塞执行，防止误操作 | FR-8 |

### 9.2 性能设计

| # | 性能项 | 设计 | 度量值 |
|---|--------|------|--------|
| P-1 | 映射表查询 | dict O(1) 查找，无循环遍历 | <1ms |
| P-2 | subprocess 开销 | 每条 op 一次 subprocess 调用，无批量优化（首期 N/A） | 单条 op 端到端 <5s（含网络往返） |
| P-3 | session 复用 | login 一次后 session.json 复用，避免每条 op 重复登录 | 仅首次 login + STATE_INVALID 重连时触发 auth login |
| P-4 | 重连限制 | STATE_INVALID 最多重试 1 次，避免无限循环 | 最多 2 次 subprocess 调用（原命令 + 重连后重试） |
| P-5 | 日志写入 | exec-log.jsonl 追加写入（O(1)），不读取全文件 | 单次写入 <1ms |
| P-6 | 内存占用 | 映射表常量约 4KB；单条 op 执行不累积数据 | <10MB 峰值 |

### 9.3 并发安全

op_mapper.py 设计为单线程顺序执行（ptm-te agent 逐条执行 case_steps）。不支持多 op 并发执行：

- 策略路由 op 存在状态依赖（config 后才能 verify/update/delete）
- ptm-atomic CLI 的 session.json 不保证多进程并发安全
- 并发执行超出首期范围（T-05 candidate 是 batch 编排）

---

## 10. 测试设计

### 10.1 测试策略

`validation_mode=mixed`：static（映射一致性 + 单元测试）+ runtime（hg3250-51 端到端）。

### 10.2 static 测试

#### 10.2.1 映射表一致性校验（CP7 核心）

| # | 测试项 | 校验方法 | 预期结果 | 真相源 |
|---|--------|---------|---------|--------|
| T-S1 | op_id 数量 | `validate_mapping_consistency()` 检查 OP_ID_TO_SUBCOMMAND 有 8 个 | passed=True | HLD §4.3 |
| T-S2 | 子命令名一致 | OP_ID_TO_SUBCOMMAND 的 7 个 policy-route 值与 `run_policy_route.py` `build_subtree()` 的 `CommandSpec.name` 一致 | 7 个子命令名精确匹配 | `run_policy_route.py` |
| T-S3 | flag 名一致 | ARGS_TO_FLAGS 的 flag 与 `run_policy_route.py` `_add_*_args()` 的 `parser.add_argument` 名一致 | 全部 flag 名精确匹配 | `run_policy_route.py` |
| T-S4 | 三层翻译覆盖 | ARGS_TO_FLAGS 的 args key 覆盖 op yaml `inputs.params` 的核心参数（source_network/dst_network/next_hop_ip/in_interface/type/id） | 5 个三层不一致参数均有翻译条目 | op yaml |
| T-S5 | CLI help 一致 | 对比 `ptm-atomic run ... policy-route config --help` 输出的 flag 与 ARGS_TO_FLAGS["fw_config_policy_route"] | flag 名一致 | CLI `--help` |
| T-S6 | rollback 一致 | ROLLBACK_STRATEGY.type 与 `ptm-atomic list` 的 rollback 字段一致 | 8 个 op 的 rollback 类型匹配 | `ptm-atomic list` |
| T-S7 | 三表 op_id 一致 | OP_ID_TO_SUBCOMMAND / ARGS_TO_FLAGS / ROLLBACK_STRATEGY 的 key 集合相同 | 8 个 op_id 三表一致 | 内部一致性 |

#### 10.2.2 单元测试

| # | 测试项 | 输入 | 预期输出 |
|---|--------|------|---------|
| T-U1 | map_op_id_to_subcommand 正常 | `"fw_config_policy_route"` | `("policy-route", "config")` |
| T-U2 | map_op_id_to_subcommand 未识别 | `"fw_unknown_op"` | 抛出 `OpNotFoundError` |
| T-U3 | map_args_to_flags config | `{"src_addr":"10.0.0.0/24","in_interface":"GE0_12","type":"ipv4"}` | `["--source-network","10.0.0.0/24","--in-interface","GE0_12","--policy-route-type","ipv4"]` |
| T-U4 | map_args_to_flags login 默认密码 | `{"username":"admin"}` | 含 `["--username","admin","--password-env","FW_WEB_PASSWORD"]` |
| T-U5 | build_command dry-run | config op, dry_run=True | 命令列表不含 `--execute` |
| T-U6 | build_command execute | config op, dry_run=False | 命令列表含 `--execute` |
| T-U7 | build_command required 缺失 | config op 缺少 `src_addr` | 抛出 `ValueError` |
| T-U8 | execute_op 未授权 | dry_run=False, authorized=False | envelope `error_type=EXEC_FAILED` |
| T-U9 | execute_op OP_NOT_FOUND | 未知 op_id | envelope `error_type=OP_NOT_FOUND` |
| T-U10 | handle_rollback inverse_op | config op | 调用 delete 清理 |
| T-U11 | handle_rollback irreversible | reset-hitcount | 返回 `rollback=waived` |
| T-U12 | handle_rollback none | verify | 返回 `rollback=not_required` |
| T-U13 | handle_rollback delete as_cleanup | delete 作为 config 清理 | 返回 `rollback=skipped` |

#### 10.2.3 dry-run 验证（不需真实设备）

| # | 测试项 | 方法 | 预期 |
|---|--------|------|------|
| T-D1 | config dry-run 命令格式 | `build_command` + 手动检查 | 命令列表格式正确（嵌套子命令） |
| T-D2 | 全 7 个 op dry-run 命令 | 遍历 7 个 op 调用 build_command | 7 个命令均格式正确 |
| T-D3 | validate_mapping_consistency | 调用校验函数 | `passed=True` |

### 10.3 runtime 测试（需 DQ-01 授权 + DQ-02 设备 hg3250-51）

| # | 测试项 | 前置 | 预期 |
|---|--------|------|------|
| T-R1 | auth login 真实执行 | hg3250-51 可达，FW_WEB_PASSWORD 已设 | session.json 写入成功，envelope status=success |
| T-R2 | config dry-run | T-R1 通过 | envelope status=success，data 含 dry-run 结果 |
| T-R3 | config execute | T-R2 通过，authorized=True | envelope status=success，data.policy_route_id 非空 |
| T-R4 | verify execute | T-R3 通过 | envelope status=success，data 含策略路由列表 |
| T-R5 | update execute | T-R3 通过，从 verify 获取 id | envelope status=success |
| T-R6 | delete execute（inverse_op 清理） | T-R3 通过 | envelope status=success，策略路由已删除 |
| T-R7 | STATE_INVALID 重连 | 手动删除 session.json 后执行 op | 自动重连成功，op 重试成功 |
| T-R8 | priority execute | T-R3 通过，两个策略路由 id | envelope status=success |
| T-R9 | reset-hitcount execute | T-R3 通过 | envelope status=success |
| T-R10 | verify-hitcount execute | T-R3 通过 | envelope status=success，data 含命中计数 |
| T-R11 | 端到端 PC 执行 | 手写最小 PC（DQ-04） | 解析 -> login -> config -> verify -> delete 清理 -> 日志，全链路 PASS |

### 10.4 测试覆盖矩阵

| op_id | map 单元测试 | build_command 测试 | dry-run 验证 | runtime 执行 | rollback 测试 |
|-------|:---:|:---:|:---:|:---:|:---:|
| fw_login_web_management | T-U1/U2 | T-U5/U6 | T-D2 | T-R1 | T-U12 |
| fw_config_policy_route | T-U1/U3 | T-U5/U6/U7 | T-D1/D2 | T-R2/R3 | T-U10 |
| fw_update_policy_route | T-U1 | T-U5/U6 | T-D2 | T-R5 | handle_rollback restore_snapshot |
| fw_delete_policy_route | T-U1 | T-U5/U6 | T-D2 | T-R6 | T-U13 |
| fw_verify_policy_route | T-U1 | T-U5/U6 | T-D2 | T-R4 | T-U12 |
| fw_update_policy_route_priority | T-U1 | T-U5/U6 | T-D2 | T-R8 | T-U12 |
| fw_reset_policy_route_hitcount | T-U1 | T-U5/U6 | T-D2 | T-R9 | T-U11 |
| fw_verify_policy_route_hitcount | T-U1 | T-U5/U6 | T-D2 | T-R10 | T-U12 |

---

## 11. 实施步骤

### 11.1 前置验证

```bash
# [1] 确认 ptm-atomic 已安装
ptm-atomic --version

# [2] 刷新缓存
ptm-atomic sync --repo <ptm-atomic-git-repository-url>

# [3] 验证 op 可用
ptm-atomic list | grep policy_route

# [4] 验证 CLI help（确认 flag 真相）
ptm-atomic run --base-url https://localhost policy-route config --help
ptm-atomic run --base-url https://localhost auth login --help
```

### 11.2 op_mapper.py 实现顺序

| 步骤 | 内容 | 依赖 |
|------|------|------|
| 1 | 创建 `skills/policy-route-execution/scripts/op_mapper.py` 文件骨架（shebang + docstring + import） | - |
| 2 | 定义 `OpNotFoundError` / `ValidationResult` 异常和类型 | 步骤 1 |
| 3 | 定义 `OP_ID_TO_SUBCOMMAND` 常量（8 个映射） | 步骤 2 |
| 4 | 定义 `ARGS_TO_FLAGS` 常量（8 个 op 的 flag 映射） | 步骤 3 |
| 5 | 定义 `ROLLBACK_STRATEGY` 常量（8 个 op 的回滚策略） | 步骤 4 |
| 6 | 定义 `OP_METADATA` 常量（8 个 op 的元数据缓存） | 步骤 5 |
| 7 | 实现 `map_op_id_to_subcommand()` | 步骤 3 |
| 8 | 实现 `map_args_to_flags()` | 步骤 4 |
| 9 | 实现 `_check_required_flags()` | 步骤 8 |
| 10 | 实现 `build_command()` | 步骤 7/8/9 |
| 11 | 实现 `_build_envelope()` / `_parse_atomic_output()` / `_append_exec_log()` | 步骤 2 |
| 12 | 实现 `_reconnect_and_retry()` | 步骤 11 |
| 13 | 实现 `execute_op()` | 步骤 10/11/12 |
| 14 | 实现 `handle_rollback()` | 步骤 13 |
| 15 | 实现 `validate_mapping_consistency()` | 步骤 3/4/5/6 |
| 16 | 实现 `__main__` CLI 入口（execute / validate / map 子命令） | 步骤 13/14/15 |
| 17 | 运行 `validate_mapping_consistency()` 自检 | 步骤 15 |
| 18 | 编写并运行单元测试（T-U1 ~ T-U13） | 步骤 16 |

### 11.3 SKILL.md 实现顺序

| 步骤 | 内容 |
|------|------|
| 1 | 创建 `skills/policy-route-execution/SKILL.md` 文件骨架（frontmatter + 标题） |
| 2 | 编写 frontmatter（name/description/argument-hint/user-invokable/status=active） |
| 3 | 编写"目标"章节 |
| 4 | 编写"前置条件"章节（ptm-atomic 已安装 + auth login 已完成） |
| 5 | 编写"双层映射表"章节（第一层 8 个 + 第二层 7 个 op + login） |
| 6 | 编写"执行流程"章节（干跑 -> 执行 -> verify 三阶段） |
| 7 | 编写"回滚策略"章节（inverse_op / restore_snapshot / irreversible 豁免 / none） |
| 8 | 编写"错误表"章节（error_type 取值 + CONFIG_REJECTED 子错误码） |
| 9 | 编写"Gotchas"章节（9 条 policy-route 相关 Gotchas） |
| 10 | 编写"参数说明"章节（按 op 分组的参数表） |

### 11.4 SKILL.md Gotchas 清单（覆盖 HLD §16 相关条目）

| Gotcha # | 内容 | 来源 |
|----------|------|------|
| G-1 | CLI 名是 `ptm-atomic` 不是 `atomic-ops` | HLD #1 |
| G-2 | 扁平格式硬报错，必须用嵌套子命令 | HLD #2 |
| G-3 | 三层命名不一致（src_addr ≠ source_network ≠ --source-network） | HLD #3 |
| G-4 | rollback 字段名是 `rollback` 不是 `rollback_strategy` | HLD #4 |
| G-5 | reset-hitcount 是 irreversible，不回滚 | HLD #5 |
| G-6 | session 由 --session-file 自动管理，不自铸 idempotency_key | HLD #6 |
| G-7 | login 签名是 --password-env 不是 --password | HLD #7 |
| G-8 | update 需要 --id（从 verify 查询获取） | HLD #10 |
| G-9 | 入接口必须路由模式（ePolicyRouteInIfModeError 需人工 Web 改） | HLD #11 |

### 11.5 验证顺序

```bash
# [1] 映射表一致性自检
python skills/policy-route-execution/scripts/op_mapper.py validate
# 预期: ValidationResult: PASS

# [2] 单元测试
python -m pytest tests/test_op_mapper.py -v
# 预期: 13 个测试全 PASS

# [3] dry-run 命令格式验证（不需设备）
python skills/policy-route-execution/scripts/op_mapper.py map \
  --op-id fw_config_policy_route \
  --args '{"src_addr":"10.0.0.0/24","in_interface":"GE0_12","type":"ipv4"}'
# 预期: 输出正确的命令列表

# [4] runtime 端到端（需 hg3250-51 + DQ-01 授权）
# 由 CP7 runtime 验证执行
```

---

## 12. 风险、难点与预研建议

### 12.1 风险

| # | 风险 | 等级 | 影响 | 缓解 | 回退 |
|---|------|------|------|------|------|
| R-1 | 三层映射漂移 | 高 | 用例无法执行（op_id/args 映射错误） | 映射表来源锁定三处真相源；`validate_mapping_consistency()` static 校验 | op_mapper.py 单点修正映射表 |
| R-2 | ptm-atomic update 命令 --id 未在 argparse 定义 | 中 | update execute 可能报 argparse 错误 | op_mapper 按 HLD §4.4 生成 --id；CP7 runtime 验证；如 ptm-atomic 报错则反馈 ptm-atomic 侧修复 | 临时跳过 update runtime 验证，留 follow-up |
| R-3 | STATE_INVALID 重连失败 | 中 | 后续 op 全部失败 | 最多重试 1 次；重连失败返回 AUTH_FAILED 终止 | 手动 auth login 后重试 |
| R-4 | ePolicyRouteInIfModeError | 低 | config/update 执行失败 | SKILL 错误表明示需人工 Web 改接口模式 | 人工 Web 修改接口为路由模式后重试 |
| R-5 | 设备不可达 | 中 | runtime 验证受阻 | 降级 dry-run-only | CP7 runtime 转 follow-up |
| R-6 | ptm-atomic CLI 输出格式变化 | 低 | _parse_atomic_output 解析失败 | 统一 `--format json`；解析失败返回 UNKNOWN_ERROR | 更新 _parse_atomic_output 适配新格式 |
| R-7 | irreversible 类残留 | 中 | reset-hitcount 后环境不洁 | SKILL 注明豁免；用例设计规避（reset 放末尾） | 手动 Web 查看命中计数 |

### 12.2 难点

| # | 难点 | 说明 | 解决方案 |
|---|------|------|---------|
| D-1 | 三层命名翻译 centralize | ptm-tde args / op yaml params / CLI flag 三层命名不一致，映射逻辑不能分散 | ARGS_TO_FLAGS 单一 dict 承载全部翻译；漂移时单点修正 |
| D-2 | update --id 的获取 | update 需要先 verify 查询获取 id，不能直接按内容更新 | op_mapper 只负责映射 --id flag；id 的获取由 ptm-te agent 编排流程在 verify 后提取并传入 args |
| D-3 | STATE_INVALID 重连的幂等性 | 重连后重试原命令，但原命令可能已部分执行 | config op 的 idempotent=true（ptm-atomic 保证）；非幂等 op（delete）重试前需 verify 确认状态 |
| D-4 | delete 作为清理动作 vs 误删回滚 | delete 自身 rollback=restore_snapshot，但作为 config 的清理动作时不触发回滚 | ROLLBACK_STRATEGY 中 delete 标记 `as_cleanup_skip=True`；handle_rollback 检查此标记 |

### 12.3 预研建议

| # | 预研项 | 目的 | 时机 |
|---|--------|------|------|
| PRE-1 | ptm-atomic update --id argparse 定义 | 确认 `_add_common_args` 是否应添加 `--id` 或 update 是否应使用独立 `_add_update_args` | S3 实现前 |
| PRE-2 | ptm-atomic `--format json` 输出稳定性 | 确认 JSON 输出格式（字段名/嵌套结构）在 dry-run 和 execute 模式下一致 | S3 实现前 |
| PRE-3 | session.json 跨设备复用安全性 | 确认同一 session.json 不会被多设备并发使用（首期单设备无此风险） | T-01 follow-up |

---

## 13. 回滚与发布策略

### 13.1 回滚策略（本 Story 产物回滚，非 op 回滚）

| 场景 | 回滚动作 | 影响范围 |
|------|---------|---------|
| op_mapper.py 映射表错误 | 修正 `OP_ID_TO_SUBCOMMAND` / `ARGS_TO_FLAGS` / `ROLLBACK_STRATEGY` 常量，重新运行 `validate_mapping_consistency()` | 单文件 |
| SKILL.md 内容错误 | 修正对应章节 | 单文件 |
| 整个 Story 回滚 | 删除 `skills/policy-route-execution/` 目录 | 2 个文件 |
| 已安装到目标项目 | `meta-flow uninstall` 或手动删除 `skills/policy-route-execution/` | 目标项目 skill 目录 |

### 13.2 发布策略

| 项 | 策略 |
|----|------|
| 发布形态 | Skill 文件（SKILL.md + scripts/op_mapper.py），随 ptm-team delivery 交付 |
| 安装方式 | `meta-flow install claude/codex/qoder --component agent`（STORY-024-04 安装器集成后） |
| 版本对齐 | op_mapper.py 映射表版本与 ptm-atomic op 版本对齐（`since_version` 见 op yaml） |
| 前置依赖 | ptm-atomic >= 0.1.0 已安装且已 sync |
| runtime 授权 | `--execute` 写操作需 CP2 DQ-01 单次授权（ADR-04） |

### 13.3 inverse_op 回滚策略（op 级，HLD §9 锁定）

| op_id | rollback 类型 | 回滚动作 | 真相源 |
|-------|-------------|---------|--------|
| fw_config_policy_route | inverse_op | 执行 `fw_delete_policy_route` 清理（用 config 返回的 policy_route_id） | `ptm-atomic list`: `inverse_op:fw_delete_policy_route` |
| fw_update_policy_route | restore_snapshot | 按 before 快照恢复原值（`pre_snapshot.full_config`） | `ptm-atomic list`: `restore_snapshot` |
| fw_delete_policy_route | restore_snapshot（as_cleanup_skip） | 作为 config 清理动作时不触发回滚；误删时按快照恢复 | `ptm-atomic list`: `restore_snapshot` |
| fw_verify_policy_route | none | 只读，不回滚 | `ptm-atomic list`: 空 |
| fw_update_policy_route_priority | none | 无 rollback 元数据，由用例设计决定 | `ptm-atomic list`: 空 |
| fw_reset_policy_route_hitcount | irreversible | 不回滚，SKILL 注明豁免 | `ptm-atomic list`: `irreversible` |
| fw_verify_policy_route_hitcount | none | 只读，不回滚 | `ptm-atomic list`: 空 |
| fw_login_web_management | none | 只读，不回滚 | observation |

### 13.4 irreversible 豁免说明

`fw_reset_policy_route_hitcount` 是 `irreversible`（命中计数清零不可恢复）。ptm-te **不强行回滚**此类步骤，在 SKILL 错误表和 Gotchas 中注明：用例设计者需接受该副作用或通过用例顺序规避（如 reset 放在用例末尾）。

---

## 14. Definition of Done

### 14.1 量化验收项

| # | 验收项 | 度量值 | 验证方法 |
|---|--------|--------|---------|
| DoD-1 | op_id->子命令映射覆盖 | 8 个 op_id 全覆盖 | `validate_mapping_consistency()` passed=True；`len(OP_ID_TO_SUBCOMMAND) == 8` |
| DoD-2 | args->flag 映射覆盖 | 7 个 op 的 flag 映射 + login | `len(ARGS_TO_FLAGS) == 8`；每个 op 的 flag 与 `run_policy_route.py` `_add_*_args()` + CLI `--help` 一致 |
| DoD-3 | 三层映射一致性校验 PASS | `validate_mapping_consistency()` passed=True | 运行 `python op_mapper.py validate` 输出 PASS |
| DoD-4 | 三表 op_id 一致 | OP_ID_TO_SUBCOMMAND / ARGS_TO_FLAGS / ROLLBACK_STRATEGY key 集合相同 | `validate_mapping_consistency()` 无 mismatches |
| DoD-5 | rollback 策略与 ptm-atomic list 一致 | 8 个 op 的 ROLLBACK_STRATEGY.type 与 `ptm-atomic list` rollback 字段一致 | T-S6 校验 PASS |
| DoD-6 | dry-run 默认门 | `execute_op` 默认 `dry_run=True` | T-U8 校验：`dry_run=False, authorized=False` 返回 `EXEC_FAILED` |
| DoD-7 | STATE_INVALID 重连 | 检测 STATE_INVALID 自动 auth login + 重试 1 次 | T-R7 runtime 验证 PASS |
| DoD-8 | inverse_op 清理覆盖 | config->delete 清理 | T-U10 校验 PASS；T-R6 runtime 验证 PASS |
| DoD-9 | restore_snapshot 覆盖 | update->按快照恢复 | handle_rollback restore_snapshot 逻辑实现 |
| DoD-10 | irreversible 豁免 | reset-hitcount 不回滚，返回 waived | T-U11 校验 PASS |
| DoD-11 | delete as_cleanup_skip | delete 作为 config 清理不触发回滚 | T-U13 校验 PASS |
| DoD-12 | 未识别 op_id 阻塞 | 返回 OP_NOT_FOUND | T-U9 校验 PASS |
| DoD-13 | envelope 契约完整 | op_id/step_name/status/data/error_type/diag_snapshot_ref 6 字段 | _build_envelope 输出校验 |
| DoD-14 | SKILL.md 文档完整 | 含映射表 + 流程 + 错误表 + Gotchas（9 条）+ 回滚策略 | 章节结构对照 §4.2 |
| DoD-15 | 单元测试 PASS | T-U1 ~ T-U13 共 13 个测试 | `pytest` 全 PASS |
| DoD-16 | 无外部依赖 | 仅标准库 | `import` 行无第三方包 |
| DoD-17 | Python >=3.9 兼容 | 无 3.10+ 专有语法 | `python3.9 -c "import op_mapper"` 不报错 |
| DoD-18 | 文件影响范围 | 2 个文件（SKILL.md + op_mapper.py），均为新建 | git diff 确认无修改现有文件 |

### 14.2 CP6 编码完成检查项

- [ ] `skills/policy-route-execution/SKILL.md` 已创建，章节结构完整
- [ ] `skills/policy-route-execution/scripts/op_mapper.py` 已创建，10 个函数全部实现
- [ ] `OP_ID_TO_SUBCOMMAND` 覆盖 8 个 op_id
- [ ] `ARGS_TO_FLAGS` 覆盖 8 个 op 的 flag 映射
- [ ] `ROLLBACK_STRATEGY` 覆盖 8 个 op 的回滚策略
- [ ] `OP_METADATA` 覆盖 8 个 op 的元数据
- [ ] `validate_mapping_consistency()` 自检 PASS
- [ ] 单元测试 T-U1 ~ T-U13 全 PASS
- [ ] 无 `__pycache__/` / `*.pyc` 缓存文件入库
- [ ] 无外部第三方依赖

### 14.3 CP7 验证完成检查项

- [ ] **static**：映射表一致性校验 PASS（T-S1 ~ T-S7）
- [ ] **static**：单元测试 PASS（T-U1 ~ T-U13）
- [ ] **static**：dry-run 命令格式验证 PASS（T-D1 ~ T-D3）
- [ ] **runtime**：auth login + config dry-run + execute（hg3250-51）PASS（T-R1 ~ T-R3）
- [ ] **runtime**：STATE_INVALID 重连验证 PASS（T-R7）
- [ ] **runtime**：端到端 PC 执行 PASS（T-R11）
- [ ] 验证结论：PASS / PASS_WITH_RISK / NEEDS_REWORK

### 14.4 Agent Dispatch Evidence

| 字段 | 值 |
|------|------|
| mode | inline-fallback |
| canonical_role | meta-dev（预期） |
| codex_agent_name | - |
| reasoning_profile | - |
| dispatch_trigger | CP5 approved -> story-execution |
| tool_name | - |
| fallback_reason | 当前仓库未安装 meta-dev 提示词；LLD 由 Host Orchestrator 基于 HLD-CR-024 v1.1 直接产出 |
| approved_by | pending（CP5 人工门禁时用户审查 LLD 确认） |
| approved_at | pending |

> inline-fallback 说明：本 LLD 基于 HLD-CR-024 v1.1 §4 三层映射 + §9 回滚策略由 Host Orchestrator 主进程产出，非 meta-dev 独立完成。CP5 人工门禁时用户审查 LLD 质量；实现阶段由 meta-dev 或 inline-fallback 执行编码。

---

## 附录 A：三层映射真相源对照（完整）

### A.1 第一层映射真相源：run_policy_route.py build_subtree()

```python
# run_policy_route.py build_subtree() 返回的 CommandSpec（真相源）
CommandSpec(name="config",         ...)  # -> fw_config_policy_route
CommandSpec(name="update",         ...)  # -> fw_update_policy_route
CommandSpec(name="delete",         ...)  # -> fw_delete_policy_route
CommandSpec(name="verify",         ...)  # -> fw_verify_policy_route
CommandSpec(name="reset-hitcount", ...)  # -> fw_reset_policy_route_hitcount
CommandSpec(name="verify-hitcount",...)  # -> fw_verify_policy_route_hitcount
CommandSpec(name="priority",       ...)  # -> fw_update_policy_route_priority
# auth login 来自 run_auth.py（非 policy-route 子树）
```

### A.2 第二层映射真相源：run_policy_route.py _add_*_args() + op yaml inputs.params + CLI --help

| op_id | _add_*_args 函数 | op yaml inputs.params（核心） | CLI --help flags（核心） | ARGS_TO_FLAGS 映射 |
|-------|------------------|-----------------------------|------------------------|-------------------|
| fw_config_policy_route | `_add_common_args` | source_network, dst_network, next_hop_ip, in_interface, type | --source-network, --dst-network, --next-hop-ip, --in-interface, --policy-route-type | src_addr->--source-network 等 5 个 |
| fw_update_policy_route | `_add_common_args` + `_require_arg(id)` | id, source_network, ..., type | 同 config + --id（注：help 未暴露 --id，见 Gotcha） | 同 config + id->--id |
| fw_delete_policy_route | `_add_delete_args` | id, is_ip6 | --id, --policy-route-type | id->--id, type->--policy-route-type |
| fw_verify_policy_route | `_add_verify_args` | -（CLI 分页参数） | --policy-route-type, --page, --size | type/page/size |
| fw_update_policy_route_priority | `_add_priority_args` | targetsite, targetid, moveid | --targetsite, --targetid, --moveid, --policy-route-type | type/moveid/targetid/targetsite |
| fw_reset_policy_route_hitcount | `_add_delete_args` | id, is_ip6 | --id, --policy-route-type | id->--id, type->--policy-route-type |
| fw_verify_policy_route_hitcount | `_add_verify_args` | - | --policy-route-type, --page, --size | type/page/size |
| fw_login_web_management | run_auth.py `_add_login_args` | username, password_env | --username, --password-env | username/password_env |

### A.3 rollback 真相源：ptm-atomic list（2026-07-10 实测）

```
op_id                              side_effect      rollback
---------------------------------  ---------------  ---------------------------------
fw_config_policy_route             state_mutation   inverse_op:fw_delete_policy_route
fw_delete_policy_route             destructive      restore_snapshot
fw_reset_policy_route_hitcount     state_mutation   irreversible
fw_update_policy_route             state_mutation   restore_snapshot
fw_update_policy_route_priority    (空)             (空)
fw_verify_policy_route             observation      (空)
fw_verify_policy_route_hitcount    observation      (空)
fw_login_web_management            observation      (空)
```

---

## 附录 B：Gotchas 完整清单

| # | Gotcha | 影响 | 规避 |
|---|--------|------|------|
| 1 | CLI 名是 `ptm-atomic` 不是 `atomic-ops` | manaul 旧 SKILL `atomic-ops run` 报 command not found | op_mapper 硬编码 `ptm-atomic` |
| 2 | 扁平格式 `ptm-atomic run <op_id>` 硬报错 | 报 `invalid choice`，无 deprecated warning | 必须用嵌套子命令 `ptm-atomic run --base-url <url> <family> <action>` |
| 3 | 三层命名不一致 | src_addr ≠ source_network ≠ --source-network | ARGS_TO_FLAGS 承载翻译，不假设三层同名 |
| 4 | rollback 字段名是 `rollback` 不是 `rollback_strategy` | ptm-atomic list 真实字段是 `side_effect` + `rollback` | OP_METADATA 用 `rollback` 字段名 |
| 5 | reset-hitcount 是 irreversible | 命中计数清零不可恢复，不回滚 | ROLLBACK_STRATEGY type=irreversible，SKILL 注明 |
| 6 | session 由 --session-file 自动管理 | 无需自铸 idempotency_key/state_ref/session_ref | op_mapper 只传 --session-file 路径 |
| 7 | login 签名是 --password-env 不是 --password | 禁止命令行明文密码 | ARGS_TO_FLAGS login: password_env->--password-env |
| 8 | update 需要 --id | 不能直接按内容更新，需先 verify 查询 id | op_mapper 映射 id->--id；id 由 ptm-te agent 从 verify 结果提取 |
| 9 | 入接口必须路由模式 | ePolicyRouteInIfModeError 需人工 Web 改 | SKILL 错误表明示，op_mapper 不自动修复 |
| 10 | update --id 在 CLI help 未暴露 | `_add_common_args` 未定义 --id argparse（疑似 ptm-atomic 待修复） | op_mapper 按 HLD §4.4 生成 --id；CP7 runtime 验证确认 |
| 11 | delete 是 restore_snapshot 不是 observation | delete 实测 rollback=restore_snapshot（destructive） | OP_METADATA delete: rollback=restore_snapshot |
| 12 | delete 作为清理动作不触发回滚 | delete 本身 rollback=restore_snapshot，但作为 config 清理时不回滚 | ROLLBACK_STRATEGY delete: as_cleanup_skip=True |
| 13 | priority 无 rollback 元数据 | side_effect 和 rollback 均为空，由用例设计决定 | ROLLBACK_STRATEGY priority: type=none |

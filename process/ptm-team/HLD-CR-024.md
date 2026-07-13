---
doc: HLD-CR-024
change_id: CR-024-ptm-te-agent
workflow_id: WF-PTM-TEAM-20260520-001
title: ptm-te 测试执行工程师 Agent 高层设计
status: draft
version: 1.1
created_at: "2026-07-10T00:00:00+08:00"
author: host-orchestrator（主进程，inline-fallback 基于 plan_ref v4）
plan_ref: /home/hyde/.claude/plans/ptm-te-agent-impl.md（v4）
cp3_auto_precheck: pending
cp3_manual_review: pending
---

# HLD - ptm-te 测试执行工程师 Agent（CR-024）

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|------|------|--------|---------|
| v1.0 | 2026-07-10 | host-orchestrator | 初版。基于计划 v4 §4 架构设计产出。锁定 op_id/args 三层映射（§5）、skill 边界（§6）、3 个 Architecture Gray Areas（§11）、5 个 ADR（§12） |
| v1.1 | 2026-07-10 | host-orchestrator | CP3 评审整改：① §4.3 修正 3 个 op rollback 类型与 `ptm-atomic list` 实测对齐（update/delete->restore_snapshot，priority->空），加 side_effect 列；② §9 回滚策略表同步修正（delete 作为清理动作不触发回滚，priority 无元数据由用例设计决定）；③ §2.2 重新设计运行目录（加 `report.md` 人类可读测试报告）+ 新增用例上传目录 `cases/upload/`（ptm-te 执行入口，与 ppdcs/delivery/ 解耦）；④ §2.1/§3.1 输入路径改 `cases/upload/`；⑤ §16 加 Gotcha #13 |

## 1. 背景与目标

### 1.1 背景

ptm-team 六 Agent 角色中，ptm-te（测试执行工程师）当前是 `planned` 占位。ptm-tde（测试设计工程师）已交付，产出物理用例（PC）于 `ppdcs/delivery/<特性名>特性测试用例.md`，含结构化 `case_steps[].atomic_op.op_id`。ptm-te 是 ptm-tde 的下游执行器，负责消费 PC 并在真实设备上执行原子操作。

### 1.2 目标（量化成功标准）

| # | 成功标准 | 度量值 |
|---|---------|--------|
| SC-1 | ptm-te agent 升格 active，含完整编排流程 | `agents/ptm-te.md` status=active，color=green，编排流程 ≥ 6 步 |
| SC-2 | 设备管理能力交付 | device-management + device-connection 两个 skill，SSH/Telnet 双轨 + 回退，快照采集 ≥ before/after 两点 |
| SC-3 | 策略路由执行能力交付 | policy-route-execution skill 覆盖 **7 个 op**（config/update/delete/verify/priority/reset-hitcount/verify-hitcount）的 flag 映射 |
| SC-4 | op_mapper 双层映射 | op_mapper.py 覆盖 8 个 op_id（含 login）-> 子命令映射 + 7 个 op 的 args->flag 映射，与 `run_policy_route.py` + op yaml 一致性校验 PASS |
| SC-5 | login-once-reuse-session | `auth login` 一次，session.json 复用，STATE_INVALID 自动重连 |
| SC-6 | inverse_op 回滚 | config 的 inverse_op=delete 清理；irreversible 类（reset-hitcount）注明豁免 |
| SC-7 | 安装器集成 | install.py 投影 ptm-te + 3 skill，三平台（claude/codex/qoder）dry-run PASS |
| SC-8 | 验证 | CP7 mixed：static（映射一致性 + 安装 dry-run）+ runtime（hg3250-51 端到端 PC 执行） |

### 1.3 非目标（Out of Scope）

- 进程管理 / 串口初始化（T-04 candidate）
- `fw_config_batch_policy_route` package 级编排（T-05 candidate）
- 1900 型号设备验证（T-02 candidate）
- 真实消费 ptm-tde 产出 PC（T-01 candidate，首期用手写最小 PC，CP2 DQ-04）
- managed rule block 注入（CP2 DQ-03 v1 不注入）
- 修改 ptm-tde 已交付基线（不授权项）

## 2. ptm-te 定位与编排模型

ptm-te 采用与 ptm-tde 一致的**编排器模式**，是 ptm-tde 的下游执行器。

### 2.1 编排流程

```
用户上传 PC 到 cases/upload/（手写最小 PC，DQ-04；或复制 ptm-tde 产出 PC，T-01）
  ↓
[1] 用例解析：读取 cases/upload/<特性名>特性测试用例.md，提取结构化 case_steps + expected_result
  ↓
[2] 设备准备：device-management 加载 devices.yaml + 型号映射查表
     └─ device-connection SSH 连接探测（失败回退 Telnet）+ 系统快照 before
  ↓
[3] login 一次：ptm-atomic run --base-url <url> auth login --username admin --password-env FW_WEB_PASSWORD
     └─ 持久化 --session-file，后续复用
  ↓
[4] 逐条原子操作执行：
     op_mapper（op_id->子命令 + args->flag）
       └─ 干跑（--dry-run）-> 执行（--execute，需 DQ-01 授权）-> verify
  ↓
[5] 结果判定：envelope（status=success / error_type=NONE + Check 点 vs expected_result）
  ↓
[6] 执行日志：结构化，分步骤 + API 状态码 + envelope
  ↓
[7] 用例清理：inverse_op 回滚
     ├─ config 的 inverse_op = policy-route delete
     ├─ restore_snapshot 按快照恢复
     └─ irreversible 类（reset-hitcount）不接受回滚，由用例设计承担
  ↓
[8] 快照 after + 结果回写
```

### 2.2 运行时工作目录与用例上传目录

ptm-te 执行用例时的工作目录结构（用户工作区，不入库）。**用例上传目录** `cases/upload/` 是 ptm-te 的执行入口，与 ptm-tde 产出目录 `ppdcs/delivery/` 解耦：用户手写最小 PC（DQ-04）或复制 ptm-tde 产出 PC（T-01）到 `cases/upload/`，ptm-te 只从此目录读取待执行用例。

```
<workspace>/
├── devices.yaml              # 设备清单（${ENV_VAR} 占位凭据，.gitignore 忽略）
├── .env                      # 凭据环境变量（.gitignore 忽略）
├── cases/                    # 用例目录
│   └── upload/               # 用例上传目录（ptm-te 执行入口；手写 PC 或复制 ptm-tde PC 到此）
│       └── <特性名>特性测试用例.md
└── runs/                     # 运行产物目录
    └── <run-id>/             # 单次用例执行产物
        ├── parse-result.json # 用例解析结果（机器可读）
        ├── snapshot-before/  # 设备快照 before
        ├── exec-log.jsonl    # 逐条 op 执行日志（机器可读，JSONL：step_name/op_id/status/error_type/API 状态码）
        ├── snapshot-after/   # 设备快照 after
        ├── result.json       # 用例结果回写（机器可读，每条 step envelope + 用例整体 PASS/FAIL）
        └── report.md         # 人类可读测试报告（逐步骤状态 + envelope 摘要 + 快照 diff 摘要 + 清理记录）
```

| 目录 | 用途 | 读/写 | 说明 |
|------|------|-------|------|
| `cases/upload/` | 用例上传目录 | 读（ptm-te） | ptm-te 执行入口；用户上传手写 PC（DQ-04）或复制 ptm-tde 产出 PC（T-01） |
| `ppdcs/delivery/` | ptm-tde 产出目录 | 不直接读 | ptm-tde 产出 PC 的源头；ptm-te 不直接耦合，用户按需复制到 `cases/upload/` |
| `runs/<run-id>/` | 单次运行产物 | 写（ptm-te） | 机器可读（result.json/exec-log.jsonl）+ 人类可读（report.md） |

## 3. 与 ptm-tde 的消费契约（集成契约显式化）

### 3.1 集成契约

| 维度 | 契约 |
|------|------|
| 调用方向 | ptm-te 消费 ptm-tde 产出（单向，ptm-te 不回调 ptm-tde） |
| 调用时机 | ptm-tde 完成 PC 交付（ppdcs/delivery/）后，ptm-te 读取执行 |
| 触发方式 | 用户指令 ptm-te 执行指定 PC 文件 |
| 输入契约 | `cases/upload/<特性名>特性测试用例.md`（用户上传/手写 PC；ptm-tde 产出 PC 可复制到此，T-01 切换后），含 `case_steps[].step_name`、`case_steps[].atomic_op.op_id`、`case_steps[].atomic_op.args`、`case_steps[].expected_result`（CR-019 定义） |
| 输出契约 | `runs/<run-id>/result.json`：每条 step 的 status/error_type/diag_snapshot_ref + 用例整体 PASS/FAIL |
| 后续衔接 | 失败 step 反馈 ptm-tae（工具缺失）或 ptm-tde（用例设计问题） |
| 降级策略 | op_id 未识别 -> 阻塞并提示工具缺失；设备不可达 -> 降级 dry-run-only；session 失效 -> 自动重连 |
| 调用方需同步修改范围 | 无。ptm-te 不要求 ptm-tde 修改（args 翻译由 op_mapper 承载，见 §5） |

### 3.2 消费结构化 case_steps（非 16 列汇总表）

ptm-te 消费**结构化 `case_steps`**（CR-019 契约）。16 列 PC 汇总表仅作人工 fallback，不作为机器消费入口。

### 3.3 envelope 契约

每条 op 执行结果封装为 envelope：

```json
{
  "op_id": "fw_config_policy_route",
  "step_name": "配置策略路由",
  "status": "success | error",
  "data": { /* ptm-atomic 返回 */ },
  "error_type": "NONE | STATE_INVALID | OP_NOT_FOUND | EXEC_FAILED | ...",
  "diag_snapshot_ref": "runs/<run-id>/snapshot-before/<step>.json"
}
```

默认 `--dry-run`（CP2 DQ-01），`--execute` 才真执行（需单次授权）。

## 4. op_id/args 三层映射锁定（CP3 核心锁定项，SGQ-04）

### 4.1 三层命名不一致问题

ptm-tde 产出 PC 的 `args` 字段名 ≠ ptm-atomic op yaml `inputs.params` ≠ ptm-atomic CLI flag：

| 层 | 来源 | 示例 |
|----|------|------|
| 第 1 层（ptm-tde PC args） | `case_steps[].atomic_op.args` | `src_addr` |
| 第 2 层（op yaml params） | `atoms/fw/fw_*.yaml` 的 `inputs.params` | `source_network` |
| 第 3 层（CLI flag） | `ptm-atomic run ... policy-route config --*` | `--source-network` |

### 4.2 锁定决策（ADR-01）

**ptm-te 的 `op_mapper.py` 承载三层映射翻译**。ptm-tde args 命名**不变**（保持已交付基线，符合不授权项"不修改 ptm-tde 基线"）。

理由：
- ptm-tde 已交付（CR-010~017 closed），改其 args 命名会破坏已交付基线
- op_mapper.py centralize 映射逻辑，漂移时单点修正
- 三层映射是 ptm-te 的消费适配职责，不应反向要求上游改名

### 4.3 第一层映射：op_id -> CLI 子命令

op_id 到 `family action` 的映射在 `run_policy_route.py` 中硬编码。op_mapper 必须覆盖全部 8 个 op_id：

| op_id（ptm-tde PC） | ptm-atomic CLI 子命令 | side_effect | rollback（ptm-atomic list 真相） |
|---|---|---|---|
| `fw_login_web_management` | `auth login` | observation | （空，建立 session，只读） |
| `fw_config_policy_route` | `policy-route config` | state_mutation | `inverse_op:fw_delete_policy_route` |
| `fw_update_policy_route` | `policy-route update` | state_mutation | `restore_snapshot`（按 before 快照恢复原值） |
| `fw_delete_policy_route` | `policy-route delete` | destructive | `restore_snapshot`（误删可按快照恢复；作为 config 清理动作时不触发回滚） |
| `fw_verify_policy_route` | `policy-route verify` | observation | （空，只读） |
| `fw_update_policy_route_priority` | `policy-route priority` | （空） | （空，无 rollback 元数据，由用例设计决定是否恢复原优先级） |
| `fw_reset_policy_route_hitcount` | `policy-route reset-hitcount` | state_mutation | `irreversible`（命中计数清零不可恢复） |
| `fw_verify_policy_route_hitcount` | `policy-route verify-hitcount` | observation | （空，只读） |

> rollback 真相源：`ptm-atomic list`（2026-07-10 实测）。`fw_config_batch_policy_route` 属 package 级编排，不走 `run` 单条命令，首期 N/A（T-05）。

### 4.4 第二层映射：args -> CLI flag（覆盖全部 7 个 op）

**config 类（config / update）**：

| ptm-tde PC `args` | op yaml `inputs.params` | ptm-atomic CLI flag |
|---|---|---|
| `src_addr` | `source_network` | `--source-network` |
| `dst_addr` | `dst_network` | `--dst-network` |
| `next_hop` | `next_hop_ip` | `--next-hop-ip` |
| `in_interface` | `in_interface` | `--in-interface` |
| `type` | `type` | `--policy-route-type` |

update 额外带 `--id`（从 verify 查询获取目标策略路由 id）。

**delete / reset-hitcount**：`--id` + `--policy-route-type`（无 ip/network 类 flag）。

**verify / verify-hitcount**：`--policy-route-type` + `--page` + `--size`。

**priority**：`--policy-route-type` + `--moveid` + `--targetid` + `--targetsite`。

**login**：`--username` + `--password-env FW_WEB_PASSWORD`（+ 可选 `--change-default-password` / `--new-password-env`）。

> **S3 范围要求**：op_mapper.py 必须覆盖全部 7 个 op 的 flag 映射（config/update/delete/verify/priority/reset-hitcount/verify-hitcount）+ login。映射表来源锁定三处真相源：`run_policy_route.py`（子命令）+ op yaml `inputs.params`（参数名）+ `ptm-atomic run ... --help`（flag）。static 校验一致性（CP7）。

## 5. Skill 设计与边界（相邻对象边界澄清）

### 5.1 Skill 清单

| Skill | 来源 | 职责 | 脚本 | 边界 |
|-------|------|------|------|------|
| `device-management` | 迁移自 manaul（元数据） | 设备清单（devices.yaml）+ 型号映射查表 | 无（纯元数据 SKILL + reference） | **只做元数据**，不含连接逻辑 |
| `device-connection` | 迁移自 manaul `collect_sysinfo.py` | SSH/Telnet 双轨连接 + 回退 + 系统快照采集 | `scripts/ssh_exec.py`、`scripts/collect_sysinfo.py` | **只做连接 + 快照**，不解析 PC、不执行策略路由 op |
| `policy-route-execution` | 参考 manaul，按 ptm-atomic 真实 CLI 重写 | op_id->子命令 + args->flag 双层映射 + 干跑/执行/verify + inverse_op 清理 | `scripts/op_mapper.py`（必需） | **只执行 policy-route 族 op**，其他 op 族（interface/ospf 等）首期 N/A |

### 5.2 相邻对象边界澄清

| 职责 | 归属 | 差异界定 |
|------|------|---------|
| 设备清单管理 | device-management | 维护 devices.yaml 结构 + 型号查表，不连接设备 |
| 设备连接 | device-connection | SSH/Telnet 建立 + 快照采集，不管理清单 |
| 策略路由执行 | policy-route-execution | 调用 ptm-atomic CLI 执行 policy-route op，不含连接逻辑（连接由 device-connection 提供） |
| 执行门控 | agents/ptm-te.md | 环境就绪/关键判定/异常记录，**不复用 checkpoint-manager**（ptm-tde 专属 GATE-1~5） |
| 用例解析 | agents/ptm-te.md 编排流程 [1] | 解析 PC case_steps，分发到 skill |

### 5.3 checkpoint-manager 不复用（P1-3 修正）

checkpoint-manager 是 ptm-tde 三阶段框架专属门控（GATE-1~GATE-5），真相源 `docs/ptm-tde/gate-spec.md`，非通用。ptm-te 的执行门控（环境就绪/关键判定/异常记录）独立写进 `agents/ptm-te.md`。

## 6. 设备连接模型

| 方式 | 用途 | 机制 | 失败处理 |
|------|------|------|---------|
| SSH | 常态命令、系统快照（首选） | paramiko | SSH 失败回退 Telnet |
| Telnet | SSH 不通设备的兜底 | telnetlib（声明 `>=3.9,<3.13`） | Telnet 失败则标记设备不可达，降级 dry-run-only |
| Web (HTTPS) | 策略路由配置 | `ptm-atomic run` REST；`auth login` 一次持久化 `--session-file`，后续复用 | session 失效（STATE_INVALID）自动重新 login |

### 6.1 连接前置校验

- devices.yaml 存在且凭据为 `${ENV_VAR}` 占位（非明文）
- 环境变量 `FW_WEB_PASSWORD` 等 已设置
- 设备 IP 可达（ping 或 TCP 443 探测）
- 校验失败 -> 终止该设备执行，记录 error_type=ENV_NOT_READY

## 7. 凭据管理

| 项 | 策略 |
|----|------|
| devices.yaml | 不入库明文凭据，用 `${ENV_VAR}` 占位；用户工作区 `.gitignore` 忽略 |
| 模板 | `skills/device-management/templates/devices.yaml.example` 提供模板 |
| 环境变量 | `.env.example` 提供变量清单；用户复制为 `.env`（.gitignore 忽略） |
| Web 密码 | 经 `--password-env FW_WEB_PASSWORD` 传入，与 `auth login` 签名一致；**禁止命令行明文密码** |
| SSH/Telnet 密码 | 同样用 `${ENV_VAR}` 占位，运行时从环境变量读取 |

## 8. login-once-reuse-session 机制

### 8.1 流程

1. 用例执行开始时，`auth login` 一次：`ptm-atomic run --base-url <url> auth login --username admin --password-env FW_WEB_PASSWORD --session-file <path>`
2. session 写入 `--session-file`（默认 `~/.local/state/ptm-atomic/ngfw/session.json`）
3. 后续 `config`/`verify`/`update`/`delete` 复用同一 session
4. 遇 `STATE_INVALID` 错误 -> 自动重新 `auth login` -> 重试当前 op（最多 1 次重试）

### 8.2 前置校验

- `auth login` 成功（返回 status=success）
- session.json 写入成功
- 失败 -> 终止用例，error_type=AUTH_FAILED

## 9. inverse_op 回滚与 irreversible 豁免

### 9.1 回滚策略

用例执行后，按 op 的 `rollback` 字段清理：

| rollback 类型 | 策略 | 示例 op |
|---|---|---|
| `inverse_op:<op_id>` | 执行 inverse_op 清理 | config -> `policy-route delete`（delete 是 config 的 inverse_op） |
| `restore_snapshot` | 按 before 快照恢复 | update（恢复原值）；delete 自身 rollback=restore_snapshot，但作为 config 的清理动作时不触发回滚 |
| `irreversible` | **不回滚**，由用例设计接受或规避 | reset-hitcount（命中计数清零不可恢复） |
| 空（observation / 无元数据） | 不需回滚（只读或无自动回滚） | verify / verify-hitcount / login（只读）；priority（无 rollback 元数据，由用例设计决定是否恢复原优先级） |

### 9.2 irreversible 豁免说明

`fw_reset_policy_route_hitcount` 是 `irreversible`（命中计数清零不可恢复）。ptm-te **不强行回滚**此类步骤，在 SKILL 错误表和 Gotchas 中注明：用例设计者需接受该副作用或通过用例顺序规避（如 reset 放在用例末尾）。

## 10. Architecture Gray Areas（CP3 前置，advisor table）

### AGA-01：三层命名映射策略

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|--------|------|------|---------------|----------------|-----------------------------|
| ptm-te op_mapper 承载翻译（ptm-tde args 不变） | 不破坏 ptm-tde 已交付基线；映射 centralize | op_mapper 复杂度增加；三层漂移需 static 校验 | op_mapper.py + CP7 static 校验 | **推荐**（ADR-01） | 若 ptm-tde 未来大改 args 命名，可同步简化 op_mapper |
| ptm-tde 改用 op yaml 参数名 | 映射简化为两层 | 破坏 ptm-tde 已交付基线；需重新交付 ptm-tde | ptm-tde 全部 PC + agents/ptm-tde.md | 不推荐 | 仅当 ptm-tde 有独立 CR 重构 args 时 |

### AGA-02：device-management 与 device-connection 拆分边界

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|--------|------|------|---------------|----------------|-----------------------------|
| 拆为两个 skill（元数据 + 连接） | 职责单一；连接逻辑可独立复用 | 两个 skill 需协调 | device-management + device-connection | **推荐**（P1-1 修正） | 若连接逻辑极简可合并，但 manaul collect_sysinfo.py 236 行表明连接逻辑不轻 |
| 合并为一个 skill | 单 skill 自包含 | 元数据与连接耦合，不利于复用 | 单 device skill | 不推荐 | - |

### AGA-03：inverse_op 回滚豁免策略

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|--------|------|------|---------------|----------------|-----------------------------|
| irreversible 类不回滚，SKILL 注明 | 符合 op 语义；不强行不可逆操作 | 用例可能留副作用 | SKILL 错误表 + Gotchas | **推荐**（ADR-03） | 若用户要求强清理，可加手动 Web 清理提示 |
| 强行回滚所有 op | 用例环境一致 | 不可逆操作无法真正回滚，误导 | 全部 op 回滚逻辑 | 不推荐 | - |

### AGA 讨论日志与恢复点

- 讨论日志：`process/discussions/CP3-HLD-DISCUSSION-LOG-CR-024.md`（已生成）
- 恢复点：`process/checks/CP3-DISCUSSION-CHECKPOINT-CR-024.json`（已生成）
- 3 个 AGA 均已基于计划 v4 评审确定方向，CP3 人工门禁确认

## 11. ADR（架构决策）

### ADR-01：op_mapper 承载三层映射翻译

- **决策**：ptm-te 的 op_mapper.py 承载 op_id->子命令 + args->flag 双层映射；ptm-tde args 命名不变
- **理由**：不破坏 ptm-tde 已交付基线；映射 centralize 便于维护
- **回写**：§4 映射表、§5 skill 边界、S3 op_mapper.py 实现

### ADR-02：login-once-reuse-session

- **决策**：auth login 一次持久化 session.json，后续 op 复用；STATE_INVALID 自动重连
- **理由**：避免每条 op 重复登录；session 由 ptm-atomic `--session-file` 自动管理
- **回写**：§8 机制、S1 agent 编排流程 [3]

### ADR-03：inverse_op 回滚 + irreversible 豁免

- **决策**：按 op 的 rollback 字段清理；irreversible 类（reset-hitcount）不回滚，SKILL 注明
- **理由**：符合 op 语义；不强行不可逆操作
- **回写**：§9 回滚策略、S3 SKILL 错误表

### ADR-04：dry-run 默认门（来自 CP2 DQ-01）

- **决策**：首期 CP7 默认 `--dry-run`；`--execute` 作为独立 runtime_authorization 决策项单次确认
- **理由**：CP2 用户批准；真实设备写操作需单次确认降低风险
- **回写**：§3.3 envelope 契约、S3 SKILL 干跑/执行、§15 验证策略

### ADR-05：device-management/device-connection 拆分

- **决策**：拆为两个 skill，元数据与连接分离
- **理由**：P1-1 修正；manaul 连接逻辑 236 行不轻；职责单一利于复用
- **回写**：§5 skill 清单、S2 实现

## 12. Story 拆解与文件布局（CP4 细化预览）

| Story | 文件布局 | tier | lld_policy |
|-------|---------|:--:|------|
| S1 | `agents/ptm-te.md`（重写） | S | full-lld |
| S2 | `skills/device-management/{SKILL.md, reference/device-reference.md, templates/devices.yaml.example}` + `skills/device-connection/{SKILL.md, scripts/ssh_exec.py, scripts/collect_sysinfo.py}` | M | full-lld |
| S3 | `skills/policy-route-execution/{SKILL.md, scripts/op_mapper.py}` | M | full-lld |
| S4 | `script/ptm_team/install.py`（改）+ `skills/README.md`（改）+ `docs/ptm-team-blueprint.md`（改） | S | technical-note |

Wave：W1(S1) -> W2(S2‖S3) -> W3(S4)。CP4 生成 DEVELOPMENT-PLAN.yaml 细化依赖与文件所有权。

## 13. 开发计划与完成准则

| Story | 完成准则 |
|-------|---------|
| S1 | agents/ptm-te.md status=active, color=green；编排流程 [1]-[8] 完整；PC 消费契约 + 三层映射 + login-once-reuse + inverse_op 清理 + 执行门控写入 |
| S2 | device-management 元数据 SKILL + devices.yaml.example（${ENV_VAR} 占位）+ device-reference.md 型号映射；device-connection SSH/Telnet 双轨 + 回退 + 快照采集脚本（声明 >=3.9,<3.13） |
| S3 | policy-route-execution SKILL（7 op flag 映射 + 干跑/执行/verify + inverse_op 清理 + 错误表 + Gotchas）+ op_mapper.py（8 op_id 映射 + 7 op flag 映射）；ptm-atomic sync 前置验证 |
| S4 | install.py AGENT_ALIASES 加 te/ptm-te + get_agent_skills 加 ptm-te 分支返回 3 skill；v1 不注入 rule block；skills/README.md 索引；blueprint 路标回写；三平台 dry-run PASS |

## 14. 验证策略（CP7 输入）

`validation_mode=mixed`：

**static**：
- SKILL.md 结构检查、agent frontmatter 校验（color: green）
- 安装器 dry-run（三平台）+ `meta-flow install-check`
- Python 脚本 lint / 单元测试（op_mapper 映射、ssh_exec、collect_sysinfo）
- **op_id->子命令 + args->flag 映射表与 run_policy_route.py + op yaml 一致性校验**
- inverse_op 回滚契约校验（config->delete、irreversible 豁免）

**runtime**（需 DQ-01 授权 + DQ-02 设备 hg3250-51）：
- `auth login` + `policy-route config` dry-run + execute（hg3250-51）
- login-once-reuse-session 验证（session.json 复用、STATE_INVALID 重连）
- 设备 SSH/Telnet 连接 + 系统快照采集
- 手写最小 PC 端到端执行（DQ-04）：解析 -> 设备准备 -> login -> op_mapper 映射 -> 执行 -> verify -> inverse_op 清理 -> 日志

产出 `docs/quality/VERIFICATION-REPORT.md` + `TEST-REPORT.md`。

## 15. 风险与回退

| 风险 | 等级 | 缓解 | 回退路径 |
|------|------|------|---------|
| op_id/args 三层映射漂移 | 高 | 映射表来源锁定三处真相源；static 校验一致性 | op_mapper.py 单点修正 |
| login session 失效 | 中 | STATE_INVALID 自动重连 | session 重建 |
| inverse_op 回滚缺失致残留 | 中 | 用例清理强制 delete；irreversible 注明 | 手动 Web 清理 |
| ePolicyRouteInIfModeError | 低 | SKILL 前置校验接口模式；错误表明示 | 人工 Web 改 |
| 真实设备不可达 | 中 | 降级 dry-run-only，runtime 留 follow-up | CP7 runtime 转 T-01 candidate |
| telnetlib Python 3.13 移除 | 低 | 脚本声明 >=3.9,<3.13 | 改用 telnetlib3 |
| 凭据泄露 | 高 | devices.yaml 不入库明文；--password-env | 凭据轮换 |
| ptm-te 与 ptm-tde 契约漂移 | 中 | CP3 锁定 PC 字段契约 + 三层映射写入 agent md | 契约对齐 CR |

## 16. Gotchas

1. **CLI 名是 `ptm-atomic` 不是 `atomic-ops`**：Python 模块叫 `atomic_ops`，但命令名是 `ptm-atomic`。manaul 旧 SKILL 用 `atomic-ops run` 会报 command not found。
2. **扁平格式硬报错**：`ptm-atomic run fw_config_policy_route` 报 `invalid choice`，无 deprecated warning。必须用嵌套子命令 `ptm-atomic run --base-url <url> policy-route config`。
3. **三层命名不一致**：ptm-tde PC `src_addr` ≠ op yaml `source_network` ≠ CLI `--source-network`。op_mapper 必须承载翻译，不能假设三层同名。
4. **rollback 字段名是 `rollback` 不是 `rollback_strategy`**：`ptm-atomic list` 真实字段是 `side_effect` + `rollback`。
5. **reset-hitcount 是 irreversible**：`fw_reset_policy_route_hitcount` 命中计数清零不可恢复，不强行回滚。
6. **session 由 --session-file 自动管理**：ptm-te 无需自铸 idempotency_key/state_ref/session_ref，CLI 层已展平。
7. **login 签名是 --password-env 不是 --password**：`auth login --username admin --password-env FW_WEB_PASSWORD`，禁止命令行明文密码。
8. **device-management 不含连接逻辑**：manaul 的连接在 collect_sysinfo.py，不在 device-management SKILL。拆分迁移时不要把连接塞进元数据 SKILL。
9. **checkpoint-manager 是 ptm-tde 专属**：不要复用到 ptm-te，执行门控独立写进 agent md。
10. **update 需要 --id**：`policy-route update` 必须带 `--id`（从 verify 查询获取），不能直接按内容更新。
11. **入接口必须路由模式**：`ePolicyRouteInIfModeError` 表示 in_interface 非路由模式，需人工 Web 改接口模式后重试。
12. **devices.yaml 不入库明文**：凭据必须 `${ENV_VAR}` 占位，manaul devices.yaml 明文凭据是反模式。
13. **delete 是 restore_snapshot 不是 observation**：`fw_delete_policy_route` 实测 rollback=restore_snapshot（destructive），不是只读。但 delete 作为 config 的清理动作时不触发回滚（它本身就是清理）。`fw_update_policy_route_priority` 无 rollback 元数据，由用例设计决定是否恢复原优先级。回滚类型必须以 `ptm-atomic list` 实测为准，不要凭 op 名字推断。

## 17. 开放问题与遗留状态

| ID | 问题 | 状态 | 决策引用 |
|----|------|------|---------|
| O-01 | ptm-tde args 命名是否统一为 op yaml 参数名 | RESOLVED | ADR-01：ptm-te op_mapper 翻译，ptm-tde 不变 |
| O-02 | device-management 与 device-connection 是否合并 | RESOLVED | ADR-05：拆分 |
| O-03 | irreversible 类是否强行回滚 | RESOLVED | ADR-03：不回滚，SKILL 注明 |
| O-04 | runtime 写操作授权 | RESOLVED | CP2 DQ-01：dry-run 默认门（ADR-04） |
| O-05 | 验证设备范围 | RESOLVED | CP2 DQ-02：hg3250-51 |
| O-06 | 用例来源 | RESOLVED | CP2 DQ-04：手写最小 PC 优先 |
| O-07 | rule block 注入 | RESOLVED | CP2 DQ-03：v1 不注入 |
| O-08 | 真实消费 ptm-tde PC 何时启用 | OPEN | T-01 candidate，CP3 锁定 args 后可切 |

## 18. Agent Dispatch Evidence

| 字段 | 值 |
|------|------|
| mode | inline-fallback |
| canonical_role | meta-se（预期） |
| codex_agent_name | - |
| reasoning_profile | - |
| dispatch_trigger | CP2 approved -> solution-design |
| tool_name | - |
| fallback_reason | 当前仓库未安装 meta-se 提示词（`.agents/agents/`、`delivery/agents/` 均空）；计划 v4 §4 已是经 4 轮评审的完整 HLD 草案，由 Host Orchestrator 基于计划 v4 直接文档化产出 |
| approved_by | pending（CP3 人工门禁时用户审查 HLD 确认） |
| approved_at | pending |
| spawned_at | - |
| completed_at | 2026-07-10（HLD v1.1 产出） |

> inline-fallback 说明：本 HLD 基于 plan_ref v4（已含完整 §4 架构设计 + §3.2 三层映射表 + 4 轮评审修正）由 Host Orchestrator 主进程产出，非 meta-se 独立完成。CP3 人工门禁时用户审查 HLD 质量；如不满意可 reject 要求重做。

## 19. 理论依据

| 枚举型框架 | 来源方法论 |
|-----------|-----------|
| 五维度影响分析 | 领域经验 + 可扩展（需求/设计/Story/安全/交付） |
| Scenario Gray Areas | Journey Mapping（用户旅程灰区识别） |
| Architecture Gray Areas advisor table | ADR + Option analysis（领域经验） |
| op 契约（side_effect/rollback） | ptm-atomic 仓库 `list` 元数据（实证，非方法论） |
| 验证策略（static + runtime） | ISTQB + ISO 25010（mixed validation_mode） |

## 20. 参考

- 实施计划：`/home/hyde/.claude/plans/ptm-te-agent-impl.md`（v4）
- CR-024：`process/changes/CR-024-ptm-te-agent.md`
- CP2 Decision Brief：`process/checkpoints/CP2-REQUIREMENTS-BASELINE-CR-024.md`
- 外部真相源：`/home/hyde/projects/ptm-atomic`（CLI + run_policy_route.py + op yaml + adapter + package）
- 参考项目：`/home/hyde/projects/manaul`（device-management 元数据 + collect_sysinfo.py）
- 关联 CR：CR-016（atomic-ops 消费契约）、CR-019（PC case_steps atomic_op 契约）

---
change_id: CR-024-ptm-te-agent
workflow_id: WF-PTM-TEAM-20260520-001
story_id: STORY-024-01
story_slug: ptm-te-agent-skeleton
lld_policy: full-lld
version: "1.0"
tier: S
status: draft
created_by: meta-dev
created_at: "2026-07-10T18:00:00+08:00"
hld_ref: process/HLD-CR-024.md
hld_version: "1.1"
plan_ref: /home/hyde/.claude/plans/ptm-te-agent-impl.md
feature_design_refs: [ptm-te-agent]
shared_fragments: []
open_items: 0
---

# LLD: STORY-024-01 - ptm-te agent 骨架与编排流程

> 本文档是 STORY-024-01 的低层设计（Low-Level Design），`lld_policy=full-lld`，需满足 14 个可见章节。真相源为 `process/HLD-CR-024.md`（v1.1）。本 Story 文件影响范围仅 `agents/ptm-te.md`（S1 唯一文件），将 50 行 `planned` 占位重写为 `active` 编排器。

---

## 0. 工程依据

本 LLD 的工程依据为 `process/HLD-CR-024.md`（v1.1）。架构决策与映射表真相源锁定于 HLD §4（op_id/args 三层映射）、§5（skill 边界）、§6（连接模型）、§7（凭据管理）、§8（login-once-reuse-session）、§9（inverse_op 回滚策略）。外部真相源：`ptm-atomic list`（rollback/side_effect 实测，2026-07-10）、`run_policy_route.py`（op_id->子命令硬编码）、op yaml `inputs.params`（参数名）、`ptm-atomic run --help`（flag）。CP2 决策（DQ-01 dry-run 默认门 / DQ-02 hg3250-51 / DQ-03 v1 不注入 rule block / DQ-04 手写最小 PC）已批准，CP3 HLD v1.1 + 5 ADR 已 approved。

## 1. Goal

将 `agents/ptm-te.md` 从 50 行 `planned` 占位（frontmatter `status: planned`）重写为 `active` 编排器（`status: active`，`color: green`），写入完整编排流程 [1]-[8]、PC 消费契约、op_id/args 三层映射声明、login-once-reuse-session 机制、inverse_op 回滚清理策略、运行时工作目录结构和执行门控。

重写后的 agent md 必须满足以下量化目标（对应 HLD v1.1 §1.2 SC-1、SC-4、SC-5、SC-6）：

| # | 目标 | 度量值 | HLD 引用 |
|---|------|--------|---------|
| G-1 | agent 升格 active | `status=active`，`color=green` | §1.2 SC-1 |
| G-2 | 编排流程完整 | 编排步骤 ≥ 8 步（[1] 用例解析 -> [8] 快照 after + 回写） | §2.1 |
| G-3 | 三层映射声明覆盖 | 声明 8 个 op_id -> 子命令映射 + 7 个 op 的 args -> flag 映射 | §4.3、§4.4 |
| G-4 | PC 消费契约写入 | `step_name` / `op_id` / `args` / `expected_result` 四字段契约 + 输入路径 `cases/upload/` | §3.1、§2.2 |
| G-5 | login-once-reuse 写入 | `auth login` 一次 + `--session-file` 复用 + `STATE_INVALID` 自动重连（最多 1 次重试） | §8 |
| G-6 | inverse_op 回滚写入 | 4 种 rollback 类型策略表（inverse_op / restore_snapshot / irreversible / 空） | §9 |
| G-7 | 执行门控写入 | 环境就绪 / 关键判定 / 异常记录三道门控，不复用 checkpoint-manager | §5.3 |
| G-8 | 运行时目录写入 | `cases/upload/` 入口 + `runs/<run-id>/` 6 个产物文件/目录 | §2.2 |

---

## 2. Requirements（Functional / Non-Functional）

### 2.1 Functional Requirements

| ID | 需求 | 验收条件 | HLD 引用 |
|----|------|---------|---------|
| FR-01 | agent frontmatter 升格 | `name: ptm-te`、`status: active`、`color: green`、`tools: [Bash, Read, Write, Edit, Grep, Glob, Skill]`；`skills` 列表含 3 个 skill（`device-management`、`device-connection`、`policy-route-execution`） | §1.2 SC-1 |
| FR-02 | 编排流程 [1] 用例解析 | 声明读取 `cases/upload/<特性名>特性测试用例.md`，提取结构化 `case_steps` + `expected_result`，写入 `runs/<run-id>/parse-result.json` | §2.1 [1]、§2.2 |
| FR-03 | 编排流程 [2] 设备准备 | 声明调用 `device-management` 加载 `devices.yaml` + 型号映射查表，调用 `device-connection` SSH 连接探测（失败回退 Telnet）+ 系统快照 before 写入 `runs/<run-id>/snapshot-before/` | §2.1 [2] |
| FR-04 | 编排流程 [3] login 一次 | 声明调用 `ptm-atomic run --base-url <url> auth login --username admin --password-env FW_WEB_PASSWORD --session-file <path>`，持久化 session，后续 op 复用 | §2.1 [3]、§8.1 |
| FR-05 | 编排流程 [4] 逐条 op 执行 | 声明 op_mapper 双层映射（op_id -> 子命令 + args -> flag），干跑（`--dry-run`）-> 执行（`--execute`，需 DQ-01 授权）-> verify | §2.1 [4]、§4 |
| FR-06 | 编排流程 [5] 结果判定 | 声明 envelope 契约（`op_id` / `step_name` / `status` / `data` / `error_type` / `diag_snapshot_ref`），Check 点 vs `expected_result` 比对 | §2.1 [5]、§3.3 |
| FR-07 | 编排流程 [6] 执行日志 | 声明逐条 op 写入 `runs/<run-id>/exec-log.jsonl`（JSONL：step_name / op_id / status / error_type / API 状态码） | §2.1 [6]、§2.2 |
| FR-08 | 编排流程 [7] 用例清理 | 声明 inverse_op 回滚策略：config -> delete（inverse_op）；update -> restore_snapshot；delete 作为清理动作不触发回滚；irreversible（reset-hitcount）不回滚；priority 无元数据由用例设计决定；verify 只读不回滚 | §2.1 [7]、§9 |
| FR-09 | 编排流程 [8] 快照 after + 回写 | 声明快照 after 写入 `runs/<run-id>/snapshot-after/`，结果回写 `runs/<run-id>/result.json` + `runs/<run-id>/report.md` | §2.1 [8]、§2.2 |
| FR-10 | PC 消费契约写入 | 声明消费 `case_steps[].step_name`、`case_steps[].atomic_op.op_id`、`case_steps[].atomic_op.args`、`case_steps[].expected_result`（CR-019 定义），输入路径 `cases/upload/<特性名>特性测试用例.md` | §3.1、§3.2 |
| FR-11 | 三层映射声明 | 在 agent md 中声明 op_id -> CLI 子命令映射表（8 个 op_id）+ args -> CLI flag 映射说明（7 个 op），并声明 op_mapper.py 承载翻译（S3 实现） | §4.2、§4.3、§4.4 |
| FR-12 | login-once-reuse 声明 | 声明 `auth login` 一次持久化 `--session-file`，后续 op 复用，遇 `STATE_INVALID` 自动重连（最多 1 次重试） | §8.1 |
| FR-13 | dry-run 默认门声明 | 声明默认 `--dry-run`，`--execute` 需单次授权（CP2 DQ-01，ADR-04） | §3.3、§11 ADR-04 |
| FR-14 | 执行门控声明 | 声明三道门控（环境就绪 / 关键判定 / 异常记录），明确不复用 checkpoint-manager（ptm-tde 专属 GATE-1~5） | §5.2、§5.3 |
| FR-15 | envelope 契约声明 | 声明 envelope 六字段结构 + `error_type` 枚举值（`NONE` / `STATE_INVALID` / `OP_NOT_FOUND` / `EXEC_FAILED` / `AUTH_FAILED` / `ENV_NOT_READY`） | §3.3 |
| FR-16 | 降级策略声明 | 声明 op_id 未识别 -> 阻塞提示工具缺失；设备不可达 -> 降级 dry-run-only；session 失效 -> 自动重连 | §3.1 |

### 2.2 Non-Functional Requirements

| ID | 需求 | 验收条件 |
|----|------|---------|
| NFR-01 | 确定性语言 | 全文使用确定性动词（创建 / 修改 / 删除 / 声明 / 调用 / 写入）+ 量化条件，禁止"尽可能""不少于""视情况"等模糊表述 |
| NFR-02 | HLD 一致性 | 所有编排流程步骤、契约字段、映射表、回滚策略与 HLD v1.1 §2-§9 一致；引用 HLD 章节号 |
| NFR-03 | 章节完整性 | agent md 包含编排流程、PC 消费契约、三层映射声明、login-once-reuse、inverse_op 回滚、运行时目录、执行门控、envelope 契约、降级策略共 9 个核心章节 |
| NFR-04 | 可读性 | 编排流程使用 ASCII 流程图或编号列表；映射表使用 Markdown 表格；代码块标注语言 |
| NFR-05 | 单文件完整 | 所有内容写入 `agents/ptm-te.md` 单文件，不引用外部未创建文件（S2/S3 skill 文件由后续 Story 创建，agent md 只声明调用契约） |

---

## 3. 模块拆分与职责

`agents/ptm-te.md` 重写后按以下模块组织，每个模块对应 agent md 的一个章节。模块划分与 HLD v1.1 §2-§9 对齐。

| 模块 | agent md 章节 | 职责 | HLD 引用 |
|------|-------------|------|---------|
| M1 - 角色定位 | `## 角色定位` | 声明 ptm-te 是 ptm-tde 下游执行器，编排器模式，消费 PC 执行原子操作 | §2 |
| M2 - 编排流程 | `## 编排流程` | 声明 [1]-[8] 八步编排流程，含 ASCII 流程图和逐步说明 | §2.1 |
| M3 - PC 消费契约 | `## PC 消费契约` | 声明输入路径 `cases/upload/`、消费 `case_steps` 四字段、envelope 契约、降级策略 | §3 |
| M4 - 三层映射声明 | `## op_id/args 三层映射声明` | 声明 op_id -> 子命令映射表（8 个）+ args -> flag 映射说明（7 个 op），声明 op_mapper.py 承载翻译 | §4 |
| M5 - login-once-reuse | `## login-once-reuse-session` | 声明 auth login 一次 + session 复用 + STATE_INVALID 重连 | §8 |
| M6 - inverse_op 回滚 | `## inverse_op 回滚与清理` | 声明 4 种 rollback 类型策略表 + irreversible 豁免说明 | §9 |
| M7 - 运行时目录 | `## 运行时工作目录` | 声明 `cases/upload/` 入口 + `runs/<run-id>/` 6 个产物结构 | §2.2 |
| M8 - 执行门控 | `## 执行门控` | 声明环境就绪 / 关键判定 / 异常记录三道门控，不复用 checkpoint-manager | §5.3 |
| M9 - 凭据管理 | `## 凭据管理` | 声明 devices.yaml `${ENV_VAR}` 占位 + `--password-env` 传 Web 密码 | §7 |
| M10 - 关联 Skill | `## 关联 Skill` | 声明 3 个 skill 的职责边界和调用时机 | §5.1 |

### 模块间依赖

```
M1（角色定位）
  └─► M2（编排流程）── 声明 8 步流程，引用 M3/M4/M5/M6/M7/M8
        ├─► M3（PC 消费契约）── [1] 用例解析输入
        ├─► M4（三层映射声明）── [4] 逐条 op 执行映射
        ├─► M5（login-once-reuse）── [3] login 一次
        ├─► M6（inverse_op 回滚）── [7] 用例清理
        ├─► M7（运行时目录）── [1][6][8] 产物写入
        ├─► M8（执行门控）── 贯穿 [2][5][7]
        ├─► M9（凭据管理）── [2][3] 设备准备和 login
        └─► M10（关联 Skill）── [2][4] skill 调用
```

### 与相邻对象边界

| 职责 | 归属 | 差异界定 |
|------|------|---------|
| agent md 编排流程声明 | `agents/ptm-te.md`（本 Story） | 声明编排步骤和调用契约，不实现脚本 |
| op_mapper.py 映射实现 | STORY-024-03（S3） | 承载三层映射翻译逻辑 |
| device-management 元数据 | STORY-024-02（S2） | 设备清单 + 型号映射，不含连接逻辑 |
| device-connection 连接 | STORY-024-02（S2） | SSH/Telnet + 快照采集 |
| checkpoint-manager | ptm-tde 专属 | **不复用**（HLD §5.3） |

---

## 4. 代码结构与文件影响范围

### 4.1 文件影响范围

| 文件 | 变更类型 | 变更范围 | Story |
|------|---------|---------|-------|
| `agents/ptm-te.md` | 全量重写 | 覆盖现有 50 行 `planned` 占位，重写为 `active` 编排器（预计 350-450 行） | STORY-024-01 |

**唯一文件**：本 Story 文件影响范围仅 `agents/ptm-te.md`，不创建、不修改其他文件。

### 4.2 重写前后对比

| 维度 | 重写前（planned 占位） | 重写后（active 编排器） |
|------|---------------------|----------------------|
| frontmatter `status` | `planned` | `active` |
| frontmatter `color` | 无 | `green` |
| frontmatter `tools` | `[Bash, Read, Write, Skill]` | `[Bash, Read, Write, Edit, Grep, Glob, Skill]` |
| frontmatter `skills` | 7 个旧 skill 名（test-execution-skill 等） | 3 个 skill（device-management / device-connection / policy-route-execution） |
| 编排流程 | 7 步旧流程（领取任务 -> 结果回写禅道） | 8 步新流程（[1] 用例解析 -> [8] 快照 after + 回写） |
| PC 消费契约 | 无 | 4 字段契约 + 输入路径 `cases/upload/` |
| 三层映射 | 无 | 8 op_id + 7 op flag 映射声明 |
| login-once-reuse | 无 | auth login 一次 + session 复用 + STATE_INVALID 重连 |
| inverse_op 回滚 | 无 | 4 种 rollback 类型策略表 |
| 运行时目录 | 无 | `cases/upload/` + `runs/<run-id>/` 6 产物 |
| 执行门控 | 3 行简单表格 | 独立门控章节，不复用 checkpoint-manager |
| 行数 | 50 行 | 预计 350-450 行 |

### 4.3 agent md 章节结构（重写后）

```
agents/ptm-te.md
├── frontmatter（name / description / status / color / tools / skills）
├── # ptm-te · 测试执行工程师
├── ## 角色定位
├── ## 编排流程（[1]-[8] ASCII 流程图 + 逐步说明）
├── ## PC 消费契约
│   ├── ### 输入路径
│   ├── ### 消费字段
│   ├── ### envelope 契约
│   └── ### 降级策略
├── ## op_id/args 三层映射声明
│   ├── ### 三层命名不一致问题
│   ├── ### 第一层：op_id -> CLI 子命令（8 个）
│   └── ### 第二层：args -> CLI flag（7 个 op）
├── ## login-once-reuse-session
├── ## inverse_op 回滚与清理
├── ## 运行时工作目录
├── ## 执行门控
├── ## 凭据管理
├── ## 关联 Skill
└── ## 约束
```

### 4.4 不修改的文件

- `agents/ptm-tde.md`（ptm-tde 已交付基线，不授权项，HLD §1.3）
- `skills/` 目录下所有文件（S2/S3 Story 范围）
- `script/ptm_team/install.py`（S4 Story 范围）
- `docs/ptm-team-blueprint.md`（S4 Story 范围）

---

## 5. 数据模型与持久化设计

本 Story 是 agent md 重写，不实现脚本代码。数据模型指 agent md 中声明的运行时数据结构，供 S2/S3 skill 实现时消费。

### 5.1 envelope 契约（HLD §3.3）

每条 op 执行结果封装为 envelope，agent md 必须声明此结构：

```json
{
  "op_id": "fw_config_policy_route",
  "step_name": "配置策略路由",
  "status": "success | error",
  "data": { "/* ptm-atomic 返回 */": "" },
  "error_type": "NONE | STATE_INVALID | OP_NOT_FOUND | EXEC_FAILED | AUTH_FAILED | ENV_NOT_READY",
  "diag_snapshot_ref": "runs/<run-id>/snapshot-before/<step>.json"
}
```

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `op_id` | string | 是 | ptm-tde PC 中的 `atomic_op.op_id` |
| `step_name` | string | 是 | ptm-tde PC 中的 `step_name` |
| `status` | enum | 是 | `success` / `error` |
| `data` | object | 否 | ptm-atomic CLI 返回的 JSON |
| `error_type` | enum | 是 | 错误类型枚举（`NONE` 表示无错误） |
| `diag_snapshot_ref` | string | 否 | 诊断快照引用路径 |

### 5.2 运行时目录结构（HLD §2.2）

agent md 必须声明以下运行时目录结构（用户工作区，不入库）：

```
<workspace>/
├── devices.yaml              # 设备清单（${ENV_VAR} 占位凭据，.gitignore 忽略）
├── .env                      # 凭据环境变量（.gitignore 忽略）
├── cases/
│   └── upload/               # 用例上传目录（ptm-te 执行入口）
│       └── <特性名>特性测试用例.md
└── runs/
    └── <run-id>/
        ├── parse-result.json # 用例解析结果（机器可读）
        ├── snapshot-before/  # 设备快照 before
        ├── exec-log.jsonl    # 逐条 op 执行日志（JSONL）
        ├── snapshot-after/   # 设备快照 after
        ├── result.json       # 用例结果回写（机器可读）
        └── report.md         # 人类可读测试报告
```

### 5.3 exec-log.jsonl 行结构

每行一条 JSON，agent md 声明此结构供 S3 op_mapper 写入：

```json
{
  "step_index": 1,
  "step_name": "配置策略路由",
  "op_id": "fw_config_policy_route",
  "cli_command": "ptm-atomic run --base-url https://10.113.55.51 policy-route config --source-network OBJ_SRC_WEB --dry-run",
  "status": "success",
  "error_type": "NONE",
  "api_status_code": 200,
  "timestamp": "2026-07-10T18:30:00+08:00"
}
```

### 5.4 result.json 结构

```json
{
  "run_id": "20260710-183000-hg3250-51",
  "case_file": "cases/upload/策略路由特性测试用例.md",
  "overall_result": "PASS | FAIL",
  "steps": [
    {
      "step_index": 1,
      "step_name": "配置策略路由",
      "op_id": "fw_config_policy_route",
      "status": "success",
      "error_type": "NONE",
      "expected_result": "策略路由规则成功创建",
      "actual_result": "策略路由规则已创建",
      "check_passed": true,
      "diag_snapshot_ref": "runs/<run-id>/snapshot-before/step-1.json"
    }
  ],
  "cleanup_summary": {
    "rollback_executed": ["fw_delete_policy_route"],
    "irreversible_skipped": [],
    "errors": []
  }
}
```

### 5.5 op_id -> 子命令映射表（HLD §4.3）

agent md 必须声明以下 8 个 op_id 映射，供 S3 op_mapper.py 实现时消费：

| op_id | CLI 子命令 | side_effect | rollback |
|-------|-----------|-------------|----------|
| `fw_login_web_management` | `auth login` | observation | （空） |
| `fw_config_policy_route` | `policy-route config` | state_mutation | `inverse_op:fw_delete_policy_route` |
| `fw_update_policy_route` | `policy-route update` | state_mutation | `restore_snapshot` |
| `fw_delete_policy_route` | `policy-route delete` | destructive | `restore_snapshot` |
| `fw_verify_policy_route` | `policy-route verify` | observation | （空） |
| `fw_update_policy_route_priority` | `policy-route priority` | （空） | （空） |
| `fw_reset_policy_route_hitcount` | `policy-route reset-hitcount` | state_mutation | `irreversible` |
| `fw_verify_policy_route_hitcount` | `policy-route verify-hitcount` | observation | （空） |

### 5.6 args -> flag 映射表（HLD §4.4）

agent md 必须声明以下 7 个 op 的 args -> flag 映射，供 S3 op_mapper.py 实现时消费：

**config 类（config / update）**：

| ptm-tde PC `args` | op yaml `inputs.params` | CLI flag |
|---|---|---|
| `src_addr` | `source_network` | `--source-network` |
| `dst_addr` | `dst_network` | `--dst-network` |
| `next_hop` | `next_hop_ip` | `--next-hop-ip` |
| `in_interface` | `in_interface` | `--in-interface` |
| `type` | `type` | `--policy-route-type` |

update 额外带 `--id`（从 verify 查询获取目标策略路由 id）。

**delete / reset-hitcount**：`--id` + `--policy-route-type`。
**verify / verify-hitcount**：`--policy-route-type` + `--page` + `--size`。
**priority**：`--policy-route-type` + `--moveid` + `--targetid` + `--targetsite`。
**login**：`--username` + `--password-env FW_WEB_PASSWORD`（+ 可选 `--change-default-password` / `--new-password-env`）。

### 5.7 inverse_op 回滚策略表（HLD §9.1）

| rollback 类型 | 策略 | 示例 op |
|---|---|---|
| `inverse_op:<op_id>` | 执行 inverse_op 清理 | config -> `policy-route delete` |
| `restore_snapshot` | 按 before 快照恢复 | update（恢复原值）；delete 作为 config 清理动作时不触发回滚 |
| `irreversible` | **不回滚**，由用例设计接受或规避 | reset-hitcount |
| 空（observation / 无元数据） | 不需回滚 | verify / verify-hitcount / login（只读）；priority（由用例设计决定） |

---

## 6. API / Interface 设计

本 Story 是 agent md 重写，"API"指 agent md 声明的消费契约和调用接口，供 S2/S3 skill 和下游消费者实现时遵循。

### 6.1 PC 消费契约（输入接口）

| 维度 | 契约 | HLD 引用 |
|------|------|---------|
| 调用方向 | ptm-te 消费 ptm-tde 产出（单向，ptm-te 不回调 ptm-tde） | §3.1 |
| 调用时机 | ptm-tde 完成 PC 交付（`ppdcs/delivery/`）后，用户上传到 `cases/upload/`，ptm-te 读取执行 | §3.1 |
| 触发方式 | 用户指令 ptm-te 执行指定 PC 文件 | §3.1 |
| 输入路径 | `cases/upload/<特性名>特性测试用例.md` | §2.2、§3.1 |
| 消费字段 | `case_steps[].step_name`、`case_steps[].atomic_op.op_id`、`case_steps[].atomic_op.args`、`case_steps[].expected_result`（CR-019 定义） | §3.1、§3.2 |
| 消费结构 | 消费**结构化 `case_steps`**（非 16 列汇总表）；16 列 PC 汇总表仅作人工 fallback | §3.2 |
| 后续衔接 | 失败 step 反馈 ptm-tae（工具缺失）或 ptm-tde（用例设计问题） | §3.1 |

### 6.2 op_mapper 调用契约（声明，S3 实现）

agent md 声明 op_mapper.py 的调用契约，S3 Story 实现：

| 维度 | 契约 |
|------|------|
| 输入 | `op_id`（string）+ `args`（dict，ptm-tde PC 字段名） |
| 输出 | CLI 子命令（string）+ flag 参数列表（list[string]） |
| 映射范围 | 8 个 op_id -> 子命令 + 7 个 op 的 args -> flag |
| 未识别处理 | `op_id` 未识别 -> 返回 `error_type=OP_NOT_FOUND`，阻塞并提示工具缺失 |
| 真相源 | `run_policy_route.py`（子命令）+ op yaml `inputs.params`（参数名）+ `ptm-atomic run ... --help`（flag） |

### 6.3 Skill 调用接口

| Skill | 调用时机 | 输入 | 输出 | HLD 引用 |
|-------|---------|------|------|---------|
| `device-management` | 编排流程 [2] | `devices.yaml` 路径 + 设备名 | 设备 IP / 型号 / 凭据占位 | §5.1 |
| `device-connection` | 编排流程 [2] | 设备 IP / 凭据 / 命令 | SSH/Telnet 执行结果 + 系统快照 | §5.1 |
| `policy-route-execution` | 编排流程 [4][7] | `op_id` + `args` + `--session-file` | envelope + CLI 输出 | §5.1 |

### 6.4 envelope 输出契约

每条 op 执行结果输出为 envelope（§5.1），写入 `exec-log.jsonl` 和 `result.json`。

### 6.5 dry-run / execute 门控接口

| 模式 | CLI flag | 授权要求 | HLD 引用 |
|------|---------|---------|---------|
| dry-run（默认） | `--dry-run` | 无需授权 | §3.3、ADR-04 |
| execute | `--execute` | 需单次授权（CP2 DQ-01） | §3.3、ADR-04 |

### 6.6 session 复用接口

| 维度 | 契约 | HLD 引用 |
|------|------|---------|
| login 命令 | `ptm-atomic run --base-url <url> auth login --username admin --password-env FW_WEB_PASSWORD --session-file <path>` | §8.1 |
| session 路径 | 默认 `~/.local/state/ptm-atomic/ngfw/session.json` | §8.1 |
| 复用范围 | `config` / `verify` / `update` / `delete` 复用同一 session | §8.1 |
| 失效处理 | 遇 `STATE_INVALID` -> 自动重新 `auth login` -> 重试当前 op（最多 1 次重试） | §8.1 |
| 失败终止 | `auth login` 失败 -> 终止用例，`error_type=AUTH_FAILED` | §8.2 |

---

## 7. 核心处理流程

### 7.1 编排流程总览（HLD §2.1）

agent md 必须声明以下 8 步编排流程：

```
用户上传 PC 到 cases/upload/<特性名>特性测试用例.md
  ↓
[1] 用例解析：读取 cases/upload/，提取 case_steps + expected_result
     └─ 写入 runs/<run-id>/parse-result.json
  ↓
[2] 设备准备：device-management 加载 devices.yaml + 型号映射查表
     └─ device-connection SSH 连接探测（失败回退 Telnet）+ 系统快照 before
        └─ 写入 runs/<run-id>/snapshot-before/
  ↓
[3] login 一次：ptm-atomic run --base-url <url> auth login --username admin --password-env FW_WEB_PASSWORD
     └─ 持久化 --session-file，后续复用
  ↓
[4] 逐条原子操作执行：
     op_mapper（op_id -> 子命令 + args -> flag）
       └─ 干跑（--dry-run）-> 执行（--execute，需 DQ-01 授权）-> verify
  ↓
[5] 结果判定：envelope（status=success / error_type=NONE + Check 点 vs expected_result）
  ↓
[6] 执行日志：写入 runs/<run-id>/exec-log.jsonl（JSONL：step_name / op_id / status / error_type / API 状态码）
  ↓
[7] 用例清理：inverse_op 回滚
     ├─ config 的 inverse_op = policy-route delete
     ├─ restore_snapshot 按快照恢复
     └─ irreversible 类（reset-hitcount）不回滚，由用例设计承担
  ↓
[8] 快照 after + 结果回写
     ├─ 写入 runs/<run-id>/snapshot-after/
     ├─ 写入 runs/<run-id>/result.json（机器可读）
     └─ 写入 runs/<run-id>/report.md（人类可读）
```

### 7.2 逐步流程详细说明

#### [1] 用例解析

- **输入**：`cases/upload/<特性名>特性测试用例.md`
- **动作**：读取 PC 文件，提取 `case_steps[].step_name`、`case_steps[].atomic_op.op_id`、`case_steps[].atomic_op.args`、`case_steps[].expected_result`
- **输出**：`runs/<run-id>/parse-result.json`（解析结果，机器可读）
- **前置校验**：PC 文件存在且含 `case_steps` 结构；缺 `case_steps` / `step_name` / `atomic_op.op_id` 时终止，记录 `error_type=PARSE_FAILED`
- **异常路径**：PC 文件不存在 -> 终止，提示用户上传到 `cases/upload/`

#### [2] 设备准备

- **动作**：调用 `device-management` skill 加载 `devices.yaml`，按设备名查表获取 IP / 型号 / 凭据占位；调用 `device-connection` skill 执行 SSH 连接探测（失败回退 Telnet），采集系统快照 before
- **输出**：`runs/<run-id>/snapshot-before/<step>.json`
- **前置校验**：`devices.yaml` 存在且凭据为 `${ENV_VAR}` 占位（非明文）；环境变量 `FW_WEB_PASSWORD` 等已设置；设备 IP 可达（ping 或 TCP 443 探测）
- **异常路径**：校验失败 -> 终止该设备执行，记录 `error_type=ENV_NOT_READY`；SSH + Telnet 均失败 -> 标记设备不可达，降级 dry-run-only

#### [3] login 一次

- **动作**：调用 `ptm-atomic run --base-url <url> auth login --username admin --password-env FW_WEB_PASSWORD --session-file <path>`
- **输出**：session 写入 `--session-file`（默认 `~/.local/state/ptm-atomic/ngfw/session.json`）
- **前置校验**：`auth login` 返回 `status=success`；`session.json` 写入成功
- **异常路径**：`auth login` 失败 -> 终止用例，`error_type=AUTH_FAILED`

#### [4] 逐条原子操作执行

- **动作**：对每条 `case_step`，调用 op_mapper 将 `op_id` + `args` 翻译为 CLI 子命令 + flag 参数，执行 `ptm-atomic run --base-url <url> --session-file <path> <family> <action> [flags]`
- **执行模式**：默认 `--dry-run`；`--execute` 需单次授权（DQ-01，ADR-04）
- **输出**：每条 op 的 envelope
- **session 失效处理**：遇 `STATE_INVALID` -> 自动重新 `auth login` -> 重试当前 op（最多 1 次重试）
- **异常路径**：`op_id` 未识别 -> 阻塞，`error_type=OP_NOT_FOUND`，提示工具缺失反馈 ptm-tae

#### [5] 结果判定

- **动作**：对每条 op 的 envelope，比对 `expected_result` 与实际结果，判定 `check_passed=true/false`
- **输出**：envelope 含 `status` / `error_type` / `check_passed`
- **判定规则**：`status=success` + `error_type=NONE` + Check 点匹配 -> `PASS`；否则 -> `FAIL`

#### [6] 执行日志

- **动作**：将每条 op 的执行记录写入 `runs/<run-id>/exec-log.jsonl`（JSONL 格式）
- **字段**：`step_index` / `step_name` / `op_id` / `cli_command` / `status` / `error_type` / `api_status_code` / `timestamp`

#### [7] 用例清理

- **动作**：按 op 的 `rollback` 字段执行清理（§5.7 策略表）
- **清理逻辑**：
  - `inverse_op:fw_delete_policy_route` -> 执行 `policy-route delete` 清理 config 创建的策略路由
  - `restore_snapshot` -> 按 `snapshot-before` 恢复原值（update）；delete 作为 config 清理动作时不触发回滚
  - `irreversible` -> 不回滚（reset-hitcount），由用例设计接受
  - 空（observation / 无元数据）-> 不需回滚（verify / verify-hitcount / login / priority）
- **输出**：`result.json` 的 `cleanup_summary` 字段记录回滚执行清单和错误

#### [8] 快照 after + 结果回写

- **动作**：采集系统快照 after，写入 `runs/<run-id>/snapshot-after/`；回写 `runs/<run-id>/result.json`（机器可读）+ `runs/<run-id>/report.md`（人类可读）
- **report.md 内容**：逐步骤状态 + envelope 摘要 + 快照 diff 摘要 + 清理记录

### 7.3 异常路径汇总

| 异常场景 | 触发条件 | 处理动作 | error_type |
|---------|---------|---------|-----------|
| PC 文件不存在 | `cases/upload/` 下无目标文件 | 终止，提示用户上传 | `PARSE_FAILED` |
| PC 缺 case_steps | PC 文件无 `case_steps` 结构 | 终止，提示用例格式问题 | `PARSE_FAILED` |
| devices.yaml 缺失 | 设备清单文件不存在 | 终止，提示创建 devices.yaml | `ENV_NOT_READY` |
| 凭据明文 | devices.yaml 含明文密码 | 终止，提示改用 `${ENV_VAR}` | `ENV_NOT_READY` |
| 环境变量未设置 | `FW_WEB_PASSWORD` 等未设置 | 终止，提示设置环境变量 | `ENV_NOT_READY` |
| 设备不可达 | SSH + Telnet 均失败 | 降级 dry-run-only | `ENV_NOT_READY` |
| auth login 失败 | login 返回非 success | 终止用例 | `AUTH_FAILED` |
| session 失效 | op 执行返回 `STATE_INVALID` | 自动重连（最多 1 次重试） | `STATE_INVALID` |
| op_id 未识别 | op_mapper 无映射 | 阻塞，提示工具缺失 | `OP_NOT_FOUND` |
| op 执行失败 | CLI 返回错误 | 记录错误，继续下一条或终止 | `EXEC_FAILED` |

---

## 8. 技术设计细节

> 本节阐述技术细节（函数级设计、数据结构、算法实现）。

### 8.1 三层映射声明设计（HLD §4）

agent md 必须声明三层映射问题并锁定 op_mapper 承载翻译的决策（ADR-01）：

**三层命名不一致问题**（HLD §4.1）：

| 层 | 来源 | 示例 |
|----|------|------|
| 第 1 层（ptm-tde PC args） | `case_steps[].atomic_op.args` | `src_addr` |
| 第 2 层（op yaml params） | `atoms/fw/fw_*.yaml` 的 `inputs.params` | `source_network` |
| 第 3 层（CLI flag） | `ptm-atomic run ... policy-route config --*` | `--source-network` |

**锁定决策**（ADR-01）：ptm-te 的 `op_mapper.py` 承载三层映射翻译，ptm-tde args 命名不变（保持已交付基线）。理由：不破坏 ptm-tde 已交付基线（CR-010~017 closed）；映射 centralize 便于维护。

agent md 声明的映射表见 §5.5（op_id -> 子命令，8 个）和 §5.6（args -> flag，7 个 op）。agent md 只声明契约，不实现翻译逻辑（S3 op_mapper.py 实现）。

### 8.2 login-once-reuse-session 设计（HLD §8）

agent md 必须声明以下机制：

1. 用例执行开始时，`auth login` 一次，持久化 `--session-file`
2. 后续 `config` / `verify` / `update` / `delete` 复用同一 session
3. 遇 `STATE_INVALID` 错误 -> 自动重新 `auth login` -> 重试当前 op（最多 1 次重试）
4. `auth login` 失败 -> 终止用例，`error_type=AUTH_FAILED`

**关键约束**：
- login 签名是 `--password-env` 不是 `--password`，禁止命令行明文密码（HLD §7、Gotcha #7）
- session 由 `--session-file` 自动管理，ptm-te 无需自铸 `idempotency_key` / `state_ref` / `session_ref`（HLD §8、Gotcha #6）

### 8.3 inverse_op 回滚设计（HLD §9）

agent md 必须声明 4 种 rollback 类型策略（§5.7 表），并注明以下关键约束：

- `fw_delete_policy_route` 实测 rollback=`restore_snapshot`（destructive），不是只读；但 delete 作为 config 的清理动作时不触发回滚（它本身就是清理）（HLD Gotcha #13）
- `fw_reset_policy_route_hitcount` 是 `irreversible`（命中计数清零不可恢复），不强行回滚（HLD Gotcha #5、§9.2）
- `fw_update_policy_route_priority` 无 rollback 元数据，由用例设计决定是否恢复原优先级（HLD §4.3、Gotcha #13）
- rollback 真相源是 `ptm-atomic list`（2026-07-10 实测），不要凭 op 名字推断（HLD Gotcha #13）

### 8.4 dry-run 默认门设计（HLD §3.3、ADR-04）

agent md 必须声明：
- 默认 `--dry-run`，`--execute` 需单次授权（CP2 DQ-01）
- `--execute` 作为独立 `runtime_authorization` 决策项单次确认

### 8.5 执行门控设计（HLD §5.3）

agent md 必须声明三道门控，不复用 checkpoint-manager：

| 门控 | 触发时机 | 检查项 | 失败行为 |
|------|---------|--------|---------|
| 环境就绪 | 编排流程 [2] 前 | devices.yaml 存在 + 凭据占位 + 环境变量设置 + 设备可达 | 终止，`error_type=ENV_NOT_READY` |
| 关键判定 | 编排流程 [5] 每条 op 后 | envelope `status=success` + `error_type=NONE` + Check 点匹配 | 记录 FAIL，继续或终止 |
| 异常记录 | 贯穿全流程 | 异常场景记录上下文（op_id / error_type / diag_snapshot_ref） | 写入 `exec-log.jsonl` + `result.json` |

**不复用 checkpoint-manager 的理由**（HLD §5.3）：checkpoint-manager 是 ptm-tde 三阶段框架专属门控（GATE-1~GATE-5），真相源 `docs/ptm-tde/gate-spec.md`，非通用。ptm-te 的执行门控独立写进 `agents/ptm-te.md`。

### 8.6 运行时目录设计（HLD §2.2）

agent md 必须声明 `cases/upload/` 和 `runs/<run-id>/` 的目录结构（§5.2），并注明：
- `cases/upload/` 是 ptm-te 执行入口，与 ptm-tde 产出目录 `ppdcs/delivery/` 解耦
- 用户手写最小 PC（DQ-04）或复制 ptm-tde 产出 PC（T-01）到 `cases/upload/`
- `runs/<run-id>/` 产物含机器可读（`result.json` / `exec-log.jsonl`）+ 人类可读（`report.md`）

### 8.7 CLI 命令真相约束（HLD §16 Gotchas）

agent md 必须在约束或 Gotchas 章节注明以下 CLI 真相：
- 命令名是 `ptm-atomic` 不是 `atomic-ops`（Gotcha #1）
- 扁平格式 `ptm-atomic run <op_id>` 硬报错，必须用嵌套子命令 `ptm-atomic run --base-url <url> <family> <action>`（Gotcha #2）
- rollback 字段名是 `rollback` 不是 `rollback_strategy`（Gotcha #4）
- update 需要 `--id`（从 verify 查询获取），不能直接按内容更新（Gotcha #10）
- 入接口必须路由模式，`ePolicyRouteInIfModeError` 表示 in_interface 非路由模式（Gotcha #11）

---

## 9. 安全与性能设计

### 9.1 凭据安全（HLD §7）

agent md 必须声明以下凭据管理策略：

| 项 | 策略 | HLD 引用 |
|----|------|---------|
| devices.yaml | 不入库明文凭据，用 `${ENV_VAR}` 占位；用户工作区 `.gitignore` 忽略 | §7 |
| 环境变量 | `.env.example` 提供变量清单；用户复制为 `.env`（`.gitignore` 忽略） | §7 |
| Web 密码 | 经 `--password-env FW_WEB_PASSWORD` 传入，**禁止命令行明文密码** | §7、Gotcha #7 |
| SSH/Telnet 密码 | 用 `${ENV_VAR}` 占位，运行时从环境变量读取 | §7 |
| 模板 | `skills/device-management/templates/devices.yaml.example` 提供模板（S2 实现） | §7 |

### 9.2 dry-run 安全门（HLD §3.3、ADR-04）

- 默认 `--dry-run`，避免意外修改真实设备
- `--execute` 需单次授权，降低真实设备写操作风险
- dry-run 已验证所有参数路由和 session 有效性

### 9.3 session 安全

- session 由 `--session-file` 自动管理，ptm-te 不自造 session 引用
- `STATE_INVALID` 自动重连限制最多 1 次重试，避免无限重试循环

### 9.4 设备不可达降级

- SSH + Telnet 均失败 -> 标记设备不可达，降级 dry-run-only
- 降级后 runtime 验证留 follow-up（T-01 candidate）

### 9.5 性能考量

- login-once-reuse-session 避免每条 op 重复登录，减少网络开销
- 逐条 op 执行，不并行（同用例内串行，避免设备并发冲突）
- 快照采集 before/after 两点，不做高频采集

---

## 10. 测试设计

本 Story 是 agent md 重写，测试以 static 检查为主（HLD §14 验证策略 static 部分）。

### 10.1 static 测试项

| TC ID | 测试项 | 验收条件 | HLD 引用 |
|-------|--------|---------|---------|
| TC-01 | frontmatter 校验 | `status=active`、`color=green`、`tools` 含 7 个工具、`skills` 含 3 个 skill | §1.2 SC-1 |
| TC-02 | 编排流程完整性 | 编排流程章节含 [1]-[8] 八步，每步有输入/动作/输出说明 | §2.1 |
| TC-03 | PC 消费契约校验 | 声明 4 字段（`step_name` / `op_id` / `args` / `expected_result`）+ 输入路径 `cases/upload/` | §3.1、§3.2 |
| TC-04 | op_id 映射覆盖校验 | 声明 8 个 op_id -> 子命令映射，与 HLD §4.3 表一致 | §4.3 |
| TC-05 | args -> flag 映射覆盖校验 | 声明 7 个 op 的 args -> flag 映射，与 HLD §4.4 一致 | §4.4 |
| TC-06 | login-once-reuse 校验 | 声明 auth login 一次 + session 复用 + STATE_INVALID 重连（最多 1 次） | §8 |
| TC-07 | inverse_op 回滚校验 | 声明 4 种 rollback 类型策略，与 HLD §9.1 一致 | §9.1 |
| TC-08 | 执行门控校验 | 声明三道门控（环境就绪 / 关键判定 / 异常记录），明确不复用 checkpoint-manager | §5.3 |
| TC-09 | 运行时目录校验 | 声明 `cases/upload/` + `runs/<run-id>/` 6 个产物，与 HLD §2.2 一致 | §2.2 |
| TC-10 | envelope 契约校验 | 声明 6 字段 + `error_type` 枚举值 | §3.3 |
| TC-11 | dry-run 默认门校验 | 声明默认 `--dry-run`，`--execute` 需授权 | §3.3、ADR-04 |
| TC-12 | 凭据安全校验 | 声明 `${ENV_VAR}` 占位 + `--password-env` + 禁止明文密码 | §7 |
| TC-13 | CLI 真相约束校验 | 声明 `ptm-atomic` 命令名 + 嵌套子命令 + rollback 字段名 | §16 Gotchas |
| TC-14 | 降级策略校验 | 声明 op_id 未识别 / 设备不可达 / session 失效三种降级 | §3.1 |
| TC-15 | HLD 一致性校验 | 所有声明与 HLD v1.1 §2-§9 一致，无矛盾 | 全文 |

### 10.2 runtime 测试项（CP7，非本 Story 范围）

runtime 测试需 DQ-01 授权 + DQ-02 设备 hg3250-51，属 CP7 验证范围，不在本 Story 范围内。agent md 声明的契约由 S2/S3 skill 实现后验证。

### 10.3 dev_gate 计算

| 条件 | 状态 |
|------|------|
| 本 Story LLD 确认（CP5） | 待确认 |
| 依赖门控 | 无依赖（W1 首个 Story） |
| 文件所有权门控 | `agents/ptm-te.md` 唯一文件，无冲突 |
| dev_gate | CP5 通过后可进入实现 |

---

## 11. 实施步骤

### TASK-024-01-01：重写 frontmatter

- **文件**：`agents/ptm-te.md`
- **动作**：将 frontmatter 从 `status: planned` 重写为：
  ```yaml
  ---
  name: ptm-te
  description: 测试执行工程师。消费 ptm-tde 产出的物理用例，在真实设备上执行原子操作，编排用例解析、设备准备、login、逐条 op 执行、结果判定、用例清理和结果回写。
  status: active
  tools: [Bash, Read, Write, Edit, Grep, Glob, Skill]
  color: green
  skills:
    - device-management
    - device-connection
    - policy-route-execution
  ---
  ```
- **验收**：`status=active`、`color=green`、`tools` 含 7 个工具、`skills` 含 3 个 skill

### TASK-024-01-02：写入角色定位章节

- **文件**：`agents/ptm-te.md`
- **动作**：创建 `## 角色定位` 章节，声明 ptm-te 是 ptm-tde 下游执行器，编排器模式，消费 PC 执行原子操作
- **内容要点**：ptm-te 是测试执行工程师；采用与 ptm-tde 一致的编排器模式；消费 ptm-tde 产出的 PC（`cases/upload/`），在真实设备上执行原子操作；关联 3 个 skill（device-management / device-connection / policy-route-execution）
- **验收**：声明下游执行器定位 + 编排器模式 + 3 个 skill 关联

### TASK-024-01-03：写入编排流程章节

- **文件**：`agents/ptm-te.md`
- **动作**：创建 `## 编排流程` 章节，写入 [1]-[8] 八步 ASCII 流程图 + 逐步说明
- **内容要点**：§7.1 流程图 + §7.2 逐步说明（每步含输入/动作/输出/前置校验/异常路径）
- **验收**：编排步骤 ≥ 8 步，每步有输入/动作/输出说明

### TASK-024-01-04：写入 PC 消费契约章节

- **文件**：`agents/ptm-te.md`
- **动作**：创建 `## PC 消费契约` 章节，含 4 个子章节：输入路径、消费字段、envelope 契约、降级策略
- **内容要点**：
  - 输入路径：`cases/upload/<特性名>特性测试用例.md`（HLD §2.2、§3.1）
  - 消费字段：`case_steps[].step_name` / `atomic_op.op_id` / `atomic_op.args` / `expected_result`（CR-019 定义）（HLD §3.1、§3.2）
  - envelope 契约：6 字段 + `error_type` 枚举（HLD §3.3，§5.1）
  - 降级策略：op_id 未识别 / 设备不可达 / session 失效（HLD §3.1）
- **验收**：4 字段契约 + 输入路径 + envelope 6 字段 + 3 种降级

### TASK-024-01-05：写入三层映射声明章节

- **文件**：`agents/ptm-te.md`
- **动作**：创建 `## op_id/args 三层映射声明` 章节，含 3 个子章节：三层命名不一致问题、第一层 op_id -> 子命令（8 个）、第二层 args -> flag（7 个 op）
- **内容要点**：
  - 三层命名不一致问题（HLD §4.1，§8.1）
  - op_id -> 子命令映射表 8 个（HLD §4.3，§5.5）
  - args -> flag 映射 7 个 op（HLD §4.4，§5.6）
  - 声明 op_mapper.py 承载翻译（ADR-01），ptm-tde args 不变
- **验收**：8 op_id 映射 + 7 op flag 映射 + ADR-01 声明

### TASK-024-01-06：写入 login-once-reuse-session 章节

- **文件**：`agents/ptm-te.md`
- **动作**：创建 `## login-once-reuse-session` 章节
- **内容要点**：auth login 一次 + `--session-file` 持久化 + 后续 op 复用 + `STATE_INVALID` 自动重连（最多 1 次重试）+ `AUTH_FAILED` 终止（HLD §8，§8.2）
- **验收**：声明 login 一次 + session 复用 + STATE_INVALID 重连 + AUTH_FAILED 终止

### TASK-024-01-07：写入 inverse_op 回滚与清理章节

- **文件**：`agents/ptm-te.md`
- **动作**：创建 `## inverse_op 回滚与清理` 章节
- **内容要点**：4 种 rollback 类型策略表（§5.7）+ irreversible 豁免说明（HLD §9.2）+ delete 作为清理动作不触发回滚 + priority 无元数据由用例设计决定
- **验收**：4 种 rollback 类型 + irreversible 豁免 + delete 清理动作说明

### TASK-024-01-08：写入运行时工作目录章节

- **文件**：`agents/ptm-te.md`
- **动作**：创建 `## 运行时工作目录` 章节
- **内容要点**：`cases/upload/` 入口 + `runs/<run-id>/` 6 个产物结构（§5.2）+ 与 `ppdcs/delivery/` 解耦说明
- **验收**：cases/upload/ + runs/<run-id>/ 6 产物 + 解耦说明

### TASK-024-01-09：写入执行门控章节

- **文件**：`agents/ptm-te.md`
- **动作**：创建 `## 执行门控` 章节
- **内容要点**：三道门控（环境就绪 / 关键判定 / 异常记录）+ 不复用 checkpoint-manager 说明（HLD §5.3，§8.5）
- **验收**：三道门控 + 不复用 checkpoint-manager

### TASK-024-01-10：写入凭据管理章节

- **文件**：`agents/ptm-te.md`
- **动作**：创建 `## 凭据管理` 章节
- **内容要点**：devices.yaml `${ENV_VAR}` 占位 + `--password-env` + 禁止明文密码 + `.env` 忽略（HLD §7，§9.1）
- **验收**：占位策略 + password-env + 禁止明文

### TASK-024-01-11：写入关联 Skill 和约束章节

- **文件**：`agents/ptm-te.md`
- **动作**：创建 `## 关联 Skill` 和 `## 约束` 章节
- **内容要点**：
  - 关联 Skill：3 个 skill 职责边界表（HLD §5.1、§5.2）
  - 约束：CLI 真相约束（`ptm-atomic` 命令名 + 嵌套子命令 + rollback 字段名 + update 需要 --id + 入接口路由模式）（HLD §16 Gotchas）
- **验收**：3 skill 边界 + 5 条 CLI 真相约束

### TASK-024-01-12：删除旧内容并校验

- **文件**：`agents/ptm-te.md`
- **动作**：删除旧 50 行占位的全部内容（职责、流程、检查点旧章节），确保旧内容零残留；运行 TC-01 ~ TC-15 static 校验
- **验收**：旧内容零残留 + 15 项 static 测试全 PASS

---

## 12. 风险、难点与预研建议

### 12.1 风险

| 风险 ID | 风险 | 等级 | 影响 | 缓解 | HLD 引用 |
|---------|------|------|------|------|---------|
| R-01 | agent md 声明与 HLD v1.1 不一致 | 高 | S2/S3 skill 实现时契约漂移 | 逐项引用 HLD 章节号；CP5 static 校验一致性（TC-15） | §14 |
| R-02 | 三层映射声明遗漏 op_id 或 flag | 高 | op_mapper.py 实现时映射不全 | 声明 8 op_id + 7 op flag，TC-04/TC-05 逐项校验 | §4.3、§4.4 |
| R-03 | inverse_op 回滚声明与 `ptm-atomic list` 实测不符 | 中 | 用例清理逻辑错误 | 以 HLD §4.3 rollback 列（`ptm-atomic list` 实测）为准，不凭 op 名推断 | §4.3、Gotcha #13 |
| R-04 | agent md 过长导致可读性下降 | 低 | 维护困难 | 按模块组织（§3），使用表格和代码块 | - |
| R-05 | ptm-tde PC args 命名与声明不符 | 中 | op_mapper 翻译失败 | HLD §4.1 锁定三层命名；CP3 已确认 ptm-tde args 不变（ADR-01） | §4.1、§4.2 |

### 12.2 难点

| 难点 | 说明 | 解决方案 |
|------|------|---------|
| 三层映射声明完整性 | 需覆盖 8 op_id + 7 op flag，不能遗漏 | 直接引用 HLD §4.3、§4.4 映射表，逐项核对 |
| inverse_op 回滚语义准确 | delete 是 restore_snapshot 但作为清理动作不触发回滚；priority 无元数据 | 引用 HLD §9.1 策略表 + Gotcha #13 说明 |
| agent md 声明与 skill 实现边界 | agent md 只声明契约，不实现脚本 | 明确"声明"vs"实现"边界，S2/S3 Story 实现 |

### 12.3 预研建议

- 无需额外预研。HLD v1.1 已锁定全部设计决策（5 ADR），`ptm-atomic list` 实测已完成，映射表已固化。

### 12.4 clarification items

本 Story 无 `blocks_lld=true` 的 clarification item。所有设计决策已在 HLD v1.1 和 CP2/CP3 确认。

---

## 13. 回滚与发布策略

### 13.1 回滚策略

| 场景 | 回滚动作 | 回滚目标 |
|------|---------|---------|
| agent md 重写后发现问题 | `git checkout HEAD -- agents/ptm-te.md` | 恢复 50 行 `planned` 占位 |
| CP5 人工审查 reject | 按 reject 意见修正后重新提交 | 修正后的 agent md |
| CP6 编码完成检查发现声明缺陷 | 修正 agent md 声明，无需回滚到占位 | 修正后的 agent md |

### 13.2 发布策略

- **发布范围**：`agents/ptm-te.md` 单文件，W1 首个 Story
- **发布条件**：CP5 通过 + CP6 编码完成（agent md 重写完成 + static 测试 PASS）
- **下游影响**：S2（device-management + device-connection）和 S3（policy-route-execution + op_mapper）依赖本 Story 声明的契约；S4（安装器集成）依赖本 Story 的 frontmatter（`status=active`、`color=green`、`skills` 列表）
- **发布顺序**：W1(S1) -> W2(S2‖S3) -> W3(S4)，本 Story 是 W1 唯一 Story，阻塞全部下游

### 13.3 灰度顺序（Review Gate）

按 CLAUDE.md Review Gate 灰度顺序，本 Story 产物（`agents/ptm-te.md`）属 agent md，灰度覆盖：
1. 先覆盖 `agents/ptm-te.md`（本 Story 产物）
2. 再覆盖 `DEVELOPMENT-PLAN.yaml`（已由 CP4 生成）
3. 最后覆盖 `docs/ptm-team-blueprint.md`（S4 Story 范围）

---

## 14. Definition of Done

### 14.1 量化验收项

| # | 验收项 | 度量值 | 验证方法 |
|---|--------|--------|---------|
| DoD-01 | agent frontmatter 升格 | `status=active`、`color=green` | 读取 frontmatter 校验 |
| DoD-02 | tools 完整 | 含 7 个工具（Bash/Read/Write/Edit/Grep/Glob/Skill） | 读取 frontmatter 校验 |
| DoD-03 | skills 列表正确 | 含 3 个 skill（device-management/device-connection/policy-route-execution） | 读取 frontmatter 校验 |
| DoD-04 | 编排流程完整 | 编排步骤 ≥ 8 步（[1]-[8]） | 检查编排流程章节 |
| DoD-05 | PC 消费契约写入 | 4 字段（step_name/op_id/args/expected_result）+ 输入路径 `cases/upload/` | 检查 PC 消费契约章节 |
| DoD-06 | 三层映射声明覆盖 | 8 个 op_id -> 子命令映射 + 7 个 op 的 args -> flag 映射 | 检查三层映射章节，逐项核对 HLD §4.3、§4.4 |
| DoD-07 | login-once-reuse 写入 | auth login 一次 + session 复用 + STATE_INVALID 重连（最多 1 次） | 检查 login-once-reuse 章节 |
| DoD-08 | inverse_op 回滚写入 | 4 种 rollback 类型策略表（inverse_op/restore_snapshot/irreversible/空） | 检查 inverse_op 章节，核对 HLD §9.1 |
| DoD-09 | 执行门控写入 | 三道门控（环境就绪/关键判定/异常记录）+ 不复用 checkpoint-manager | 检查执行门控章节 |
| DoD-10 | 运行时目录写入 | `cases/upload/` + `runs/<run-id>/` 6 个产物 | 检查运行时目录章节，核对 HLD §2.2 |
| DoD-11 | envelope 契约写入 | 6 字段 + error_type 枚举（≥ 6 个值） | 检查 PC 消费契约 envelope 子章节 |
| DoD-12 | dry-run 默认门写入 | 默认 `--dry-run`，`--execute` 需授权 | 检查编排流程或约束章节 |
| DoD-13 | 凭据管理写入 | `${ENV_VAR}` 占位 + `--password-env` + 禁止明文密码 | 检查凭据管理章节 |
| DoD-14 | CLI 真相约束写入 | ≥ 5 条（命令名/嵌套子命令/rollback 字段名/update --id/入接口路由模式） | 检查约束章节 |
| DoD-15 | 降级策略写入 | 3 种（op_id 未识别/设备不可达/session 失效） | 检查 PC 消费契约降级子章节 |
| DoD-16 | 旧内容零残留 | 旧 50 行占位内容（禅道/旧流程/旧 skill 名）零残留 | 全文搜索"禅道""test-execution-skill"等旧关键词 |
| DoD-17 | HLD 一致性 | 所有声明与 HLD v1.1 §2-§9 一致，无矛盾 | TC-15 一致性校验 |
| DoD-18 | 确定性语言 | 全文使用确定性动词 + 量化条件，无模糊表述 | 人工审查 |

### 14.2 LLD 完整性验收

- [ ] 14 个章节全部填写完成
- [ ] 文件影响范围仅 `agents/ptm-te.md`（S1 唯一文件）
- [ ] 接口契约（§6）与 HLD §3-§4 一致
- [ ] 测试项（§10）覆盖 15 项 static 检查
- [ ] 实施步骤（§11）含 12 个 TASK-ID，可直接指导编码
- [ ] 风险（§12）含 5 项风险 + 3 项难点
- [ ] DoD（§14）含 18 项量化验收项
- [ ] frontmatter 含 `change_id` / `workflow_id` / `story_id` / `lld_policy` / `version` / `hld_ref`
- [ ] `confirmed=false` 时不进入实现

---

## 人工确认区

> **CP5 - Story LLD 可实现性门**
> meta-dev 先写入 `process/checks/CP5-STORY-024-01-ptm-te-agent-skeleton-LLD-IMPLEMENTABILITY.md` 自动预检结果。
> host-orchestrator 收齐全部目标 Story 的 LLD、CP4 自动预检摘要和 CP5 自动预检后，生成并提示用户审查 `process/checkpoints/CP5-ALL-STORIES-LLD-BATCH.md`。
> 用户统一确认全部目标 Story 的 LLD 后，仍需满足当前 Wave、依赖门控与文件所有权门控方可进入实现。

**CP5 checklist 摘要**：

| # | 检查项 | 状态 | 证据 |
|---|--------|------|------|
| 1 | LLD 覆盖 AC | 待检查 | 第 2 / 10 / 14 节：FR-01~FR-16 + 18 项 DoD 全部覆盖 |
| 2 | 与 HLD / ADR 一致 | 待检查 | 第 3 / 8 / 12 节：ADR-01（op_mapper 承载三层映射）、ADR-02（login-once-reuse）、ADR-03（inverse_op 豁免）、ADR-04（dry-run 默认门）、ADR-05（skill 拆分）全部落实 |
| 3 | 文件影响范围明确 | 待检查 | 第 4 / 11 节：仅修改 `agents/ptm-te.md`，12 个 TASK-ID 全部覆盖 |
| 4 | 接口契约完整 | 待检查 | 第 6 节：PC 消费契约 + op_mapper 调用契约 + 3 个 skill 调用接口 + envelope 输出 + dry-run 门控 + session 复用 |
| 5 | 测试与 dev_gate 可计算 | 待检查 | 第 10 / 14 节：15 项 static 测试，dev_gate = CP5 通过（无依赖，W1 首个） |
| 6 | clarification queue 已收敛 | 待检查 | 第 12.4 节：无 `blocks_lld=true` 项，所有决策已在 HLD v1.1 和 CP2/CP3 确认 |

**人工确认回复**：

请直接回复以下任一整行：

```text
approve
修改: <具体修改点>
reject
```

- `approve`：LLD 设计合理，允许进入实现。
- `修改: <具体修改点>`：指出具体修改点后由 meta-dev 更新重提。
- `reject`：设计方向有根本问题，需重新设计。

**人工审查结果回填**：

- 结论：`approved | changes_requested | rejected`
- 审查人：
- 审查时间：
- 修改意见：
- 风险接受项：

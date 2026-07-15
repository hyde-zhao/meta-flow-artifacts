---
change_id: CR-024-ptm-te-agent
workflow_id: WF-PTM-TEAM-20260520-001
story_id: STORY-024-02
lld_policy: full-lld
version: 1.0
hld_ref: process/HLD-CR-024.md
created_at: "2026-07-10T00:00:00+08:00"
author: host-orchestrator（主进程，inline-fallback 基于 HLD v1.1）
status: draft
---

# LLD - STORY-024-02：device-management + device-connection skill

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|------|------|--------|---------|
| v1.0 | 2026-07-10 | host-orchestrator | 初版。基于 HLD v1.1 §5/§6/§7 产出。锁定 6 文件布局（device-management 3 + device-connection 3）；ssh_exec.py / collect_sysinfo.py 函数级设计；devices.yaml schema（${ENV_VAR} 占位）；快照 before/after 数据结构；安全凭据策略 |

---

## 0. 工程依据

本 LLD 的工程依据为 `process/HLD-CR-024.md`（v1.1）。架构决策与映射表真相源锁定于 HLD §4（op_id/args 三层映射）、§5（skill 边界）、§6（连接模型）、§7（凭据管理）、§8（login-once-reuse-session）、§9（inverse_op 回滚策略）。外部真相源：`ptm-atomic list`（rollback/side_effect 实测，2026-07-10）、`run_policy_route.py`（op_id->子命令硬编码）、op yaml `inputs.params`（参数名）、`ptm-atomic run --help`（flag）。CP2 决策（DQ-01 dry-run 默认门 / DQ-02 hg3250-51 / DQ-03 v1 不注入 rule block / DQ-04 手写最小 PC）已批准，CP3 HLD v1.1 + 5 ADR 已 approved。

## 1. Goal

### 1.1 目标

为 ptm-te（测试执行工程师）Agent 提供设备管理与设备连接两个 skill，作为编排流程 [2]（设备准备）的底层能力支撑。

- **device-management**：纯元数据 SKILL，管理 `devices.yaml` 设备清单结构 + 型号映射查表，不含任何连接逻辑
- **device-connection**：连接 SKILL，提供 SSH（paramiko，首选）/ Telnet（telnetlib，回退）双轨连接 + 系统快照采集（before/after 两点），不管理设备清单

### 1.2 量化成功标准

| # | 成功标准 | 度量值 |
|---|---------|--------|
| SC-1 | 2 个 skill 创建完成 | `skills/device-management/`（3 文件）+ `skills/device-connection/`（3 文件）= 6 文件 |
| SC-2 | SSH/Telnet 双轨 + 回退 | SSH 首选，失败回退 Telnet；Telnet 失败标记设备不可达 |
| SC-3 | 系统快照 before/after | 快照存 `runs/<run-id>/snapshot-before/` 和 `snapshot-after/`，含 CPU/内存/磁盘/进程 4 维度 |
| SC-4 | 凭据安全 | devices.yaml 凭据全部 `${ENV_VAR}` 占位，0 个明文凭据入库；`.gitignore` 忽略 devices.yaml + .env |
| SC-5 | Python 版本声明 | ssh_exec.py + collect_sysinfo.py 声明 `requires-python = ">=3.9,<3.13"` |
| SC-6 | 迁移来源适配 | manaul `collect_sysinfo.py`（236 行）连接逻辑迁移适配；manaul device-management 元数据迁移；manaul devices.yaml 明文凭据改 `${ENV_VAR}` 占位 |

### 1.3 关联 HLD 锁定项

| HLD 章节 | 锁定内容 | 本 LLD 对应 |
|---------|---------|------------|
| HLD §5.1 | device-management 只做元数据，不含连接逻辑；device-connection 只做连接+快照，不管理清单 | §3 模块拆分、§4 文件影响范围 |
| HLD §6 | SSH（paramiko 首选）失败回退 Telnet（telnetlib 声明 >=3.9,<3.13）；Web 走 ptm-atomic | §7 核心流程、§8 技术设计 |
| HLD §6.1 | 连接前置校验：devices.yaml 存在 + ${ENV_VAR} 占位 + 环境变量已设 + IP 可达 | §7 核心流程、§9 安全设计 |
| HLD §7 | devices.yaml 不入库明文，${ENV_VAR} 占位；templates/devices.yaml.example 模板；.env.example 变量清单；禁止命令行明文密码 | §5 数据模型、§9 安全设计 |
| HLD §2 | 快照 before（流程 [2]）/ after（流程 [8]），存 runs/<run-id>/snapshot-before/ 和 snapshot-after/ | §5 数据模型、§7 核心流程 |
| HLD §12 | S2 文件布局 6 文件 | §4 文件影响范围 |
| HLD Gotcha #8 | device-management 不含连接逻辑 | §3 模块拆分 |
| HLD Gotcha #12 | devices.yaml 不入库明文 | §9 安全设计 |

---

## 2. Requirements

### 2.1 Functional Requirements

| ID | 需求 | 优先级 | 来源 |
|----|------|--------|------|
| FR-01 | device-management SKILL.md 定义元数据管理流程（添加设备 / 查询型号 / 查询设备） | P0 | HLD §5 |
| FR-02 | device-management 提供型号映射查表能力，覆盖 manaul 现有全部型号（HG3250/NXP1046/160pro/160/290/TG-C236/TG-J1900 等） | P0 | manaul SKILL.md |
| FR-03 | device-management templates/devices.yaml.example 提供设备清单模板，凭据全部 `${ENV_VAR}` 占位 | P0 | HLD §7 |
| FR-04 | device-management reference/device-reference.md 提供硬件系列->TGFW 型号对照表 | P0 | manaul reference |
| FR-05 | device-connection SKILL.md 定义连接流程（前置校验 -> SSH 首选 -> Telnet 回退 -> 快照采集） | P0 | HLD §6 |
| FR-06 | ssh_exec.py 提供 SSH 远程命令执行能力，含连接 / 执行 / 关闭完整生命周期 | P0 | manaul ssh_exec.py |
| FR-07 | collect_sysinfo.py 提供系统信息采集（CPU/内存/磁盘/进程 4 维度），SSH 失败回退 Telnet | P0 | manaul collect_sysinfo.py |
| FR-08 | collect_sysinfo.py 支持指定设备名 / `--all` / `--host` 三种目标模式 | P1 | manaul collect_sysinfo.py |
| FR-09 | collect_sysinfo.py 从 devices.yaml 加载设备配置，自动解析 `${ENV_VAR}` 占位凭据 | P0 | HLD §7 |
| FR-10 | collect_sysinfo.py 快照输出到 `runs/<run-id>/snapshot-{phase}/<device>.json` | P0 | HLD §2.2 |
| FR-11 | collect_sysinfo.py 支持 `--phase before|after` 参数控制采集阶段 | P0 | manaul collect_sysinfo.py |
| FR-12 | collect_sysinfo.py 连接前置校验：devices.yaml 存在 + 环境变量已设 + IP 可达 | P0 | HLD §6.1 |

### 2.2 Non-Functional Requirements

| ID | 需求 | 度量值 |
|----|------|--------|
| NFR-01 | Python 版本兼容 | `>=3.9,<3.13`（telnetlib 在 3.13 移除） |
| NFR-02 | 依赖最小化 | 仅依赖 paramiko（SSH）、telnetlib（标准库）、yaml（PyYAML）；无额外第三方依赖 |
| NFR-03 | 凭据安全 | devices.yaml 入库时凭据字段值 100% 为 `${ENV_VAR}` 格式；运行时从环境变量读取；禁止命令行 `--password` 明文参数入库 |
| NFR-04 | 连接超时 | SSH 连接超时 15s；Telnet 连接超时 10s；单命令执行超时 15s |
| NFR-05 | 错误处理 | SSH 失败自动回退 Telnet（逐命令级别）；Telnet 失败标记 `ERROR: <exception>`，不崩溃 |
| NFR-06 | 快照可读性 | JSON 格式，`ensure_ascii=False`，`indent=2`；含 device / phase / timestamp 元数据 |
| NFR-07 | 可维护性 | 函数职责单一，每个函数 ≤ 50 行；类型注解覆盖全部公共函数 |
| NFR-08 | 脚本可执行性 | ssh_exec.py 和 collect_sysinfo.py 可独立通过 `uv run python <script>` 或 `python <script>` 执行 |

---

## 3. 模块拆分与职责

### 3.1 device-management（元数据 SKILL）

| 组件 | 职责 | 边界 |
|------|------|------|
| SKILL.md | 定义设备元数据管理流程：添加设备 / 查询型号 / 查询设备。触发词、参数提示、操作流程、模板引用 | 不含任何连接逻辑（HLD Gotcha #8） |
| reference/device-reference.md | 硬件系列 -> TGFW 型号对照表，含型号特征（CPU 平台 / 内存 / 硬盘）用于消歧 | 只读参考，不执行 |
| templates/devices.yaml.example | 设备清单模板，凭据全部 `${ENV_VAR}` 占位 | 是模板非运行时文件；用户复制到工作区根目录为 devices.yaml |

### 3.2 device-connection（连接 SKILL）

| 组件 | 职责 | 边界 |
|------|------|------|
| SKILL.md | 定义连接流程：前置校验 -> SSH 首选 -> Telnet 回退 -> 快照采集。调用说明、参数说明、错误处理 | 不管理设备清单（清单由 device-management 维护）；不执行策略路由 op（由 policy-route-execution 承载） |
| scripts/ssh_exec.py | SSH 远程命令执行封装：连接 / 执行 / 关闭 | 单条命令级别；不含回退逻辑（回退由 collect_sysinfo.py 承载） |
| scripts/collect_sysinfo.py | 系统信息采集：加载 devices.yaml -> 前置校验 -> SSH/Telnet 双轨采集 -> 快照存储 | 采集 4 维度（CPU/内存/磁盘/进程）；输出 JSON 快照 |

### 3.3 职责边界矩阵

| 职责 | device-management | device-connection | 不归属说明 |
|------|:-:|:-:|------|
| 维护 devices.yaml 结构 | 是 | 否 | device-connection 只读不写 devices.yaml |
| 型号映射查表 | 是 | 否 | device-connection 不关心型号 |
| SSH 连接 | 否 | 是 | device-management 不含连接逻辑 |
| Telnet 连接 | 否 | 是 | 同上 |
| 系统快照采集 | 否 | 是 | 快照是连接后的采集动作 |
| 快照存储路径管理 | 否 | 是 | 输出到 runs/<run-id>/snapshot-{phase}/ |
| 凭据环境变量解析 | 否 | 是 | collect_sysinfo.py 运行时解析 ${ENV_VAR} |
| 策略路由 op 执行 | 否 | 否 | 归属 policy-route-execution（STORY-024-03） |
| Web 管理 (HTTPS) | 否 | 否 | 归属 ptm-atomic CLI（非本 Story 范围） |

---

## 4. 代码结构与文件影响范围

### 4.1 文件清单（6 文件）

| # | 文件路径 | 类型 | 来源 | 行数估算 |
|---|---------|------|------|---------|
| 1 | `skills/device-management/SKILL.md` | 新建 | 迁移自 manaul，适配 ${ENV_VAR} 占位 | ~80 行 |
| 2 | `skills/device-management/reference/device-reference.md` | 新建 | 迁移自 manaul，补充型号特征 | ~100 行 |
| 3 | `skills/device-management/templates/devices.yaml.example` | 新建 | 参照 manaul devices.yaml，凭据改 ${ENV_VAR} | ~60 行 |
| 4 | `skills/device-connection/SKILL.md` | 新建 | 新编写，参照 HLD §6 连接模型 | ~90 行 |
| 5 | `skills/device-connection/scripts/ssh_exec.py` | 新建 | 迁移自 manaul ssh_exec.py（27 行），扩展类型注解 + 错误处理 | ~60 行 |
| 6 | `skills/device-connection/scripts/collect_sysinfo.py` | 新建 | 迁移自 manaul collect_sysinfo.py（236 行），适配 ${ENV_VAR} + 快照路径 + 前置校验 | ~280 行 |

### 4.2 文件所有权与并行安全

- 本 Story 6 文件全部在 `skills/device-management/` 和 `skills/device-connection/` 目录下
- 与 STORY-024-03（`skills/policy-route-execution/`）无文件所有权冲突
- 与 STORY-024-01（`agents/ptm-te.md`）无文件所有权冲突
- W2 波次可与 STORY-024-03 并行（DEVELOPMENT-PLAN-CR-024.yaml `parallel_safety.W2_conflict_check.overlap = none`）

### 4.3 不修改的文件（不授权项）

- `agents/ptm-te.md`（STORY-024-01 所有）
- `skills/policy-route-execution/**`（STORY-024-03 所有）
- `script/ptm_team/install.py`（STORY-024-04 所有）
- `skills/README.md`（STORY-024-04 所有）

---

## 5. 数据模型与持久化设计

### 5.1 devices.yaml Schema

devices.yaml 是设备清单的唯一真相源，存放在用户工作区根目录（`.gitignore` 忽略）。

```yaml
# devices.yaml - 设备清单（凭据用 ${ENV_VAR} 占位，禁止明文）
device_groups:
  - name: <设备组名>              # 必填，唯一标识，如 hg3250-51
    description: "<设备描述>"      # 可选

    firewall:                      # 防火墙设备块
      host: <管理IP>               # 必填，如 10.113.55.51
      role: firewall               # 固定值
      device_type: <TGFW型号>      # 必填，如 DAS-TGFW-A1300-HU
      web:                         # Web 管理连接
        host: <Web_IP>            # 必填
        port: 443                  # 可选，默认 443
        user: admin                # 必填
        password: ${FW_WEB_PASSWORD}  # 必填，${ENV_VAR} 占位
      telnet:                      # Telnet 连接（SSH 回退用）
        host: <Telnet_IP>         # 必填
        port: <Telnet端口>         # 必填，如 10004
        user: root                 # 必填
        password: ${FW_TELNET_PASSWORD}  # 必填，${ENV_VAR} 占位
      ssh:                         # SSH 连接（首选）
        host: <SSH_IP>            # 必填
        port: 22                   # 可选，默认 22
        user: root                 # 必填
        password: ${FW_SSH_PASSWORD}  # 必填，${ENV_VAR} 占位
        enabled: false             # 可选，默认 false（需串口开启）

    tags:                          # 可选标签
      - firewall
      - <硬件系列>
```

**Schema 约束**：
- 所有 `password` 字段值必须匹配 `${ENV_VAR}` 格式（正则 `^\$\{[A-Z_][A-Z0-9_]*\}$`）
- `device_type` 必须在 reference/device-reference.md 对照表中存在
- `ssh.enabled = false` 时 SSH 连接预期失败，自动回退 Telnet

### 5.2 templates/devices.yaml.example

提供 2 个示例设备组（1 个防火墙 + 1 个 server/client 对），全部凭据使用 `${ENV_VAR}` 占位。用户复制为工作区 `devices.yaml` 后替换实际环境变量。

### 5.3 .env.example（环境变量清单）

```
# .env.example - 凭据环境变量清单（复制为 .env 后填入实际值，.gitignore 忽略 .env）
FW_WEB_PASSWORD=          # 防火墙 Web 管理密码
FW_SSH_PASSWORD=          # 防火墙 SSH 密码
FW_TELNET_PASSWORD=       # 防火墙 Telnet 密码
```

> .env.example 文件归属于 device-management skill 目录（`skills/device-management/templates/.env.example`），不在本 Story 的 6 文件清单内，但实现时需一并创建。若 CP5 审查认为应纳入文件清单，回写 DEVELOPMENT-PLAN。

### 5.4 快照数据结构

快照 JSON 存储在 `runs/<run-id>/snapshot-{phase}/<device>.json`：

```json
{
  "device": "hg3250-51",
  "phase": "before",
  "timestamp": "2026-07-10T14:30:00",
  "connection_method": "ssh",
  "cpu": "top -bn1 | head -5 的输出文本",
  "memory": "free -m 的输出文本",
  "disk": "df -h 的输出文本",
  "processes": "ps -ef | grep opt 的输出文本"
}
```

| 字段 | 类型 | 说明 |
|------|------|------|
| `device` | string | 设备组名 |
| `phase` | string | `before` 或 `after` |
| `timestamp` | string | ISO 8601 时间戳（秒精度） |
| `connection_method` | string | `ssh` 或 `telnet`（实际采集使用的连接方式） |
| `cpu` | string | CPU 采集结果（top 输出）；失败时为 `ERROR: <exception>` |
| `memory` | string | 内存采集结果（free 输出）；失败时为 `ERROR: <exception>` |
| `disk` | string | 磁盘采集结果（df 输出）；失败时为 `ERROR: <exception>` |
| `processes` | string | 进程采集结果（ps 输出）；失败时为 `ERROR: <exception>` |

### 5.5 采集命令定义

```python
COLLECT_COMMANDS = {
    "cpu": "top -bn1 | head -5",
    "memory": "free -m",
    "disk": "df -h",
    "processes": "ps -ef | grep opt",
}
```

4 个命令均为只读观测命令，不修改设备状态。

### 5.6 持久化路径规则

| 数据 | 存储路径 | 生命周期 | .gitignore |
|------|---------|---------|------------|
| devices.yaml（运行时） | `<workspace>/devices.yaml` | 用户维护 | 是 |
| .env | `<workspace>/.env` | 用户维护 | 是 |
| devices.yaml.example（模板） | `skills/device-management/templates/devices.yaml.example` | skill 随包安装 | 否（入库） |
| 快照 before | `<workspace>/runs/<run-id>/snapshot-before/<device>.json` | 单次运行 | 是 |
| 快照 after | `<workspace>/runs/<run-id>/snapshot-after/<device>.json` | 单次运行 | 是 |

---

## 6. API / Interface 设计

### 6.1 device-management SKILL.md 接口

SKILL.md 是 agent 消费入口，定义以下操作流程：

| 操作 | 输入 | 输出 | 流程 |
|------|------|------|------|
| 添加设备 | 设备名、硬件系列、SSH IP、Telnet IP:端口、Web IP | devices.yaml 新增条目 | 查表确定 device_type -> 按模板写入 -> 凭据用 ${ENV_VAR} 占位 -> 验证 YAML 语法 |
| 查询型号 | 硬件系列或 CPU 平台 | device-reference.md 匹配行 | 在对照表按系列/平台/CPU 匹配 |
| 查询设备 | 设备名或 IP | devices.yaml 对应条目 | 按名称或 host 字段匹配 |

### 6.2 device-connection SKILL.md 接口

| 操作 | 输入 | 输出 | 调用脚本 |
|------|------|------|---------|
| 采集系统快照 | 设备名 + phase（before/after） | runs/<run-id>/snapshot-{phase}/<device>.json | `uv run python scripts/collect_sysinfo.py <设备名> --phase <before\|after> --run-id <run-id>` |
| 采集所有设备快照 | phase | 多个快照文件 | `uv run python scripts/collect_sysinfo.py --all --phase <before\|after> --run-id <run-id>` |
| 直接指定 IP 采集 | host + user + password-env + phase | 单个快照文件 | `uv run python scripts/collect_sysinfo.py --host <ip> --user <user> --password-env <ENV_VAR> --phase <before\|after> --run-id <run-id>` |
| SSH 执行单条命令 | host + user + password-env + command | stdout / stderr | `uv run python scripts/ssh_exec.py <host> <user> <password-env> <command>` |

### 6.3 ssh_exec.py 与 collect_sysinfo.py 的集成契约

| 维度 | 契约 |
|------|------|
| 调用方向 | collect_sysinfo.py 导入 ssh_exec.py 的 `ssh_exec` 函数（单向） |
| 调用时机 | collect_sysinfo.py 采集每条命令时调用 ssh_exec |
| 触发方式 | Python import 调用 |
| 输入契约 | ssh_exec(host: str, port: int, user: str, password: str, command: str, timeout: int = 15) -> tuple[str, str] |
| 输出契约 | (stdout: str, stderr: str) |
| 后续衔接 | collect_sysinfo.py 接收 stdout/stderr，组装快照 JSON |
| 降级策略 | ssh_exec 抛异常时，collect_sysinfo.py 回退到 telnet_exec |
| 调用方需同步修改范围 | 无。ssh_exec 函数签名独立稳定 |

### 6.4 与 ptm-te agent 的集成契约

| 维度 | 契约 |
|------|------|
| 调用方向 | ptm-te agent（编排流程 [2]）调用 device-management 加载 devices.yaml + device-connection 采集 before 快照；（编排流程 [8]）调用 device-connection 采集 after 快照 |
| 调用时机 | 用例执行开始（before）和结束（after） |
| 触发方式 | agent 按编排流程调用 skill 脚本 |
| 输入契约 | devices.yaml 路径 + 设备名 + run-id + phase |
| 输出契约 | snapshot-before/<device>.json + snapshot-after/<device>.json |
| 降级策略 | SSH+Telnet 双失败 -> 标记设备不可达 -> ptm-te 降级 dry-run-only（HLD §3.1） |
| 调用方需同步修改范围 | agents/ptm-te.md 编排流程 [2][8] 引用本 skill（STORY-024-01 范围） |

---

## 7. 核心处理流程

### 7.1 连接前置校验流程

```
collect_sysinfo.py 启动
  │
  ├─ [1] 检查 devices.yaml 存在
  │     └─ 不存在 -> 打印错误 + 退出（exit 1）
  │
  ├─ [2] 加载 devices.yaml，解析设备配置
  │     └─ 每个 password 字段检查 ${ENV_VAR} 格式
  │        ├─ 匹配 -> 从 os.environ 读取实际值
  │        └─ 环境变量未设置 -> 记录 ENV_NOT_READY，跳过该设备
  │
  ├─ [3] IP 可达性探测（TCP 443 或 ping）
  │     ├─ 可达 -> 继续
  │     └─ 不可达 -> 记录 UNREACHABLE，跳过该设备
  │
  └─ [4] 前置校验通过 -> 进入采集流程
```

### 7.2 系统快照采集流程（SSH -> Telnet 双轨回退）

```
对目标设备的每条采集命令（cpu/memory/disk/processes）：
  │
  ├─ [1] 尝试 SSH 执行（ssh_exec）
  │     ├─ 成功 -> 记录 connection_method=ssh，存结果
  │     └─ 失败（异常）-> 进入回退
  │
  ├─ [2] 回退 Telnet 执行（telnet_exec）
  │     ├─ 成功 -> 记录 connection_method=telnet，存结果
  │     └─ 失败 -> 记录 connection_method=none，结果为 "ERROR: <exception>"
  │
  └─ [3] 所有命令采集完成 -> 组装快照 JSON -> 写入 runs/<run-id>/snapshot-{phase}/<device>.json
```

**回退粒度**：逐命令级别。SSH 在第一条命令失败后，后续命令直接使用 Telnet（不重复尝试 SSH）。与 manaul collect_sysinfo.py `collect_from_device` 的 `use_ssh` 标志逻辑一致。

### 7.3 凭据环境变量解析流程

```
load_devices(yaml_path)
  │
  ├─ 读取 devices.yaml
  │
  ├─ 遍历 device_groups
  │   └─ 遍历 firewall.web/telnet/ssh 的 password 字段
  │       ├─ 值匹配 ${ENV_VAR} -> resolve_env_var("${ENV_VAR}") -> os.environ.get("ENV_VAR")
  │       │   ├─ 存在 -> 替换为实际值
  │       │   └─ 不存在 -> 置空字符串，标记 ENV_NOT_READY
  │       └─ 值不匹配 ${ENV_VAR} -> 警告"检测到非占位凭据"，仍使用原值（兼容）
  │
  └─ 返回 devices dict
```

### 7.4 Telnet 命令执行流程

```
telnet_exec(host, port, user, password, command, wait=3)
  │
  ├─ [1] telnetlib.Telnet(host, port, timeout=10) 建立连接
  ├─ [2] read_until(b"login:", timeout=10) -> write(user + "\r\n")
  ├─ [3] sleep(1) -> read_until(b"Password:", timeout=5) -> write(password + "\r\n")
  ├─ [4] sleep(2) -> read_very_eager() 清空缓冲
  ├─ [5] write(command + "\r\n") -> sleep(wait)
  ├─ [6] read_very_eager().decode() -> 读取输出
  ├─ [7] close() 关闭连接
  └─ [8] 清理：去掉首行命令回显 + 末尾 shell 提示符 -> 返回 (output, "")
```

---

## 8. 技术设计细节

> 本节阐述技术细节（函数级设计、数据结构、算法实现）。

### 8.1 ssh_exec.py 函数级设计

```python
#!/usr/bin/env python3
"""SSH 远程命令执行工具

声明: requires-python >=3.9,<3.13
依赖: paramiko
"""

import sys
import os
import paramiko


def ssh_connect(
    host: str,
    port: int = 22,
    user: str = "root",
    password: str = "",
    timeout: int = 15,
) -> paramiko.SSHClient:
    """建立 SSH 连接，返回 SSHClient 实例。

    参数:
        host: 目标主机 IP
        port: SSH 端口，默认 22
        user: 登录用户名，默认 root
        password: 登录密码（运行时从环境变量解析后的明文）
        timeout: 连接超时秒数，默认 15

    返回:
        paramiko.SSHClient 已连接实例

    异常:
        paramiko.AuthenticationException: 认证失败
        paramiko.SSHException: SSH 协议错误
        socket.timeout: 连接超时
        ConnectionRefusedError: 连接被拒绝
    """
    ssh = paramiko.SSHClient()
    ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
    ssh.connect(host, port=port, username=user, password=password, timeout=timeout)
    return ssh


def ssh_exec(
    host: str,
    port: int,
    user: str,
    password: str,
    command: str,
    timeout: int = 15,
) -> tuple[str, str]:
    """通过 SSH 执行单条命令，返回 (stdout, stderr)。

    参数:
        host: 目标主机 IP
        port: SSH 端口
        user: 登录用户名
        password: 登录密码（运行时已解析）
        command: 要执行的 shell 命令
        timeout: 执行超时秒数，默认 15

    返回:
        (stdout: str, stderr: str)，均已 strip() 处理

    异常:
        paramiko.AuthenticationException: 认证失败
        paramiko.SSHException: SSH 协议错误
        socket.timeout: 连接或执行超时
        Exception: 其他连接异常（由 collect_sysinfo.py 捕获触发 Telnet 回退）

    生命周期:
        建立连接 -> exec_command -> 读取 stdout/stderr -> close
        每次调用独立连接，不复用 session
    """
    ssh = ssh_connect(host, port, user, password, timeout)
    try:
        stdin, stdout, stderr = ssh.exec_command(command, timeout=timeout)
        out = stdout.read().decode(errors="replace")
        err = stderr.read().decode(errors="replace")
    finally:
        ssh.close()
    return out.strip(), err.strip()


def resolve_password_from_env(env_var: str) -> str:
    """从环境变量解析密码。

    参数:
        env_var: 环境变量名（不含 ${} 包裹）

    返回:
        环境变量值；未设置时返回空字符串

    说明:
        命令行入口使用 --password-env <ENV_VAR> 传入环境变量名，
        本函数从 os.environ 读取实际值，避免命令行明文密码。
    """
    return os.environ.get(env_var, "")


if __name__ == "__main__":
    # CLI 入口: ssh_exec.py <host> <user> <password-env> <command> [--port 22] [--timeout 15]
    # password-env 是环境变量名，不是明文密码
    import argparse

    parser = argparse.ArgumentParser(description="SSH 远程命令执行")
    parser.add_argument("host", help="目标主机 IP")
    parser.add_argument("user", help="登录用户名")
    parser.add_argument("password_env", help="密码环境变量名（如 FW_SSH_PASSWORD）")
    parser.add_argument("command", help="要执行的命令")
    parser.add_argument("--port", type=int, default=22, help="SSH 端口")
    parser.add_argument("--timeout", type=int, default=15, help="超时秒数")
    args = parser.parse_args()

    password = resolve_password_from_env(args.password_env)
    if not password:
        print(f"错误: 环境变量 {args.password_env} 未设置", file=sys.stderr)
        sys.exit(1)

    out, err = ssh_exec(args.host, args.port, args.user, password, args.command, args.timeout)
    if out:
        print(out, end="")
    if err:
        print(err, end="", file=sys.stderr)
```

**与 manaul ssh_exec.py 的差异**：

| 差异点 | manaul 原版 | 本设计 | 理由 |
|--------|-----------|--------|------|
| 函数拆分 | 单函数 ssh_exec | 拆为 ssh_connect + ssh_exec | collect_sysinfo.py 只需 ssh_exec，但 ssh_connect 可复用 |
| 类型注解 | 无 | 全覆盖 | NFR-07 可维护性 |
| 端口参数 | 无（默认 22 硬编码） | 显式 port 参数 | devices.yaml ssh.port 可配置 |
| 密码传入 | 命令行明文 sys.argv[3] | --password-env 环境变量名 | HLD §7 禁止命令行明文密码 |
| 错误处理 | 无 | try/finally 确保 close | 防止连接泄漏 |
| decode | .decode() | .decode(errors="replace") | manaul collect_sysinfo.py 已用 errors="replace"，ssh_exec 对齐 |

### 8.2 collect_sysinfo.py 函数级设计

```python
#!/usr/bin/env python3
"""采集防火墙设备系统信息快照（before/after 对比用）

声明: requires-python >=3.9,<3.13
依赖: paramiko, telnetlib(标准库), yaml(PyYAML)

用法:
    uv run python scripts/collect_sysinfo.py <设备名> --phase before --run-id <run-id>
    uv run python scripts/collect_sysinfo.py --all --phase after --run-id <run-id>
    uv run python scripts/collect_sysinfo.py --host 10.113.55.51 --phase before --run-id <run-id>
"""

import argparse
import json
import os
import re
import sys
import time
from datetime import datetime
from pathlib import Path

import paramiko
import telnetlib
import yaml

# 采集命令定义（4 维度，全部只读）
COLLECT_COMMANDS = {
    "cpu": "top -bn1 | head -5",
    "memory": "free -m",
    "disk": "df -h",
    "processes": "ps -ef | grep opt",
}

ENV_VAR_PATTERN = re.compile(r"^\$\{([A-Z_][A-Z0-9_]*)\}$")


def resolve_env_var(value: str) -> str:
    """解析 ${ENV_VAR} 占位符为环境变量实际值。

    参数:
        value: 原始值，可能为 "${FW_SSH_PASSWORD}" 或明文

    返回:
        环境变量值（占位符情况）或原值（非占位符情况）
        环境变量未设置时返回空字符串

    异常:
        无。环境变量未设置时返回空字符串，由调用方判定 ENV_NOT_READY
    """
    match = ENV_VAR_PATTERN.match(value)
    if match:
        env_var = match.group(1)
        return os.environ.get(env_var, "")
    return value


def check_env_var_set(value: str) -> bool:
    """检查 ${ENV_VAR} 占位符对应的环境变量是否已设置。

    参数:
        value: 原始值

    返回:
        True 如果值非占位符，或占位符对应环境变量已设置且非空
        False 如果占位符对应环境变量未设置或为空
    """
    match = ENV_VAR_PATTERN.match(value)
    if match:
        env_var = match.group(1)
        return bool(os.environ.get(env_var, ""))
    return True  # 非占位符视为已就绪


def load_devices(yaml_path: Path) -> dict[str, dict]:
    """从 devices.yaml 加载设备配置，解析 ${ENV_VAR} 占位凭据。

    参数:
        yaml_path: devices.yaml 文件路径

    返回:
        devices dict: {设备组名: {ssh_host, ssh_port, telnet_host, telnet_port, user, password, ...}}

    异常:
        FileNotFoundError: yaml_path 不存在
        yaml.YAMLError: YAML 语法错误

    说明:
        - password 字段自动解析 ${ENV_VAR} 占位符
        - 不修改原始 devices.yaml 文件
    """
    if not yaml_path.exists():
        raise FileNotFoundError(f"设备配置文件不存在: {yaml_path}")

    with open(yaml_path, "r", encoding="utf-8") as f:
        data = yaml.safe_load(f)

    devices = {}
    for group in data.get("device_groups", []):
        name = group.get("name", "")
        fw = group.get("firewall")
        if not fw:
            continue

        telnet = fw.get("telnet", {})
        ssh = fw.get("ssh", {})
        password_raw = ssh.get("password", fw.get("password", ""))
        password = resolve_env_var(password_raw)

        devices[name] = {
            "ssh_host": ssh.get("host", fw.get("host", "")),
            "ssh_port": ssh.get("port", 22),
            "ssh_enabled": ssh.get("enabled", False),
            "telnet_host": telnet.get("host", ""),
            "telnet_port": telnet.get("port", 23),
            "user": ssh.get("user", fw.get("user", "root")),
            "password": password,
            "password_raw": password_raw,
            "env_ready": check_env_var_set(password_raw),
        }

    return devices


def check_reachable(host: str, port: int = 443, timeout: int = 5) -> bool:
    """检查设备 IP 可达性（TCP 端口探测）。

    参数:
        host: 目标 IP
        port: 探测端口，默认 443（Web 管理端口）
        timeout: 超时秒数，默认 5

    返回:
        True 如果 TCP 连接成功
        False 如果连接失败或超时
    """
    import socket
    try:
        with socket.create_connection((host, port), timeout=timeout):
            return True
    except (socket.timeout, ConnectionRefusedError, OSError):
        return False


def ssh_exec(host, port, user, password, command, timeout=15):
    """SSH 执行单条命令（从 ssh_exec.py 导入或内联实现）。

    参数与返回值同 ssh_exec.py 的 ssh_exec 函数。
    本函数签名与 ssh_exec.py 保持一致，collect_sysinfo.py 优先 import；
    若 import 失败则使用内联实现保证脚本独立可执行。
    """
    ssh = paramiko.SSHClient()
    ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
    ssh.connect(host, port=port, username=user, password=password, timeout=timeout)
    try:
        stdin, stdout, stderr = ssh.exec_command(command, timeout=timeout)
        out = stdout.read().decode(errors="replace")
        err = stderr.read().decode(errors="replace")
    finally:
        ssh.close()
    return out.strip(), err.strip()


def telnet_exec(
    host: str,
    port: int,
    user: str,
    password: str,
    command: str,
    wait: int = 3,
) -> tuple[str, str]:
    """通过 Telnet 执行单条命令，返回 (stdout, "")。

    参数:
        host: Telnet 主机 IP
        port: Telnet 端口
        user: 登录用户名
        password: 登录密码（运行时已解析）
        command: 要执行的命令
        wait: 命令执行后等待秒数，默认 3

    返回:
        (output: str, "")：Telnet 无 stderr 概念，固定返回空字符串

    异常:
        EOFError: 连接意外关闭
        socket.timeout: 超时
        Exception: 其他 Telnet 异常

    说明:
        流程: 连接 -> login 提示 -> 用户名 -> Password 提示 -> 密码
              -> 清空缓冲 -> 发送命令 -> 等待 -> 读取输出 -> 关闭
        清理: 去掉首行命令回显 + 末尾 shell 提示符
    """
    tn = telnetlib.Telnet(host, port, timeout=10)
    tn.read_until(b"login:", timeout=10)
    tn.write(user.encode() + b"\r\n")
    time.sleep(1)
    tn.read_until(b"Password:", timeout=5)
    tn.write(password.encode() + b"\r\n")
    time.sleep(2)
    tn.read_very_eager()
    tn.write(command.encode() + b"\r\n")
    time.sleep(wait)
    output = tn.read_very_eager().decode(errors="replace")
    tn.close()
    # 清理命令回显和提示符
    lines = output.strip().splitlines()
    if lines and command in lines[0]:
        lines = lines[1:]
    while lines and re.match(r"^[#\[$]", lines[-1].strip()):
        lines.pop()
    return "\n".join(lines).strip(), ""


def collect_from_device(
    name: str,
    config: dict,
) -> tuple[dict, str]:
    """采集设备系统信息，SSH 首选 -> Telnet 回退。

    参数:
        name: 设备组名
        config: 设备配置 dict（含 ssh_host/ssh_port/telnet_host/telnet_port/user/password）

    返回:
        (results: dict, connection_method: str)
        results: {cpu: str, memory: str, disk: str, processes: str}
        connection_method: "ssh" | "telnet" | "none"

    说明:
        - 逐命令级别回退：SSH 在第一条命令失败后，后续命令直接使用 Telnet
        - SSH/Telnet 双失败时结果为 "ERROR: <exception>"
        - connection_method 记录实际使用的连接方式
    """
    results = {}
    use_ssh = True
    connection_method = "none"

    for key, cmd in COLLECT_COMMANDS.items():
        try:
            if use_ssh:
                out, err = ssh_exec(
                    config["ssh_host"], config["ssh_port"],
                    config["user"], config["password"], cmd
                )
                connection_method = "ssh"
                results[key] = out if out else err
            else:
                out, err = telnet_exec(
                    config["telnet_host"], config["telnet_port"],
                    config["user"], config["password"], cmd
                )
                connection_method = "telnet"
                results[key] = out if out else err
        except Exception as e:
            if use_ssh:
                print(f"  SSH 失败 ({e})，回退到 Telnet...")
                use_ssh = False
                try:
                    out, err = telnet_exec(
                        config["telnet_host"], config["telnet_port"],
                        config["user"], config["password"], cmd
                    )
                    connection_method = "telnet"
                    results[key] = out if out else err
                except Exception as e2:
                    print(f"  Telnet 也失败: {e2}")
                    results[key] = f"ERROR: {e2}"
            else:
                print(f"  Telnet 失败: {e}")
                results[key] = f"ERROR: {e}"

    return results, connection_method


def save_snapshot(
    device: str,
    phase: str,
    data: dict,
    connection_method: str,
    run_id: str,
    workspace: Path,
) -> Path:
    """保存快照到 runs/<run-id>/snapshot-{phase}/<device>.json。

    参数:
        device: 设备组名
        phase: "before" 或 "after"
        data: 采集结果 dict（cpu/memory/disk/processes）
        connection_method: "ssh" | "telnet" | "none"
        run_id: 运行 ID（如 20260710-143000）
        workspace: 工作区根目录 Path

    返回:
        快照文件路径 Path

    说明:
        - 自动创建目录 runs/<run-id>/snapshot-{phase}/
        - JSON 格式: ensure_ascii=False, indent=2
        - 含元数据: device / phase / timestamp / connection_method
    """
    snapshot_dir = workspace / "runs" / run_id / f"snapshot-{phase}"
    snapshot_dir.mkdir(parents=True, exist_ok=True)
    filepath = snapshot_dir / f"{device}.json"

    snapshot = {
        "device": device,
        "phase": phase,
        "timestamp": datetime.now().isoformat(timespec="seconds"),
        "connection_method": connection_method,
        **data,
    }
    filepath.write_text(json.dumps(snapshot, ensure_ascii=False, indent=2))
    return filepath


def main():
    """CLI 入口函数。

    参数:
        device: 设备名（位置参数，可选，与 --all / --host 三选一）
        --phase: 采集阶段 before|after（必填）
        --all: 采集所有设备
        --host: 直接指定 IP
        --port: SSH 端口（默认 22）
        --user: 登录用户（默认 root）
        --password-env: 密码环境变量名（--host 模式必填）
        --telnet-host: Telnet 主机（回退用）
        --telnet-port: Telnet 端口（默认 23）
        --config: devices.yaml 路径（默认工作区根目录）
        --run-id: 运行 ID（必填，用于快照存储路径）
        --workspace: 工作区根目录（默认当前目录）

    退出码:
        0: 成功
        1: 错误（设备未找到、配置缺失、环境变量未设置等）
    """
    parser = argparse.ArgumentParser(description="采集防火墙系统信息快照")
    parser.add_argument("device", nargs="?", help="设备名（如 hg3250-51）")
    parser.add_argument("--phase", required=True, choices=["before", "after"], help="采集阶段")
    parser.add_argument("--all", action="store_true", help="采集所有设备")
    parser.add_argument("--host", help="直接指定设备 IP")
    parser.add_argument("--port", type=int, default=22, help="SSH 端口（默认 22）")
    parser.add_argument("--user", default="root", help="登录用户（默认 root）")
    parser.add_argument("--password-env", help="密码环境变量名（--host 模式必填）")
    parser.add_argument("--telnet-host", help="Telnet 主机（回退用）")
    parser.add_argument("--telnet-port", type=int, default=23, help="Telnet 端口")
    parser.add_argument("--config", default="devices.yaml", help="设备配置文件路径")
    parser.add_argument("--run-id", required=True, help="运行 ID（快照存储路径用）")
    parser.add_argument("--workspace", default=".", help="工作区根目录")
    args = parser.parse_args()

    workspace = Path(args.workspace).resolve()
    config_path = workspace / args.config if not Path(args.config).is_absolute() else Path(args.config)

    # --host 直接指定 IP 模式
    if args.host:
        if not args.password_env:
            parser.error("--host 模式需要 --password-env 参数")
        password = os.environ.get(args.password_env, "")
        if not password:
            print(f"错误: 环境变量 {args.password_env} 未设置", file=sys.stderr)
            sys.exit(1)
        config = {
            "ssh_host": args.host,
            "ssh_port": args.port,
            "ssh_enabled": True,
            "telnet_host": args.telnet_host or args.host,
            "telnet_port": args.telnet_port,
            "user": args.user,
            "password": password,
        }
        name = args.device or args.host
        targets = {name: config}
    else:
        # 从 devices.yaml 加载
        try:
            devices = load_devices(config_path)
        except FileNotFoundError as e:
            print(f"错误: {e}", file=sys.stderr)
            sys.exit(1)

        if args.all:
            if not devices:
                print("错误: 未找到设备配置，请检查 devices.yaml", file=sys.stderr)
                sys.exit(1)
            targets = devices
        elif args.device:
            device_key = None
            for key in devices:
                if key.upper() == args.device.upper():
                    device_key = key
                    break
            if device_key:
                targets = {device_key: devices[device_key]}
            else:
                print(f"错误: 未知设备 '{args.device}'", file=sys.stderr)
                print(f"可选设备: {', '.join(devices.keys())}", file=sys.stderr)
                sys.exit(1)
        else:
            parser.error("请指定设备名、使用 --all 或 --host")

    # 逐设备采集
    for name, config in targets.items():
        # 前置校验: 环境变量就绪
        if not config.get("env_ready", True):
            print(f"警告: 设备 {name} 环境变量未设置，跳过", file=sys.stderr)
            continue

        # 前置校验: IP 可达
        if not check_reachable(config["ssh_host"]):
            print(f"警告: 设备 {name} ({config['ssh_host']}) 不可达，跳过", file=sys.stderr)
            continue

        print(f"\n{'='*50}")
        print(f"采集设备: {name} ({args.phase})")
        print(f"连接地址: {config['ssh_host']}:{config['ssh_port']}")
        print(f"{'='*50}")

        data, conn_method = collect_from_device(name, config)
        filepath = save_snapshot(name, args.phase, data, conn_method, args.run_id, workspace)

        print(f"\n采集结果 (连接方式: {conn_method}):")
        for key, value in data.items():
            preview = value[:80].replace("\n", " | ") if value else "(空)"
            print(f"  {key}: {preview}...")
        print(f"\n快照已保存: {filepath}")


if __name__ == "__main__":
    main()
```

### 8.3 与 manaul collect_sysinfo.py 的差异

| 差异点 | manaul 原版（236 行） | 本设计（~280 行） | 理由 |
|--------|---------------------|-------------------|------|
| 快照路径 | `snapshots/{device}_{phase}.json` | `runs/<run-id>/snapshot-{phase}/<device>.json` | HLD §2.2 运行时工作目录规范 |
| 凭据解析 | 仅 firewall.password 字段解析 ${ENV_VAR} | ssh/telnet/web 全部 password 字段解析 | HLD §7 所有凭据用 ${ENV_VAR} 占位 |
| CLI 密码参数 | `--password`（明文） | `--password-env`（环境变量名） | HLD §7 禁止命令行明文密码 |
| run-id 参数 | 无 | `--run-id` 必填 | 快照路径需要 run-id |
| workspace 参数 | 无（硬编码脚本父目录） | `--workspace` 参数 | 快照路径可配置 |
| IP 可达性检查 | 无 | `check_reachable()` 前置校验 | HLD §6.1 连接前置校验 |
| 环境变量就绪检查 | 无 | `check_env_var_set()` + `env_ready` 标志 | HLD §6.1 连接前置校验 |
| connection_method 记录 | 无 | 快照 JSON 含 `connection_method` 字段 | 审计追溯实际连接方式 |
| ssh_exec 导入 | 内联定义 | 优先 import ssh_exec.py，fallback 内联 | 模块化复用 |
| 类型注解 | 无 | 全覆盖公共函数 | NFR-07 |
| resolve_env_var | 内联在 load_devices | 独立函数 + 正则匹配 | 可测试性 + 可复用 |

### 8.4 device-management SKILL.md 结构设计

```
---
name: device-management
description: 设备清单管理与型号映射查表。触发词：添加设备、查询型号、设备对照、查找设备、device_type、硬件系列。
argument-hint: "<操作> [设备信息] 如：添加设备 hg3250-51 10.113.55.51、查询型号 HG3250"
user-invokable: true
status: active
---

# 设备管理（元数据）

## 目的
管理 devices.yaml 中的设备列表，并在添加或查询设备时从型号对照表查找正确的 device_type。
本 SKILL 只做元数据管理，不含连接逻辑（连接由 device-connection skill 承载）。

## 设备型号参考
链接 reference/device-reference.md

## 添加设备流程
1. 提取设备名、硬件系列、SSH IP、Telnet IP:端口、Web IP
2. 查表确定 device_type
3. 按 templates/devices.yaml.example 模板写入，凭据用 ${ENV_VAR} 占位
4. 验证 YAML 语法

## 查询设备流程
1. 按名称或 IP 在 devices.yaml 中查找
2. 返回匹配条目

## 凭据安全
- 所有 password 字段必须使用 ${ENV_VAR} 占位
- 禁止明文凭据写入 devices.yaml
- 用户需在 .env 中设置对应环境变量

## 模板
引用 templates/devices.yaml.example
```

### 8.5 device-connection SKILL.md 结构设计

```
---
name: device-connection
description: 设备 SSH/Telnet 双轨连接与系统快照采集。触发词：采集快照、系统信息、连接设备、snapshot、sysinfo。
argument-hint: "<设备名> --phase <before|after> --run-id <run-id>"
user-invokable: true
status: active
---

# 设备连接与快照采集

## 目的
提供 SSH（paramiko 首选）/ Telnet（telnetlib 回退）双轨连接能力，
采集设备系统信息快照（CPU/内存/磁盘/进程 4 维度），用于用例执行 before/after 对比。
本 SKILL 只做连接 + 快照，不管理设备清单（清单由 device-management skill 维护）。

## 连接模型
- SSH 首选（paramiko），失败回退 Telnet（telnetlib）
- 逐命令级别回退：SSH 第一条命令失败后，后续命令直接用 Telnet
- SSH + Telnet 双失败 -> 标记设备不可达

## 前置校验
1. devices.yaml 存在
2. ${ENV_VAR} 占位符对应环境变量已设置
3. 设备 IP 可达（TCP 443 探测）

## 脚本
- scripts/ssh_exec.py: SSH 远程命令执行
- scripts/collect_sysinfo.py: 系统信息采集 + 快照存储

## Python 版本
>=3.9,<3.13（telnetlib 在 3.13 移除）

## 快照输出
runs/<run-id>/snapshot-before/<device>.json
runs/<run-id>/snapshot-after/<device>.json

## 凭据安全
- 密码从环境变量读取，--password-env 传入变量名
- 禁止命令行明文密码
```

---

## 9. 安全与性能设计

### 9.1 凭据安全（核心安全设计）

| 安全项 | 策略 | 实现 |
|--------|------|------|
| devices.yaml 凭据 | 100% `${ENV_VAR}` 占位，0 明文 | templates/devices.yaml.example 全部占位；SKILL.md 添加设备流程强制占位 |
| .gitignore | 忽略 devices.yaml + .env + runs/ | 用户工作区 .gitignore 需包含这三项 |
| 命令行密码 | 禁止 `--password` 明文参数 | ssh_exec.py CLI 用 `--password-env`；collect_sysinfo.py 同理 |
| 环境变量读取 | 运行时从 os.environ 解析 | `resolve_env_var()` 函数 |
| 环境变量未设置 | 不崩溃，标记 ENV_NOT_READY 跳过设备 | `check_env_var_set()` + `env_ready` 标志 |
| manaul 反模式规避 | manaul devices.yaml 明文凭据（ngfw123!@# 等）不照搬 | templates/devices.yaml.example 全部改 `${ENV_VAR}` |
| paramiko host key | AutoAddPolicy（自动接受） | 测试环境可接受；生产环境需评估是否改用 known_hosts |

### 9.2 .gitignore 规则

用户工作区 `.gitignore` 必须包含：

```gitignore
# 设备凭据
devices.yaml
.env

# 运行产物
runs/
```

### 9.3 连接安全

| 项 | 策略 |
|----|------|
| SSH 加密 | paramiko 默认加密传输 |
| Telnet 明文 | Telnet 协议本身明文传输；仅作为 SSH 不通时的回退，不用于敏感操作 |
| 超时控制 | SSH 连接 15s、Telnet 连接 10s、命令执行 15s |
| 连接关闭 | try/finally 确保 SSH 连接关闭；Telnet 显式 close() |

### 9.4 性能设计

| 项 | 度量值 | 说明 |
|----|--------|------|
| 单设备采集耗时 | ≤ 30s（4 命令 × 15s 超时 + 回退） | 最坏情况 SSH 全失败 + Telnet 全成功 |
| 多设备采集 | 串行 | `--all` 模式逐设备采集；不并行（避免连接数膨胀） |
| 快照文件大小 | ≤ 50KB/设备 | 4 维度文本输出，JSON 序列化 |
| 内存占用 | ≤ 50MB | paramiko + telnetlib + yaml 依赖 |

---

## 10. 测试设计

### 10.1 静态测试

| 测试项 | 方法 | 期望结果 | 工具 |
|--------|------|---------|------|
| Python 语法检查 | `python -m py_compile` | 0 错误 | python |
| 类型注解检查 | mypy（可选） | 公共函数全覆盖 | mypy |
| SKILL.md frontmatter 校验 | 检查 name/description/status 字段 | 3 个必填字段存在 | 手动 / 脚本 |
| devices.yaml.example 凭据检查 | 正则 `^\$\{[A-Z_][A-Z0-9_]*\}$` | 100% password 字段匹配 | grep / python |
| .gitignore 规则检查 | 检查 devices.yaml / .env / runs/ 存在 | 3 条规则存在 | grep |

### 10.2 单元测试

| 测试对象 | 测试用例 | 输入 | 期望输出 |
|---------|---------|------|---------|
| `resolve_env_var()` | 占位符解析 | `"${FW_SSH_PASSWORD}"`，env 已设 | 环境变量值 |
| `resolve_env_var()` | 占位符未设置 | `"${NOT_SET}"`，env 未设 | `""` |
| `resolve_env_var()` | 非占位符 | `"plaintext"` | `"plaintext"` |
| `check_env_var_set()` | 占位符已设 | `"${FW_SSH_PASSWORD}"`，env 已设 | `True` |
| `check_env_var_set()` | 占位符未设 | `"${NOT_SET}"`，env 未设 | `False` |
| `check_env_var_set()` | 非占位符 | `"plaintext"` | `True` |
| `check_reachable()` | 可达 IP | `("127.0.0.1", 22)`，本地 SSH | `True` |
| `check_reachable()` | 不可达 IP | `("192.0.2.1", 443)`，超时 | `False` |
| `ENV_VAR_PATTERN` | 合法占位符 | `"${FW_WEB_PASSWORD}"` | 匹配 |
| `ENV_VAR_PATTERN` | 非法占位符 | `"${fw-password}"` | 不匹配 |
| `load_devices()` | 合法 YAML | devices.yaml.example | dict 含设备组 |
| `load_devices()` | 文件不存在 | `/nonexistent.yaml` | FileNotFoundError |
| `save_snapshot()` | 写入快照 | 临时目录 + mock 数据 | 文件存在 + JSON 合法 |

### 10.3 集成测试（需真实设备）

| 测试项 | 前置 | 步骤 | 期望结果 |
|--------|------|------|---------|
| SSH 连接 + 采集 | hg3250-51 SSH 可达 | `collect_sysinfo.py hg3250-51 --phase before --run-id test` | 快照文件生成，connection_method=ssh |
| Telnet 回退 | hg3250-51 SSH 不通 + Telnet 可达 | 同上 | 快照文件生成，connection_method=telnet |
| 双轨失败 | hg3250-51 SSH + Telnet 均不通 | 同上 | 快照文件生成，结果为 ERROR |
| --all 模式 | 多设备在 devices.yaml | `collect_sysinfo.py --all --phase before --run-id test` | 每设备一个快照文件 |
| --host 模式 | 直接指定 IP | `collect_sysinfo.py --host 10.113.55.51 --password-env FW_SSH_PASSWORD --phase before --run-id test` | 快照文件生成 |
| before/after 对比 | 同设备两次采集 | 先 `--phase before` 再 `--phase after` | 两个快照文件，路径不同 |
| 环境变量未设置 | unset FW_SSH_PASSWORD | 同上 | 警告 + 跳过设备 |
| IP 不可达 | 不可达 IP in devices.yaml | 同上 | 警告 + 跳过设备 |

### 10.4 验证模式

`validation_mode=mixed`（HLD §14）：
- **static**：语法检查 + 单元测试 + frontmatter 校验 + .gitignore 检查 + 凭据占位检查
- **runtime**：需 DQ-01 授权 + DQ-02 设备 hg3250-51；SSH/Telnet 连接 + 快照采集

---

## 11. 实施步骤

### 步骤 1：创建 device-management skill 目录结构

```bash
mkdir -p skills/device-management/reference
mkdir -p skills/device-management/templates
```

### 步骤 2：编写 skills/device-management/SKILL.md

内容结构：
1. frontmatter：name=device-management, description, argument-hint, user-invokable=true, status=active
2. 目的：管理 devices.yaml + 型号映射查表，不含连接逻辑
3. 设备型号参考：链接 reference/device-reference.md
4. 添加设备流程：提取信息 -> 查表 -> 按模板写入（${ENV_VAR} 占位）-> 验证 YAML
5. 查询设备流程：按名称/IP 查找
6. 凭据安全：强制 ${ENV_VAR} 占位，禁止明文
7. 模板引用：templates/devices.yaml.example

### 步骤 3：编写 skills/device-management/reference/device-reference.md

内容结构：
1. 硬件系列 -> TGFW 型号对照表（表格）
2. 覆盖 manaul 现有全部型号：HG3250/A1300-HU、NXP1046/1900、160pro/160-PRO、160/160、290/290、TG-C236/trex-236、TG-J1900
3. 型号特征列（CPU 平台 / 内存 / 硬盘）用于消歧
4. 添加新设备时的查表说明

### 步骤 4：编写 skills/device-management/templates/devices.yaml.example

内容结构：
1. 注释说明：凭据用 ${ENV_VAR} 占位，禁止明文
2. 2 个示例设备组：
   - hg3250-51（防火墙，含 web/telnet/ssh 三种连接方式，全部 ${ENV_VAR} 占位）
   - tgfw-primary（server/client 对，凭据 ${ENV_VAR} 占位）
3. 所有 password 字段值为 `${FW_WEB_PASSWORD}` / `${FW_SSH_PASSWORD}` / `${FW_TELNET_PASSWORD}`

### 步骤 5：创建 device-connection skill 目录结构

```bash
mkdir -p skills/device-connection/scripts
```

### 步骤 6：编写 skills/device-connection/SKILL.md

内容结构：
1. frontmatter：name=device-connection, description, argument-hint, user-invokable=true, status=active
2. 目的：SSH/Telnet 双轨连接 + 快照采集，不管理清单
3. 连接模型：SSH 首选 -> Telnet 回退 -> 双失败标记不可达
4. 前置校验：devices.yaml 存在 + 环境变量已设 + IP 可达
5. 脚本说明：ssh_exec.py + collect_sysinfo.py 用法和参数
6. Python 版本声明：>=3.9,<3.13
7. 快照输出路径：runs/<run-id>/snapshot-{phase}/<device>.json
8. 凭据安全：--password-env 传环境变量名，禁止明文

### 步骤 7：编写 skills/device-connection/scripts/ssh_exec.py

按 §8.1 函数级设计实现：
1. 文件头 docstring + requires-python 声明
2. import paramiko, os, sys, argparse
3. `ssh_connect()` 函数
4. `ssh_exec()` 函数（try/finally 确保 close）
5. `resolve_password_from_env()` 函数
6. `__main__` CLI 入口（argparse，--password-env 非明文）

### 步骤 8：编写 skills/device-connection/scripts/collect_sysinfo.py

按 §8.2 函数级设计实现：
1. 文件头 docstring + requires-python 声明
2. import paramiko, telnetlib, yaml, json, os, re, sys, time, argparse, pathlib
3. `COLLECT_COMMANDS` 常量（4 维度）
4. `ENV_VAR_PATTERN` 正则常量
5. `resolve_env_var()` 函数
6. `check_env_var_set()` 函数
7. `load_devices()` 函数
8. `check_reachable()` 函数
9. `ssh_exec()` 函数（优先 import ssh_exec.py，fallback 内联）
10. `telnet_exec()` 函数
11. `collect_from_device()` 函数
12. `save_snapshot()` 函数
13. `main()` CLI 入口

### 步骤 9：编写 skills/device-management/templates/.env.example

```
# 凭据环境变量清单（复制为 .env 后填入实际值）
FW_WEB_PASSWORD=
FW_SSH_PASSWORD=
FW_TELNET_PASSWORD=
```

### 步骤 10：验证

1. `python -m py_compile skills/device-connection/scripts/ssh_exec.py`
2. `python -m py_compile skills/device-connection/scripts/collect_sysinfo.py`
3. 检查 devices.yaml.example 所有 password 字段为 `${ENV_VAR}` 格式
4. 检查 SKILL.md frontmatter 完整
5. 运行单元测试（resolve_env_var / check_env_var_set / check_reachable）

---

## 12. 风险、难点与预研建议

### 12.1 风险矩阵

| ID | 风险 | 等级 | 影响 | 缓解措施 | 回退路径 |
|----|------|------|------|---------|---------|
| R-01 | telnetlib 在 Python 3.13 移除 | 中 | Telnet 回退不可用 | 脚本声明 `>=3.9,<3.13`；SKILL.md 注明版本约束 | 改用 `telnetlib3`（第三方库） |
| R-02 | paramiko 依赖安装失败 | 低 | SSH 连接不可用 | SKILL.md 注明 `pip install paramiko` 或 `uv add paramiko` | 回退到 Telnet only 模式 |
| R-03 | 凭据泄露（devices.yaml 明文入库） | 高 | 安全风险 | templates 全部 ${ENV_VAR} 占位；.gitignore 忽略；SKILL.md 强制规则 | 凭据轮换 + 审计 |
| R-04 | SSH host key AutoAddPolicy 安全风险 | 低 | 中间人攻击 | 测试环境可接受；SKILL.md 注明生产环境需评估 | 改用 known_hosts 策略 |
| R-05 | Telnet 登录提示符不匹配 | 中 | Telnet 回退失败 | `read_until(b"login:")` + `read_until(b"Password:")` 标准提示符 | 调整 wait 超时或提示符匹配 |
| R-06 | 设备 SSH 端口未开启（enabled: false） | 中 | SSH 首选失败，每次都先超时再回退 | devices.yaml ssh.enabled 标志；SKILL.md 注明需串口开启 SSH | 直接使用 Telnet |
| R-07 | PyYAML 依赖与目标项目版本冲突 | 低 | load_devices 失败 | 声明 PyYAML 最低版本 | 使用 ruamel.yaml 替代 |

### 12.2 Gotchas

1. **telnetlib 是标准库但在 3.13 移除**：Python 3.13+ 运行 collect_sysinfo.py 会 `ImportError: No module named 'telnetlib'`。脚本头必须声明 `requires-python >=3.9,<3.13`，SKILL.md 必须注明。
2. **paramiko AutoAddPolicy 不验证 host key**：测试环境方便，但存在中间人攻击风险。生产环境应改用 `RejectPolicy` + known_hosts 文件。
3. **devices.yaml 明文凭据是反模式**：manaul devices.yaml 含 `ngfw123!@#` 等明文密码，迁移时必须全部改 `${ENV_VAR}` 占位。HLD Gotcha #12。
4. **device-management 不含连接逻辑**：manaul 的连接在 collect_sysinfo.py，不在 device-management SKILL。拆分迁移时不要把连接塞进元数据 SKILL。HLD Gotcha #8。
5. **SSH 逐命令回退不是逐设备回退**：SSH 在第一条命令失败后，后续命令直接用 Telnet，不会每条命令都先试 SSH。这与 manaul collect_sysinfo.py `use_ssh` 标志逻辑一致。
6. **Telnet 输出需清理回显和提示符**：Telnet 会回显输入的命令，末尾有 shell 提示符（#/$/[），需清理。manaul 的清理逻辑保留。
7. **--password-env 传的是变量名不是密码**：ssh_exec.py CLI 的第三个参数是环境变量名（如 `FW_SSH_PASSWORD`），不是密码本身。脚本内部 `os.environ.get()` 读取实际值。
8. **快照路径含 run-id**：快照不存固定 `snapshots/` 目录，而是 `runs/<run-id>/snapshot-{phase}/`。collect_sysinfo.py 的 `--run-id` 是必填参数。
9. **check_reachable 用 TCP 443 不是 ping**：部分设备禁 ICMP，TCP 端口探测更可靠。默认探测 443（Web 管理端口）。
10. **ssh.enabled=false 不阻止 SSH 尝试**：devices.yaml 的 `ssh.enabled` 是信息标志，collect_sysinfo.py 仍会尝试 SSH（预期失败后回退 Telnet）。若要跳过 SSH 直接用 Telnet，需在 SKILL.md 说明或未来加 `--prefer-telnet` 参数。

### 12.3 预研建议

| 预研项 | 必要性 | 说明 |
|--------|--------|------|
| telnetlib3 替代方案 | 低（3.13 普及前不紧急） | 调研 telnetlib3 API 兼容性，为 Python 3.13+ 做预案 |
| paramiko known_hosts 策略 | 低（测试环境够用） | 调研生产环境 host key 验证方案 |
| 快照 diff 自动化 | 中（after-before 对比） | 调研快照 diff 工具或脚本，当前只存储不对比 |

---

## 13. 回滚与发布策略

### 13.1 回滚策略

| 回滚场景 | 触发条件 | 回滚动作 |
|---------|---------|---------|
| skill 文件错误 | SKILL.md frontmatter 缺失或格式错误 | 修正 frontmatter，重新安装 |
| 脚本语法错误 | py_compile 失败 | 修正语法，重新验证 |
| SSH/Telnet 连接全失败 | 真实设备不可达 | 降级 dry-run-only（HLD §3.1），runtime 转 follow-up |
| 凭据泄露 | devices.yaml 明文凭据入库 | 立即从 git 历史移除 + 凭据轮换 + 审计 |
| paramiko/telnetlib 依赖冲突 | import 失败 | 修复依赖版本或降级到 Telnet only |

### 13.2 回滚不影响项

- device-management 和 device-connection 是新建 skill，不修改已有文件
- 回滚只需删除 `skills/device-management/` 和 `skills/device-connection/` 目录
- 不影响 STORY-024-01（agents/ptm-te.md）和 STORY-024-03（policy-route-execution）

### 13.3 发布策略

| 项 | 策略 |
|----|------|
| 发布形态 | skill 目录（SKILL.md + reference/ + templates/ + scripts/） |
| 安装方式 | install.py 投影（STORY-024-04 实现） |
| 版本 | 随 CR-024 发布，不独立版本化 |
| 文档更新 | skills/README.md 索引（STORY-024-04 实现） |
| 依赖声明 | SKILL.md 注明 paramiko + PyYAML + Python >=3.9,<3.13 |

---

## 14. Definition of Done

### 14.1 量化验收项

| # | 验收项 | 度量值 | 验证方法 |
|---|--------|--------|---------|
| DoD-01 | 2 个 skill 创建完成 | `skills/device-management/` + `skills/device-connection/` | 目录存在检查 |
| DoD-02 | 6 文件全部存在 | SKILL.md×2 + device-reference.md + devices.yaml.example + ssh_exec.py + collect_sysinfo.py | 文件存在检查 |
| DoD-03 | SSH/Telnet 双轨 + 回退 | SSH 首选，失败回退 Telnet，双失败标记不可达 | collect_sysinfo.py 代码审查 + 集成测试 |
| DoD-04 | 快照 before/after 两点 | `runs/<run-id>/snapshot-before/` + `snapshot-after/` | collect_sysinfo.py `--phase before\|after` 测试 |
| DoD-05 | 快照 4 维度 | CPU + 内存 + 磁盘 + 进程 | 快照 JSON 含 4 个字段 |
| DoD-06 | 凭据 ${ENV_VAR} 占位 | devices.yaml.example 所有 password 字段为 `${ENV_VAR}` 格式 | 正则校验 100% 匹配 |
| DoD-07 | 禁止命令行明文密码 | ssh_exec.py + collect_sysinfo.py CLI 用 `--password-env` | 代码审查无 `--password` 参数 |
| DoD-08 | Python 版本声明 | ssh_exec.py + collect_sysinfo.py docstring 含 `>=3.9,<3.13` | 文件头检查 |
| DoD-09 | .gitignore 规则 | devices.yaml + .env + runs/ 三条规则 | 用户工作区 .gitignore 检查 |
| DoD-10 | Python 语法通过 | `python -m py_compile` 0 错误 | 编译检查 |
| DoD-11 | SKILL.md frontmatter 完整 | name + description + status 3 字段存在 | frontmatter 检查 |
| DoD-12 | 型号对照表覆盖 | reference/device-reference.md 含 manaul 全部型号 | 对照检查 |
| DoD-13 | 连接前置校验 | devices.yaml 存在 + 环境变量已设 + IP 可达 | 代码审查 + 测试 |
| DoD-14 | 快照含 connection_method | JSON 含 `connection_method` 字段 | 快照 JSON 检查 |
| DoD-15 | 函数类型注解 | 公共函数 100% 覆盖类型注解 | 代码审查 |
| DoD-16 | device-management 不含连接逻辑 | SKILL.md 无 SSH/Telnet/paramiko/telnetlib 引用 | 代码审查 |
| DoD-17 | device-connection 不管理清单 | SKILL.md 无 devices.yaml 写入/添加设备流程 | 代码审查 |

### 14.2 CP6 检查项

- [ ] 6 文件全部创建
- [ ] Python 语法编译通过
- [ ] SKILL.md frontmatter 完整
- [ ] devices.yaml.example 凭据全部 ${ENV_VAR} 占位
- [ ] 函数类型注解覆盖
- [ ] 实现执行证据写入 `process/stories/STORY-024-02-device-management-connection-IMPLEMENTATION.md`

### 14.3 CP7 检查项

- [ ] static：语法检查 PASS
- [ ] static：单元测试 PASS（resolve_env_var / check_env_var_set / check_reachable）
- [ ] static：凭据占位校验 PASS
- [ ] static：SKILL.md frontmatter 校验 PASS
- [ ] runtime（需 DQ-01 授权 + DQ-02 设备）：SSH 连接 + 快照采集 PASS
- [ ] runtime：Telnet 回退 PASS（或 N/A + 原因）
- [ ] runtime：before/after 快照生成 PASS

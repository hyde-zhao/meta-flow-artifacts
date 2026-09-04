---
cr_id: "CR-033"
artifact_type: "dependency-map"
version: "1.1"
created_at: "2026-07-28T11:20:00+08:00"
author: "meta-se"
status: "draft"
source_blueprint: "docs/design/BLUEPRINT-PTM-TE-EXEC.md"
---

# CR-033 ptm-te 执行引擎依赖图

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-28 | meta-se | CR-033 依赖方向与禁止依赖初稿 |
| 1.1 | 2026-07-28 | meta-se | CP3 评审范围扩展：环境文件驱动 resolve_env_refs。op_mapper 依赖 env_topology（case_runner load_env_file 加载传入）；数据依赖图加 env-file/build_env_topology/resolve_env_refs；文件所有权矩阵 op_mapper 加 resolve_env_refs，ST-EX-13 加 ${ENV.*} 改写 |

## 1. 模块依赖方向

### 1.1 模块层次

```
┌─────────────────────────────────────────────────────────┐
│  Layer 4: 编排层（case_runner.py）                       │
│  skills/case-execution/scripts/case_runner.py            │
│  职责：用例发现/解析/执行编排/四态分级/报告生成           │
└──────────────┬──────────────────────────────────────────┘
               │ import（AGA-01=A）
               v
┌─────────────────────────────────────────────────────────┐
│  Layer 3: 映射+执行层（op_mapper.py）                    │
│  skills/policy-route-execution/scripts/op_mapper.py      │
│  职责：op_id->CLI 映射/build_command/execute_op/rollback  │
└──────────────┬──────────────────────────────────────────┘
               │ subprocess
               v
┌─────────────────────────────────────────────────────────┐
│  Layer 2: CLI 层（ptm-atomic）                           │
│  外部依赖，不改本体（R-C-001）                            │
│  职责：auth/policy-route/object/interface/tg/operation-log│
└──────────────┬──────────────────────────────────────────┘
               │ HTTP/HTTPS
               v
┌─────────────────────────────────────────────────────────┐
│  Layer 1: 设备层（DUT + TG）                              │
│  DUT: 防火墙 Web API                                      │
│  TG: trex-api FastAPI 服务（:8000）                      │
└─────────────────────────────────────────────────────────┘
```

### 1.2 数据依赖

```
devices.yaml ──读──> case_runner.py ──读──> topology yaml / env-file
    │                   │                        │
    │ tg.api_server     │ firewall.host           │ port_mapping/nodes/links
    │ (fallback)        │                        v
    v                   v                build_env_topology -> env_topology
op_mapper._build_exec_env    op_mapper.execute_op <──── resolve_env_refs(args, env_topology)
    │                            │
    │ TREX_API_URL 注入            │ --base-url
    v                            v
   TG op 执行                   DUT op 执行
```

### 1.3 Skill 间依赖

| 依赖方 | 被依赖方 | 依赖类型 | 接口 | 说明 |
|---|---|---|---|---|
| case-execution | policy-route-execution | import | `from op_mapper import execute_op, build_command, handle_rollback` | AGA-01=A，进程内函数调用 |
| case-execution | device-management | 数据 | devices.yaml tg 块 + firewall 块 | case_runner 读取设备元数据 |
| case-execution | traffic-skill / trex-traffic | 数据 | topology yaml / env-file | case_runner 读取 TG 接口映射 + 环境契约（port_mapping/nodes/links，ADR-09） |
| case-execution | ptm-atomic | 间接 | op_mapper -> subprocess -> ptm-atomic | 不直接依赖 |
| policy-route-execution (op_mapper) | env-file (env_topology) | 数据输入 | env_topology 由 case_runner load_env_file 加载传入 | resolve_env_refs 消费 env_topology 解析 ${ENV.*}（ADR-09，CP3 评审范围扩展） |
| install.py | case-execution | 安装 | script/ptm_team/install.py 安装 skill 到 workspace | FE-EX-03 EP-EX-06 |
| install.py | policy-route-execution | 安装 | 同上 | 既有 |

## 2. 依赖方向矩阵

| 模块 | case-execution | policy-route-execution | device-management | traffic-skill | trex-traffic | device-connection | ngfw-install | ptm-atomic | install.py |
|---|---|---|---|---|---|---|---|---|---|
| case-execution | - | **import** | 读 devices.yaml | 读 topology yaml | - | - | - | 间接 | - |
| policy-route-execution | - | - | - | - | - | - | - | subprocess | - |
| device-management | - | - | - | - | - | - | - | - | - |
| traffic-skill | - | - | - | - | - | - | - | - | - |
| trex-traffic | - | - | - | - | - | - | - | - | - |
| install.py | 安装 | 安装 | 安装 | - | 安装 | - | - | - | - |

**说明**：粗体为 CR-033 新增依赖。

## 3. 禁止依赖

| 禁止方向 | 原因 | 约束来源 |
|---|---|---|
| policy-route-execution -> case-execution | 映射层不依赖编排层（避免循环） | AGA-01=A 职责边界 |
| device-management -> case-execution | 设备管理不依赖执行引擎 | Feature 边界 |
| case-execution -> ngfw-install | 执行引擎不依赖安装 skill | R-C-005 不改 ngfw-install |
| case-execution -> device-connection | 执行引擎不直接连接设备，连接由 op_mapper -> ptm-atomic 承载 | 职责边界 |
| case-execution -> traffic-skill/trex-traffic (代码级) | 只读 topology yaml 数据，不 import traffic-skill 代码 | R-C-005 不改 traffic-skill |
| op_mapper -> case-execution | 映射层不依赖编排层 | 避免循环 |
| install.py -> ptm-atomic | install.py 不直接调用 ptm-atomic | 职责边界 |

## 4. 跨仓库依赖

| 依赖方（ptm-team canonical） | 被依赖方（ptm-te workspace） | 依赖类型 | 安装机制 | 说明 |
|---|---|---|---|---|
| skills/case-execution/ | workspace .claude/skills/case-execution/ | 安装 | install.py 回填 | SGA-02=A |
| skills/policy-route-execution/ | workspace .claude/skills/policy-route-execution/ | 安装 | install.py 回填（既有） | 既有 |
| skills/device-management/ | workspace .claude/skills/device-management/ | 安装 | install.py 回填（既有） | 既有 |
| skills/trex-traffic/ | workspace .claude/skills/trex-traffic/ | 安装 | install.py 回填（既有） | 既有 |
| - | workspace cases/三级/四级/五级/*.md | 数据 | 不迁入 ptm-team（R-C-008） | 24 用例 md 留 workspace |
| - | workspace devices.yaml | 数据 | 不迁入 ptm-team | 设备清单留 workspace |
| - | workspace runs/<run-id>/ | 数据 | 不迁入 ptm-team | 执行结果留 workspace |
| - | workspace exec_v4.py | 废弃 | 迁移后加废弃标记 | RA-009 |

## 5. 循环依赖检查

```
case_runner.py -> op_mapper.py -> ptm-atomic (subprocess) -> DUT/TG (HTTP)
                                                                    |
                                                                    v
                                                              case_runner.py (envelope 回流)
```

- case_runner -> op_mapper：单向 import，无循环
- op_mapper -> ptm-atomic：subprocess 调用，无代码级依赖
- ptm-atomic -> DUT/TG：HTTP 调用，结果以 envelope 返回
- envelope 回流到 case_runner：数据流，非代码依赖

**结论**：无循环依赖。

## 6. 文件所有权矩阵

| 文件 | 写入方（CR-033） | 读取方 | 冲突风险 |
|---|---|---|---|
| skills/case-execution/SKILL.md | meta-dev（ST-EX-04） | install.py / 用户 | 新建，无冲突 |
| skills/case-execution/scripts/case_runner.py | meta-dev（ST-EX-04..12） | install.py / 用户 | 新建，无冲突 |
| skills/policy-route-execution/scripts/op_mapper.py | meta-dev（ST-EX-03,07） | case_runner / CLI | 既有文件，新增 fw_logout + TREX_API_URL 注入 + resolve_env_refs（env_topology 参数） |
| skills/device-management/SKILL.md | meta-dev（ST-EX-01） | 用户 | 既有文件，新增 TG 流程段 |
| skills/device-management/reference/device-reference.md | meta-dev（ST-EX-01） | 用户 | 既有文件，新增 TG 6 组合 |
| skills/device-management/templates/devices.yaml.example | meta-dev（ST-EX-01） | 用户 | 既有文件，新增 tg 块示例 |
| script/ptm_team/install.py | meta-dev（ST-EX-02） | install.py CLI | 既有文件，新增规则 + case-execution 安装 |
| workspace cases/三级/四级/五级/*.md | meta-dev（ST-EX-13） | case_runner | 24 用例迁移 + ${ENV.*} 改写，旧 cases/upload/ 保留废弃标记 |
| workspace exec_v4.py | meta-dev（ST-EX-13） | - | 加废弃标记，不删除 |

---
cr_id: "CR-033"
story_id: "STORY-EX-01"
title: "TG 设备建模（devices.yaml tg 块 + 6 组合 + SKILL.md）"
wave: 1
priority: "P0"
lld_policy: "technical-note"
feature_design_refs: []
depends_on: []
dependency_type: "none"
status: "lld-ready"
design_evidence_type: "technical-note"
lld_policy_required_level: "technical-note"
source_hld: "docs/design/HLD-PTM-TE-EXEC.md"
source_adr: "docs/design/ARCHITECTURE-DECISION-PTM-TE-EXEC.md"
source_requirements: "docs/product/REQUIREMENTS-PTM-TE-EXEC.md"
created_at: "2026-07-28T16:00:00+08:00"
author: "meta-dev"
---

# ST-EX-01 TG 设备建模（devices.yaml tg 块 + 6 组合 + SKILL.md）

## 设计证据类型

technical-note（FE-EX-01 waived，无 Feature 级 DESIGN.md，Story 内技术说明承载设计证据）。

## 设计依据

| 依据类型 | 引用 | 关联点 |
|---|---|---|
| HLD | §8 场景 1（TG 设备建模 6 组合全覆盖） | TG 块字段 + 6 组合覆盖流程 |
| HLD | §10 模块表（device-management SKILL.md / device-reference.md / devices.yaml.example） | 修改对象与职责 |
| HLD | §3 蓝图承接（AGA-02=A，用户已确认） | devices.yaml 元数据 + topology yaml 接口 |
| ADR | ADR-08（TG 设备数据归属，用户已确认 A） | devices.yaml 只存 TG 元数据，接口拓扑留 topology yaml |
| 需求 | R-F-001（TG 类型设备建模）/ R-F-002（6 组合）/ R-F-003（device-reference.md 对照） | 成功标准 SM-EX-01 |
| 开发计划 | DEVELOPMENT-PLAN.yaml Wave 1 ST-EX-01 | file_ownership + depends_on |

## 文件影响范围

| 文件 | 归属段 | 变更类型 | 说明 |
|---|---|---|---|
| `skills/device-management/SKILL.md` | TG 流程段 | 新增章节 | 在现有 TGFW 防火墙流程后新增 TG 测试仪设备管理流程 |
| `skills/device-management/reference/device-reference.md` | TG 6 组合 | 新增表格 | 新增 TG 测试仪 6 组合型号对照表 |
| `skills/device-management/templates/devices.yaml.example` | tg 块 | 新增示例 | 新增 TG 设备块示例（含 type/host/sub_type/hardware_platform/ssh/api_server） |
| `skills/device-management/templates/.env.example` | TG 环境变量 | 新增变量 | 新增 `TG_SSH_PASSWORD` 环境变量声明 |

**file_ownership 声明**（与 DEVELOPMENT-PLAN.yaml 一致）：
- `skills/device-management/SKILL.md#TG 流程段`
- `skills/device-management/reference/device-reference.md#TG 6 组合`
- `skills/device-management/templates/devices.yaml.example#tg 块`

**不触碰文件**：`skills/policy-route-execution/scripts/op_mapper.py`（ST-EX-03 拥有）；`skills/traffic-skill/configs/*.yaml`（R-C-005 不改，case_runner 只读）；`script/ptm_team/install.py`（ST-EX-02 拥有）。

## 接口设计要点

### devices.yaml tg 块 schema（ADR-08 方案 A）

devices.yaml 只存 TG 元数据，**不含接口拓扑**（接口拓扑留 traffic-skill topology yaml，case_runner 经 `--topology-yaml` / `--env-file` 读取）。

```yaml
device_groups:
  - name: <TG 设备名>
    description: "<TG 设备描述>"

    tg:                          # TG 测试仪设备块（与 firewall 块对称）
      host: <管理IP>             # TG 主机管理 IP
      type: TG                   # 固定值 TG，区分于 firewall
      sub_type: <ixia-c|trex>    # TG 子类型
      hardware_platform: <EP|C236|J1900>  # 硬件平台
      serial_url: <串口URL>       # 串口管理地址（可选）
      api_server: <IP:端口>      # trex-api 服务地址（TREX_API_URL 来源 fallback，ADR-05）
      ssh:
        host: <SSH_IP>
        port: 22
        user: root
        password: ${TG_SSH_PASSWORD}
        enabled: false

    tags:
      - tg
      - <sub_type>
      - <hardware_platform>
```

**字段约束**：
- `type: TG` 固定值，case_runner 据此识别 TG 设备块
- `sub_type` 枚举 `ixia-c` / `trex`（2 值）
- `hardware_platform` 枚举 `EP` / `C236` / `J1900`（3 值）
- `api_server` 格式 `IP:端口`（如 `10.113.52.253:8450`），op_mapper `_build_exec_env` 注入为 `TREX_API_URL` 环境变量（ST-EX-03 实现）
- `password` 必须 `${ENV_VAR}` 占位（与 firewall 块凭据安全规则一致）

### TG 6 组合型号对照表

| sub_type | hardware_platform | 组合编号 | 说明 |
|---|---|---|---|
| ixia-c | EP | 1 | ixia-c on EP 平台 |
| ixia-c | C236 | 2 | ixia-c on C236 平台 |
| ixia-c | J1900 | 3 | ixia-c on J1900 平台 |
| trex | EP | 4 | trex on EP 平台 |
| trex | C236 | 5 | trex on C236 平台 |
| trex | J1900 | 6 | trex on J1900 平台 |

**覆盖校验**：2 sub_type × 3 hardware_platform = 6 组合（SM-EX-01）。

### SKILL.md TG 流程段

在现有"添加设备流程"后新增"添加 TG 测试仪设备"子流程，与 firewall 流程对称：
1. 从用户输入提取：设备名、sub_type、hardware_platform、管理 IP、api_server
2. 查阅 device-reference.md TG 6 组合表确认 sub_type × hardware_platform 合法
3. 按 devices.yaml.example tg 块模板写入新条目
4. 凭据 `${TG_SSH_PASSWORD}` 占位
5. 验证 YAML 语法

## 实施步骤

1. **device-reference.md 新增 TG 6 组合表**：在现有 TGFW 型号对照表后新增"TG 测试仪型号对照表"章节，含 6 行（2 sub_type × 3 hardware_platform）
2. **devices.yaml.example 新增 tg 块示例**：在现有 firewall 块示例后新增 TG 设备块示例，含全部 schema 字段
3. **.env.example 新增 TG_SSH_PASSWORD**：在现有环境变量清单后新增 `TG_SSH_PASSWORD` 行
4. **SKILL.md 新增 TG 流程段**：在"添加设备流程"章节后新增"添加 TG 测试仪设备"子章节，含流程步骤 + devices.yaml 写入模板（tg 块）
5. **SKILL.md 凭据安全章节扩展**：环境变量约定表新增 `TG_SSH_PASSWORD` 行
6. **SKILL.md 职责边界补充**：明确 TG 设备的接口拓扑不存 devices.yaml，由 traffic-skill topology yaml 承载（ADR-08）

## 回滚策略

- 本 Story 为纯文档/模板增量变更，无代码逻辑修改，无运行时副作用
- 回滚方式：git revert 相关 commit 即可恢复
- 不影响既有 firewall 设备管理流程（增量新增，不修改现有段落）

## 测试要点

| 测试项 | 验证方法 | 预期结果 |
|---|---|---|
| TG 6 组合全覆盖 | 人工审查 device-reference.md TG 表行数 | 6 行（2×3），SM-EX-01 |
| tg 块 YAML 语法 | `python -c "import yaml; yaml.safe_load(open('devices.yaml.example'))"` | 无异常 |
| SKILL.md 流程完整性 | 人工审查 TG 流程段含查表/写入/凭据/验证步骤 | 5 步齐全 |
| 凭据占位合规 | grep tg 块 password 字段 | `${TG_SSH_PASSWORD}` 占位，无明文 |
| api_server 字段存在 | grep tg 块 api_server | 字段存在且格式 `IP:端口` |
| 与 firewall 块对称 | 对比 tg 块与 firewall 块结构 | 块名不同（tg vs firewall），字段风格一致 |


## 权限与风险

- **权限**：NO_CREDENTIAL_READ（不读凭据）/ NO_PRODUCTION_WRITE（dry-run 默认门，不写生产）/ NO_EXTERNAL_PUBLISH（不发布）
- **失败行为**：devices.yaml tg 块格式错误或字段缺失 -> 解析失败返回错误，不静默继续
- **风险**：tg 块配置不完整导致 6 组合型号无法解析，TG 操作取址失败
## 开放项

无。ADR-08 用户已确认 A（2026-07-28T12:30:00+08:00），无阻塞问题。

## 与 HLD/ADR 追溯一致性

| HLD/ADR 条目 | 本 Story 落实点 |
|---|---|
| HLD §8 场景 1（6 组合全覆盖） | device-reference.md TG 6 组合表 |
| HLD §10 模块表（device-management 3 文件变更） | SKILL.md + device-reference.md + devices.yaml.example |
| HLD §3 AGA-02=A（devices.yaml 元数据 + topology yaml 接口） | tg 块只存元数据，不含接口拓扑 |
| ADR-08（TG 数据归属，用户确认 A） | devices.yaml tg 块 schema 不含 interfaces |
| SM-EX-01（6 组合） | device-reference.md 6 行 |
| R-F-001/002/003 | TG 类型建模 + 6 组合 + 对照表 |

## implementation_context（CP6 实现执行证据摘要）

**evidence_type**: technical-note（低风险纯文档/模板增量，无代码逻辑，无运行时副作用）

### 实现对象清单

| 文件 | 变更 | 验证 |
|---|---|---|
| `skills/device-management/reference/device-reference.md` | 新增"TG 测试仪型号对照表（6 组合）"章节（2 sub_type × 3 hardware_platform = 6 行）+ 字段约束 + 查表流程 | 6 组合全覆盖（SM-EX-01） |
| `skills/device-management/templates/devices.yaml.example` | 新增示例 3：tg 块（type/host/sub_type/hardware_platform/serial_url/api_server/ssh/tags 全字段） | `yaml.safe_load` OK，3 device_groups，tg 块 password=${TG_SSH_PASSWORD} 占位 |
| `skills/device-management/templates/.env.example` | 新增 `TG_SSH_PASSWORD` 环境变量声明 | 4 个环境变量（FW_WEB/SSH/TELNET + TG_SSH） |
| `skills/device-management/SKILL.md` | 新增"添加 TG 测试仪设备"子流程（5 步）+ tg 块写入模板 + 凭据安全环境变量表新增 TG_SSH_PASSWORD + 职责边界补充 ADR-08（接口拓扑不存 devices.yaml） | 流程 5 步齐全，字段约束完整 |

### 设计契约映射

| LLD 契约 | 实现 | 一致性 |
|---|---|---|
| devices.yaml tg 块 schema（ADR-08 方案 A） | tg 块含 type/host/sub_type/hardware_platform/serial_url/api_server/ssh，不含 interfaces | 一致 |
| TG 6 组合型号对照表 | 2 sub_type × 3 hardware_platform = 6 行 | 一致（SM-EX-01） |
| api_server 格式 IP:端口 | `10.113.52.253:8450` | 一致 |
| password ${TG_SSH_PASSWORD} 占位 | tg 块 ssh.password | 一致 |
| SKILL.md TG 流程段 5 步 | 查表/写入/凭据/验证 + 字段约束 | 一致 |

### 本地验证

| 验证项 | 命令 | 结果 |
|---|---|---|
| devices.yaml.example YAML 语法 | `python3 -c "import yaml; yaml.safe_load(open(...))"` | OK，3 device_groups，tg 块字段完整 |
| TG 6 组合全覆盖 | device-reference.md TG 表行数 | 6 行（SM-EX-01） |
| 凭据占位合规 | tg 块 password 字段 | `${TG_SSH_PASSWORD}`，无明文 |
| api_server 字段 | tg 块 | `10.113.52.253:8450`（IP:端口） |
| 与 firewall 块对称 | 结构对比 | 块名 tg vs firewall，字段风格一致 |

### 平台差异

- 纯文档/模板增量，无代码逻辑，无平台差异
- install.py 安装 device-management skill 时，4 个文件同步到 workspace（既有安装逻辑不变）

### Agent Dispatch Evidence

inline-fallback（主进程直接执行），technical-note 低风险无需 subagent 调度。

### 回滚策略

git revert 即可恢复（纯文档增量，无运行时副作用，不影响既有 firewall 设备管理流程）。

---
change_id: CR-024-ptm-te-agent
workflow_id: WF-PTM-TEAM-20260520-001
created_at: "2026-07-10T00:00:00+08:00"
created_by: host-orchestrator（主进程）
status: "closed"
implementation_status: draft
workflow_mode: standard
engagement_mode: production
impact_level: high
complexity: standard
rollback_to: story-planning
approval_source: user-request
depends_on:
  - ptm-atomic CLI（外部依赖，已安装 0.1.0，非阻塞）
  - ptm-tde 已交付（PC 产出契约上游，CR-010~017 closed）
plan_ref: /home/hyde/.claude/plans/ptm-te-agent-impl.md（v4）
cp0_intake: "2026-07-10 (CP0 auto PASS)"
cp2_approved: "2026-07-10 (CP2 approved, DQ-01~04 all accepted)"
cp3_approved: "2026-07-10 (CP3 approved, HLD v1.1 + 5 ADR + inline-fallback)"
cp5_approved: "2026-07-10 (CP5 approved, 4 LLD + inline-fallback)"
cp6: "2026-07-10 (CP6 PASS, 4 Story 编码 + 三平台 dry-run)"
cp7: "2026-07-10 (CP7 PASS_WITH_RISK, static 12/12, runtime N/A)"
cp8_approved: "2026-07-10 (CP8 approved, READY_WITH_RISK, runtime follow-up)"
closed_at: "2026-07-10T00:00:00+08:00"
cross_references:
  - CR-016-atomic-ops-consumption-gap（ptm-tde 侧 atomic-ops 消费契约，ptm-te 是执行侧对端）
  - CR-019-pc-step-name-atomic-op-contract（PC case_steps[].atomic_op.op_id 契约，ptm-te 直接消费）
lifecycle_status: "closed"
gate_status: "cp8-approved"
---

# CR-024 - ptm-te 测试执行工程师 Agent 实现

## 变更请求摘要

实现 `ptm-te`（测试执行工程师）agent，前期具备两个能力：

1. **设备管理能力**（精简核心）：设备清单管理（`devices.yaml`）+ SSH/Telnet 双轨连接封装（含回退）+ 系统快照采集
2. **策略路由用例执行能力**：基于 `ptm-atomic` CLI 消费 `fw_*_policy_route` 原子操作（嵌套子命令 `policy-route config/update/delete/verify/...`），执行 ptm-tde 产出的策略路由物理用例（PC）

ptm-te 是 **ptm-tde 的下游执行器**，采用与 ptm-tde 一致的**编排器模式**。参考项目：`/home/hyde/projects/manaul`（防火墙测试助手）。

实施走 ptm-team Meta Flow 正式 CR + CP0-CP8。完整实施计划见 `plan_ref`（v4，已通过 4 轮评审）。

## 背景与目标

### 项目架构定位

ptm-team 六 Agent 角色中，ptm-te 当前是 `planned` 占位（`agents/ptm-te.md` 50 行）。本 CR 将其升格为 `active` 编排器，填补"测试设计 -> 测试执行"链路缺口：

```
ptm-tde（已交付，产出 PC：ppdcs/delivery/<特性名>特性测试用例.md，含 case_steps[].atomic_op）
  ↓ 消费结构化 case_steps
ptm-te（本 CR）：用例解析 -> 设备准备 -> login -> 逐条原子操作执行 -> 结果判定 -> 用例清理 -> 结果回写
```

### 核心设计要点（来自计划 v4）

1. **ptm-atomic CLI 真相**：命令名 `ptm-atomic`（非 `atomic-ops`）；`run` 是嵌套子命令 `ptm-atomic run --base-url <url> <family> <action>`；16 个操作族中 ptm-te 仅用 `auth` + `policy-route`；扁平格式 `ptm-atomic run <op_id>` 硬报错不兼容，**必须用嵌套子命令**。
2. **op_id -> CLI 双层映射**（`op_mapper.py` 必需，非可选）：
   - 第一层：op_id -> 子命令（如 `fw_config_policy_route` -> `policy-route config`），映射在 `run_policy_route.py` 硬编码
   - 第二层：args -> CLI flag（如 `src_addr` -> `--source-network`），三层命名不一致（ptm-tde PC args ≠ op yaml params ≠ CLI flag），**CP3 必须锁定**
3. **login-once-reuse-session**：`auth login` 一次持久化 `--session-file`，后续 op 复用；遇 `STATE_INVALID` 重新 login。
4. **inverse_op 回滚**：用例清理用 config 的 `inverse_op:fw_delete_policy_route`；`irreversible` 类（如 `fw_reset_policy_route_hitcount`）不接受回滚，由用例设计承担。
5. **op 契约**：`ptm-atomic list` 真实字段是 `side_effect` + `rollback`（非 `rollback_strategy`）；rollback 取值 `inverse_op:<op_id>`/`restore_snapshot`/`irreversible`/`observation`/空。session 由 `--session-file` 自动管理，**ptm-te 无需自铸 idempotency_key/state_ref/session_ref**。
6. **设备管理拆分**：device-management（元数据：devices.yaml + 型号映射）+ device-connection（连接逻辑：SSH/Telnet + 快照），连接逻辑不在元数据 SKILL 内。
7. **checkpoint-manager 不复用**：是 ptm-tde 三阶段专属门控，ptm-te 执行门控独立写进 `agents/ptm-te.md`。

## 调研结论（已实测验证，详见 plan_ref §3）

- `ptm-atomic` CLI 已安装（`/home/hyde/.local/bin/ptm-atomic`，0.1.0），`--help`/`list`/`run` 全部可用。前置仅 `ptm-atomic sync` 刷新缓存 + 可选 `uv tool install . --force` 升级。**非阻塞**。
- ptm-atomic 仓库（`/home/hyde/projects/ptm-atomic`）已存在全部所需 op + `adapters/ngfw/default.yaml` + `packages/ngfw_*.yaml`。op 无需新建。
- manaul `device-management` SKILL 只做元数据，可直接迁移；连接逻辑在 `scripts/collect_sysinfo.py`，需单独迁移适配。
- manaul `fw-config-policy-route` SKILL.md 命令格式过时（`atomic-ops run <op_id>`），不可直接迁移，仅复用参数表/错误表/Gotchas 内容。
- ptm-tde PC 消费契约（CR-019）：`case_steps[].step_name`、`case_steps[].atomic_op.op_id`、`case_steps[].atomic_op.args`、`case_steps[].expected_result`，输入路径 `ppdcs/delivery/<特性名>特性测试用例.md`。

## 五维度影响分析

### 1. 需求影响

| 维度 | 影响 |
|------|------|
| 现有需求 | 不变。ptm-tde/tm/tse/tae/qa 角色定位不变 |
| 新增需求 | ptm-te agent 升格 active；新增设备管理 + 策略路由执行两个能力 |
| 需求冲突 | 无。ptm-te 是 ptm-tde 下游，消费其 PC 产出，不改 ptm-tde 契约 |
| 场景主体 | `scenario_subject_type=target-artifact`（ptm-te agent 本身是交付对象，非当前仓库工作流） |

### 2. 设计影响

| 维度 | 影响 |
|------|------|
| 架构决策 | ptm-te 编排器模式（与 ptm-tde 一致）；op_mapper.py 承载双层映射；login-once-reuse-session；inverse_op 回滚 |
| HLD 影响 | 新增 ptm-te HLD（本 CR 产出），锁定 op_id/args 三层映射、skill 边界、连接模型、凭据管理 |
| ADR 影响 | 新增 ADR：三层命名映射策略（ptm-tde args vs op yaml params vs CLI flag）；runtime 授权 dry-run 默认门；irreversible 类步骤回滚豁免 |
| 蓝图影响 | `docs/ptm-team-blueprint.md` 路标回写：ptm-te 状态 planned -> active |

### 3. Story/实现影响

| 维度 | 影响 |
|------|------|
| 受影响文件 | 新建 9 个（1 agent + 3 skill 目录 + op_mapper.py + ssh_exec.py + collect_sysinfo.py + devices.yaml.example + device-reference.md），修改 2 个（install.py + skills/README.md） |
| Story 拆解 | 4 Story（S1-S4），3 Wave |
| 实现复杂度 | 中。S3 的 op_mapper.py 双层映射是核心复杂点（覆盖 7 个 op 的 flag 映射）；S2 连接逻辑迁移适配；S1/S4 相对直接 |

### 4. 安全/权限影响

| 维度 | 影响 |
|------|------|
| 凭据管理 | `devices.yaml` 不入库明文凭据（`${ENV_VAR}` 占位）；Web 密码经 `--password-env FW_WEB_PASSWORD` 传入；提供 `.env.example`；用户项目 `devices.yaml` 放工作区根目录且 `.gitignore` 忽略 |
| 运行时授权 | runtime 写操作（`--execute`）作为独立 `runtime_authorization` 决策项；首期默认 dry-run（决策 #1） |
| 设备访问 | 真实设备 SSH/Telnet/Web 访问，仅限验证设备 hg3250-51（决策 #2） |
| 风险 | 中。涉及真实设备策略变更（`--execute` 时）+ 凭据管理，已通过 dry-run 默认门 + 凭据占位缓解 |

### 5. 交付影响

| 维度 | 影响 |
|------|------|
| 安装器 | `install.py` 新增 ptm-te 投影：`AGENT_ALIASES` 加 te/ptm-te；`get_agent_skills` 加 ptm-te 分支返回 3 个 skill；v1 不注入 rule block（决策 #3） |
| 文档 | `skills/README.md` 新增 ptm-te skill 索引；`docs/ptm-team-blueprint.md` 路标回写 |
| 向后兼容 | 完全兼容。ptm-te 是新增 agent，不影响已交付的 ptm-tde 安装与运行 |
| 平台投影 | claude->`.claude/agents`+`.claude/skills`；codex->`.codex/agents`+`.agents/skills`；qoder->`.qoder/agents`+`.qoder/skills`（复用现有 install.py 机制） |

## 结构化影响面

| 字段 | 值 |
|------|------|
| impact_capability_refs | `ptm-te.device-management`, `ptm-te.device-connection`, `ptm-te.policy-route-execution` |
| impact_feature_refs | `ptm-te-agent`（新增 agent 能力） |
| impact_module_paths | `agents/ptm-te.md`, `skills/device-management/`, `skills/device-connection/`, `skills/policy-route-execution/`, `script/ptm_team/install.py`, `skills/README.md` |
| impact_policy_refs | `docs/ptm-team-blueprint.md`（路标回写） |
| impact_process_refs | 本 CR + S1-S4 Story 卡片/LLD |
| impact_runtime_refs | `ptm-atomic` CLI（外部，已就位）, hg3250-51 设备（验证用） |
| impact_data_refs | `devices.yaml`（用户工作区，不入库）, `ppdcs/delivery/*.md`（ptm-tde 产出，ptm-te 消费） |

## 修改文件清单

### 新建

| 优先级 | 文件 | 变更范围 | Story |
|--------|------|---------|-------|
| P0 | `agents/ptm-te.md` | 重写 50 行 planned 占位为 active 编排器（color: green，编排流程，PC 消费契约，三层映射，login-once-reuse，inverse_op 清理，执行门控） | S1 |
| P0 | `skills/device-management/SKILL.md` | 元数据 SKILL（设备清单 + 型号映射查表） | S2 |
| P0 | `skills/device-management/reference/device-reference.md` | 型号映射参考 | S2 |
| P0 | `skills/device-management/templates/devices.yaml.example` | 设备清单模板（${ENV_VAR} 占位） | S2 |
| P0 | `skills/device-connection/SKILL.md` | SSH/Telnet 双轨连接 + 回退 + 快照采集 | S2 |
| P0 | `skills/device-connection/scripts/ssh_exec.py` | paramiko SSH 执行封装 | S2 |
| P0 | `skills/device-connection/scripts/collect_sysinfo.py` | 系统快照采集（迁移自 manaul） | S2 |
| P0 | `skills/policy-route-execution/SKILL.md` | op_id->子命令 + args->flag 映射 + 干跑/执行/verify + inverse_op 清理 + 错误表 + Gotchas | S3 |
| P0 | `skills/policy-route-execution/scripts/op_mapper.py` | 双层映射（必需，覆盖 7 个 op） | S3 |

### 修改

| 优先级 | 文件 | 变更范围 | Story |
|--------|------|---------|-------|
| P1 | `script/ptm_team/install.py` | AGENT_ALIASES 加 te/ptm-te；get_agent_skills 加 ptm-te 分支返回 3 skill；v1 不注入 rule block | S4 |
| P1 | `skills/README.md` | 新增 ptm-te 3 个 skill 索引 | S4 |
| P2 | `docs/ptm-team-blueprint.md` | ptm-te 路标回写 planned -> active | S4 |

### 文档处理决策

| 受影响对象 | 处理方式 | 旧基线保留方式 |
|-----------|---------|--------------|
| `agents/ptm-te.md` | 整体重写 | 旧 50 行 planned 占位保留在 git 历史；frontmatter 升格 status: active |
| `skills/device-management/` 等 3 个 skill | 新建 | 不适用 |
| `script/ptm_team/install.py` | 原文档增量更新 | 保留现有 AGENT_ALIASES/get_agent_skills 结构，追加 ptm-te 分支 |
| `skills/README.md` | 原文档增量更新 | 追加 ptm-te skill 索引段 |
| `docs/ptm-team-blueprint.md` | 原文档增量更新 | ptm-te 状态行回写，追加 `## 修订记录` |

## 复杂度判定

**判定：standard**。

不满足 fast-lane 条件（命中多个升级条件）：
- 4 Story / 3 Wave，存在 W2 并行依赖（S2+S3）和 W3 串行依赖（S4 依赖 S1/S2/S3）
- 新增 agent + 3 个 skill + 安装器集成（平台安装）
- 涉及外部接口（ptm-atomic CLI）+ 真实设备访问（SSH/Telnet/Web）
- 涉及权限/安全边界（凭据管理 + runtime 写授权）
- 核心复杂点 op_mapper.py 双层映射覆盖 7 个 op

## 文件所有权冲突分析

| 现有 CR | 状态 | 与 CR-024 重叠文件 | 冲突处理 |
|---------|------|-------------------|---------|
| CR-018 | closed | 无（CR-018 是 ptm-tde 合规治理，不改 ptm-te） | 无冲突 |
| CR-023 | implemented（fast-lane） | `AGENTS.md`/`CLAUDE.md`（CR-023 标注投影边界，CR-024 不改规则文件） | 无冲突；决策 #3 v1 不注入 rule block，不动 AGENTS.md/CLAUDE.md |
| ptm-tde 已交付基线 | delivered | `script/ptm_team/install.py`（共享安装器） | 追加 ptm-te 分支，不改 ptm-tde 既有逻辑；`skills/README.md` 同理追加 |

**结论**：无文件所有权冲突。`install.py` 和 `skills/README.md` 是追加式修改，不破坏 ptm-tde 既有投影。

## Story 拆分与 Wave

| Story | 内容 | 依赖 | Wave | tier | lld_policy |
|-------|------|------|:--:|:--:|------|
| **S1** | ptm-te agent 骨架与编排流程（重写 agents/ptm-te.md：active 编排器，color: green，执行流程，PC 消费契约，op_id/args 三层映射，login-once-reuse，inverse_op 清理，运行时工作目录，执行门控） | 无 | W1 | S | full-lld |
| **S2** | device-management + device-connection skill（元数据 SKILL + reference + devices.yaml.example；连接 SKILL + ssh_exec.py + collect_sysinfo.py，双轨+回退+快照） | S1 | W2 | M | full-lld |
| **S3** | policy-route-execution skill（SKILL.md 按真实 CLI 重写，op_id->子命令 + args->flag 映射覆盖 7 op，干跑/执行/verify，inverse_op 清理，错误表，Gotchas）+ op_mapper.py（必需）；前置 ptm-atomic sync 验证 | S1 | W2 | M | full-lld |
| **S4** | 安装器集成与验证（AGENT_ALIASES + get_agent_skills 加 ptm-te 分支返回 3 skill；v1 不注入 rule block；skills/README.md 更新；blueprint 路标回写；安装 dry-run + 真实安装验证） | S1,S2,S3 | W3 | S | technical-note |

- **W1**：S1（agent 骨架是基础，定义所有下游 skill 的消费契约）
- **W2**：S2、S3 并行（两个独立 skill，无文件所有权冲突）
- **W3**：S4（依赖前三个 Story 产物）

> S4 是安装器集成 + 文档回写，复杂度低，使用 technical-note；S1/S2/S3 涉及编排逻辑/连接实现/映射核心，使用 full-lld。

## Decision Brief（CR intake）

### 审批者摘要

本次确认服务目标：**批准启动 CR-024（standard 模式，4 Story / 3 Wave）实现 ptm-te agent**。

- **推荐动作**：`approve`--批准启动 CR-024，进入 CP1 场景完备 -> CP2 需求基线。
- **approve 后会发生什么**：Host Orchestrator 自动推进 CP1（场景完备性检查）、CP2 自动预检，然后在 CP2 人工门禁停下等你确认 4 项决策。
- **approve 不授权什么**：不授权真实设备 `--execute` 写操作（决策 #1 单独确认）；不授权修改 ptm-tde 已交付基线；不授权注入 AGENTS.md/CLAUDE.md rule block（决策 #3）。
- **不确认会阻塞什么**：ptm-te 实现无法启动，CP1/CP2 无法推进。

### 决策分层

| 层级 | 决策项 |
|------|--------|
| 必须用户决策 | CR024-DQ-01（runtime 授权）、CR024-DQ-02（设备范围）、CR024-DQ-04（用例来源） |
| 高风险策略确认 | CR024-DQ-03（rule block 注入策略） |
| agent 默认处理 | Story tier/lld_policy 分配、Wave 顺序、文件清单细化（CP3/CP4 锁定） |
| 仅审计记录 | ptm-atomic CLI 已安装的非阻塞前置、checkpoint-manager 不复用 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 | 回退条件 |
|---|---|---|---|---|---|---|---|
| CR024-DQ-01 | runtime_authorization | runtime 写操作授权边界 | **dry-run 默认门**：首期 CP7 默认 `--dry-run`；`--execute` 写操作作为独立 runtime_authorization 决策项，需用户单次确认 | A: 完全放开 --execute（无人工确认）/ B: 全 dry-run（--execute 留 v2） | dry-run 已验证参数路由和 session 有效性；--execute 涉及真实设备策略变更需单次确认 | 选 A: 设备被意外修改不可追溯；选 B: 回滚/清理逻辑无法验证 | dry-run 结果与预期一致且用户确认后，单次升权 --execute |
| CR024-DQ-02 | scope | 验证设备范围 | **hg3250-51**（10.113.55.51，DAS-TGFW-A1300-HU，Web 443） | A: nxp1046-95（10.113.55.95，DAS-TGFW-1900）/ B: 两台都验证 | hg3250-51 是 manaul 已验证策略路由可通设备，风险最低 | 选 A: 不同型号 API 行为可能差异；选 B: CP7 工作量翻倍 | hg3250-51 不可达时 switch 到 nxp1046-95 |
| CR024-DQ-03 | implementation | ptm-te rule block 注入策略 | **v1 不注入**：执行规则写进 agents/ptm-te.md，不写 AGENTS.md/CLAUDE.md managed block | A: 注入 managed block（参考 ptm-tde 的 render_ptm_tde_rule_body） | ptm-te 首期流程简单，不需跨项目工作区隔离规则；ptm-tde 的 rule block 因三阶段框架+多特性隔离+Gate 才需要 | 选 A: 增加安装器复杂度 + managed block 跨项目残留风险；不注入: 跨项目使用靠 agent md 自述 | 用户反馈跨项目行为不一致时，v2 补 managed block |
| CR024-DQ-04 | scope | 用例来源 | **手写最小 PC 优先**：首期手写最小策略路由 PC（1 config + 1 verify + 1 delete）验证全链路；真实消费 ptm-tde PC 留 follow-up | A: 真实消费 ptm-tde 已产出 PC / B: 不消费 PC，纯 CLI 命令验证 | 手写最小 PC 覆盖端到端消费链路但不依赖 ptm-tde 产出质量 | 选 A: 验证阻塞在 ptm-tde 侧 args 命名等未锁定项；选 B: case_steps 消费链路不可验证 | CP3 锁定 ptm-tde args 命名后 PC 质量达标，可切换为真实 PC |

### Decision Collection Coverage

| 来源 | 候选问题数 | 纳入待决策数 | N/A / 缺失原因 |
|------|----------|------------|--------------|
| plan_ref §11 决策建议 v4 | 4 | 4 | 决策 #5（op_id 映射实现）已删除--扁平格式硬报错，映射表是唯一路径，无决策空间 |
| Scenario Gray Areas（CP1 识别） | 4 | 已映射到 DQ-01~04 | runtime 授权/设备范围/用例来源/args 命名约定分别对应 DQ-01/02/04/CP3 锁定 |
| Architecture Gray Areas（CP3 预识别） | 3 | CP3 锁定 | 三层映射策略/连接模型拆分/inverse_op 豁免，CP3 HLD 锁定，不升级为 CP2 决策 |
| **合计** | 11 | 4（CP2）+ 3（CP3） | - |

### 不授权项

- 不授权真实设备 `--execute` 写操作（决策 #1 dry-run 默认门，--execute 单独确认）
- 不授权修改 ptm-tde 已交付基线（agents/ptm-tde.md、skills/* 既有 skill、install.py 既有 ptm-tde 分支）
- 不授权注入 AGENTS.md / CLAUDE.md managed rule block（决策 #3 v1 不注入）
- 不授权修改 ptm-atomic 仓库（外部依赖，仅消费）
- 不授权 devices.yaml 入库明文凭据（必须 ${ENV_VAR} 占位）

## 风险与回退

| 风险 | 等级 | 缓解 | 回退路径 |
|------|------|------|---------|
| op_id/args 三层映射与 run_policy_route.py/op yaml/ptm-tde args 漂移 | 高 | 映射表来源锁定三处真相源；static 校验一致性；CP3 锁定 ptm-tde args 命名 | op_mapper.py 映射表 centralize，漂移时单点修正 |
| login session 失效（STATE_INVALID） | 中 | 执行流检测 STATE_INVALID 自动重新 auth login | session 重建 |
| inverse_op 回滚缺失致策略路由残留 | 中 | 用例清理强制 policy-route delete；restore_snapshot 按快照恢复；irreversible 类 SKILL 注明由用例设计承担 | 手动 Web 清理 |
| ePolicyRouteInIfModeError（入接口非路由模式） | 低 | SKILL 前置校验接口模式；错误表明示需人工 Web 改 | 人工 Web 修正接口模式 |
| 真实设备不可达 | 中 | 降级 dry-run-only，runtime 留 follow-up | CP7 runtime 转 follow-up candidate |
| telnetlib 在 Python 3.13 移除 | 低 | 脚本声明 >=3.9,<3.13 | 改用 telnetlib3 |
| 凭据泄露 | 高 | devices.yaml 不入库明文，--password-env 传 Web 密码 | 凭据轮换 |
| ptm-te 与 ptm-tde 契约漂移 | 中 | CP3 锁定 PC 字段契约 + 三层映射写入 agent md | 契约对齐 CR |

## 后续 CR 候选

| 编号 | 描述 | 优先级 | 状态 |
|------|------|--------|------|
| T-01 | 真实消费 ptm-tde 产出 PC（决策 #4 回退条件触发） | P1 | candidate |
| T-02 | 1900 型号设备验证（决策 #2 备选 B） | P2 | candidate |
| T-03 | ptm-te managed rule block v2（决策 #3 回退条件触发） | P3 | candidate |
| T-04 | 进程管理 / 串口初始化（设备管理范围扩展） | P3 | candidate |
| T-05 | fw_config_batch_policy_route package 级编排支持 | P3 | candidate |

## 验证方法

- `validation_mode=mixed`（static + runtime）
- **static**：SKILL.md 结构检查、agent frontmatter 校验（含 color: green）、安装器 dry-run、Python 脚本 lint/单元测试、op_id->子命令 + args->flag 映射表与 run_policy_route.py + op yaml 一致性校验、inverse_op 回滚契约校验
- **runtime**（需决策 #1 授权 + 决策 #2 设备）：
  - `ptm-atomic run ... auth login` + `policy-route config` dry-run + execute（hg3250-51）
  - login-once-reuse-session 验证（session.json 复用、STATE_INVALID 重连）
  - 设备 SSH/Telnet 连接 + 系统快照采集
  - PC 用例端到端执行（解析 -> 设备准备 -> login -> op_mapper 映射 -> 执行 -> verify -> inverse_op 清理 -> 日志）
- 产出 `docs/quality/VERIFICATION-REPORT.md` + `TEST-REPORT.md`

## Checkpoint Index

| CP | 类型 | 真相源 | 状态 |
|----|------|--------|------|
| CP0 | auto | `process/checks/CP0-REQUEST-INTAKE-CR-024.md` | PASS（2026-07-10） |
| CP1 | auto | `process/checks/CP1-USE-CASE-COMPLETENESS-CR-024.md` | pending |
| CP2 | auto+manual | `process/checks/CP2-REQUIREMENTS-BASELINE-CR-024.md` + `process/checkpoints/CP2-REQUIREMENTS-BASELINE-CR-024.md` | pending |
| CP3 | auto+manual | `process/checks/CP3-HLD-CONSISTENCY-CR-024.md` + `process/checkpoints/CP3-HLD-REVIEW-CR-024.md` | pending |
| CP4 | auto_precheck | `process/checks/CP4-STORY-DAG-PARALLEL-SAFETY-CR-024.md` | pending |
| CP5 | batch_auto+manual | `process/checks/CP5-STORY-024-*-LLD-IMPLEMENTABILITY.md` + `process/checkpoints/CP5-ALL-STORIES-LLD-BATCH-CR-024.md` | pending |
| CP6 | rolling_auto | `process/checks/CP6-STORY-024-*-CODING-DONE.md` | pending |
| CP7 | rolling_auto | `process/checks/CP7-CR-024-VERIFICATION-DONE.md` | pending |
| CP8 | auto+manual | `process/checks/CP8-DELIVERY-READINESS-CR-024.md` + `process/checkpoints/CP8-DELIVERY-READINESS-CR-024.md` | pending |

## 参考

- 实施计划：`/home/hyde/.claude/plans/ptm-te-agent-impl.md`（v4）
- 关联 CR：CR-016（atomic-ops 消费契约）、CR-019（PC case_steps atomic_op 契约）
- 外部依赖：`/home/hyde/projects/ptm-atomic`（CLI + op + adapter + package）
- 参考项目：`/home/hyde/projects/manaul`（device-management 元数据 + collect_sysinfo.py 连接逻辑）
- 问题工作流：WF-PTM-TEAM-20260520-001

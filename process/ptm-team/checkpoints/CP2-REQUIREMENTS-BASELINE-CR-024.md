---
check_id: CP2-REQUIREMENTS-BASELINE-CR-024
workflow_id: WF-PTM-TEAM-20260520-001
change_id: CR-024-ptm-te-agent
cp: CP2
type: manual_gate
status: approved
gate: CP2
created_at: "2026-07-10T00:00:00+08:00"
checklist_path: process/checkpoints/CP2-REQUIREMENTS-BASELINE-CR-024.md
auto_precheck: process/checks/CP2-REQUIREMENTS-BASELINE-CR-024.md（PASS 11/11）
decision_brief_profile: full
launch_message_profile: compact
pending_decisions: 4
---

# CP2 - 需求/场景/范围基线门 Decision Brief（CR-024）

## 自动预检摘要

| 项 | 结论 |
|---|---|
| CP2 自动预检 | PASS（11/11） |
| 真相源 | `process/checks/CP2-REQUIREMENTS-BASELINE-CR-024.md` |
| Scenario Gray Areas | 4 个已识别（SGQ-01~04），3 个升级为 CP2 决策，1 个 CP3 锁定 |
| SGQ 证据 | 4 条（满足 ≥1 条要求） |
| 待决策项 | 4 项（CR024-DQ-01~04） |
| checklist 路径 | `process/checkpoints/CP2-REQUIREMENTS-BASELINE-CR-024.md` |

## Decision Brief

### 审批者摘要

本次确认服务的整体目标：**批准 CR-024 需求/场景/范围基线，确认 4 项决策，进入 CP3 HLD 设计**。

- **整体目标**：实现 ptm-te 测试执行工程师 agent（设备管理 + 策略路由用例执行），填补 ptm-tde 下游执行链路。
- **推荐动作**：`approve`--接受以下 4 项推荐决策方案，进入 CP3 HLD（锁定 op_id/args 三层映射）。
- **approve 后会发生什么**：Host Orchestrator 自动推进 CP3（委托 meta-se 产出 HLD + 处理 Architecture Gray Areas + CP3 自动预检），在 CP3 人工门禁停下。
- **approve 不授权什么**：不授权真实设备 `--execute` 写操作（DQ-01 dry-run 默认门，--execute 单次确认）；不授权修改 ptm-tde 已交付基线；不授权注入 AGENTS.md/CLAUDE.md rule block（DQ-03 v1 不注入）；不授权 devices.yaml 入库明文凭据。
- **不确认会阻塞什么**：CP3 HLD 无法启动，ptm-te 实现停滞。

如果你回复 `approve`，表示你接受以下 4 项推荐方案（DQ-01 dry-run 默认门 / DQ-02 hg3250-51 / DQ-03 v1 不注入 / DQ-04 手写最小 PC），不表示授权以下 5 项禁止操作（--execute 真实写设备 / 改 ptm-tde 基线 / 注入 rule block / 改 ptm-atomic 仓库 / devices.yaml 入库明文凭据）。

### 决策分层

| 层级 | 决策项 | 说明 |
|------|--------|------|
| 必须用户决策 | CR024-DQ-01（runtime 授权）、CR024-DQ-02（设备范围）、CR024-DQ-04（用例来源） | 涉及运行授权、设备访问、验证链路，必须用户拍板 |
| 高风险策略确认 | CR024-DQ-03（rule block 注入策略） | 影响安装器复杂度与跨项目行为，需用户确认 |
| agent 默认处理 | Story tier/lld_policy 分配（S1/S2/S3 full-lld，S4 technical-note）、Wave 顺序（W1->W2->W3）、文件清单细化 | CP3/CP4 锁定，不升级为用户决策 |
| 仅审计记录 | ptm-atomic CLI 已安装非阻塞前置、checkpoint-manager 不复用、SGQ-04 args 命名 CP3 锁定 | 记录在案，无需用户确认 |

### Context Capsule Summary

| 字段 | 值 |
|------|------|
| capsule 路径 | process/context/CP2-REQUIREMENT-CONTEXT-CR-024.yaml（待 CP2 通过后生成） |
| read_profile | compact |
| 关键输入 | 计划 v4、CR-024、CP1 检查、CP2 讨论日志 |
| 必读 | CR-024 §五维度影响分析 + §Decision Brief |
| 按需读 | plan_ref §3 调研结论、§4 架构设计 |
| 默认读取策略 | capsule-first（compact profile），默认只读 capsule + 必读项，不默认读取完整 HLD/LLD/TEST-MATRIX/diff |
| 全文档读取 | 仅当 capsule 缺失/冲突/字段不足/人工审计/深度评审/用户明确要求时读取完整正式文档，需在 READ-EXPANSION-LEDGER 写明 full_doc_read_reason |
| do_not_read_by_default | process/archive/**、process/discussions/**（讨论日志仅审计用） |

### Decision Collection Coverage

| 来源 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|-----------|---------|----------|------------|--------------|
| plan_ref §11 决策建议 v4 | 已扫描 | 4 | 4（DQ-01~04） | 决策 #5 已删除--映射表唯一路径无决策空间 |
| Scenario Gray Areas（CP1 识别） | 已扫描 | 4 | 3（DQ-01/02/04） | SGQ-04 转 CP3 HLD 锁定（技术实现细节） |
| Architecture Gray Areas（CP3 预识别） | 已扫描 | 3 | 0（CP3 锁定） | 三层映射/连接拆分/inverse_op 豁免，CP3 锁定 |
| **合计** | - | 11 | 4 | 7 项 N/A 有理由 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| CR024-DQ-01 | runtime_authorization | runtime 写操作授权边界 | **dry-run 默认门**：首期 CP7 默认 `--dry-run`；`--execute` 写操作作为独立 runtime_authorization 决策项，需用户单次确认 | A: 完全放开 --execute（无人工确认）/ B: 全 dry-run（--execute 留 v2） | dry-run 已验证参数路由和 session 有效性；--execute 涉及真实设备策略变更需单次确认 | 选 A: 设备被意外修改不可追溯；选 B: 回滚/清理逻辑无法验证 | dry-run 结果与预期一致且用户确认后，单次升权 --execute |
| CR024-DQ-02 | scope | 验证设备范围 | **hg3250-51**（10.113.55.51，DAS-TGFW-A1300-HU，Web 443） | A: nxp1046-95（10.113.55.95，DAS-TGFW-1900）/ B: 两台都验证 | hg3250-51 是 manaul 已验证策略路由可通设备，风险最低 | 选 A: 不同型号 API 行为可能差异；选 B: CP7 工作量翻倍 | hg3250-51 不可达时 switch 到 nxp1046-95 |
| CR024-DQ-03 | implementation | ptm-te rule block 注入策略 | **v1 不注入**：执行规则写进 agents/ptm-te.md，不写 AGENTS.md/CLAUDE.md managed block | A: 注入 managed block（参考 ptm-tde 的 render_ptm_tde_rule_body） | ptm-te 首期流程简单，不需跨项目工作区隔离规则；ptm-tde 的 rule block 因三阶段框架+多特性隔离+Gate 才需要 | 选 A: 增加安装器复杂度 + managed block 跨项目残留风险；不注入: 跨项目使用靠 agent md 自述 | 用户反馈跨项目行为不一致时，v2 补 managed block |
| CR024-DQ-04 | scope | 用例来源 | **手写最小 PC 优先**：首期手写最小策略路由 PC（1 config + 1 verify + 1 delete）验证全链路；真实消费 ptm-tde PC 留 follow-up | A: 真实消费 ptm-tde 已产出 PC / B: 不消费 PC，纯 CLI 命令验证 | 手写最小 PC 覆盖端到端消费链路但不依赖 ptm-tde 产出质量 | 选 A: 验证阻塞在 ptm-tde 侧 args 命名等未锁定项；选 B: case_steps 消费链路不可验证 | CP3 锁定 ptm-tde args 命名后 PC 质量达标，可切换为真实 PC |

用户需决策事项：本次共 4 项决策需用户确认。CR024-DQ-01（runtime_authorization，真实设备 --execute 写操作）为 blocking 决策；CR024-DQ-02（设备范围）、CR024-DQ-03（rule block 注入）、CR024-DQ-04（用例来源）为 high-risk/scope 决策。用户回复 approve 表示接受全部 4 项推荐方案。

Blocking / high-risk 决策：CR024-DQ-01（runtime_authorization，涉及真实设备写操作）为 blocking 决策，必须用户确认。

### 不授权项

- 不授权真实设备 `--execute` 写操作（DQ-01 dry-run 默认门，--execute 单次确认）
- 不授权修改 ptm-tde 已交付基线（agents/ptm-tde.md、skills/* 既有 skill、install.py 既有 ptm-tde 分支）
- 不授权注入 AGENTS.md / CLAUDE.md managed rule block（DQ-03 v1 不注入）
- 不授权修改 ptm-atomic 仓库（外部依赖，仅消费）
- 不授权 devices.yaml 入库明文凭据（必须 ${ENV_VAR} 占位）

### 风险与回退

| 风险 | 等级 | 缓解 | 回退路径 |
|------|------|------|---------|
| 用户调整 DQ-01 授权范围 | 中 | 按决策修订再发布规则重算影响面 | 重新生成 Decision Brief |
| 用户选 DQ-04 备选 A（真实 PC） | 中 | 阻塞于 ptm-tde args 命名未锁定 | CP3 先锁定 args 再切真实 PC |
| 用户要求 DQ-03 注入 rule block | 低 | 增加 S4 工作量 | v1 注入 managed block，参考 ptm-tde 模式 |

## 后续 CR 候选（CP8 分流预登记）

| 编号 | 描述 | 优先级 | 状态 |
|------|------|--------|------|
| T-01 | 真实消费 ptm-tde 产出 PC | P1 | candidate |
| T-02 | 1900 型号设备验证 | P2 | candidate |
| T-03 | ptm-te managed rule block v2 | P3 | candidate |
| T-04 | 进程管理 / 串口初始化 | P3 | candidate |
| T-05 | fw_config_batch_policy_route package 级编排 | P3 | candidate |

## 回复选项

- `approve`：接受全部 4 项推荐方案（DQ-01 dry-run 默认门 / DQ-02 hg3250-51 / DQ-03 v1 不注入 / DQ-04 手写最小 PC），进入 CP3 HLD。
- `修改: <具体修改点>`：调整某项决策（如 `修改: DQ-02 改为两台都验证`）。
- `reject`：不批准当前基线，说明原因。

## 人工审查结果

| 字段 | 值 |
|------|------|
| 审查人 | user |
| 审查时间 | 2026-07-10 |
| 决策结果 | approve（接受全部 4 项推荐方案） |
| 审查意见 | 用户回复 approve，接受 DQ-01 dry-run 默认门 / DQ-02 hg3250-51 / DQ-03 v1 不注入 / DQ-04 手写最小 PC。进入 CP3 HLD。 |

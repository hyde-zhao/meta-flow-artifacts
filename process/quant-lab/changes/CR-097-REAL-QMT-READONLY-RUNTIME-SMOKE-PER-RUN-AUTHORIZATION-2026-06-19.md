---
cr_id: "CR-097"
status: "active-pending-runtime-preflight"
impact_level: "high"
workflow_mode_before: "production"
workflow_mode_after_change: "standard"
fast_lane_upgrade_reason: "真实 QMT / MiniQMT / XtQuant / gateway / runner runtime 属于高风险运行授权，必须 standard gate"
rollback_to: "delivered"
approval_result: "runtime-scope-authorized-by-user-pending-preflight"
created_at: "2026-06-19T10:45:00+08:00"
created_by: "host-orchestrator"
approved_by: "user"
approved_at: "2026-06-19T10:45:00+08:00"
source: "cp8-follow-up"
linked_issue: ""
parent_cr: "CR-092"
source_checkpoint: "process/checkpoints/CP8-CR092-DELIVERY-READINESS.md"
source_decision_id: "DQ-CP8-CR092-04"
follow_up_type: "real-readonly-runtime-smoke-per-run-authorization"
risk_class: "high"
owner: "host-orchestrator"
revisit_condition: "用户提供 QMT 客户端运行入口、账户模式和只读 smoke 执行条件"
acceptance_criteria: "仅在用户逐 run 授权范围内执行真实 QMT 只读 smoke；scope 限定为 health、capabilities、query_positions_readonly；输出脱敏 evidence；forbidden counters 全部为 0"
close_condition: "只读 runtime smoke 完成并产生脱敏 evidence / 或因缺少 QMT 客户端、入口、账户模式、权限或人工协助转 blocked；不得以替代扫描绕过"
cr_index_path: "process/changes/CR-INDEX.yaml"
---

# CR-097 Real QMT Readonly Runtime Smoke Per-run Authorization

## 变更描述

启动 `CR092-FU-02 Real readonly runtime smoke per-run authorization`。用户已明确表示：“我同意授权真实 QMT runtime”。本 CR 将该授权收敛为一个可审计的真实 QMT 只读 runtime smoke 门禁。

本 CR 的目标是验证真实 QMT / MiniQMT / XtQuant / gateway / runner runtime 的只读通路是否可用，范围仅限：

- `health`
- `capabilities`
- `query_positions_readonly`

本 CR 不授权凭据读取、NAS、真实账户原文、下单、撤单、simulation/live、provider/lake/publish 或任何写操作。若实际 `query_positions_readonly` 触及真实账户信息，输出只能是脱敏摘要：数量 bucket、digest、状态和 forbidden counters；不得写出账户号、证券代码、数量、资金、委托、成交或原始日志。

## 用户授权解释

| 字段 | 内容 |
|---|---|
| 授权原文 | “我同意授权真实 QMT runtime。如果需要人工协助，你告诉我。” |
| 授权类型 | `runtime_authorization` |
| 授权范围 | 启动真实 QMT 只读 runtime smoke 门禁，并在完成 preflight 后执行 health / capabilities / query_positions_readonly |
| 不授权范围 | NAS、`.env` / 凭据读取、账户原文、资金 / 持仓明细 / 委托 / 成交原文、submit / cancel / buy / sell、simulation / live、provider fetch、lake write、catalog publish、真实 release |
| 有效期 | 本 CR 当前 run；若中断或改 scope，需重新确认 |
| 当前状态 | 已授权 scope，但实际执行仍等待 runtime preflight 和人工协助信息 |

## CP8 Follow-up 来源

| 字段 | 内容 |
|---|---|
| 父级 CR | `CR-092` |
| 来源检查点 | `process/checkpoints/CP8-CR092-DELIVERY-READINESS.md` |
| 来源决策 ID | `DQ-CP8-CR092-04` |
| follow-up 类型 | real readonly runtime smoke per-run authorization |
| 风险等级 | high |
| owner | host-orchestrator |
| 重访条件 | 用户提供 QMT 客户端运行入口、账户模式和只读 smoke 执行条件 |
| 验收标准 | 真实只读 smoke 或明确 blocked 证据，且 evidence 完全脱敏 |
| 关闭条件 | CP8 完成，或 runtime preflight blocked 后用户确认关闭 / 延后 |

## 启动前冲突预检

| 检查项 | 结论 | 证据 | 说明 |
|---|---|---|---|
| process 路由 | PASS | `meta-flow workspace check` | `process_link_health: ok` |
| 当前 active formal CR | PASS | `meta-flow check cr-tracking --strict-warnings` | active formal CRs: none |
| blocked CR 重叠 | PASS_WITH_RISK | `CR-089` | CR089 仍 blocked；本 CR 不恢复 CR089、不使用 NAS、不复用旧 runtime 路线，单独执行 per-run readonly smoke |
| 候选来源 | PASS | `process/changes/CR-092-FOLLOW-UP-TRACKING-2026-06-18.md` | `CR092-FU-02` 为 candidate |
| 用户 runtime 授权 | PASS_WITH_PRECONDITIONS | 当前对话 | 用户同意授权真实 QMT runtime；实际执行仍需 QMT 客户端、账户模式和只读命令入口 |
| 不授权边界 | PASS | 本 CR | 不授权 NAS、凭据、账户原文、交易写、simulation/live、publish |

## 文档处理决策

| 受影响文档 | 处理方式 | 旧基线保留方式 | 修订记录位置 | 批准状态 |
|---|---|---|---|---|
| `docs/product/USE-CASES.md` | 不变 | 不涉及产品场景变更 | 不适用 | approved |
| `docs/product/REQUIREMENTS.md` | 不变 | 不涉及需求基线变更 | 不适用 | approved |
| `docs/qmt/CR092-REAL-QMT-READONLY-RUNTIME-SMOKE-MANUAL-GUIDE.md` | 后续按需更新 | 保留 CR092 静态 guide；CR097 若执行成功再补 runtime runbook 附录 | 修订记录 | pending |
| `process/changes/CR-092-FOLLOW-UP-TRACKING-2026-06-18.md` | 原文档更新 | 保留 CR092-FU-02 原候选行并更新为 CR097 active | 台账表 | approved |
| `process/changes/CR-INDEX.yaml` | 原文档更新 | 保留旧 CR 索引和历史项；新增 CR097 active | YAML 结构化索引 | approved |
| `process/STATE.md` | 原文档更新 | 保留历史记录；顶层 active_change 切换到 CR097 | history / cr_tracking | approved |

## 五维度影响分析

| 维度 | 评估问题 | 受影响对象 | 结论 | 处理动作 |
|------|----------|-----------|------|---------|
| 需求层 | 是否新增、删除或重定义 REQ-* | `REQUIREMENTS.md` | false | 不改需求基线 |
| 场景层 | 是否改变测试矩阵覆盖范围 | CR092 runtime smoke 场景 | true | 仅激活 CR092-FU-02 只读 runtime smoke 场景，不扩大为交易或 NAS |
| 计划层 | 是否改变 Phase、Wave、Story / 任务依赖 | `process/STATE.md` / CR tracking | true | 新增 CR097 active，进入 runtime preflight |
| 安全层 | 是否引入新的高风险动作或权限要求 | QMT runtime / 账户边界 | true | 必须使用 standard gate，逐 run 授权，脱敏 evidence，forbidden counters |
| 交付层 | 是否需要重新生成交付物或回归子集 | runtime smoke evidence | true | 需要生成 runtime preflight、执行证据或 blocked evidence、CP6/CP7/CP8 |

## 运行范围

| 项目 | 允许 / 禁止 | 说明 |
|---|---|---|
| QMT / MiniQMT / XtQuant runtime | 允许，限本 CR 只读 smoke | 需用户确认客户端已启动 / 已登录 / 可达 |
| health | 允许 | 只读探测 |
| capabilities | 允许 | 只读能力查询 |
| query_positions_readonly | 允许，脱敏输出 | 不输出真实账号、代码、数量、资金或原始日志 |
| NAS | 禁止 | 不参与本 CR |
| `.env` / 凭据读取 | 禁止 | Codex 不读取凭据；由用户本地客户端已登录承担 |
| submit / cancel / buy / sell | 禁止 | forbidden counter 必须为 0 |
| simulation / live 交易 | 禁止 | 本 CR 不是交易授权 |
| provider / lake / publish | 禁止 | 不执行数据写入或发布 |

## 执行前人工协助需求

| 编号 | 需要用户协助 | 为什么需要 | 当前状态 |
|---|---|---|---|
| H-CR097-01 | 确认运行位置：当前 Linux 环境是否能访问 QMT 客户端，或需要在 Windows / 交易主机执行 | QMT / MiniQMT / XtQuant 通常依赖本地客户端和登录态 | RESOLVED：QMT 运行在 Windows，Codex 当前在 WSL；优先采用 Windows 侧执行 / WSL 侧接收脱敏 evidence |
| H-CR097-02 | 确认账户模式：模拟账户 / 实盘只读 / 其他 | 决定 evidence 脱敏级别和是否允许 query_positions_readonly | RESOLVED：模拟账户 |
| H-CR097-03 | 确认只读入口：已有 gateway / runner 命令、XtQuant Python 环境，或需要用户手工执行后回传 evidence | 决定是由 Codex 执行命令，还是用户在交易主机手工运行 | RESOLVED：Windows gateway；Windows host `172.30.32.1`；推荐端口 `18765`；WSL client IP `172.30.33.29/32` |
| H-CR097-04 | 确认 evidence 输出路径 | 推荐 `/home/hyde/.quant-lab/evidence/qmt/cr097/redacted/`；真实交易主机可先生成本地文件再转脱敏摘要 | RESOLVED：使用 `/home/hyde/.quant-lab/evidence/qmt/cr097/redacted/`，Windows 侧可写入 `C:\Users\<user>\.quant-lab\evidence\qmt\cr097\redacted\` 后复制到 WSL |
| H-CR097-05 | 运行时保持人工可中止 | 防止误触非只读操作或客户端异常 | PARTIAL：用户在 Windows 侧启动 gateway；执行 health / capabilities / query_positions 前保持人工可中止，若出现异常立即 Ctrl+C 停止 gateway |

## Runtime Preflight 参数

| 字段 | 值 |
|---|---|
| account_mode | simulated |
| execution_mode | windows_gateway |
| windows_gateway_host | `172.30.32.1` |
| windows_gateway_port | `18765` |
| base_url | `http://172.30.32.1:18765` |
| wsl_client_ip | `172.30.33.29` |
| recommended_allowlist | `172.30.33.29/32` |
| runtime_authorization_ref | `cr097-readonly-smoke-20260619-sim` |
| run_id | `cr097-readonly-smoke-20260619-sim-001` |
| request_id | `cr097-readonly-smoke-20260619-sim-001-query-positions` |
| wsl_redacted_evidence_dir | `/home/hyde/.quant-lab/evidence/qmt/cr097/redacted/` |

## fast-lane / standard 判定

| 条件 | 是否命中 | 说明 |
|---|---|---|
| 仅低风险轻量实现 / 文档 / 规则修改 | false | 真实 runtime 高风险 |
| 修改架构、权限、安全边界或平台安装路径 | true | 运行授权和账户边界 |
| 修改外部接口契约、文件所有权或多 Story 依赖 | true | QMT / MiniQMT / XtQuant runtime |
| 需要 HLD / LLD 才能解释影响 | true | 复用 CR092 HLD，仍需 runtime preflight |
| 是否保持 fast-lane | false | 升级为 standard |

## LLD / runtime preflight 门禁

- 是否需要 LLD 设计批次：true
- batch_id：`CR097-runtime-preflight`
- 批次范围来源：CR092-FU-02 runtime authorization
- 批次内 Story：`CR097-REAL-QMT-READONLY-RUNTIME-SMOKE`
- 开发 / 执行启动条件：
  - [x] 用户明确授权真实 QMT runtime 只读 smoke
  - [x] 冲突预检通过
  - [ ] 用户补充 `H-CR097-01..05`
  - [ ] 生成 runtime preflight evidence
  - [ ] 明确最终执行命令 / 手工运行步骤和 evidence 输出路径
  - [ ] forbidden counters 初始为 0

## 执行链路

| 顺序 | 责任角色 | 动作 | 输入 | 输出 | 门控 | 完成后下一步 |
|---|---|---|---|---|---|---|
| 1 | `host-orchestrator` | 创建 CR097 并登记 active | 用户请求 / CR092 follow-up tracking | 本 CR、STATE / CR-INDEX 更新 | 冲突预检通过 | 收集人工协助信息 |
| 2 | `host-orchestrator` | runtime preflight | H-CR097-01..05 | preflight evidence | 不读取凭据；不执行交易写 | 准备执行命令 |
| 3 | `host-orchestrator` / 用户 | 执行只读 smoke | 明确命令或用户手工执行 | redacted evidence | 只读 scope | CP6 / CP7 验证 |
| 4 | `host-orchestrator` | 收敛终验 | evidence / release context | CP8 自动预检与人工审查稿 | 用户确认 | 关闭 CR097 |

## 自动终验授权

- 是否启用：false
- 授权范围：不适用
- 适用检查点：CP8
- 自动通过条件：N/A
- 授权原文：
- 授权时间：
- 回填要求：N/A

## 后续事项台账

- 是否存在后续事项：true
- 台账路径：`process/changes/CR-092-FOLLOW-UP-TRACKING-2026-06-18.md`
- CR 索引路径：`process/changes/CR-INDEX.yaml`
- 一致性检查：`meta-flow check cr-tracking --project-root . --strict-warnings`

## 处理结论

- 审批结论：`runtime-scope-authorized-by-user-pending-preflight`
- [ ] 自动批准（低风险启动）
- [x] 待人工确认（高风险 runtime；用户已给出启动授权，但执行前仍需 H-CR097-01..05）
- [ ] 待人工审批（高风险且无授权）

## 关联对象

| 类型 | 标识 | 说明 |
|---|---|---|
| CP8 | `process/checkpoints/CP8-CR092-DELIVERY-READINESS.md` | 来源 follow-up 决策 |
| Follow-up tracking | `process/changes/CR-092-FOLLOW-UP-TRACKING-2026-06-18.md` | `CR092-FU-02` 候选来源 |
| CR092 HLD | `docs/qmt/CR092-REAL-QMT-READONLY-RUNTIME-SMOKE-HLD.md` | runtime smoke 设计基线 |
| CR092 manual guide | `docs/qmt/CR092-REAL-QMT-READONLY-RUNTIME-SMOKE-MANUAL-GUIDE.md` | 静态 guide；CR097 需要 runtime preflight 扩展 |
| CR096 evidence path pattern | `/home/hyde/.quant-lab/evidence/qmt/cr096/redacted/` | CR097 推荐沿用 `.quant-lab/evidence/qmt/cr097/redacted/` |

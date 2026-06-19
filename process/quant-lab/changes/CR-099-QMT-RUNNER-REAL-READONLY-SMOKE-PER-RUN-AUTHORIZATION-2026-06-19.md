---
cr_id: "CR-099"
status: "closed-current-delivery"
impact_level: "high"
workflow_mode_before: "production"
workflow_mode_after_change: "standard"
fast_lane_upgrade_reason: "命中 QMT runner、Windows gateway、HMAC client env、真实只读账户查询、redacted evidence、runtime_authorization 和 per-run authorization；不得使用 fast-lane。"
rollback_to: "CR098 closed-current-delivery / READY_WITH_RISK"
approval_result: "cp8-approved-ready-with-risk"
created_at: "2026-06-19T15:24:50+08:00"
created_by: "host-orchestrator"
approved_by: "user"
approved_at: "2026-06-19T15:54:07+08:00"
cp8_approved_at: "2026-06-19T17:15:29+08:00"
source: "cp8-follow-up"
linked_issue: ""
parent_cr: "CR-098"
source_checkpoint: "process/checkpoints/CP8-CR098-DELIVERY-READINESS.md"
source_decision_id: "DQ-CP8-CR098-02"
follow_up_type: "runner-real-readonly-smoke-per-run-authorization"
risk_class: "trading-runtime-boundary"
owner: "host-orchestrator"
revisit_condition: "用户明确要求推进 `CR098-FU-01 Runner real readonly smoke per-run authorization`。"
acceptance_criteria: "仅在用户逐 run 授权范围内执行 runner -> Windows gateway 真实只读 smoke；scope 限定为 runner 调用 health、capabilities、query_positions_readonly；输出脱敏 evidence；forbidden counters 全部为 0；不输出账户、证券代码、数量、资金、委托、成交或原始日志。"
close_condition: "完成 CP2/CP3/CP5 门禁和逐 run 授权后，执行 runner real readonly smoke 并产出脱敏 evidence / 或因缺少授权、gateway、client env、账户模式、人工协助或安全边界转 blocked。"
cr_index_path: "process/changes/CR-INDEX.yaml"
---

# CR099 QMT Runner Real Readonly Smoke Per-run Authorization

## 变更描述

用户要求推进 `CR098-FU-01 Runner real readonly smoke per-run authorization`。本 CR 的目标是把 CR098 已完成的离线 runner readonly integration 推进到真实 Windows gateway 只读路径下的 runner smoke：runner 作为消费方调用 gateway `health`、`capabilities` 和 `query_positions_readonly`，并只输出 `.quant-lab` 下的脱敏 evidence。

本 CR 当前只启动正式 CR 和 CP2 scope 门禁。当前用户请求不等于真实运行授权；在 CP2 / CP3 / CP5 和明确逐 run 授权完成前，不读取 HMAC secret、不读取 Windows `.env`、不启动 gateway、不执行 runner runtime、不查询账户、不访问 NAS、不执行交易写、simulation/live、provider/lake/publish 或真实 release。

## CP8 Follow-up 来源

| 字段 | 内容 |
|---|---|
| 父级 CR | `CR-098` |
| 来源检查点 | `process/checkpoints/CP8-CR098-DELIVERY-READINESS.md` |
| 来源决策 ID | `DQ-CP8-CR098-02` |
| follow-up 类型 | runner real readonly smoke per-run authorization |
| 风险等级 | high / trading-runtime-boundary |
| owner | host-orchestrator |
| 重访条件 | 用户要求推进 `CR098-FU-01` |
| 验收标准 | runner 真实消费 Windows gateway readonly path，输出脱敏 evidence，forbidden counters 为 0 |
| 关闭条件 | CP8 完成，或 runtime preflight / authorization blocked 后用户确认关闭或延后 |

## 启动前冲突预检

| 检查项 | 结论 | 证据 | 说明 |
|---|---|---|---|
| process 路由 | PASS | `uv run --python 3.11 meta-flow workspace check` | `process_link_health: ok` |
| 当前 active formal CR | PASS | `uv run --python 3.11 meta-flow check cr-tracking --project-root .` | 启动前 active formal CR 为 none |
| blocked CR 重叠 | PASS_WITH_RISK | `CR-089` | CR089 涉及 NAS strategy package exchange / interface validation；CR099 不恢复 CR089、不使用 NAS、不合并 package exchange |
| 候选来源 | PASS | `process/changes/CR-098-FOLLOW-UP-TRACKING-2026-06-19.md` | `CR098-FU-01` 为 candidate-not-started，用户本轮明确推进 |
| 上游 runner baseline | PASS | `CR-098` / CP8 | runner readonly integration 离线实现和验证已关闭为 READY_WITH_RISK |
| 上游 gateway baseline | PASS | `CR-097` / CP8 | Windows gateway readonly smoke 已通过，但 CR099 不能复用 CR097 的一次性运行授权 |
| 用户 runtime 授权 | PENDING | 当前对话 / CP2 | 用户明确推进 CR；真实 run 仍需 CP2/CP3/CP5 后的逐 run 授权 |
| 不授权边界 | PASS | 本 CR | 不授权 HMAC secret、Windows `.env`、账户原文、NAS、交易写、simulation/live、provider/lake/publish |

## 文档处理决策

| 受影响文档 | 处理方式 | 旧基线保留方式 | 修订记录位置 | 批准状态 |
|---|---|---|---|---|
| `docs/product/USE-CASES.md` | 不变 | 不涉及产品场景基线重写 | 不适用 | approved-cp2 |
| `docs/product/REQUIREMENTS.md` | 不变 | 不涉及需求基线重写 | 不适用 | approved-cp2 |
| `process/changes/CR-099-QMT-RUNNER-REAL-READONLY-SMOKE-PER-RUN-AUTHORIZATION-2026-06-19.md` | 新增 | 保留 CR097 / CR098 原交付事实和不授权边界 | 本文件 | approved-cp2 |
| `process/changes/CR-098-FOLLOW-UP-TRACKING-2026-06-19.md` | 原文档更新 | 保留 `CR098-FU-01` 原候选行并更新为 active / CR099 | 台账表 | approved-cp2 |
| `process/changes/CR-INDEX.yaml` | 原文档更新 | 保留历史 CR 索引；新增 CR099 active | YAML 结构化索引 | approved-cp2 |
| `process/STATE.md` | 原文档更新 | 保留 CR098 closure history；顶层 active_change 切换到 CR099 | history / cr_tracking | approved-cp2 |
| `process/context/CP2-CR099-RUNNER-REAL-READONLY-SMOKE-CONTEXT.yaml` | 新增 | 作为 CP2 最小上下文胶囊 | capsule metadata | approved-cp2 |
| `process/checkpoints/CP2-CR099-RUNNER-REAL-READONLY-SMOKE-SCOPE-REVIEW.md` | 新增 | CP2 人工门禁 | 人工审查结果 | approved |
| `docs/qmt/CR099-QMT-RUNNER-REAL-READONLY-SMOKE-HLD.md` | 新增 | CP3 HLD 基线 | 修订记录 | approved-cp3 |
| `process/checkpoints/CP3-CR099-RUNNER-REAL-READONLY-SMOKE-HLD-REVIEW.md` | 新增 | CP3 人工门禁 | 人工审查结果 | approved |
| `docs/features/cr099-runner-real-readonly-smoke/DESIGN.md` | 新增 | CP5 Feature Design | 修订记录 | approved-cp5-cp6-pass |
| `docs/features/cr099-runner-real-readonly-smoke/TEST-PLAN.md` | 新增 | CP5 Test Plan | 修订记录 | approved-cp5-cp6-pass |
| `docs/features/cr099-runner-real-readonly-smoke/TASKS.md` | 新增 | CP5 Tasks | 修订记录 | approved-cp5-cp6-pass |
| `process/stories/CR099-RUNNER-REAL-READONLY-SMOKE-LLD.md` | 新增 | CP5 full LLD | 修订记录 | approved-cp5 |
| `process/context/CP5-CR099-RUNNER-REAL-READONLY-SMOKE-CONTEXT.yaml` | 新增 | CP5 最小上下文胶囊 | capsule metadata | approved-cp5 |
| `process/checkpoints/CP5-CR099-RUNNER-REAL-READONLY-SMOKE-READINESS.md` | 新增 | CP5 人工门禁 | 人工审查结果 | approved |
| `scripts/check_cr099_redacted_evidence.py` | 新增 | CP6 offline checker | code | cp6-pass |
| `tests/test_cr099_runner_real_readonly_smoke_contract.py` | 新增 | CP6 tests | pytest | cp6-pass |
| `process/stories/CR099-RUNNER-REAL-READONLY-SMOKE-IMPLEMENTATION.md` | 新增 | CP6 实现证据 | implementation evidence | cp6-pass |
| `process/checks/CP6-CR099-RUNNER-REAL-READONLY-SMOKE-CODING-DONE.md` | 新增 | CP6 自动检查 | check result | PASS |

## 五维度影响分析

| 维度 | 评估问题 | 受影响对象 | 结论 | 处理动作 |
|---|---|---|---|---|
| 需求层 | 是否新增、删除或重定义 REQ-* | runner real readonly smoke follow-up | true | 新增 CR099 gate，不重写既有需求基线 |
| 场景层 | 是否改变测试矩阵覆盖范围 | runner health、capabilities、query_positions_readonly、redacted evidence | true | 激活 CR098-FU-01 真实 runner readonly smoke 场景 |
| 计划层 | 是否改变 Phase / Wave / Story / 任务依赖 | `process/STATE.md` / CR tracking | true | 新增 CR099 active，进入 CP2 scope review |
| 安全层 | 是否引入新的高风险动作或权限要求 | HMAC client env、Windows gateway、真实只读账户查询 | true | 标准模式、分阶段门禁、逐 run 授权、脱敏 evidence、forbidden counters |
| 交付层 | 是否需要重新生成交付物或回归子集 | runtime preflight / smoke evidence / CP6 / CP7 / CP8 | true | CP2/CP3/CP5 后才允许准备真实 run；CP8 收敛剩余风险 |

## 目标范围

### In Scope

- runner 通过受控 gateway client 调用：
  - `health`
  - `capabilities`
  - `query_positions_readonly`
- 复用 CR098 已实现的 runner readonly adapter / evidence 合同。
- 复用 CR097 已验证的 Windows gateway 只读路径作为候选运行方式，但不复用 CR097 的一次性授权。
- 每次真实运行必须有新的 `authorization_ref` / `run_id`，并明确 host、port、allowlist、client env 处理方式、允许接口和中止条件。
- evidence 只允许写入 `.quant-lab/evidence/qmt/cr099/redacted/` 或 CP5/运行授权明确的等价 redacted 目录。

### Out of Scope / Not Authorized

- NAS package exchange / pull / publish。
- Windows `.env` 自动读取。
- HMAC secret 自动读取或复用 CR097 env。
- 账号、资金、持仓明细原文、证券代码、委托、成交或原始日志输出。
- submit / cancel、buy / sell、simulation / live。
- provider fetch、lake write、catalog publish。
- CR089 auto-start 或 CR020 gateway route restore。
- CR098 CP8 approve 或本 CR 创建动作不构成 runtime 授权。

## 推荐方案

推荐采用 `runner real readonly smoke with explicit per-run authorization`：

- CP2 确认范围和不授权边界。
- CP3 固化真实 runner smoke 架构、运行参数来源、失败/中止路径和脱敏策略。
- CP5 确认运行前置条件、命令 / 手工步骤、evidence schema、forbidden counters 和回滚条件。
- CP5 之后，只有用户提供明确逐 run 授权文本，才允许读取最小 client env / 连接 gateway / 执行 runner readonly smoke。

## 备选方案

| 方案 | 优点 | 缺点 | 切换条件 |
|---|---|---|---|
| A. runner real readonly smoke with explicit per-run authorization（推荐） | 直接证明 runner 真实消费 gateway readonly path；授权边界可审计 | 需要用户配合运行环境和逐 run 授权 | CP2/CP3/CP5 批准且用户提供授权 |
| B. 只做离线 reverify | 风险最低，不触碰 runtime | 不能关闭真实 runner runtime 风险 | 用户暂不方便提供运行环境 |
| C. 合并非空 / 交易日复测 | 覆盖更强 | 需要非空持仓或交易日条件，可能扩大等待时间 | 用户明确要求同 CR 覆盖且接受延期 |
| D. 合并 NAS / order-write | 一次规划更多后续能力 | 范围和权限风险显著上升 | 不推荐；必须另起独立 CR |

## 当前门控

| 门控 | 状态 | 说明 |
|---|---|---|
| 候选启动 | completed | 用户要求推进 `CR098-FU-01` |
| 冲突预检 | PASS_WITH_RISK | CR089 blocked 但不合并 / 不恢复；CR099 独立处理 runner readonly smoke |
| CP2 scope review | approved | 用户回复“同意”，接受 `DQ-CP2-CR099-01..05`；仍不授权 runtime |
| CP3 HLD | approved | 用户回复“同意”，接受 `DQ-CP3-CR099-01..05`；仍不授权 runtime |
| CP5 design readiness | approved | 用户回复“同意”，接受 `DQ-CP5-CR099-01..05`；仍不授权 runtime |
| CP6 offline implementation | PASS | 离线 run contract / redacted evidence checker 和 tests 已完成 |
| CP7 offline verification | PASS_WITH_RISK | 离线 checker、negative tests、CR098 回归和过程证据已验证；真实 runtime 未授权未执行 |
| Runtime authorization | authorized-attempt-blocked | 用户已授权 CR099 单次 WSL client-side smoke；health/capabilities 通过，positions 返回 `session_expired` |
| CP8 delivery readiness | approved | 用户接受 CR099 CP8 的 5 项推荐方案和风险；关闭当前交付为 `closed-current-delivery / READY_WITH_RISK`；不授权新的 runtime run、下单 / 撤单验证、读取 Windows `.env`、访问 NAS、publish 或任何交易写操作 |

## Runtime 授权执行记录

| 字段 | 内容 |
|---|---|
| authorization_ref | `cr099-runner-readonly-smoke-20260619-sim` |
| run_id | `cr099-runner-readonly-smoke-20260619-sim-001` |
| base_url | `http://172.30.32.1:18765` |
| client env 来源 | `/home/hyde/.quant-lab/runtime/qmt/cr097/client.env.template`，仅由运行命令读取；未打印 / 未记录 secret 值 |
| evidence path | `/home/hyde/.quant-lab/evidence/qmt/cr099/redacted/cr099-runner-readonly-smoke-20260619-sim-001/evidence.json` |
| CP7 runtime attempt check | `process/checks/CP7-CR099-RUNTIME-AUTHORIZED-SMOKE-ATTEMPT-2026-06-19.md` |
| evidence schema | PASS，CR099 checker `passed=true`，errors=0 |
| health | PASS，`status=ok`，`session_ready=true` |
| capabilities | PASS，`status=ok`，`readonly_supported=true` |
| query_positions_readonly | BLOCKED，`reason_code=session_expired` |
| forbidden counters | PASS，全部为 0 |
| 当前结论 | `BLOCKED_RUNTIME_SESSION_EXPIRED` |
| 下一步 | 用户在 Windows 端刷新 / 重启 gateway session 后，复跑同一 CR099 smoke；当前未发现需要 GitHub server-side push |

### Runtime 授权复跑结果

| 字段 | 内容 |
|---|---|
| rerun_at | `2026-06-19T16:54:06+08:00` |
| trigger | 用户回复“我已经重新启动起来了” |
| CP7 rerun check | `process/checks/CP7-CR099-RUNTIME-AUTHORIZED-SMOKE-RERUN-PASS-2026-06-19.md` |
| evidence path | `/home/hyde/.quant-lab/evidence/qmt/cr099/redacted/cr099-runner-readonly-smoke-20260619-sim-001/evidence.json` |
| evidence schema | PASS，CR099 checker `passed=true`，errors=0，warnings=0 |
| overall_status | `pass` |
| health | PASS，`status=ok`，`session_ready=true` |
| capabilities | PASS，`status=ok`，`readonly_supported=true` |
| query_positions_readonly | PASS，`status=ok`，`position_count_bucket=zero`，`items_redacted_count=0` |
| forbidden counters | PASS，全部为 0 |
| 当前结论 | `PASS`，CR099 可进入 CP8 delivery readiness / risk acceptance |

## LLD / runtime preflight 门禁

- 是否需要 LLD 设计批次：true
- batch_id：`CR099-runner-real-readonly-smoke`
- 批次范围来源：CR098-FU-01 runtime authorization follow-up
- 批次内 Story：`CR099-RUNNER-REAL-READONLY-SMOKE`
- 开发 / 执行启动条件：
  - [ ] CP2 scope review approved
  - [x] CP3 HLD approved
  - [x] CP5 design readiness approved
  - [ ] 用户提供逐 run 授权文本，明确 `authorization_ref`、`run_id`、host、port、allowlist、允许 endpoint、evidence dir 和中止条件
  - [ ] forbidden counters 初始为 0
  - [ ] evidence redaction gate ready
  - [ ] 不读取或记录 HMAC secret、Windows `.env`、账户原文、NAS、交易写或 provider/lake/publish

## 执行链路

| 顺序 | 责任角色 | 动作 | 输入 | 输出 | 门控 | 完成后下一步 |
|---|---|---|---|---|---|---|
| 1 | `host-orchestrator` | 创建 CR099 并登记 active | 用户请求 / CR098 follow-up tracking | 本 CR、STATE / CR-INDEX 更新、CP2 材料 | 冲突预检通过 | 发起 CP2 人工确认 |
| 2 | `host-orchestrator` / `meta-se` | HLD / runtime design | CP2 approved | CP3 HLD / Decision Brief | CP3 approved | 进入 CP5 |
| 3 | `host-orchestrator` / `meta-dev` | LLD / test plan / run contract | CP3 approved | CP5 design readiness | CP5 approved | 等待逐 run 授权 |
| 4 | `host-orchestrator` / 用户 | 执行真实 runner readonly smoke或收集用户执行证据 | 明确逐 run 授权 | redacted evidence / blocked evidence | 只读 scope | CP6 / CP7 验证 |
| 5 | `host-orchestrator` | 收敛终验 | evidence / release context | CP8 自动预检与人工审查稿 | 用户确认 | 关闭 CR099 或转 follow-up |

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
- 台账路径：`process/changes/CR-098-FOLLOW-UP-TRACKING-2026-06-19.md`
- CR 索引路径：`process/changes/CR-INDEX.yaml`
- 一致性检查：`meta-flow check cr-tracking --project-root . --strict-warnings`

## CP8 审查结果

| 字段 | 内容 |
|---|---|
| 结论 | `approved / READY_WITH_RISK` |
| 审查人 | `user` |
| 审查时间 | `2026-06-19T17:15:29+08:00` |
| 用户回复 | 接受 CR099 CP8 的 5 项推荐方案和风险，不表示授权新的 runtime run、下单 / 撤单验证、读取 Windows `.env`、访问 NAS、publish 或任何交易写操作 |
| 接受的决策项 | `DQ-CP8-CR099-01`、`DQ-CP8-CR099-02`、`DQ-CP8-CR099-03`、`DQ-CP8-CR099-04`、`DQ-CP8-CR099-05` |
| 关闭状态 | `closed-current-delivery` |
| release decision | `READY_WITH_RISK` |
| 后续建议 | 非空 / 交易日 readonly retest 保留为候选；NAS package exchange、order-write / simulation / live design gate 均需独立门禁和明确授权 |
| 不授权边界 | 新的 runtime run、下单 / 撤单验证、读取 Windows `.env`、访问 NAS、publish 或任何交易写操作均不授权 |

## 处理结论

- 审批结论：`cp5-approved-cp6-pass`
- [ ] 自动批准（低风险启动）
- [x] 已完成人工确认（高风险 runner / runtime scope；用户回复“同意”）
- [ ] 待人工审批（真实 runtime 执行；CP5 后逐 run 授权）

## CP2 审查结果

| 字段 | 内容 |
|---|---|
| 结论 | `approved` |
| 审查人 | `user` |
| 审查时间 | `2026-06-19T15:54:07+08:00` |
| 用户回复 | `同意` |
| 接受的决策项 | `DQ-CP2-CR099-01`、`DQ-CP2-CR099-02`、`DQ-CP2-CR099-03`、`DQ-CP2-CR099-04`、`DQ-CP2-CR099-05` |
| 下一门控 | CP3 HLD review |
| runtime 授权 | `not-authorized` |

CP2 approve 只表示用户接受 CR099 的范围、风险和不授权边界，不表示授权读取 HMAC secret、读取 Windows `.env`、启动 gateway、执行 runner runtime、查询账户、访问 NAS、执行交易写、simulation/live、provider/lake/publish 或真实 release。

## CP3 审查结果

| 字段 | 内容 |
|---|---|
| 结论 | `approved` |
| 审查人 | `user` |
| 审查时间 | `2026-06-19T16:04:52+08:00` |
| 用户回复 | `同意` |
| 接受的决策项 | `DQ-CP3-CR099-01`、`DQ-CP3-CR099-02`、`DQ-CP3-CR099-03`、`DQ-CP3-CR099-04`、`DQ-CP3-CR099-05` |
| 下一门控 | CP5 design readiness |
| runtime 授权 | `not-authorized` |

CP3 approve 只表示用户接受 CR099 HLD 推荐架构和不授权边界，不表示授权读取 HMAC secret、读取 Windows `.env`、启动 gateway、执行 runner runtime、查询账户、访问 NAS、执行交易写、simulation/live、provider/lake/publish 或真实 release。

## CP5 审查结果

| 字段 | 内容 |
|---|---|
| 结论 | `approved` |
| 审查人 | `user` |
| 审查时间 | `2026-06-19T16:13:23+08:00` |
| 用户回复 | `同意` |
| 接受的决策项 | `DQ-CP5-CR099-01`、`DQ-CP5-CR099-02`、`DQ-CP5-CR099-03`、`DQ-CP5-CR099-04`、`DQ-CP5-CR099-05` |
| 下一门控 | CP6 offline implementation / CP7 verification |
| runtime 授权 | `not-authorized` |

CP5 approve 只允许进入 CP6 offline contract / schema checker implementation，不表示授权读取 HMAC secret、读取 Windows `.env`、启动 gateway、执行 runner runtime、查询账户、访问 NAS、执行交易写、simulation/live、provider/lake/publish 或真实 release。

## CP6 实现结果

| 字段 | 内容 |
|---|---|
| 结论 | `PASS` |
| 检查文件 | `process/checks/CP6-CR099-RUNNER-REAL-READONLY-SMOKE-CODING-DONE.md` |
| 实现摘要 | `process/stories/CR099-RUNNER-REAL-READONLY-SMOKE-IMPLEMENTATION.md` |
| 实现对象 | `scripts/check_cr099_redacted_evidence.py`、`tests/test_cr099_runner_real_readonly_smoke_contract.py` |
| 验证 | CR099 pytest 7 passed；CR099 + CR098 regression 14 passed；py_compile PASS；CLI help PASS |
| runtime 授权 | `not-authorized` |

## CP7 验证结果

| 字段 | 内容 |
|---|---|
| 结论 | `PASS_WITH_RISK` |
| 检查文件 | `process/checks/CP7-CR099-RUNNER-REAL-READONLY-SMOKE-VERIFICATION-DONE.md` |
| 验证报告 | `docs/features/cr099-runner-real-readonly-smoke/VERIFICATION.md` |
| 测试报告 | `docs/features/cr099-runner-real-readonly-smoke/TEST-REPORT.md` |
| Review | `docs/features/cr099-runner-real-readonly-smoke/REVIEW.md` |
| Fixes | `docs/features/cr099-runner-real-readonly-smoke/FIXES.md` |
| 主要剩余风险 | 真实 runner -> Windows gateway smoke 未执行；HMAC env / gateway connectivity 未验证；非空持仓路径未证明 |
| runtime 授权 | `not-authorized` |

## 关联对象

| 类型 | 标识 | 说明 |
|---|---|---|
| Parent CR | `CR-098` | runner readonly integration 离线实现和验证 |
| Upstream runtime CR | `CR-097` | Windows gateway readonly smoke 通过；不复用运行授权 |
| Source follow-up | `CR098-FU-01` | 本 CR 启动来源 |
| Follow-up tracking | `process/changes/CR-098-FOLLOW-UP-TRACKING-2026-06-19.md` | CR098 后续事项台账 |
| Context handoff | `process/context/CR098-CLOSURE-CONTEXT-RESET-HANDOFF-2026-06-19.md` | 清上下文恢复入口 |

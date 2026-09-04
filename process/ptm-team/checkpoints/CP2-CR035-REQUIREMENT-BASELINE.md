---
checkpoint_id: "CP2-CR035-REQUIREMENT-BASELINE"
checkpoint_name: "CR-035 需求/范围基线门"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
reviewed_by: "zhaohaibo"
reviewed_at: "2026-07-31T09:50:00+08:00"
created_at: "2026-07-31T09:40:00+08:00"
auto_check_result: "process/checks/CP1-CR035.result.json"
cp1_result_ref: "process/checks/CP1-CR035.result.json"
cp0_result_ref: "process/checks/CP0-CR035.result.json"
target:
  phase: "requirement-clarification"
  cr_id: "CR-035"
---

# CP2 人工审查 - CR-035 需求/范围基线门

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP0-CR035.result.json` | PASS | 0 | bootstrap artifacts 就位（CR-035.md/summary/INDEX/ledger/route-plan）；route plan 派生（CP0/CP1/CP2/CP3/CP5/CP6/CP7/CP8 applies，CP4 N/A）；runtime/credential 未授权。 |
| `process/checks/CP1-CR035.result.json` | PASS | 0 | 4 点 TG 生命周期约束 + 调研事实结构化于 CR-035.md；实现范围按 op_mapper/case_runner/文档/测试分层；5 个待决策项已识别；实现 CR 范畴确认（CR-034 明确 case_runner 行为变更属实现 CR）。 |
| Scenario Gray Areas | N/A | 0 | 实现 follow-up CR，无新用户场景灰区（fast-lane 等效，CP2 直接确认需求/范围 + 5 个设计决策）。 |

## Decision Brief

### 审批者摘要

| 字段 | 内容 |
|---|---|
| 本次确认服务的整体目标 | 确认 CR-035 需求/范围基线（TG 端口生命周期 config-once：run 级 acquire + tg_config_interface config-once skip + release）+ 5 个设计决策（DQ-035-01~05），授权进入 CP3 HLD -> CP5 设计证据 -> CP6 实现 -> CP7 验证。 |
| 推荐动作 | `approve`：接受 TG 生命周期 config-once 范围 + 5 个待决策项推荐方案（DQ-035-01 step 级 force_config / DQ-035-02 dev<IP末段>-<run哈希> / DQ-035-03 中断 / DQ-035-04 首次后 skip / DQ-035-05 注册 verify_ownership）。 |
| approve 后会发生什么 | host-orchestrator 自动推进 CP3 HLD（config-once 架构方案：skip 模式 + user_id 占用模式结合），委托 meta-se 直连用户确认架构灰区；CP3 通过后 CP5 设计证据 -> CP6 实现（op_mapper 注册 + case_runner 编排 + 文档 + 测试） -> CP7 验证（dry-run + validate + 单测）；validation_mode=static-only + dry-run-only，无 runtime。 |
| approve 不授权什么 | 不授权真实设备 `--execute` 写操作（FU-01 runtime 端到端，独立授权）、ptm-atomic CLI / trex-api / trex-traffic 代码改动（已实现，本 CR 只消费）、24 用例 md 改动（R-C-008）、外部写入/生产/发布。设计通过不等于运行授权。 |
| 不确认会阻塞什么 | 阻塞 CP3 HLD 与 CP6 实现；force_config 标识 / user_id 生成 / acquire 失败处理未定，case_runner 编排逻辑无法落地。 |

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP1-CR035.context.json` |
| read_profile | compact |
| 默认读取策略 | CR-035.md 需求/范围/决策项 + CP0/CP1 result + 调研报告结论；不读取完整 case_runner.py / op_mapper.py 全文（行号已核实引用）。 |
| 关键数字 | 4 点约束 / 3 op 新注册（acquire/release/verify-ownership）+ tg 族 user_id 映射 / 2 核心函数改（main + execute_steps）/ 5 待决策项 / 0 runtime |
| 全文档读取 | 默认不读完整 case_runner.py/op_mapper.py；已按 read_expansion_refs 扩展读取 CR-035.md（CP0/CP1 审计边界）+ CR-034.md（范畴确认）。 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| 用户 4 点约束 | 用户消息 TG 生命周期约束描述 | scanned | 5 | 5 | DQ-035-01~05 全部转入待决策 |
| case_runner.py | main/execute_steps/preconfigure/fw_login skip | scanned | 0 | 0 | 调研已定位切入点，行号核实 |
| op_mapper.py | OP_ID_TO_SUBCOMMAND/ARGS_TO_FLAGS | scanned | 1 | 0 | acquire/release 注册方案明确（agent 默认处理） |
| trex-traffic SKILL.md | acquire/release/user_id 占用模型 | scanned | 1 | 1 | DQ-035-02 user_id 生成规则 |
| CR-033 follow-up 台账 | FU-02 聚合占位符 | scanned | 1 | 0 | FU-02 并入本 CR 补测试关闭（agent 默认） |

### 决策分层

| 分类 | 数量 | 处理方式 |
|---|---:|---|
| 必须用户决策 | 3 | DQ-035-01 force_config 标识（影响 case_steps schema）/ DQ-035-02 user_id 生成（影响 acquire/release 命名）/ DQ-035-03 acquire 失败处理（影响 run 容错） |
| 高风险策略确认 | 1 | DQ-035-05 verify_ownership 注册（影响 op 范围 26 vs 25，EXPECTED_OP_COUNT） |
| agent 默认处理 | 1 | DQ-035-04 dry-run config-once 行为（与 fw_login dry-run 一致，低风险可回退） |
| 仅审计记录 | 2 | CP0/CP1 自动预检 PASS；调研事实行号已核实 |

### 待人工决策清单

用户需决策事项：DQ-035-01（implementation）、DQ-035-02（implementation）、DQ-035-03（implementation）、DQ-035-05（scope）。4 项均阻塞 CP3 HLD/CP6 实现。DQ-035-04 为 agent 默认（approve 时一并接受推荐）。`approve` 表示接受以下全部推荐方案。

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-035-01 | implementation | 用例"显式要改 TG IP"的标识方式 | step 级 `force_config: true` 标志：case_steps 中 tg_config_interface step 加 `force_config: true`，execute_steps 检测到则真实执行并重置 `tg_interface_configured` | A: case 级 frontmatter `tg_reconfig_required: true`（整用例所有 tg_config_interface 真实执行）；B: 独立 op_id `tg_reconfig_interface` | 推荐粒度最细（step 级），与 case_steps 契约一致，复用现有 skip 判断分支；A 粗粒度但简单；B 改 op_mapper 范围大、与 config_interface 重复 | 影响 case_steps schema（新增 force_config 字段）+ execute_steps 判断逻辑；低风险（增量字段，向后兼容） | step 级 -> case 级可平滑迁移（force_config 字段保留）；切 B 需 op_mapper 改动 |
| DQ-035-02 | implementation | run 级 tg_user_id 生成规则（需同 run 一致、跨 run 唯一） | `dev<运行机IP末段>-<run_id短哈希>`（如 `dev172-a3f2`），从运行机 IP + run_id 派生 | A: 从 devices.yaml `tg.api_server` 派生；B: 固定值 `dev-ptm-te` | 推荐 cross-run 唯一、多用户并发隔离；A 依赖配置、同机多 run 冲突；B 多 run 共用 user_id 冲突 | 影响 acquire/release 命令 user_id；需确认运行机 IP 获取（socket.gethostname 解析）；低风险 | 生成规则可调整（user_id 是运行时参数，不影响 op_mapper 注册） |
| DQ-035-03 | implementation | run 级 tg_acquire_ports 失败时如何处理 | 中断整个 run（类比 `login_ok=False` 中断），记录 acquire 失败，不进入逐用例执行 | A: 降级走 default force 模式（不 acquire，每个 tg op 自动 acquire+release） | 推荐中断安全（避免无占用下跑用例导致端口抢占），与 login_ok 失败语义一致；降级兼容但失去 config-once 核心价值 | 影响 run 容错策略；中断可能掩盖环境问题（dry-run 不触发，仅 --execute 触发） | 可改为降级（acquire_ok 标志 + fallback 分支） |
| DQ-035-04 | implementation | dry-run 下 tg_config_interface config-once 行为（agent 默认） | 首次 build_command 后置 `tg_interface_configured=True`，后续 step skip（status=skipped），与 fw_login dry-run 一致 | A: 每次都 build_command（不 skip） | 推荐与 --execute 真实行为一致，dry-run 报告可预测；A 简单但不反映真实 skip 行为 | 影响 dry-run 报告准确性；低风险 | agent 默认处理（低风险可回退，与 fw_login 一致） |
| DQ-035-05 | scope | 是否在 op_mapper 注册 `tg_verify_ownership`（-> `tg trex verify-ownership`） | 注册（与 trex-api `GET /api/v1/ownership` 查询接口对齐，run 中可查询占用状态，便于调试） | A: 不注册（最小范围，EXPECTED_OP_COUNT 23->25 而非 26） | 推荐完整暴露 ownership 查询能力，与已实现服务端/CLI 对齐，调试友好；A 范围最小但缺查询能力 | 影响 EXPECTED_OP_COUNT（26 vs 25）+ tests/test_tg_op_mapping.py 断言；低风险（增量 op） | 可后补注册（op_mapper 增量） |

## Entry Criteria

| 条目 | 状态 | 证据 |
|---|---|---|
| CP0 bootstrap | PASS | `process/checks/CP0-CR035.result.json` |
| CP1 需求结构化 | PASS | `process/checks/CP1-CR035.result.json` |
| 调研事实行号核实 | PASS | CR-035.md 背景与来源（preconfigure L2319-2336 / fw_login skip L1473-1487 / op_mapper 6 tg op / ptm-atomic CLI 已支持） |
| 五维度影响分析 | PASS | CR-035.md 五维度影响分析 |
| 待决策项识别 | PASS | DQ-035-01~05 |

## Checklist

| # | 检查项 | 审查结果 | 证据 |
|---|---|---|---|
| 1 | TG 生命周期 config-once 范围明确（4 点约束） | PASS | CR-035.md 需求 |
| 2 | preconfigure_dut_interfaces 已 run 级一次（DUT 侧 config-once 已满足，不改） | PASS | CR-035.md 背景与来源 |
| 3 | ptm-atomic CLI 已支持 acquire/release/verify-ownership（不改 CLI，R-C-001 不冲突） | PASS | CR-035.md 背景与来源 |
| 4 | op_mapper 未注册 acquire/release + tg 族无 user_id 映射（核心缺口） | PASS | CR-035.md 背景与来源 |
| 5 | 5 个待决策项含推荐 + 备选 + 优劣 + 回退 | PASS | DQ-035-01~05 |
| 6 | validation_mode=static-only + dry-run-only | PASS | CR-035.md 约束 |
| 7 | FU-02 并入本 CR 补测试关闭，FU-01/03 保持独立 | PASS | CR-035.md 关联 |

## Exit Criteria

| 条目 | 审查结果 | 证据 |
|---|---|---|
| 自动预检无未豁免失败 | PASS | CP0/CP1 result |
| 用户终验确认 | PENDING | 本文件（待用户 approved） |

## Deliverables

| 交付物 | 路径 |
|---|---|
| CR formal doc | `process/changes/CR-035.md` |
| route plan | `process/checks/CP0-CR035.route-plan.json` |
| CP0 result | `process/checks/CP0-CR035.result.json` |
| CP1 result | `process/checks/CP1-CR035.result.json` |
| CP2 Decision Brief | `process/checkpoints/CP2-CR035-REQUIREMENT-BASELINE.md` |
| 关联台账 | `process/changes/CR-033-FOLLOW-UP-TRACKING-2026-07-30.md`（FU-02 并入） |

## 不授权范围

- 真实设备 `--execute` 写操作（FU-01 runtime 端到端，独立授权）。
- ptm-atomic CLI / trex-api 服务端 / trex-traffic CLI 代码改动（已实现，本 CR 只消费）。
- 24 用例 md 改动（R-C-008，留 ptm-te workspace）。
- 外部写入、生产操作或发布。

## 回复方式

本次为 CP2 人工门禁。请用以下三个 exact 回复之一：

- `approve` -- 接受 TG 生命周期 config-once 范围 + DQ-035-01~05 全部推荐方案（DQ-035-04 agent 默认），授权进入 CP3 HLD。
- `修改: <具体修改点>` -- 指出需调整的范围/决策项（如"修改: DQ-035-02 用固定值 dev-ptm-te"），host-orchestrator 重新计算影响面并重新发起确认。
- `reject` -- 不通过，说明原因，CR-035 暂停。

> 内部兼容别名：`1/通过` = approve，`2/修改: ...` = 修改，`3/不通过` = reject。请勿混排。

## 修订记录

| 版本 | 日期 | 变更要点 |
|---|---|---|
| v1.0 | 2026-07-31 | 初始 Decision Brief：4 点 TG 生命周期约束 + 5 DQ（DQ-035-01~05） |
| v1.1 | 2026-07-31 | CP3 评审发现 DQ-035-04 表述歧义（"dry-run 首次后 skip"与"与 fw_login dry-run 一致"矛盾，fw_login dry-run 实际不 skip L1474），提请 DQ-035-06 澄清（推荐 A dry-run 不 skip）。DQ-035-04 决策不撤销，由 DQ-035-06 细化 dry-run 行为。 |

## 人工审查结果

**approved**（zhaohaibo，2026-07-31T09:50:00+08:00）

接受 TG 生命周期 config-once 范围 + 5 项决策推荐方案：
- DQ-035-01：step 级 `force_config: true` 标志
- DQ-035-02：`dev<运行机IP末段>-<run_id短哈希>` 生成规则
- DQ-035-03：acquire 失败中断整个 run（类比 login_ok）
- DQ-035-04：dry-run 首次 build 后置标志后续 skip（agent 默认）
- DQ-035-05：注册 tg_verify_ownership（EXPECTED_OP_COUNT=26）

授权推进 CP3 HLD -> CP5 设计证据 -> CP6 实现 -> CP7 验证 -> CP8 交付。validation_mode=static-only + dry-run-only，无 runtime。
- CP0/CP1 自动预检 PASS
- approve 不授权：真实设备 --execute 写操作、ptm-atomic/trex-api/trex-traffic 代码改动、24 用例 md 改动、外部写入/生产/发布

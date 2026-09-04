---
checkpoint_id: "CP5-CR035-DESIGN-EVIDENCE-BATCH"
checkpoint_name: "CR-035 设计证据批量确认门"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
reviewed_by: "zhaohaibo"
reviewed_at: "2026-07-31T10:55:00+08:00"
created_at: "2026-07-31T10:45:00+08:00"
cp3_result_ref: "process/checkpoints/CP3-CR035-HLD-REVIEW.md"
hld_ref: "process/changes/CR-035-HLD.md"
context_ref: "process/context/CP5-CR035.context.json"
dispatch_evidence_ref: "process/state/AGENT-DISPATCH-LEDGER.ndjson#DISPATCH-CR035-CP5-meta-dev-001"
target:
  phase: "story-planning"
  cr_id: "CR-035"
---

# CP5 人工审查 - CR-035 设计证据批量确认门

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| CP3 approved | PASS | 0 | HLD v0.2 架构方案 + DQ-035-06 方案 A + AGA-1/3/4 agent 默认 + P0 整改已 approved。 |
| STORY-035-S01-LLD.md | PASS | 0 | full-lld lld-check OK（15 章节 + 14 token）；op_mapper +3 op + user_id 映射 + EXPECTED_OP_COUNT=26。 |
| STORY-035-S02-LLD.md | PASS | 0 | full-lld lld-check OK；main [4c]/[6c] + acquire_ok 初始 True（P0 整改）+ tg_port_ownership。 |
| STORY-035-S03-LLD.md | PASS | 0 | full-lld lld-check OK；config-once skip + force_config + tg_user_id 注入 + dry-run 不 skip（DQ-035-06）。 |
| STORY-035-S04.md | PASS | 0 | technical-note lld-check OK；三份文档同步 + Gotcha。 |
| STORY-035-S05.md | PASS | 0 | technical-note lld-check OK；FU-02 关闭 + 36 测试用例。 |
| cp5-context-check | PASS | 0 | CP5 context capsule-first 校验 OK（read_profile=compact，do_not_read_by_default 含完整设计/测试文档）。 |
| Agent Dispatch Evidence | valid | 0 | meta-dev 真实 subagent 调度（a78891926970bcbfa），记录于 AGENT-DISPATCH-LEDGER DISPATCH-CR035-CP5-meta-dev-001。 |

## Decision Brief

### 审批者摘要

| 字段 | 内容 |
|---|---|
| 本次确认服务的整体目标 | 确认 CR-035 全部 5 个 Story 设计证据（S01/S02/S03 full-lld + S04/S05 technical-note），授权进入 CP6 实现（Wave 1-4 调度）-> CP7 验证。 |
| 推荐动作 | `approve`：批量确认 5 Story 设计证据 + Wave 编排（S01->S03, S02->S03, S01/S02/S03->S05, S04 并行起草）。 |
| approve 后会发生什么 | host-orchestrator 自动推进 CP6 实现（Wave 1: S01 op_mapper + S04 起草；Wave 2: S02 main；Wave 3: S03 execute_steps；Wave 4: S05 测试 + S04 定稿），委托 meta-dev 实现；CP6 后 CP7 验证（dry-run + validate + 单测）；validation_mode=static-only + dry-run-only，无 runtime。 |
| approve 不授权什么 | 不授权真实设备 `--execute` 写操作（FU-01）、ptm-atomic/trex-api/trex-traffic 代码改动、24 用例 md 改动（R-C-008）、外部写入/生产/发布。设计证据确认不等于运行授权。 |
| 不确认会阻塞什么 | 阻塞 CP6 实现；5 Story LLD 未确认无法进入实现阶段。 |

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP5-CR035.context.json` |
| read_profile | compact |
| 默认读取策略 | 5 Story LLD 摘要 + HLD v0.2 + CP3 Decision Brief；不默认读完整 docs/design/HLD.md / TEST-MATRIX / TEST-REPORT / REVIEW（do_not_read_by_default）。 |
| 关键数字 | 5 Story（3 full-lld + 2 technical-note）/ 3 op 新注册 / 2 核心函数改 / 36 测试用例 / 0 新决策项 |
| 全文档读取 | LLD 已由 meta-dev 产出 + host-orchestrator lld-check 校验；P0/DQ-035-06/AGA 均已纳入。 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| 5 Story LLD | process/stories/STORY-035-S0*.md | scanned | 0 | 0 | 设计证据确认（无新决策项，HLD 已 CP3 approved） |
| HLD v0.2 | process/changes/CR-035-HLD.md | scanned | 0 | 0 | 架构方案 + 5 DQ + 4 AGA 已 CP2/CP3 approved |
| Wave 编排 | HLD Story 拆解 | scanned | 0 | 0 | S01->S03, S02->S03, ->S05, S04 并行（agent 默认） |

### 决策分层

| 分类 | 数量 | 处理方式 |
|---|---:|---|
| 必须用户决策 | 0 | 本门为设计证据批量确认，无新决策项（HLD/CP2/CP3 已 approved 全部决策） |
| 高风险策略确认 | 0 | 无 runtime/credential/production-write |
| agent 默认处理 | 1 | Wave 编排（S02/S03 同文件串行，S04 并行起草）|
| 仅审计记录 | 5 | S01-S05 设计证据 lld-check PASS + P0/DQ-035-06/AGA 纳入 |

### 待人工决策清单

本轮待人工决策项: 0

零决策原因：CP2/CP3 已闭环全部 6 项决策（DQ-035-01~06 approved + AGA-1/3/4 agent 默认 + P0 评审整改）。CP5 仅落实为 5 Story 设计证据（S01-S05，3 full-lld + 2 technical-note），无新范围/方案/授权决策。5 Story 的实现细节属 agent 默认处理（LLD 已明确 before/after + 伪代码 + 退出条件，approve 即授权 CP6 按 Wave 1-4 实现）。

### 5 Story 设计证据摘要

| Story | evidence_type | lld_policy | 文件 | 关键实现点 | open_items |
|---|---|---|---|---|---|
| S01 | full-lld | full-lld | `process/stories/STORY-035-S01-LLD.md` | OP_ID_TO_SUBCOMMAND +3 op（acquire-ports/release-ports/verify-ownership）；ARGS_TO_FLAGS +3 op + 6 现有 tg op 加 user_id；EXPECTED_OP_COUNT 23->26；build_command 无需改（L784-786 自动三层） | 无 Story 级 |
| S02 | full-lld | full-lld | `process/stories/STORY-035-S02-LLD.md` | **P0 整改**：acquire_ok 初始 True（与 login_ok L2292 对称），仅 --execute 失败置 False；_build_tg_user_id（dev{ip}-{hash4}，run_id 加 uuid 后缀）；_resolve_tg_ports（port_mapping 收集 .tg）；[4c]/[6c] 块；[5] 条件 `if login_ok and acquire_ok:`；result.json tg_port_ownership + config_once_skip_count 聚合 | O-035-02 runner_ip 来源（env_topology.runner_ip 优先 + socket fallback，dry-run 验证） |
| S03 | full-lld | full-lld | `process/stories/STORY-035-S03-LLD.md` | execute_steps 签名加 tg_run_ctx（AGA-1A，None 向后兼容）；config-once skip（§6.3，含 `and not dry_run` 守卫，**DQ-035-06 dry-run 不 skip**）；force_config 例外重置/成功置标志/失败不置（AGA-4）；tg_user_id 自动注入（TG_OPS_NEED_USER_ID 6 op） | 无 Story 级（force_config 插入行号实施时定位） |
| S04 | technical-note | technical-note | `process/stories/STORY-035-S04.md` | ptm-te.md TG 生命周期 config-once 三段 + Gotcha；case-execution SKILL.md config-once skip Gotcha（类比 #8）+ force_config + tg_port_ownership；trex-traffic SKILL.md 两层/三层命令差异 + owned vs legacy force | 无（S01-S03 实现后定稿） |
| S05 | technical-note | technical-note | `process/stories/STORY-035-S05.md` | FU-02 聚合占位符测试关闭；op_mapping 13 用例（EXPECTED_OP_COUNT==26 + 3 新 op + 6 tg op user_id）；config-once skip 12 用例（mock tg_run_ctx，5 分支）；dry-run 10 用例（acquire_ok True/不真实调用/tg_user_id 格式）；共 36 用例 | 无（mock 与 --execute 偏差由 CP7 N/A + 风险登记） |

### Wave 编排（agent 默认）

| Wave | Story | 说明 |
|---|---|---|
| 1 | S01（op_mapper）+ S04 起草 | S01 无依赖；S04 文档可并行起草 |
| 2 | S02（main） | 依赖 S01（op 注册） |
| 3 | S03（execute_steps） | 依赖 S01（user_id 映射）+ S02（tg_run_ctx 签名）；与 S02 同文件 case_runner.py 串行 |
| 4 | S05（测试）+ S04 定稿 | 依赖 S01/S02/S03 |

## Entry Criteria

| 条目 | 状态 | 证据 |
|---|---|---|
| CP3 approved | PASS | `process/checkpoints/CP3-CR035-HLD-REVIEW.md` |
| 5 Story LLD lld-check | PASS | 5 文件 lld-check OK（3 full-lld + 2 technical-note） |
| cp5-context-check | PASS | `process/context/CP5-CR035.context.json` |
| Agent Dispatch Evidence | PASS | AGENT-DISPATCH-LEDGER DISPATCH-CR035-CP5-meta-dev-001（meta-dev 真实 subagent） |
| P0/DQ-035-06/AGA 纳入 LLD | PASS | S02 acquire_ok 初始 True / S03 dry-run 不 skip / S03 tg_run_ctx |

## Checklist

| # | 检查项 | 审查结果 | 证据 |
|---|---|---|---|
| 1 | 5 Story 设计证据全覆盖（S01-S05） | PASS | process/stories/STORY-035-S0*.md |
| 2 | full-lld 15 章节 + 14 token（S01/S02/S03） | PASS | lld-check OK |
| 3 | technical-note 8 token（S04/S05） | PASS | lld-check OK |
| 4 | P0 acquire_ok 初始 True 纳入 S02 | PASS | S02 LLD §5.2/§6.1 |
| 5 | DQ-035-06 dry-run 不 skip 纳入 S03 | PASS | S03 LLD §5.1/§6.2 |
| 6 | AGA-1/3/4 纳入（tg_run_ctx/单标志/失败不置） | PASS | S02/S03 LLD |
| 7 | FU-02 关闭路径（S05） | PASS | S05 LLD 聚合占位符测试 |
| 8 | Wave 编排 + 文件所有权无冲突 | PASS | S01 独占 op_mapper；S02/S03 同文件区段不重叠串行；S04/S05 独占 |
| 9 | validation_mode=static-only + dry-run-only | PASS | S05 dry-run 测试 + CP7 N/A runtime |

## Exit Criteria

| 条目 | 审查结果 | 证据 |
|---|---|---|
| 自动预检无未豁免失败 | PASS | 5 lld-check + cp5-context-check OK |
| 用户终验确认 | PENDING | 本文件（待用户 approved） |

## Deliverables

| 交付物 | 路径 |
|---|---|
| S01 LLD | `process/stories/STORY-035-S01-LLD.md` |
| S02 LLD | `process/stories/STORY-035-S02-LLD.md` |
| S03 LLD | `process/stories/STORY-035-S03-LLD.md` |
| S04 technical-note | `process/stories/STORY-035-S04.md` |
| S05 technical-note | `process/stories/STORY-035-S05.md` |
| HLD v0.2 | `process/changes/CR-035-HLD.md` |
| CP5 context | `process/context/CP5-CR035.context.json` |
| Agent Dispatch 证据 | `process/state/AGENT-DISPATCH-LEDGER.ndjson` |

## 不授权范围

- 真实设备 `--execute` 写操作（FU-01 runtime 端到端，独立授权）。
- ptm-atomic CLI / trex-api / trex-traffic CLI 代码改动（已实现，本 CR 只消费）。
- 24 用例 md 改动（R-C-008，留 ptm-te workspace）。
- 外部写入、生产操作或发布。

## 回复方式

本次为 CP5 人工门禁（设计证据批量确认，0 新决策项）。请用以下三个 exact 回复之一：

- `approve` -- 确认 5 Story 设计证据 + Wave 编排，授权进入 CP6 实现（Wave 1-4 调度，委托 meta-dev）。
- `修改: <具体修改点>` -- 指出需调整的 Story/Wave（如"修改: S02 拆分为 S02a/S02b"），host-orchestrator 重新计算并重新发起确认。
- `reject` -- 不通过，说明原因。

> 内部兼容别名：`1/通过` = approve，`2/修改: ...` = 修改，`3/不通过` = reject。请勿混排。

## 修订记录

| 版本 | 日期 | 变更要点 |
|---|---|---|
| v1.0 | 2026-07-31 | 初始 Decision Brief：5 Story 设计证据批量确认（3 full-lld + 2 technical-note），0 新决策项 |

## 人工审查结果

**approved**（zhaohaibo，2026-07-31T10:55:00+08:00）

确认 5 Story 设计证据（S01/S02/S03 full-lld + S04/S05 technical-note）+ Wave 编排。评审 P2/P3 已修复：
- P2：S02 LLD §5.2 port_mapping 缺失 dry-run quick fail 加注释（缺 ports 无法 build_command）
- P3：S05 LLD test_config_once_skip_count_aggregation 从 test_tg_dry_run.py 移至 test_config_once_skip.py（语义属 config-once，计数 +12->+13 / +10->+9）

授权推进 CP6 实现（Wave 1-4，委托 meta-dev）-> CP7 验证 -> CP8 交付。validation_mode=static-only + dry-run-only，无 runtime。
- CP0/CP1/CP2/CP3 预检 PASS
- approve 不授权：真实设备 --execute（FU-01）、ptm-atomic/trex-api/trex-traffic 代码改动、24 用例 md（R-C-008）、外部写入/生产/发布

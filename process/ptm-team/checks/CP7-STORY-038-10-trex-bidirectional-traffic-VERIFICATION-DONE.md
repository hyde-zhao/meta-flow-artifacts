---
checkpoint_id: "CP7"
checkpoint_name: "验证完成检查"
type: "auto"
status: "PASS_WITH_RISK"
story_id: "STORY-038-10"
story_slug: "trex-bidirectional-traffic"
cr_id: "CR-038"
wave: 4
design_evidence_type: "technical-note"
validation_mode: "mixed"
executed_by: "meta-qa"
executed_at: "2026-08-15"
---

# CP7 验证完成检查 — STORY-038-10 trex 双向发流验证（loss=0）

## Entry Criteria

| 条目 | 结果 |
|---|---|
| Story `status=ready-for-verification` | PASS |
| CP6 编码完成门 PASS | PASS |
| technical-note 设计证据已确认 | PASS |
| 依赖门控（S03/04/05/06/13 hard）满足 | PASS |

## 验证对象清单

| 对象 | 文件 | 核对结果 |
|------|------|---------|
| `BidirectionalTrafficError`/`REQUIRED_TG1_MULTI_INSTANCE_FIELDS`/`load_yaml`/`validate_env_file`/`validate_traffic_template`/`load_traffic_template`/`build_bidirectional_plan`/`dry_run`/`main` | bidirectional_traffic.py | 与 technical-note 一致 |
| `fixtures/loopback_env_file.yaml` | tests/fixtures/ | tg1 多实例四字段 + 2 臂 + dut1 pppoe + sw1 |
| `fixtures/bidirectional_traffic_template.yaml` | tests/fixtures/ | udp/100pps/max_loss=0 + fwd/rev streams |
| `test_bidirectional_traffic.py`（24 例） | tests/ | 全绿 |

## 验证追踪矩阵

| 场景 | 结果 |
|------|------|
| SCN-038-POS-009（双向发流 loss=0） | PASS_WITH_RISK（dry-run 自检 + fixture 通过；真机 loss 结论 N/A） |
| SCN-038-NEG-003（未授权阻断） | PASS（`--execute` → EXECUTE_NOT_AUTHORIZED，exit=1） |
| 缺 tg1 多实例字段结构化报错 | PASS（parametrize 4 字段 + 负向 exit=1） |
| env-file 结构校验 | PASS |
| fixture 可加载 | PASS |

## 设计契约验证

- `REQUIRED_TG1_MULTI_INSTANCE_FIELDS`（trex_instance/trex_sync_port/trex_async_port/trex_api_url）齐全。
- `build_bidirectional_plan`：forward port1→port2 + reverse port2→port1。
- `--execute` 显式拒绝（EXECUTE_NOT_AUTHORIZED），默认 dry-run 不发真实流量，不读凭据。
- 与 return packet 一致：不修改 trex-traffic skill 现有 src/ 公开入口，不新增 env-file 字段/占位。

## 分层验证结果

- 单测：`skills/trex-traffic/tests/` 24/24 PASS。
- dry-run 自检（正向）：exit=0，双向 plan 生成（fwd 1/1/1→1/1/2，rev 1/1/2→1/1/1），notes 声明未发真实流量。
- `--execute`：EXECUTE_NOT_AUTHORIZED，exit=1。
- 负向（文件不存在）：ENV_FILE_NOT_FOUND 结构化报错，exit=1。
- 真机发流：N/A（runtime_authorization，未执行）。

## 功能 / 异常 / 回归 / 集成

- 功能：dry-run 验证资产（env-file 校验 + fixture 加载 + 双向 plan）。
- 异常：缺多实例字段/缺 tg1/dut1/缺顶层键/端口映射不完整/接口不匹配/fixture 结构错误 → 结构化报错。
- 回归：trex-traffic 此前无 tests/ 目录，无回归基线。
- 集成：dry-run 不校验 port_mapping.tg 物理端口 → TRex 逻辑端口映射（CP7 真机侧）；tg_config_interface 为 CP7 配置步骤（plan.notes 已标注）。

## 非功能

- 无凭据/真机/网络写；脚本默认 dry-run；GE1_1~4 禁改动（脚本不真机下发，无端口操作）。

## 缺陷 / 问题

- Return Packet 存在（STORY-038-10.return.json），无实现缺陷。

## 剩余风险

- trex 真机发流 loss=0（SCN-038-POS-009 最终判定）N/A，独立 runtime_authorization。
- port_mapping.tg 物理→逻辑端口映射、tg_config_interface 配置步骤为真机侧，dry-run 未覆盖。

## 阶段决策

**PASS_WITH_RISK** —— dry-run 验证资产完整，24 例全绿，`--execute` 拒绝 + 负向结构化报错，无凭据；真机发流 loss 结论 N/A 汇入 CP8。路由：host-orchestrator。

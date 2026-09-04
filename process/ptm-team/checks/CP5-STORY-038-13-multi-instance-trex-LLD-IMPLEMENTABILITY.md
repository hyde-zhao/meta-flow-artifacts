---
doc_type: CP5-CHECK
story_id: STORY-038-13
story_slug: multi-instance-trex
cr_id: CR-038
checkpoint: CP5
status: auto-prelim-pass
created_at: "2026-08-15"
owner: meta-dev
evidence: process/stories/STORY-038-13-multi-instance-trex-LLD.md
---

# CP5 设计证据可实现性自动预检 — STORY-038-13

## Entry Criteria

- [x] Story `status=lld-ready`，`lld_policy.required_level=full-lld`。
- [x] Feature 设计（DESIGN/TEST-PLAN/TASKS）可读且与 HLD §10.2/§10.3/§10.5 对齐。
- [x] ADR-CR038-02 / ADR-CR038-05 已存在。
- [x] 前序 Story（S02/S03/S06）file_ownership 与 merge_owner 关系明确。

## Checklist

| 项 | 结果 | 说明 |
|---|---|---|
| 14 个可见章节齐全 | PASS | 背景与目标/文件影响范围/接口设计/数据模型/核心流程/异常处理/测试设计/实施步骤/回滚策略/安全约束/契约映射/依赖与前置/风险与开放项/验收与交接 |
| 数据模型确定性 | PASS | `trex_instances` 块、`interfaces[].instance`、`InterfaceInfo.interface_kind`、`NodeInfo.trex_instance`、`UnifiedNode.trex_instances`、`UnifiedInterface.instance` 均给字段+默认值 |
| 接口设计确定性 | PASS | `get_trex_instances`/`get_trex_instance_of_port`/`get_free_ports`/`_resolve_env`/`_resolve_tg_instance` 均给签名+返回+失败语义 |
| 异常路径确定性 | PASS | 错误码 `TREX_INSTANCE_NOT_FOUND`/`TREX_INSTANCE_AMBIGUOUS` + 降级路径表（§6） |
| 测试设计对应接口 | PASS | §7 单测清单逐条覆盖 R-F-013~017 + SCN POS-010/011 + BND-003 |
| 实施步骤 + TASK-ID 对应 | PASS | TASK-E3-13-01~06 ↔ 5 文件 + tests，一一对应（§8/§11） |
| 回滚策略 | PASS | 逐文件回退 + 数据模型向后兼容 + 无台账副作用（§9） |
| 安全约束 | PASS | GE1_1~4 禁改、凭据占位、dry-run、确定性（§10） |
| 契约映射（需求/ADR→落地→验证） | PASS | R-F-013~017 + ADR-CR038-02/05 + ADR-09 + R-NF-004（§11） |
| 依赖与前置 | PASS | S02/S03/S06 门控 + CP4/CP5 门禁（§12） |
| 风险与开放项 | PASS | 4 风险 + 3 LCQ（均 blocks_lld=false）+ 3 灰区记录（§13） |
| 实现灰区与取舍记录 | PASS | §13.2 + QUESTION-LEDGER（LCQ-STORY-038-13-01/02/03） |
| Definition of Done | PASS | §14.1 8 项 DoD 可检验 |
| 文件所有权不冲突 | PASS | 本 Story 为 merge_owner；primary=topology_model/parser；shared=physical_pool(S03)/pool_merge(S02)/exporter(S06)，Wave 串行 |
| 未修改已 approve 对象 | PASS | 不触碰 REQUIREMENTS/HLD/ARCHITECTURE-DECISION/蓝图三件套 |

## 澄清项状态

| LCQ | blocks_lld | 默认方案 | 状态 |
|---|---|---|---|
| LCQ-STORY-038-13-01 | false | A（devices.yaml 嵌套 instances 块） | open |
| LCQ-STORY-038-13-02 | false | A（多实例未声明 → TREX_INSTANCE_AMBIGUOUS） | open |
| LCQ-STORY-038-13-03 | false | A（仅 tg1 节点级，不产 tg2 键名） | open |

## Exit Criteria

- [x] LLD 14 章节齐全且语义要点覆盖（数据模型/接口/异常/测试/实施/回滚/安全/契约/依赖/风险/DoD）。
- [x] TASK-ID ↔ 文件影响一一对应。
- [x] 无 blocks_lld=true 的澄清项。
- [x] 不依赖未确认设计证据即可完成可实现性判定。

## Deliverables

- `process/stories/STORY-038-13-multi-instance-trex-LLD.md`
- `process/state/QUESTION-LEDGER.ndjson`（3 条 LCQ）
- 本检查文件

## 结论

**PASS（auto-prelim）**：STORY-038-13 设计证据可实现性通过，等待 host-orchestrator 收齐全量目标 Story 设计证据后发起 CP5 统一人工确认。

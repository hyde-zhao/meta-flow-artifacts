---
doc_type: cp_check_result
id: CP5-CR037-S3-LLD-IMPLEMENTABILITY
story_id: "CR037-S3"
cr_id: "CR-037"
stage: CP5
checkpoint_type: auto
status: PASS
created_at: "2026-08-05"
owner: "meta-dev"
evidence_type: "full-lld"
evidence_path: "process/stories/STORY-CR037-S3-LLD.md"
---

# CP5-CR037-S3：设计证据可实现性自动预检

| 条目 | 结果 |
|------|:---:|
| lld_policy = full-lld 匹配 | PASS |
| 14 章节完整（0-15） | PASS |
| Goal 明确可量化 | PASS |
| Requirements 11 FR + 3 NFR 覆盖 | PASS |
| 模块拆分与职责清晰（merge_pool + 9 子函数） | PASS |
| 代码结构与文件影响范围明确（1 新增 + 3 只读消费） | PASS |
| 数据模型定义完整（UnifiedPool/UnifiedNode/UnifiedInterface/PoolData/DevicesData/LedgerData + 关联键表 + 台账三角映射） | PASS |
| API/Interface 契约完整（merge_pool + 加载器 + 内部关联函数） | PASS |
| 核心处理流程有 6 步流程图 + 错误路径表 | PASS |
| 技术细节（DUT host 关联 + TG api_server 关联 H2 + 台账三角映射 + 池全量承载降级） | PASS |
| 安全与性能分析（凭据不透传 + 只读 + yaml.safe_load + <200ms） | PASS |
| 测试设计（15 用例 + 5 fixture 文件） | PASS |
| 实施步骤 11 TASK + 12h 预估 | PASS |
| 风险 5 项 + 缓解 | PASS |
| 实现灰区 3 项 + 取舍记录（TG 关联键 P1-4 归一化关闭/台账数据来源/冲突优先级） | PASS |
| 回滚与发布策略 | PASS |
| DoD 12 项可验证 | PASS |
| 输出文件与 DEVELOPMENT-PLAN 一致 | PASS |
| 文件所有权无冲突 | PASS |
| 依赖 P-1 满足（P-1 模板化先于 S3 实施，design evidence 已完成） | PASS |
| tier="A" 合理 | PASS |
| open_items 无 | PASS |
| HLD §4.3 对齐（AG-3 归并策略 + H2 TG 归并） | PASS |
| UnifiedPool 输出与 S4 topo_mapper 消费契约对齐 | PASS |
| lld-check: LLD Structure Check | OK |

## 开放项

- **O-S3-01**: S3 依赖 P-1（physical_pool.yaml 模板化 schema），P-1 尚未实现。S3 实现前需确认 P-1 模板 schema 字段与 `UnifiedPool` 数据类字段映射一致，无需额外转换层。建议 S3 实施时先确认 P-1 产物格式。

## 修订说明（2026-08-05 设计审查修订）

- **P0-3**：台账消费改为现有 **list + `port_status`** schema（`LedgerData.devices` 为 list，`ledger_by_id` 索引），按 `device_id` 关联 pool node，不再按 node 名查 dict。
- **P1-3**：`_resolve_hardware_platform` 返回 hardware_platform **等价类**（如 HG3250，复用 S2 `platform_alias.yaml` 的 `equiv_class`）。
- **P1-4**：TG api_server 关联前 `_normalize_api_server` 归一化（去 scheme / 统一 IP:port），§6.5/§8.4/§13 灰区1 关闭。
- **P1-5**：`_merge_dut`/`_merge_tg`/`_check_devices_existence` 的 `node_id` 改用 `pool_node.get('node_id', key)`（key 为 pool.nodes dict key）。
- **P2-5**：台账 `model` 与池 `device_type` 可能同值直通；`UnifiedNode.ledger_model` 记为完整 device_type。

## 依赖门控说明

S3 对 P-1 为 `precedence`（前置）依赖——P-1 的 `physical_pool.yaml` 模板 schema 确定 S3 `load_pool` 的解析字段。CP5 阶段两 Story 设计证据均为 `lld-ready-for-review`，均可并行审批。实现阶段 P-1 先完成，S3 才能开始。

## 修订说明（2026-08-05，v1.2 CP5 第二轮复审 Minor-1 修订）

- **Minor-1 host 兜底伪代码**：§13 灰区1 内联循环「api_server + host 同判」改为**显式决策表（api_server 主键 → host 兜底 → unmatched）+ 两遍完整循环**（先遍历全部 candidates 找 api_server 主键，再遍历找 host 兜底），消除「首组 host 兜底误命中、遮蔽后续 api_server 主键候选」缺陷；实现语义与 §6.5 `_find_device_group_by_api_server` 对齐。
- §10 新增单测 `test_tg_host_fallback_decision_table`（candidate[0] api_server 不匹配但 host 匹配 + candidate[1] api_server 匹配 → 必须命中 api_server 主键，非首组 host 兜底）。
- lld-check 复跑 OK。

---
status: draft
version: "1.0"
cr_id: "CR-139"
source_scenarios: "process/SCENARIOS-CR139.yaml"
source_requirements: "process/REQUIREMENTS.md"
---

# Test Matrix — CR-139 Strategy Data Foundation

> CP2 阶段为覆盖规划矩阵，所有场景 `automation_status=planned`。CP5/CP6/CP7 由 meta-dev/meta-qa 落地真实测试文件。Wave3 P2 项标注 deferred。

## 覆盖矩阵

| Scenario ID | Requirement ID | 测试类型 | 自动化状态 | 手工验收 | 测试文件 / 命令（CP5 落地） | 未覆盖原因 |
|---|---|---|---|---|---|---|
| SCN-CR139-001 | REQ-212, REQ-213 | integration | planned | pending | `tests/test_cr139_t8_inventory.py` / `tests/test_cr139_t7_golden.py` | — |
| SCN-CR139-002 | REQ-204 | unit / regression | planned | pending | `tests/test_cr139_pit_asof_reader.py` | — |
| SCN-CR139-003 | REQ-204, REQ-226 | negative / regression | planned | pending | `tests/test_cr139_pit_lookahead_block.py` | — |
| SCN-CR139-004 | REQ-232, REQ-214 | integration | planned | pending | `tests/test_cr139_published_panel_reader.py` | — |
| SCN-CR139-005 | REQ-232 | negative / permission | planned | pending | `tests/test_cr139_unpublished_block.py` | — |
| SCN-CR139-006 | REQ-205 | unit / regression | planned | pending | `tests/test_cr139_read_dedup.py` | — |
| SCN-CR139-007 | REQ-205 | boundary | planned | pending | `tests/test_cr139_38run_dedup_boundary.py` | — |
| SCN-CR139-008 | REQ-248 | permission / security | planned | pending | `tests/test_cr139_no_authz_counters.py` | — |
| SCN-CR139-009 | REQ-219 | integration | planned | pending | `tests/test_cr139_feature_versioning.py` | — |
| SCN-CR139-010 | REQ-215 | negative | planned | pending | `tests/test_cr139_ml_bypass_block.py` | — |
| SCN-CR139-011 | REQ-221, REQ-222, REQ-225 | integration | planned | pending | `tests/test_cr139_manifest_modelhash_split.py` | — |
| SCN-CR139-012 | REQ-223, REQ-224 | negative / security | planned | pending | `tests/test_cr139_label_leakage_gate.py` | — |
| SCN-CR139-013 | REQ-243, REQ-244 | integration | planned | pending | `tests/test_cr139_incremental_pointer.py` | — |
| SCN-CR139-014 | REQ-234 | negative | planned | pending | `tests/test_cr139_readiness_gate_block.py` | — |
| SCN-CR139-015 | REQ-233, REQ-242 | integration / security | planned | pending | `tests/test_cr139_read_audit_runid.py` | — |
| SCN-CR139-016 | REQ-240, REQ-241 | integration | planned | pending | `tests/test_cr139_broker_facts_cost_gate.py` | — |
| SCN-CR139-017 | REQ-236, REQ-237, REQ-238 | integration | planned | pending | `tests/test_cr139_config_fact_source_versioning.py` | — |
| SCN-CR139-018 | REQ-220 | boundary | planned | pending | `tests/test_cr139_schema_freeze.py` | — |
| SCN-CR139-019 | REQ-235 | negative / security | planned | pending | `tests/test_cr139_decision_time_lookahead.py` | — |
| SCN-CR139-020 | REQ-229, REQ-230, REQ-231 | integration | planned | pending | `tests/test_cr139_cross_source_pit.py` | — |
| SCN-CR139-021 | REQ-246 | precheck / review | planned | pending | CP3 门禁集 review checklist | — |
| SCN-CR139-022 | REQ-247 | precheck / review | planned | pending | Wave1 N1 后置 review checklist | — |
| SCN-CR139-023 | REQ-243, REQ-244 | external-failure | planned | pending | `tests/test_cr139_dryrun_contract_not_executed.py` | — |
| SCN-CR139-024 | REQ-201 | boundary | planned | pending | `tests/test_cr139_runid_no_rename.py` | — |

## 需求→场景覆盖反向矩阵

| Requirement ID | 覆盖场景 | 覆盖状态 | 说明 |
|---|---|---|---|
| REQ-201 (N1) | SCN-CR139-024 | covered | D3 不重命名边界 |
| REQ-202 (N2) | — | planned (Wave2) | run_id 前缀规约，CP5 落地测试 |
| REQ-203 (N3) | — | planned (Wave2) | CR 编号路径规约，CP5 落地 |
| REQ-204 (C1) | SCN-002, 003 | covered | PIT as-of 正向 + 前视阻断 |
| REQ-205 (C2) | SCN-006, 007 | covered | 去重正向 + 边界 |
| REQ-206 (C3) | — | planned (Wave2) | events schema 修复，CP5 落地 |
| REQ-207 (C4) | — | planned (Wave2) | 写入去重保证，CP5 落地 |
| REQ-208 (M1) | — | planned (Wave2) | catalog/manifest 定主，CP5 落地 |
| REQ-209 (M2) | — | planned (Wave2) | lineage_checksum 回填，CP5 落地 |
| REQ-210 (M3) | — | deferred (Wave3 P2) | quality/ 分区整理 |
| REQ-211 (M4) | — | deferred (Wave3 P2) | CR→数据审计链 |
| REQ-212 (T7) | SCN-001 | covered | 黄金值基线 |
| REQ-213 (T8) | SCN-001 | covered | 自动化清册 |
| REQ-214 (R1) | SCN-004 | covered | panel reader |
| REQ-215 (R2) | SCN-010 | covered | ML 旁路阻断 |
| REQ-216 (R3) | — | planned (Wave2) | DuckDB 只读 adapter，CP3/CP5 批准后 |
| REQ-217 (R4) | — | deferred (Wave3 P2) | 列裁剪下推 |
| REQ-218 (V2) | — | planned (Wave2) | 训练快照，CP5 落地 |
| REQ-219 (V3) | SCN-009 | covered | feature 版本化层 |
| REQ-220 (V4) | SCN-018 | covered | schema 冻结 |
| REQ-221 (E1) | SCN-011 | covered | ExperimentManifest 闭环 |
| REQ-222 (E2) | SCN-011 | covered | model hash 闭环 |
| REQ-223 (E3) | SCN-012 | covered | label 泄漏 gate |
| REQ-224 (E4) | SCN-012 | covered | 离线/在线一致性 |
| REQ-225 (E5) | SCN-011 | covered | split manifest 冻结 |
| REQ-226 (T2) | SCN-003 | covered | PIT 正确性回归 |
| REQ-227 (T3) | — | planned (Wave2) | 去重正确性测试，CP5 落地 |
| REQ-228 (T1) | — | deferred (Wave3 P2) | DuckDB e2e 测试 |
| REQ-229 (X1) | SCN-020 | covered | 复权 PIT 校验 |
| REQ-230 (X2) | SCN-020 | covered | 跨源一致性 |
| REQ-231 (X4) | SCN-020 | covered | PIT universe 成分链 |
| REQ-232 (V1) | SCN-004, 005 | covered | published pointer 正向 + 阻断 |
| REQ-233 (L3) | SCN-015 | covered | 读审计 + run-id 贯通 |
| REQ-234 (L4) | SCN-014 | covered | 读前质量门禁 |
| REQ-235 (X3) | SCN-019 | covered | decision_time lookahead |
| REQ-236 (F1) | SCN-017 | covered | benchmark 版本化 |
| REQ-237 (F2) | SCN-017 | covered | commission 版本化 |
| REQ-238 (F3) | SCN-017 | covered | universe/risk policy 版本化 |
| REQ-239 (F4) | — | deferred (Wave3 P2) | 政策周期/shortability 版本化 |
| REQ-240 (T4) | SCN-016 | covered | broker facts 闭环 |
| REQ-241 (T5) | SCN-016 | covered | 成本前置门禁 |
| REQ-242 (T6) | SCN-015 | covered | run-id 贯通 |
| REQ-243 (L1) | SCN-013, 023 | covered | 增量 append + 降级 |
| REQ-244 (L2) | SCN-013, 023 | covered | pointer 前移 + 降级 |
| REQ-245 (L5) | — | deferred (Wave3 P2) | replay 单日重放 |
| REQ-246 | SCN-021 | covered | 四组门禁集 precheck |
| REQ-247 | SCN-022 | covered | 物理迁移后置 precheck |
| REQ-248 | SCN-008 | covered | 不授权范围计数 |
| REQ-249 | SCN-008 | covered | 既有合同闭环原则 |

## 覆盖统计

| 维度 | 总数 | covered | planned | deferred | gap | 说明 |
|---|---:|---:|---:|---:|---:|---|
| HIGH 场景 | 14 | 14 | 0 | 0 | 0 | 四道 P0 防线 + 基线 + ML 门禁 + 审计 + 门禁集 |
| 权限 / 安全场景 | 5 | 5 | 0 | 0 | 0 | 未授权计数/PIT 阻断/label 泄漏/lookahead/未发布阻断 |
| 外部失败场景 | 1 | 1 | 0 | 0 | 0 | dry-run 合同未执行降级 |
| P0 需求覆盖 | 8 | 8 | 0 | 0 | 0 | C1/C2/V1/R1/T7/T8 + 约束 246/247/248 |
| P2 deferred 需求 | 7 | 0 | 0 | 7 | 0 | M3/M4/R4/T1/F4/L5 + 部分 P2 |
| 总需求 | 49 | 33 | 9 | 7 | 0 | 33 直接覆盖，9 Wave2 planned，7 Wave3 deferred |

## 缺口处理

| Gap ID | 来源 | 缺口 | 阻断等级 | 推荐处理 | 责任方 |
|---|---|---|---|---|---|
| GAP-CR139-01 | REQ-202/203/206/207/208/209/218/227 | Wave2 P1 项暂无独立 SCN（覆盖于 UC） | OPTIONAL | CP5 落地时补 unit/integration 测试 | meta-dev / meta-qa |
| GAP-CR139-02 | REQ-210/211/217/228/239/245 | Wave3 P2 deferred 项无 SCN | OPTIONAL | 入 BACKLOG-CR139，Wave1/2 完成后按需 | meta-pm |

> 无 BLOCKING 缺口。P0 防线与门禁集全部覆盖；P1 项 CP5 落地测试；P2 项 deferred 到 Wave3。

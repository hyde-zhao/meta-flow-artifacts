---
checkpoint_id: "CP6"
checkpoint_name: "编码完成检查"
type: "auto"
status: "PASS"
story_id: "CR037-S10"
story_slug: "test-fixture-two-layer"
cr_id: "CR-037"
wave: 4
design_evidence_type: "technical-note"
executed_by: "meta-dev"
executed_at: "2026-08-05T00:00:00+00:00"
---

# CP6 编码完成检查 — CR037-S10 测试（层A fixture 验证 + 既有失败适配）

## 1. 实现对象清单

| 文件 | 操作 | 说明 |
|------|:---:|------|
| `tests/test_topo_plan_cr037.py` | **新建** | 层A 集成测试（20 用例）：collection 12 命中 100% / limit 解析（all 语义 + 3 链路模式 + 13 平台别名 + 空 limit）/ per-link TE 约束（成功 + 无 TE 结构化失败 + `--report-out`）/ ledger 闭环（allocate/避让/release）/ 约束叠加 + env-file `load_env_file` + `LINK_COUNT_MISMATCH` / install dry-run + Minor-4 case-sensitive / 参考用例 IPv4策略路由端到端 dry-run / 3 项专项单测（H4 TE+copper 不可满足 / include-at-least-one 软约束 / L2 透传不计入 LINK_COUNT_MISMATCH） |
| `skills/topo-config/tests/fixtures/pool_legacy.sample.yaml` | **新建** | 既有测试适配池 fixture（结构与 P-1 前真机池一致：FW1/FW2/SW1/SW2/PC1/PC2/TG1 + 11 cable；密码 scrubbed 为 test-placeholder） |
| `skills/topo-config/tests/fixtures/ledger_legacy.sample.yaml` | **新建** | 既有测试适配台账 fixture（device_id 201~207，全 free） |
| `skills/topo-planning/tests/fixtures/case_ipv4_policy_route.md` | **新建** | 参考用例 IPv4策略路由 PC fixture（组网描述 `TOPO-01 基础三链路 (node2_dut1_tg1_link3)`） |
| `skills/topo-planning/tests/fixtures/devices_direct.yaml` | **新建** | 参考用例端到端测试设备清单（匹配 pool_direct_tg_dut.yaml） |
| `skills/topo-config/tests/test_environment_manager.py` | 修改 | 数据源切到 `tests/fixtures/pool_legacy.sample.yaml` + `ledger_legacy.sample.yaml` |
| `skills/topo-config/tests/test_mapping_validator.py` | 修改 | 数据源切到 legacy fixture（同上） |
| `skills/topo-config/tests/test_topo_mapper.py` | 修改 | 数据源切到 legacy fixture（同上） |
| `skills/topo-config/tests/test_vlan_isolator.py` | 修改 | 数据源切到 legacy fixture（同上） |
| `skills/topo-config/tests/test_vrf_isolator.py` | 修改 | 数据源切到 legacy fixture（同上） |
| `skills/topo-config/config/physical_pool.yaml` | 修改 | **单行注释清理**（Minor-4 前置）：`device_type: ${DEVICE_TYPE_TG}` 注释删除示例型号 `IXIA-C`（CP5 S10 验收准则要求 config 模板不含 `IXIA-C` case-sensitive 特征串） |

## 2. 设计契约映射

| Story / HLD §11 契约 | 实现 |
|------|------|
| 层A 准则 1：12 拓扑 `find_topology_file` 命中率 100% | ✅ `test_collection_12_topos_hit_rate_100`（遍历 12 个 topo_id，返回 YAML 路径 + `parse_topology` 回读）；`test_collection_ref_topo_double_source`（组网约束/组网描述双源提取） |
| 层A 准则 2：limit 解析（all 语义 + 3 链路模式 + 13 平台别名 + 空 limit 默认） | ✅ `test_limit_all_semantics` / `test_limit_three_link_modes` / `test_limit_platform_aliases_13_series` / `test_limit_empty_default` |
| 层A 准则 3：per-link TE 约束（含 TE 成功 / 无 TE 结构化失败 + `--report-out`） | ✅ `test_per_link_te_constraint_success` / `test_per_link_te_constraint_no_te_failure_report` |
| 层A 准则 4：ledger 占用闭环（allocate 写回 / 二次映射避让 / release 释放） | ✅ `test_ledger_allocate_writeback_avoid_release`（tmp_path 台账副本，不写共享 fixture） |
| 层A 准则 5：约束叠加 + env-file `load_env_file` + `LINK_COUNT_MISMATCH` | ✅ `test_constraint_stack_priority` / `test_env_file_load_env_file_ok` / `test_link_count_mismatch` |
| 层A 准则 6：install dry-run（PTM_TE_SKILLS 7 + 模板副本不含造数 + `copy_skill_tree` 裁剪 tests/） | ✅ `test_install_ptm_te_skills_7_and_template_copy_no_fixtures` |
| 层A 准则 7：参考用例 IPv4策略路由（TOPO-01，fixture 池）端到端 dry-run | ✅ `test_reference_case_end_to_end_pipeline`（collection→find_topology_file→parse→limit(空)→pool_merge→match→exporter→env-file→`load_env_file`）+ `test_reference_case_cli_dry_run`（CLI `topo-plan plan --case-file` dry-run）+ `test_reference_case_cli_extract_topo_id` |
| Minor-4：模板占位化 case-sensitive 精确特征串断言 | ✅ `test_minor4_template_case_sensitive_assert`（config 模板 + install 模板副本扫描 8 个黑名单 token；**前置修复**：`physical_pool.yaml` TG 注释删除 `IXIA-C` 示例） |
| 专项 S10-A：H4 合并语义 TE+copper 不可满足 → 结构化失败 | ✅ `test_h4_te_copper_unsatisfiable`（TE+copper 池 + `mode=specific` 硬约束 → 有效需求 {TE,fiber} 不可满足 → failed + 报告含 `link_constraint_unsatisfied`） |
| 专项 S10-A：include-at-least-one 软约束（池仅 1 条 TE 链路） | ✅ `test_include_at_least_one_soft_constraint`（pool_direct 仅 link2 TE fiber → at least 1 命中即 matched） |
| 专项 S10-A：L2 透传 link 不计入 LINK_COUNT_MISMATCH | ✅ `test_l2_transparent_link_not_counted`（auto_inserted_switch 标记链路排除 + env-file links 不含该链路） |
| 层 B（可选）：目标项目真实数据验证 + 第 10 条 `--execute` 真实执行 | ⚠️ **N/A（S10-B）**：P-2 未就绪 → 层 B 标注未验证风险；`--execute` 真实执行依赖 DQ-037-04 独立 runtime_authorization，S10 只做 fixture 层 dry-run（0 次真机写） |

## 3. 测试结果

### 3.1 28 项既有失败适配

- **基线**（适配前）：`skills/topo-config/tests/` = **28 failed / 54 passed**（全部因 config/physical_pool.yaml 模板占位符，无法被 `PhysicalPool.get_devices_by_type` 匹配）。
- **适配方式**：新建 `pool_legacy.sample.yaml` + `ledger_legacy.sample.yaml`（结构与 P-1 前真机池一致），5 个既有测试文件数据源从 `config/physical_pool.yaml` 切到 fixture。
- **适配后**：`skills/topo-config/tests/` = **82 passed / 0 failed**（与用 P-1 前真实池跑出的 82 passed 完全一致，验证适配未改变测试语义）。

### 3.2 全量测试

命令：

```
uv run --python 3.11 python -m pytest skills/topo-config/tests/ skills/topo-planning/tests/ tests/test_topo_plan_cr037.py tests/test_install_templates.py tests/test_install_mapping.py
```

结果：**271 passed / 0 failed**（4.60s）

| 套件 | 结果 |
|------|------|
| `skills/topo-config/tests/` | 82 passed（含适配后既有 28 项） |
| `skills/topo-planning/tests/` | 141 passed |
| `tests/test_topo_plan_cr037.py`（S10 层A 集成） | 20 passed |
| `tests/test_install_templates.py` + `tests/test_install_mapping.py` | 28 passed |

### 3.3 参考用例 dry-run 链路结果

CLI `topo-plan plan --case-file skills/topo-planning/tests/fixtures/case_ipv4_policy_route.md --pool-path pool_direct_tg_dut.yaml --ledger-path ledger_direct.yaml --devices-yaml devices_direct.yaml --dry-run`：

- Step 2 选用逻辑拓扑 `node2_dut1_tg1_link3`（来源: **组网描述**，E2 双源提取）
- limit 为空（默认无约束）
- Step 1 物理池摘要：2 台设备（tg-dut1 TG / fw-dut1 DUT），6 端口 6 空闲
- Step 3 映射摘要：2 设备映射，3 链路直连，status=matched
- 导出 env-file：`metadata + port_mapping + nodes + links` 四段
- **case-execution `load_env_file` + `build_env_topology` 校验通过**（nodes: dut1/tg1，links: 3）
- dry-run 0 次真机写、不写台账（ledger_direct.yaml 保持全 free）

## 4. install dry-run 结果

`uv run --python 3.11 python script/install.py install claude --agent ptm-te --dry-run`：

- 安装 **7 个 skills**（device-management/device-connection/policy-route-execution/trex-traffic/case-execution/topo-config/topo-planning），均打印「排除 tests/__pycache__/*.pyc」（S8-A `copy_skill_tree` 裁剪）
- 生成模板副本 `topology/physical_pool.yaml` + `topology/physical_ledger.yaml`（DryRun）
- 追加 `.gitignore` 规则 `topology/*.yaml`
- 安装关联 resource：network-topology `tgfw-topo-collection`（collection.md + index.yaml）

## 5. 偏离与设计缺口反馈

| 偏离点 | 说明 |
|--------|------|
| **P-1 模板注释残留清理（Minor-4 前置，单行注释）** | `skills/topo-config/config/physical_pool.yaml` 第 159 行 TG1 `device_type` 注释含示例型号 `IXIA-C`，违反 CP5 S10 验收准则「config 模板不含 IXIA-C case-sensitive 特征串」（S10 技术说明 Minor-4 黑名单）。S8 的 `_MODEL_FEATURE_TOKENS` 不含 IXIA-C，故 install dry-run 不拦截。S10 Minor-4 严格断言必须清理该注释。属于 S10 技术说明「异常与回退」预设的 P-1 模板化遗留（Minor-3 漏清），以单行注释清理闭环，不改 schema/功能。 |
| **新建 legacy fixture（pool_legacy/ledger_legacy.sample.yaml）** | 既有 28 项失败因模板占位符无法匹配；P-1 fixture `physical_pool.sample.yaml` 的 SW 仅 1 口、无 TE fiber 直连，不满足既有测试语义（TG TE fiber 直连 / H3C SW 品牌 / PC GE copper）。S10 新建 legacy fixture（结构与 P-1 前真机池一致，密码 scrubbed）并让既有测试消费，保留原测试意图。 |
| **测试数据源修改 5 个既有测试文件** | S10 output_files 声明 `skills/topo-planning/tests/` + `tests/test_topo_plan_cr037.py`；适配 28 项失败必须修改 `skills/topo-config/tests/*.py` 的 POOL_PATH/LEDGER_PATH（数据源），属任务显式授权的「适配既有测试断言/数据源」，不改变断言语义。 |
| 层 B 第 10 条 N/A | P-2 未就绪 → 层 B 标注未验证风险，不阻断 CP7 fixture 结论；`--execute` 真实执行依赖 DQ-037-04 独立授权（S10-B）。 |

## 6. 已知限制

- 层 B（目标项目真实池映射）未执行：P-2 目标项目 `topology/physical_pool.yaml` 未填写真实数据。
- S9 `DeployBridge.deploy(dry_run=True)` 由 S9 CP6 单测覆盖（test_deploy_bridge.py 8 用例），S10 集成测试未重复（引用 S9 单测结论）。

## 7. 后续交接

- 交付 meta-qa：
  - 验证入口：`tests/test_topo_plan_cr037.py`（20 用例）、`skills/topo-config/tests/`（82）、`skills/topo-planning/tests/`（141）
  - 参考用例 dry-run：`topo_plan_cli.py plan --case-file case_ipv4_policy_route.md ... --dry-run`（链路打通，env-file 通过 case-execution `load_env_file`）
  - 风险提示：层 B P-2 未就绪（未验证风险）；`--execute` 真机执行 N/A（DQ-037-04）
- S11 消费：README / 执行指导已按实现回填。

## 8. 结论

编码完成，CP6 PASS。S10 层A 8 条完成准则 + 3 项专项单测全部覆盖并通过；28 项既有失败适配后全量 271 passed / 0 failed；参考用例端到端 dry-run 链路打通。等待 meta-qa CP7 验证。

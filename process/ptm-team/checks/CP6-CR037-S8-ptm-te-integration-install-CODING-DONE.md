---
checkpoint_id: "CP6"
checkpoint_name: "编码完成检查"
type: "auto"
status: "PASS"
story_id: "CR037-S8"
story_slug: "ptm-te-integration-install"
cr_id: "CR-037"
wave: 3
design_evidence_type: "full-lld"
executed_by: "meta-dev"
executed_at: "2026-08-05T00:00:00+00:00"
---

# CP6 编码完成检查 — CR037-S8 ptm-te 集成 + install.py

## 1. 实现对象清单

| 文件 | 操作 | 说明 |
|------|:---:|------|
| `script/install.py` | 修改 | `PTM_TE_SKILLS` 5→7（增 `topo-config` + `topo-planning`，方案 A）；`copy_skill_tree` 过滤复制（排除 `tests/`/`__pycache__/`/`*.pyc`/`*.pyo`，S8-A）；新增 `ensure_target_project_templates` + `_ensure_gitignore_rule` + `_assert_p1_template_placeholderized`（P1-8 + Minor-2）；`install_agent` 接线 ptm-te 模板副本生成 |
| `script/ptm_team/install.py` | 修改（保持一致扩展） | `PTM_TE_SKILLS` 5→7 + `copy_skill_tree` 同一裁剪语义（两 install.py 一致，避免安装行为漂移） |
| `agents/ptm-te.md` | 修改 | 编排 [1]→[2] 之间插入 [1.5] 拓扑映射（E1）；topo_id 双源提取说明（E2）；[7] 清理挂接 `PhysicalPool.release` + `save_ledger`（P1-7）；关联 Skill 表增 `topo-config` + `topo-planning`；运行时工作目录增 physical_pool/physical_ledger/preview env-file；修订记录 v1.9 |
| `resource/component-resource-links.yaml` | 修改 | 新增 `ptm-te` 组件 → network-topology `tgfw-topo-collection`（required，S8-C/FR12） |
| `tests/test_install_templates.py` | 新建 | 16 个 pytest 单元测试（对齐 LLD §10 清单） |
| `tests/test_install_mapping.py` | 修改 | `test_ptm_te_skills_consistent` 断言更新为 7 个 skill（CR-037 需求变更） |

## 2. 设计契约映射

| LLD / Story 契约 | 实现 |
|------|------|
| `PTM_TE_SKILLS` 5→7 增 topo-config + topo-planning（FR6/方案 A，AC3） | ✅ 两 install.py 同步；`install --agent ptm-te --dry-run` 打印 7 个 skill |
| `copy_skill_tree` 裁剪 tests/__pycache__/*.pyc（S8-A/FR7，AC5） | ✅ `shutil.ignore_patterns("tests", "__pycache__", "*.pyc", "*.pyo")`；`test_copy_skill_tree_strips_tests` 断言目标无 tests//__pycache__/*.pyc，SKILL.md/src 保留 |
| `ensure_target_project_templates` 生成模板副本（P1-8/FR7-FR8，AC4） | ✅ 源 = `skills/topo-config/config/{physical_pool,physical_ledger}.yaml`；幂等跳过已有；`topology/` 目录创建；返回 actions 清单 |
| **P-1 占位化前置校验**（Minor-2，AC4） | ✅ `_assert_p1_template_placeholderized`：源含 `A1600-HU`/`hg3250` 等特征串 → fail-fast（RuntimeError）；dry-run 同样执行；`test_generate_templates_requires_p1_placeholder` 验证 |
| `.gitignore` 追加 `topology/*.yaml`（FR9/S8-F，AC4） | ✅ `_ensure_gitignore_rule` 精确匹配单条规则，已有则跳过；保留既有规则（`test_gitignore_preserve_existing`） |
| 模板副本不含测试造数 + 台账 `list + port_status`（FR10/P0-3，AC4） | ✅ `test_generate_templates_no_fixtures` 断言无 hg3250/A1600-HU；`test_ledger_template_schema` 断言 devices list + allocated_to/allocated_vlan |
| install `--dry-run` 不写文件（FR11） | ✅ `test_dry_run_no_write` 断言 topology/ 与 .gitignore 不存在 |
| ptm-te → network-topology 资源关联（S8-C/FR12，AC7） | ✅ `component-resource-links.yaml` 新增 ptm-te 条目；`test_ptm_te_resource_link/installed` 断言 `get_component_resources("ptm-te")` 返回 tgfw-topo-collection required；`install --dry-run` 打印复制 collection.md |
| 编排 [1.5] 拓扑映射（E1/FR2-FR4，AC1-2） | ✅ ptm-te.md 编排流程插入 [1.5]：E2 双源提取 + 调 S7 CLI + execute/dry-run 分支 + `TOPO_MAPPING_FAILED` 终止 |
| dry-run 落预览 env-file（S8-B/FR3，AC6） | ✅ ptm-te.md [1.5] dry-run 分支：`--output runs/<run-id>/topology.preview.yaml` 落预览副本（executable:false），下游 load_env_file 可解析；S7 CLI `_is_preview_output` + `_write_preview_copy` 实现 |
| release 挂接 [7]（P1-7/FR5，AC8） | ✅ ptm-te.md [7] 新增 release 台账占用（`pool.release(env_name)` + `save_ledger`；仅 execute 且使用过 topo-planning；dry-run 跳过；失败记 cleanup_summary 不阻断 [8]） |
| 授权分层（S8-E/AC9） | ✅ ptm-te.md [1.5] 授权分层：映射+台账占用=执行前自动步骤（S7 `--authorized`，不需 DQ-037-04）；真机下发（S9）才需 DQ-037-04 |

## 3. 单元测试结果

命令：`uv run --python 3.11 python -m pytest tests/test_install_templates.py tests/test_install_mapping.py`

结果：**test_install_templates.py 16 passed** + **test_install_mapping.py 全绿**（含更新后的 `test_ptm_te_skills_consistent`）。

覆盖要点：
- `test_ptm_te_skills_includes_topo`（FR6）
- `test_generate_templates_requires_p1_placeholder` / `test_assert_p1_placeholder_passes_for_real_template`（Minor-2）
- `test_generate_templates_first_run` / `test_generate_templates_idempotent` / `test_generate_templates_no_fixtures`
- `test_ledger_template_schema` / `test_pool_template_placeholder`
- `test_gitignore_append` / `test_gitignore_idempotent` / `test_gitignore_preserve_existing`
- `test_dry_run_no_write`
- `test_copy_skill_tree_strips_tests` / `test_copy_skill_tree_dry_run`（S8-A）
- `test_ptm_te_resource_link` / `test_ptm_te_resource_installed`（S8-C）

## 4. install dry-run 结果

命令：`uv run --python 3.11 python script/install.py install claude --agent ptm-te --dry-run`

结果：**成功**（dry-run，未改真实环境）
- 安装 7 个 skills（device-management/device-connection/policy-route-execution/trex-traffic/case-execution/topo-config/topo-planning），均打印「排除 tests/__pycache__/*.pyc」
- 打印模板副本 DryRun：`topology/physical_pool.yaml` + `topology/physical_ledger.yaml`
- 打印 `.gitignore` DryRun 追加 `topology/*.yaml`
- 安装关联 resource：network-topology `tgfw-topo-collection`（`topology-collection.md` + `index.yaml` 到 `~/.ptm-team/resource/network-topology/`）

## 5. 偏离 LLD 设计点

1. **同步修改 `script/ptm_team/install.py`**（LLD scope 只列 `script/install.py`）：既有测试 `test_ptm_te_skills_consistent` 断言两 install.py 的 ptm-te skills 一致。为保持安装行为一致 + 造数隔离承诺（S8-A）在两安装路径都成立，同步更新其 `PTM_TE_SKILLS` 与 `copy_skill_tree`。属于「保持一致性」的合理扩展，不改变任何 Story 边界。
2. **`test_install_mapping.py::test_ptm_te_skills_consistent` 断言更新**：原断言精确等于 5 个 skill（CR-036 预存）。CR-037 需求变更（5→7），断言更新为 7 个。属需求变更驱动的测试基线更新。

## 6. 已知限制

- `script/ptm_team/install.py` 未加 `ensure_target_project_templates`（模板副本生成仅在 `script/install.py` 主安装器；ptm_team/install.py 为包级兼容入口，模板副本逻辑不在其 scope）。
- 模板副本源依赖 P-1 占位化：若源含真实型号特征串，`_assert_p1_template_placeholderized` fail-fast（已验证通过）。

## 7. 后续交接

- 交付给 meta-qa：`script/install.py` / `agents/ptm-te.md` / `resource/component-resource-links.yaml` 修改 + 16 单测 + install dry-run 结果。
- S9 消费：ptm-te 编排 [1.5] 映射后，真机部署由 S9 `deploy_bridge` 承接（独立 DQ-037-04）；[7] release 挂接由 S8 承载。
- S10 集成验证入口：`install --agent ptm-te` 后 `~/.ptm-team/resource/network-topology/topology-collection.md` 存在；`.claude/skills/topo-config/` 无 tests/（S8-A）；模板副本无造数。

## 8. 结论

编码完成，CP6 PASS。等待 meta-qa CP7 验证。

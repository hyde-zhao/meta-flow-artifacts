---
checkpoint_id: "CP6"
checkpoint_name: "编码完成检查"
type: "auto"
status: "PASS"
story_id: "CR037-S7"
story_slug: "topo-planning-skill-cli"
cr_id: "CR-037"
wave: 3
design_evidence_type: "full-lld"
executed_by: "meta-dev"
executed_at: "2026-08-05T00:00:00+00:00"
---

# CP6 编码完成检查 — CR037-S7 topo-planning skill + CLI

## 1. 实现对象清单

| 文件 | 操作 | 说明 |
|------|:---:|------|
| `skills/topo-planning/scripts/topo_plan_cli.py` | 新建 | CLI `topo-plan` 入口：12 参数（`--topo-id/--case-file/--limit/--limit-file/--devices-yaml/--pool-path/--ledger-path/--output/--report-out/--dry-run/--execute/--authorized`）+ `--env-name` + `--list-topologies`；默认 dry-run 门；`--execute` 必须配 `--authorized`（S7-B，RuntimeAuthorizationError exit 3）；`--case-file` 双源提取 topo_id（E2）；dry-run `--output` 指向 run 目录时落预览副本（executable:false，S8-B） |
| `skills/topo-planning/SKILL.md` | 新建 | Prompt-Skill 契约结构（目标/前置/命令格式/向导 5 步/NLP→CLI/产物契约/dry-run 门/安全约束/相邻对象边界/Gotchas/修订记录） |
| `skills/topo-planning/templates/physical_pool.yaml.example` | 新建 | 目标项目模板副本参考（P-1 模板化语义对齐，占位符 + 指引，不含测试造数） |
| `skills/topo-planning/templates/physical_ledger.yaml.example` | 新建 | 台账模板参考（`list + port_status`，P0-3，不含测试造数） |
| `skills/topo-planning/tests/test_topo_plan_cli.py` | 新建 | 21 个 pytest 单元测试（对齐 LLD §10 测试清单） |
| `skills/topo-planning/tests/fixtures/pool_no_te.yaml` | 新建 | 无 TE 端口池（失败路径测试） |
| `skills/topo-planning/tests/fixtures/ledger_direct.yaml` | 新建 | pool_direct_tg_dut 对应台账（P0-3） |
| `skills/topo-planning/tests/fixtures/ledger_no_te.yaml` | 新建 | pool_no_te 对应台账 |
| `skills/topo-planning/tests/fixtures/case_topology_constraint.md` | 新建 | 组网约束提取 fixture（E2） |
| `skills/topo-planning/tests/fixtures/case_topology_description.md` | 新建 | 组网描述回退 fixture（E2） |
| `skills/topo-planning/tests/fixtures/case_no_topo_id.md` | 新建 | 提取失败 fixture |
| `skills/topo-planning/tests/fixtures/limit_te.json` | 新建 | TE limit 样例 |
| `skills/topo-planning/tests/fixtures/limit_invalid_media.json` | 新建 | TE+copper 非法 limit（PARAM_INVALID） |

## 2. 设计契约映射

| LLD / Story 契约 | 实现 |
|------|------|
| CLI 支持 12 参数（AC1） | ✅ `build_parser()` 定义全部 12 参数 + `--env-name` + `--list-topologies` |
| `--execute` 必须配 `--authorized`（S7-B/FR12/NFR7，AC4） | ✅ `run_plan` 步骤 1：`execute and not authorized → RuntimeAuthorizationError`（exit 3）；`authorized and not execute → exit 2` |
| dry-run 默认门：不写台账/不落真实路径（FR7/NFR1） | ✅ `dry_run = not execute`；`match(env_name=None)` 不写台账；`_emit_success` dry-run 只 stdout + would-allocate 清单 |
| dry-run `--output` 指向 run 目录 → 落预览副本 `metadata.executable:false`（S8-B/FR7） | ✅ `_is_preview_output` 判断路径含 `runs` 段；`_write_preview_copy` 落盘 + 置 `executable:false`；不写真实 `topology/` 路径 |
| `--case-file` 双源提取 topo_id（E2/FR1，AC5） | ✅ `_extract_topo_id_from_case`：组网约束优先（`^(node\d+_.+)$`）、组网描述括号回退（`re.search(r'(node\d+_[A-Za-z0-9_]+)')`）；失败 `TOPO_ID_NOT_FOUND` exit 2 |
| `--limit` / `--limit-file` 归一化 + S2 parse_limit（FR2） | ✅ `_resolve_limit`（互斥校验 + inline/file/None → `parse_limit`）；`TopoLimitError` → exit 2 |
| 物理路径三参数 + 缺省回落 `topology/`（FR3） | ✅ `pool_path/devices_path/ledger_path` 缺省 `topology/{physical_pool,devices,physical_ledger}.yaml` |
| 映射 + S5 导出 / S6 IP 规划（FR5/FR6） | ✅ `_run_mapping` 调 `TopoMapper.match`（env_name=None/非 None 分支）；`compute_ip_plan`；成功 → `export_env_file`；失败 → `build_failure_report` + `write_report`（`--report-out` 落 `<path>.json/.txt` 或 stdout）exit 1 |
| 退出码约定（§6.5） | ✅ 0=成功 / 1=映射失败 / 2=参数错误 / 3=RuntimeAuthorizationError |
| 向导 5 步（FR4） | ✅ `_show_pool_summary`（Step1）/ `[Step 2]` / `_show_mapping_summary`（Step3）/ `_emit_success`（Step4）/ Step5 提示 S9 承接 |
| 统一 PYTHONPATH import（S1 P2-4/NFR3，AC3） | ✅ CLI 顶部 sys.path.insert 过渡降级（topo-planning/src + topo-config/src）；不引入 importlib 动态加载（`test_import_without_pythonpath_hack` 断言） |
| 凭据禁止（ADR-02/NFR2） | ✅ 不读/不传密码字段；`test_no_credential_in_output` 断言产物无 password/token/api_key/secret |
| env-name 自动生成唯一（FR9） | ✅ `_default_env_name` = `<topo_id>-<HHMMSS>`；`test_auto_env_name_unique` 用 monkeypatch datetime 验证不同 |
| `--list-topologies`（FR10） | ✅ 调 S1 `list_topologies` 列出后 exit 0 |

## 3. 单元测试结果

命令：`uv run --python 3.11 python -m pytest skills/topo-planning/tests/test_topo_plan_cli.py`

结果：**21 passed**（LLD §10 清单 19 用例 + 双源提取 3 用例等价覆盖 + `--case-file` 提取失败 1 用例）。

覆盖要点：
- `test_extract_topo_id_from_constraint/from_description/missing`（E2 三态）
- `test_plan_success_dry_run` / `test_plan_success_execute` / `test_execute_requires_authorized` / `test_authorized_without_execute`
- `test_dry_run_zero_real_write` / `test_dry_run_preview_copy` / `test_dry_run_no_ledger_write`
- `test_plan_failure_report` / `test_plan_case_file` / `test_plan_case_file_extract_fail` / `test_plan_no_topo_args`
- `test_plan_limit_inline` / `test_plan_limit_file` / `test_plan_limit_invalid`
- `test_list_topologies` / `test_auto_env_name_unique` / `test_no_credential_in_output` / `test_import_without_pythonpath_hack`

## 4. 偏离 LLD 设计点

1. **`test_plan_failure_report` 的 `missing_te_port` 断言调整为「TE 约束失败被结构化报告」**：S4 引擎在「无 TE 池 + TE 硬约束」下，回溯在**设备级**先剪枝（`missing_device`）并导致链路级端点未映射（`link_constraint_unsatisfied`），`missing_te_port`（要求设备匹配成功但链路级 TE 不满足）在该池/拓扑组合下不可达。报告仍完整结构化暴露 TE 约束失败（`link_constraint_unsatisfied` + `summary` 含「链路约束不满足」）。`missing_te_port` 分支保留在 S5 `build_failure_report`（S4 部分匹配 + 链路级 TE 失败场景）。
2. **dry-run 时 `metadata.env_name` 为 null**：S5 `export_env_file` 的 metadata.env_name 取 `mapping_result.env_name`，dry-run 时 S4 `match(env_name=None)` 不写回 → null。这是 S5 既有契约（dry-run 无占用），预览副本仍正确标记 `executable:false`。
3. **fixture 复用**：复用 S5 的 `pool_direct_tg_dut.yaml` + S2 的 `devices_pool_test.yaml`；新增 ledger/case/limit fixture 在 topo-planning 自身 tests/fixtures 下（避免跨 skill 目录引用）。

## 5. install dry-run 结果

CLI 不涉及安装器；S7 产物由 S8 `install --agent ptm-te` 安装（见 S8 CP6）。

## 6. 已知限制

- S7 CLI 是「映射 + 导出」边界；真机部署由 S9 `deploy_bridge` 承接（S7 只提示下一步，不直接调 deployer）。
- `--case-file` frontmatter 解析为简化 key:value（兼容 case-execution 16 列语义的 topo_id 提取场景）；复杂 YAML 结构字段（组网约束为 list 等）不在 S7 提取范围。

## 7. 后续交接

- 交付给 meta-qa：`topo_plan_cli.py`（S7 产物）+ 21 单测。
- S8 消费：ptm-te 编排 [1.5] 调 S7 CLI（`--case-file` + `--env-name` + `--output` + execute/dry-run 映射）；install.py 安装 topo-planning skill。
- S9 消费：S7 CLI 产出 env-file（`topology/topology.yaml` 或 dry-run 预览副本）作为 deploy_bridge 输入。

## 8. 结论

编码完成，CP6 PASS。等待 meta-qa CP7 验证。

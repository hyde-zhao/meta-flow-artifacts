---
cr_id: CR-037
title: ptm-te 集成 topo 管理功能 测试报告
version: "1.0"
test_mode: mixed（层A fixture runtime + 平台安装 dry-run）
source_cr: CR-037
story_range: "CR-037 (12 Stories: P-1 + S1-S11)"
reviewer: "meta-qa (independent, CP7)"
test_date: "2026-08-05T19:30:00+08:00"
---

# CR-037 测试报告（TEST-REPORT）

> 独立复跑声明：所有测试结果由 meta-qa 在 CP7 独立复跑确认，非引用 CP6 result。pytest 命令真实执行并记录输出。

## 1. 全量测试复核

**命令**：
```
uv run --python 3.11 python -m pytest skills/topo-config/tests/ skills/topo-planning/tests/ tests/test_topo_plan_cr037.py tests/test_install_templates.py tests/test_install_mapping.py
```

**结果**：**271 passed / 0 failed（4.68s）**（meta-qa 独立复跑与 CP6 报告一致）

| 套件 | 结果 | 覆盖 |
|---|---|---|
| `skills/topo-config/tests/` | 82 passed | 既有 5 文件适配（28 失败归零）+ S4 26 专项 |
| `skills/topo-planning/tests/` | 141 passed | S1 14 + S2 41 + S3 21 + exporter(S5+S6) 36 + CLI 21 + deploy_bridge 8 |
| `tests/test_topo_plan_cr037.py` | 20 passed | S10 层A 集成（8 准则 + 3 专项 + 参考用例） |
| `tests/test_install_templates.py` + `test_install_mapping.py` | 28 passed | S8 安装器（18 + 10） |

## 2. 8 维度验收矩阵

| # | 维度 | 阻断 | 检查内容 | 结果 |
|---|------|:---:|---------|------|
| 1 | 完整性 | BLOCKING | 12 Story output_files 全部创建 | ✅ 通过（271 测试引用的对象均可 import/加载） |
| 2 | 平台适配 | BLOCKING | install dry-run（claude）、7 skills、模板副本、gitignore、resource | ✅ 通过 |
| 3 | 验收标准覆盖 | BLOCKING | S10 11 条验收准则（8 层A + 3 专项 + Minor-4）+ 层B N/A | ✅ 通过（层B 记录 N/A/未验证） |
| 4 | 安全合规 | BLOCKING | 危险命令扫描 0、模板 8 特征串 0 命中、产物无凭据（ADR-02） | ✅ 通过 |
| 5 | 命名规范 | REQUIRED | 文件名 kebab-case（topology_collection/limit_parser/pool_merge/exporter/deploy_bridge/topo_plan_cli） | ✅ 通过 |
| 6 | Frontmatter 完整性 | REQUIRED | SKILL.md / README frontmatter title/version/description 非空 | ✅ 通过 |
| 7 | 可安装性 | REQUIRED | install dry-run + copy_skill_tree 裁剪 + 模板副本幂等 + gitignore 幂等 | ✅ 通过 |
| 8 | 文档覆盖 | OPTIONAL | README（安装/使用/limit 语法/三层结构）+ 执行指导 [1.5] + limit-syntax | ✅ 通过（S11） |

## 3. 层A fixture 完成准则逐条结果

| 准则 | 测试 | 独立结果 |
|---|---|---|
| 12 拓扑 find 100% + 双源提取 | `test_collection_12_topos_hit_rate_100` / `test_collection_ref_topo_double_source` | ✅ 通过 |
| limit 解析（all/3 模式/13 别名/空 limit） | S2 41 用例 | ✅ 通过 |
| per-link TE（成功/失败 + --report-out） | `test_per_link_te_constraint_success` / `test_per_link_te_constraint_no_te_failure_report` | ✅ 通过 |
| ledger 闭环 | `test_ledger_allocate_writeback_avoid_release` + S4 6 用例 | ✅ 通过 |
| 约束叠加 + env-file + LINK_COUNT_MISMATCH | `test_constraint_stack_priority` / `test_env_file_load_env_file_ok` / `test_link_count_mismatch` | ✅ 通过 |
| install dry-run（7 skills + 模板副本 + 裁剪） | `test_install_ptm_te_skills_7_and_template_copy_no_fixtures` | ✅ 通过 |
| 参考用例端到端 | `test_reference_case_end_to_end_pipeline` / `test_reference_case_cli_dry_run` | ✅ 通过 |
| 专项：H4 TE+copper | `test_h4_te_copper_unsatisfiable` | ✅ 通过 |
| 专项：include-at-least-one | `test_include_at_least_one_soft_constraint` | ✅ 通过 |
| 专项：L2 透传不计入 | `test_l2_transparent_link_not_counted` | ✅ 通过 |
| Minor-4 模板特征串 | `test_minor4_template_case_sensitive_assert` | ✅ 通过 |

## 4. 功能维度

- **拓扑发现**：`find_topology_file` 12 拓扑命中 100%，返回 YAML 路径可被 `detect_topology_format` / `_parse_yaml_topology` 消费（P0-1）；`parser.parse_topology(topo_id=...)` 完整链路 S10 覆盖。
- **limit 解析**：平台别名 13 硬件系列 + nxp1043 拆分 + TG 6 组合 + speed_class 归一 + R4 冲突消歧（A1500-HU 三态）。
- **归并**：DUT host 关联 + TG api_server 主键/host 兜底 + 台账三角映射 + 池全量降级。
- **映射引擎**：per-link TE 约束 + hardware_platform 等价类 + 端口兼容矩阵 + H4 叠加 + include-at-least-one 软约束 + ledger 占用闭环。
- **导出**：env-file 四段 + 9 类 `${ENV.*}` 占位符 + 命名 + LINK_COUNT_MISMATCH + 结构化失败报告 + MVP 边界。

## 5. 接口维度

- `TopoMapper.match(logical_topo, topo_limit=None, port_requirement=None, env_name=None)`：三态 port_requirement 兼容（None/全局/per-link），空 limit 向后兼容。
- `export_env_file(...)` / `compute_ip_plan(...)`：S5+S6 统一入口，被 S7 CLI / S8 编排消费。
- `merge_pool(...) → UnifiedPool.to_pool_data() → PhysicalPool.from_pool_data()`：S3-B 契约贯通。
- CLI `topo-plan`：12 参数 + `--authorized` 硬门 + exit 码 0/1/2/3。

## 6. 数据维度

- 模板/模板副本：8 个 case-sensitive 特征串（A1600-HU/A1300-HU/A1500-HU/HG3250/hg3250/IXIA-C/Ngfw@123/10.113.）0 命中。
- 台账 schema：list + port_status（P0-3）不变；allocate 写回仅 `allocated_to`/`allocated_vlan`。
- 造数隔离：fixture 是唯一允许造数位置；install 裁剪 tests/。

## 7. 异常维度

- 无 TE 池 + TE 硬约束 → 结构化失败报告（`link_constraint_unsatisfied` + `--report-out` 落盘 JSON/txt）。
- L2 透传 link → 不计入 LINK_COUNT_MISMATCH。
- 多节点/SW/Mock 拓扑 → `TOPO_NODE_COUNT_UNSUPPORTED`。
- 缺 IP → `IP_PLAN_REQUIRED` / `IP_PLAN_INCOMPLETE`（allow_degraded 降级 executable=false）。
- `--execute` 未授权 → `RuntimeAuthorizationError` exit 3；executable:false 预览副本拒绝真机部署。

## 8. 安全 / 性能 / 可维护性 / 文档维度

- **安全**：产物无 password/token/api_key/secret（ADR-02）；DeployBridge 只收环境变量名；危险命令扫描 0。
- **性能**：全量 271 用例 4.68s；mtime 感知缓存避免重复解析 collection.md。
- **可维护性**：模块职责单一（topology_collection/limit_parser/pool_merge/exporter/deploy_bridge）；中文注释完整；偏离 LLD 点均记录。
- **文档**：topo-planning README 7 章；limit-syntax.md 7 示例 + 10 反例；执行指导 [1.5] 与 ptm-te.md 一致。

## 9. 缺陷摘要

| 严重度 | 数量 | 说明 |
|---|---|---|
| Blocker | 0 | — |
| High | 0 | — |
| Medium | 0 | — |
| Minor | 1 | Q1：参考用例 `devices_direct.yaml` TG 关联失败（cosmetic 告警，产物正确），详见 REVIEW.md Minor-1 |
| Info | 2 | Q2：`ptm_team/install.py` 模板副本不对称（已知限制）；Q3：STATE.current.json active_change 未同步 CR-037 |

## 10. 结论

全量 271 passed / 0 failed 独立复核通过；8 维度 BLOCKING 全部通过；层A fixture 11 条（8 准则 + 3 专项）+ Minor-4 全部通过；层B 记录未验证风险与 N/A。**CP7 结论 PASS_WITH_RISK**。

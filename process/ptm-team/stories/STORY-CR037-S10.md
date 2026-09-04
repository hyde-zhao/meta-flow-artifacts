---
story_id: "CR037-S10"
story_slug: "test-fixture-two-layer"
cr_id: "CR-037"
title: "测试（层A fixture 造数必达 + 层B 目标项目真实数据可选）"
priority: "P0"
wave: 4
status: "ready-for-verification"
depends_on: ["CR037-S1", "CR037-S2", "CR037-S3", "CR037-S4", "CR037-S5", "CR037-S6", "CR037-S7", "CR037-S8", "CR037-S9"]
dependency_type: "precedence"
assignee: null
dev_context: "按 HLD §11 完成准则两层验证：层A（fixture 造数必达）——collection/limit/pool_merge/mapper/exporter/ledger 闭环单测 + 参考用例 dry-run；层B（目标项目真实数据可选，依赖 P-2）——真实池映射验证。测试造数隔离在 ptm-team tests/fixtures，不注入目标项目。专项单测：H4 合并语义（TE+copper 不可满足）、include-at-least-one 软约束（池仅 1 条 TE 链路）、L2 透传 link 不计入 LINK_COUNT_MISMATCH（S10-A）。层 B 第 10 条（--execute 真实执行）标注 N/A（依赖 DQ-037-04 独立授权，S10-B）。"
validation_context: "层A fixture 验证 8 条完成准则 + 3 项专项单测（H4 合并语义 / include-at-least-one 软约束 / L2 透传链路计数）+ 层B 目标项目真实数据验证（可选，P-2 未就绪时标注未验证风险，不阻断 CP7 fixture 结论）。层B 第 10 条 --execute 真实执行标注 N/A。"
acceptance_criteria:
  - "层A：12 拓扑 find_topology_file 命中率 100%（fixture 造数）"
  - "层A：limit 解析单测（设备约束 all 语义 + 3 链路模式 + 平台别名 13 系列 + 空 limit 默认）"
  - "层A：per-link TE 约束（fixture 池含 TE 口成功 / 不含 TE 口结构化失败 + --report-out）"
  - "层A：ledger 占用闭环单测（allocate 写回 / 二次映射避让 / release 释放）"
  - "层A：约束叠加优先级 + env-file load_env_file 校验 + LINK_COUNT_MISMATCH 错误路径"
  - "专项：H4 合并语义——TE 链路约束叠加 TE+copper 媒体类型不可满足 → 结构化失败（S10-A）"
  - "专项：include-at-least-one 软约束——池仅 1 条 TE 链路时至少命中 1 条即成功，不强制 all 覆盖（S10-A）"
  - "专项：L2 透传 link（SW 数据面透明）不计入 LINK_COUNT_MISMATCH，链路数一致性按逻辑 topo 有效链路计（S10-A）"
  - "层A：install dry-run（PTM_TE_SKILLS 7 + 模板副本不含造数 + copy_skill_tree 裁剪 tests/）；P-1 模板占位化断言用 case-sensitive 精确特征串（Minor-4）：config 模板与模板副本不含 A1600-HU/A1300-HU/A1500-HU/HG3250/hg3250/IXIA-C/Ngfw@123/10.113. 等特征串，仅含 ${PLACEHOLDER} 模式"
  - "层A：参考用例 IPv4策略路由（TOPO-01，fixture 池）映射 → env-file → dry-run 链路打通"
  - "层B（可选）：目标项目真实池映射验证，P-2 未就绪标注未验证风险；第 10 条 --execute 真实执行标注 N/A（DQ-037-04，S10-B）"
output_files:
  - "skills/topo-planning/tests/"
  - "tests/test_topo_plan_cr037.py"
file_ownership:
  primary:
    - "skills/topo-planning/tests/"
  shared: []
  merge_owner: null
  forbidden: []
feature_design_refs:
  - "F-CR037"
lld_policy:
  required_level: "technical-note"
  trigger_reasons:
    - "测试 Story，验证覆盖"
    - "两层验证（H5）"
  rationale: "按 HLD §11 完成准则（层A fixture 必达 + 层B 目标项目可选）验证"
dev_gate: "cp5-approved"
---

# STORY-CR037-S10：测试

## 目标

按 HLD §11 两层完成准则验证全部 Story 实现。

## 技术说明（technical-note）

### 设计依据

- HLD §11 完成准则（H5）两层验证：层 A（fixture 造数必达，8 条量化准则）+ 层 B（目标项目真实数据可选，依赖 P-2）；S10 是测试 Story，验证 S1-S9 全部实现 Story 的设计契约闭环。
- 评审核实：测试造数隔离在 ptm-team `tests/fixtures`，不注入目标项目（ADR-03 / HLD §8 隔离）；install 模板副本不含造数（AG-7）；`copy_skill_tree` 裁剪 `tests/`（S8-A）保证安装产物无造数。
- CP5 修订（S10-A/S10-B/S10-C）：补齐 H4 合并语义边界、include-at-least-one 软约束、L2 透传 link 计数三项专项单测；层 B 第 10 条 `--execute` 真实执行标注 N/A（依赖 DQ-037-04）；fixture 所有权去重（归 P-1）。
- CP5 第二轮复审（S10 Minor-4）：P-1 模板「不含具体型号」断言改为 **case-sensitive 精确特征串**清单（`A1600-HU`/`A1300-HU`/`A1500-HU`/`HG3250`/`hg3250`/`IXIA-C`/`Ngfw@123`/`10.113.` 逐字精确匹配），避免模糊匹配误触发（如小写 `hg3250` vs 注释残留）；断言对象限定 config 模板 + 模板副本，fixture 不纳入扫描（唯一允许造数位置）。

### 文件影响

| 文件 | 操作 | 说明 |
|------|------|------|
| `skills/topo-planning/tests/` | **新建** | S7/S9 单测（test_topo_plan_cli.py / test_deploy_bridge.py）+ 专项单测 |
| `tests/test_topo_plan_cr037.py` | **新建** | 层 A 全链路集成测试（collection/limit/mapper/exporter/ledger + 参考用例 dry-run） |
| `skills/topo-config/tests/fixtures/physical_pool.sample.yaml` | **只读消费（不写入）** | **所有权归 P-1**（S10-C），S10 只消费其造数（hg3250 + TE 端口） |
| `skills/topo-config/config/{physical_pool,physical_ledger}.yaml` | **只读消费** | P-1 模板（占位符），S10 校验模板副本一致性 + **模板占位化 case-sensitive 精确断言（Minor-4）** |

### 接口变化

- **不新增产品接口**：S10 只写测试，不改 S1-S9 产品接口。
- **验证入口**：复用 S1 `find_topology_file`、S2 `parse_limit`、S3 `merge_pool`、S4 `TopoMapper.match` / `PhysicalPool.release`、S5 `export_env_file` / `build_failure_report`、S6 `compute_ip_plan`、S7 CLI `topo-plan`、S8 install + 编排、S9 `DeployBridge` 作为被测接口。
- **测试桩契约**：case-execution `load_env_file` 作为 env-file 契约校验入口（S5 NFR1）；`install --agent ptm-te` 作为安装链路验证入口。

### 权限变化

- **不新增运行权限**：S10 全部为 fixture 层 dry-run / 单测，0 次真机写、不读写凭据。
- **层 B 第 10 条 N/A（S10-B）**：`--execute` 真实执行标注 N/A——依赖 DQ-037-04 独立 runtime_authorization（真机下发），不在 S10 默认执行范围；S10 只做 `DeployBridge.deploy(dry_run=True)`（0 次真机写）验证。
- **凭据约束**：测试 fixture 密码用占位符（test-placeholder），断言产物无 `password:`/`token:`/`api_key:`/`secret:`（ADR-02）。

### 层 A（fixture 造数，必达）

- **测试数据来源与所有权（S10-C）**：测试数据来自 `skills/topo-config/tests/fixtures/physical_pool.sample.yaml`（含 hg3250 假设设备 + TE 端口），**该文件为 P-1 Story 的 output_files / primary ownership，S10 只消费不重复声明**（已从本 Story output_files 移除）。造数隔离于目标项目：S8 install 的 `copy_skill_tree` 裁剪 `tests/`，模板副本源 `skills/topo-config/config/` 为占位模板，双保险保证造数不注入目标项目。
- 覆盖（8 条完成准则）：collection 12 拓扑命中、limit 解析（all 语义/3 模式/13 系列/空 limit）、per-link TE 约束、ledger 闭环（allocate/避让/release）、约束叠加、env-file 契约、install dry-run、参考用例链路。

### 层 A 专项单测（S10-A，CP5 修订新增）

| 专项单测 | 场景 | 预期 |
|---------|------|------|
| `test_h4_te_copper_unsatisfiable` | H4 合并语义：用户链路约束 TE + collection link 自带约束叠加后要求 `speed_class=TE, media_type=copper`（TE 仅 fiber，合并不可满足） | 映射判 failed，结构化报告含 `link_constraint_unsatisfied`，不兜底（验证 H4 叠加公式边界） |
| `test_include_at_least_one_soft_constraint` | include-at-least-one 软约束：池仅 1 条 TE 链路，limit 要求「TE\|10GE 至少 1 条」 | 映射成功（至少命中 1 条 TE 链路即满足，不强制 all 覆盖）；断言映射链路含该 TE 链路 |
| `test_l2_transparent_link_not_counted` | L2 透传 link（SW 数据面透明，exporter 自动透传跳过）不进入 env-file links / 不计入 LINK_COUNT_MISMATCH | 链路数一致性按逻辑 topo **有效链路**（L2 透传除外）计，不误报 LINK_COUNT_MISMATCH |

### 模板占位化断言（Minor-4，case-sensitive 精确特征串）

S10 对 P-1 模板「不含具体型号」的断言统一使用 **case-sensitive 精确特征串**（避免模糊匹配误触发，如 `hg3250` 小写 vs 注释残留的 `HG3250`；也避免「前缀包含」误判）。

**断言对象**（仅扫描模板路径，不扫描 fixture）：
- `skills/topo-config/config/physical_pool.yaml`（P-1 模板）
- `skills/topo-config/config/physical_ledger.yaml`（P-1 台账模板）
- install 生成的模板副本 `topology/physical_pool.yaml` + `topology/physical_ledger.yaml`（源 = 上述模板，S8）

**断言方式**：
1. **占位符模式**：模板正文仅含 `${PLACEHOLDER}` 模式（`re.search(r'\$\{[A-Z_]+\}')` 之外的裸值视为占位化失败）。
2. **具体型号特征串黑名单（case-sensitive 精确匹配，非子串模糊）**：

| 特征串 | 说明 |
|--------|------|
| `A1600-HU` | 具体安装型号（原真实数据 device_type） |
| `A1300-HU` | 造数型号（仅允许存在于 tests/fixtures，模板正文禁止） |
| `A1500-HU` | 歧义型号（S2 消歧，模板正文禁止） |
| `HG3250` | 硬件平台等价类具体值（模板只允许 `${HARDWARE_PLATFORM}`） |
| `hg3250` | 造数设备前缀（仅 fixture，模板正文禁止） |
| `IXIA-C` | TG 具体型号（模板正文禁止，只允许 `${DEVICE_TYPE}`） |
| `Ngfw@123` | 真实密码（ADR-02 凭据禁止，模板正文禁止） |

3. **真实 IP 特征**：`10.113.` 前缀（真实管理地址）在模板正文中不得出现（模板只允许 `${MGMT_HOST}`）。

**断言语义**：上述特征串在断言对象文件中**逐字精确匹配**（`if token in text` 区分大小写）；命中任意一条 → 断言失败，提示 P-1 模板化未完成或 S8 模板副本源错误。fixture（`tests/fixtures/physical_pool.sample.yaml`）是**唯一**允许含造数型号的位置，不纳入本断言扫描路径。

**与 Minor-3 的闭环**：P-1 修订保证模板正文/模板副本不含上述特征串（示例块统一标注「示例/占位，勿直接使用」）；S10 用上述精确清单在层 A 校验（install dry-run + 模板副本内容扫描），防止模板漂移或造数泄漏。

### 层 B（目标项目真实数据，可选）

- 依赖 P-2（用户填写目标项目 `topology/physical_pool.yaml`）。未就绪时标注未验证风险，不阻断 CP7 fixture 结论。
- **第 10 条 N/A（S10-B）**：HLD §11 层 B 第 10 条「真实 dry-run/`--execute` 执行验证」中 `--execute` 真实执行标注 **N/A**——依赖 DQ-037-04 独立 runtime_authorization（真机下发），不在 S10 默认执行范围；S10 只做 fixture 层的 dry-run 链路验证（`DeployBridge.deploy(dry_run=True)` 0 次真机写）。`--execute` 真机验证留待用户单独授权后（follow-up），S10 不默认触发。

### 异常与回退

- 层 A 某项完成准则失败 → 对应 Story 标记 CP7 NEEDS_REWORK / NEEDS_DESIGN_CLARIFICATION 路由（按验证结论路由）。
- 层 B 未就绪（P-2 缺失）→ 记录「未验证风险」到 VERIFICATION-REPORT，不阻断 CP7 fixture 结论。
- fixture 数据与模板不一致 → 校验失败，提示 P-1 模板化或 S8 安装链路问题。
- 模板/模板副本含具体型号特征串（Minor-4 case-sensitive 精确断言命中任一黑名单项）→ 断言失败，提示 P-1 模板化未完成（模板正文仅允许 `${PLACEHOLDER}` 模式，fixture 是唯一允许造数位置）。

## 依赖

CR037-S1~S9 全部实现 Story。

## 设计证据

本卡片内嵌 technical-note（S10 无独立 LLD 文件）。

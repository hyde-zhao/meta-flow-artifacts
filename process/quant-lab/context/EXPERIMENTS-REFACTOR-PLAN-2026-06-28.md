# experiments/ 目录重构与并行因子栈消除 — 执行计划

- 日期：2026-06-28
- 分支：`precheck/ql-rd-000-redesign-baseline`（worktree clean，已验证）
- 状态：**Phase 0/1/2 待执行批准；Phase 3 决策已收敛（§7 1B+2A），前置 BLOCKING 满足后执行；Phase 4 独立 CR**
- 评审历史：v1（评审基线，已废弃）→ 评审 6 条 + 自检 5 条 → v2（仍含 v1 冲突内容，已废弃）→ 评审 6 条 + 自检 → **本版（单一可执行版，v1/v2 正文已清除）**

> 本文档为唯一可执行版本。历史评审结论折叠在文末 §9（仅审计用，不含执行指令）。

---

## 0. 执行前置与 CR 流程（新会话首读）

> 本仓库是 **Meta Flow 项目**（见 `CLAUDE.md`）。本重构为结构性变更，**走正式 CR 流程**（用户 2026-06-28 确认），不走 fast-lane。新会话须先完成 CR 登记，再按 Phase 执行。

### 0.1 环境前提（已验证）
- Python：`requires-python = ">=3.11,<3.13"`（`pyproject.toml:5`）；统一用 `uv run --python 3.11`
- 依赖：`uv.lock` 存在；首次执行前跑 `uv sync` 确保环境就绪
- 测试命令：`uv run --python 3.11 pytest -q`（186 测试文件，扁平在 `tests/`，`conftest.py` 仅插 sys.path）
- 分支：当前 `precheck/ql-rd-000-redesign-baseline`，worktree clean

### 0.2 包结构事实（已验证，勿重复探索）
- `experiments/` **无 `__init__.py`**（namespace package）——靠 sys.path 解析，移走文件后所有 `experiments.X` import 须改路径
- `engine/__init__.py` 存在但**不注册模块、无副作用导入**——移入 `engine/research_reporting.py` / `engine/production_current_truth_rerun.py` **无需改 `__init__.py`**，导入靠 sys.path
- `scripts/` 与 `scripts/data_lake/` **无 `__init__.py`**（namespace package）

### 0.3 CR 登记（执行第一步）
1. **编号核实**：`CR-INDEX.yaml` 当前最大为 CR-139（已存在），本 CR 应为 **CR-140**（执行时再核实 `ls process/changes/ | grep -oE 'CR-[0-9]+' | sort -t- -k2 -n | tail`）
2. 创建 CR 文件 `process/changes/CR-140-EXPERIMENTS-REFACTOR-2026-06-28.md`，照 `CR-138-*.md` frontmatter 格式（参考 `process/changes/CR-138-RUNNER-QMT-GATEWAY-OPERATIONAL-USE-CASE-BASELINE-2026-06-24.md`），必填字段：`cr_id/cr_type/cr_kind/lifecycle_status/readiness_status/gate_status/gate_profile/impact_level/approval_result/created_at/created_by/source`；`cr_kind=requirement-change`，`gate_profile=standard`，`impact_level=medium`（结构重构非高风险），`source=user-request`
3. CR 范围：Phase 0-3 + Phase 5（Phase 4 明确列为后续独立 CR 候选，在 CR 文件 `follow_up` 标注）
4. 更新 `process/changes/CR-INDEX.yaml` 与 `CR-INDEX.json`（追加 CR-140 行）
5. 更新 `process/STATE.md`：**先读实际结构**（`active_change`/`cr_tracking` 字段名以 STATE.md 现有 YAML 为准，勿假设字段名），登记本 CR 为 active
6. CP2（需求/范围基线）引用本文档作为范围基线；CP6/CP7/CP8 终验跑全量 pytest + 本文档 §4 验证清单
7. **不**跳过 CP 门禁；若用户后续授权 fast-lane 再另行处理

### 0.4 git 操作约定
- 用 `git mv` 移文件（保留历史）
- 每 Phase 一个 commit，commit message 前缀 `refactor(experiments): Phase N - <摘要>`
- 不 push 除非用户要求；分支 `precheck/ql-rd-000-redesign-baseline` 上分次提交便于回退
- **回退判定**：任一 Phase 后全量 pytest 出现"基线外新增失败"且 2 次尝试未修复 → `git revert` 该 Phase commit，回到上一绿态，记录阻塞

### 0.5 Phase 0 基线方法（操作性）
1. 跑 `uv run --python 3.11 pytest -q --tb=no -p no:cacheprovider 2>&1 | tee /tmp/baseline-pytest.txt`
2. 解析 pass/fail 计数与失败测试名集合，存 `/tmp/baseline-failures.txt`（每行一个失败节点 id）
3. 后续每 Phase 后跑同命令，`diff` 失败集合：**只允许失败集合缩小或不变**；出现新失败 = 阻断
4. 若基线本身有失败（redesign 中途可能），在 CR 文件记录基线失败清单作为"已知非本次引入"

### 0.6 parents[N] 算术（已验证，勿重算）
- `scripts/legacy/cr/cr012_...` 在 depth-3 → `REPO_ROOT = parents[3]`（当前 `parents[1]` 错误，须修）
- 迁移后 `scripts/data_lake/run_data_lake_readiness_audit.py` 在 depth-2 → `PROJECT_ROOT = parents[2]`（当前在 experiments/ depth-1 用 `parents[1]`）
- 无仓库级 repo-root helper（`engine/research_paths.py` 无 root 函数），维持 `parents[N]`

### 0.7 Phase 3 旧输出冻结（golden fixture 前置）
turnover 实验无直接测试，"旧实现输出"须在 Phase 3 **切换前**冻结：
1. 在 Phase 3 起始，用合成 lake fixture 跑**当前（未改）**实验，捕获 factor_matrix / valid_mask_count / rebalance_dates / single_sort / double_sort / spread / NW 输出，存 `tests/fixtures/turnover_golden/`（含 quantile 边界用例：低样本/全常数/重复值）
2. 切 engine + 4 adapter 后，新实现跑同 fixture，逐项对比 §3 golden fixture 表
3. 旧输出冻结前不得改实验代码

### 0.8 测试命名与新增清单
- 新增（命名 = 模块路径 + 语义，脱离 CR/实验编号）：
  - `tests/test_turnover_factor_integration.py`（Phase 3，含 golden fixture 对比 + quantile 边界用例）
- 现有 `test_cr0NN_*`/`test_experiment_NN_*` **仅改导入路径，不改名**（避免 churn）
- Phase 1 各迁移：若现有 CR 测试已覆盖契约（如 `test_cr008/010/013/014` 覆盖 reporting 边界），**不**新增，仅改导入；无覆盖才补 `tests/test_<module>_contract.py`

### 0.9 不变量与合规红线（执行中不得违反）
- 旧报告 baseline guard **永久保留**（`run_experiment_17_21_factor_suite.py:441`，`test_cr011:113` 断言）——Phase 5 不删
- `engine/research_reporting.py` **不可**用 `engine/reporting.py` 路径（自毁）
- DEV-LOG 历史条目**不改就地路径**（历史真相，§3 1.2）
- Phase 3 不扩张 engine 契约（1A 伪造入参 / 2C 加参数均禁止，归后续 CR）
- 切 engine statistics 不得改变实验语义（4 adapter 必须全部到位）

---

## 1. 背景与目标

## 1. 背景与目标

`experiments/` 目录存在四类问题：非实验文件混入、helper 复制粘贴、命名混乱、与 `engine/` 因子栈并行重复。

**本轮目标**：
1. 三个非实验文件归位（P0）
2. 收敛重复 helper + 建实验编号索引（P1）
3. turnover 实验因子评估侧切 engine（P2a，条件批准）
4. Phase 4（exp15/16/17_21 切 engine）**拆独立 CR，本轮不做**
5. 旧报告 baseline guard **永久保留，不删**

engine 缺失组件的 GAP（ML/walk-forward/风险叠加/可交易性/集成回测）留后续 CR。

---

## 2. 前置事实（已验证，执行时以这些为准）

| 事实 | 证据 | 影响 |
|------|------|------|
| worktree clean | `git status --short` 空 | 我此前 S3"根目录 cr012 已删"主张**错误**，cr012 按 active 处理 |
| cr012 仍 active | `scripts/data_lake/repair_market_data.py:13` `from scripts.legacy.cr.cr012_limited_window_lake_repair import *` | 迁 audit 文件**必须**同步修 cr012 的 import + REPO_ROOT，stable wrapper 须继续可运行 |
| cr012 导入面更大 | `cr012:28-40` 导入 `AuditConfig` + 7 个 `DATASET_*` 常量 + 4 个私有 helper（`_coverage_context`/`_pairs_from_frame`/`_probe_all`/`_read_catalog_entries`） | 迁移须跟进全部 12 个符号，非仅 5 个 |
| cr012 REPO_ROOT 错误 | `cr012:24` `REPO_ROOT=parents[1]`=scripts/legacy（非仓库根） | 独立运行时 import 已脆弱，迁移时一并修为 `parents[3]` |
| 无仓库级 repo-root helper | `engine/research_paths.py` 无 root 函数；各脚本各自 `parents[N]` | Finding 5 建议"repo-root helper"无处复用，维持 `parents[N]` |
| guardrail 脚本缺失 | `scripts/check_delivery_guardrails.py` 不存在；`README.md:1029` 标注"流程债" | 验证项删除该步，改用 `check_script_entrypoints.py` + 静态 import scan + 全量 pytest |
| reporting.py 引用 9 处（4 测试 import + 2 实验 import + 3 扫描目标 string） | `rg "experiments\.reporting\|experiments/reporting\.py"` 命中 9 处：`test_cr013:8`、`test_cr008:24`、`test_cr014:13`、`test_cr010_experiments_realism:9`（import）+ `run_experiment_14:19`、`run_experiment_15:32`（import）+ `test_cr010_consumer_boundary:12`、`test_cr008:31`、`test_cr014:293`（扫描目标 string） | Phase 1.1 **rg 驱动更新全部 9 处**，不写固定数字 |
| turnover 实验无直接测试 + 强依赖真实 lake | `run_experiment_turnover_factor` 在 `tests/` 无直接 import（但 `rg turnover tests/` 命中 26 文件，abnormal_turnover calculator 等有覆盖）；`turnover_factor.py:56-59` 读 `LAKE_CANONICAL` parquet | Phase 3 golden fixture **前置 BLOCKING**：须先解决离线可跑（合成 lake fixture 或注入），否则无法对齐。注意 engine factor calculator 已有测试覆盖，勿误判为零 |
| 实验与 engine 分组边界行为不同 | 实验 `assign_quantile_groups`（`turnover_factor.py:475`）在 `len<group_count`/`nunique<2`/`qcut ValueError` 时返回全 NA；engine `factor_research_matrices.quantile_groups`（`:72`）默认 `require_full_count=False`、`qcut(duplicates="drop")` + `min(quantiles,len(valid))`，常数/低样本不返回全 NA | Phase 3 替换须写 adapter 保实验语义，或 golden fixture 覆盖低样本/全常数/重复值 |
| engine 有两套分组实现 | `factor_statistics._quantile_groups`（`:286`，被 `single_sort_returns`/`independent_double_sort_returns`/`conditional_double_sort_returns` 内部用）与 `factor_research_matrices.quantile_groups`（`:72`）是两套不同实现 | Phase 3 排序切 statistics 后，`assign_quantile_groups` 不单独声明切另一实现，避免同实验依赖两套 |
| 旧报告 guard 是永久不变量 | `README.md:124/700/710` + `test_cr011_factor_panel_robust_validation.py:113` `pytest.raises(match="禁止覆盖旧实验 17-21 baseline 报告")` | Phase 5 **不删** guard |
| abnormal_turnover 已在 library+calculator | `factor_library.py:25/186` + `factor_calculators.py:180`；未在 `factor_registry.py` catalog | Phase 3 catalog 注册仅按需 |
| engine 扩展点不完整 | `factor_calculators.py:113` direction 硬编码查 core；`factor_library.py:229` 只查 core map | Phase 4 拆 CR 的技术根因 |
| 无循环 import 风险 | `engine/research_dataset.py` 不 import `engine.reporting` | `engine/research_reporting.py` 落 engine 安全 |

---

## 3. 执行 Phase

### Phase 0 — 记录测试基线
- 跑 `uv run --python 3.11 pytest -q`，记录 pass/fail 集合（当前分支 redesign 中途，可能有既有失败）
- 后续每 Phase 只对比"基线外新增失败"，避免基线失败掩盖/误归咎

### Phase 1 — P0：非实验文件归位

**1.1 `experiments/reporting.py` → `engine/research_reporting.py`**
- 不可用 `engine/reporting.py`（通用 tabular-safety 模块，被本文件 import，会自毁）
- **rg 驱动更新全部 9 处引用**（见 §2 事实表，不写固定数字）：
  - 4 测试 import：`tests/test_cr013_unsupported_register_claim_boundary.py:8`、`tests/test_cr008_research_input_metadata.py:24`、`tests/test_cr014_research_consumer_boundary.py:13`、`tests/test_cr010_experiments_realism_metadata.py:9`
  - 2 实验 import：`experiments/run_experiment_14.py:19`、`experiments/run_experiment_15_factor_framework.py:32`
  - 3 扫描目标 string：`tests/test_cr010_consumer_boundary.py:12`（CONSUMER_FILES）、`tests/test_cr008_research_input_metadata.py:31`（TARGET_FILES）、`tests/test_cr014_research_consumer_boundary.py:293`（touched_files）→ 改为 `engine/research_reporting.py`
- `engine/research_reporting.py` 替换 `CONSUMER_FILES` 中原 `experiments/reporting.py` 条目，确保迁入后仍受 forbidden-imports 扫描覆盖（engine 文件本在扫描范围）
- 验收：`rg -n "experiments\.reporting|experiments/reporting\.py"` 旧路径仅允许出现在历史说明/legacy 证据中

**1.2 `experiments/production_current_truth_rerun.py` → `engine/production_current_truth_rerun.py`**
- 改 1 导入方：`tests/test_cr018_production_current_truth_rerun.py`
- `DEV-LOG.md:482/488` 是 CR018-S08 已完成事件的实现文件清单（历史真相记录），**不改就地路径**——改写会让历史记录失真（同 Finding 1 guard 不变量原则）；仅在新日期条目里追加迁移说明
- 验收：`rg -n "experiments\.production_current_truth_rerun|experiments/production_current_truth_rerun\.py"` 仅历史说明

**1.3 `experiments/run_data_lake_readiness_audit.py` → `scripts/data_lake/run_data_lake_readiness_audit.py`**
- 改 `parents[1]` → `parents[2]`（`run_data_lake_readiness_audit.py:21`）；prog 文案 `:1790` 更新为 `scripts/data_lake/run_data_lake_readiness_audit.py`
- 改 2 导入方，跟进**全部 12 个符号**：
  - `scripts/legacy/cr/cr012_limited_window_lake_repair.py:28`（`AuditConfig` + 7 `DATASET_*` + 4 私有 helper）；**同时修 cr012:24 `REPO_ROOT` 为 `parents[3]`**，保证 `scripts/data_lake/repair_market_data.py` stable wrapper 继续可运行
  - `tests/test_data_lake_readiness_audit.py:8`（`AuditConfig` + 3 状态常量 + `INDEX_MEMBERS_AUDIT_MODE_*` + `_is_legacy_data_path` + `run_audit`）
- 登记 `scripts/quality/check_script_entrypoints.py` REQUIRED_STABLE_ENTRYPOINTS（set 字面量，`:24`，追加 `"scripts/data_lake/run_data_lake_readiness_audit.py"`）
- 在 `docs/components/SCRIPT-ENTRYPOINTS.md` §4 表新增一行，能力名 **`production_strict readiness audit (2020-2024, CR012 helper)`**，legacy 来源标注 `experiments/run_data_lake_readiness_audit.py`；**明确说明**：与现有"数据 readiness"（`check_market_data_readiness.py`，CR-034 chapter-3 范围）policy/scope/默认窗口/输出目录不同，**不替代**现有 readiness CLI
- 验收：`rg -n "experiments\.run_data_lake_readiness_audit|experiments/run_data_lake_readiness_audit\.py"` 仅历史说明；`repair_market_data.py` 经 stable wrapper 可运行

### Phase 2 — P1：helper 收敛 + 实验索引表

**2.1 helper 收敛**
- 家族 A（percent-aware，06_07 系）：删 `run_experiment_08.py` 本地全套副本改 import 06_07；删 08/09/10/12/13 死代码 `_resolve_date_range`；09/10/12 的 `_markdown_table`/`_format_value` 改 import 06_07；13 的 dimension 版本保留
- 家族 B（simple-float，15/14 系）：维持 15 为基座，`run_experiment_14.py` 改 import 15 的 `markdown_table`/`format_value`
- 不引入 `engine/research_cli.py` 新 helper

**2.2 实验索引表**
- `docs/components/EXPERIMENTS.md` 新增 §4「实验编号索引」表（实验编号 | 主题 | 文件 | 关联 chapter/CR | 输出目录 | 测试 | 状态），录入全部实验，标注缺失编号去向与 GAP 项为「后续 CR 候选」
- `SCRIPT-ENTRYPOINTS.md` §4 只新增 §1.3 的 readiness audit 入口（按 Finding 6 能力名 + 差异说明），**不**把 `experiments/run_experiment_*` 提升为稳定入口

### Phase 3 — P2a：turnover 因子评估侧切 engine（条件批准）

**前置 BLOCKING（须先解决再执行切换）**：

(a) **离线可跑**：实验读真实 `LAKE_CANONICAL` parquet（`turnover_factor.py:56-59`），CI 无法跑全流程。先建合成 lake fixture 或数据注入层，使实验可离线运行，否则 golden fixture 无从对齐。

(b) **golden fixture 验收定义**（逐项 exact / tolerance / accepted-diff）：

| 比较对象 | 字段 | 容差 | 可接受差异 | 阻断条件 |
|----------|------|------|-----------|----------|
| factor_matrix | abnormal_turnover 21/252 比值矩阵 | exact（须薄 wrap 对齐 min_periods=15/60 + clip[0.01,10]） | 无（二选一：薄 wrap 完全对齐→exact；若选接受差异则禁止用 exact，须把差异字段/方向/影响写进本表） | 任一 cell 不等 |
| valid_mask_count | 有效样本数 | exact | 无 | 不等 |
| rebalance_dates | 20 日调仓日期集合 | exact | 无 | 不等 |
| quantile_edge_cases | 低样本（len<5）/全常数（nunique<2）/重复值横截面 分组结果 | exact | 无（实验返回全 NA；engine 须经 adapter 对齐，不得退化分组） | 边界行为不等 |
| single_sort_summary | G1-G5 单排序收益 | tolerance 1e-9 | 无 | spread 方向翻转 |
| double_sort_cells | 5×5 市值×换手 双排序 cell | tolerance 1e-9 | 无 | long-short 方向翻转 |
| spread_direction | low-minus-high 符号（实验 G1−G5） | exact | 无（engine 须 `high_minus_low=False`） | 符号相反 |
| spread_nw | long-short NW t-stat | — | **显式接受差异**（实验 lag 规则 vs engine lag 规则不同；NW 保留实验实现，见执行切换） | 不阻断，记录为已知差异 |

> **factor_matrix 一致性约束（已锁定决策 2A）**：薄 wrap 完全对齐（short_min_periods=15/long_min_periods=60/clip[0.01,10]）→表中 exact 成立。

(c) **决策已收敛**（见 §7）：admission=1B、min_periods/clip=2A、cr012=active，三项均已锁定，Phase 3 可执行。

**执行切换（前置满足后）**：
- 因子评估侧切 engine：`FactorSpec`+`validate_factor_spec`、`FactorRunSpec`+`compute_config_hash`+`validate_factor_run_spec`、`build_factor_evaluation_report`、双排序 → `engine.factor_statistics`（`single_sort_returns`/`independent_double_sort_returns`/`conditional_double_sort_returns`）
- `calculate_abnormal_turnover` → `engine.factor_calculators._calculate_abnormal_turnover_21_252` + **薄 wrap adapter**（决策 2A）：wrap 只负责参数/clip 适配（固定 `short_min_periods=15`/`long_min_periods=60`/`clip[0.01,10]`），不复制整套因子栈；golden fixture 证明 adapter 输出与旧实验 exact 一致；后续如要把 min_periods/clip 参数化进 engine，另起 CR
- `build_forward_return_20d` → `engine.factor_statistics.build_forward_return_matrix(horizon=20)`
- **分组实现统一**：排序切 `engine.factor_statistics` 后，`single_sort_returns`/`independent_double_sort_returns`/`conditional_double_sort_returns` 内部用 `factor_statistics._quantile_groups`（`:286`）；实验 `assign_quantile_groups` 若仍被公开面使用，写 adapter 对齐实验语义（低样本/全常数/重复值返回全 NA），**不**单独声明切 `factor_research_matrices.quantile_groups`（避免同实验依赖两套分组实现，见 §2 事实）
- **valid_mask + rebalance_dates adapter**（engine `single_sort_returns`/`independent_double_sort_returns`/`conditional_double_sort_returns` 无此参数，默认遍历全部 common dates；实验先按 `valid_mask` 过滤再只跑 `rebalance_dates`，见 `turnover_factor.py:409/584` vs `factor_statistics.py:26/60`）：调用 engine 前先用 `valid_mask` 将 factor/forward/size 矩阵非有效样本置 NaN（`matrix.where(mask)`），再 `.loc[rebalance_dates]` 截取调仓日子集，再传入 engine statistics——保持"仅调仓日 + mask 过滤"语义
- **spread 方向适配**（实验 spread = G1−G5 = 低异常换手率−高 = low-minus-high = 看多方向，`turnover_factor.py:630/716`；engine `long_short_summary`/`long_short_summary_from_double_sort` 默认 `high_minus_low=True` = high-minus-low，`factor_statistics.py:152/177`）：所有 turnover spread 调用 engine summary 时显式传 `high_minus_low=False`；golden fixture 验证 spread 符号与旧实现一致
- **admission 打包 = 决策 1B**：保留实验轻量 research_only 摘要，**不**调 `build_strategy_admission_package`；改 schema 加显式标注字段（`artifact_type: research_only_admission_summary` / `not_engine_strategy_admission_package: true` / `not_simulation_authorization: true` / `not_qmt_authorization: true`）
- **NW t-stat 保留实验自有实现**（复刻保真度，不切 engine；golden fixture 记录为已知差异）
- **admission 打包按 §7 决策项 1**：不默认切 engine（实验手搓的是"research_only 研究参考摘要"，非 engine `StrategyAdmissionPackage` 契约形态）；若决策为不升格，保留实验轻量摘要或删除产物
- catalog 注册仅按需（`abnormal_turnover_21_252` 已在 library+calculator，未在 catalog）
- 新增 `tests/test_turnover_factor_integration.py`（含 golden fixture 对比 + quantile 边界用例）

### Phase 4 — 拆独立 CR（本轮不做）
exp15/16/17_21 切 engine 涉及注册 10 实验因子 + 新增 RSI/MACD/MA-gap/volume-change/max-drawdown calculator，且 engine 扩展点不完整（direction/definition 硬编码查 core）。拆独立 CR：先设计 `factor_ids→definitions→calculators→direction` 统一扩展合同 + feature design + 回归矩阵。

### Phase 5 — 清理残留 + 终验
- **guard 永久保留，删除动作取消**（旧 baseline 防护不变量）
- 清 Phase 2/3 产生的死代码
- 全量 `uv run --python 3.11 pytest -q` 通过（对比 Phase 0 基线，无新增失败）
- 同步 `EXPERIMENTS.md` 索引状态列与 `SCRIPT-ENTRYPOINTS.md`

---

## 4. 验证

每个 Phase 结束：
1. `uv run --python 3.11 pytest -q`（对比 Phase 0 基线，无新增失败才进下一 Phase）
2. Phase 1 各子项：`rg -n "experiments[./](reporting|production_current_truth_rerun|run_data_lake_readiness_audit)"` 旧路径（同时覆盖 dotted import 与 slash 字符串）仅历史说明
3. Phase 1.3：`scripts/data_lake/repair_market_data.py` 经 stable wrapper 可运行（`uv run --python 3.11 python scripts/data_lake/repair_market_data.py --help`）
4. Phase 3：`uv run --python 3.11 pytest -q tests/test_turnover_factor_integration.py` golden fixture 全过
5. 提交前运行 `uv run --python 3.11 python scripts/quality/check_script_entrypoints.py`（替代缺失的 guardrail 脚本，见 §2 事实）
6. 最终 `git status` 确认 `experiments/` 只剩 `run_experiment_*` 实验脚本

> `scripts/check_delivery_guardrails.py` 不存在（README:1029 标注流程债），**不**作为验证步骤。替代：`check_script_entrypoints.py` + 静态 import scan（`rg`）+ 全量 pytest。

## 5. 测试命名方案

新建测试按**重构后模块路径 + 契约/集成语义**命名，脱离 CR/实验编号：
- `tests/test_turnover_factor_integration.py`（Phase 3，含 golden fixture）
- 现有 `test_cr0NN_*`/`test_experiment_NN_*` 仅改导入路径，不改名

## 6. 风险与回退

- cr012 stable wrapper 断裂 → 迁移后立即验证 `repair_market_data.py --help`；失败则回退 1.3
- Phase 3 离线可跑无法解决 → 回退 Phase 3 至仅 golden fixture 设计，不动实现
- engine 新注册触发 `factor_registry.validate_factor_catalog` 失败 → 先跑该校验
- 按 Phase 分次提交，便于回退

## 7. 决策（已锁定，2026-06-28 用户确认）

1. **admission 打包 = 决策 1B**：保留实验轻量 research_only 摘要，**不**升格为 engine `StrategyAdmissionPackage` 契约。实验产物是声明"因子研究参考，不可直接用于 simulation/QMT/live"，非提交 Stage6 准入包；强行调 `build_strategy_admission_package` 须伪造 `portfolio_plan`/`stage6_gate`/`order_intent_draft_ref`，成伪契约。落地：改 schema/字段名更清晰，加显式标注字段：
   ```json
   {
     "artifact_type": "research_only_admission_summary",
     "not_engine_strategy_admission_package": true,
     "not_simulation_authorization": true,
     "not_qmt_authorization": true
   }
   ```
2. **min_periods/clip = 决策 2A**：实验层薄 wrap 对齐原实验规格（`short_min_periods=15`/`long_min_periods=60`/`clip[0.01,10]`），factor_matrix golden fixture 继续 **exact**。不选 2B（改 engine 默认 min_period_ratio + 无 clip 会改变因子值 → 分组/spread/t 值变化，属研究结论层面变化，不适合目录整改顺手做）；不选 2C（扩张 engine calculator 契约，应独立 CR）。Phase 3 目标：engine calculator 提供通用 abnormal turnover 核心能力；turnover 实验保留复刻规格 adapter；golden fixture 证明 adapter 输出与旧实验完全一致；后续如要把 min_periods/clip 参数化进 engine，另起 CR。wrap 只负责参数/clip 适配，**不复制整套因子栈**。
3. cr012 状态已确认为 active（§2 事实），按 active 处理。

> Phase 3 另含两个已纳入的约束（见 §3 执行切换）：valid_mask + rebalance_dates adapter；spread 方向固定 G1−G5 / low-minus-high（engine 调用传 `high_minus_low=False`）。切 engine statistics 不得改变实验语义。

---

## 8. 评审结论映射（当前批准范围）

| Phase | 评审结论 | 本版处理 |
|-------|----------|----------|
| Phase 0/1/2 | 可批准 | 待用户最终批准执行 |
| Phase 3 | 决策已收敛（§7 1B+2A） | 前置 BLOCKING（离线可跑）+ golden fixture 表（含 quantile_edge_cases/spread_direction）+ 4 adapter（薄 wrap / mask+rebalance / high_minus_low=False / 分组统一）满足后执行 |
| Phase 4 | 独立 CR | 本轮不做 |
| Phase 5 guard | 拒绝删除 | 永久保留 |

## 9. 历史评审结论（仅审计，不含执行指令）

- **v1 评审 6 条**（全部采纳）：旧报告 guard 永久保留 / Phase 4 拆 CR / engine 扩展点不完整 / Phase 3 golden fixture 前置 / data_lake 路径修正 / experiments 不入稳定入口
- **自检 5 条**（S5 撤回，净剩 5）：契约扫描目标清单 / admission 打包非同类对象 / cr012 路径机制 / NW 保留实验实现 / 测试基线前置
- **v2 评审 6 条**（全部采纳）：v1 冲突内容清除（本版重写）/ golden fixture 验收定义（§3 表）/ cr012 active 处理（§2 事实）/ rg 验收（§4）/ guardrail 脚本 N/A（§4）/ readiness 入口能力名+差异说明（§3 1.3）
- **本轮自检**：cr012 导入面 12 符号 / 无 repo-root helper / turnover 离线可跑 BLOCKING
- **v3 评审 6 条**（全部采纳）：①reporting 引用 9 处非 3 处，rg 驱动（§2/§3 1.1）②golden fixture 表与 §7 min_periods/clip 矛盾，改二选一锁定（§3 表 + §7）③quantile 边界行为不同，加 edge_cases 行 + adapter（§3 表 + 执行切换）④engine 两套分组实现，统一走 statistics（§2/§3 执行切换）⑤turnover 测试措辞改"无直接测试"（§2 事实）⑥验证命令补 `--python 3.11`（§4）
- **v4 评审 3 条**（全部采纳）：①Phase 3 缺 valid_mask/rebalance_dates adapter，补"matrix.where(mask).loc[rebalance_dates] 再调 engine"（§3 执行切换）②spread 方向 G1−G5=low-minus-high，engine 默认 high_minus_low=True，补 `high_minus_low=False` + golden fixture `spread_direction` 行（§3 表 + 执行切换）③§4 统一 regex 只查 dotted 漏 slash，改 `experiments[./](...)`（§4）

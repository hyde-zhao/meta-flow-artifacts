---
story_id: "CR139-S03"
title: "C2a 重复画像 — LLD"
change_id: "CR-139"
owner_feature: "FEAT-02 读侧"
lld_policy:
  required_level: "full-lld"
wave: "CR139-W1-BASELINE-FREEZE"
status: "lld-ready-for-review"
req_coverage: ["REQ-205(C2a)"]
depends_on: ["CR139-S01", "CR139-S02"]
dependency_type: ["runtime"]
source_hld: "process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md"
source_adr: "process/docs/design/ARCHITECTURE-DECISION-STRATEGY-DATA-FOUNDATION.md"
feature_design_matrix: "docs/design/FEATURE-DESIGN-MATRIX.md v1.13"
story_card: "process/stories/CR139-S03-duplicate-fingerprint-profiling.md"
created_at: "2026-06-28T18:30:00+08:00"
delegated_by: "host-orchestrator"
delegated_to: "meta-dev"
not_authorized: "实现/runtime/真实 lake 写入/物理分区迁移/Git remote write（CP5 通过后才实现）"
---

# CR139-S03 C2a 重复画像 — 详细设计（LLD）

> 本文件是 CR-139 Wave1 基线门第 3 步 C2a 重复画像的 full-lld 设计证据。消费 companion HLD v0.2（confirmed-cp3）、ADR v0.2、MATRIX v1.13、REQUIREMENTS v1.17 REQ-205(C2a)/REQ-247/REQ-248/REQ-249。本 LLD 是设计证据，不含实现代码；CP5-W1 全量人工确认通过且 Story 进入 dev-ready 后才允许实现。

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 0.1 | 2026-06-28 | meta-dev | CR-139 CP5-W1 首版 full-lld：14 章齐全；独立只读分析函数（LCQ-01 resolved）；readers.py 切片边界（与 S07 C2b 分离）；Wave1 基线门第 3 步时序；量化验收 + 实质性 Gotchas。 |

---

## 1. 问题定义与目标

### 1.1 问题陈述

canonical prices 数据集在物理上由 38 个 `run_id=*` 分区目录组成（`lake_layout.canonical_dataset_root` 下 `run_id=<run_id>` 子目录，`_canonical_paths` 用 `root.rglob("*.parquet")` 一次性 glob 全部分区，`_read_paths` 用 `pd.concat` 拼接）。由于多个 run 写入了重叠的 `(symbol, trade_date)` 区间且读层无去重（`read_dataset` 路径中无 `drop_duplicates`），`read_dataset` 返回的帧对同一 `(symbol, trade_date)` 可能出现多行，污染下游因子/回测。

当前状态（handoff §1.1/§3.2 C2）：(1) 38 个 run_id 分区重叠；(2) `read_canonical`/`read_dataset` glob-concat 全量分区；(3) 读层无去重保证；(4) 无任何画像机制告诉运维/数据负责人"哪些键重复、来自哪些 run"。在 Wave1 基线门语境下，结构性变更（V1/C1/R1/C2b/N1）启动前必须先冻结基线，而冻结基线的最后一步就是用 C2a 画像把"重复指纹"显式化，使后续 C2b 去重有可归因的 before 快照（REQ-247/D6 强制基准）。

### 1.2 目标（量化）

| 目标 ID | 目标 | 量化成功标准 |
|---|---|---|
| G-S03-1 | 重复键画像完整 | 对 prices（38 run_id 分区）执行画像，输出全部重复 `(symbol, trade_date)` 键清单；重复键计数 = 画像前 `read_dataset(prices)` 帧按 `(symbol, trade_date)` 分组后 count>1 的组数（机械核验，脚本对账，不手算） |
| G-S03-2 | 来源 run 可追溯 | 每个重复键列出全部贡献 run_id（来源分区目录 `run_id=` 段 + 行内 `source_run_id` 列双源对账），来源 run_id 计数 = 该键在原始 38 分区里的出现分区数 |
| G-S03-3 | 独立只读 | 画像函数不修改 `read_dataset`/`_canonical_paths`/`_entry_paths`/`_read_paths`/`_filter_frame` 任何读路径函数；不写入 lake；不触发 provider；`lake_write`/`provider_fetch`/`credential_read` 安全计数 = 0 |
| G-S03-4 | 与 S01 清册交叉核对 | 画像输出重复键总数与 S01 清册 `duplicate_keys` 字段一致（差异 = 0）；不一致时画像以逐分区机械统计为准并标注 S01 清册偏差 |
| G-S03-5 | 基线门可归因 | 画像产物作为 S02 黄金值基线的"重复指纹 before 快照"固化；S07 C2b 去重后可对画像做 after 对比，归因"结构修复 vs 历史数据变化" |

### 1.3 验收标准（量化，可验证）

1. 执行画像命令（dry-run / static fixture）对 prices 38 分区输出重复键清单 + 来源 run_id；重复键计数与脚本逐分区机械重算一致（差异=0）。
2. 画像函数为独立只读分析函数，grep 确认未修改 `read_dataset`/`_canonical_paths`/`_entry_paths`/`_read_paths`/`_filter_frame` 函数体。
3. 画像输出与 S01 清册 `duplicate_keys` 交叉核对，差异=0 或显式标注 S01 偏差原因。
4. 画像产物落盘为结构化 JSON/CSV（非自由文本），可被 S07 C2b 与 S02 黄金值基线消费。
5. 安全计数 `lake_write=0`、`provider_fetch=0`、`credential_read=0`、`physical_partition_migration=0`。

### 1.4 约束

- LCQ-01 resolved：C2a 用**独立只读分析函数**，不改读路径；C2b（S07）才嵌入 `read_dataset` 出口去重。本 Story 严格不改读路径。
- Wave1 基线门第 3 步（REQ-247）：S03 依赖 S01/S02 verified；S03 verified 后基线冻结，结构性变更（含 S07）才能实现。
- 不授权（REQ-248）：runtime/NAS/QMT/trading/provider-lake-catalog 写入/物理分区迁移/Git remote write/真实 lake 写入。本 Story 只读分析。
- 既有合同闭环非新建（REQ-249）：`_canonical_paths`/`_read_paths` 合同已存在，C2a 消费既有路径枚举与读取合同，不重复设计，仅新增分析函数。

---

## 2. 模块拆分与文件影响范围

### 2.1 文件影响

| 文件 | 影响类型 | 切片边界 | merge_order |
|---|---|---|---|
| `market_data/readers.py` | 新增函数（只读分析） | 新增 `profile_duplicate_fingerprints`（及私有 helper `_collect_partition_run_map`），**不修改** `_canonical_paths`(:375)/`_read_paths`(:406)/`_entry_paths`(:412)/`read_dataset`(:2728)/`read_canonical`(:2691)/`_filter_frame` | S03 属 W1-BASELINE-FREEZE，先于 readers_py merge_order 链（S05→S06→S07→…）落地；S03 新函数与 S07 `read_dataset` 修改是不同函数，无冲突 |

### 2.2 readers.py 切片边界（S03 vs S07，LCQ-01/LCQ-02 resolved）

| 切片 | S03 C2a（本 Story） | S07 C2b（后续） |
|---|---|---|
| 函数 | 新增 `profile_duplicate_fingerprints` + `_collect_partition_run_map` | 修改 `read_dataset` 出口 + 新增 `_dedup_by_latest_run` helper |
| 是否改读路径 | 否（只读分析，不改 `read_dataset` 行为） | 是（在 `read_dataset` 出口插入去重） |
| 触点行 | 调用 `_canonical_paths`(:375) 枚举分区（只读调用） | `read_dataset`(:2728) `_read_paths`(:2828) 之后、`_filter_frame` 之前/之后 |
| 时序 | W1-BASELINE-FREEZE（基线门第 3 步，先） | W1-STRUCTURAL-P0（merge_order 第 3 位，S05/S06 后） |

> 边界保证：S03 新增函数不触碰 S07 将修改的 `read_dataset` 函数体，二者函数级正交。S03 落地后 S05/S06/S07 按 readers_py merge_order 串行修改 `read_dataset`/新增 `read_panel_as_of`/`read_panel`/去重层，互不覆盖 S03 的新函数。

### 2.3 模块职责

- **画像入口函数** `profile_duplicate_fingerprints`：接收 dataset（默认 prices）、lake_root、可选 filters；枚举 canonical 分区，逐分区读取（保留来源 run_id 标记），按主键分组，输出重复键 + 来源 run_id。
- **分区映射 helper** `_collect_partition_run_map`：把 `_canonical_paths` 返回的每个 parquet 路径映射回其 `run_id=` 分区段，并与行内 `source_run_id` 列双源对账。

---

## 3. 数据模型 / schema 变更

### 3.1 不变更

- 本 Story **不变更**任何 lake parquet schema、catalog schema、manifest schema。
- 不新增持久化表；画像产物是离线报告产物（JSON/CSV），落盘到 reports/quality 目录（与 S01 清册同目录约定），不写入 lake canonical/gold/catalog。

### 3.2 画像产物 schema（新增离线产物，非 lake schema）

| 字段 | 类型 | 含义 |
|---|---|---|
| `dataset` | str | 画像目标 dataset（默认 prices） |
| `total_partitions_scanned` | int | 扫描的 run_id 分区数（验收：=38 for prices） |
| `duplicate_key_count` | int | 重复 `(symbol, trade_date)` 键数（count>1 的组数） |
| `duplicate_keys` | list[{key, run_ids, row_counts, source_run_ids}] | 每个重复键：键值、贡献分区 run_id、每分区行数、行内 source_run_id 对账 |
| `partition_run_map` | list[{path, run_id_from_path, source_run_id_from_data, mismatch}] | 路径 run_id 段 vs 行内 source_run_id 列对账；mismatch 标注不一致 |
| `cross_check_with_inventory` | {inventory_duplicate_keys, profile_duplicate_keys, diff, diff_reason} | 与 S01 清册交叉核对 |

### 3.3 主键约定

- prices 主键：`(symbol, trade_date)`（REQ-205 明确）。
- 画像函数接受 `dedup_keys` 参数（默认 `("symbol","trade_date")`），为后续 dataset 扩展预留；本 Story 验收只针对 prices。

---

## 4. 接口契约

### 4.1 画像入口函数签名

```python
def profile_duplicate_fingerprints(
    dataset: str = "prices",
    lake_root: str | Path | None = None,
    *,
    dedup_keys: Sequence[str] = ("symbol", "trade_date"),
    filters: Mapping[str, Any] | None = None,
) -> DuplicateFingerprintReport:
    """只读画像：枚举 canonical run_id 分区，输出重复主键 + 来源 run_id。

    不修改读路径；不写入 lake；不触发 provider。
    LCQ-01: C2a 独立只读分析函数（不改读路径）。
    """
```

### 4.2 返回类型

```python
@dataclass
class DuplicateFingerprintReport:
    dataset: str
    total_partitions_scanned: int
    duplicate_key_count: int
    duplicate_keys: list[dict]          # 见 §3.2
    partition_run_map: list[dict]
    cross_check_with_inventory: dict | None
    safety_counters: dict               # lake_write=0, provider_fetch=0, credential_read=0
    generated_at: str
```

### 4.3 与相邻 Story 的契约方向

| 方向 | 对端 Story | 契约 |
|---|---|---|
| 消费（上游） | S01 T8 清册 | 读取 S01 清册产物 `duplicate_keys` 字段做交叉核对（§3.2 cross_check） |
| 消费（上游） | S02 T7 黄金值 | 画像产物作为 S02 黄金值基线的"重复指纹 before 快照"固化入基线 |
| 产出（下游） | S07 C2b 读层去重 | S07 消费画像的"重复键 + 来源 run_id"作为去重策略输入（按 source_run_id 取最新）；S07 去重后对画像做 after 对比归因 |
| 产出（下游） | S29 T3 去重正确性测试 | S29 T3 fixture 可复用画像揭示的重复键构造回归用例 |

> 契约方向单向：S03 只读产出，不反向依赖 S07/S29。S07 实现时消费 S03 画像产物，但 S03 不等待 S07。

### 4.4 与既有合同的关系（REQ-249）

- 消费既有 `_canonical_paths`（路径枚举合同）、`pd.read_parquet`（读取合同）。不重复设计路径枚举，只在其上新增分析逻辑。
- 不消费 `read_dataset`（避免与 S07 切片混淆）；画像直接走 `_canonical_paths` + 逐分区 `pd.read_parquet`，独立于 `read_dataset` 读路径。

---

## 5. 关键流程（含 Wave1 基线门时序）

### 5.1 Wave1 基线门时序（REQ-247，必须落入流程）

```
S01 T8 清册 (verified) → S02 T7 黄金值基线 (verified) → S03 C2a 重复画像 (verified)
                                                          ↓
                                                  基线冻结门（REQ-247）
                                                          ↓
S04 V1 pointer / S05 C1 PIT / S06 R1 panel / S07 C2b 去重 / S08 N1 后置（结构性变更才允许）
```

- S03 是基线冻结**最后一步**：S01/S02/S03 三步 verified 后基线冻结，结构性变更（含 S07）才能实现。
- S03 verified 前不得启动 S04-S09/S31（REQ-247 硬依赖）。
- S03 画像产物固化入 S02 黄金值基线，使 S07 去重后可归因"结构修复 vs 历史数据变化"（D6 强制基准）。

### 5.2 画像主流程

```
profile_duplicate_fingerprints(dataset="prices", lake_root, dedup_keys)
  ├─ 1. 解析 lake_root（_lake_root，复用既有 helper）
  ├─ 2. layout = LakeLayout(root); canonical_root = layout.canonical_dataset_root(dataset, SCHEMA_VERSION)
  ├─ 3. paths = _canonical_paths(layout, dataset)   # 复用既有路径枚举（只读调用）
  ├─ 4. 对每个 path：
  │     ├─ 从 path 解析 run_id 分区段（path.parts 中 "run_id=<x>" 段）→ run_id_from_path
  │     ├─ df_i = pd.read_parquet(path)
  │     ├─ run_id_from_data = df_i["source_run_id"].mode()（行内列对账）
  │     ├─ 标记每行 _partition_run_id = run_id_from_path（不改 df_i schema，临时列）
  │     └─ 收集 (path, run_id_from_path, run_id_from_data, mismatch)
  ├─ 5. all_df = pd.concat([df_i 带 _partition_run_id], ignore_index=True)
  ├─ 6. 按 dedup_keys 分组：grouped = all_df.groupby(list(dedup_keys))
  ├─ 7. 重复组 = grouped.size()[size>1]
  ├─ 8. 对每个重复组：收集 {key, run_ids=unique(_partition_run_id), row_counts, source_run_ids}
  ├─ 9. 交叉核对：读取 S01 清册产物 duplicate_keys，对比 diff
  ├─ 10. 组装 DuplicateFingerprintReport（safety_counters 全 0）
  └─ 11. 返回（可选落盘到 reports/quality/，由调用方/CLI 决定，本函数默认返回对象）
```

### 5.3 关键不变量

- 画像不调用 `read_dataset`（独立于读路径，避免与 S07 切片耦合）。
- 画像不调用 `read_canonical`（同上）。
- 画像只读 parquet + 只读 S01 清册产物；无任何写操作。
- `source_run_id` 列缺失时（部分 dataset 可能缺），降级为仅用 path run_id 段，并在 `partition_run_map.mismatch` 标注。

---

## 6. 异常处理与失败路径

| 场景 | 失败行为 | 终止/降级 |
|---|---|---|
| lake_root 缺失 | 返回 `status="required_missing"`，issues=[{code:"lake_root_missing"}] | 终止（不降级） |
| canonical_root 不存在 | `total_partitions_scanned=0`，`duplicate_key_count=0`，issues=[{code:"canonical_missing"}] | 终止（空报告，不伪造） |
| dataset 不在 DATASETS | 返回 `status="unavailable"`，issues=[{code:"unknown_dataset"}] | 终止 |
| 某 parquet 读取异常 | 跳过该分区，issues 追加 `{code:"partition_read_error", path, error}`；继续其余分区 | 降级（部分画像） |
| `source_run_id` 列缺失 | 降级为仅 path run_id 段；`partition_run_map.mismatch="source_run_id_column_missing"` | 降级（标注） |
| S01 清册产物缺失 | `cross_check_with_inventory=None`，issues=[{code:"inventory_not_found"}] | 降级（跳过交叉核对） |
| 与 S01 清册 diff≠0 | `cross_check_with_inventory.diff=…`，`diff_reason` 标注（画像以逐分区机械统计为准） | 继续并标注 |

> 规则 5：每个执行阶段定义前置校验和失败行为。画像不静默成功：canonical_missing 时返回空报告 + issue，不返回伪造的 0 重复。

---

## 7. 测试设计

### 7.1 fixture 策略（static-only，CP6/CP7 默认）

| 测试 | 目的 | fixture | 断言 |
|---|---|---|---|
| `test_profile_duplicate_fingerprints_detects_overlap` | 重复键检测准确性 | 构造 tmp lake，3 个 run_id 分区写入重叠 `(symbol,trade_date)`（symbol=A,date=D1 在 run1+run2 各 1 行） | `duplicate_key_count==1`，`duplicate_keys[0].run_ids=={"run1","run2"}` |
| `test_profile_no_duplicates` | 无重复时 0 误报 | 3 个 run_id 分区写入不相交 (symbol,date) | `duplicate_key_count==0` |
| `test_source_run_id_mismatch_flagged` | path vs 行内 source_run_id 对账 | path run_id=run1 但行内 source_run_id=run2 | `partition_run_map[*].mismatch` 非空 |
| `test_readonly_no_read_dataset_modification` | 独立只读不变读路径 | grep/AST 断言 `read_dataset`/`_canonical_paths`/`_read_paths` 函数体未被本 Story 修改 | AST 对比基线，0 diff |
| `test_safety_counters_zero` | 安全计数 | 任意 fixture | `lake_write==0, provider_fetch==0, credential_read==0` |
| `test_cross_check_with_inventory` | 与 S01 清册交叉核对 | mock S01 清册产物 duplicate_keys | `cross_check_with_inventory.diff==0`（一致时）或显式 diff_reason |
| `test_38_partition_scale` | 38 分区规模不崩 | 构造 38 个 run_id 分区小 fixture | 画像完成且 OOM 不发生；`total_partitions_scanned==38` |

### 7.2 T3 测试关联

- S29 T3（去重正确性回归）fixture 可复用 S03 画像揭示的重复键模式构造"重复键 fixture → 去重 read 后唯一"用例。S03 画像为 S29 T3 提供真实重复键样本。

### 7.3 验证命令

```
uv run --python 3.11 pytest -q tests/test_cr139_duplicate_fingerprint_profiling.py
```

---

## 8. 实施步骤（最小实现切片）

> CP5 通过后才实施。本节是实施切片规划，不含实现代码。

| Slice | 内容 | 依赖 |
|---|---|---|
| S03-S1 | 新增 `DuplicateFingerprintReport` dataclass + `profile_duplicate_fingerprints` 函数骨架（lake_root 解析 + canonical_root 校验 + 空报告返回） | 无 |
| S03-S2 | 实现 `_collect_partition_run_map`：path → run_id 段解析 + 逐分区 `pd.read_parquet` + source_run_id 对账 | S03-S1 |
| S03-S3 | 实现分组重复检测：concat → groupby(dedup_keys) → size>1 → 收集 run_ids/source_run_ids | S03-S2 |
| S03-S4 | 实现 S01 清册交叉核对：读取 S01 产物 duplicate_keys → diff 计算 | S03-S3 |
| S03-S5 | fixture 测试（§7.1 全部）+ safety_counters | S03-S4 |
| S03-S6 | （可选）CLI 入口：`python -m market_data.readers profile-duplicates --dataset prices` dry-run | S03-S5 |

---

## 9. 风险与应对

| 风险 ID | 风险 | 应对 |
|---|---|---|
| R-S03-1 | 38 分区全量 concat 内存峰值 | 逐分区读取 + 仅保留 dedup_keys + run_id 标记列做分组，不全量持有原始宽表；若仍 OOM 标注 open_item 转 S13 DuckDB 只读（W2） |
| R-S03-2 | `source_run_id` 列在部分分区缺失/全空 | 降级为 path run_id 段 + mismatch 标注（§6）；不伪造对账 |
| R-S03-3 | 与 S01 清册 diff≠0 归因困难 | 画像以逐分区机械统计为准；diff_reason 标注"S01 清册统计口径差异"或"S01 清册过时" |
| R-S03-4 | 画像函数误改读路径（违反 LCQ-01） | AST/grep 自检（§7.1 test_readonly_no_read_dataset_modification）；CP6 逐项核验 |
| R-S03-5 | 基线门时序被绕过（S07 在 S03 verified 前实现） | REQ-247 硬依赖写入 Story dev_gate；CP6 检查 S03 verified 状态 |

---

## 10. 发布与回滚策略

- **发布**：S03 是只读分析函数，无数据迁移、无 schema 变更、无 lake 写入。发布 = 代码合入 + 画像产物落盘到 reports/quality/。
- **回滚**：删除新增函数 + 删除画像产物文件。无状态回滚，无数据回滚。
- **基线不可回滚**：S03 画像产物一旦固化入 S02 黄金值基线，不得删除/覆盖（D6 强制基准）；后续 S07 去重后做 after 对比依赖此 before 快照。

---

## 11. 平台差异检查

| 维度 | 检查 | 结论 |
|---|---|---|
| 平台目标 | 日频离线研究 + 模拟盘场景；DuckDB 只读引擎（D4）；parquet 仍是存储 | 适用；画像只用 pandas + parquet，不引入 DuckDB 依赖（DuckDB 仅 S13 W2） |
| Python 环境 | uv 管理；`uv run --python 3.11` | 遵循；测试命令用 `uv run` |
| 路径构造 | `lake_layout.canonical_dataset_root` + `run_id=` 分区段 | 复用既有 LakeLayout，不推断路径 |
| 安装结构 | 不涉及安装器/Guardrail/平台适配 | N/A |

---

## 12. 设计契约映射

| LLD 章节 | HLD 章节 | ADR | REQ | Story |
|---|---|---|---|---|
| §1 目标 G-S03-1..5 | HLD §1.3 G-1（去重唯一）、§12.3 Wave1 基线门 | Wave1 基线门（REQ-247） | REQ-205(C2a)、REQ-247、REQ-248 | S03 |
| §2 文件影响 readers.py | HLD §10 读侧模块（`readers.py:2728 read_dataset`） | AGA-1 A1（读写分离） | REQ-249（既有合同闭环） | S03/S07 切片 |
| §4 接口契约 | HLD §10 读侧、§18（reader 接口下沉 Feature 设计） | — | REQ-205(C2a) | S03 → S07 消费 |
| §5.1 基线门时序 | HLD §12.3（T8→T7→C2a→基线冻结门→结构性变更） | Wave1 基线门 | REQ-247 | S01→S02→S03→S04-S08 |
| §7 测试设计 T3 关联 | HLD §8 SIM-139-01（C2a 画像→C2b 去重） | — | REQ-205(C2a) | S03 → S29 T3 |
| §1.4 不授权 | HLD §1.4 约束 + Gotchas#1 | 不授权范围声明 | REQ-248 | S03 |
| §4.4 既有合同 | HLD §10 + Gotchas#2（d2 闭环非新建） | D8b | REQ-249 | S03 消费 _canonical_paths |

---

## 13. Gotchas

> 实质性 Gotchas，非占位（规则 9）。

1. **"画像不改读路径"是 LCQ-01 硬约束，不是风格偏好**：C2a 画像函数如果顺手在 `read_dataset` 里加几行"顺便去重"，会与 S07 C2b 的 merge_order 切片冲突，且违反基线门时序（S03 是基线冻结步，S07 是结构性变更步，S07 必须在 S03 verified 后才实现）。画像必须独立调用 `_canonical_paths` + 逐分区 `pd.read_parquet`，完全不经过 `read_dataset`。AST 自检必须覆盖。

2. **`source_run_id` 列与 path `run_id=` 段是两个来源，必须双源对账**：handoff §3.2 C2 触点 `readers.py:375/409` 指向 `_canonical_paths`（path 维度）和 `_read_paths`（concat 维度）。`source_run_id` 是行内列（已在 readers.py:1779/2162 等多处引用，确认存在），path `run_id=` 段是目录维度。二者不一致意味着某分区写入时 source_run_id 标记错误（数据血缘问题），画像必须标注 mismatch 而非静默取其一。这是后续 N2/N3 命名规约（W2）要修的根因，S03 只揭示不修复。

3. **38 分区是 prices 的现状，不是恒定常量**：handoff §1.1 记录"prices 38 run_id 分区"，但分区数会随新 run 写入增长。画像 `total_partitions_scanned` 必须动态枚举（`_canonical_paths` 实时 rglob），不得硬编码 38。验收时"38"是 before 快照断言，after 快照（S07 去重后）分区数不变（S07 只读层去重，N1 才物理迁移）。

4. **画像产物是基线证据，不可覆盖**：D6 强制基准要求 S07 去重后能对 T7 黄金值归因。S03 画像 = 重复指纹 before 快照，固化入 S02 基线后，任何"重新跑画像覆盖旧产物"都会破坏归因链。产物必须带 `generated_at` 时间戳 + run 上下文，版本化保留，不得 in-place 覆盖。

5. **与 S01 清册交叉核对不是"必须一致"，而是"差异可归因"**：S01 清册 `duplicate_keys` 是清册时点统计，S03 画像是画像时点统计。若两次之间有新 run 写入，diff≠0 是正常的。画像必须输出 diff + diff_reason，而不是强制 diff=0。把"必须一致"当验收会掩盖真实的分区增长。

---

## 14. open_items / tier / shared_fragments

### open_items

| ID | 描述 | 阻断 | 重访条件 |
|---|---|---|---|
| O-S03-1 | `source_run_id` 列在非 prices dataset 的覆盖情况未核验（本 Story 验收只针对 prices） | 否 | 扩展到其他 dataset 时核验列覆盖；若缺失则降级 path run_id 段 |
| O-S03-2 | 38 分区全量 concat 内存峰值在真实 lake 规模下的表现未测（fixture 是小规模） | 否 | CP7 runtime 验证时（需独立授权）测真实 lake；若 OOM 转 S13 DuckDB 只读（W2） |
| O-S03-3 | 画像产物落盘路径（reports/quality/ vs process/）需与 S01 清册目录约定对齐 | 否 | 实施时确认 S01 清册产物路径约定，保持同目录 |

### tier

- `tier=P0`（Wave1 基线门第 3 步，解锁结构性变更）

### shared_fragments

- 复用 `_canonical_paths`（readers.py:375，路径枚举合同）
- 复用 `_lake_root`（lake_root 解析 helper）
- 复用 `LakeLayout.canonical_dataset_root`（lake_layout.py:131，路径构造合同）
- 画像产物 schema（§3.2）被 S07 C2b / S02 黄金值 / S29 T3 消费

### lld_gate

- required: true
- required_level: full-lld
- status: ready-for-cp5-review（本 LLD 提交后等待 CP5-W1 全量人工确认）

### dev_gate

- cp5_required: true
- implementation_allowed_before_cp5: false
- baseline_gate: S03 是基线冻结第 3 步；S03 verified 前不得启动 S04-S09/S31

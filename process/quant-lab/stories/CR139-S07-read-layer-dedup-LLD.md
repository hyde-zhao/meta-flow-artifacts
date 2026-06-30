---
story_id: "CR139-S07"
title: "C2b 读层去重 — LLD"
change_id: "CR-139"
owner_feature: "FEAT-02 读侧"
lld_policy:
  required_level: "full-lld"
wave: "CR139-W1-STRUCTURAL-P0"
status: "lld-ready-for-review"
req_coverage: ["REQ-205(C2b)"]
depends_on: ["CR139-S06"]
dependency_type: ["contract"]
source_hld: "process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md"
source_adr: "process/docs/design/ARCHITECTURE-DECISION-STRATEGY-DATA-FOUNDATION.md"
feature_design_matrix: "docs/design/FEATURE-DESIGN-MATRIX.md v1.13"
story_card: "process/stories/CR139-S07-read-layer-dedup.md"
created_at: "2026-06-28T18:35:00+08:00"
delegated_by: "host-orchestrator"
delegated_to: "meta-dev"
not_authorized: "实现/runtime/真实 lake 写入/物理分区迁移/Git remote write（CP5 通过后才实现）"
---

# CR139-S07 C2b 读层去重 — 详细设计（LLD）

> 本文件是 CR-139 Wave1 结构性 P0 C2b 读层去重的 full-lld 设计证据。消费 companion HLD v0.2（confirmed-cp3）、ADR v0.2、MATRIX v1.13、REQUIREMENTS v1.17 REQ-205(C2b)/REQ-247/REQ-248/REQ-249。本 LLD 是设计证据，不含实现代码；CP5-W1 全量人工确认通过、基线门（S01/S02/S03 verified）满足且 Story 进入 dev-ready 后才允许实现。

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 0.1 | 2026-06-28 | meta-dev | CR-139 CP5-W1 首版 full-lld：14 章齐全；嵌入 read_dataset 出口去重（LCQ-01 resolved）；readers.py 函数切片 + merge_order 第 3 位（LCQ-02 resolved）；按 source_run_id 取最新；Wave1 基线门依赖 S01/S02/S03 verified；量化验收 + 实质性 Gotchas。 |

---

## 1. 问题定义与目标

### 1.1 问题陈述

canonical prices 数据集物理上由 38 个 `run_id=*` 分区组成（`_canonical_paths` 用 `root.rglob("*.parquet")` glob 全部分区，`_read_paths` 用 `pd.concat` 拼接，`read_dataset` 在 :2778 published gate 通过后调用二者）。多个 run 写入重叠 `(symbol, trade_date)` 区间，`read_dataset` 路径中无任何去重步骤（已 grep 核验：`read_dataset`/`_read_paths`/`_filter_frame` 均无 `drop_duplicates`），导致返回帧对同一 `(symbol, trade_date)` 出现多行，污染下游因子/回测（前视/版本漂移）。

C2a（S03）画像已揭示"哪些键重复、来自哪些 run"（before 快照）。C2b（本 Story）在读层**嵌入 `read_dataset` 出口去重**，按 `source_run_id` 取最新版本，保证读层返回的 `(symbol, trade_date)` 唯一。这是读侧四道 P0 防线之一（去重唯一，HLD §12.2 G-1）。

### 1.2 目标（量化）

| 目标 ID | 目标 | 量化成功标准 |
|---|---|---|
| G-S07-1 | 读层去重唯一 | `read_dataset("prices")` 去重后按 `(symbol, trade_date)` 分组 count>1 的组数 = 0（重复键=0） |
| G-S07-2 | 按来源取最新 | 对每个重复键，保留的行其 `source_run_id` 在该键全部贡献 run 中按"最新"排序居首（catalog `latest_manifest_run_id` 优先，其后确定性 tiebreaker） |
| G-S07-3 | 嵌入读路径出口 | 去重步骤嵌入 `read_dataset` 出口（`_read_paths` 返回后、`_filter_frame` 前后边界明确），`read_canonical`/`read_panel_as_of`（S05）/`read_panel`（S06）复用同一去重语义 |
| G-S07-4 | 不物理迁移 | 只读层去重，不删除/合并/重命名 run_id 分区目录；物理分区收敛后置到 S08 N1（基线冻结后）；`physical_partition_migration` 计数 = 0 |
| G-S07-5 | 基线门可归因 | 去重后对 S03 C2a 画像做 after 对比，归因"结构修复"（重复键消失）vs"历史数据变化"（新增/消失键） |
| G-S07-6 | T3 回归通过 | S29 T3 去重正确性回归：重复键 fixture 去重 read 后 `(symbol,trade_date)` 唯一 |

### 1.3 验收标准（量化，可验证）

1. `read_dataset("prices")` 返回帧按 `(symbol, trade_date)` 分组，count>1 组数 = 0。
2. 去重步骤在 `read_dataset` 函数体内（AST/grep 确认插入位置），`read_canonical` 等共享读路径复用。
3. 去重保留行的 `source_run_id` 符合"最新"策略（catalog `latest_manifest_run_id` 优先 + 确定性 tiebreaker）。
4. 不修改/删除/重命名任何 run_id 分区目录（`physical_partition_migration=0`）。
5. 安全计数 `lake_write=0`、`provider_fetch=0`、`credential_read=0`。
6. S29 T3 fixture 回归通过（after CP6/CP7）。

### 1.4 约束

- LCQ-01 resolved：C2b 嵌入 `read_dataset` 出口按 `source_run_id` 取最新（C2a S03 才是独立只读分析函数）。
- LCQ-02 resolved：readers.py 函数切片 + merge_order 串行 S05→S06→S07→S09→S25→S27→S28→S13→S16→S36。S07 是 merge_order 第 3 位（基于 S05 C1 / S06 R1 契约）。
- Wave1 基线门（REQ-247）：S07 是结构性变更，依赖 S01/S02/S03 verified（基线冻结）+ S06（panel reader 读路径契约）。基线冻结前不得实现。
- 不授权（REQ-248）：runtime/NAS/QMT/trading/provider-lake-catalog 写入/物理分区迁移/Git remote write/真实 lake 写入。S07 只读层去重，不物理迁移（N1 后置）。
- 既有合同闭环（REQ-249）：`read_dataset`/`_read_paths`/`_filter_frame` 合同已存在，C2b 在既有读路径上插入去重，不重复设计读路径。

---

## 2. 模块拆分与文件影响范围

### 2.1 文件影响

| 文件 | 影响类型 | 切片边界 | merge_order |
|---|---|---|---|
| `market_data/readers.py` | 修改 `read_dataset` + 新增 helper | 在 `read_dataset`(:2728) 出口插入去重调用；新增 `_dedup_by_latest_run` helper；不修改 `_canonical_paths`(:375)/`_entry_paths`(:412)（S03 已声明只读调用，不冲突） | readers_py merge_order 第 3 位：S05→S06→**S07**→S09→… |

### 2.2 readers.py 切片边界（S07 vs S03，LCQ-01/LCQ-02 resolved）

| 切片 | S07 C2b（本 Story） | S03 C2a（前置，已 LLD） |
|---|---|---|
| 函数 | 修改 `read_dataset`(:2728) + 新增 `_dedup_by_latest_run` | 新增 `profile_duplicate_fingerprints` + `_collect_partition_run_map` |
| 是否改读路径 | 是（`read_dataset` 出口插去重） | 否（独立只读分析，不改读路径） |
| 触点行 | `read_dataset` :2728 / `_read_paths` 返回后 :2828 / `_filter_frame` 边界 | 调用 `_canonical_paths`(:375) 只读枚举 |
| 时序 | W1-STRUCTURAL-P0（merge_order 第 3 位，S05/S06 后） | W1-BASELINE-FREEZE（基线门第 3 步，先） |
| 依赖 | S06（panel reader 读路径契约）+ 基线门 S01/S02/S03 | S01/S02 |

> 边界保证：S03 新增独立函数不触碰 `read_dataset` 函数体；S07 修改 `read_dataset` 不触碰 S03 的 `profile_duplicate_fingerprints`。二者函数级正交，merge_order 串行无冲突。

### 2.3 readers.py merge_order 第 3 位上下文

```
readers_py merge_order: S05(C1 read_panel_as_of) → S06(R1 read_panel) → S07(C2b 去重) → S09(V4 兼容回退) → S25(X3) → S27(L3) → S28(L4) → S13(R3) → S16(V2) → S36(R4)
```

- S07 基于 S05/S06 已建立的 `read_panel_as_of`/`read_panel` 读路径契约，去重语义统一应用于 `read_dataset` 出口，被下游 `read_panel_as_of`/`read_panel` 复用。
- S07 实现时确认每 Story 函数级切片边界（LCQ-02 resolved 要求）：S07 只改 `read_dataset` + 新增 `_dedup_by_latest_run`，不改 S05 的 `read_panel_as_of`、S06 的 `read_panel` 函数体（它们调用 `read_dataset` 间接受益于去重）。

### 2.4 模块职责

- **`_dedup_by_latest_run` helper**：接收 concat 后的 DataFrame + catalog entry（含 `latest_manifest_run_id`）+ dedup_keys + latest_by 策略；按 dedup_keys 分组，组内按"最新"排序保留首行。
- **`read_dataset` 修改**：在 `_read_paths` 返回后、`_filter_frame` 之前调用 `_dedup_by_latest_run`（去重应在过滤前，避免过滤后丢失用于排序的 source_run_id 上下文；具体边界实施时按 fixture 验证确定，本 LLD 锁定"去重在 concat 后、用户 filters 前"）。

---

## 3. 数据模型 / schema 变更

### 3.1 不变更

- 本 Story **不变更**任何 lake parquet schema、catalog schema、manifest schema。
- 不新增持久化列；`source_run_id` 列已存在于数据中（readers.py:1779/2162 等多处引用，已 grep 核验）。
- 不新增 run_id 分区目录；不删除/合并/重命名既有 run_id 分区（N1 后置）。

### 3.2 去重语义模型（非 schema，是读层行为契约）

| 概念 | 定义 |
|---|---|
| dedup_keys | 数据集自然主键；prices = `(symbol, trade_date)`（REQ-205）；函数接受 `dedup_keys` 参数默认 `("symbol","trade_date")` |
| latest_by 策略 | 组内"最新"排序：(1) `source_run_id == entry.latest_manifest_run_id` 的行优先（catalog current run）；(2) 其余按 `source_run_id` 字符串降序（确定性 tiebreaker，假设 run_id 命名编码时序，N2 W2 将形式化）；(3) `source_run_id` 缺失/空时按 path run_id 段降序（行内列缺失降级） |
| 保留规则 | 每组保留排序首行；其余行丢弃（不写回 lake，只读层丢弃） |
| 不变量 | 去重后 `(symbol,trade_date)` 唯一；去重不改变保留行的字段值（不合成/不覆盖） |

### 3.3 主键扩展性

- prices 主键 `(symbol, trade_date)` 是 P0 验收目标。
- 其他 dataset（financial_pit/market_cap 等）主键可能含 `event_date`/`report_date` 等；`dedup_keys` 参数化预留。本 Story 验收只针对 prices；其他 dataset 的去重键在实施时按 dataset schema 确认（open_item）。

---

## 4. 接口契约

### 4.1 去重 helper 签名

```python
def _dedup_by_latest_run(
    frame: pd.DataFrame,
    *,
    entry: CatalogEntry | None,
    dedup_keys: Sequence[str] = ("symbol", "trade_date"),
    latest_by: str = "source_run_id",
) -> pd.DataFrame:
    """读层去重：按 dedup_keys 分组，组内按 latest_by 取最新版本。

    LCQ-01: 嵌入 read_dataset 出口；按 source_run_id 取最新。
    不写 lake；不删除分区；只读层丢弃重复行。
    """
```

### 4.2 read_dataset 修改契约

```python
def read_dataset(...) -> ReaderResult:
    ...  # published gate (:2778) + quality gate 不变
    paths = [path for path in _entry_paths(layout, dataset, entry) if path.exists()]
    ...
    frame = _read_paths(paths)                       # 既有：concat 全分区
    # === S07 C2b 插入点：去重 ===
    frame = _dedup_by_latest_run(
        frame,
        entry=entry,
        dedup_keys=_dataset_dedup_keys(dataset),    # prices -> (symbol,trade_date)
        latest_by="source_run_id",
    )
    # === S07 插入结束 ===
    filters = dict(filters or {})
    frame = _filter_frame(...)                        # 既有：用户 filters
    ...
```

### 4.3 与相邻 Story 的契约方向

| 方向 | 对端 Story | 契约 |
|---|---|---|
| 消费（上游） | S03 C2a 画像 | S07 去重策略（按 source_run_id 取最新）的"before 重复指纹"来自 S03 画像；去重后做 after 对比归因 |
| 消费（上游） | S05 C1 PIT / S06 R1 panel | S07 去重嵌入 `read_dataset` 出口，`read_panel_as_of`（S05）/`read_panel`（S06）调用 `read_dataset` 间接受益；S07 基于 S05/S06 读路径契约（merge_order 第 3 位） |
| 消费（上游） | S01/S02/S03 基线门 | S07 是结构性变更，基线冻结（S01/S02/S03 verified）前不得实现（REQ-247） |
| 产出（下游） | S29 T3 去重正确性测试 | S07 去重语义被 S29 T3 fixture 回归验证（重复键 fixture → 去重 read 后唯一） |
| 产出（下游） | S32 C4 写入去重 | S07 读层去重语义对应 S32 写入去重（写侧保证 + 读侧保证双闭环）；S32 消费 S07 语义 |
| 产出（下游） | S08 N1 物理分区迁移 | S07 读层去重就位后 N1 物理迁移更安全（N1 依赖 S07，merge_order + 基线门） |

> 契约方向：S07 修改共享读路径 `read_dataset`，下游 S05/S06/S09/S25/S27/S28/S13/S16/S36（readers_py merge_order 后续）基于 S07 去重后的干净帧继续工作。

### 4.4 与既有合同的关系（REQ-249）

- 消费既有 `read_dataset`（:2728）、`_read_paths`（:406）、`_entry_paths`（:412）、`CatalogEntry`（含 `latest_manifest_run_id`/`published`）。
- 不重复设计读路径，只在既有路径出口插入去重步骤。
- `source_run_id` 列既有（行内血缘列），不新增列。

---

## 5. 关键流程（含 Wave1 基线门时序）

### 5.1 Wave1 基线门时序（REQ-247，必须落入流程）

```
S01 T8 清册 (verified) → S02 T7 黄金值 (verified) → S03 C2a 画像 (verified)
                                                          ↓
                                                  基线冻结门（REQ-247）
                                                          ↓
S04 V1 pointer → S05 C1 PIT → S06 R1 panel → S07 C2b 读层去重 → S08 N1 后置
                                  ↑_______________|
                                  S07 依赖 S06 读路径契约（merge_order 第 3 位）
```

- S07 是结构性变更，**必须**在 S01/S02/S03 verified（基线冻结）后实现。
- S07 依赖 S06（panel reader 读路径契约）：`read_panel` 调用 `read_dataset`，S07 去重嵌入 `read_dataset` 出口后 `read_panel` 自动受益。
- S07 verified 后 S08 N1（物理分区迁移）更安全（N1 依赖 S07，DEVELOPMENT-PLAN dependency_graph: S07→S08）。

### 5.2 去重主流程

```
read_dataset(dataset="prices", ...)
  ├─ 1. lake_root 解析 + catalog entry 获取（既有，不变）
  ├─ 2. published gate (:2778)（既有，不变）
  ├─ 3. quality gate（既有，不变）
  ├─ 4. paths = _entry_paths(...) (既有，不变)
  ├─ 5. frame = _read_paths(paths)               # 既有：concat 全 38 分区
  ├─ 6. 【S07 新增】frame = _dedup_by_latest_run(frame, entry, dedup_keys, latest_by="source_run_id")
  │     ├─ 6.1 校验 frame 含 dedup_keys 列；缺失返回原 frame + issue（不阻断，降级）
  │     ├─ 6.2 校验 frame 含 source_run_id 列；缺失降级为 path run_id 段（需 _read_paths 携带来源标记，见 §6）
  │     ├─ 6.3 排序键：is_current = (source_run_id == entry.latest_manifest_run_id)
  │     ├─ 6.4 groupby(dedup_keys)，组内按 (is_current desc, source_run_id desc) 排序
  │     ├─ 6.5 每组保留首行（head(1)）
  │     └─ 6.6 返回去重后 frame（行数 <= 原始；重复键=0）
  ├─ 7. frame = _filter_frame(...)               # 既有：用户 filters
  └─ 8. 返回 ReaderResult（含去重后 frame + safety_counters）
```

### 5.3 关键不变量

- 去重在 `_read_paths`（concat）后、`_filter_frame`（用户 filters）前：确保排序有完整 source_run_id 上下文，且用户 filters 不丢失重复判定所需行。
- 去重只丢弃行，不修改保留行字段值（不合成/不覆盖）。
- 去重不写 lake、不删分区、不重命名分区。
- `read_canonical`（prices 专用 fast reader，:2691）若走独立路径需同步去重；实施时确认 `read_canonical` 是否复用 `read_dataset` 或独立 concat，若独立则同样插入去重（open_item）。

---

## 6. 异常处理与失败路径

| 场景 | 失败行为 | 终止/降级 |
|---|---|---|
| frame 缺 dedup_keys 列（如 dataset 无 symbol/trade_date） | 返回原 frame + issue `{code:"dedup_keys_missing", dataset}` | 降级（不去重，标注） |
| frame 缺 source_run_id 列 | 降级为 path run_id 段排序；需 `_read_paths` 或上层携带来源 path 标记；若不可得则按行序保留首行 + issue `{code:"source_run_id_missing"}` | 降级（标注） |
| `entry.latest_manifest_run_id` 为空 | 跳过 is_current 优先级，仅按 source_run_id desc 排序 + issue `{code:"catalog_run_id_missing"}` | 降级（标注） |
| 去重后仍存在重复键（理论不应发生） | 返回 frame + issue `{code:"dedup_incomplete", remaining_dup_keys=…}`；阻断（required=True 时 status="quality_failed"） | 终止（不静默成功） |
| `read_canonical` 独立路径未同步去重 | 实施时确认并同步；若遗漏则 CP6/CP7 发现重复键≠0 时阻断 | 终止（CP6 自检） |
| 内存峰值（38 分区全量 concat + 排序） | 排序在分组内进行（不全局 sort）；若仍 OOM 标注 open_item 转 S13 DuckDB 只读（W2） | 降级（标注） |

> 规则 5：去重不静默成功。dedup_incomplete 时阻断（required dataset），不返回带重复键的帧假装通过。

---

## 7. 测试设计

### 7.1 fixture 策略（static-only，CP6/CP7 默认）

| 测试 | 目的 | fixture | 断言 |
|---|---|---|---|
| `test_read_dataset_dedup_unique` | 去重后唯一 | tmp lake，3 run_id 分区写入重叠 (symbol,trade_date)（A,D1 在 run1+run2+run3） | `read_dataset("prices")` 帧按 (symbol,trade_date) 分组 count>1 组数 == 0 |
| `test_dedup_keeps_latest_run` | 按 source_run_id 取最新 | run1(旧)+run2(新)+run3(current=catalog latest_manifest_run_id) 写同一 (A,D1) | 保留行 source_run_id == run3（current 优先） |
| `test_dedup_keeps_latest_when_no_current` | 无 catalog current 时确定性 tiebreaker | catalog latest_manifest_run_id=None；run1+run2 同键 | 保留行 source_run_id == max(run1,run2)（字典序降序） |
| `test_dedup_no_duplicates_passthrough` | 无重复时 0 副作用 | 3 分区不相交键 | 去重后行数 == 原始；字段值不变 |
| `test_dedup_source_run_id_missing_degrade` | source_run_id 缺失降级 | 行内无 source_run_id 列 | 降级 path run_id 段 + issue；不阻断 |
| `test_dedup_keys_missing_degrade` | dedup_keys 缺失降级 | dataset 无 trade_date 列 | 返回原 frame + issue；不阻断 |
| `test_no_physical_migration` | 不物理迁移 | 去重前后对比 lake 目录结构 | run_id 分区目录数不变（0 删除/重命名） |
| `test_safety_counters_zero` | 安全计数 | 任意 fixture | lake_write=0, provider_fetch=0, credential_read=0 |
| `test_read_canonical_sync` | read_canonical 同步去重 | 若 read_canonical 独立 concat | 同样去重后唯一 |

### 7.2 T3 回归关联（S29）

- S29 T3（去重正确性回归）断言 `(symbol,trade_date)` 唯一。S07 去重是 T3 验证对象。
- S29 T3 fixture 可复用 S03 画像揭示的真实重复键模式。
- S07 实施后 S29 T3 必须通过（CP6/CP7）。

### 7.3 与 S03 画像 after 对比

- S07 去重后对 S03 画像做 after 对比：before 重复键数 → after 重复键数=0，归因为"结构修复"。
- 若 after 出现 S03 before 未记录的新键，归因为"历史数据变化"（D6 强制基准）。

### 7.4 验证命令

```
uv run --python 3.11 pytest -q tests/test_cr139_read_layer_dedup.py
uv run --python 3.11 pytest -q tests/test_cr139_dedup_correctness.py   # S29 T3
```

---

## 8. 实施步骤（最小实现切片）

> CP5 通过 + 基线门满足后才实施。本节是实施切片规划，不含实现代码。

| Slice | 内容 | 依赖 |
|---|---|---|
| S07-S1 | 新增 `_dedup_by_latest_run` helper（签名 + 排序逻辑 + 降级路径）+ `_dataset_dedup_keys` 映射 | S05/S06 read 路径契约（merge_order 前置） |
| S07-S2 | 在 `read_dataset` 出口插入 `_dedup_by_latest_run` 调用（concat 后、filter 前） | S07-S1 |
| S07-S3 | 确认 `read_canonical`（:2691）是否需同步去重；若独立 concat 则插入 | S07-S2 |
| S07-S4 | fixture 测试（§7.1 全部）+ safety_counters + no_physical_migration | S07-S3 |
| S07-S5 | 与 S03 画像 after 对比（基线归因） | S07-S4 + S03 verified |
| S07-S6 | S29 T3 回归（去重后唯一） | S07-S5 + S29 实现 |

---

## 9. 风险与应对

| 风险 ID | 风险 | 应对 |
|---|---|---|
| R-S07-1 | "最新"排序依赖 run_id 命名编码时序，但 N2 命名规约在 W2（S07 是 W1） | 确定性 tiebreaker：catalog latest_manifest_run_id 优先 + source_run_id 字典序降序；N2 实现后可切换更精确时序排序（open_item，重访条件） |
| R-S07-2 | `source_run_id` 行内列与 path run_id 段不一致 | 优先用行内 source_run_id（数据自带血缘）；缺失时降级 path 段 + issue；不静默取其一 |
| R-S07-3 | `read_canonical` 独立路径漏去重导致 prices 仍重复 | 实施时确认 read_canonical 路径；CP6 自检 prices 去重后唯一（覆盖 read_canonical） |
| R-S07-4 | 去重改变保留行字段值（合成/覆盖）破坏数据真实性 | 不变量：去重只丢弃行，不修改保留行；fixture `test_dedup_no_duplicates_passthrough` 验证字段值不变 |
| R-S07-5 | 基线门被绕过（S07 在 S01/S02/S03 verified 前实现） | REQ-247 硬依赖写入 dev_gate；CP6 检查 S03 verified 状态 |
| R-S07-6 | readers.py merge_order 冲突（S07 与 S05/S06 同时改 read_dataset） | merge_order 串行 S05→S06→S07；S07 只改 read_dataset 出口 + 新增 helper，不改 S05/S06 函数体 |
| R-S07-7 | 38 分区全量 concat + 分组排序内存峰值 | 分组内排序（不全局 sort）；若 OOM 转 S13 DuckDB 只读（W2） |

---

## 10. 发布与回滚策略

- **发布**：S07 是读层去重（代码修改），无数据迁移、无 schema 变更、无 lake 写入。发布 = 代码合入。去重后 `read_dataset` 行为变化（重复键消失），下游因子/回测结果可能变化——这正是基线归因要捕捉的（D6）。
- **回滚**：移除 `read_dataset` 中的 `_dedup_by_latest_run` 调用 + 移除 helper。回滚后 `read_dataset` 恢复 glob-concat 行为（重复键重现）。无数据回滚。
- **基线归因不可回滚**：S07 去重后的 after 快照（与 S03 before 对比）固化入 S02 黄金值基线，不得覆盖。回滚代码不删除基线归因记录。
- **下游影响**：回滚后下游因子/回测会重新看到重复键污染——这是回滚的已知代价，需在回滚决策中声明。

---

## 11. 平台差异检查

| 维度 | 检查 | 结论 |
|---|---|---|
| 平台目标 | 日频离线研究 + 模拟盘场景；DuckDB 只读引擎（D4）；parquet 仍是存储 | 适用；去重用 pandas，不引入 DuckDB（DuckDB S13 W2 会自然解决全量 concat，但 S07 W1 先用 pandas 去重） |
| Python 环境 | uv 管理；`uv run --python 3.11` | 遵循 |
| 路径构造 | `_canonical_paths`/`_entry_paths`/`LakeLayout` 既有合同 | 复用，不推断路径 |
| 安装结构 | 不涉及安装器/Guardrail/平台适配 | N/A |
| DuckDB 边界 | D4 只读引擎，不替代存储；S07 不引入 .duckdb 事实源 | 遵守；S13 W2 才引入 DuckDB 依赖 |

---

## 12. 设计契约映射

| LLD 章节 | HLD 章节 | ADR | REQ | Story |
|---|---|---|---|---|
| §1 目标 G-S07-1..6 | HLD §1.3 G-1（去重唯一，重复键=0）、§12.2 读侧四道 P0 防线 | D7（四组门禁 CP3 必过） | REQ-205(C2b) | S07 |
| §2 文件影响 readers.py | HLD §10 读侧模块（`readers.py:2728 read_dataset`） | AGA-1 A1（读写分离） | REQ-249 | S07（merge_order 第 3 位） |
| §4 接口契约 | HLD §10/§18（reader 接口下沉 Feature 设计） | — | REQ-205(C2b) | S07 ← S03/S05/S06 消费 |
| §5.1 基线门时序 | HLD §12.3（基线冻结门→结构性变更） | Wave1 基线门 | REQ-247 | S01/S02/S03 → S07 |
| §5.2 去重嵌入出口 | HLD §12.2（reader→…→读层去重 source_run_id 取最新） | — | REQ-205(C2b) | S07 |
| §7 测试 T3 关联 | HLD §8 SIM-139-01（C2a 画像→C2b 去重） | — | REQ-205(C2b) | S07 → S29 T3 |
| §1.4 不授权 | HLD §1.4 + Gotchas#1/#3 | 不授权范围声明 | REQ-248 | S07 |
| §4.4 既有合同 | HLD §10 + Gotchas#2 | D8b | REQ-249 | S07 消费 read_dataset |
| §3.2 latest_by 策略 | HLD §12.2（source_run_id 取最新） | — | REQ-205(C2b) | S07 ← S08 N2（W2 run_id 命名） |

---

## 13. Gotchas

> 实质性 Gotchas，非占位（规则 9）。

1. **"嵌入 read_dataset 出口"不是"新增一个 read_dataset_dedup 函数"**：LCQ-01 resolved 明确 C2b 嵌入 `read_dataset` 出口。如果新加一个 `read_dataset_dedup` 让调用方选，下游 `read_panel_as_of`（S05）/`read_panel`（S06）可能调旧的不去重版本，重复键污染依旧。去重必须是 `read_dataset` 的默认行为，所有走 `read_dataset` 的读路径自动受益。新增独立函数是 C2a（S03）的模式，不是 C2b。

2. **"按 source_run_id 取最新"的"最新"在 N2（W2）落地前是近似排序**：当前 run_id 命名不一致（handoff §3.1 N2：前缀不一致 + unknown run_id）。S07（W1）的 latest_by 用"catalog latest_manifest_run_id 优先 + source_run_id 字典序降序"作确定性 tiebreaker，是 N2 形式化前的近似。N2（W2）实现 run_id 统一 `run-<purpose>-<window>-<source>-<YYYYMMDD>` 后，可切换更精确的时序排序。S07 必须文档化这个近似，不能假装"最新"是精确语义。重访条件：N2 verified 后重访 latest_by 排序。

3. **去重位置在 concat 后、用户 filters 前，不能反**：如果去重放在 `_filter_frame`（用户 filters）后，用户 filter 可能丢弃部分同键行，剩下的行可能恰好都是"非最新"版本，去重保留错误版本。去重必须在完整 concat 帧上做，确保排序有全部同键行的 source_run_id 上下文。这个顺序是正确性强约束，不是性能优化。

4. **`read_canonical`（prices 专用 fast reader，:2691）是隐藏的重复键漏网点**：`read_canonical` 可能独立 concat（不走 `read_dataset`），如果 S07 只改 `read_dataset` 不改 `read_canonical`，prices 经 `read_canonical` 读仍会重复。实施时必须确认 `read_canonical` 路径并同步去重。CP6 自检必须覆盖 `read_canonical` 的 prices 去重后唯一，不能只测 `read_dataset`。

5. **去重是"读层丢弃"，不是"写入去重"**：S07 只在读层丢弃重复行，不写回 lake、不删分区。物理分区的重复数据仍在磁盘上（38 分区不变）。真正的写入侧去重是 S32 C4（W2），物理分区收敛是 S08 N1（W1 后置）。S07 + S32 + N1 三者闭环：S07 读层保证、S32 写层保证、N1 物理收敛。把 S07 当成"删分区"会违反 REQ-247（物理迁移后置）和 REQ-248（不授权物理迁移）。

6. **去重后下游结果会变，这是基线归因要捕捉的，不是 bug**：S07 上线后因子/回测结果可能变化（重复键污染消除）。这正是 D6 强制基准的目的——S02 黄金值 before/after 对比归因"结构修复"。如果 S07 上线后下游结果不变，反而说明去重没生效或下游没走 `read_dataset`。CP7 验证时"下游结果变化"是预期，需归因，不是缺陷。

7. **`source_run_id` 列存在但不保证每行非空**：readers.py:2162-2171 已处理 `source_run_id` 缺失（`output["source_run_id"] = entry.latest_manifest_run_id if ... else ""`，且检测全空时记入 missing_lineage）。S07 去重必须处理空 source_run_id：空值不参与"最新"排序，降级到 path run_id 段或行序保留。不能把空 source_run_id 当成一个 run 版本参与排序，否则会错误保留空版本行。

---

## 14. open_items / tier / shared_fragments

### open_items

| ID | 描述 | 阻断 | 重访条件 |
|---|---|---|---|
| O-S07-1 | `latest_by` 排序依赖 run_id 命名编码时序，N2（W2）未实现前用字典序降序近似 | 否 | S08 N2 verified 后重访 latest_by 排序，切换精确时序 |
| O-S07-2 | `read_canonical`（:2691）是否独立 concat 需实施时确认并同步去重 | 否（实施时确认） | S07-S3 切片确认；CP6 自检覆盖 |
| O-S07-3 | 非 prices dataset 的 dedup_keys 未定义（financial_pit/market_cap 等） | 否 | 扩展到其他 dataset 时按 schema 确认 dedup_keys；本 Story 验收只针对 prices |
| O-S07-4 | 38 分区全量 concat + 分组排序在真实 lake 规模的内存表现未测 | 否 | CP7 runtime 验证（需独立授权）测真实 lake；若 OOM 转 S13 DuckDB（W2） |
| O-S07-5 | `source_run_id` 空值行的处理策略（降级 path 段 vs 行序保留）需 fixture 覆盖 | 否 | S07-S4 fixture 补空值用例 |

### tier

- `tier=P0`（读侧四道 P0 防线之一，去重唯一；Wave1 结构性变更）

### shared_fragments

- 修改 `read_dataset`（readers.py:2728，既有读路径合同）
- 新增 `_dedup_by_latest_run` helper（被 `read_dataset`/`read_canonical` 共享）
- 消费 `CatalogEntry.latest_manifest_run_id`（catalog 既有字段）
- 消费 `source_run_id` 列（数据既有行内血缘列）
- 去重语义被 S05 `read_panel_as_of`/S06 `read_panel` 间接复用（走 `read_dataset`）
- 去重语义对应 S32 C4 写入去重（写侧保证）
- after 对比依赖 S03 C2a 画像 before 快照

### lld_gate

- required: true
- required_level: full-lld
- status: ready-for-cp5-review（本 LLD 提交后等待 CP5-W1 全量人工确认）

### dev_gate

- cp5_required: true
- implementation_allowed_before_cp5: false
- baseline_gate: S07 是结构性变更，依赖 S01/S02/S03 verified（基线冻结）+ S06（panel reader 读路径契约）；基线冻结前不得实现
- readers_py_merge_order_position: 3（S05→S06→S07→…）

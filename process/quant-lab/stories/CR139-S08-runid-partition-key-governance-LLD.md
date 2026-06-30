---
title: "LLD — CR139-S08 N1 run_id 分区键治理（deferred-execution）"
story_id: "CR139-S08"
story_slug: "runid-partition-key-governance"
change_id: "CR-139"
owner_feature: "FEAT-02 写侧"
wave: "CR139-W1-STRUCTURAL-P0"
lld_policy:
  required_level: "full-lld"
  trigger_reasons: "c 范围扩展，结构性分区变更"
  rationale: "N1 是 c 类范围扩展，改变分区结构"
source_hld: "process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md v0.2"
source_adr: "process/docs/design/ARCHITECTURE-DECISION-STRATEGY-DATA-FOUNDATION.md v0.2"
feature_design_matrix: "docs/design/FEATURE-DESIGN-MATRIX.md v1.13"
req_coverage: ["REQ-201"]
depends_on: ["CR139-S01", "CR139-S02", "CR139-S03", "CR139-S07"]
dependency_type: ["contract", "runtime"]
tier: "P0"
status: "lld-ready-for-review"
dev_gate: "deferred-until-baseline-verified"
execution_phase: "design-only-cp5"
created_at: "2026-06-28T18:00:00+08:00"
author: "meta-dev"
---

# LLD — CR139-S08 N1 run_id 分区键治理（deferred-execution）

> 本 LLD 覆盖 REQ-201（N1 run_id 分区键治理）。设计来源：HLD-STRATEGY-DATA-FOUNDATION v0.2 §10 写侧模块 / §12.3 Wave1 基线门；ADR D3（不重命名存量 run_id）+ Wave1 基线门（REQ-247）；handoff v0.7 §3.1 N1 + §6 D3。本 LLD 由 meta-dev 在 CP5 阶段起草，**只写设计，不实现**：`dev_gate=deferred-until-baseline-verified`，物理分区迁移实现后置到 S01/S02/S03 verified 且 T7 黄金值归因门通过之后（REQ-247）。CP5/CP6 不授权物理分区迁移执行（REQ-248）。

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 0.1 | 2026-06-28 | meta-dev | CR-139 CP5 首版 full LLD：14 章齐全；量化验收；跨边界契约拆分（S08 own lake_layout/normalization/cli，publish/catalog/manifest 为消费方契约）；deferred-execution 标注；D3 不重命名存量；LCQ-03/04 落地。 |

---

## 1. 问题定义与目标（Goal + 问题定义）

### 1.1 问题陈述

当前 canonical 写入路径把 `run_id` 当作**长期物理分区键**：`market_data/normalization.py:806-811` 的 `_canonical_path` 构造 `canonical/<dataset>/<schema_version>/run_id=<run_id>/part-<batch>.parquet`，导致 prices 下堆积 38 个 `run_id=` 分区（handoff §1.1）。`market_data/cli.py:1871` 的 `_catalog_canonical_path` 同样按 `canonical_dataset_root(dataset) / f"run_id={run_id}"` 解析路径，固化了 run_id 作为分区键的读路径。

后果：
- `read_canonical` / `read_dataset` glob-concat 全量 38 分区，`(symbol,trade_date)` 可能重复（C2）。
- run_id 既是**运行批次标识**又是**长期分区键**，两个语义耦合，分区膨胀不可收敛。
- CR 审计链断（M4）：catalog 无 `triggered_by_cr` 字段，无法追溯某批数据由哪个 CR 触发写入。

### 1.2 目标（量化验收）

| 目标 ID | 目标 | 量化成功标准 |
|---|---|---|
| N1-G1 | 新数据不含 run_id 长期分区键 | 新数据写入路径 `dataset/schema_version/[current\|archive]/` 下 `run_id=` 路径段计数 = 0；run_id 仅存 manifest lineage 元数据与 catalog 元数据，不进物理路径 |
| N1-G2 | current/archive 区分 | 新数据写入落 `current/`；被新版本 supersede 的旧版本落 `archive/`；`current/` 下每个 `(dataset, schema_version)` 同时至多 1 个 active 版本（active count ≤ 1） |
| N1-G3 | catalog 含 triggered_by_cr | 新数据 catalog 条目 `triggered_by_cr` 字段非 null（本批 = `CR-139`）；字段缺失计数 = 0 |
| N1-G4 | 存量 run_id 不重命名（D3） | 存量 `run_id=<旧run_id>` 分区重命名计数 = 0；存量路径保持原样，由"下次重跑替换"自然收敛 |
| N1-G5 | run_id 可追溯（不丢失） | 新数据 run_id 100% 记录在 manifest lineage（LCQ-03：run-id 在 publish.py 生成，manifest.py 记录）；run_id 丢失计数 = 0 |
| N1-G6 | 双读兼容（过渡期） | reader 能同时解析旧 `run_id=` 分区与新 `current/archive` 分区；过渡期旧路径读取失败计数 = 0 |

### 1.3 非目标

- **不重命名存量 run_id 分区**（D3 / HLD §1.5）。存量 38 个 `run_id=` 分区保持原样，靠"旧数据下次重跑替换"自然收敛到新规约。
- **不执行物理分区迁移**（REQ-247/248）。本 LLD 只设计分区规约与读写路径契约；物理迁移执行后置到基线冻结之后，CP5/CP6 不授权。
- **不修改 publish.py / catalog.py / manifest.py 的实现**（文件所有权不在 S08）。S08 只声明跨边界契约，实现由 S04/S14/S22（publish.py）、S33/S39（catalog.py）、S33/S34（manifest.py）承接。
- **不改变 raw 层分区结构**（raw 层 `run_id=` 分区是运行批次原始记录，保留）。

---

## 2. Requirements（Functional / Non-Functional）

### 2.1 功能需求

| FR ID | 需求 | 来源 |
|---|---|---|
| N1-FR1 | lake_layout.py 提供新 canonical 分区路径函数，输出 `canonical/<dataset>/<schema_version>/current/` 与 `.../archive/`，不含 `run_id=` 段 | REQ-201 / handoff §3.1 N1 |
| N1-FR2 | lake_layout.py 保留旧 `canonical_dataset_root` + `run_id=` 路径函数，供存量数据与过渡期双读使用（D3） | ADR D3 |
| N1-FR3 | normalization.py 新数据写入路径调用新 current/archive 分区函数，不再写 `run_id=` 分区 | REQ-201 |
| N1-FR4 | normalization.py 保留旧 `_canonical_path`（run_id 分区）路径仅用于**存量重跑兼容**，新数据默认走新路径 | D3 / N1-FR3 |
| N1-FR5 | cli.py:1871 `_catalog_canonical_path` 支持双读：先解析新 `current/` 路径，回退旧 `run_id=` 路径 | N1-G6 |
| N1-FR6 | 声明跨边界契约：publish.py 生成 run-id 并写入 lineage，新数据 publish 写入走新分区（LCQ-03） | LCQ-03 |
| N1-FR7 | 声明跨边界契约：catalog.py 新数据条目含 `triggered_by_cr` 字段（REQ-201）；`run_lineage` 由 M4/S39 承接 | REQ-201 / handoff §3.5 M4 |
| N1-FR8 | 声明跨边界契约：manifest.py 记录 run-id 到 lineage 元数据（不进路径） | LCQ-03 |

### 2.2 非功能需求

| NFR ID | 需求 | 设计 |
|---|---|---|
| N1-NFR1 | 性能 | 新分区不含 run_id，消除 38 分区 glob-concat；current/archive 至多 2 目录，读路径 glob 次数从 38 降到 ≤2 |
| N1-NFR2 | 可维护性 | run_id 语义与分区键解耦：run_id 退化为元数据（manifest/catalog），分区键由 dataset/schema_version/version-tier(current/archive) 承载 |
| N1-NFR3 | 可审计性 | catalog `triggered_by_cr` 使每批数据可追溯到触发 CR；run_id 在 manifest lineage 保留运行批次追溯 |
| N1-NFR4 | 安全 | 不授权物理分区迁移执行 / 真实 lake 写入 / Git remote write（REQ-248）；CP5/CP6 只产出路径函数与读写契约，不触发迁移 |
| N1-NFR5 | 兼容性 | 旧 `run_id=` 分区与读路径保留，过渡期双读，不破坏既有 `read_canonical` / `read_dataset` 合同 |

---

## 3. 模块拆分与职责

| 模块 | 子域 | 职责（S08 范围内） | Owner | 关键文件 | S08 写入权 |
|---|---|---|---|---|---|
| 路径契约 | lake_layout 分区路径 | 新增 current/archive canonical 分区路径函数；保留旧 run_id 路径函数 | FEAT-02 写侧 | `market_data/lake_layout.py` | **是（merge_order 首位）** |
| 写入路径 | normalization canonical 写 | 新数据写入走新 current/archive 路径；旧 `_canonical_path` 保留供存量重跑 | FEAT-02 写侧 | `market_data/normalization.py:806-811` | **是（merge_order 首位）** |
| 读路径解析 | cli canonical 路径解析 | `_catalog_canonical_path` 双读：新 current 优先，回退旧 run_id | FEAT-02 写侧 | `market_data/cli.py:1871` | **是（merge_order 首位）** |
| run-id 生成 | publish run-id 生成 | 生成 run-id 并写入 lineage；新数据 publish 走新分区 | FEAT-02 写侧 | `market_data/publish.py` | **否（cross-boundary contract，S22 T6 own）** |
| catalog 元数据 | catalog triggered_by_cr | 新数据 catalog 条目含 triggered_by_cr | FEAT-02 写侧 | `market_data/catalog.py` | **否（cross-boundary contract，S39 M4 own）** |
| manifest lineage | manifest run-id 记录 | run-id 记入 lineage 元数据 | FEAT-02 写侧 | `market_data/manifest.py` | **否（cross-boundary contract，S33/S34 own）** |

### 3.1 跨边界所有权说明（关键）

S08 的**写入所有权**严格限于 `lake_layout.py` / `normalization.py:806-811` / `cli.py:1871`（三者 merge_order S08 均首位）。`publish.py` / `catalog.py` / `manifest.py` 不在 S08 写入范围：

- `publish.py` merge_order：S04 V1 → S14 L2 → S22 T6（S08 不在内）。S08 声明"新数据 publish 写入走新分区 + run-id 在 publish 生成"契约，由 S22 T6 实现 run-id 生成、S04 V1/S14 L2 实现 publish 写入路径。
- `catalog.py` merge_order：S04 V1 → S33 M1 → S39 M4（S08 不在内）。S08 声明"新数据 catalog 含 `triggered_by_cr`"契约（REQ-201），由 S39 M4 实现 catalog 字段；`run_lineage` 字段亦属 M4（handoff §3.5）。
- `manifest.py` merge_order：S33 M1 → S34 M2（S08 不在内）。S08 声明"run-id 记入 manifest lineage 元数据"契约（LCQ-03），由 S33/S34 实现。

S08 对这三类文件**只声明契约、不写代码**。契约以本 LLD §4 接口契约为准，消费方 Story（S04/S14/S22/S33/S34/S39）实现时必须回引本 LLD。

---

## 4. 代码结构与文件影响范围

### 4.1 lake_layout.py（S08 own，merge_order 首位）

**新增**（不删除既有函数）：

```python
# 概念签名（实现延迟到 dev_gate 通过后）
def canonical_current_root(self, dataset: str, schema_version: str = SCHEMA_VERSION) -> Path:
    """新数据 current 分区根路径：canonical/<dataset>/<schema_version>/current/"""

def canonical_archive_root(self, dataset: str, schema_version: str = SCHEMA_VERSION) -> Path:
    """被 supersede 的旧版本归档：canonical/<dataset>/<schema_version>/archive/"""

def canonical_current_partition_path(
    self, dataset: str, schema_version: str = SCHEMA_VERSION, *,
    partition_date=None, trade_date=None, exchange=None, board=None,
) -> Path:
    """新数据写入分区路径，不含 run_id 段。"""

def canonical_archive_partition_path(
    self, dataset: str, schema_version: str = SCHEMA_VERSION, *,
    partition_date=None, trade_date=None, exchange=None, board=None,
) -> Path:
    """归档分区路径，不含 run_id 段。"""
```

**保留**（D3，不修改）：
- `canonical_dataset_root(dataset, schema_version)` → `canonical/<dataset>/<schema_version>`（line 131-136）
- 既有 `candidate_*` / `published_*` / `catalog_current_pointer_path` 函数不变

**路径规约**：

| 层 | 旧规约（保留） | 新规约（S08 新增） |
|---|---|---|
| canonical | `canonical/<dataset>/<schema_version>/run_id=<run_id>/part-<batch>.parquet` | `canonical/<dataset>/<schema_version>/current/<可选 trade_date=/exchange=/board=>/part-<batch>.parquet` |
| canonical archive | （无） | `canonical/<dataset>/<schema_version>/archive/<可选 trade_date=/exchange=/board=>/part-<batch>.parquet` |
| run_id 归属 | 物理路径段 | manifest lineage 元数据 + catalog 元数据（不进路径） |

### 4.2 normalization.py:806-811（S08 own，merge_order 首位）

**现状**（`_canonical_path`，line 806-811）：
```python
def _canonical_path(layout: LakeLayout, dataset: str, run_id: str, batch_id: str) -> Path:
    return (
        layout.canonical_dataset_root(dataset, SCHEMA_VERSION)
        / f"run_id={run_id}"
        / f"part-{batch_id}.parquet"
    )
```

**变更**（实现延迟）：
- 新增 `_canonical_current_path(layout, dataset, batch_id, ...)` 调用 `layout.canonical_current_partition_path(...)`，新数据写入走此路径。
- 保留 `_canonical_path`（run_id 版）仅用于**存量重跑兼容**（D3：旧数据下次重跑时若仍按旧规约产出，走旧路径；新规约重跑走新路径）。
- 写入入口加 `partition_layout` 参数（`"legacy_run_id"` / `"current_archive"`），默认新数据走 `"current_archive"`。

### 4.3 cli.py:1871（S08 own，merge_order 首位）

**现状**（`_catalog_canonical_path`，line 1861-1874）：
```python
def _catalog_canonical_path(layout, dataset, canonical_paths, run_id):
    ...
    if run_id:
        run_root = layout.canonical_dataset_root(dataset) / f"run_id={run_id}"
        if run_root.exists():
            return run_root
    return None
```

**变更**（实现延迟）：
- 双读：先解析 `layout.canonical_current_root(dataset)`，命中则返回；未命中回退旧 `run_id=` 路径。
- 过渡期（存量未全部重跑前）两条路径并存；存量全部重跑收敛后可废弃 run_id 回退（标记为后续 cleanup，不在 S08 范围）。

### 4.4 跨边界文件（S08 不写，仅声明契约）

| 文件 | S08 契约声明 | 实现 Story | merge_order |
|---|---|---|---|
| `publish.py` | 新数据 publish 写入调用 `canonical_current_partition_path`；run-id 在 publish 时生成并写入 lineage | S04 V1 / S14 L2 / S22 T6 | S04→S14→S22 |
| `catalog.py` | 新数据 catalog 条目含 `triggered_by_cr: "CR-139"` 字段（REQ-201）；`run_lineage` 字段由 M4 承接 | S39 M4（triggered_by_cr/run_lineage）；S04 V1（current pointer）；S33 M1（定主） | S04→S33→S39 |
| `manifest.py` | run-id 记入 lineage 元数据（`run_id` 字段已存在 line 15/37/58/190，新数据 publish 时由 publish.py 传入） | S33 M1 / S34 M2 | S33→S34 |

---

## 5. 数据模型与持久化设计

### 5.1 分区路径 schema 变更

```
canonical/
├── <dataset>/
│   ├── <schema_version>/
│   │   ├── run_id=<旧run_id>/        # 存量（D3 保留，不重命名）
│   │   │   └── part-<batch>.parquet
│   │   ├── current/                   # 新数据（S08 新增）
│   │   │   └── [trade_date=.../exchange=.../board=.../] part-<batch>.parquet
│   │   └── archive/                   # 被 supersede 旧版本（S08 新增）
│   │       └── [trade_date=.../] part-<batch>.parquet
```

### 5.2 catalog 元数据 schema 变更（跨边界契约，S39 实现）

| 字段 | 类型 | 含义 | 实现方 |
|---|---|---|---|
| `triggered_by_cr` | `str \| None` | 触发本批数据写入的 CR ID（本批 = `"CR-139"`） | S39 M4（catalog.py 写入）；S08 声明契约 |
| `run_lineage` | `list[str] \| None` | 本批数据 run-id 链路（M4 范围，S08 引用不实现） | S39 M4 |
| `latest_manifest_run_id` | `str`（既有） | 既有字段，run-id 仍记录 | 既有（catalog.py:47/105） |

### 5.3 manifest lineage 元数据（跨边界契约，S33/S34 实现）

manifest 既有 `run_id` 字段（manifest.py:15/37/58/190）。S08 契约：新数据 publish 时 run-id 由 publish.py 生成并传入 manifest 记录，run-id **不进物理路径**，仅存 manifest lineage + catalog 元数据。

### 5.4 持久化设计要点

- parquet 仍是存储（D4，不变）。
- 新分区物理结构由 lake_layout.py 路径函数定义，normalization.py 写入时调用。
- current/archive 区分由 publish 逻辑（S04/S14）决定：新版本写入 current 前，既有 current 数据移入 archive（supersede 语义）。**S08 只定义路径函数，supersede 移动逻辑由 publish.py 消费方实现**。
- 无新数据库 / 无 DuckDB 持久事实源（D4）。

---

## 6. API / Interface 设计

### 6.1 lake_layout.py 新增路径函数接口

```python
class LakeLayout:
    def canonical_current_root(
        self, dataset: str, schema_version: str = SCHEMA_VERSION
    ) -> Path:
        """返回新数据 current 分区根路径，不创建目录。
        路径: canonical/<dataset>/<schema_version>/current/
        不含 run_id 段。"""

    def canonical_archive_root(
        self, dataset: str, schema_version: str = SCHEMA_VERSION
    ) -> Path:
        """返回归档分区根路径，不创建目录。
        路径: canonical/<dataset>/<schema_version>/archive/
        不含 run_id 段。"""

    def canonical_current_partition_path(
        self, dataset: str, schema_version: str = SCHEMA_VERSION, *,
        partition_date: object | None = None,
        trade_date: object | None = None,
        exchange: str | None = None,
        board: str | None = None,
    ) -> Path:
        """返回新数据写入分区路径，Hive-style，不含 run_id 段。"""

    def canonical_archive_partition_path(
        self, dataset: str, schema_version: str = SCHEMA_VERSION, *,
        partition_date: object | None = None,
        trade_date: object | None = None,
        exchange: str | None = None,
        board: str | None = None,
    ) -> Path:
        """返回归档分区路径，Hive-style，不含 run_id 段。"""
```

**前置校验**：复用既有 `_clean_partition_value` / `_compact_date`（line 28-36 / 19-25），空值 / glob 字符 / 目录分隔符 → `MarketDataPathError`。

**失败行为**：路径函数只解析不写入；`_clean_partition_value` 校验失败 raise `MarketDataPathError`，不静默截断。

### 6.2 normalization.py 写入路径接口（实现延迟）

```python
def _canonical_current_path(
    layout: LakeLayout, dataset: str, batch_id: str, *,
    trade_date=None, exchange=None, board=None,
) -> Path:
    """新数据写入路径：调用 layout.canonical_current_partition_path。"""

# 既有 _canonical_path(layout, dataset, run_id, batch_id) 保留（D3 存量重跑兼容）
```

写入入口（`_write_parquet_atomic`，line 814-820）不变；只改路径构造来源。

### 6.3 cli.py:1871 双读接口（实现延迟）

```python
def _catalog_canonical_path(layout, dataset, canonical_paths, run_id):
    # 1. 新路径优先
    current_root = layout.canonical_current_root(dataset)
    if current_root.exists():
        # 匹配 canonical_paths 中属于 current_root 的路径
        ...
    # 2. 回退旧 run_id 路径（D3 存量）
    if run_id:
        run_root = layout.canonical_dataset_root(dataset) / f"run_id={run_id}"
        if run_root.exists():
            return run_root
    return None
```

### 6.4 跨边界契约接口（S08 声明，消费方实现）

| 契约 ID | 契约 | 消费方 | 验证方式 |
|---|---|---|---|
| N1-C1 | publish.py 写新数据时调用 `layout.canonical_current_partition_path`，不写 `run_id=` 分区 | S04/S14/S22 | 契约测试：新数据 publish 产出路径 `run_id=` 段计数 = 0 |
| N1-C2 | publish.py 生成 run-id 并传入 manifest lineage | S22 T6（LCQ-03） | 契约测试：manifest 记录含 run_id 字段 |
| N1-C3 | catalog.py 新数据条目含 `triggered_by_cr` 字段 | S39 M4 | 契约测试：新数据 catalog 条目 triggered_by_cr 非空 |
| N1-C4 | manifest.py run-id 记入 lineage 元数据（不进路径） | S33/S34 | 契约测试：新数据 manifest run_id 字段非空，路径无 run_id 段 |

---

## 7. 核心处理流程

### 7.1 新数据写入流程（设计，实现延迟）

```
normalize（PIT 盖戳 + 去重 C4）
  → _canonical_current_path(layout, dataset, batch_id, ...)  # 新路径，无 run_id 段
  → _write_parquet_atomic(frame, path)                        # 既有原子写
  → publish gate（S04/S14）
    → 生成 run-id（LCQ-03，publish.py）
    → 旧 current 数据移入 archive（supersede，publish.py 消费方实现）
    → 新数据写入 current
    → manifest 记录 run-id lineage（S33/S34）
    → catalog 条目写 triggered_by_cr="CR-139"（S39 M4）
```

### 7.2 存量数据流程（D3，不变）

```
存量 canonical/<dataset>/<schema_version>/run_id=<旧run_id>/part-*.parquet
  → 不重命名（D3）
  → 下次重跑时若走新规约 → 写入 current/，旧 run_id= 分区自然弃用
  → 过渡期 reader 双读（cli.py:1871 current 优先，回退 run_id=）
```

### 7.3 读路径流程（cli.py:1871 双读）

```
_catalog_canonical_path(layout, dataset, canonical_paths, run_id):
  1. current_root = layout.canonical_current_root(dataset)
     if current_root.exists() and matches canonical_paths → return current path
  2. if run_id:
       run_root = canonical_dataset_root(dataset) / f"run_id={run_id}"
       if run_root.exists() → return run_root  # D3 回退
  3. return None
```

### 7.4 dev_gate 门控流程（deferred-until-baseline-verified）

```
CP5 LLD 设计（本文件）→ CP5 人工确认
  → [不实现，等待]
  → S01 T8 清册 verified
  → S02 T7 黄金值基线 verified
  → S03 C2a 重复画像 verified
  → T7 黄金值归因门通过（REQ-247 基线门）
  → S07 C2b 读层去重 verified（contract 依赖）
  → dev_gate 解锁 → S08 实现
  → CP6 → CP7
```

**关键**：在 S01/S02/S03 verified 且 T7 黄金值归因门通过前，S08 不得开始实现（REQ-247）。

---

## 8. 技术设计细节

### 8.1 current/archive supersede 语义

- `current/`：当前 active 版本，每个 `(dataset, schema_version)` 至多 1 个 active 版本。
- `archive/`：被 supersede 的历史版本，保留可追溯。
- supersede 移动逻辑（current → archive）由 publish.py 消费方实现（S04/S14），**S08 只提供 archive 路径函数**。
- 移动是文件级（parquet part 文件），不是 run_id 级（新规约无 run_id 分区）。

### 8.2 run_id 退化语义

| 维度 | 旧 | 新 |
|---|---|---|
| 物理路径 | `run_id=<run_id>` 段 | 无 run_id 段 |
| manifest | run_id 字段（既有） | run_id 字段（保留，publish 生成传入） |
| catalog | `latest_manifest_run_id`（既有） | `latest_manifest_run_id` + `triggered_by_cr` + `run_lineage`（M4） |
| 语义 | 运行批次标识 + 长期分区键（耦合） | 运行批次标识（元数据，解耦） |

### 8.3 双读过渡期

- 过渡期：存量 `run_id=` 分区与新 `current/archive` 分区并存。
- 双读顺序：current 优先 → 回退 run_id=（cli.py:1871）。
- 收敛条件：存量全部重跑替换后，run_id= 回退可废弃（标记后续 cleanup，不在 S08 范围）。
- 收敛判定（不在 S08 实现）：T8 清册（S01）可统计 `run_id=` 分区数是否归零。

### 8.4 partition_layout 参数

normalization.py 写入入口加 `partition_layout: Literal["legacy_run_id", "current_archive"] = "current_archive"`：
- 新数据默认 `"current_archive"` → 走新路径。
- 存量重跑兼容 `"legacy_run_id"` → 走旧路径（D3，仅当显式指定）。
- 默认值 `"current_archive"` 保证新数据不退回旧规约。

---

## 9. 安全与性能设计

### 9.1 安全

- **不授权物理分区迁移执行**（REQ-248）：S08 LLD 只设计路径函数与契约，CP5/CP6 不触发物理迁移。
- **不写真实 lake**：路径函数只解析路径，`_write_parquet_atomic` 调用由实现阶段（dev_gate 通过后）触发，CP5 不执行。
- **路径注入防护**：复用 `_clean_partition_value`（line 28-36）校验空值 / glob 字符 / 目录分隔符，失败 raise `MarketDataPathError`。
- **run_id 不进路径**：消除 run_id 作为路径段的注入面（run_id 含特殊字符不再影响物理路径）。

### 9.2 性能

- 新分区消除 38 分区 glob-concat：current/archive 至多 2 目录，读路径 glob 次数从 38 降到 ≤2。
- reader 双读过渡期有 1 次额外 `exists()` 检查（current_root），开销可忽略。
- current/archive 分区下可选 `trade_date=` / `exchange=` / `board=` 子分区，支持谓词下推（R4 受益）。

---

## 10. 测试设计

### 10.1 测试策略（static-only，CP6/CP7 默认）

`validation_mode=static-only`（CP6/CP7 默认，runtime 需独立授权）。验证命令：`uv run --python 3.11 pytest -q tests/test_cr139_s08_partition_governance.py`。

### 10.2 测试用例

| TC ID | 场景 | 断言 | 类型 |
|---|---|---|---|
| N1-TC1 | `canonical_current_root(dataset)` 路径构造 | 路径 = `canonical/<dataset>/<schema_version>/current/`，不含 `run_id=` 段 | unit |
| N1-TC2 | `canonical_archive_root(dataset)` 路径构造 | 路径 = `canonical/<dataset>/<schema_version>/archive/`，不含 `run_id=` 段 | unit |
| N1-TC3 | `canonical_current_partition_path` 含 trade_date/exchange | Hive-style 路径正确，无 run_id 段 | unit |
| N1-TC4 | `_clean_partition_value` 校验空值/glob/分隔符 | raise `MarketDataPathError` | unit |
| N1-TC5 | `_canonical_current_path`（normalization）调用新路径 | 产出路径无 `run_id=` 段 | unit |
| N1-TC6 | `_canonical_path`（legacy）保留 | 仍产出 `run_id=<run_id>` 路径（D3 存量兼容） | unit |
| N1-TC7 | `_catalog_canonical_path` 双读：current 命中 | 返回 current 路径，不查 run_id= | unit |
| N1-TC8 | `_catalog_canonical_path` 双读：current 未命中，run_id 回退 | 返回旧 run_id= 路径（D3 过渡期） | unit |
| N1-TC9 | 新数据写入路径 `run_id=` 段计数 | = 0（N1-G1） | contract |
| N1-TC10 | 存量 `run_id=` 分区重命名计数 | = 0（N1-G4，D3） | contract |
| N1-TC11 | catalog 新数据条目 `triggered_by_cr` 非空（跨边界契约，mock catalog） | triggered_by_cr = "CR-139"（N1-G3） | contract |
| N1-TC12 | manifest 新数据 run_id 字段非空（跨边界契约，mock manifest） | run_id 字段存在（N1-G5，LCQ-03） | contract |

### 10.3 fixture

- 临时 LakeLayout（tmp_path）构造 current/archive 路径，不写真实 lake。
- mock catalog / manifest 跨边界契约（S08 不依赖 S39/S33 实现即可验证契约声明）。
- 既有 `run_id=` 路径 fixture（D3 回退验证）。

### 10.4 未覆盖

- 物理分区迁移执行（后置到基线冻结之后，CP5/CP6 不覆盖）。
- publish.py run-id 生成实现（S22 T6 范围）。
- catalog.py triggered_by_cr 字段实现（S39 M4 范围）。
- 真实 lake 写入（runtime 需独立授权）。

---

## 11. 实施步骤（最小切片，实现延迟标注）

> `dev_gate=deferred-until-baseline-verified`。以下步骤为 dev_gate 解锁后的最小实现切片，CP5 阶段不执行。

### Slice N1-1：lake_layout.py 新增路径函数（S08 own）

1. 新增 `canonical_current_root` / `canonical_archive_root` / `canonical_current_partition_path` / `canonical_archive_partition_path`。
2. 复用 `_clean_partition_value` / `_compact_date` 校验。
3. 单测 N1-TC1..TC4。
4. 验证：`uv run --python 3.11 pytest -q tests/test_cr139_s08_partition_governance.py::test_current_root`

### Slice N1-2：normalization.py 写入路径切换（S08 own）

1. 新增 `_canonical_current_path`，调用 `layout.canonical_current_partition_path`。
2. 写入入口加 `partition_layout` 参数，默认 `"current_archive"`。
3. 保留 `_canonical_path`（legacy）。
4. 单测 N1-TC5..TC6。
5. 验证：`uv run --python 3.11 pytest -q tests/test_cr139_s08_partition_governance.py::test_write_path`

### Slice N1-3：cli.py:1871 双读（S08 own）

1. `_catalog_canonical_path` 加 current 优先解析，回退 run_id=。
2. 单测 N1-TC7..TC8。
3. 验证：`uv run --python 3.11 pytest -q tests/test_cr139_s08_partition_governance.py::test_dual_read`

### Slice N1-4：跨边界契约测试（S08 声明，mock 消费方）

1. 契约测试 N1-TC9..TC12（mock publish/catalog/manifest，验证契约声明可被消费方满足）。
2. 验证：`uv run --python 3.11 pytest -q tests/test_cr139_s08_partition_governance.py::test_contracts`

**实现延迟标注**：Slice N1-1..N1-4 均标注 `dev_gate=deferred-until-baseline-verified`，在 S01/S02/S03 verified + T7 黄金值归因门通过后启动。

---

## 12. 风险、难点与预研建议

### 12.1 风险

| 风险 ID | 风险 | 应对 | 来源 |
|---|---|---|---|
| N1-R1 | **物理迁移与历史数据变化归因混淆**（核心 gotcha） | 物理分区迁移后置到 T7 黄金值归因门通过之后（REQ-247）；迁移前后必须能对 T7 黄金值做归因对比，区分"结构修复"vs"历史数据变化" | HLD §12.3 / Gotcha#3 |
| N1-R2 | 双读过渡期 current/run_id= 路径冲突 | 双读顺序 current 优先；过渡期 T8 清册（S01）统计两种路径分区数 | N1-G6 |
| N1-R3 | 跨边界契约消费方未实现导致 N1 验收无法闭环 | S08 LLD §4 声明契约 + §6.4 契约表；CP5 确认 S04/S14/S22/S33/S34/S39 消费方回引本 LLD；契约测试 mock 消费方 | §3.1 |
| N1-R4 | supersede 移动逻辑（current→archive）实现方不明确 | S08 只提供 archive 路径函数；supersede 移动由 publish.py 消费方（S04/S14）实现；本 LLD 显式标注 | §8.1 |
| N1-R5 | 存量重跑未收敛导致 run_id= 分区长期残留 | D3 接受残留（不重命名）；T8 清册（S01）跟踪收敛进度；收敛判定后续 cleanup（不在 S08） | D3 |

### 12.2 难点

- current/archive supersede 语义与 publish gate（V1/L2）的交互：publish 写新 current 前如何原子地把旧 current 移入 archive。**S08 不实现**，由 S04/S14 消费方设计；S08 只提供路径函数。
- 双读过渡期的路径匹配：`_catalog_canonical_path` 如何判断 canonical_paths 属于 current_root vs run_id_root。实现期细化。

### 12.3 预研建议

- 在 dev_gate 解锁前，用 fixture 模拟 current/archive 双读场景，验证路径函数正确性（可在 CP5 后、dev_gate 解锁前做静态 fixture 验证）。

---

## 13. 回滚与发布策略

### 13.1 回滚策略

| 场景 | 回滚方式 | 难度 |
|---|---|---|
| 新路径函数引入 bug | 回退 lake_layout.py 新增函数（保留既有函数不变，D3 存量不受影响） | 低 |
| normalization 写入走错路径 | `partition_layout` 参数切回 `"legacy_run_id"`（显式） | 低 |
| cli 双读解析错误 | 回退到只读 `run_id=` 路径（删 current 优先分支） | 低 |
| **物理分区迁移执行后发现问题** | **回滚困难**（物理文件已移动） | **高** |

### 13.2 物理迁移回滚困难 → 基线门保护

物理分区迁移（存量 run_id= 分区重写为 current/archive）**回滚困难**：一旦物理文件移动/重写，难以恢复原状。因此：

- **REQ-247 基线门**：物理迁移后置到 T7 黄金值归因门通过之后。
- **T7 黄金值归因**：迁移前后必须对 T7 黄金值做对比，差异 100% 归因到"结构修复"或"历史数据变化"二者之一（HLD G-6）。归因失败则不迁移。
- **CP5/CP6 不授权物理迁移执行**（REQ-248）：本 LLD 只设计路径规约，物理迁移由独立 runtime_authorization 授权（action scope / 运行窗口 / 脱敏 / 回滚 / 审计范围）。

### 13.3 发布策略

- S08 实现完成后 CP6 编码完成门。
- CP7 验证（static-only 默认）：分区路径函数 + 双读 + 契约测试。
- 物理分区迁移执行**不在 S08 CP7 范围**：需独立 runtime_authorization，后置到基线冻结之后。
- 发布判定（CP8）：S08 路径规约 + 读写契约就绪可标 PASS_WITH_RISK（物理迁移未执行是已知风险，进入 CP8 risk acceptance）。

---

## 14. 平台差异检查

| 维度 | 当前平台 | S08 影响 | 处理 |
|---|---|---|---|
| 数据湖存储 | parquet（`data/lake/` 软链外部存储，已 gitignore） | 无变化，parquet 仍是存储（D4） | N/A |
| 路径构造 | `lake_layout.py` 单一咽喉（handoff §1.3） | 新增路径函数同在 lake_layout.py，结构改动触点可控 | 复用既有 `_clean_partition_value` |
| 操作系统 | Linux（WSL2） | Path 用 `pathlib.Path`，跨平台 | N/A |
| Python | uv 管理（`uv run --python 3.11`） | 无新依赖 | N/A |
| DuckDB | 只读引擎（D4，S13 范围） | S08 不引入 DuckDB | N/A |
| NAS / QMT | 不授权（REQ-248） | S08 不触 NAS/QMT | N/A |

**平台差异结论**：S08 无平台差异风险，路径函数与双读逻辑跨平台兼容。

---

## 15. 设计契约映射

| 契约来源 | 契约内容 | S08 落地章节 |
|---|---|---|
| HLD §10 写侧模块 | 写侧 normalize/publish/lineage，`lake_layout.py` 单一咽喉 | §3 / §4.1 |
| HLD §12.3 Wave1 基线门 | T8→T7→C2a 基线冻结 → 结构性变更 → N1 物理迁移后置 | §7.4 / §12.1 / §13.2 |
| HLD §1.5 非目标 | 不重命名存量 run_id（D3） | §1.3 / §4.1 / §7.2 |
| ADR D3 | 不重命名，新数据规范化 + 旧数据下次重跑替换 | §1.3 / §7.2 / §12.1 N1-R5 |
| ADR Wave1 基线门 | 结构性变更顺序，N1 物理迁移后置 | §7.4 / §13.2 |
| REQ-201 | 新数据 `dataset/schema_version/` + current + archive；catalog 加 `triggered_by_cr`；存量 run_id 按 D3 不重命名 | §1.2 / §2.1 / §4 / §5 / §10 |
| REQ-247 | Wave1 结构性变更基线门，物理迁移后置到基线冻结之后 | §7.4 / §12.1 / §13.2 |
| REQ-248 | 不授权物理分区迁移执行 / runtime / 真实 lake 写入 / Git remote write | §1.3 / §9.1 / §13.2 |
| REQ-249 | 既有合同闭环非新建（d2）；N1 是 c 类范围扩展，非 d2 | §1.1（N1 = c 类） |
| LCQ-03 | run-id 在 publish.py 生成，manifest.py 记录，与 V1/L2 共享 publish.py，merge_order V1→L2→T6 | §3.1 / §4.4 / §6.4 N1-C1/C2 |
| LCQ-04 | N1 LLD 在 CP5 写（设计不阻塞）；dev_gate=deferred-until-baseline-verified | §7.4 / §11 |
| handoff §3.1 N1 | 新数据 `dataset/schema_version/` + current + archive；catalog 加 `triggered_by_cr`；代码触点 lake_layout.py/cli.py:1871/normalization.py:808 | §4 / §6 |
| handoff §6 D3 | 存量 run_id 不重命名，新数据规范化 + 旧数据下次重跑替换 | §1.3 / §7.2 |
| handoff §3.5 M4 | catalog 加 `triggered_by_cr`、`run_lineage`（M4/S39 范围，S08 引用） | §5.2 / §6.4 N1-C3 |
| HLD Gotcha#3 | Wave1 顺序不可调（REQ-247 是 CP2 用户追加约束），N1 物理迁移明确后置 | §12.1 N1-R1 |

---

## 16. 实现灰区与取舍记录

### 16.1 LCQ-04（已 resolved）

| 字段 | 内容 |
|---|---|
| id | LCQ-139-04 |
| story_id | CR139-S08 |
| question | N1 deferred-execution 边界：N1 LLD 在 CP5 写还是等基线冻结后？dev_gate 是否标 deferred-until-baseline-verified？ |
| options | (A) N1 LLD 在 CP5 写，dev_gate=deferred-until-baseline-verified（推荐）；(B) 等基线冻结后写 LLD |
| recommendation | (A) N1 LLD 在 CP5 写（设计不阻塞）；dev_gate=deferred-until-baseline-verified（实现延迟到 S01/S02/S03 verified 后）；CP4/CP5 不授权物理分区迁移执行 |
| pros_cons | (A) 设计与实现解耦，CP5 可统一确认全批 LLD，实现按基线门门控；(B) 延迟 LLD 会导致 CP5 批次不完整，阻塞全量确认 |
| impact_surface | N1 LLD/实现时序；基线门交互 |
| blocks_lld | false |
| answer | 采纳推荐：N1 LLD 在 CP5 写（设计不阻塞）；dev_gate=deferred-until-baseline-verified（实现延迟到 S01/S02/S03 verified 后）；CP4/CP5 不授权物理分区迁移执行，后置到基线冻结之后（REQ-247） |
| status | resolved |
| 重访条件 | 若基线门定义变化（REQ-247 修订），重新评估 dev_gate 解锁条件 |

### 16.2 跨边界文件所有权取舍

| 取舍 | 决策 | 理由 |
|---|---|---|
| catalog.py `triggered_by_cr` 由 S08 还是 S39 实现 | S39 M4 实现，S08 声明契约 | catalog.py merge_order S04→S33→S39，S08 不在内；S08 写 catalog.py 会违反文件所有权门控 |
| publish.py run-id 生成由 S08 还是 S22 实现 | S22 T6 实现，S08 声明契约 | publish.py merge_order S04→S14→S22，S08 不在内；LCQ-03 明确 run-id 在 publish.py 生成 |
| supersede 移动（current→archive）由 S08 还是 S04/S14 实现 | S04/S14 实现，S08 提供路径函数 | publish.py 写入与 pointer 前移属 S04/S14；S08 只提供 archive 路径函数 |

### 16.3 current/archive vs 单 current 取舍

| 选项 | 优势 | 代价 |
|---|---|---|
| current + archive（推荐，REQ-201） | 保留历史版本可追溯；supersede 语义清晰 | 多一层目录，读写逻辑略复杂 |
| 单 current（覆盖写） | 简单 | 历史版本丢失，无法回溯 supersede 前数据 |

采纳 current + archive（REQ-201 明确要求 current + archive）。

---

## 17. Gotchas（规则 9，实质性）

> 实质性 Gotchas，非占位。

1. **物理迁移与历史数据变化归因混淆是 N1 核心 gotcha**：N1 改的是分区**结构**（run_id= → current/archive），不是数据**内容**。但物理迁移执行时会同时改变"读哪些分区"和"读到什么数据"，导致整改后下游因子/回测黄金值差异无法归因——是"结构修复让重复消失了"还是"历史数据本身变了"？这正是 REQ-247 基线门的核心目的：T7 黄金值必须在迁移前冻结，迁移后差异 100% 归因到"结构修复"或"历史数据变化"二者之一。**若 CP5 把 N1 实现排在 T7 黄金值之前，归因失败，整改不可审计**（HLD Gotcha#3）。本 LLD 因此 `dev_gate=deferred-until-baseline-verified`。

2. **D3"不重命名存量"不等于"不动存量"**：D3 是不主动重命名存量 `run_id=` 分区，但存量数据会在"下次重跑"时按新规约写入 current/，旧 run_id= 分区自然弃用。常见误区是把 D3 理解为"存量永远不变"，导致 N1 验收"存量 run_id= 分区数"无法收敛。正确理解：D3 是"不一次性批量重命名"，收敛靠"重跑替换"渐进发生。

3. **run_id 退化为元数据后，readers.py glob 逻辑必须同步**（跨 Story 依赖）：N1 把 run_id 从路径移到元数据后，`readers.py:375 _canonical_paths` / `:409` 的 glob 全量分区逻辑（C2 重复源）需要 C2b 读层去重（S07）配合。若 S07 未实现去重而 S08 已切新路径，current/ 下仍可能因重跑残留多版本导致重复。依赖顺序：S07 C2b verified 是 S08 的 contract 依赖（dev_plan 已确认）。

4. **跨边界契约消费方漏实现是隐性断链**：S08 声明的 N1-C1..C4 契约（publish/catalog/manifest）若消费方 Story（S04/S14/S22/S33/S34/S39）未回引本 LLD 实现，会出现"路径函数就绪但 publish 仍写 run_id=" 或"catalog 无 triggered_by_cr"的隐性断链。CP5 必须确认消费方 Story LLD/技术说明回引本 LLD §6.4 契约表。

5. **current/archive supersede 移动不是 S08 的活**：常见误区是让 S08 实现 current→archive 的文件移动逻辑。S08 只提供 archive 路径函数；移动逻辑由 publish.py 消费方（S04/S14）在 publish gate 时实现（旧 current 移入 archive → 新数据写入 current）。若 S08 越界实现移动，会与 S04/S14 publish 逻辑冲突（文件所有权）。

6. **partition_layout 默认值必须是 "current_archive"**：normalization.py 写入入口的 `partition_layout` 参数默认值若设为 `"legacy_run_id"`，新数据会退回旧规约，N1 验收失败。默认值 `"current_archive"` 保证新数据走新路径，仅存量重跑兼容显式指定 `"legacy_run_id"`。

---

## 18. open_items / tier / shared_fragments

### 18.1 open_items

| O-ID | 描述 | 状态 | 阻断 |
|---|---|---|---|
| N1-O1 | supersede 移动逻辑（current→archive）实现细节由 S04/S14 在其 LLD/技术说明中细化 | OPEN（跨边界，消费方 LLD 承接） | 否（S08 提供路径函数即可） |
| N1-O2 | 双读过渡期 current/run_id= 路径匹配判定（canonical_paths 属于哪个 root）实现期细化 | OPEN（实现期） | 否 |
| N1-O3 | 物理分区迁移执行的 runtime_authorization（action scope/窗口/脱敏/回滚/审计）由 host-orchestrator 单独授权 | OPEN（后置到基线冻结之后） | 否（CP5/CP6 不授权） |
| N1-O4 | 收敛判定（run_id= 分区归零）与废弃 run_id= 回退的 cleanup Story | OPEN（后续 cleanup，不在 S08） | 否 |

### 18.2 tier

- **tier: P0**（Wave CR139-W1-STRUCTURAL-P0；REQ-201 P0；c 类范围扩展结构性分区变更）。

### 18.3 shared_fragments

- `deferred-execution`：dev_gate=deferred-until-baseline-verified，实现延迟到 S01/S02/S03 verified + T7 黄金值归因门通过之后（REQ-247）。
- `cross-boundary-contract-declarer`：S08 声明 publish/catalog/manifest 跨边界契约（N1-C1..C4），消费方 Story 实现。
- `dual-read-transition`：cli.py:1871 双读过渡期，current 优先回退 run_id=（D3）。
- `no-physical-migration-cp5-cp6`：CP5/CP6 不授权物理分区迁移执行（REQ-248），只设计路径规约与读写契约。

### 18.4 Definition of Done（CP5 LLD 阶段）

- [x] 14 章齐全（本文件 §1-18）
- [x] 量化验收条件（§1.2 N1-G1..G6）
- [x] 文件影响范围与跨边界所有权明确（§3 / §4）
- [x] 接口契约含跨边界契约表（§6.4）
- [x] dev_gate=deferred-until-baseline-verified 标注（§7.4 / §11）
- [x] D3 不重命名存量落地（§1.3 / §7.2）
- [x] LCQ-03/04 落地（§3.1 / §16.1）
- [x] 设计契约映射（§15）
- [x] Gotchas 实质性（§17）
- [x] open_items / tier / shared_fragments（§18）

### 18.5 不授权范围声明（CP5 阶段）

本 LLD CP5 阶段不授权：实现 / runtime / NAS / QMT / trading / provider-lake-catalog 写入 / 物理分区迁移执行 / published pointer 前移执行 / broker lake 实盘写 / Git remote write / 真实 lake 写入。后续物理分区迁移执行按 action scope、运行窗口、脱敏、回滚和审计范围单独 runtime_authorization 授权，后置到基线冻结之后（REQ-247/248）。

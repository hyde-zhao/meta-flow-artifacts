---
story_id: "CR139-S01"
title: "T8 整改对象自动化清册 — LLD"
change_id: "CR-139"
owner_feature: "FEAT-02"
lld_policy:
  required_level: "full-lld"
  trigger_reasons: "d1 纯新建对象，无既有合同"
  rationale: "T8 是 d1 遗漏分析纯新建（handoff §3.9 / §3.12 机械统计 d1=6 之一），必须 full-lld"
wave: "CR139-W1-BASELINE-FREEZE"
tier: "P0"
status: "lld-ready-for-review"
source_hld: "process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md"
source_hld_version: "0.2 (confirmed-cp3)"
source_adr: "process/docs/design/ARCHITECTURE-DECISION-STRATEGY-DATA-FOUNDATION.md"
feature_design_matrix: "docs/design/FEATURE-DESIGN-MATRIX.md v1.13"
req_coverage: ["REQ-213"]
depends_on: []
dependency_type: []
shared_fragments: []
file_ownership:
  primary: ["scripts/lake_inventory.py", "market_data/remediation_inventory.py"]
  shared_read_only: ["market_data/catalog.py", "market_data/manifest.py", "market_data/readers.py", "market_data/lake_layout.py", "market_data/contracts.py"]
  forbidden_write: ["market_data/normalization.py", "market_data/publish.py", "market_data/catalog.py", "market_data/manifest.py", "market_data/readers.py", "market_data/lake_layout.py"]
engagement_mode: "production"
scenario_subject_type: "target-artifact"
created_at: "2026-06-28T19:00:00+08:00"
delegated_by: "host-orchestrator"
---

# CR139-S01 LLD — T8 整改对象自动化清册

> 本 LLD 是 CR139-S01 的 full-lld 设计证据，覆盖 REQ-213（T8 整改对象自动化清册，d1 纯新建）。设计来源：companion HLD `HLD-STRATEGY-DATA-FOUNDATION.md` v0.2 (confirmed-cp3) + ADR v0.2 + handoff v0.7 §3.9/§1.1/§1.2/§1.3.1 + FEATURE-DESIGN-MATRIX v1.13。本文件只写设计证据，不含实现代码；CP5 全量人工确认 + dev_gate 满足后才进入实现。

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 0.1 | 2026-06-28 | meta-dev | CR139-S01 CP5 首版 full-lld：14 章齐全；量化验收 AC-1..AC-8；文件所有权切片（新清册脚本独占，catalog/manifest/readers 只读复用）；Wave1 基线门第 1 步（REQ-247）；与 S02 黄金值契约方向；既有读能力 grep 核验（memory `verify-existing-code-before-claiming-absent`）；3 项非阻断实现灰区写入 open_items。 |

---

## 1. 问题定义与目标

### 1.1 问题陈述

CR-139 Wave1 基线门（REQ-247，CP2 用户追加约束）要求：任何结构性变更（V1 pointer / C1 reader / R1 panel / C2b 读层去重 / N1 物理迁移）必须在 **T8 清册 → T7 黄金值基线 → C2a 重复画像** 三步完成后才允许执行。当前整改对象缺乏"一行命令可核查的自动化清册"：catalog 登记了 17 个 dataset，但 `published` 17/17 全 false、`lineage_checksum` 17/17 全缺失、`pit_status` 14 null + 3 pit_available（handoff §1.3.1）、prices 下堆积 38 个 run_id 分区导致 `(symbol,trade_date)` 可能重复（handoff §1.1/§1.3）。没有清册就无法冻结基线，无法冻结基线则结构性变更后差异无法归因"结构修复 vs 历史数据变化"（D6 强制基准 / HLD §12.3）。

### 1.2 目标（量化）

| 目标 ID | 目标 | 量化成功标准 |
|---|---|---|
| G-S01-1 | 一行命令全量清册 | `uv run --python 3.11 python scripts/lake_inventory.py --lake-root <root>` 退出码 0 且输出 InventoryReport，包含恰好一条 entry 对应 catalog 登记的每个 dataset |
| G-S01-2 | 字段完备 | 每个 entry 含 17 个必填字段（见 §4.2 schema），缺失字段数 = 0 |
| G-S01-3 | 重复键可核查 | 对含 `(symbol,trade_date)` 两列的 dataset，`duplicate_key_count = 总行数 − unique(symbol,trade_date)`，`unique_key_count = unique(symbol,trade_date)`；二者代数和可复核 |
| G-S01-4 | 如实不伪造 | `published` 全 false 时 `published=false` 条目数 = entry 总数（基线 17）；`lineage_checksum_present=false` 条目数 = 17；`pit_status=null` 条目数 = 14、`pit_available` 条目数 = 3（基线，与 handoff §1.3.1 一致）；清册本身不修改这些值 |
| G-S01-5 | 全覆盖 | inventory entry 数 == `CatalogStore.list()` 返回长度 == 17（当前基线，可扩展到 N）；缺漏数 = 0 |
| G-S01-6 | 确定性 | 同一 lake 状态多次运行，除 `scanned_at` 外所有数据字段逐值一致；行序按 dataset 字典序固定 |
| G-S01-7 | 只读边界 | `provider_fetch=0, lake_write=0, catalog_write=0, current_pointer_publish=0, credential_read=0`（REQ-248） |

### 1.3 Requirements（Functional / Non-Functional）

**Functional：**
- FR-1 扫描 catalog（`CatalogStore.list()`）获取全部已登记 dataset，对每个 dataset 扫描 canonical 物理路径（`LakeLayout.canonical_dataset_root` + rglob `*.parquet`，复用 `_canonical_paths` 模式）。
- FR-2 对每个 dataset 统计：row_count、partition_count（run_id 分区数）、source_run_ids、coverage_start、coverage_end、schema_version。
- FR-3 对每个 dataset 从 `CatalogEntry` 读取：published、published_at、pit_status、lineage_checksum（present/absent）、latest_manifest_run_id、quality_status、readiness_status、canonical_path。
- FR-4 对含 `(symbol,trade_date)` 两列的 dataset 计算重复键；对不含两列的 dataset 记录实际 key_schema（若可推断）并标 `key_check_applicable`。
- FR-5 输出机器可解析 JSON（InventoryReport）+ 人类可读摘要（stdout），支持 `--out <path>` 写文件（默认 stdout）。
- FR-6 catalog 登记但 canonical 物理路径不存在的 dataset：记录 `physical_path_exists=false, row_count=0`，不报错。
- FR-7 不修改 catalog/manifest/readers/normalization/publish/lake_layout 任一文件；不连接 provider/NAS/QMT；不读 .env/凭据。

**Non-Functional：**
- NFR-1 性能：对当前 18G raw + 3.5G canonical + 17 dataset 规模，单次清册完成时间 ≤ 5 分钟（pandas read_parquet 统计，非全量加载到宽表——见 §5 技术细节）。
- NFR-2 确定性：输出字段值与扫描顺序无关；source_run_ids 按 run_id 字典序排序。
- NFR-3 安全：满足 REQ-248 不授权范围；只读 parquet + 只读 catalog.json + 只读 manifest.jsonl（如需交叉核验）。
- NFR-4 可维护：纯函数 + dataclass，无全局状态，可被 S02 黄金值基线作为库函数 `build_inventory(lake_root)` 直接调用。
- NFR-5 可审计：InventoryReport 含 `scanned_at`（ISO8601）、`lake_root`（脱敏为相对/标签，不暴露真实私有路径——遵循 handoff §1.1 lake 是 gitignore 软链）、`tool_version`、`schema_version`。

### 1.4 Definition of Done

- AC-1..AC-8（§7）全部 PASS。
- CP6 编码完成门通过：实现对象清单、设计契约映射、测试/Fixture 计划、最小切片验证、平台差异、交接摘要齐备。
- 文件所有权不冲突：仅创建 `scripts/lake_inventory.py` + `market_data/remediation_inventory.py` + `tests/test_cr139_s01_inventory.py`；未修改 catalog/manifest/readers/normalization/publish/lake_layout。
- 只读计数器全 0（REQ-248）。
- 可作为 S02（T7 黄金值基线）的输入被调用。

---

## 2. 模块拆分与文件影响范围

### 2.1 模块拆分

| 模块 | 职责 | 文件（新建=NEW，复用=REUSE 只读） | 类型 |
|---|---|---|---|
| InventoryCore | 清册核心逻辑：扫描 catalog + canonical、统计 row/partition/coverage、计算重复键、组装 InventoryReport | `market_data/remediation_inventory.py` (NEW) | 库函数 + dataclass |
| InventoryCLI | 一行命令入口：解析 `--lake-root`/`--out`/`--format`，调用 InventoryCore，打印/写文件 | `scripts/lake_inventory.py` (NEW) | 薄 CLI 入口 |
| InventoryFixture | fixture 测试：mini lake + 已知重复键 + published=false/lineage 缺失 | `tests/test_cr139_s01_inventory.py` (NEW) | pytest |
| CatalogRead（复用） | 读取 catalog 登记的 dataset 列表与元数据 | `market_data/catalog.py` `CatalogStore.list()/get()` (REUSE 只读) | 既有能力 |
| LakeLayoutRead（复用） | 构造 canonical 路径 | `market_data/lake_layout.py` `LakeLayout.canonical_dataset_root()` (REUSE 只读) | 既有能力 |
| DatasetConstants（复用） | 17 dataset 名常量与 P0/W3 分组 | `market_data/contracts.py` / `market_data/catalog.py` `P0_DATASETS/W3_REQUIRED_DATASETS` (REUSE 只读) | 既有常量 |
| PathScanPattern（复用模式，非函数导入） | canonical parquet rglob 模式 | `market_data/readers.py:375 _canonical_paths` 模式 (REUSE 模式，本地重新实现 rglob，避免导入私有函数) | 模式复用 |

### 2.2 文件影响范围（所有权门控）

| 文件 | 操作 | 所有权 | 冲突检查 |
|---|---|---|---|
| `scripts/lake_inventory.py` | 新建 | primary 独占 | 无冲突（scripts/ 下无同名） |
| `market_data/remediation_inventory.py` | 新建 | primary 独占 | 无冲突（market_data/ 下无同名） |
| `tests/test_cr139_s01_inventory.py` | 新建 | primary 独占 | 无冲突 |
| `market_data/catalog.py` | 只读调用 `CatalogStore.list()/get()` | shared_read_only | 不修改；与 CR139 其他 Story（S03 catalog pointer / S04 readers）无写冲突 |
| `market_data/manifest.py` | 只读（仅交叉核验 lineage_checksum 字段时） | shared_read_only | 不修改 |
| `market_data/readers.py` | 不导入 `_canonical_paths`（私有），仅复用 rglob 模式 | forbidden_write | 不修改；S04/S05 reader Story 写 |
| `market_data/lake_layout.py` | 只读 `LakeLayout` 构造路径 | forbidden_write | 不修改 |
| `market_data/normalization.py` / `publish.py` | 不触碰 | forbidden_write | — |

> memory `verify-existing-code-before-claiming-absent` 已 grep 核验：`CatalogStore.list()`（catalog.py:440）、`CatalogStore.get()`（:417）、`LakeLayout.canonical_dataset_root()`（lake_layout.py:131）、`_canonical_paths` rglob 模式（readers.py:375）均为既有读能力；`scripts/lake_inventory.py` 与 `market_data/remediation_inventory.py` 经 `ls` 确认不存在，是 d1 纯新建。T8 清册脚本是新建，catalog/manifest 读取是既有能力（REQ-249 不适用于 T8 本身，但 T8 不得把既有读能力当新建重复实现——故只复用不重写 `CatalogStore`）。

---

## 3. 数据模型与 schema 变更

### 3.1 schema 变更：无

本 Story 不变更任何既有 schema（catalog.json 结构、manifest.jsonl schema、canonical parquet schema、`CatalogEntry`/`CatalogPointer` dataclass 均不改）。T8 是只读清册，不写 catalog/manifest/canonical。

### 3.2 离线清册产物 schema（InventoryReport，新建离线产物）

```jsonc
{
  "tool_version": "cr139-s01-v0.1",
  "schema_version": "inventory_v1",
  "scanned_at": "2026-06-28T19:05:00+08:00",      // ISO8601，允许跨运行变化
  "lake_root_label": "<redacted-relative-or-tag>", // 脱敏：不暴露真实私有路径
  "catalog_dataset_count": 17,                     // == CatalogStore.list() 长度
  "inventory_entry_count": 17,                     // 必须等于 catalog_dataset_count（AC-5）
  "summary": {
    "published_true_count": 0,                     // 基线 0（17/17 false）
    "published_false_count": 17,
    "lineage_checksum_present_count": 0,           // 基线 0（17/17 缺失）
    "lineage_checksum_absent_count": 17,
    "pit_status_distribution": {"null": 14, "pit_available": 3},
    "duplicate_key_total": <int>,                  // 跨所有 (symbol,trade_date) dataset 的重复行总和
    "physical_missing_count": <int>                // catalog 登记但 canonical 路径不存在的 dataset 数
  },
  "entries": [
    {
      "dataset": "prices",
      "schema_version": "v1",
      "registered": true,
      "published": false,
      "published_at": null,
      "pit_status": null,
      "lineage_checksum_present": false,
      "lineage_checksum_value": null,
      "latest_manifest_run_id": "<str|null>",
      "quality_status": "<str|null>",
      "readiness_status": "<str|null>",
      "canonical_path_catalog": "<str|null>",      // CatalogEntry.canonical_path
      "physical_path_exists": true,
      "physical_canonical_root": "<redacted-relative>",
      "row_count": <int>,                          // 全部 run_id 分区行数总和
      "partition_count": <int>,                    // run_id 分区数（prices 基线 38）
      "source_run_ids": ["<run_id>", ...],         // 字典序排序
      "coverage_start": "<YYYYMMDD|null>",
      "coverage_end": "<YYYYMMDD|null>",
      "key_schema": "(symbol,trade_date)",
      "key_check_applicable": true,
      "duplicate_key_count": <int>,                // row_count - unique(symbol,trade_date)
      "unique_key_count": <int>,                   // unique(symbol,trade_date)
      "columns_present": ["symbol","trade_date",...]
    }
    // ... 其余 16 个 dataset
  ]
}
```

### 3.3 持久化

- InventoryReport 默认输出 stdout（JSON）。`--out <path>` 时写入本地文件系统（如 `reports/cr139/inventory_<scanned_at>.json`）。
- 不写入 lake（`data/lake/**`）、不写入 catalog、不写入 manifest。写 InventoryReport 文件是本地离线产物，不属于 REQ-248 的 lake write / catalog write 计数。

---

## 4. 接口契约

### 4.1 清册命令签名

**CLI（`scripts/lake_inventory.py`）：**

```
uv run --python 3.11 python scripts/lake_inventory.py \
  --lake-root <path> \
  [--out <path>] \
  [--format json|summary] \
  [--dataset <name>]     # 可选：只扫单个 dataset，默认全量
```

- `--lake-root` 必填（或读 `MARKET_DATA_LAKE_ROOT` 环境变量，复用 `readers.py:_lake_root` 模式）。
- `--format json`（默认）：输出完整 InventoryReport JSON。
- `--format summary`：输出人类可读表格摘要（dataset / row_count / coverage / dup / published / pit / lineage）。
- 退出码：0 = 全量扫描完成（含 published 全 false / lineage 全缺等"如实记录"情况）；2 = 硬错误（catalog.json 缺失或不可读、lake_root 不存在、canonical 根不存在）。

**库函数（`market_data/remediation_inventory.py`，供 S02 调用）：**

```python
def build_inventory(lake_root: str | Path) -> InventoryReport: ...
def scan_dataset(dataset: str, layout: LakeLayout, entry: CatalogEntry) -> InventoryEntry: ...
def compute_duplicate_keys(parquet_paths: Sequence[Path], key: tuple[str, ...]) -> DuplicateKeyResult: ...
```

### 4.2 输出结构契约

InventoryReport / InventoryEntry / DuplicateKeyResult 为 frozen dataclass，字段见 §3.2。`to_dict()` / `to_json()` 方法保证可序列化。字段必填性：§3.2 列出的 17 个 entry 字段全部必填（可为 null/0，但键必须存在）——满足 G-S01-2。

### 4.3 与 S02 黄金值基线的契约方向（S01 输出 → S02 消费）

- **方向**：S01 → S02 单向。S01 产出 InventoryReport，S02（T7 黄金值快照）消费它作为"整改前基线状态"。
- **消费字段**：S02 至少消费 `dataset / row_count / coverage_start / coverage_end / duplicate_key_count / unique_key_count / partition_count / source_run_ids`，作为"结构修复 vs 历史数据变化"归因的输入快照。
- **不变量**：S02 不得修改 S01 的 InventoryReport；S01 不依赖 S02（depends_on=[]）。
- **版本协调**：S01 `tool_version` + `schema_version` 写入 InventoryReport 头部，S02 读取时校验 schema_version 兼容；若 S01 schema 演进破坏兼容，S02 须显式处理。
- **回链**：S02 LLD 须在"依赖与文件所有权"引用 S01 return packet / InventoryReport schema 路径。

---

## 5. 关键流程（含 Wave1 基线门）

### 5.1 核心处理流程

```
1. 解析 --lake-root（或 MARKET_DATA_LAKE_ROOT）→ LakeLayout(root)
2. CatalogStore(layout).list() → 全部 CatalogEntry（基线 17 个）         [REUSE 只读]
3. 对每个 entry：
   a. layout.canonical_dataset_root(dataset, schema_version) → root       [REUSE 只读]
   b. root.rglob("*.parquet")（排除 .tmp）→ sorted paths                  [复用 _canonical_paths 模式]
   c. 若 paths 空：physical_path_exists=false, row_count=0, 跳过统计
   d. 若 paths 非空：
      - 用 pyarrow ParquetFile.metadata 逐文件读 row_count（不全量加载 DataFrame，控内存）
      - partition_count = 唯一 run_id 目录数；source_run_ids = 解析路径 run_id= 段，字典序
      - coverage_start/end = min/max(trade_date)（若有 trade_date 列，从 metadata 或首行推断；无则 null）
      - 若 dataset 含 (symbol, trade_date) 两列：
          用 pyarrow 读这两列（列裁剪，不全量加载）→ compute_duplicate_keys
          duplicate_key_count = len(rows) - unique(symbol,trade_date)
          unique_key_count = unique(symbol,trade_date)
      - 否则 key_check_applicable=false, key_schema=<推断或"unknown">
   e. 从 CatalogEntry 读 published/published_at/pit_status/lineage_checksum/quality_status/readiness_status
   f. 组装 InventoryEntry
4. 组装 InventoryReport（summary + entries，entries 按 dataset 字典序）
5. 输出 JSON / summary；--out 则写文件
6. 只读计数器断言：provider_fetch=0, lake_write=0, catalog_write=0, credential_read=0
```

### 5.2 Wave1 基线门定位（REQ-247）

本 Story 是 Wave1 基线冻结门的**第 1 步**（CR-139 §5 Wave1 顺序：T8 → T7 → C2a → 基线冻结门 → V1 → C1 → R1 → C2b → N1 后置）。契约约束：

- **前置**：S01 无 depends_on（Wave1 第 1 步，无前置 Story）。
- **后置门控**：S01 verified 后才能启动 S02（T7 黄金值）；S02 + S03（C2a 重复画像）verified 后才能启动结构性变更 Story（V1/C1/R1/C2b/N1）。
- **归因契约**：S01 InventoryReport 是 S02 黄金值对比的"整改前快照"；结构性变更后 S02 必须能对 S01 快照做差异归因（结构修复 vs 历史数据变化）。
- **不先改物理分区**：S01 只读不改；N1 物理迁移明确后置到基线冻结之后（REQ-247）。

### 5.3 技术设计细节

- **行数统计不全量加载**：用 `pyarrow.parquet.ParquetFile(path).metadata.num_rows` 逐文件累加，避免 pandas concat 全量加载 18G（NFR-1）。仅重复键检查时读 `(symbol, trade_date)` 两列（列裁剪），不全量加载宽表。
- **路径脱敏**：InventoryReport 中 `physical_canonical_root` / `lake_root_label` 输出相对 lake_root 的路径或标签，不输出绝对私有路径（NFR-5，遵循 handoff §1.1 lake 是 gitignore 软链）。
- **确定性**：source_run_ids 排序；entries 按 dataset 字典序；不依赖 os.walk 顺序（用 sorted）。

---

## 6. 异常处理与失败路径

### 6.1 失败路径表

| 场景 | 行为 | 退出码 | 是否伪造 |
|---|---|---|---|
| `--lake-root` 缺失且无 `MARKET_DATA_LAKE_ROOT` | 报错 "lake_root missing"，不扫描 | 2 | 否 |
| `catalog.json` 缺失或不可读（CatalogError） | 报错 "catalog unreadable"，不输出 InventoryReport | 2 | 否 |
| canonical 根不存在（dataset 登记但物理路径无） | entry 记录 `physical_path_exists=false, row_count=0`，继续扫其他 dataset | 0 | 否 |
| `published` 全 false（17/17） | summary 如实记录 `published_false_count=17, published_true_count=0` | 0 | **不伪造为 true** |
| `lineage_checksum` 全缺失（17/17） | 如实记录 `lineage_checksum_present_count=0` | 0 | **不伪造存在** |
| `pit_status` 14 null + 3 pit_available | 如实记录分布 `{"null":14,"pit_available":3}` | 0 | **不伪造为 pit_available** |
| parquet 文件损坏（read_parquet 抛错） | entry 记录 `row_count=-1, error="parquet_unreadable: <code>"`，继续扫其他 | 0 | 否 |
| dataset 无 trade_date 列 | `key_check_applicable=false, key_schema=<推断或"unknown">, duplicate_key_count=null` | 0 | 否 |
| 重复键 > 0 | 如实记录 `duplicate_key_count=<N>`，不阻断（清册是观察工具，非门禁） | 0 | 否 |

### 6.2 硬约束

- 清册是**观察工具**，不是门禁：发现重复键/未发布/lineage 缺失只如实记录，不阻断、不修复、不伪造。
- 任何"catalog 登记了但物理不存在"或"物理存在但 catalog 未登记"的不一致都如实暴露为独立字段，不静默抹平。

---

## 7. 测试设计

### 7.1 验收标准（量化，AC-1..AC-8）

- **AC-1**：`uv run --python 3.11 python scripts/lake_inventory.py --lake-root <fixture-root>` 退出 0，输出 InventoryReport JSON，`inventory_entry_count == catalog_dataset_count`（fixture 注入 17 个 dataset）。
- **AC-2**：每个 entry 含 §3.2 列出的 17 个必填字段，缺失字段数 = 0。
- **AC-3**：对 fixture 中含 `(symbol,trade_date)` 的 dataset，注入已知重复键 K 条，断言 `duplicate_key_count == K` 且 `unique_key_count == 注入 unique 数`。
- **AC-4**：fixture 中 `published` 全 false / `lineage_checksum` 全缺 / `pit_status` 分布 14null+3pit，断言 summary 计数逐项匹配，且 entry 值未被伪造。
- **AC-5**：17→N 全覆盖：`inventory_entry_count == len(CatalogStore.list())`，缺漏数 = 0；构造 1 个 catalog 登记但物理缺失的 dataset，断言其 `physical_path_exists=false, row_count=0`，且总 entry 数不变。
- **AC-6**：确定性：同一 fixture 运行 2 次，除 `scanned_at` 外所有字段逐值相等；entries 顺序为 dataset 字典序。
- **AC-7**：只读计数器：运行后 `provider_fetch=0, lake_write=0, catalog_write=0, current_pointer_publish=0, credential_read=0`（断言 fixture catalog.json mtime/content 未变）。
- **AC-8**：库函数契约：`build_inventory(lake_root)` 可被 S02 直接 import 调用，返回 `InventoryReport` dataclass，`to_dict()` 可 JSON 序列化。

### 7.2 Fixture 设计（`tests/test_cr139_s01_inventory.py`）

| Fixture | 构造 | 断言 |
|---|---|---|
| `mini_lake_17_datasets` | tmp 目录 + 17 个 dataset 的 mini catalog.json + 每个 dataset 少量 parquet | AC-1, AC-2, AC-5 |
| `mini_lake_with_duplicates` | prices dataset 注入 3 条重复 (symbol,trade_date) | AC-3: duplicate_key_count=3 |
| `mini_lake_all_unpublished` | 17 entry 全 published=false, lineage_checksum=null, pit_status 14null+3pit | AC-4 |
| `mini_lake_missing_physical` | 1 个 dataset catalog 登记但 canonical 路径不建 | AC-5 physical_path_exists=false |
| `mini_lake_no_trade_date` | trade_calendar dataset 无 trade_date 列（或非 symbol×trade_date 键） | key_check_applicable=false |
| `mini_lake_deterministic` | 同一 fixture 跑 2 次比对 | AC-6 |
| `mini_lake_readonly` | 运行前后比较 catalog.json hash | AC-7 |

### 7.3 测试执行

```
uv run --python 3.11 pytest -q tests/test_cr139_s01_inventory.py
```

验证方式：static/fixture（CP6/CP7 默认 static-only，runtime 需独立授权）。不连接真实 lake、不读 .env、不启动 runtime。

---

## 8. 实施步骤（最小切片）

| Slice | TASK-ID | 内容 | 文件 | 验证 |
|---|---|---|---|---|
| S01-1 | T8-1 | 新建 `market_data/remediation_inventory.py`：dataclass（InventoryReport/InventoryEntry/DuplicateKeyResult）+ `build_inventory()` + `scan_dataset()` + `compute_duplicate_keys()` | `market_data/remediation_inventory.py` | unit test：mini_lake_17_datasets 通过 |
| S01-2 | T8-2 | 新建 `scripts/lake_inventory.py`：argparse + 调用 `build_inventory` + JSON/summary 输出 + `--out` | `scripts/lake_inventory.py` | AC-1 CLI 退出 0 |
| S01-3 | T8-3 | 新建 `tests/test_cr139_s01_inventory.py`：7 个 fixture + AC-1..AC-8 断言 | `tests/test_cr139_s01_inventory.py` | `pytest -q` 全 PASS |
| S01-4 | T8-4 | 脱敏 + 确定性收尾：path 脱敏、source_run_ids 排序、scanned_at 隔离 | 同 S01-1 文件 | AC-6 确定性 |

每个 Slice 完成后跑 `uv run --python 3.11 pytest -q tests/test_cr139_s01_inventory.py`，全 PASS 才进下一 Slice。CP6 前 4 个 Slice 全完成。

---

## 9. 风险与应对

| 风险 ID | 风险 | 应对 | 残余风险 |
|---|---|---|---|
| R-S01-1 | 行数统计对 18G canonical 全量加载撑爆内存 | 用 pyarrow metadata.num_rows 逐文件累加 + 列裁剪读 (symbol,trade_date)，不全量 DataFrame | 低 |
| R-S01-2 | 重复键检查对 prices（5886 文件）列裁剪仍慢 | 可接受 ≤5min（NFR-1）；若超时，后置到 S03 C2a 重复画像专题 | 中（依赖真实 lake 规模） |
| R-S01-3 | catalog 登记与物理不一致被静默抹平 | 独立字段 physical_path_exists + registered 双向暴露，不抹平 | 低 |
| R-S01-4 | 输出路径泄露真实私有 NAS 路径 | 脱敏为相对/标签（NFR-5） | 低 |
| R-S01-5 | 被误当读路径接入策略（违反 HLD §18 reader 接口下沉） | InventoryReport 不是 reader；`build_inventory` 不返回 DataFrame，不接入 `read_dataset`/`read_panel`；Gotcha §13.1 | 低 |
| R-S01-6 | 与 S03（C2a 重复画像）职责重叠 | S01 是全 dataset 清册（行数/覆盖/键/published/pit/lineage 全字段）；S03 是 prices 38 分区重复键深度画像（哪些 run 重复、来源）。S01 输出 `duplicate_key_count` 标量，S03 输出重复键→run 明细。S03 depends_on S01 | 低 |

---

## 10. 发布与回滚策略

### 10.1 发布

- 本 Story 只读无状态，无数据迁移、无 catalog 写入、无 pointer 前移。
- "发布"= 新增 3 个文件（脚本 + 模块 + 测试）进入仓库；无运行时服务、无部署、无端口。
- 通过 CP6/CP7 后即视为可交付；无独立发布窗口。

### 10.2 回滚

- 回滚 = `git revert` 删除 3 个新建文件。因只读无状态，回滚后 lake/catalog/manifest 状态与 Story 实施前逐字节一致（AC-7 已断言 catalog.json 未变）。
- 无需数据回滚、无 pointer 回退、无 rollback_target。回滚成本 = 删 3 文件。

---

## 11. 平台差异检查

| 维度 | 检查 | 结论 |
|---|---|---|
| 平台目标 | 日频离线研究 + 模拟盘场景；本地 Python 工具（pyproject `requires-python >=3.11,<3.13`） | 无平台差异 |
| 路径 | lake 是 gitignore 软链到外部存储（handoff §1.1）；`--lake-root` 接受软链路径 | 无 |
| 依赖 | 仅复用既有 `market_data` + `pyarrow`/`pandas`（已在 pyproject）；不引入新依赖（DuckDB 不涉及，D4 只读引擎属 S13 相关 Story） | 无新增依赖 |
| OS | WSL2 Linux；纯 Python，无平台特定调用 | 无 |
| 安装结构 | 不涉及安装器/Guardrail/平台适配；无 `PLATFORM-INSTALL-SPEC.md` 依赖 | N/A |
| uv 约束 | 命令统一 `uv run --python 3.11`；不裸 pip、不系统 Python | 符合 CLAUDE.md Python 约束 |

---

## 12. 设计契约映射

| 契约来源 | 章节/ID | 映射到本 LLD | 一致性 |
|---|---|---|---|
| HLD §10 模块职责 | 写侧 normalize/publish/lineage、读侧 PIT/panel/dedup/selector/audit | T8 不属写侧也不属读侧 reader——T8 是 §3.9 测试与护栏层审计工具，只读复用 catalog+canonical 路径，不成为新读路径 | 一致（§2/§13.1） |
| HLD §12.3 Wave1 基线门 | T8 清册 → T7 黄金值 → C2a 画像 → 基线冻结门 → 结构性变更 | S01 = Wave1 第 1 步；S01→S02→S03 verified 后才允许结构性变更 | 一致（§5.2） |
| HLD §18 reader 接口下沉 | 读侧 reader 具体接口签名下沉到 Feature DESIGN | T8 不是 reader，不消费 §18 下沉契约；`build_inventory` 不返回 DataFrame，不接入 read_dataset/read_panel | 一致（§13.1） |
| HLD §1.3 G-6 | T8 清册覆盖 17→N dataset 全量；T7 黄金值差异 100% 归因 | G-S01-5 全覆盖；§4.3 S02 消费契约 | 一致 |
| ADR D6 | 强制基准快照 | S01 是基线冻结第 1 步，S02 黄金值依赖 S01 快照 | 一致 |
| ADR D8a | (d1) 纯新建 6 项纳入（L3/E4/T7/T8/X1/X2） | T8 = d1 纯新建 → full-lld（本文件） | 一致 |
| REQ-213 | T8 一行命令输出全 dataset 行数/覆盖/重复键/pit/published/lineage | FR-1..FR-7 + AC-1..AC-8 | 一致 |
| REQ-247 | Wave1 结构性变更基线门 | §5.2 S01 是基线冻结第 1 步，无前置 | 一致 |
| REQ-248 | 不授权 runtime/NAS/QMT/trading/provider-lake-catalog 写入/物理分区迁移/Git remote write | FR-7 + NFR-3 + AC-7 只读计数器全 0；不写 lake/catalog/manifest | 一致 |
| REQ-249 | 既有合同闭环非新建 | T8 是 d1 不适用 REQ-249；但 T8 复用既有 `CatalogStore`/`LakeLayout` 读能力，不重复实现（memory 核验） | 一致（N/A 但遵守原则） |
| handoff §1.1/§1.2 | lake 物理结构 + 17 dataset 清单 | §1.1 问题陈述 + §3.2 schema + AC-5 | 一致 |
| handoff §1.3.1 | catalog 元数据实际分布（published 17/17 false、pit 14null+3pit、lineage 17/17 缺） | AC-4 如实记录不伪造 | 一致 |
| handoff §3.9 T8 | 一行命令输出全 dataset 清册（行数/覆盖/重复键/pit/published/lineage） | §1.2 G-S01-1..4 + §4 接口 | 一致 |
| memory `verify-existing-code-before-claiming-absent` | 写否定表述前 grep 核验 | §2.2 已 grep 核验 CatalogStore.list/get、LakeLayout.canonical_dataset_root、_canonical_paths 存在；inventory 脚本不存在 | 一致 |
| memory `mechanical-recount-with-script` | 分类计数用脚本机械核验 | AC-4 summary 计数由脚本统计，非手算 | 一致 |

---

## 13. Gotchas（实质性）

1. **T8 清册不是 reader，不能被策略接入（HLD §18 边界）**：常见坑是把 `build_inventory` 当成"全库读接口"给策略或 ML 用。T8 是审计/观察工具，输出 InventoryReport（结构化统计），不输出 DataFrame，不接入 `read_dataset`/`read_panel_as_of`。若 CP5/CP6 把它接入读路径，违反 HLD §18 reader 接口下沉和 FD-47（validate pass ≠ current truth）——清册扫描的是物理 canonical 全部分区（含未发布），与 reader 只读 published current pointer 是两条路径，不可混用。

2. **行数统计必须扫物理全分区，不能只读 catalog pointer 指向的**：清册目的是冻结基线真相，必须扫 `canonical_dataset_root` 下全部 `run_id=` 分区（prices 38 个），统计物理行数与重复键。若只读 `CatalogEntry.canonical_path` 指向的单个路径，会漏掉 38 分区里 catalog 未指向但物理存在的重复行——这正是 C2a 要画像的问题，清册必须先暴露。但清册只输出 `duplicate_key_count` 标量，重复键→run 明细归 S03 C2a。

3. **`published` 全 false / `lineage` 全缺不是错误，是基线真相**：handoff §1.3.1 明确 published 17/17 false、lineage 17/17 缺。清册若把这些当错误阻断或静默填默认值（如 published=true、lineage=fake-hash），就破坏了基线冻结的意义——S02 黄金值对比会基于伪造基线。必须如实记录 `published_false_count=17`、`lineage_checksum_present_count=0`，退出码 0。

4. **catalog 登记与物理不一致要双向暴露，不抹平**：可能出现"catalog 登记了 dataset 但 canonical 路径无文件"或反向。清册用 `registered` + `physical_path_exists` 两个独立字段双向暴露，不能默认二者一致。这是后续 V1 publish gate 修复的输入证据。

5. **路径脱敏不能漏**：lake 是 gitignore 软链到外部存储（handoff §1.1），InventoryReport 若输出绝对路径会泄露私有 NAS 路径（违反 handoff §1.1 + CR-011 凭据脱敏原则）。`lake_root_label` + `physical_canonical_root` 必须输出相对路径或标签。但脱敏不能影响确定性——同一 lake 多次运行的脱敏后路径必须一致。

6. **`build_inventory` 必须是纯函数可被 S02 库调用，不能只做 CLI**：若把全部逻辑塞进 `scripts/lake_inventory.py`，S02 黄金值基线无法 import 复用，会重复实现扫描逻辑。核心逻辑必须在 `market_data/remediation_inventory.py`，CLI 只是薄入口。这是 §4.1 拆 CLI/库的原因。

7. **不要导入 `readers._canonical_paths`（私有函数）**：复用 rglob 模式但本地重新实现，避免依赖私有 API 被 S04/S05 reader 改动时连带破坏清册。catalog.py `CatalogStore.list/get` 是公开 API 可直接导入。

8. **重复键判定主键不是所有 dataset 都是 (symbol,trade_date)**：handoff §2.2.1 的去重防线特指 `(symbol,trade_date)`。financial_pit 有 ann_date/report_period、stock_basic 无 trade_date、trade_calendar 无 symbol。清册对含两列的 dataset 算重复，否则标 `key_check_applicable=false` 并记录推断的 key_schema，不能强行对所有 dataset 套 (symbol,trade_date)。

---

## 14. open_items / tier / shared_fragments

### 14.1 tier

- **P0**（Wave1 基线门第 1 步，阻断 S02/S03 及全部结构性变更 Story）。

### 14.2 shared_fragments

- 无。S01 不向其他 Story 共享可复用代码片段（`build_inventory` 是库函数供 S02 调用，但属接口契约非 shared_fragment）。

### 14.3 实现灰区与取舍记录（LCQ items，非阻断）

> 本 Story 为单线程 LLD 写作（max_parallel_lld=1 场景），按 LLD Clarification Queue 协议例外1，灰区以推荐方案推进并记录于此，blocks_lld=false，CP5 不阻断。host-orchestrator 可在 CP5 统一确认时一并裁决。

| LCQ ID | 灰区 | 推荐方案（默认动作） | 备选 | 影响面 | blocks_lld |
|---|---|---|---|---|---|
| LCQ-CR139-S01-01 | InventoryReport 默认输出位置：stdout vs `reports/cr139/inventory_<ts>.json` | 默认 stdout JSON，`--out <path>` 可选写文件；不写 lake | A: 强制写 reports/ 固定路径（便于 S02 定位）；B: 只支持 stdout（最简） | 接口/S02 消费/CI artifact | false |
| LCQ-CR139-S01-02 | coverage_start/end 推断方式：pyarrow statistics vs 读首/末行 trade_date | 优先 pyarrow `RowGroup.metadata.row_group(i).column(trade_date_idx).statistics.min/max`；不可得则回退读列 min/max | A: 全量读 trade_date 列 min/max（确定性高，慢）；B: 不推断 coverage，留 null | 性能/字段完备性 | false |
| LCQ-CR139-S01-03 | `--format summary` 表格字段集：6 列 vs 全 17 字段宽表 | 6 列精简（dataset/row_count/coverage/dup/published+pit+lineage 合并标记）；完整字段走 `--format json` | A: 17 字段宽表（信息全但终端难读）；B: 仅 dataset+row_count+dup（过简） | 人类可读性/CI 日志 | false |

### 14.4 open_items

| ID | 问题 | 状态 | 重访条件 |
|---|---|---|---|
| O-S01-1 | LCQ-CR139-S01-01/02/03 三项灰区推荐方案是否被 CP5 接受 | OPEN（推荐方案推进） | CP5 人工确认时用户回复 `修改:` 则回更 |
| O-S01-2 | 真实 lake 规模下重复键列裁剪扫描是否满足 NFR-1 ≤5min | OPEN（需 runtime 验证，但 CP6/CP7 默认 static-only） | CP7 若授权 runtime 或 S03 C2a 画像阶段实测 |
| O-S01-3 | S02 LLD 是否确认消费 S01 InventoryReport schema_version 兼容校验 | OPEN（待 S02 LLD 起草） | S02 LLD §依赖与文件所有权 引用本文件 §4.3 |

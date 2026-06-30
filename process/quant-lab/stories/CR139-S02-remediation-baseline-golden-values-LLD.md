---
story_id: "CR139-S02"
title: "T7 整改回归基线+黄金值快照 — LLD"
change_id: "CR-139"
lld_policy: "full-lld"
wave: "CR139-W1-BASELINE-FREEZE"
req_coverage: ["REQ-212"]
status: "lld-ready-for-review"
lld_gate:
  status: "ready-for-cp5-review"
created_at: "2026-06-28T18:30:00+08:00"
created_by: "meta-dev (host-orchestrator fallback)"
source_hld: "process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md v0.2"
source_adr: "process/docs/design/ARCHITECTURE-DECISION-STRATEGY-DATA-FOUNDATION.md v0.2"
---

# CR139-S02 T7 整改回归基线+黄金值快照 — LLD

> CR-139 Wave1 基线门第 2 步 full-lld 设计证据。消费 companion HLD v0.2（confirmed-cp3）、ADR v0.2、MATRIX v1.13、REQUIREMENTS v1.17 REQ-212/REQ-247/REQ-248/REQ-249、handoff v0.7 §3.9/§5/§8.5。本 LLD 是设计证据，不含实现代码；CP5-W1 全量人工确认通过、基线门 S01 verified 后才允许实现 S02，S02 verified 后结构性变更才允许实现（REQ-247）。

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 0.1 | 2026-06-28 | meta-dev (host-orchestrator fallback) | CR139-S02 CP5-W1 首版 full-lld：14 章齐全；黄金值冻结/对比/归因机制；Wave1 基线门第 2 步（REQ-247，D6 强制基准）；消费 S01 InventoryReport；量化验收 + 实质性 Gotchas。meta-dev 子 agent 两次超时，host-orchestrator 代执行（dispatch.mode=inline-fallback，记录于交还摘要）。 |
| 0.2 | 2026-06-28 | host-orchestrator | CP5 评审返修：冻结最小 MetricSpec 7 类（§3.3，row_count/unique_key_dup_count/partition_digest/pit_reader_summary/published_unavailable_count/benchmark_window_symbol_set/panel_join_row_count），实现必须全覆盖不得选择性取值；O-S02-1 RESOLVED；归因规则 §5.3 同步引用 structural_relevance。 |

## 1. 问题定义与目标

CR-139 Wave1 基线门（REQ-247，CP2 用户追加约束）要求：任何结构性变更（V1 pointer / C1 reader / R1 panel / C2b 读层去重 / N1 物理迁移）必须在 **T8 清册 → T7 黄金值基线 → C2a 重复画像** 三步完成后才允许执行。T8（S01）产出清册回答"lake 有什么/缺什么/哪里重复"，但**无法回答"整改后下游结果变了，到底是结构修出来的还是历史数据自然变化的"**。没有黄金值基线 + 归因机制，结构性变更与历史数据变化混在一起无法归因（handoff §5 评审重排核心痛点 / §8.5）。

T7（S02）整改前冻结下游因子/回测结果黄金值快照，整改后重跑对比，输出差异归因报告，100% 归因到"结构修复"或"历史数据变化"二者之一（D6 强制基准 / HLD §12.3）。

### 1.1 量化目标（G-S02）

| ID | 目标 | 量化验收 |
|---|---|---|
| G-S02-1 | 整改前冻结黄金值快照 | `uv run --python 3.11 python scripts/lake_golden_baseline.py freeze --lake-root <root> --out <dir>` 退出码 0 且产出 GoldenBaselineSnapshot，含至少 1 个下游指标（因子值/回测结果）的冻结记录 |
| G-S02-2 | 整改后对比归因 | `compare --baseline <dir> --current <dir>` 退出码 0 且产出 GoldenDiffReport，每条差异 100% 标注 attribution ∈ {structural_fix, historical_data_change, ambiguous} |
| G-S02-3 | 归因可分性 | 结构修复类变更（如 C2b 去重）产生的差异必须能从历史数据变化（如新交易日数据追加）中区分；ambiguous 率 ≤ 5%（量化，非"尽可能低"） |
| G-S02-4 | 快照可复现 | 同一 lake 状态 + 同一 tool_version 重复 freeze 产出 bit-wise 一致的 GoldenBaselineSnapshot（确定性） |
| G-S02-5 | 范围消费 S01 | 黄金值冻结范围由 S01 InventoryReport 的 dataset 清单驱动，不得冻结 S01 未登记的 dataset |

### 1.2 FR / NFR

- FR-1 `freeze` 子命令：消费 S01 `build_inventory(lake_root)` 确定范围，对每个目标 dataset 跑下游因子/回测黄金值计算，冻结快照。
- FR-2 `compare` 子命令：读 baseline + current 两份快照，逐指标对比，输出 GoldenDiffReport（差异指标 / 差异值 / attribution / evidence）。
- FR-3 归因规则：结构性变更（C1/C2/V1/R1/N1 等 Story 实现后）触发的差异 → structural_fix；纯数据追加/历史窗口变化 → historical_data_change；无法区分 → ambiguous（计入 G-S02-3 分母）。
- FR-4 输出机器可解析 JSON（GoldenBaselineSnapshot / GoldenDiffReport）+ 人类可读摘要 stdout。
- NFR-1 性能：freeze 单 dataset ≤ 10 min（日频离线研究场景，CP7 runtime 验证）。
- NFR-2 确定性：同输入同输出（G-S02-4），不含时间戳随机性；`scanned_at` 记录但不参与指标值哈希。
- NFR-3 可审计：快照含 `frozen_at` / `lake_root`（脱敏）/ `tool_version` / `schema_version` / `source_inventory_id`（引用 S01 InventoryReport）。
- NFR-4 不授权范围（REQ-248）：只读 lake，不写 lake/catalog/manifest，不连 provider/NAS/QMT，不读 .env，不启动 runtime。

## 2. 模块拆分与文件影响范围

### 2.1 模块表

| 模块 | 职责 | 文件 | 类型 |
|---|---|---|---|
| GoldenBaselineCore | 黄金值冻结核心：消费 S01 清册 → 跑下游因子/回测 → 组装 GoldenBaselineSnapshot | `market_data/remediation_baseline.py` (NEW) | 库函数 + dataclass |
| GoldenDiffCore | 对比核心：读两份快照 → 逐指标 diff → 归因 → 组装 GoldenDiffReport | `market_data/remediation_baseline.py` (NEW) | 库函数 + dataclass |
| CLI 入口 | `freeze` / `compare` 子命令 | `scripts/lake_golden_baseline.py` (NEW) | CLI |
| 黄金值 fixture | 冻结的黄金值快照样本（测试用） | `tests/golden/` (NEW) | 测试 fixture |

### 2.2 文件所有权

| 文件 | 所有权 | 冲突检查 |
|---|---|---|
| `market_data/remediation_baseline.py` | S02 独占（新建） | 无冲突（新文件） |
| `scripts/lake_golden_baseline.py` | S02 独占（新建） | 无冲突 |
| `tests/golden/` | S02 独占（新建） | 无冲突 |
| `market_data/remediation_inventory.py` (S01) | S02 **只读消费** `build_inventory()` | 跨 Story contract 依赖，非 file-conflict |
| `market_data/readers.py` | S02 **只读**（跑下游因子需读 canonical） | readers.py merge_order 不含 S02（S02 只读不改）；S02 在 S05/S06/S07 修改 readers.py 之前 freeze，读的是整改前状态 |

> 关键：S02 是基线门第 2 步，**在结构性变更（S04-S08）之前**执行。S02 freeze 时 readers.py 仍是整改前状态（glob-concat 38 分区、无 PIT、无去重）——这正是要冻结的"整改前基线"。S02 不修改 readers.py，只读。

## 3. 数据模型 / schema 变更

不变更 lake schema。新建离线产物 schema：

### 3.1 GoldenBaselineSnapshot schema_v1

```json
{
  "schema_version": "golden_baseline_v1",
  "snapshot_id": "<uuid5(lake_root_hash, frozen_at)>",
  "frozen_at": "<ISO8601>",
  "lake_root": "<脱敏相对路径/标签>",
  "tool_version": "<cr139-s02 v0.1>",
  "source_inventory_id": "<S01 InventoryReport.inventory_id>",
  "datasets": [
    {
      "dataset": "prices",
      "row_count": 5886,
      "coverage": {"start": "2019-01-01", "end": "2025-12-31"},
      "metrics": [
        {"metric_id": "factor_momentum_20d_mean", "value": 0.0023, "evidence": {"func": "...", "args": {...}}},
        {"metric_id": "backtest_hs300_annual_return", "value": 0.15, "evidence": {...}}
      ]
    }
  ]
}
```

### 3.2 GoldenDiffReport schema_v1

```json
{
  "schema_version": "golden_diff_v1",
  "baseline_snapshot_id": "...",
  "current_snapshot_id": "...",
  "compared_at": "<ISO8601>",
  "diffs": [
    {
      "dataset": "prices",
      "metric_id": "factor_momentum_20d_mean",
      "baseline_value": 0.0023,
      "current_value": 0.0025,
      "delta": 0.0002,
      "attribution": "structural_fix",
      "evidence": {"structural_change_story": "CR139-S07", "historical_window_change": false}
    }
  ],
  "ambiguous_rate": 0.03
}
```

### 3.3 最小 MetricSpec（CP5 冻结，不再 blocks_lld=false）

> CP5 评审要求：T7 黄金值是 Wave1 基线冻结核心，指标集必须冻结，否则实现可选择性取值，削弱"可复现、可审计、可归因"。以下最小 MetricSpec 在 CP5 冻结，实现必须覆盖全部指标，不得选择性取值；新增指标需 CR/重访。

| metric_id | 维度 | dataset | 计算口径 | structural_relevance | 说明 |
|---|---|---|---|---|---|
| `row_count` | 数据规模 | 每个 dataset | canonical parquet 总行数 | low（数据追加会变） | 历史数据变化敏感 |
| `unique_key_dup_count` | 去重唯一性 | prices/financial_pit/market_cap | `(symbol,trade_date)` 重复键计数 | **high**（S07 C2b 去重后应变 0） | 结构修复敏感：S07 前可能 >0，S07 后 =0 |
| `partition_digest` | 分区结构 | 每个 dataset | 各 run_id 分区 parquet 文件数 + content checksum（按 source_run_id 聚合） | **high**（S08 N1 改分区结构后变） | 结构修复敏感：S08 前 38 run_id 分区，S08 后 dataset/schema_version |
| `pit_reader_summary` | PIT 读取语义 | financial_pit/market_cap/events | 对固定 as_of 样本窗口，`available_at <= as_of` 最新记录数 + 被过滤的未来记录数 | **high**（S05 C1 PIT reader 实现后变） | 结构修复敏感：S05 前无 PIT 过滤（未来记录读入），S05 后过滤 |
| `published_unavailable_count` | published 门禁 | 全 dataset | catalog `published=false` 的 dataset 计数 + reader 拒绝读取次数 | **high**（S04 V1 pointer 前移后变） | 结构修复敏感：S04 前 17/17 false，S04 后部分 true |
| `benchmark_window_symbol_set` | 基准样本 | prices/hs300_index | 基准样本窗口（as_of 起止日期 + symbol 集合的有序 hash） | medium（窗口变化是历史数据变化） | 归因基准：用于区分 structural_fix vs historical_data_change |
| `panel_join_row_count` | 多表 join | panel（prices×financial_pit×market_cap） | 固定 as_of 下 `read_panel` 输出行数 | **high**（S06 R1 panel reader 实现后变） | 结构修复敏感：S06 前 ML 旁路无 panel，S06 后有统一 panel |

**冻结约束**：
- 实现必须计算并冻结上述全部 7 类指标，缺一不可（G-S02-1 验收扩展为"含全部最小 MetricSpec"）。
- `structural_relevance=high` 的指标是归因依据：整改后若这些指标变化且对应 Story（S04-S08）已实现 → structural_fix；仅 `benchmark_window_symbol_set` 变化（窗口/symbol 变） → historical_data_change。
- as_of 样本窗口由 S01 InventoryReport 的 coverage 决定（取每个 dataset 的 [start, end] 中固定 3 个时点：start / mid / end-1day）。
- `partition_digest` 的 checksum 不含时间戳（确定性，G-S02-4）。
- 新增 MetricSpec 需走 CR（重访条件：新整改项引入新结构敏感指标）。

## 4. 接口契约

### 4.1 库函数签名

```python
# market_data/remediation_baseline.py
def freeze_golden_baseline(
    lake_root: Path,
    inventory: InventoryReport,  # 来自 S01 build_inventory()
    metric_specs: list[MetricSpec],
    out_dir: Path,
) -> GoldenBaselineSnapshot: ...

def compare_golden_baselines(
    baseline: GoldenBaselineSnapshot,
    current: GoldenBaselineSnapshot,
    structural_changes: list[StructuralChangeRef],  # 整改后传入：哪些 Story 已实现
) -> GoldenDiffReport: ...
```

### 4.2 CLI 签名

```
uv run --python 3.11 python scripts/lake_golden_baseline.py freeze --lake-root <root> --out <dir>
uv run --python 3.11 python scripts/lake_golden_baseline.py compare --baseline <dir> --current <dir> [--structural-changes <json>]
```

### 4.3 相邻 Story 契约方向

| 方向 | 契约 | 说明 |
|---|---|---|
| S01 → S02 | `build_inventory(lake_root) -> InventoryReport` | S02 freeze 消费 S01 清册确定 dataset 范围（G-S02-5）；引用 `source_inventory_id` |
| S02 → S03 | S02 verified 是 S03 画像的前置 | Wave1 基线门：S01→S02→S03 verified 后基线冻结 |
| S02 → S04-S08 | S02 GoldenBaselineSnapshot 是结构性变更后的对比基准 | 整改后 S04-S08 实现，重跑 compare 归因 |
| S02 → S07 | 归因证据 | S07 去重产生的差异归因为 structural_fix，引用 S02 baseline 对比 |

## 5. 关键流程（含 Wave1 基线门时序）

### 5.1 Wave1 基线门时序（REQ-247）

```
S01 T8 清册 (verified) → S02 T7 黄金值 (verified) → S03 C2a 画像 (verified)
                                                          ↓
                                                   基线冻结门（freeze gate）
                                                          ↓
                                          S04 V1 / S05 C1 / S06 R1 / S07 C2b（结构性变更，dev_gate 要求基线门 verified）
                                                          ↓
                                                   整改后重跑 S02 compare → 归因报告
```

S02 是基线门第 2 步。S02 freeze 必须在 S01 verified 后、S03 画像前完成（S03 画像也属于基线冻结，但 S02 黄金值是下游结果层面的冻结，S03 是数据重复层面的画像，二者互补）。S02 verified 后基线冻结门才关闭，结构性变更才允许实现。

### 5.2 freeze 主流程

1. 调用 S01 `build_inventory(lake_root)` 获取 InventoryReport（若 S01 未 verified 则阻断，dev_gate）。
2. 对 InventoryReport 中每个 dataset，按 MetricSpec 跑下游因子/回测黄金值计算（只读 canonical，用整改前 readers.py 状态）。
3. 组装 GoldenBaselineSnapshot（含 source_inventory_id 引用）。
4. 写 out_dir（JSON），输出人类可读摘要。

### 5.3 compare 主流程

1. 读 baseline + current 两份 GoldenBaselineSnapshot。
2. 逐 dataset 逐 metric diff。
3. 对每条差异，结合 structural_changes（哪些 Story 已实现）判定 attribution：
   - 若 metric 涉及的 dataset 被结构性 Story（S04-S08）触及且 Story 已实现 → structural_fix
   - 若仅 coverage 窗口变化（新交易日追加）→ historical_data_change
   - 二者均不成立 → ambiguous
4. 计算 ambiguous_rate，组装 GoldenDiffReport。

## 6. 异常处理与失败路径

| 场景 | 失败行为 | 处理 |
|---|---|---|
| S01 InventoryReport 缺失 | freeze 阻断 | 退出码非 0，提示先跑 S01；dev_gate 检查 S01 verified |
| 下游因子计算异常 | 该 metric 标 `value=null, error=<msg>` | 不阻断整次 freeze；报告中标注 |
| dataset 无数据 | 该 dataset 标 `row_count=0, metrics=[]` | 如实记录，不伪造 |
| compare 两快照 schema_version 不一致 | 阻断 | 退出码非 0，提示 tool_version 不匹配 |
| ambiguous 率 > 5% | 不阻断 | GoldenDiffReport 标 WARN，提示归因规则需细化（重访 MetricSpec） |
| lake_root 不可读 | 阻断 | 退出码非 0 |

## 7. 测试设计

| AC | 验收 | fixture |
|---|---|---|
| AC-1 | freeze 产出 GoldenBaselineSnapshot 含 ≥1 metric | `tests/golden/test_freeze_basic.json` |
| AC-2 | compare 产出 GoldenDiffReport，每条差异 attribution ∈ {structural_fix, historical_data_change, ambiguous} | `tests/golden/test_compare_attribution.json` |
| AC-3 | 结构修复差异（模拟 S07 去重）归因为 structural_fix | `tests/golden/test_structural_fix.json` |
| AC-4 | 历史窗口变化（追加交易日）归因为 historical_data_change | `tests/golden/test_historical_change.json` |
| AC-5 | 同输入重复 freeze bit-wise 一致（确定性） | `tests/golden/test_determinism.json` |
| AC-6 | ambiguous_rate ≤ 5% | `tests/golden/test_ambiguous_rate.json` |
| AC-7 | source_inventory_id 引用 S01 InventoryReport | `tests/golden/test_s01_contract.json` |
| AC-8 | 只读计数器全 0（不写 lake/catalog，不连 provider/NAS） | `tests/golden/test_no_write.json` |

验证命令：`uv run --python 3.11 pytest -q tests/test_cr139_s02_golden_baseline.py`

## 8. 实施步骤（最小切片）

| Slice | 内容 | 退出条件 |
|---|---|---|
| S02-S1 | GoldenBaselineSnapshot / GoldenDiffReport dataclass + schema | schema 单测过 |
| S02-S2 | freeze_golden_baseline 库函数 + CLI freeze | AC-1/5/7/8 过 |
| S02-S3 | compare_golden_baselines 库函数 + CLI compare + 归因规则 | AC-2/3/4/6 过 |
| S02-S4 | 黄金值 fixture + 全量 AC | AC-1..8 过 |

每 Slice 跑 pytest 才进下一。实现延迟到 CP5-W1 通过 + S01 verified 后。

## 9. 风险与应对

| ID | 风险 | 应对 |
|---|---|---|
| R-S02-1 | 归因歧义（ambiguous 率超 5%） | MetricSpec 显式标注 structural_relevance；重访条件：ambiguous 率 > 5% 时细化归因规则 |
| R-S02-2 | 下游因子计算依赖整改前 readers.py 状态，实现时 readers.py 已被 S05/S06/S07 改 | S02 freeze 在结构性变更前执行；compare 时读历史快照（不重跑），避免 readers.py 漂移影响 |
| R-S02-3 | 真实 lake 规模 freeze 超 NFR-1（10 min） | CP7 runtime 验证；重访条件：超时则裁剪 metric 范围 |
| R-S02-4 | 黄金值快照体积过大 | JSON + 压缩；metric 值只存标量，不存全量 frame |
| R-S02-5 | S01 schema 变更导致 source_inventory_id 断链 | S02 消费 S01 schema_version 校验（S01 §4.3 已声明） |

## 10. 发布与回滚策略

- S02 是离线只读工具，无状态变更。回滚 = `git revert` 删除 `market_data/remediation_baseline.py` + `scripts/lake_golden_baseline.py` + `tests/golden/`，lake/catalog 逐字节不变。
- 黄金值快照产物（JSON）是审计证据，回滚代码不删除已生成的快照文件。
- compare 归因结果不可回滚（审计事实），但可重跑。

## 11. 平台差异检查

- 纯 Python（pyarrow/pandas），跨 Linux/WSL 兼容，无平台差异。
- 不引入 DuckDB（D4 只读引擎归 S13 Wave2）。
- 不涉及安装器/护栏/平台适配。

## 12. 设计契约映射

| 契约 | 来源 | S02 落地 |
|---|---|---|
| Wave1 基线门第 2 步 | HLD §12.3 / REQ-247 / ADR D6 | §5.1 时序 / §4.3 S01→S02 契约 |
| D6 强制基准 | ADR D6 / handoff §6 | §1 G-S02-1/2 / §5.3 compare 归因 |
| 黄金值对比+归因 | handoff §8.5 / §5 | §1.1 G-S02-3 / §5.3 / §13 Gotcha#1 |
| 既有合同闭环非新建 | REQ-249 / memory | T7 是 d1 纯新建（handoff §3.12），无既有合同；S01 build_inventory 是 S01 新建能力（contract 依赖，非重复设计） |
| 不授权范围 | REQ-248 / CR-139 §9 | §1.2 NFR-4 / AC-8 / §11 |
| 消费 S01 清册范围 | S01 LLD §4.3 | §4.3 S01→S02 / §1.1 G-S02-5 |

## 13. Gotchas（规则 9）

1. **归因歧义是核心 gotcha**：structural_fix 与 historical_data_change 在"数据追加 + 结构修复同时发生"时天然难分。不要试图 100% 消除 ambiguous，而是把 ambiguous_rate 量化（≤5%）并标注 evidence；超阈值时细化 MetricSpec 的 structural_relevance 标注，而非调参。
2. **freeze 必须在结构性变更前**：S02 freeze 读的是整改前 readers.py 状态（glob-concat 38 分区、无 PIT、无去重）。这是特性不是 bug——这就是要冻结的"整改前基线"。若在 S05/S06/S07 后才 freeze，冻结的是"整改后状态"，失去基线意义。
3. **compare 不重跑下游**：compare 读历史 GoldenBaselineSnapshot 的 metric 值，不重新计算。重跑会用整改后 readers.py，污染归因。
4. **确定性 vs 时间戳**：snapshot_id 含 frozen_at 哈希用于区分，但 metric 值计算不含时间戳随机性（G-S02-4 bit-wise 一致）。不要把 frozen_at 混入 metric 值哈希。
5. **S01 contract 是硬依赖**：S02 无 S01 InventoryReport 无法确定范围。dev_gate 必须检查 S01 verified，不要 fallback 到自建清册（那会重复 S01，违反 REQ-249）。
6. **黄金值快照是审计证据不是缓存**：不要为节省空间删除旧快照。归因报告依赖历史快照对比。

## 14. open_items / tier / shared_fragments

- **tier**: P0（Wave1 基线门第 2 步，D6 强制基准）
- **shared_fragments**: 无（S02 独占新文件）
- **dependency_type**: runtime（依赖 S01 verified）

### open_items

| O-ID | 描述 | 状态 | 阻断 |
|---|---|---|---|
| O-S02-1 | ~~MetricSpec 具体指标集需 CP5 确认~~ **RESOLVED 2026-06-28**：最小 MetricSpec 7 类已在 §3.3 冻结（row_count/unique_key_dup_count/partition_digest/pit_reader_summary/published_unavailable_count/benchmark_window_symbol_set/panel_join_row_count），实现必须全覆盖，不得选择性取值 | RESOLVED | 否（已冻结） |
| O-S02-2 | 真实 lake 规模 freeze 性能 ≤10min 需 CP7 runtime 验证 | OPEN | 否（CP6/CP7 默认 static-only） |
| O-S02-3 | 归因规则 structural_relevance 标注粒度在 S04-S08 实现后细化 | OPEN | 否（重访条件：ambiguous_rate > 5%） |

## 不授权范围声明

本 Story CP5-W1 阶段不授权：实现 / runtime / NAS / QMT / trading / provider-lake-catalog 写入 / 物理分区迁移执行 / published pointer 前移执行 / broker lake 实盘写 / Git remote write / 真实 lake 写入。LLD 是设计证据，不含实现代码。后续必要验证按 action scope、运行窗口、脱敏、回滚和审计范围单独授权。

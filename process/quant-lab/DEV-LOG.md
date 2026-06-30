---
change_id: CR-139
log_scope: design-evidence (LLD writing phase)
last_updated: 2026-06-28T18:30:00+08:00
---

# CR-139 Development Log

> 记录 CR-139 设计证据写作、实现、验证、修复的滚动日志。每条记录标明所属 Wave / 调度批次。

## 2026-06-28 CP5 Wave1 LLD 批次 — S04/S05/S06 设计证据（meta-dev）

**Wave**: CR139-W1-STRUCTURAL-P0
**调度批次**: CR139-STRATEGY-DATA-FOUNDATION-LLD-BATCH-W1（max_parallel_lld=3，本线程 owns S04/S05/S06）
**Capsule**: `process/context/CP5-CR139-WAVE1-LLD-CONTEXT.yaml`

### 产出

| Story | lld_policy | 设计证据路径 | 状态 |
|---|---|---|---|
| CR139-S04 V1 published pointer / read selector | technical-note（a 类） | `process/stories/CR139-S04-published-pointer-read-selector.md` §技术说明 | lld-ready-for-review |
| CR139-S05 C1 PIT as-of reader | technical-note（a 类） | `process/stories/CR139-S05-pit-as-of-reader.md` §技术说明 | lld-ready-for-review |
| CR139-S06 R1 统一 panel reader | full-lld（c 类） | `process/stories/CR139-S06-unified-panel-reader-LLD.md`（14 章） | lld-ready-for-review |

### CP5 自动预检

- `process/checks/CP5-CR139-S04-published-pointer-read-selector-LLD-IMPLEMENTABILITY.md` — PASS
- `process/checks/CP5-CR139-S05-pit-as-of-reader-LLD-IMPLEMENTABILITY.md` — PASS
- `process/checks/CP5-CR139-S06-unified-panel-reader-LLD-IMPLEMENTABILITY.md` — PASS

### 14 章节自检（S06 full-lld）

S06 LLD 14 章齐全：§1 问题定义与目标（量化 G-S06-1..5）/ §2 模块拆分与文件影响 / §3 数据模型 / §4 接口契约 / §5 关键流程（含基线门时序）/ §6 异常处理 / §7 测试设计 / §8 实施步骤（最小切片）/ §9 风险 / §10 发布回滚 / §11 平台差异 / §12 设计契约映射 / §13 Gotchas（7 条实质性）/ §14 open_items/tier/shared_fragments。另含 §0 上游设计依据。

### readers.py merge_order（关键依赖链）

- **readers.py merge_order**（DEVELOPMENT-PLAN-CR139.yaml，LCQ-02 resolved）：**S05→S06→S07**→S09→S25→S27→S28→S13→S16→S36。S06 在第 2 位，基于 S05 C1 契约。
- **注意**：S04 **不在** readers.py merge_order（S04 owns publish.py/catalog.py/cli.py）。"S04→S05→S06" 是 DAG contract 依赖链，不是 readers.py merge_order。readers.py 实际 merge_order 首位是 S05。
- 其他文件 merge_order：publish_py S04→S14→S22；catalog_py S04→S33→S39；cli_py S08→S04→S14→S35；normalization_py S08→S31→S32→S34→S14→S23→S22。

### 与 HLD/REQ 契约映射摘要

- S04 消费 HLD-DATA-LAKE §5/§17.4/§17.7.1（publish gate / catalog current pointer / 写入时序）；REQ-232 V1；既有合同 `publish.py:605 publish_current_pointer`（dry-run only，已 grep 核验）。
- S05 消费 HLD-DATA-LAKE §17.7.1/§14/§4.3（写入时序 / available_at_rule / PIT universe）；REQ-204 C1；既有合同 `readers.py:2728 read_dataset` + published 门禁 `:2778`（已 grep 核验，修正 handoff :2796 → 实际 :2778）。
- S06 消费 HLD-DATA-LAKE §17.7.1/§14（a 类 read_dataset 合同边界）；companion HLD §5/§9/§10/§12.2/§17；REQ-214 R1；跨边界 owner=FEAT-02 读侧 / consumer=FEAT-03 ML（FD-46..53）；HLD Gotchas#4 R1→V3→R2 顺序。
- Wave1 基线门 REQ-247：S04/S05/S06 属结构性变更，实现前必须 S01/S02/S03 verified（经 S04→S05→S06 链）。

### 既有合同位置核验（memory `verify-existing-code-before-claiming-absent`）

| 既有合同 | 位置 | 核验 |
|---|---|---|
| publish_current_pointer | `publish.py:605` | grep 核验 PASS（dry-run only） |
| read_dataset + published 门禁 | `readers.py:2728` + `:2778` | grep 核验 PASS（handoff 称 :2796 实为 :2778，已修正） |
| read_canonical | `readers.py:2691` | grep 核验 PASS（prices 专用） |
| _canonical_paths | `readers.py:375` | grep 核验 PASS |
| decision_time | `readers.py:227/1790` | grep 核验 PASS（部分支持） |
| catalog published / published_at / pit_status / current_pointer | `catalog.py:56/57/59/248/256` | grep 核验 PASS |

### clarification queue / 未决点（open_items）

- **无新增 LCQ**：4 项 LCQ（LCQ-139-01..04）已 resolved by host-orchestrator 2026-06-28T17:50:00+08:00，本批次遵循 resolved 答案。
- S04 open_items：无（a 类消费既有合同，无设计灰区）。
- S05 open_items：catalog.py 文件所有权边界（S05 只读 pit_status `:59`，不写 catalog；若需物化 pit_status 值则拆 file-conflict 依赖）—— CP5 review 确认。
- S06 open_items：①异粒度 dataset 的 forward-fill/as-of 物化算法（ann_date→trade_date 网格）CP6 实现时冻结；②catalog.py 只读边界（同 S05，不写 catalog）CP5 review 确认。

### 依赖类型 / 文件所有权门控

- S04 depends_on S01/S02/S03（contract + runtime）；owns publish.py/catalog.py/cli.py（merge_order 首位/首位/第2位）。
- S05 depends_on S04（contract）；owns readers.py（merge_order 首位）+ catalog.py（只读，不在 merge_order）。
- S06 depends_on S05（contract）；owns readers.py（merge_order 第 2 位）；跨边界 owner=FEAT-02 读侧 / consumer=FEAT-03 ML。
- 三 Story 无 dev_running 写冲突（readers.py 串行 S05→S06，publish.py/catalog.py/cli_py S04 首位）。

### CP5 结果与待确认项

- 三 Story CP5 自动预检均 PASS。
- 待 host-orchestrator 收齐 8 个 Wave1 Story（S01-S08）设计证据 + 自动预检后，统一发起 `process/checkpoints/CP5-ALL-STORIES-LLD-BATCH.md` 人工确认。
- **本线程停止**：S04/S05/S06 设计证据已提交，等全量 CP5 人工确认。确认通过 + Wave1 基线门满足后，优先复用本线程继续 S04/S05/S06 实现。

### 不授权范围（REQ-248）

本批次不授权：实现 / runtime / NAS / QMT / trading / provider-lake-catalog 写入 / 物理分区迁移执行 / published pointer 前移执行 / 真实 lake 写入 / Git remote write / DuckDB 依赖引入（归 S13 Wave2）。设计证据不含实现。

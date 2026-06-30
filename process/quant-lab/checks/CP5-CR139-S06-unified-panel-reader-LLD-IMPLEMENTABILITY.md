---
checkpoint: CP5
story_id: CR139-S06
story_slug: unified-panel-reader
lld_policy: full-lld
batch_id: CR139-STRATEGY-DATA-FOUNDATION-LLD-BATCH-W1
auto_precheck: true
status: ready-for-review
created_at: 2026-06-28T18:30:00+08:00
created_by: meta-dev
---

# CP5-CR139-S06 LLD Implementability Precheck — R1 统一 panel reader

> 本文件是 CR-139 Wave1 LLD 批次 S06 的 CP5 自动预检。S06 是 c 类 full-lld，设计证据写入 `process/stories/CR139-S06-unified-panel-reader-LLD.md`（14 章）。本预检不含人工确认；全量人工确认由 host-orchestrator 收齐 8 个 Wave1 Story 后统一发起。

## Entry Criteria

- [x] Story 卡片 `process/stories/CR139-S06-unified-panel-reader.md` 存在且 status=lld-pending
- [x] HLD/ADR/MATRIX 已 confirmed-cp3
- [x] lld_policy=full-lld（c 类范围扩展，跨 FEAT-02/03 契约）
- [x] depends_on S05（contract），基线门硬依赖 REQ-247（经 S04→S05 链）
- [x] 文件所有权 readers.py（merge_order 第 2 位）无 dev_running 冲突

## Checklist

| # | 检查项 | 结果 | 证据 |
|---|---|---|---|
| 1 | LLD 文件已创建 `process/stories/CR139-S06-unified-panel-reader-LLD.md` | PASS | 文件存在 |
| 2 | 14 章节齐全（问题定义/模块拆分/数据模型/接口契约/关键流程/异常/测试/实施步骤/风险/发布回滚/平台差异/契约映射/Gotchas/open_items） | PASS | LLD §1-14 + §0 上游依据 |
| 3 | 量化验收条件（未发布行=0 / 泄漏行=0 / 多表 as-of join / 跨边界契约） | PASS | §1.3 G-S06-1..5 |
| 4 | 文件影响基于 handoff §3 代码触点 + DEVELOPMENT-PLAN merge_order | PASS | §2.1 readers.py 切片 + merge_order 第 2 位 |
| 5 | 接口契约含函数签名 + 输入输出 + 与相邻 Story 契约方向 | PASS | §4.1 函数签名 + §4.2 相邻 Story 契约表 |
| 6 | 关键流程含 Wave1 基线门时序（REQ-247） | PASS | §5.1 基线门时序图 + §5.2 调用流程 |
| 7 | 异常处理与失败路径覆盖（未发布/PIT 缺 available_at/复权混用/内存不足） | PASS | §6 失败路径表 |
| 8 | 测试设计含 fixture / 多表 as-of join 正确性 / published 门禁阻断 | PASS | §7 测试用例 T-S06-01..07 |
| 9 | 实施步骤含最小实现切片 | PASS | §8 Slice S06-S1..S4 |
| 10 | 风险与应对（readers.py 串行 / 内存 / catalog 所有权 / 跨边界 / 基线门 / published 门禁） | PASS | §9 R-S06-1..6 |
| 11 | 发布与回滚策略 | PASS | §10 |
| 12 | 平台差异检查（DuckDB 不引入 / parquet 存储 / uv） | PASS | §11 |
| 13 | 设计契约映射回链 HLD §10/§12.2/§17 + ADR + REQ + DEVELOPMENT-PLAN | PASS | §12 契约映射表 |
| 14 | Gotchas 实质性（7 条，非占位） | PASS | §13 Gotchas#1..7 |
| 15 | open_items / tier / shared_fragments | PASS | §14（2 个 open_item，tier=L，shared_fragments=[]） |
| 16 | c 类范围扩展，跨模块契约（FEAT-02 读侧 owner / FEAT-03 ML consumer） | PASS | frontmatter cross_boundary + §4.2 |
| 17 | 复用既有 read_dataset published 门禁（不重复设计，REQ-249） | PASS | §2.2/§4.1 复用 :2728/:2778 |
| 18 | LCQ-02 resolved 遵循（readers.py 函数切片 + merge_order 串行） | PASS | §2.1/§13 Gotcha#5 |
| 19 | 不授权范围声明（无 lake 写 / 无 pointer 前移 / 无 DuckDB 依赖） | PASS | §10/§11/frontmatter |
| 20 | 四道 P0 防线之一（多表 as-of join）标注 | PASS | §1.3 G-S06-1 / companion HLD §12.2 |

## Exit Criteria

- [x] LLD 14 章节齐全且契约可追溯
- [x] 量化验收条件可验证
- [x] 文件所有权无 dev_running 冲突（readers.py merge_order 第 2 位）
- [x] open_items 已暴露（异粒度物化算法 / catalog.py 只读边界）
- [x] Story 状态更新为 lld-ready-for-review
- [ ] 全量 CP5 人工确认（等 host-orchestrator 收齐 8 Story 统一发起）

## Deliverables

- `process/stories/CR139-S06-unified-panel-reader-LLD.md`（14 章 full-lld）
- 本预检文件

## 结论

PASS（自动预检通过，待全量人工确认；2 个 open_item 待 CP5 review / CP6 实现时冻结）

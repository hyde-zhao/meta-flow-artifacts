---
status: draft
version: "1.0"
cr_id: "CR-139"
---

# Backlog — CR-139 Strategy Data Foundation

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-06-28 | meta-pm | 初始 backlog，P2 deferred 项 + 后续 CR 候选 |

## P2 Deferred 项（Wave3）

| Backlog ID | 内容 | 关联 REQ | 优先级 | 延后原因 | 重启条件 |
|---|---|---|---|---|---|
| BL-CR139-01 | L5 replay 接通 published as_of 单日重放 | REQ-245 | P2 | 实盘运维尾部 | Wave2 完成后 |
| BL-CR139-02 | R4 列裁剪/谓词下推 | REQ-217 | P2 | DuckDB 接入自然解决 | SLICE-12 后 |
| BL-CR139-03 | T1 DuckDB 只读 e2e 测试 | REQ-228 | P2 | adapter 实现后 | SLICE-12 后 |
| BL-CR139-04 | M3 quality/ 分区整理 | REQ-210 | P2 | 维护优化 | Wave2 后 |
| BL-CR139-05 | M4 CR→数据审计链 | REQ-211 | P2 | 审计增强 | Wave2 后 |
| BL-CR139-06 | F4 政策周期/shortability 版本化 | REQ-239 | P2 | 因子/交易增强 | SLICE-15 后 |

## 后续 CR 候选

| Candidate ID | 内容 | 来源 | 触发条件 |
|---|---|---|---|
| CAND-CR139-01 | ML feature store 切换独立 store | DEF-139-01 / D5 附条件 | feature 规模/跨湖查询/在线推理延迟触发 |
| CAND-CR139-02 | 物理分区迁移 / 候选压缩执行 | DEF-139-02 / Wave1 N1 后置 | T7 黄金值对比归因完成后 |
| CAND-CR139-03 | 存量 run_id 一次性重命名 | DEF-139-04 / D3 | 重跑替换不可行且命名冲突阻塞 |
| CAND-CR139-04 | runtime_authorization（真实 lake write/pointer 前移/broker lake write/读审计写） | REQ-248 不授权范围 | Wave1/2 切片需真实执行时，按 action scope 单独授权 |
| CAND-CR139-05 | DuckDB 依赖引入 | DQ-CR139-02 | CP3/CP5 批准 + 内存瓶颈 |

## 备注

- 本 backlog 仅作 CP2 范围规划输入，不构成 CR 启动授权。
- 后续 CR 启动需重新做冲突预检、CP2/CP3/CP5 和运行授权（规则 41）。
- P2 deferred 项不占执行锁，`candidate`/`spike_candidate` 状态（规则 42）。

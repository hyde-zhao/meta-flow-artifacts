---
checkpoint_id: "CP7"
checkpoint_name: "验证完成检查"
type: "auto"
status: "PASS"
story_id: "STORY-038-07"
story_slug: "case-runner-sw"
cr_id: "CR-038"
wave: 3
design_evidence_type: "technical-note"
validation_mode: "mixed"
executed_by: "meta-qa"
executed_at: "2026-08-15"
---

# CP7 验证完成检查 — STORY-038-07 case_runner env-file SW 契约兼容

## Entry Criteria

| 条目 | 结果 |
|---|---|
| Story `status=ready-for-verification` | PASS |
| CP6 编码完成门 PASS | PASS |
| technical-note 设计证据已确认 | PASS |
| 依赖门控（S06 hard）满足 | PASS |

## 验证对象清单

| 对象 | 文件 | 核对结果 |
|------|------|---------|
| `_PRIMARY_NODE_IDS = ("tg1","dut1")` | case_runner.py | 主实例必填键显式隔离 |
| `load_env_file` docstring + 注释 | case_runner.py | swN 容忍契约固化，行为不变 |
| `build_env_topology` 循环改常量 | case_runner.py | `for node_id in _PRIMARY_NODE_IDS` |
| `test_env_file_sw.py`（10 例） | skills/case-execution/tests/ | 全绿 |

## 验证追踪矩阵

| 场景 | 结果 |
|------|------|
| SCN-038-POS-007（swN 容忍） | PASS（single/multi/incomplete swN） |
| tg1/dut1 主实例校验不破坏 | PASS（tg1/dut1 缺失/缺 interfaces 仍抛 EnvTopologyError） |
| R-NF-004（无 swN 行为不变） | PASS |
| SCN-038-BND-001（CR-037 回归） | PASS（55/55） |

## 设计契约验证

- `build_env_topology` 仅遍历 `_PRIMARY_NODE_IDS`，swN 天然跳过，不引入新校验失败；tg1/dut1 缺失仍抛 `EnvTopologyError`（DA-006）。
- 与 return packet `deviations_from_lld` 一致：现状已天然容忍 swN，本 Story 以常量 + 注释固化契约并锁测试，行为零变化。

## 分层验证结果

- 单测：`skills/case-execution/tests/` 10/10 PASS。
- 回归：`tests/test_topo_plan_cr037.py tests/test_resolve_addresses.py tests/test_tg_dry_run.py tests/test_config_once_skip.py` 55/55 PASS。
- `--execute` 真机执行：N/A（DQ-038-02，本 Story 仅解析层放宽，无真机/网络写）。

## 功能 / 异常 / 回归 / 集成

- 功能：env-file 解析层容忍 swN，tg1/dut1 必填校验不破坏。
- 异常：tg1/dut1 缺失/非 dict/缺 interfaces 抛 EnvTopologyError；trex_api_url/dut.host 缺失降级 warning 不抛（ADR-05）。
- 回归：CR-037 load_env_file 契约测试全绿。
- 集成：swN 容忍依赖 build_env_topology 仅遍历主实例的实现约定，常量已隔离边界。

## 非功能

- 无凭据/真机/网络写；行为零变化；加性常量 + 注释。

## 缺陷 / 问题

- Return Packet 存在（STORY-038-07.return.json），无缺陷。
- 风险：若未来 build_env_topology 改为全量遍历 nodes 校验，需保留 swN 跳过语义（常量已显式隔离）。

## 剩余风险

- swN 容忍依赖实现约定（非阻塞，注释 + 常量已固化）。
- `--execute` 真机执行路径的 swN 消费不在本 Story（无 swN 占位，DQ-038-02）。

## 阶段决策

**PASS** —— 解析层加性放宽，测试 + 回归全绿，tg1/dut1 校验不破坏，无凭据/真机。路由：none（进入 verified）。

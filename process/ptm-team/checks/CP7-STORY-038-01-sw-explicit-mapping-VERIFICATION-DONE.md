---
story_id: STORY-038-01
story_slug: sw-explicit-mapping
cr_id: CR-038
checkpoint: CP7
result: PASS_WITH_RISK
validation_mode: mixed
created_at: "2026-08-15"
author: meta-qa
design_evidence_type: full-lld
---

# CP7-STORY-038-01-sw-explicit-mapping-VERIFICATION-DONE

## 结论

| 项目 | 内容 |
|---|---|
| 阶段决策 | PASS_WITH_RISK |
| validation_mode | mixed（pytest + dry-run 等价验证） |
| 路由 | none（verified；风险汇入 CP8） |

## 验证对象清单

| 文件 | 操作 | 验证结果 |
|---|---|---|
| `skills/topo-config/src/topo_mapper.py` | 修改（`explicit_sw_reserved`/`_compute_explicit_sw_reserved`/`_build_unmapped_reason`/`_try_l2_pass_through` 跳过/`_build_result`） | PASS |
| `skills/topo-config/tests/test_topo_mapper_sw.py` | 新增（8 用例） | PASS |
| `skills/topo-config/tests/fixtures/sw_explicit.sample.yaml` | 新增 | PASS |

## 验证追踪矩阵

| 场景/契约 | 测试 | 结果 |
|---|---|---|
| SCN-038-POS-001/002 | `test_explicit_sw_matched_and_role` | PASS |
| SCN-038-NEG-001 | `test_sw_no_candidate_reason` | PASS |
| SCN-038-NEG-002 | `test_sw_interface_mismatch_reason` | PASS |
| SCN-038-BND-002 | `test_mock_still_virtual` | PASS |
| R-F-018（direct/端到端/顺序无关） | `test_explicit_sw_reserved_{excludes_passthrough,end_to_end,order_invariance}` | PASS |
| R-NF-004（回归无破坏） | `test_cr037_node2_regression` | PASS |

## 设计契约验证

- R-F-001/R-F-002 已由既有 matched 分支 `role=node.role` 确认（S3 审查结论，无需改字段）
- R-F-018 显式 SW 候选 ∩ 自动透传候选 = ∅，交换处理顺序不变量不变（顺序无关单测守护）
- R-NF-002 结构化失败 reason 四级分类接入 `_build_result`
- 平台差异 N/A（纯内存映射引擎）

## 分层验证计划

- 层A：`test_topo_mapper_sw.py` 独立复跑 8 passed；全量 340 passed / 0 failed
- 层B：N/A（无真机下发）

## 8 维度验收

| # | 维度 | 结果 |
|---|---|---|
| 1 | 完整性（BLOCKING） | PASS（3 产物 ≥ 验收标准覆盖） |
| 2 | 平台适配（BLOCKING） | PASS（N/A：纯内存引擎，无平台路径） |
| 3 | 验收标准覆盖（BLOCKING） | PASS（7/7 覆盖） |
| 4 | 安全合规（BLOCKING） | PASS（无凭据/无真机/无危险命令） |
| 5 | 命名规范（REQUIRED） | PASS（kebab-case/snake_case） |
| 6 | Frontmatter（REQUIRED） | PASS（Story 卡片字段完整） |
| 7 | 可安装性（REQUIRED） | PASS（N/A：纯代码，无安装脚本） |
| 8 | 文档覆盖（OPTIONAL） | 文档阶段检查 |

## 测试证据

- `uv run --python 3.11 pytest skills/topo-config/tests/test_topo_mapper_sw.py -q` → 8 passed
- 全量 `... pytest skills/topo-config/tests/ skills/topo-planning/tests/ -q` → 340 passed, 0 failed

## 问题与缺陷

| # | 严重度 | 问题 | 状态 |
|---|---|---|---|
| Q1 | Minor（输入缺陷） | 缺 `process/returns/STORY-038-01.return.json` + `process/evidence/STORY-038-01.index.json` | 建议 CP8 前回填 |

## 剩余风险

- `explicit_sw_reserved` 保守保留（ADR-CR038-01）：真机极端场景可能过度保守，备选方案见 LLD §9
- 证据链缺口（缺 return/index）

## 阶段决策

PASS_WITH_RISK → verified；风险（证据链缺口 + 保守保留语义）汇入 CP8 Decision Brief。

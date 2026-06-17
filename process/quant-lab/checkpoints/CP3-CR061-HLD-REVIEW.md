# CP3-CR061 HLD Review 人工检查点

## 自动预检摘要

自动预检结论：PASS_WITH_RISK。CR061 架构推荐 staged compatibility-first：保留现有顶层 import roots，后续经批准新增 `quant_lab` compatibility namespace；bulk physical relayout 需要二次执行授权。

## Decision Brief

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP3-CR061-DESIGN-CONTEXT.yaml` |
| capsule 状态 | ready |
| read_profile | compact |
| 默认读取策略 | 先读 capsule；架构冲突时读 layout plan 和 candidate list |
| 全文档读取扩展 | 已读取 `pyproject.toml`、repo import scan 摘要和 CR060 / CR058 边界 |
| 缺失 / waived 理由 | N/A |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue | `process/STATE.md.human_gate_decisions.pending_human_decisions[]` | scanned | 0 | 0 | 当前无既有 pending DQ。 |
| 自动预检结果 | `process/checks/CP3-CR061-HLD-CONSISTENCY.md` | scanned | 5 | 5 | 架构风险转 DQ。 |
| layout plan | `docs/release/REPO-LOCAL-PACKAGE-LAYOUT-PLAN-CR061.md` | scanned | 7 | 5 | 合并为架构决策。 |
| candidate list | `docs/release/CR061-PACKAGE-IMPORT-CANDIDATE-LIST.md` | scanned | 16 | 5 | 高风险项合并。 |
| repo import scan | `rg from/import roots` | scanned | 295 files | 2 | 转换为 legacy root / regression 策略。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP3-CR061-01 | architecture | package/import/layout 架构是否采用 staged compatibility-first？ | 采用 staged compatibility-first。 | 备选 A：一次性 bulk move；备选 B：永久 legacy roots。 | 推荐方案最稳；bulk move 最整洁但失败面最大；永久旧 roots 不完成收敛。 | 会短期保留双入口。 | 完成 full dry-run 和二次授权后可切 bulk move。 |
| DQ-CP3-CR061-02 | architecture | legacy import roots 是否在兼容期保留？ | 保留 `engine`、`market_data`、`strategies`、`trading`。 | 备选 A：立即弃用旧 roots；备选 B：只保留部分 roots。 | 推荐方案保护现有测试和脚本；弃用会造成大规模改动。 | 新旧命名并存导致文档需说明。 | 新 namespace 和全量测试稳定后再 deprecate。 |
| DQ-CP3-CR061-03 | implementation | 是否新增 `quant_lab` namespace 作为后续实现候选，而非本次立即创建？ | 作为后续候选；当前只审批设计。 | 备选 A：本轮立即创建；备选 B：不建 namespace。 | 推荐方案保守；立即创建需要 CP6/CP7；不建则迁移价值有限。 | 下一步还需实现授权。 | 用户 approve 后进入实现准备。 |
| DQ-CP3-CR061-04 | security | `trading/**` 是否仅允许 static import/layout review？ | 是，禁止 runtime、账户、交易和凭据。 | 备选 A：合并 CR046 runtime 恢复；备选 B：排除 `trading/**`。 | 推荐方案保留代码结构视野且不触发 runtime；恢复 CR046 与用户指令冲突；排除会留下布局缺口。 | 与 CR046 潜在冲突需持续标注。 | 用户明确恢复 CR046 时重新路由。 |
| DQ-CP3-CR061-05 | implementation | 测试架构是否优先 offline full pytest / import smoke？ | 优先 offline full pytest；必要时用批准的 subset + smoke。 | 备选 A：只做静态 grep；备选 B：运行 provider/lake/runtime 集成。 | 推荐方案验证强；静态 grep 不够；provider/lake/runtime 越权。 | full pytest 可能耗时或受 dirty state 影响。 | 如果 full pytest 阻塞，记录 subset 风险。 |

不授权项：CP3 通过不授权真实 file move、bulk import rewrite、`pyproject.toml` / `uv.lock` 修改、Git remote、NAS、lake、provider、凭据、runtime 或 CR046 recovery。

auto_final_authorization: false

## Entry Criteria

| 条件 | 结果 |
|---|---|
| CR061 baseline 已创建 | PASS |
| candidate list 和 layout plan 存在 | PASS |

## Checklist

| 检查项 | 结果 |
|---|---|
| 架构方案有备选和切换条件 | PASS |
| legacy root 兼容策略明确 | PASS |
| high-risk `trading/**` 边界明确 | PASS_WITH_RISK |
| 回归策略明确 | PASS_WITH_RISK |

## Exit Criteria

| 条件 | 结果 |
|---|---|
| 用户确认后可进入 CP5 readiness gate | PASS |

## Deliverables

- `process/checks/CP3-CR061-HLD-CONSISTENCY.md`
- `process/checkpoints/CP3-CR061-HLD-REVIEW.md`
- `process/context/CP3-CR061-DESIGN-CONTEXT.yaml`

## 人工审查结果

- 结论：approved
- 审查人：user
- 审查时间：2026-06-15T07:52:05+08:00
- 修改意见：用户回复“同意”，按 `approve` 解析，接受 DQ-CP3-CR061-01..05 推荐方案。
- 风险接受项：接受 staged compatibility-first 架构；legacy import roots 在兼容期保留；`trading/**` 仅 static import/layout review，不触发 CR046 runtime。

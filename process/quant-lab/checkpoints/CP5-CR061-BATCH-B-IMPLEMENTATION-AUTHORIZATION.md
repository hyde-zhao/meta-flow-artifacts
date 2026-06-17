# CP5-CR061 Batch B Implementation Authorization 人工检查点

## 自动预检摘要

自动预检结论：PASS_WITH_RISK。CR061 gate-only CP5 已批准；本检查点只请求授权一个窄实现切片：`pyproject.toml` metadata、`uv.lock` 再生成、可选 `quant_lab` 兼容命名空间和离线 import smoke test。用户回复 `approve` 后，才允许执行这些 Batch B narrow implementation；仍不授权 bulk move、bulk import rewrite、Git remote/NAS/data lake/provider/runtime/凭据/CR046 recovery。

## Decision Brief

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP5-CR061-BATCH-B-AUTHORIZATION-CONTEXT.yaml` |
| capsule 状态 | ready-for-human-gate |
| read_profile | compact |
| 默认读取策略 | 先读 capsule；实现前读取 Batch B manifest、dirty worktree policy、Feature design/test/tasks。 |
| 全文档读取扩展 | 已读取 CR061 gate-only CP5、candidate list、layout plan、manifest、rollback ref 和 preserve-audit。 |
| 缺失 / waived 理由 | N/A |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| CR061 gate-only CP5 | `process/checkpoints/CP5-CR061-PACKAGE-IMPORT-LAYOUT-READINESS.md` | scanned | 5 | 2 | Gate-only 批准后仍需二次实现授权。 |
| Dirty worktree policy | `docs/release/CR061-BATCH-B-DIRTY-WORKTREE-INCLUDE-EXCLUDE.yaml` | scanned | 8 | 1 | include/exclude 和 exclude exceptions 转 DQ。 |
| Batch B manifest | `docs/release/CR061-BATCH-B-CANDIDATE-MANIFEST.yaml` | scanned | 12 | 2 | action set、blocked action、rollback requirement 转 DQ。 |
| Feature design | `docs/features/cr061-package-import-layout/DESIGN.md` | scanned | 7 | 0 | 已并入 DQ-CP5-CR061B-01..05。 |
| Test plan | `docs/features/cr061-package-import-layout/TEST-PLAN.md` | scanned | 7 | 0 | 已并入 DQ-CP5-CR061B-05。 |
| STATE pending queue | `process/STATE.md.human_gate_decisions.pending_human_decisions[]` | scanned | 0 | 0 | 当前无额外 pending DQ。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP5-CR061B-01 | runtime_authorization | 是否授权 CR061 Batch B 窄实现切片？ | 授权仅执行 `pyproject.toml` metadata、`uv.lock` 再生成、可选 `quant_lab` 兼容命名空间和离线 smoke test。 | 备选 A：只授权 metadata + lock；备选 B：继续停留在 gate-only。 | 推荐方案能实际推进 package identity；只改 metadata 风险更低但价值较小；gate-only 不推进迁移。 | 会产生真实 repo-local 代码 / lock diff，但不触碰外部面。 | 任一实现动作需要 bulk move/import rewrite 时立即停止并返工门禁。 |
| DQ-CP5-CR061B-02 | implementation | dirty worktree include/exclude 是否作为 Batch B 执行边界？ | 采用 `CR061-BATCH-B-DIRTY-WORKTREE-INCLUDE-EXCLUDE.yaml`，只允许列出的 include paths。 | 备选 A：要求 clean worktree 后再实现；备选 B：扩大 include 到更多文档。 | 推荐方案可在当前脏工作树继续小步推进；clean worktree 更稳但会推迟；扩大 include 增加误改历史证据风险。 | 当前大量 CR058-CR061 过程文件不应被误纳入实现 diff。 | 实现前 `git status` 与 include/exclude 冲突时 BLOCKED。 |
| DQ-CP5-CR061B-03 | risk_acceptance | rollback_ref 当前复用 CR059 planning-only bundle，是否允许 Batch B 以轻量 diff rollback 继续？ | 接受：Batch B 只做小 diff 回滚，不刷新 Git bundle；实现前记录 pre/post diff 和 smoke evidence。 | 备选 A：先刷新 Git bundle；备选 B：无 rollback evidence 直接做。 | 推荐方案动作小且不触发 Git/NAS；刷新 bundle 更强但扩大操作；无证据直接做不可接受。 | 若实现 diff 超出小范围，现有 rollback 不足。 | diff 超出 include 或 lock diff 异常时停止并要求 fresh bundle。 |
| DQ-CP5-CR061B-04 | security | 是否继续禁止 Git remote、NAS/data lake、provider、runtime、凭据和 CR046 recovery？ | 确认继续禁止；Batch B 只做 repo-local offline implementation。 | 备选 A：合并 Git remote 初始化；备选 B：合并 NAS/lake/runtime 验证。 | 推荐方案风险分层清晰；合并 Git 影响发布面；合并 NAS/lake/runtime 会混合外部状态和凭据风险。 | 完整迁移仍需后续 CR。 | 用户要求合并时需新 CR 影响分析和单独门禁。 |
| DQ-CP5-CR061B-05 | risk_acceptance | Batch B 验证范围是否限定为离线 import smoke 和必要 pytest subset？ | 接受离线 subset；不执行 provider/lake/runtime；全量 pytest 如触发外部面则降级并记录风险。 | 备选 A：强制 full pytest；备选 B：只做静态检查。 | 推荐方案能验证关键 import 风险且避免外部副作用；full pytest 覆盖更强但可能触发外部依赖；只静态检查不足。 | 可能留下非 import 行为回归风险。 | 若 smoke 失败或 full pytest 可离线安全运行，则调整验证范围。 |

不授权项：

| Item ID | 不授权操作 |
|---|---|
| NA-CP5-CR061B-001 | 不执行 physical move / rename / delete。 |
| NA-CP5-CR061B-002 | 不执行 bulk import rewrite。 |
| NA-CP5-CR061B-003 | 不删除 `engine/**`、`market_data/**`、`strategies/**`、`trading/**` legacy roots。 |
| NA-CP5-CR061B-004 | 不改写历史 `process/**`、checks、checkpoints、handoffs、Story、LLD、DEV-LOG 或历史 CR。 |
| NA-CP5-CR061B-005 | 不执行 Git remote add / push / tag / branch rename / history rewrite。 |
| NA-CP5-CR061B-006 | 不执行 NAS 数据 copy / move / delete / archive promote。 |
| NA-CP5-CR061B-007 | 不执行 `MARKET_DATA_LAKE_ROOT` replacement、lake write、catalog publish、provider fetch。 |
| NA-CP5-CR061B-008 | 不读取 `.env`、token、password、cookie、session、private key、账户或交易事实。 |
| NA-CP5-CR061B-009 | 不启动 QMT / MiniQMT runtime、账户查询、下单、撤单、simulation/live。 |
| NA-CP5-CR061B-010 | 不恢复或推进 CR046 CP7。 |

auto_final_authorization: false

## Entry Criteria

| 条件 | 结果 |
|---|---|
| CR061 gate-only CP5 已 approved | PASS |
| Batch B candidate manifest 已冻结 | PASS |
| Dirty worktree include/exclude 已冻结 | PASS |
| Feature design / test / tasks 已生成 | PASS |
| 自动预检已完成 | PASS_WITH_RISK |

## Checklist

| 检查项 | 结果 |
|---|---|
| 实现范围是否明确且可回滚 | PASS |
| 决策项是否完整 | PASS |
| 不授权项是否完整 | PASS |
| rollback 风险是否显式化 | PASS_WITH_RISK |
| 用户授权 | PASS |

## Exit Criteria

| 条件 | 结果 |
|---|---|
| 用户回复 `approve` 后，只授权 Batch B-ACT-001..005 中仍必要的窄实现动作 | PASS |
| 用户回复 `修改: <具体修改点>` 后返工门禁 | N/A |
| 用户回复 `reject` 后停止 CR061 Batch B | N/A |

## Deliverables

- `docs/release/CR061-BATCH-B-DIRTY-WORKTREE-INCLUDE-EXCLUDE.yaml`
- `docs/release/CR061-BATCH-B-CANDIDATE-MANIFEST.yaml`
- `docs/design/FEATURE-DESIGN-MATRIX-CR061.md`
- `docs/features/cr061-package-import-layout/DESIGN.md`
- `docs/features/cr061-package-import-layout/TEST-PLAN.md`
- `docs/features/cr061-package-import-layout/TASKS.md`
- `process/context/CP5-CR061-BATCH-B-AUTHORIZATION-CONTEXT.yaml`
- `process/checks/CP5-CR061-BATCH-B-IMPLEMENTATION-AUTHORIZATION.md`
- `process/checks/CP5-CR061-BATCH-B-HUMAN-GATE-LAUNCH-MESSAGE.md`

## 人工审查结果

- 结论：approved
- 审查人：user
- 审查时间：2026-06-15T08:55:37+08:00
- 修改意见：用户回复“同意”，按 `approve` 解析，接受 DQ-CP5-CR061B-01..05 推荐方案。
- 风险接受项：授权 CR061 Batch B 窄实现：`pyproject.toml` metadata、`uv.lock` 再生成、可选 `quant_lab` 兼容命名空间和离线 smoke test；接受 dirty worktree include/exclude 作为执行边界；接受轻量 diff rollback，不刷新 Git bundle；验证限定为离线 import smoke 和必要 pytest subset。仍不授权 physical move / rename / delete、bulk import rewrite、删除 legacy roots、Git remote、NAS/data lake/provider/catalog、凭据、QMT/MiniQMT runtime 或 CR046 recovery。

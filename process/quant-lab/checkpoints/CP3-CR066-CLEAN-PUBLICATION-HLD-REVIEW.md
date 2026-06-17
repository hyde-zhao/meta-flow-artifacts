# CP3-CR066 Clean Publication HLD Review 人工检查点

## 自动预检摘要

自动预检结论：PASS_WITH_RISK。推荐 clean snapshot / allowlist manifest 架构；CR066 不执行 branch creation、history rewrite、commit、remote add 或 push，后续执行必须另起 CR067 或等价执行门禁。

## Decision Brief

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP3-CR066-CLEAN-PUBLICATION-DESIGN-CONTEXT.yaml` |
| capsule 状态 | ready |
| read_profile | compact |
| 默认读取策略 | 先读 capsule；架构冲突时读 strategy / audit / manifest |
| 全文档读取扩展 | 已读取 CR066 strategy、audit、manifest |
| 缺失 / waived 理由 | N/A |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue | `process/STATE.md.human_gate_decisions.pending_human_decisions[]` | scanned | 0 | 0 | 当前 CR066 DQ 已在本 checkpoint 聚合。 |
| 自动预检结果 | `process/checks/CP3-CR066-CLEAN-PUBLICATION-HLD-CONSISTENCY.md` | scanned | 5 | 5 | 架构和安全风险转 DQ。 |
| strategy | `docs/release/CLEAN-PUBLICATION-STRATEGY-CR066.md` | scanned | 8 | 5 | 方案对比和推荐路线转 DQ。 |
| audit | `docs/release/TRACKED-FORBIDDEN-PATHS-AUDIT-CR066.md` | scanned | 5 | 3 | tracked findings 合并。 |
| manifest | `docs/release/CLEAN-PUBLICATION-MANIFEST-CR066.yaml` | scanned | 10 | 4 | 禁止项和条件 include 合并。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP3-CR066-01 | architecture | clean publication 架构是否采用 clean snapshot / allowlist manifest？ | 采用 clean snapshot / allowlist manifest，执行另起 CR。 | 备选 A：current branch as-is；备选 B：orphan branch；备选 C：history rewrite。 | 推荐风险可控；as-is 违反 exclude；orphan 需分支授权；rewrite 高风险。 | 发布延迟但安全。 | 若用户要求同仓 clean branch，进入 CR067 设计。 |
| DQ-CP3-CR066-02 | security | `.env.example` 是否 conditional allow？ | 是；仅 placeholder-only / no-secret scan PASS 后允许。 | 备选 A：始终排除；备选 B：始终允许。 | 推荐平衡；始终排除降低可用性；始终允许风险高。 | 需要扫描规则。 | 任一真实敏感模式命中则排除。 |
| DQ-CP3-CR066-03 | implementation | `reports/**` / `runs/**` 是否默认排除？ | 默认排除 clean public baseline。 | 备选 A：风险接受后发布；备选 B：私有仓库；备选 C：历史清理。 | 推荐符合源码发布面；其他方案要扩大授权。 | 过程证据可能不进入公开仓。 | 用户接受风险时另列 DQ。 |
| DQ-CP3-CR066-04 | security | NAS / internal topology 文件如何处理？ | 默认不纳入；如需要发布必须 redaction 或风险接受。 | 备选 A：原样纳入；备选 B：全部删除本地文件；备选 C：私有文档。 | 推荐减少内部路径暴露；原样风险高；删除本地文件越界；私有文档改变目标。 | 可能需要 redaction 计划。 | redaction 不足时排除。 |
| DQ-CP3-CR066-05 | implementation | CR066 是否允许任何 Git 写动作？ | 不允许；只输出策略和后续执行门禁输入。 | 备选 A：允许创建 clean branch；备选 B：允许 commit docs；备选 C：允许 remote add。 | 推荐遵守最小授权；备选均需 runtime_authorization。 | 当前不会推进远端。 | 用户明确授权时重发 CP5 / 新 CR。 |

不授权项：CP3 通过不授权 `git add`、`git commit`、`git remote add`、`git push`、`git tag`、branch creation / rename、history rewrite、force push、remote deletion、NAS/data lake/provider/catalog/runtime、凭据读取、QMT/MiniQMT runtime、CR046 recovery、physical move 或 bulk import rewrite。

auto_final_authorization: false

## Entry Criteria

| 条件 | 结果 |
|---|---|
| CR066 baseline 已创建 | PASS |
| strategy / audit / manifest 存在 | PASS |

## Checklist

| 检查项 | 结果 |
|---|---|
| 架构方案有备选和切换条件 | PASS |
| 推荐方案明确 | PASS |
| `.env.example` 策略明确 | PASS_WITH_RISK |
| tracked reports/runs 策略明确 | PASS_WITH_RISK |
| Git 写动作边界明确 | PASS |

## Exit Criteria

| 条件 | 结果 |
|---|---|
| 用户确认后可进入 CP5 readiness gate | PASS |

## Deliverables

- `process/checks/CP3-CR066-CLEAN-PUBLICATION-HLD-CONSISTENCY.md`
- `process/checkpoints/CP3-CR066-CLEAN-PUBLICATION-HLD-REVIEW.md`
- `process/context/CP3-CR066-CLEAN-PUBLICATION-DESIGN-CONTEXT.yaml`

## 人工审查结果

- 结论：approved
- 审查人：user
- 审查时间：2026-06-15T10:46:59+08:00
- 修改意见：N/A
- 风险接受项：接受 DQ-CP3-CR066-01..05 推荐方案；采用 clean snapshot / allowlist manifest 架构，`.env.example` 仅在 placeholder-only / no-secret scan PASS 后 conditional allow，`reports/**` / `runs/**` 默认排除，NAS / internal topology 文件默认不纳入。CP3 approved 不授权任何 Git 写动作。

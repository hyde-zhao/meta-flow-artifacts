---
status: ready-for-review
version: "1.0"
feature_id: "FEAT-WT-03"
feature_name: "Quality Governance"
source_blueprint: "process/docs/design/CR047-WORKFLOW-TRUTH-BLUEPRINT.md"
source_hld: "process/docs/design/CR047-WORKFLOW-TRUTH-HLD.md"
source_adr: "process/docs/design/CR047-WORKFLOW-TRUTH-ARCHITECTURE-DECISION.md"
source_matrix: "process/docs/design/CR047-FEATURE-DESIGN-MATRIX.md"
related_stories: ["ST-WT-003"]
lld_policy_summary: "full-lld=1"
---

# Feature Design: Quality Governance

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-14 | host-orchestrator-inline / meta-se | 定义 B0_pre/B0_cp7、lifecycle/read classification、append-only history 与 Doctor exit contract。 |

## 摘要与边界

| 项目 | 内容 |
|---|---|
| 目标 | active/default-required 超预算保持阻断，同时让 cold/legacy/unavailable 历史以可追溯 warning 收敛。 |
| 推荐方案 | 扩展现有 token/artifact、quality/workflow Doctor 与 policy，不创建第二套质量模型。 |
| 下游 Story | ST-WT-003 |
| 非目标 | 全局提高预算、截断历史 result、伪造 read/run/receipt。 |

## 上游依据与现有代码

| 来源/区域 | 路径 | 消费内容 / 当前职责 |
|---|---|---|
| ADR | `ADR-WT-003` | 动态分类锚、typed severity、历史不可变 |
| Budget | `meta_flow/checks/token_budget.py` | artifact/token scan 与阈值 |
| Quality | `meta_flow/checks/quality_governance.py` | Quality/Workflow Doctor |
| Context | `meta_flow/context_pack/read_expansion.py`, `builder.py` | read provenance 与 capsule |
| Policies | `process/policies/QUALITY-MODEL.yaml`, `READ-POLICY.json`, `LEDGER-RETENTION.yaml` | source、read/lifecycle/retention |
| Tests | `tests/test_token_budget.py`, `tests/test_quality_governance.py` | budget/doctor fixtures |

## 数据与接口

每个 finding 至少包含 `path,artifact_kind,bytes,limit,lifecycle_class,read_class,severity,reason,remediation_ref`。聚合输出包含：

```text
observed classified unclassified blocking_active warning cold
```

- `B0_pre.observed=21` 只存历史事实。
- CP7 入口采集 `B0_cp7`；终态对新增/删除/重分类逐项给出 delta reason。
- `must_read/default-read/current/package/security` 优先级高于 cold lifecycle，不能降为 warning。
- unknown classification、缺 remediation ref、Quality Model source 缺失或伪造 provenance 均阻断。

## 流程、失败与回退

1. 扫描 artifact 并解析 formal lifecycle、context read class、package/security precedence。
2. 分类 finding，校验 remediation ref 和 immutable hash/correction 语义。
3. 聚合 blocker/warning；只有 blocker=0 且 unclassified=0 返回成功。
4. Run ledger 对未来命令追加真实事件；历史只追加 recovered/legacy-unverified。

任何分类歧义保持 blocker。历史文件移动前先验证 ref graph/hash；失败则保留原位置并仅生成 compact summary/index。已追加 correction 不删除，只能再追加 superseding correction。

## 安全、测试与下游契约

- 不接受自报 receipt、actor、timestamp 或 token measurement。
- 覆盖 active overage、cold overage、unknown、missing remediation、B0 delta、Quality source、six legacy provenance、empty/future Run ledger。
- `TEST-PLAN.md` 区分 unit/fixture/doctor integration；`TASKS.md` 保持 policy→checker→migration/correction→regression 顺序。
- LLD 必须列明 22 当前 observed 只是 CP3 观测，不是固定验收数。

## 风险与 Gotchas

- Doctor exit 0 不等于 warning=0。
- “CR-047 自己生成”不是超预算豁免理由。
- archive/cold 是读取策略，不是删除许可。

# CR166 CP3 Human Gate Launch Message

## 审批者摘要

| 字段 | 内容 |
|---|---|
| 本次确认服务的整体目标 | 冻结 CR166 fixture/static C2 producer 架构，使后续实现能够生成可审计、leakage-safe、lineage-bound、deterministic 的 Walk-forward/OOS evidence。 |
| 推荐动作 | `approve`：批准四项 CP3 推荐决策。 |
| approve 后会发生什么 | 不拉起子 Agent；Host Orchestrator 以内联方式完成 CP4 Story/DAG/Feature design 和 CP5 全量设计证据准备，在 CP5 required gate 停止。 |
| approve 不授权什么 | 不授权源代码/测试实现、真实 fold/OOS 或历史重算、真实 lake/NAS/provider/凭据/外部框架/runtime、broker/trading、deploy/publish/Git remote write，也不启动 Stage 3。 |
| 不确认会阻塞什么 | CP4 Story planning、CP5 设计证据以及 CP6/CP7。 |

- 人工 checklist：`process/checkpoints/CP3-CR166-WALK-FORWARD-OOS-EVIDENCE-HLD-REVIEW.md`
- 自动预检结论：`PASS`，13/13 checks，blocker=0，waiver=0。
- 继承治理说明：仓库级 read-expansion checker 仍被 CR166 之前的 legacy entries 阻断；CR166 本轮记录无新增 ERROR，CP8 不得宣称该全局检查全绿。
- Context Capsule：`process/context/CP3-CR166-WALK-FORWARD-OOS-EVIDENCE-CONTEXT.yaml`，ready / read_profile=compact / capsule-first。
- 授权全文扩展：4 组，仅用于审计 CR151/154/155/164 相邻设计；均有 ledger 记录。
- 决策收集覆盖：STATE/route/CP2、discussion、五份 design docs、CP3 result 均已扫描；候选 4，纳入 4。
- 本轮待人工决策项：4 项，`DQ-CP3-CR166-001..004`。

## 决策分层

| 分类 | 数量 | 处理 |
|---|---:|---|
| 必须用户决策 | 4 | 全部列入下方决策表。 |
| 高风险策略确认 | 1 | DQ-004：zero-dereference、design-only 与 Stage ceiling。 |
| agent 默认处理 | 4 类 | 类名/文件 owner、metric defaults、golden values、error codes 留给 CP4/CP5。 |
| 仅审计记录 | 7 类 | 字段、负向、fixture、consumer、hash、扩展数和 Stage flags。 |

## 四项推荐决策

| 决策 ID | 类型 | 推荐方案 | 备选方案 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|
| DQ-CP3-CR166-001 | architecture | neutral primitives + C1 compatibility + stable envelope/static catalog | C2 依赖 C1；复制 canonical；dynamic registry | 公共 contract 与 C1 hash compatibility | 任一 C1 golden/API 不一致即暂停抽取 |
| DQ-CP3-CR166-002 | architecture | half-open folds + explicit purge/embargo + separate availability/outcome + declared denominator | 字符串检查；隐式 calendar；过滤坏 fold | 泄漏和 pass-rate 虚高风险 | 真实 calendar/session 另起 Stage3 设计 |
| DQ-CP3-CR166-003 | architecture | 3 existing projections；event explicit N/A | 新 gate；event 空壳；暂停 integration | consumer owner 与 event 假覆盖 | event semantics 独立冻结后新 CR |
| DQ-CP3-CR166-004 | security | 5 个 CP4 输入；zero-dereference/design-only/Stage ceiling | 修改或暂停 | 只解锁设计，不授权实现 | 新数据/runtime/external/write 需求立即停 |

如果你回复 approve，表示批准这四项推荐架构，并允许继续 inline 推进到 CP5；不表示授权实现或真实运行。不授权项包括：子 Agent、源代码/测试实现、真实 fold/OOS、真实数据、凭据、NAS/provider、外部 framework、runtime、broker/trading、deploy/publish 和 Git remote write。

请回复：

- `approve`
- `修改: <具体修改点>`
- `reject`

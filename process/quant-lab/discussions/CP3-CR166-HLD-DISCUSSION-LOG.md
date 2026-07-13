# CP3 CR166 HLD Discussion Log

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 0.1 | 2026-07-13 | host-orchestrator inline meta-se-critical | 基于 CP2 已批准范围、当前源码消费者与 CR151/154/155/164 设计基线，完成五个 Architecture Gray Areas 的 table-first 分析。 |

## 执行模式

- 用户指令：`approve，继续推进到下一个人工门禁`；既有指令要求不拉起子 Agent。
- 执行方式：用户批准的 `inline-fallback`；未启动、恢复或消息任何子 Agent。
- 模型声明：无 child model；不声称使用 `.codex/agents/meta-se-critical.toml` 指定模型。
- 设计边界：只推进至 CP3 required human gate，不形成正式 Story、LLD、代码或测试。

## Architecture Gray Areas

| Gray Area | 关键问题 | 推荐 | 备选 | 影响 / 切换条件 |
|---|---|---|---|---|
| AGA-166-01 | C2 是否直接依赖 C1 `statistical_evidence.py` 的 canonical/status primitives？ | 抽取 method-neutral `evidence_contracts` primitives；C1 通过兼容 wrapper/re-export 保持既有 API 与 hash 不变。 | C2 直接导入 C1；复制 canonical 逻辑。 | 推荐避免 C2→C1 错向依赖和双实现漂移；若兼容 golden hash 无法保持，则暂停抽取、先用受控 wrapper。 |
| AGA-166-02 | common envelope 如何为 C3/C4 留扩展点而不引入动态 runtime registry？ | stable header + 静态、版本化 component descriptor/registry；当前 active component 仅 C2，C3/C4 calculator=0。 | 单体 C2 schema；动态 plugin registry。 | 推荐满足向后兼容且不增加发现/安全面；第三方、多包扩展出现时再评审动态 registry。 |
| AGA-166-03 | fold 时间与 purge/embargo 如何校验？ | ISO-8601 半开区间 `[start,end)`；fold 内 `train < validation < OOS`；显式 declared/required/applied purge/embargo，禁止推断缺失事实。 | 只校验非空字符串；按日期差隐式推断。 | 推荐可审计、跨 daily/ML；真实 calendar/session 计算需 Stage 3 授权后独立设计。 |
| AGA-166-04 | event strategy 是否进入 CR166 producer？ | `not_applicable_with_reason:event_fold_semantics_unfrozen`；不交付 event fixture/producer 空壳。 | 复用日历 fold 假覆盖；把 event producer 升为 P0。 | event-time、window、可用时点与 overlap 语义独立冻结后，通过后续 CR 接入 common envelope。 |
| AGA-166-05 | producer 如何进入既有门禁？ | producer 只生成 evidence；三个薄 projection adapter 复用 CR151、CR154、StrategyAdmissionPackage，采用 worse-state merge，不新建 gate。 | 在 producer 内做 admission；新建 C2 gate。 | 推荐保持 policy owner 唯一；消费者契约无法无破坏扩展时回 CP3 修订 adapter。 |

## Advisor Table

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| A. Neutral envelope + pure producer + thin projections | 单向依赖、确定性强、3 consumers 可复用、C3/C4 可扩展 | 需兼容迁移 canonical primitives，并新增显式 adapter | contract/modules/tests/consumers | 推荐 | C1 API/hash compatibility 100%；若不成立则先保留 wrapper。 |
| B. 扩展 `WalkForwardValidationPlan` 原类 | 文件少、旧 consumer 就近 | consumer DTO 反向拥有 producer schema；缺 fold lineage/typed extension；ML/daily 耦合 | maintainability/compatibility | 不推荐 | 仅一次性补丁且无未来 C3/C4 扩展时才适用。 |
| C. Dynamic plugin registry + artifact store | 扩展能力最大 | 当前过度设计；引入 runtime discovery、版本、存储与权限面 | runtime/security/deploy | 暂缓 | 第三方 component 或跨包部署成为真实需求时另起 CR。 |

## 方案形成输入

| 来源 | 影响章节 | 处理结果 | 说明 |
|---|---|---|---|
| CP2 approved REQ/QAC/SCN | envelope、fold policy、event、authorization | adopted | 9 requirements、11 scenarios、12 QAC 不扩张。 |
| 当前源码消费者 | modules、projection、compatibility | adopted | `WalkForwardValidationPlan`、Gate 2 和 admission package 都保留 policy ownership。 |
| CR164 C1 envelope | shared primitive boundary | adopted-with-compatibility | 复用 canonical 语义，不让 C2 依赖 method-specific evidence。 |
| CR155 daily artifact | daily adapter/negative regression | adopted | 当前 split manifest 是输入兼容面，不等于 C2 producer；CR155 必须保持 blocked。 |
| HLD 后评审 | N/A | pending | CP3 人工意见只能作为成文后修订，不倒填为前置讨论。 |

## Deferred / switch conditions

- C3 economic cost 与 C4 capacity/liquidity：只保留 component versioning 契约；计算、输入合同和真实市场数据均为后续 CR。
- event-specific producer：事件时间、available-at、事件窗口和 overlap policy 冻结后再启用；CR166 只输出明确 N/A。
- real fold/OOS ingestion、calendar/session resolver、lake/provider/NAS：Stage 3 独立 CR 与授权后再进入。
- dynamic registry/artifact service：第三方 component、跨包部署或并发服务化需求出现时重开 ADR。

## 待 CP3 决策

`DQ-CP3-CR166-001..004` 全部进入正式 CP3 Decision Brief；当前无 blocking clarification。

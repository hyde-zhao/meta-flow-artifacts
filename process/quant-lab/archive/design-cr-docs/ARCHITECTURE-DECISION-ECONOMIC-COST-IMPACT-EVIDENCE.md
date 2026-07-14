---
status: approved-cp3
version: "1.2"
cr_id: "CR-168"
source_hld: "process/archive/design-cr-docs/HLD-ECONOMIC-COST-IMPACT-EVIDENCE.md"
---

# Architecture Decisions：Economic Cost / Slippage / Impact Evidence

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-14 | host-orchestrator inline meta-se-critical | 起草五项 CR168 CP3 ADR 候选，待人工批准。 |
| 1.1 | 2026-07-14 | host-orchestrator inline meta-se-critical | 吸收 CP3 深度评审：明确 component/envelope hash 输入域、数值/舍入/release profile、test double/reason code，并冻结 v1 与 C4 adapter 演进治理。 |
| 1.2 | 2026-07-14 | host-orchestrator inline fallback | 按 CR131 design-surface 规则归档；仅更新 archive route 元数据，ADR 正文不变。 |

## ADR-CR168-001：复用 neutral envelope，C3 作为叶子模块扩展

- 状态：Accepted，2026-07-14，DQ-CP3-CR168-ARCH。
- 背景：CR166 已提供稳定 header、canonical primitives 和 static component catalog。
- 决策：新增单一 C3 contract/producer 叶子模块，依赖 `engine.strategy_evidence` public API；将 `economic_cost@v1` 作为唯一 active schema；neutral 层不得反向 import C3。
- 备选：建立独立 C3 envelope/registry。
- 拒绝理由：会产生平行真相源，违反 QAC-CR168-09。
- 后果：C1/C2 golden compatibility 成为实现前置；C3 可独立回退，C4 保持 reserved。
- 切换条件：只有出现获批的跨包动态 discovery 需求才考虑项目级 registry，且须独立 CR。

## ADR-CR168-002：v1 只启用显式静态 square-root impact approximation

- 状态：Accepted，2026-07-14，DQ-CP3-CR168-METHOD。
- 决策：present C3 的 active impact family 精确为 `square_root`；所有参数显式提供并声明 synthetic/static lineage。`almgren_chriss`、`gatheral`、`custom` deferred；`n/a-with-reason` 只形成 typed_unavailable/blocked path，不进入 present Gate4 projection。
- 备选：一次支持 canonical Gate4 的全部 family。
- 选择理由：最小方法面仍能证明透明 impact contract，且不需要真实校准、custom policy 或多算法验证。
- 后果：v1 schema 简单、可重算；扩展 family 需要新版本/后续 CR，不可静默改变 v1。
- 版本治理：`economic_cost@v1` 批准后不可变；新增 active family、改变算术/舍入或启用 rebate 必须通过独立 method CR 发布新 schema version（默认 v2），不得在 v1 增加可选捷径。
- 切换条件：有明确真实方法需求、参数 lineage、独立验证和授权。

## ADR-CR168-003：冻结最小 C3/C4 shared header，不冻结 C4 方法语义

- 状态：Accepted，2026-07-14，DQ-CP3-CR168-HEADER（含 A1 hash-domain revision）。
- 决策：identity、unit/currency/calendar/notional basis、lineage/provenance/authorization 为 shared header；C3 拥有 gross/trade/cost/impact assumptions；C4 独占 ADV/capacity/liquidity/alpha-decay。九字段族均需校验，但 manifest/run/strategy/package identity 是 attachment identity，只进入 neutral envelope hash；字段族 2-9 的 subject-neutral 成本语义进入 component hash。
- 备选：C3 完全独立 header。
- 选择理由：避免 identity/basis/lineage 重复，又不预占 C4 calculator 或内部 schema。
- 后果：daily/ML 的成本语义可跨 package 比较且 identity tamper 仍由 envelope hash 阻断；未来 C4 可复用 stable header，但任何 trade summary 复用映射需 C4 CR 再冻结。
- 回退：若 CP4 发现 owner table 冲突或锁死 C4，则回到独立 C3 header 并重开 CP2。

## ADR-CR168-004：Gate4 风险采用 adapter-local containment

- 状态：Accepted，2026-07-14，DQ-CP3-CR168-GUARD。
- 背景：canonical Gate4 对 C4 ref missing 使用 `not _has_ref and not _has_na_reason`，因此 absent+reason 可能绕过 blocked claim。
- 决策：CR168 只有一个 projection adapter；从 typed component 新建四字段 payload；精确 8-key presence denylist；strict allowlist；reason escape 在 canonical 调用前拒绝且调用数 0；safe absent 固定 `release_profile="candidate-release"` 调用后断言 BLOCKED + 三个 C4 missing claims；PASS 与 postcondition 违规使用不同 reason code；SIM-05 仅以 public validator test double 模拟返回值；adapter 外 CR168 direct call 数 0；不运行时依赖 `_has_na_reason`。
- 备选 A：component-only，projection deferred。
- 备选 B：修改 canonical Gate4。
- 选择理由：局部满足 CP2 compatibility 与 fail-closed 目标，且不改变全局历史语义。
- 后果：CR168 能证明自己的入口安全，但不能宣称 canonical 对其他 caller 全局安全。
- 回退：局部 guard 无法保证时采用 component-only 并重开 CP2；不得自动采用 canonical 修改。

## ADR-CR168-005：Canonical 全局 hardening 与 aggregate integration 后置

- 状态：Accepted，2026-07-14，DQ-CP3-CR168-GLOBAL。
- 决策：canonical Gate4 全局 N/A 语义复核、C1-C4 aggregate orchestration 和 CR155 综合 regression 保留给 `FU-CR161-007`；任何未来绕过 CR168 adapter 的 direct caller 进入前必须触发重访。C4 calculator 继续归 `FU-CR161-005`；CR168 adapter 不预先设计 C4 present 路径，FU-005 负责决定扩展现有 adapter 还是新增 C4-owned projection，并提供迁移证据。
- 备选：CR168 同时全局修复 canonical 并接 aggregate。
- 拒绝理由：会改变 CR168 的目标、owner、回归面和授权边界。
- 后果：当前 canonical residual risk 明确存在但 dormant/contained；后续不得把 CR168 containment 当作豁免。

## ADR 一致性矩阵

| ADR | HLD | Blueprint | Domain | Dependency | REQ/SC |
|---|---|---|---|---|---|
| ADR-001 | §4/6 | CAP-CR168-01/03 | OBJ-04/05 | FD-01 | REQ-001/005 |
| ADR-002 | §5.3 | CAP-CR168-02 | NUM-04 | allowed producer dependency | REQ-003；N03/N04/N05 |
| ADR-003 | §5.1 | §4 owner table | OBJ-01/07 | CYCLE-03 | REQ-002/009 |
| ADR-004 | §7 | §6 invariants | §5/6 | FD-02/03/04 | REQ-006；B01/B02 |
| ADR-005 | §12 | DAI-01/02 | RULE-04 | FD-05/06 | REQ-009；G01 |

## Gotchas

- ADR-004 不是 canonical bug fix 的批准；它只批准 CR168 adapter 的 containment 设计。
- ADR-002 不允许把真实/历史估计参数包装成“static”绕过授权。
- ADR-003 的 shared header 是合同，不是共享 calculator 或共享数据读取层。
- 五项 ADR 已在吸收 A1–E3 评审修订后获用户批准；批准只解锁 CP4/CP5 设计证据，CP5 未批准前不授权代码或测试实现。

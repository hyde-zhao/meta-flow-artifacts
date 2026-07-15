---
status: "approved-cp3"
version: "1.1"
change: "CR-169"
source_story_map: "docs/product/STORY-MAP.md"
source_mvp_scope: "docs/product/MVP-SCOPE.md"
source_hld: "docs/design/HLD-CAPACITY-LIQUIDITY-ADV-EVIDENCE-PRODUCER.md"
confirmed_by: "user"
confirmed_at: "2026-07-14T18:48:03+08:00"
---

# C4 Capacity / Liquidity / ADV Evidence Producer Blueprint

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-14 | host-orchestrator inline meta-se-critical | CR-169 CP3 草案：C4 fixture producer、correlation boundary、strict joint adapter、Stage2 exit evidence 的 owner 与依赖边界。 |
| 1.1 | 2026-07-14 | host-orchestrator inline meta-se-critical | CP3 已批准五项推荐方案；correlation boundary 采用 13 字段 exact header，alpha-decay 后置 `FU-CR161-008`，Stage2 历史缺口不由 CR169 代修。 |

## 能力地图

| Capability ID | 能力域 | 用户价值 | 覆盖需求 / 场景 | Owner Feature |
|---|---|---|---|---|
| CAP-CR169-01 | C4 static evidence | 把显式 synthetic/static ADV、capacity、liquidity 假设变为可复算 typed C4 component | REQ169-001..003 / SC-CR169-P01,B03..B06 | FEAT-169-01 |
| CAP-CR169-02 | C3/C4 correlation | 仅让同一 declared context 的 C3 与 C4 evidence 安全组合 | REQ169-004 / SC-CR169-P02,B03,B04 | FEAT-169-02 |
| CAP-CR169-03 | Gate 4 fixture compatibility | 证明 7 字段静态 payload 可被 consumer 合同读取，而不产生 aggregate claim | REQ169-005 / SC-CR169-P03,B01,B02 | FEAT-169-03 |
| CAP-CR169-04 | Claim / exit governance | 防止 C4 fixture 误触发 Stage3、CR155 或 canonical 全局状态升级 | REQ169-006..009 / SC-CR169-E01 | FEAT-169-04 |

## Feature / Epic 边界

| Feature ID | 名称 | 负责事项 | 不负责事项 | 拥有数据 | 只读数据 | 禁止依赖 |
|---|---|---|---|---|---|---|
| FEAT-169-01 | C4 Contract & Producer | C4 input normalize/validate、static proxy calculation、`capacity_liquidity@v1`、semantic hash | 真实 ADV/liquidity、alpha calculator、C3、aggregate | `CapacityLiquidityInputV1`、`CapacityLiquidityEvidenceV1` | neutral envelope public catalog / serializer | provider/lake/NAS/runtime/broker、canonical source write |
| FEAT-169-02 | Correlation Boundary | 读取并比较 C3/C4 envelope binding 与 minimal shared header | 改写 C3/C4 component、创建 shared registry | `C3C4CorrelationDecision` | verified C3、verified C4、envelope binding | private Gate4 helper、identity-in-hash mutation |
| FEAT-169-03 | Strict Joint Adapter | exact 7-key mapping、public canonical call、本地 postcondition、fixture-only outcome | canonical Gate4 改写、C3-only adapter 改写、aggregate / admission | `Gate4FixtureCompatibilityOutcome` | C3/C4 verified components、canonical public result | `StrategyAdmissionPackage` write、C4 N/A reason pass-through |
| FEAT-169-04 | Claim & Exit Guard | claim ceiling / verification refs、Stage2 7/7 exit result、CR155 blocked regression | Stage3 authorization、FU007 startup、release / runtime | `Stage2ExitVerification` | historical contract refs、test outcomes | runtime authorization mutation、remote / catalog write |

## 跨 Feature 流程

| Flow ID | 触发 | 参与 Feature | 数据写入 Owner | 失败路径 | 验证入口 |
|---|---|---|---|---|---|
| FLOW-CR169-01 | explicit static C4 fixture | FEAT-169-01 -> neutral envelope | FEAT-169-01 | validation failure -> unavailable/BLOCKED；不得产生 component | SC-CR169-P01,B03..B06 |
| FLOW-CR169-02 | C3+C4 fixture compatibility request | FEAT-169-01 / CR168 C3 -> FEAT-169-02 -> FEAT-169-03 -> canonical read-only | FEAT-169-03 | header mismatch / reason escape -> BLOCKED/REJECTED，canonical call=0 | SC-CR169-P02,P03,B01,B02 |
| FLOW-CR169-03 | CP8 exit review | FEAT-169-04 -> stage exit result | FEAT-169-04 | any 1/7 evidence missing -> Stage2 completion claim false；历史 6 项回 CR157 / 新治理 CR，不扩大 CR169 | SC-CR169-E01 |

## 共享能力

| Shared ID | 名称 | 使用方 | Owner | 调用方向 | 降级策略 |
|---|---|---|---|---|---|
| SH-CR169-01 | Neutral evidence envelope / catalog | CR168 C3、FEAT-169-01、FEAT-169-02 | existing CR166 envelope feature | producer -> public catalog/serializer | contract incompatibility -> CP3 clarification，不建平行 envelope |
| SH-CR169-02 | Canonical Gate 4 public validator | FEAT-169-03 | existing reliability-gates module | joint adapter -> read-only public call | result unexpected -> local BLOCKED/REJECTED；不得修改 canonical |
| SH-CR169-03 | C3-only absent-C4 guard | CR168 adapter | CR168 | independent existing consumer path | C4 absent 继续 fail-closed；CR169 不修改 |

## 所有权与数据边界自检

| 检查项 | 结果 | 说明 |
|---|---|---|
| 每个新对象有唯一 owner | PASS | `CapacityLiquidityEvidence`、correlation decision、joint outcome、exit result 分别归 FEAT-169-01..04。 |
| shared header 未被重复拥有 | PASS | schema owner 为现有 neutral envelope；CR169 只读取和验证 occurrence。 |
| consumer 没有反向写 producer | PASS | FEAT-169-03 不修改 C3/C4 evidence；只产生自己的 outcome。 |
| aggregate / Stage3 不被隐式持有 | PASS | FEAT-169-03/04 都禁止写 admission、runtime 或 authorization。 |

## 待确认边界

| Decision ID | 决策类型 | 问题 | 推荐方案 | 备选方案 | 推荐 / 备选优劣 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP3-CR169-METHOD | architecture | C4 是 static proxy 还是 schema-only？ | explicit fixture/static proxy | schema-only | 推荐能验证真实的计算/consumer 合同；备选更小但价值不足 | proxy 不得误称真实 capacity | 无法定义审计限制时降级，回 CP3 |
| DQ-CP3-CR169-HEADER | architecture | identity 应否进入 component hash？ | identity 仅 envelope binding；join 前 exact check | identity 进入 component hash | 推荐分离方法 hash 与 subject binding | header 规则不清会导致 cross-context join | envelope 不支持则回 CP3 |
| DQ-CP3-CR169-JOINT | architecture | C4 consumer contract 谁负责？ | CR169 strict joint adapter | FU007 再实现 | 推荐有 consumer evidence 且不做 aggregate | adapter 误用会形成虚假 readiness | public call 不可安全隔离时延后 |
| DQ-CP3-CR169-ALPHA | scope | alpha-decay 是否进 C4 v1？ | 0 calculator，独立 / C2-adjacent follow-up | C4 v1 calculator | 推荐不预占 C2/OOS 语义 | 后续可能有 one-time integration cost | 新 CR 证明 C4 owner 后再变更 |
| DQ-CP3-CR169-TRANSITION | follow_up_tracking | Stage2 completion 是否可自动推出 Stage3 ready？ | 否；7/7 CP8 核验；FU007a/b 仅提案 | 自动推进 | 推荐保持授权与声明边界 | 未核验会错误过渡 | 仅独立 transition CR 可改变 |

## Gotchas

1. C4 对象的 refs 是 typed present evidence，不是缺字段时塞入 reason 的容器。
2. `Gate4FixtureCompatibilityOutcome` 是独立 owner，不能被当作 canonical result 的替代或写回 admission package。
3. `Stage2ExitVerification` 是 CP8 治理证据，不是 C4 producer 的运行时 API。

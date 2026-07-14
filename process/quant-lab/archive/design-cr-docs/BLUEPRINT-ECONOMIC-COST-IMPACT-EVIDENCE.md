---
status: approved-cp3
version: "1.2"
cr_id: "CR-168"
source_story_map: "docs/product/STORY-MAP.md"
source_mvp_scope: "docs/product/MVP-SCOPE.md"
source_requirements: "docs/product/REQUIREMENTS.md"
source_hld: "process/archive/design-cr-docs/HLD-ECONOMIC-COST-IMPACT-EVIDENCE.md"
confirmed_by: "user"
confirmed_at: "2026-07-14T10:57:44+08:00"
---

# Blueprint：Economic Cost / Slippage / Impact Evidence

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-14 | host-orchestrator inline meta-se-critical | 建立 CR168 companion blueprint；冻结 C3、neutral envelope、Gate 4 projection adapter 的能力边界，并把 canonical Gate 4 全局硬化留给后续治理。 |
| 1.1 | 2026-07-14 | host-orchestrator inline meta-se-critical | 吸收 CP3 深度评审：冻结 component/envelope 两级 identity 绑定、candidate release profile、数值边界与 C4/schema 演进 owner。 |
| 1.2 | 2026-07-14 | host-orchestrator inline fallback | 按 CR131 design-surface 规则归档；仅更新 archive route 元数据，能力边界和 owner 不变。 |

## 1. 定位

本蓝图把 CR168 限定为 Stage 2 已完成后的 fixture/static C3 桥接增强。它不重开 Stage 2，不启动 Stage 3，也不创建新的 admission gate。核心价值是把显式静态成本假设变成可审计、确定性、fail-closed 的 `economic_cost@v1` component，并以一个受控 adapter 验证其对 C3+C4 联合 Gate 4 的兼容性。

## 2. 能力地图

| Capability ID | 能力域 | 用户价值 | 精确边界 | Owner |
|---|---|---|---|---|
| CAP-CR168-01 | C3 typed input contract | 9/9 字段族具备 required/optional/N/A/auth 规则 | 只接收显式 synthetic/static 输入；真实数据读取 0 | FEAT-168-01 |
| CAP-CR168-02 | Transparent cost computation | 可重算 fee/tax/spread/slippage/impact/total/gross-to-net | v1 active impact family 仅 `square_root`；真实 TCA/calibration 0 | FEAT-168-01 |
| CAP-CR168-03 | Typed evidence and identity | 单一 `economic_cost@v1` component 可进入 CR166 neutral envelope | component/schema 1/1；平行 envelope/registry/gate 0 | FEAT-168-01 + FEAT-166-01 |
| CAP-CR168-04 | Joint Gate 4 compatibility | C3 present、C4 unavailable 时确定性 fail-closed | projection 1；capacity/aggregate PASS 0；canonical/aggregate 修改 0 | FEAT-168-02 |
| CAP-CR168-05 | Fixture compatibility | daily 与 ML package 使用相同 C3 算术合同 | fixture 族 2/2；event producer 0 | 后续验证 Story，当前只冻结合同 |

## 3. Feature / Epic 边界

| Feature ID | 名称 | 负责事项 | 不负责事项 | 拥有对象 | 只读对象 | 禁止依赖 |
|---|---|---|---|---|---|---|
| FEAT-168-01 | Economic Cost Evidence | input/assumption schema、normalization、验证、静态计算、component semantic hash、typed C3 outcome | C4、真实 TCA、数据加载、admission policy、aggregate | EconomicCostEvidenceInput、CostAssumptionSet、CostBreakdown、EconomicCostEvidenceV1 | CR166 public canonical primitives | provider/lake/NAS/runtime/broker；Gate4 私有 helper |
| FEAT-166-01 | Strategy Evidence Envelope（既有） | neutral header、descriptor、catalog、inventory/envelope hash | C3 计算、Gate4 policy、动态 registry | StrategyEvidenceEnvelope、ComponentDescriptor、catalog | economic_cost descriptor | 反向 import C3 producer；外部 discovery |
| FEAT-168-02 | Gate 4 C3 Projection Adapter | typed component 到 Gate4 flat payload 的严格转换、8-key guard、pre/post fail-closed | canonical Gate4 规则修改、C4 calculator、aggregate orchestration | C3Gate4ProjectionOutcome | EconomicCostEvidenceV1、canonical Gate4 public validator | `_has_na_reason` 等私有 helper；任意 mapping 透传；StrategyAdmissionPackage 修改 |

## 4. C3/C4 共享与独占 Owner Table

| 字段族 | 共享/独占 | 当前 owner | CR168 行为 | C4 后续约束 |
|---|---|---|---|---|
| manifest/run/strategy/package identity | shared attachment header | neutral envelope | 必填并参与 envelope identity；不进入 subject-neutral component semantic hash | 复用，不重命名同义字段 |
| unit/currency/calendar/price/notional basis | shared header | neutral contract + C3 | C3 冻结规范化语义；只允许显式 conversion | C4 可复用 basis，但不得改变 C3 算术 |
| lineage/provenance/authorization refs | shared header | neutral envelope | refs-only、零解引用 | C4 复用 refs 语义 |
| gross/pre-cost performance basis | C3 exclusive | FEAT-168-01 | 计算 gross-to-net | 不作为 C4 必填 |
| trade/position-change/turnover/notional summary | C3-owned, C4-readable future input | FEAT-168-01 | 仅静态摘要，不推导 ADV/capacity | C4 若复用须另起 CR 冻结映射 |
| fee/tax/spread/slippage/impact assumptions | C3 exclusive | FEAT-168-01 | static-only | C4 不覆盖或回写 |
| `adv_participation_ref` / `capacity_dollars_ref` / `liquidity_sizing_refs` / alpha decay | C4 exclusive | FU-CR161-005 | absent、calculator 0 | 由 C4 CR 独立冻结，不受占位实现细节锁死 |

## 5. 跨 Feature 流程

| Flow ID | 触发 | 调用方向 | 数据写入 Owner | 失败路径 | 验证入口 |
|---|---|---|---|---|---|
| FLOW-CR168-01 | 提供显式 fixture/static C3 输入 | caller → FEAT-168-01 normalize/validate/compute | FEAT-168-01 immutable value | 缺失/冲突/越权分别 typed_unavailable 或 blocked | SC-CR168-N01..N10 |
| FLOW-CR168-02 | C3 component 构建完成 | FEAT-168-01 → FEAT-166-01 public envelope API | FEAT-166-01 envelope | catalog/schema/hash mismatch blocked | SC-CR168-P01/P02/N10 |
| FLOW-CR168-03 | 兼容性验证请求 | FEAT-168-02 → canonical `validate_gate4_capacity_impact` | FEAT-168-02 projection outcome | C3 非 present、C4 present/out-of-scope、8-key escape 均在调用前 blocked；unexpected PASS 在调用后 blocked | SC-CR168-B01/B02 |
| FLOW-CR168-04 | 后续 aggregate admission | 不在 CR168 执行 | FU-CR161-007 | 当前 aggregate 调用数 0 | QAC-CR168-08/13 |

## 6. Projection 安全不变量

1. CR168 新增 Gate4 调用 `100%` 经过 FEAT-168-02；adapter 外直接调用数 `0`。
2. adapter 从 typed component 新建 payload，只允许四个 C3 key：`impact_model_family`、`impact_model_ref`、`cost_underestimation_status`、`no_real_tca_claim`；不透传任意上游 mapping。
3. 以下精确 `8/8` key 只要出现即拒绝，与值是否为空无关：三组 C4 field 的 `_na_reason` / `_n_a_reason`，以及通用 `na_reason` / `n_a_reason`。
4. reason escape 路径 canonical 调用数 `0`；不得运行时依赖 `_has_na_reason`。
5. safe absent 路径固定 `release_profile="candidate-release"` 调用 canonical Gate4 后必须得到 BLOCKED 且包含三个 C4 missing claims；意外 PASS 转为 `gate4_unexpected_pass`，非 PASS 但 claims 不完整转为 `gate4_postcondition_violation`，均停止且不进入 aggregate。
6. 这些不变量只证明 CR168 adapter 路径，不证明 canonical Gate4 对其他 caller 已全局安全。

## 7. 共享能力

| Shared ID | 名称 | 使用方 | Owner | 调用方向 | 降级策略 |
|---|---|---|---|---|---|
| SH-CR168-01 | Neutral canonical/envelope | FEAT-168-01/02、未来 C4 | FEAT-166-01 | C3/C4 → public API | C1/C2 golden 不一致即停止，不更新 golden 掩盖回归 |
| SH-CR168-02 | C3/C4 stable header | C3、未来 C4 | neutral envelope | component → envelope | owner table 不清晰则回退 C3 独立 header 并重开 CP2 |
| SH-CR168-03 | Canonical Gate4 validator | projection adapter、未来 aggregate | FEAT-15 | adapter → public validator | CR168 只做 guarded call；全局语义问题进入 FU-CR161-007 |

## 8. CP3 待确认决策

| Decision ID | 类型 | 推荐方案 | 备选方案 | 切换条件 |
|---|---|---|---|---|
| DQ-CP3-CR168-ARCH | architecture | 单一 C3 module + 既有 neutral envelope + 独立 guarded projection adapter | 只输出 component、projection 全延后 FU-007 | adapter 无法在不改 canonical 的情况下满足 B01/B02 时回退备选并重开 CP2 |
| DQ-CP3-CR168-METHOD | architecture | v1 active impact family 只支持显式静态 `square_root`；其他 family deferred；impact N/A 不进入 present projection | v1 同时支持 canonical 全部 family | 真实方法需求、可靠校准或额外 metadata 获独立授权时扩展版本 |
| DQ-CP3-CR168-HEADER | architecture | 冻结最小 shared header + exclusive owner table；identity 由 envelope hash 绑定，字段族 2-9 的 subject-neutral 成本语义由 component hash 绑定 | C3 完全独立 header | owner/hash 域冲突或会锁死 C4 语义时切换并重开 CP2 |
| DQ-CP3-CR168-GUARD | security | adapter strict allowlist + exact 8-key presence denylist + pre/post guard + adapter-only call surface | 修改 canonical Gate4 | 当前 CR 禁止备选；若局部 guard 不足，回退 component-only，不隐式扩大范围 |
| DQ-CP3-CR168-GLOBAL | follow_up_tracking | canonical 全局 N/A 语义复核留给 FU-CR161-007；新增 direct caller 前强制重访 | 立即另起独立 Gate4 remediation CR | 在 CR168 之外出现直接 caller 或安全审计要求全局保证时启动独立 CR |

## 9. Gotchas

- `Stage2 complete=true` 是 CR168 之前已成立的阶段事实，不是本 CR 的交付 claim。
- `typed_unavailable` 不等于 N/A key 可以送入 Gate4；CR168 的安全表示是 C4 refs 完全 absent。
- `impact_reference_notional` 是显式静态参数，不得命名或解释为真实 ADV/capacity。
- component semantic hash 相同不要求不同 subject/provenance 的完整 envelope hash 相同。
- `participation_proxy > 1` 合法但不是 capacity/ADV claim；非有限或负值才是数值违规。
- 本蓝图已由用户批准进入 CP4/CP5 设计证据阶段；CP5 未批准前仍未授权代码或测试实现。

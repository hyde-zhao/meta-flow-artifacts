---
status: approved-cp3
version: "1.1"
cr_id: "CR-168"
source_blueprint: "process/archive/design-cr-docs/BLUEPRINT-ECONOMIC-COST-IMPACT-EVIDENCE.md"
source_hld: "process/archive/design-cr-docs/HLD-ECONOMIC-COST-IMPACT-EVIDENCE.md"
---

# Dependency Map：Economic Cost / Slippage / Impact Evidence

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-14 | host-orchestrator inline meta-se-critical | 冻结 C3 producer、neutral envelope、Gate4 adapter/canonical validator 的单向依赖和禁止依赖。 |
| 1.1 | 2026-07-14 | host-orchestrator inline fallback | 按 CR131 design-surface 规则归档；仅更新 archive route 元数据，依赖规则不变。 |

## 1. 允许依赖

| From | To | 类型 | 允许方向 | 原因 | 验证 |
|---|---|---|---|---|---|
| `engine.economic_cost_evidence`（候选） | `engine.strategy_evidence` public API | import / value contract | allowed | 复用 canonical、availability、descriptor、envelope | import graph + C1/C2 golden regression |
| Gate4 projection adapter（候选与 C3 同模块或独立小模块，CP4 定文件） | EconomicCostEvidenceV1 | typed read | allowed | 只接受 validated present component，不接受任意 mapping | type/contract tests |
| Gate4 projection adapter | `validate_gate4_capacity_impact` public function | call | allowed | 验证现有 joint Gate4 compatibility | SC-CR168-B01/B02 |
| daily/ML fixture adapters | EconomicCostEvidenceInput | construct | allowed | strategy type 只影响 attach point，不改变成本语义 | 2/2 fixture |
| neutral envelope builder | EconomicCostComponentDescriptor | value read | allowed | component 进入现有 envelope inventory | catalog/envelope tests |

## 2. 禁止依赖

| ID | From | To | 禁止原因 | 替代路径 | 违反风险 |
|---|---|---|---|---|---|
| FD-CR168-01 | `engine.strategy_evidence` | `engine.economic_cost_evidence` | neutral 层不得反向依赖具体 C3 方法 | catalog 只保存 type/version/status 常量 | 循环依赖、C2 回归 |
| FD-CR168-02 | CR168 adapter | `_has_na_reason` 或其他 Gate4 私有 helper | 私有实现不是稳定 contract，且会把 permissive 语义带入 adapter | 自有精确 8-key constant + strict allowlist | reason escape、升级脆弱性 |
| FD-CR168-03 | CR168 adapter | 任意上游 mapping passthrough | 无法证明 payload 无通用/C4 reason key | 从 typed component 新建四字段 payload | 虚假 Gate4 PASS |
| FD-CR168-04 | canonical Gate4 | CR168 producer/adapter | canonical validator 保持通用且本 CR 修改数 0 | 单向 adapter 调用 public validator | 全局语义耦合、范围扩大 |
| FD-CR168-05 | CR168 | C4/ADV/liquidity/capacity/alpha-decay calculator | owner 属于 FU-CR161-005 | C4 marker 只表达 unavailable | 偷渡 C4 |
| FD-CR168-06 | CR168 | StrategyAdmissionPackage aggregate orchestration | owner 属于 FU-CR161-007 | 仅返回 standalone projection outcome | C3 单独提升 admission |
| FD-CR168-07 | CR168 | lake/NAS/provider/credential/runtime/broker | 未授权且 fixture/static-only | 只消费调用者显式值与 refs，不解引用 | 安全与 claim 越界 |
| FD-CR168-08 | CR168 | dynamic capability/component registry | 平行 registry=0；当前 capability registry 缺失不构成创建授权 | 既有 feature/module refs + 独立治理 follow-up | 重复真相源 |

## 3. 候选文件影响面（非 Story/file ownership）

| 文件 | CP3 允许的设计判断 | CP3 不授权的动作 |
|---|---|---|
| `engine/strategy_evidence.py` | catalog 将 `economic_cost@v1` 设 active，保留 `capacity_liquidity@reserved` | 当前不得改代码 |
| `engine/economic_cost_evidence.py` | 候选新模块，拥有 C3 contracts/compute/projection guard | 当前不得创建实现文件 |
| `engine/cross_strategy_reliability_gates.py` | 只调用 public validator；canonical source changes=0 | 不改 validator、私有 helper或 aggregate |
| `tests/` | 后续覆盖 10/10、2/2、10→1、B01/B02 | 当前不得创建测试 |
| `docs/features/strategy-evidence-envelope/DESIGN.md` | CP3 后在 implementation-design 阶段增量更新 | 当前不提前冻结 Feature TASKS/LLD |

## 4. 循环风险

| Cycle ID | 涉及对象 | 风险 | 当前处理 |
|---|---|---|---|
| CYCLE-CR168-01 | neutral envelope ↔ C3 producer | neutral 层反向 import 具体 component | 通过静态 catalog descriptor 消除；只允许 C3→neutral |
| CYCLE-CR168-02 | C3 adapter ↔ canonical Gate4 | canonical 为满足 CR168 反向依赖 adapter | canonical changes=0；只允许 adapter→validator |
| CYCLE-CR168-03 | C3 ↔ C4 shared input | C3 owner table锁死 C4 方法或 C4 回写 C3 | 只冻结最小 header；C4-exclusive shape 保持 reserved |

## 5. Gate4 调用约束

```text
validated EconomicCostEvidenceV1
        |
        v
CR168 projection adapter
  - typed input only
  - 8-key presence denylist
  - strict four-key payload builder
  - C4 refs absent
        |
        v
public validate_gate4_capacity_impact
        |
        v
postcondition: C4 unavailable => status != PASS
```

CR168 新增 call sites 精确为 `1` 个 adapter；adapter 外直接 call sites 精确为 `0`。未来任何 direct caller 或 aggregate path 必须先重访 `FU-CR161-007` 的 canonical N/A 语义决策。

## 6. 回退策略与 Gotchas

- 若无法保证 canonical 调用前 payload 纯净或调用后非 PASS，则删除/不实施 projection slice，保留 component-only 设计并重开 CP2；不得通过修改 canonical 偷渡范围。
- 若 `engine.economic_cost_evidence.py` 与现有模块产生循环，CP4 可把 projection adapter 拆到单独叶子模块，但不能新增公共 runtime/registry 层。
- Gotcha：测试可以描述当前 canonical 行为作为审计证据，但不能建立“permissive 行为必须保持”的长期 golden。
- Gotcha：`capacity_or_aggregate_pass=0` 是 CR168 端到端安全结果，不表示 Gate4 的所有其他 caller 已满足该不变量。

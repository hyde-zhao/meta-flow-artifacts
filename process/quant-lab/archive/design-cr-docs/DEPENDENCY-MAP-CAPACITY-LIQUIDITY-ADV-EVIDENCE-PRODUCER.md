---
status: "approved-cp3"
version: "1.1"
change: "CR-169"
source_blueprint: "docs/design/BLUEPRINT-CAPACITY-LIQUIDITY-ADV-EVIDENCE-PRODUCER.md"
confirmed_by: "user"
confirmed_at: "2026-07-14T18:48:03+08:00"
---

# C4 Capacity / Liquidity / ADV Evidence Producer Dependency Map

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-14 | host-orchestrator inline meta-se-critical | CR-169 CP3 草案：限定 C4 producer、neutral envelope、CR168 C3、canonical Gate4 的单向依赖，并列出禁止关系。 |
| 1.1 | 2026-07-14 | host-orchestrator inline meta-se-critical | CP3 已批准；公开依赖锁定 `validate_gate4_capacity_impact`、alpha-decay 路由 `FU-CR161-008`，Stage2 历史缺口只路由 CR157 / 新治理 CR。 |

## 依赖关系

| From | To | 依赖类型 | 允许方向 | 原因 | 验证 / 监控 |
|---|---|---|---|---|---|
| FEAT-169-01 C4 Producer | existing neutral envelope public catalog / serializer | read / public API | allowed | 复用 typed component catalog 与 canonical serialization | contract / import test |
| FEAT-169-02 Correlation Boundary | CR168 C3 evidence + CR169 C4 evidence | read | allowed | 验证两份 verified evidence 的 minimal header | SC-CR169-P02,B03,B04 |
| FEAT-169-03 Joint Adapter | FEAT-169-02 correlation decision | read | allowed | 只有 matched pair 才能映射 Gate4 | adapter call-count test |
| FEAT-169-03 Joint Adapter | `engine.cross_strategy_reliability_gates.validate_gate4_capacity_impact` | read / public callable | allowed | 固定 `release_profile="candidate-release"` 复用既有验证语义，不改写实现 | public-call / diff guard |
| FEAT-169-04 Claim Guard | C4 / joint verification evidence | read / process result | allowed | 汇总 bounded claims 和 CP8 exit proof | CP8 result check |
| CR168 C3-only adapter | canonical Gate4 public validator | existing read-only | allowed, unchanged | C4 absent safety regression 独立存在 | SC-CR169-B01,B02 |

## 禁止依赖

| Forbidden ID | From | To | 禁止原因 | 替代路径 | 违反风险 |
|---|---|---|---|---|---|
| FD-CR169-01 | FEAT-169-01 | provider/lake/NAS/real ADV/liquidity source | CR169 只允许 fixture/static input | explicit synthetic/static fixture | 未授权真实数据与错误 claim |
| FD-CR169-02 | FEAT-169-01 / 03 | runtime / broker / QMT / trading | evidence foundation 不执行操作 | no-runtime fixture validation | 未授权操作 |
| FD-CR169-03 | FEAT-169-03 | `engine.cross_strategy_reliability_gates` source write | canonical global hardening 不在 CR169 范围 | future FU-007a CR | 影响未知全局消费者 |
| FD-CR169-04 | FEAT-169-03 | canonical private helpers（含 `_has_na_reason`） | private semantics 不是稳定集成契约 | public validator + local denylist / postcondition | 隐式耦合与行为漂移 |
| FD-CR169-05 | FEAT-169-03 | CR168 C3-only adapter source write | absent-C4 safety regression 必须保持原样 | 新的独立 joint adapter | 破坏 C3-only fail-closed |
| FD-CR169-06 | FEAT-169-03 | StrategyAdmissionPackage / aggregate orchestration write | FU-007 才拥有全链路聚合 | fixture outcome 独立保存 | fixture PASS 被误升格 |
| FD-CR169-07 | FEAT-169-01 | alpha-decay calculator | CP3 已批准不纳入 C4 v1 | `FU-CR161-008` independent / C2-adjacent follow-up | C2/C4 语义重叠 |
| FD-CR169-08 | FEAT-169-04 | Stage3 authorization / FU007 CR creation | 仅可记录后续候选 | formal CR + CP0 + explicit authorization | 隐式阶段启动 |

## 循环风险

| Cycle ID | 涉及对象 | 风险 | 当前处理 |
|---|---|---|---|
| CYCLE-CR169-01 | C4 producer -> joint adapter -> C4 producer | consumer failure反向要求 producer 填充 N/A / extra key | eliminated：adapter 只读 evidence，producer 绝不接收 consumer补写。 |
| CYCLE-CR169-02 | canonical Gate4 -> CR169 adapter -> canonical source modification | 为适配 fixture 而改写 global validator | eliminated：public read-only call + FD-CR169-03。 |
| CYCLE-CR169-03 | Stage2 exit result -> Stage3 authorization | exit result 被解释为授权 | eliminated：`stage3_entry_ready=false`，authorization 必须独立 CR。 |
| CYCLE-CR169-04 | C4 alpha input -> C2 OOS semantics -> C4 calculator | 为统一而双向预占 alpha-decay | eliminated：ADR-004 将 alpha v1 calculator 置 0。 |

## 依赖自检

| 检查项 | 结果 | 证据 |
|---|---|---|
| 所有调用方向单向且包含失败降级 | PASS | 依赖关系 + FD-CR169-01..08 |
| public / private Gate4 边界明确 | PASS | FD-CR169-03/04 |
| C3-only 与 C3+C4 路径无写冲突 | PASS | FD-CR169-05 / CYCLE-01 |
| 不新增 runtime / real-data 授权 | PASS | FD-CR169-01/02/08 |

## Gotchas

1. canonical validator 的 public callable 是依赖边界，不代表其 PASS 可直接提升为业务 admission。
2. C4 static producer 不能把“读取 C3 evidence”反向变成 C3 依赖 C4 的循环；correlation 是只读 consumer。
3. FU-007a/b 的记录不是依赖边，不应放进 CR169 的实现 DAG。

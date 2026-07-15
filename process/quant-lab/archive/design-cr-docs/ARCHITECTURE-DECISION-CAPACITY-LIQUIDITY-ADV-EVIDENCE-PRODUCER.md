---
status: "accepted-cp3"
version: "1.1"
change: "CR-169"
source_hld: "docs/design/HLD-CAPACITY-LIQUIDITY-ADV-EVIDENCE-PRODUCER.md"
confirmed_by: "user"
confirmed_at: "2026-07-14T18:48:03+08:00"
---

# CR-169 C4 Capacity / Liquidity / ADV Evidence — Architecture Decisions

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-14 | host-orchestrator inline meta-se-critical | CP3 proposed ADR：C4 fixture 方法、hash / correlation 分域、strict joint adapter、alpha-decay owner、Stage2→3 claim、FU-007 拆分提案。 |
| 1.1 | 2026-07-14 | host-orchestrator inline meta-se-critical | 回填 CP3 五项批准；冻结 correlation header 13 字段、Gate 4 public callable、Stage2 7/7 失败分流和 `FU-CR161-008` alpha owner 评估候选。 |

## ADR-CR169-001：C4 v1 使用显式 fixture/static proxy

| 项 | 内容 |
|---|---|
| 状态 | Accepted，DQ-CP3-CR169-METHOD，2026-07-14 |
| 背景 | C4 需要可复算 contract，但真实 ADV / liquidity / capacity data 不在 Stage 2 授权内。 |
| 决策 | 只接受显式 synthetic/static basis、模型版本、participation cap 与 limitations，生成 `capacity_liquidity@v1`；无真实数据读取 / 校准。 |
| 备选 | schema-only，无 producer。 |
| 后果 | 能验证算术、hash 与 consumer contract，但不能代表真实 capacity。 |
| 切换条件 | 若无法定义可审计 static proxy，则降级 schema-only 并以新 CP3 / CR 重新确认。 |

## ADR-CR169-002：component hash 与 correlation header 分域

| 项 | 内容 |
|---|---|
| 状态 | Accepted，DQ-CP3-CR169-HEADER，2026-07-14 |
| 背景 | C3/C4 既要验证相同 declared context 才能 join，又不能让 package identity 污染方法算术的 semantic hash。 |
| 决策 | C4 component semantic hash 仅覆盖 normalized computational C4 body、unit、model version 与 static limitations；attachment identity 只写入 envelope binding。`C3C4CorrelationHeaderV1` 精确含 13 字段：`manifest_ref`、`run_ref`、`strategy_ref`、`package_ref`、`price_basis`、`notional_basis`、`currency`、`calendar`、`as_of`、`horizon_start`、`horizon_end`、`lineage_context_ref`、`authorization_context_ref`。joint adapter 必须逐字段 exact match；不创建独立 header hash 域。 |
| 备选 | identity 进入 component hash。 |
| 后果 | 可审计地分离“方法相同”与“同一 subject 可 join”；不同 subject 不因 hash 相同自动可组合。basis / currency / calendar / temporal fields 仍分别进入 component normalized semantic body，组件专属 provenance refs 独立校验，不要求彼此相等。 |
| 切换条件 | 若 neutral envelope 不支持 binding 比对，只能回到 CP3 设计兼容扩展，不得自建 parallel registry。 |

## ADR-CR169-003：joint adapter 在 CR-169 局部实现，canonical Gate 4 只读调用

| 项 | 内容 |
|---|---|
| 状态 | Accepted，DQ-CP3-CR169-JOINT，2026-07-14 |
| 背景 | C4 producer 应有 consumer-side compatibility evidence；canonical Gate 4 的全局 N/A 语义未在本 CR 获得全局审计授权。 |
| 决策 | 新建 strict joint adapter，只构造 C3 4 字段与 C4 3 refs 的 exact 7-key payload；拒绝所有 N/A reason / extra keys；调用公开 `engine.cross_strategy_reliability_gates.validate_gate4_capacity_impact`，固定 `release_profile="candidate-release"`，再执行 local postcondition。唯一 positive outcome 为 `gate4_fixture_contract_pass`，不写 aggregate/admission。精确 Python 签名、依赖注入与 callable double 在 CP5 LLD 冻结。 |
| 备选 | 推迟所有 C4 consumer 验证至 FU-007。 |
| 后果 | C4 有可消费合同证据，同时 containment 不改变 global canonical / CR-168 adapter。 |
| 切换条件 | 若 public callable 无法隔离调用，停止实现并回到 CP3；不得依赖私有 helper。 |

## ADR-CR169-004：alpha-decay 不进入 C4 v1

| 项 | 内容 |
|---|---|
| 状态 | Accepted，DQ-CP3-CR169-ALPHA，2026-07-14 |
| 背景 | alpha-decay 主要是预测能力的时间衰减，靠近 C2/OOS，且并非 canonical Gate 4 的 C4 字段。 |
| 决策 | C4 v1 `alpha_decay_calculator=0`；alpha-decay 登记为 `FU-CR161-008`，供独立 / C2-adjacent owner 与方法边界评估；本 CR 不创建其 formal CR。 |
| 备选 | 在 C4 v1 增加 alpha decay static calculator。 |
| 后果 | 保持 C4 输入合同聚焦 capacity/liquidity/ADV，避免 C2 语义和测试边界被预占。 |
| 切换条件 | 如后续 architecture CR 证明其必须共享 C4 owner 和输入，需新 CR 明示 schema 版本、测试与边界。 |

## ADR-CR169-005：Stage 2 完成与 Stage 3 entry readiness 分离

| 项 | 内容 |
|---|---|
| 状态 | Accepted，DQ-CP3-CR169-TRANSITION，2026-07-14 |
| 背景 | evidence index 可能在 CR-169 后补齐，但 Stage 3 还需要 canonical hardening、verifier independence 与运行授权。 |
| 决策 | claim ceiling 固定 `stage3_entry_ready=false`。CP8 必须产出 `STAGE2-EXIT-VERIFICATION.result.json`，对 FactorSpec、FactorRunSpec、factor panel、label window、evaluation、portfolio/risk、admission package/evidence index 7/7 逐项给出 `PASS/FAIL/BLOCKED` 和 evidence ref。CR-169 的本地 C4 交付只对 evidence-index/C4 条目负责；历史 6 项有缺口时回 CR-157 或新治理 CR，不扩大或阻塞 CR-169 本地交付，但 `stage2_complete` / Stage 2 exit claim 必须保持 false。 |
| 备选 | 以 `stage2_complete=true` 隐含 Stage 3 ready。 |
| 后果 | 退出合同可审计，且无法因 C4 fixture success 自动启动 Stage 3；历史治理缺口不会被迫塞进 C4 Story，但也不能被 claim ceiling 掩盖。 |
| 切换条件 | 只有独立 Stage 3 authorization / transition CR 能将 `stage3_entry_ready` 改为 true。 |

## ADR-CR169-006：FU-007a/007b 仅作为非约束性 follow-up 拆分提案

| 项 | 内容 |
|---|---|
| 状态 | Accepted tracking boundary，非 CR-169 执行范围 |
| 背景 | global canonical N/A hardening 与 aggregate / CR-155 regression 的依赖和风险不同。 |
| 决策 | 仅登记可能的 FU-007a（canonical Gate 4 hardening）与 FU-007b（aggregate orchestration + CR-155 regression）拆分建议；不创建 CR、不改代码、不做依赖承诺。 |
| 备选 | 维持 FU-007 单一候选。 |
| 后果 | 为未来排程提供清晰边界，不影响 CR-169 completion。 |
| 切换条件 | 任一后续启动均需正式 CR、CP0 conflict precheck、独立 scope / authorization。 |

## ADR 一致性矩阵

| ADR | 回写 HLD | Blueprint / Domain / Dependency | 产品 / 场景 | 状态 |
|---|---|---|---|---|
| 001 | HLD §4、§6、§7 | CAP-CR169-01 / OBJ-CR169-01 / DEP-CR169-01 | REQ169-001..003 / SC-CR169-P01 | Accepted |
| 002 | HLD §6、§7.2 | CAP-CR169-02 / OBJ-CR169-03 / DEP-CR169-03 | REQ169-004 / SC-CR169-P02,B03 | Accepted |
| 003 | HLD §5、§7.2–7.4 | CAP-CR169-03 / OBJ-CR169-04 / DEP-CR169-05 | REQ169-005 / SC-CR169-P03,B01,B02 | Accepted |
| 004 | HLD §3、§6.4 | CAP-CR169-01 / OBJ-CR169-01 | REQ169-006 / SC-CR169-E01 | Accepted |
| 005 | HLD §1.2、§12 | CAP-CR169-04 / OBJ-CR169-05 | REQ169-008,009 / SC-CR169-E01 | Accepted |
| 006 | HLD §12 | DEP-CR169-08 | REQ169-009 | Accepted tracking boundary |

## Gotchas

1. ADR-003 的 positive outcome 不能被改名为 canonical Gate 4 PASS、capacity-ready 或 aggregate PASS。
2. ADR-002 的 hash 分域不放松 correlation；它反而要求 join 前精确 header comparison。
3. ADR-006 不是授权或 CR 创建；任何“并行启动 FU-007a”都必须重新取得用户明确指令。

# CP3 CR168 HLD Discussion Log

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 0.1 | 2026-07-14 | host-orchestrator inline meta-se-critical | 基于 CP2 approved 基线、canonical Gate4 代码事实、CR166 neutral envelope 与 feature contracts，完成五个 Architecture Gray Areas 的 table-first 分析。 |
| 0.2 | 2026-07-14 | host-orchestrator inline meta-se-critical | 吸收用户提交的 CP3 深度评审 A1–E3：A1 并入 DQ-HEADER 并在批准前修订；B–E 全部固化为 HLD/Domain/ADR 可实现契约。 |

## CP3 深度评审修订闭环

| Finding | 结论 | 修订位置 | 状态 |
|---|---|---|---|
| A1 component hash 与 multi-strategy fixture 歧义 | identity 只进入 envelope hash；字段族 2-9 的 subject-neutral 成本语义进入 component hash | HLD §5.4/§6/SIM-02；Blueprint owner table；ADR-003 | RESOLVED，DQ-HEADER approved |
| B1 participation proxy 边界 | 有限 `[0,+∞)`，允许 `>1`，不得产生 capacity claim | HLD §5.3；Domain NUM-04 | RESOLVED |
| B2 分项舍入 | 五分项 precision=28 保留；raw sum 后量化 total，随后量化 net | HLD §5.2；Domain NUM-05 | RESOLVED |
| B3 Gate4 release profile | 固定 `candidate-release`，caller 不可覆盖 | HLD §7；Domain §6；ADR-004 | RESOLVED |
| B4 minor unit 缺失 | 缺失/非正直接 blocked，无 fallback | HLD §5.2；Domain NUM-02 | RESOLVED |
| C1 canonical double | 仅替换 public validator callable，不复制私有实现 | HLD SIM-05 | RESOLVED |
| C2 reason code | 拆分 `gate4_unexpected_pass` / `gate4_postcondition_violation` | HLD §7.4；Domain §5/§7 | RESOLVED |
| D1 C4 adapter 演进 | CR168 不预设计；FU-005 决定扩展或 C4-owned projection | HLD §12；ADR-005 | RESOLVED |
| D2 schema 演进 | v1 immutable；新 family/算法/舍入/rebate 走新 schema + method CR | HLD §12；ADR-002 | RESOLVED |
| E1 registry | REQUIRED disposition=N/A-with-reason + feature/module refs | HLD §1.6/§8 | RESOLVED |
| E2 ADR capability IDs | 修正为 `CAP-CR168-*` | ADR consistency matrix | RESOLVED |
| E3 rebate | 独立 method CR + 新 schema | HLD §5.2/§12；ADR-002 | RESOLVED |

## 执行模式

- 用户指令：按照评审建议整改、批准 CP2，并推进到下一个人工门禁。
- 用户明确要求：不拉起子 Agent。
- 执行方式：用户批准的 `inline-fallback`；Host Orchestrator 承担 `meta-se-critical` 职责，`model_verified=false`，未声称启动 child thread。
- 本阶段只生成 Blueprint/Domain/Dependency/HLD/ADR、context、自动预检与 CP3 人工门禁；Story、LLD、source、test、verification 新增数均为 `0`。

## 代码事实

| Fact | 证据 | 架构影响 |
|---|---|---|
| Gate4 是 C3+C4 joint gate | `validate_gate4_capacity_impact` | C3 present 不能被解释为 capacity ready |
| C4 missing check 接受 `_has_na_reason` | canonical validator | absent+reason 可能绕过三项 missing blocked claim |
| `_has_na_reason` 候选为 6 个字段级 + 2 个通用 key | private helper | adapter threat model 精确为 8-key denylist |
| 当前生产调用点尚未出现；CR168 将新增首个兼容路径 | `rg` 调用面审计 | 风险当前 dormant，但必须在首个 caller 处 containment |
| CR166 已有 neutral envelope/catalog/hash | `engine/strategy_evidence.py` | 不需要新 envelope/registry |

## Architecture Gray Areas

| AGA | 问题 | 推荐 | 备选 | 影响 / 切换条件 |
|---|---|---|---|---|
| AGA-CR168-01 | C3、envelope 与 Gate4 如何分层？ | C3 叶子模块 + CR166 neutral envelope + 独立 guarded adapter | component-only；或全局 Gate4 修改 | 局部 guard 无法满足 B01/B02 时回退 component-only 并重开 CP2 |
| AGA-CR168-02 | v1 impact family 范围？ | present path 只启用显式 static square-root；N/A 为 unavailable；其他 family deferred | 一次支持全部 canonical family | 获得方法/参数/校准/独立验证授权后扩展版本 |
| AGA-CR168-03 | shared header 冻结到哪？ | identity、basis、lineage/auth 最小共享；C3/C4 exclusive owner 明确 | C3 完全独立 | owner 冲突或锁死 C4 时切换并重开 CP2 |
| AGA-CR168-04 | N/A escape 在哪里整改？ | CR168 adapter strict allowlist + exact 8-key presence denylist + pre/post guard | 修改 canonical；完全不做 projection | 当前禁止 canonical 备选；局部不足则回退 component-only |
| AGA-CR168-05 | registry/evidence taxonomy 缺口如何处理？ | 不创建 capability registry；feature/module refs 代替；formal required_evidence 在 CP3 approved 后映射到 `rerun_consistency` + `admission_package` | 顺手新建 registry/taxonomy | 只有项目级治理需求获独立 CR 才新建 |

## Advisor Table

| Option | Pros | Cons | Recommendation | When to switch |
|---|---|---|---|---|
| A. Layered C3 + guarded adapter | 达成 1 projection 与真实 consumer compatibility；不改全局 contract | adapter 多一组 guard | 推荐 | guard 无法无全局修改保证时切 B |
| B. Component-only | 最小、隔离强 | 无法满足当前 projection QAC，需要重开 CP2 | 备选 | A 失败 |
| C. Canonical global hardening | 长期覆盖全部 caller | 影响历史语义、需要全量 regression、超出范围 | 当前拒绝 | FU-007/direct caller/独立安全 CR |

## 关键设计选择

1. adapter 从 typed component 新建四字段 payload，不接受/透传任意 mapping。
2. 8-key denylist 按 key presence 拒绝，不因空值而放行。
3. reason escape 的 canonical 调用数为 `0`。
4. safe absent 固定 `candidate-release` 进入 canonical 后必须 BLOCKED 且含三个 C4 missing claims；PASS 与 postcondition violation 使用不同 reason code，aggregate 调用数为 `0`。
5. 不运行时 import `_has_na_reason`；只把当前候选 keys 固化成 CR168 自己的公开 contract。
6. 不建立“canonical permissive 行为必须保持”的长期 golden；测试只锁定 adapter 的安全后果。
7. Stage2 在 CR168 前已经 complete；本 CR 不“退出 Stage2”，也不启动 Stage3。

## Deferred / Follow-up

- `FU-CR161-005`：C4 capacity/liquidity/ADV/alpha-decay producer。
- `FU-CR161-007`：C1-C4 aggregate、StrategyAdmissionPackage 最终集成、canonical Gate4 N/A 语义复核、CR155 综合 regression/promotion decision。
- `FU-CR161-006`：独立 verifier lane；若 CP7 继续 inline，CP8 必须披露。
- 独立 real-data/method CR：真实 TCA、market impact calibration、多 impact family、真实 FX/ADV/liquidity。

## CP3 决策结果

用户于 2026-07-14 明确要求“按照评审意见修改后，批准 CP3”；A1–E3 已全部修订并通过一致性复核。`DQ-CP3-CR168-ARCH`、`METHOD`、`HEADER`、`GUARD`、`GLOBAL` 五项均 accepted；当前无 blocking clarification。

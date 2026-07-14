---
title: "CR-168 质量评审"
status: "cp7-complete-with-risk"
version: "1.1"
cr_id: "CR-168"
created_at: "2026-07-14T14:55:00+08:00"
owner: "host-orchestrator inline meta-qa"
---

# CR-168 质量评审

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-14 | host-orchestrator inline meta-qa | 对实现边界、测试、全仓回归归因与发布前风险完成评审。 |
| 1.1 | 2026-07-14 | host-orchestrator inline meta-qa | §2：纳入状态压缩审计归档的卫生分类整改与最终全仓回归复核。 |

## 1. 评审范围

审查 C3 typed component、计算与 hash 域、envelope attachment、局部 Gate4 projection guard、fixture/QAC/auth/CR155 regression、design/process/provenance 治理及最终测试证据。审查不包含真实数据、TCA、C4、canonical global remediation、aggregate、runtime 或发布操作。

## 2. Findings（按状态与严重度）

| ID | 严重度 | 状态 | 发现 | 处置与证据 |
|---|---|---|---|---|
| F-CR168-001 | 高 | RESOLVED | CR-named CP3 design 正文位于 current design root，违反 current-index design surface。 | 归档至 `process/archive/design-cr-docs/`，修复引用和 archive index；design-surface=PASS。 |
| F-CR168-002 | 高 | RESOLVED | C3 新 source/process assets 尚未获得 active-CR hygiene classification。 | 登记 CR-168 source/process asset set；hygiene unclassified=0。 |
| F-CR168-003 | 中 | RESOLVED | 七个 C3 测试文件未写入 test provenance。 | `tests/PROVENANCE.yaml` 完成 CR/Story 映射；taxonomy=PASS。 |
| F-CR168-004 | 高 | RESOLVED | Gate4 canonical 对 C4 absent + N/A reason 存在 permissive PASS 逃逸。 | S04 adapter 只允许四个 C3 fields、拒绝八个 reason keys，safe absent 强制精确 C4-missing BLOCKED；不修改 canonical。 |
| F-CR168-005 | 中 | RESOLVED | S05 的错误质量路由文字引用可能污染文档路径约束。 | 更正为抽象的“错误质量文档路由”表述；静态 route scan PASS。 |
| F-CR168-006 | 中 | RESOLVED | 状态 v2 slim 生成的 `archive/state/*` 审计文件被卫生检查误判为未分类。 | 将固定 slim 审计文件归为共享工作流资产；hygiene=PASS、专用 suite 21/21 PASS、最终全仓 2077 passed。 |

未发现开放的 BLOCKER 或 HIGH finding。所有已解决 finding 均经过定向和最终全仓 suite 验证。

## 3. 关键质量判断

- C3 component semantic hash 不含 manifest/run/strategy/package identity；同成本语义 daily/ML component hash 相同，而 envelope hash 保留 subject identity。
- `build_economic_cost_evidence` 统一执行 normalize→validate→issues short-circuit→calculate→produce；有 issue 时 calculator calls=0。
- Gate4 adapter 依赖 public `validate_gate4_capacity_impact`；测试仅使用 keyword-only public callable DI，无 monkeypatch、无 private `_has_na_reason` 依赖。
- canonical Gate4 和 aggregate orchestration source modifications=0/0；因此该修复是本地 caller containment，不能被误标为全局 canonical remediation。
- CR155 继续 `BLOCKED` 且 `paper_candidate=false`；没有 C4/capacity/aggregate PASS。

## 4. 剩余风险与建议

| 风险 | 影响 | 建议 |
|---|---|---|
| R-CR168-VERIFIER-INDEPENDENCE | CP7 由 inline fallback 执行，缺少独立 verifier lane。 | CP8 明确接受或拒绝该风险；若接受，readiness 必须是 `READY_WITH_RISK`。 |
| R-CR168-GATE4-C3-C4-SEMANTIC | 其他 canonical direct callers 仍可能允许 C4 N/A reason；本 CR adapter 已拒绝该 escape。 | FU-007 或独立 remediation 评估 global hardening/aggregate integration。 |
| R-CR168-TRUE-TCA-OVERCLAIM | 使用者可能将静态 approximation 误读为真实 TCA。 | CP8 release notes 和 claim ceiling 明确禁止此解释。 |

## 5. 评审结论

**PASS_WITH_RISK。** CR-168 的实现、测试与全仓回归符合已批准范围；可以打开 CP8 人工门禁。CP8 只能确认 fixture/static-only 交付就绪及风险接受，不构成 Stage 3、real TCA、runtime、C4/aggregate、CR155 promotion 或远端写入授权。

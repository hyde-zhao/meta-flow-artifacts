---
status: "verified-with-risk"
version: "1.0"
story_id: "CR046"
story_slug: "qmt-miniqmt-dual-target-framework"
feature_id: "FEAT-09"
validation_mode: "mixed"
verification_result: "PASS_WITH_RISK"
source_story: "process/changes/CR-046-TERMINAL-NATIVE-SIMULATION-STRATEGY-EXPORT-2026-06-13.md"
source_implementation: "process/stories/CR046-BATCH-A-IMPLEMENTATION.md"
created_by: "meta-qa"
created_at: "2026-06-18T06:34:50+08:00"
updated_at: "2026-06-18T06:34:50+08:00"
---

# Verification: CR046 QMT / MiniQMT Dual-Target Framework

## 1. 结论

| 项目 | 内容 |
|---|---|
| 阶段决策 | PASS_WITH_RISK |
| validation_mode | mixed |
| 是否可进入下一阶段 | yes |
| 需要路由 | CP8 risk acceptance |
| CP7 证据 | `process/checks/CP7-CR046-DUAL-TARGET-FRAMEWORK-BATCH-A-VERIFICATION-DONE.md` |
| Context Capsule | `process/context/CP7-CR046-VERIFICATION-CONTEXT.yaml` |

CR046 的 CP7 验证仅证明双目标策略交付框架、策略包契约、QMT terminal target 设计、MiniQMT runner 安装设计、验证证据模型和后续 CR 门禁在文档 / 契约层面闭环。它不证明任何真实 QMT / MiniQMT / XtQuant / gateway runtime 可用，不证明策略已可交易，不证明账户、资金、持仓、委托、成交、submit/cancel、simulation 或 live 能力。

## 2. 验证范围

| 项 | 内容 |
|---|---|
| Feature / Story | FEAT-09 / CR046-S01..S07 |
| 验证范围 | 静态文档、fixture-plan、契约标记、状态台账、禁止项边界 |
| 非范围 | QMT/MiniQMT/XtQuant/gateway 启动、NAS、`.env`、凭据、账号、账户、资金、持仓、委托、成交、日志原文、submit/cancel、simulation/live |
| 上游需求 | `process/USE-CASES.md` UC-28..32、`process/REQUIREMENTS.md` REQ-186..200 |
| 上游设计 | CR046 HLD、ADR、FEATURE-DESIGN-MATRIX、S01..S07 LLD / technical-note |
| 实现摘要 | `process/stories/CR046-BATCH-A-IMPLEMENTATION.md` |
| 阻塞条件 | N/A |

说明：当前仓库的 CR046 产品基线仍以 legacy `process/USE-CASES.md` / `process/REQUIREMENTS.md` 为准，REQ-198 明确不静默迁移到 `docs/product/` 双真相源。因此 `docs/product/SCENARIOS.yaml` 和 `docs/product/TEST-MATRIX.md` 缺失不作为 CP7 阻断，但作为 CP8 风险接受项登记。

## 3. 验证对象清单

| 对象 | 类型 | 来源 / 变更原因 | 验证方式 | 是否阻塞 | 结果 |
|---|---|---|---|---|---|
| `process/changes/CR-046-TERMINAL-NATIVE-SIMULATION-STRATEGY-EXPORT-2026-06-13.md` | formal CR | CR046 范围与不授权边界 | review | yes | PASS |
| `process/context/CP6-CR046-IMPLEMENTATION-CONTEXT.yaml` | context | CP6 实现上下文 | yaml parse / review | yes | PASS |
| `process/checks/CP6-CR046-DUAL-TARGET-FRAMEWORK-BATCH-A-CODING-DONE.md` | checkpoint | CP6 PASS | review | yes | PASS |
| `process/stories/CR046-BATCH-A-IMPLEMENTATION.md` | implementation evidence | Batch A 实现证据 | review | yes | PASS |
| `process/docs/design/HLD-CR046-QMT-MINIQMT-DUAL-TARGET-FRAMEWORK.md` | design | HLD | contract review | yes | PASS |
| `process/docs/design/ARCHITECTURE-DECISION-CR046.md` | design | ADR | contract review | yes | PASS |
| `process/docs/design/FEATURE-DESIGN-MATRIX.md` | design | FEAT-09 mapping | trace review | yes | PASS |
| `process/docs/source-archive/docs/qmt/CR046-DUAL-TARGET-STRATEGY-FRAMEWORK.md` | docs / contract | S01/S02/S03 | marker scan / review | yes | PASS |
| `process/docs/source-archive/docs/qmt/CR046-MINIQMT-RUNNER-INSTALL-DESIGN.md` | docs / install design | S04 | marker scan / review | yes | PASS |
| `process/docs/source-archive/docs/qmt/CR046-VERIFICATION-FRAMEWORK.md` | docs / evidence model | S05 | marker scan / wording review | yes | PASS |
| `process/docs/source-archive/docs/research/CR046-RESEARCH-FRAMEWORK-FOLLOWUP.md` | docs / follow-up | S07 | review | yes | PASS |
| `process/stories/CR046-S06-follow-up-strategy-delivery-gate.md` | technical-note | S06 | review | yes | PASS |

## 4. 验证追踪矩阵

| Scenario | Requirement | Story | Design Contract | Implementation | Test / Check | Status | Risk |
|---|---|---|---|---|---|---|---|
| TS-046-01 | REQ-186 / REQ-187 | S01 | FEAT-09 / StrategyCoreContract | dual-target framework doc | contract marker scan | PASS | N/A |
| TS-046-02 | REQ-188 / REQ-189 | S02 | StrategyPackageContract | package layout / manifest / sha256 / transfer design | schema and artifact wording review | PASS | No real transfer |
| TS-046-03 | REQ-190 / REQ-191 | S03 | QMTTerminalTargetContract | terminal target import and shadow plan | no-runtime boundary review | PASS | No terminal run |
| TS-046-04 | REQ-192 / REQ-193 | S04 | MiniQMTRunnerTargetContract | install design / uv / dirs / kill switch | install design review | PASS | No real install / connection |
| TS-046-05 | REQ-194 / REQ-195 | S05 | StrategyValidationEvidence | validation framework doc | runtime claim scan | PASS | Runtime evidence unavailable |
| TS-046-06 | REQ-196 / REQ-197 | S06 | follow-up strategy delivery gate | technical-note + follow-up tracking | CR gate review | PASS | Follow-up not started |
| TS-046-07 | REQ-198..REQ-200 | S07 | research framework follow-up | research follow-up doc | baseline route review | PASS_WITH_RISK | Legacy product baseline |

## 5. 设计契约验证清单

| 契约 | 来源 | 验证方式 | 是否阻塞 | 结果 | 证据 |
|---|---|---|---|---|---|
| FEAT-09 独立承载 QMT / MiniQMT 双目标策略交付 | CP3 / HLD / S01 | docs review | yes | PASS | framework doc `implemented-cp6` |
| StrategyCoreContract 平台无关，不引入 QMT / XtQuant / MiniQMT runtime | S01 | marker scan | yes | PASS | `StrategyCoreContract` |
| 策略包默认 zip + sha256 + manifest + docs bundle | S02 | docs review | yes | PASS | `StrategyPackageContract` |
| QMT terminal target 只定义人工导入和 shadow plan | S03 | wording review | yes | PASS | `QMTTerminalTargetContract` |
| MiniQMT runner 只做安装设计，不做真实安装 / 连接 | S04 | docs review | yes | PASS | `MiniQMTRunnerTargetContract` |
| StrategyValidationEvidence 不声明 `runtime_verified=true` | S05 | runtime claim scan | yes | PASS | `runtime_verified: false` |
| 具体策略交付后置 | S06 | follow-up review | yes | PASS | CR047/CR048/CR049 later gate |
| 研究框架完善后置 | S07 | follow-up review | yes | PASS | CR051 / later research handoff |

## 6. 分层验证计划

| 验证层 | 方法 | 目标 | 必跑 | 结果 | 未覆盖风险 |
|---|---|---|---|---|---|
| 工作区健康 | `meta-flow workspace check` | symlink / process ledger 可用 | yes | PASS | N/A |
| 静态格式 | scoped `git diff --check` | 空白和格式 | yes | PASS | 新增 CP7 文件需最终复跑 |
| YAML / 状态 | scoped YAML parse + CR consistency | CP6 context / plan / CR ledger | yes | PASS_WITH_RISK | CR-INDEX 全量 YAML parse 有非 CR046 历史债 |
| 契约检查 | marker scan | 合同与不授权标记存在 | yes | PASS | N/A |
| wording guardrail | ready/runtime 负向语境检查 | 防止误写 runtime ready | yes | PASS | N/A |
| fixture-plan | 文档证据分级审查 | fixture/static/manual plan 不等于 runtime | yes | PASS | 无真实 runtime |
| runtime / platform | N/A | 用户未授权 | no | N/A | 必须后续独立 CR |
| 人工 / 语义审查 | meta-qa 子 agent | 范围、风险和边界复核 | yes | PASS_WITH_RISK | CP8 需接受剩余风险 |

## 7. 自动化验证结果

| Command ID | 命令 / 检查 | 结果 | 证据 | 说明 |
|---|---|---|---|---|
| CMD-00 | `uv run --python 3.11 meta-flow workspace check --project-root /home/hyde/workspace/quant-lab` | PASS | `process_link_health: ok` / `artifact_git_dirty: clean` | 用户要求的首条命令已先执行。 |
| CMD-01 | `uv run --python 3.11 python scripts/check_cr_tracking_consistency.py --project-root /home/hyde/workspace/quant-lab` | PASS | `CR tracking consistency: PASS` | CR020 deleted、CR046 active、CR089/CR091 blocked 口径通过。 |
| CMD-02 | scoped YAML parse for `process/DEVELOPMENT-PLAN.yaml` and `process/context/CP6-CR046-IMPLEMENTATION-CONTEXT.yaml` | PASS | `YAML_OK ...` | 不解析 `.env` 或凭据。 |
| CMD-03 | full YAML parse for `process/changes/CR-INDEX.yaml` | PASS_WITH_RISK | fails at non-CR046 CR075 `blocked_by` scalar | 历史台账格式债，不在 CR046 修复范围内。 |
| CMD-04 | scoped `git diff --check` before CP7 file creation | PASS | no output | 最终会随新增文件复跑。 |
| CMD-05 | contract marker scan | PASS | `CR046_CONTRACT_MARKERS_OK 9` | 关键合同和禁止项标记齐全。 |
| CMD-06 | ready/runtime negative-context scan | PASS | `CR046_READY_RUNTIME_TERMS_NEGATIVE_CONTEXT_OK 8` | 未发现把 ready/verified 误写为运行授权的上下文。 |
| CMD-07 | dangerous executable / runtime pattern scan | PASS | `NO_EXECUTABLE_DANGEROUS_OR_RUNTIME_PATTERNS` | 未发现可执行运行时启动或交易动作。 |

## 8. Fixture / Contract 验证

| Fixture ID | 输入 / 场景 | 期望 | 实际 | 结果 | 证据 |
|---|---|---|---|---|---|
| FX-CR046-01 | StrategyCoreContract markers | core 合同存在且平台无关 | marker present | PASS | framework doc |
| FX-CR046-02 | StrategyPackageContract markers | zip / sha256 / manifest / docs bundle 合同存在 | marker present | PASS | framework doc |
| FX-CR046-03 | QMT terminal target | 仅 shadow plan / manual import design | runtime not authorized | PASS | framework + verification docs |
| FX-CR046-04 | MiniQMT install design | 仅目录、uv、依赖隔离、kill switch、rollback 设计 | real install false | PASS | install design doc |
| FX-CR046-05 | StrategyValidationEvidence | `runtime_verified` 不可为 true | `runtime_verified: false` | PASS | verification framework |
| FX-CR046-06 | Follow-up gates | CR047/CR048/CR049/CR051 后置，不自动启动 | no auto-start | PASS | S06/S07 evidence |

## 9. 平台适配验证

| 平台 / 对象 | 检查项 | 预期 | 结果 | 证据 |
|---|---|---|---|---|
| Codex | 使用真实 meta-qa 子 agent 复核 CP7 | yes | PASS | `qa-he` / `019ed7b0-70b5-78d2-a59c-09a96ee97c14` |
| QMT terminal | 启动 / shadow / 导入 | not-authorized | N/A | 未执行 |
| MiniQMT / XtQuant | install / connect / runtime | not-authorized | N/A | 未执行 |
| NAS | strategy package exchange | not-authorized | N/A | 未访问 |
| `.env` / 凭据 / 账号 | read / parse / print | not-authorized | N/A | 未读取 |
| submit/cancel / simulation/live | runtime action | not-authorized | N/A | 未执行 |

## 10. 人工 / 语义质量审查

| 检查项 | 结果 | 是否阻塞 | 说明 |
|---|---|---|---|
| 需求一致性 | PASS | no | UC-28..32、REQ-186..200 与 S01..S07 有对应关系。 |
| 场景覆盖 | PASS_WITH_RISK | no | TS-046-01..07 在 legacy product baseline 中可追溯；canonical docs/product 文件缺失作为风险。 |
| 文档可用性 | PASS | no | 四份 CR046 交付文档均为 `implemented-cp6`，合同边界清晰。 |
| 错误信息可行动 | PASS | no | 后续真实运行必须另起 CR / runtime gate。 |
| 是否只覆盖 happy path | PASS | no | 明确了禁止项、后续门禁和 runtime evidence unavailable。 |
| 是否误授权真实操作 | PASS | no | 未发现 runtime 授权被打开。 |

## 11. 问题清单

| ID | Severity | 状态 | 说明 | 处理 |
|---|---|---|---|---|
| R-CR046-CP7-001 | MEDIUM | open-for-cp8-risk-acceptance | `docs/product/SCENARIOS.yaml` / `docs/product/TEST-MATRIX.md` 缺失；CR046 等价基线在 legacy `process/USE-CASES.md` / `process/REQUIREMENTS.md`。 | CP8 风险接受；不要静默迁移双真相源。 |
| R-CR046-CP7-002 | MEDIUM | open-non-cr046-ledger-debt | `process/changes/CR-INDEX.yaml` 全量 YAML parse 在非 CR046 的 CR075 行失败。 | 不在 CR046 中修复；后续 ledger hygiene CR 处理。 |
| R-CR046-CP7-003 | MEDIUM | expected-by-scope | 无真实 QMT/MiniQMT/NAS/凭据/账户/submit/cancel/simulation/live 证据。 | 这是本轮授权边界，不阻断 framework-first CP7。 |

## 12. 剩余风险

| 风险 | 等级 | 影响 | 缓解 |
|---|---|---|---|
| framework-ready 被误读为 trade-ready | high | 可能越权执行真实交易路线 | CP7 / CP8 明确只能声明 framework-ready-with-risk，不授权 runtime。 |
| legacy product baseline 与 docs/product 路由差异 | medium | 后续审计需要知道真实来源 | 报告中显式引用 REQ-198 和 legacy 文件。 |
| CR-INDEX 非 CR046 YAML 债 | medium | 通用 YAML 工具读取受影响 | 保持脚本级一致性 PASS，后续独立修复。 |
| CR089 / CR091 与 CR046 重叠 | high | 后续自动启动可能误越权 | 本轮明确保持 blocked，不借 CR046 CP7 自动恢复。 |

## 13. 阶段决策

| 项 | 决策 |
|---|---|
| CP7 决策 | PASS_WITH_RISK |
| 可进入 | CP8 Delivery Readiness |
| 阻塞缺陷 | 0 |
| 高风险误授权 | 0 |
| 需要用户接受 | CP8 风险接受：当前只关闭 framework-first 文档 / 契约交付，不授权 runtime / NAS / 凭据 / 交易动作 |

## 14. CP8 输入

- CP8 可评审 CR046 是否关闭为 `framework-ready-with-risk` 或等价状态。
- CP8 不得被解释为授权 QMT/MiniQMT/XtQuant/gateway 启动。
- CP8 不得被解释为授权 NAS、`.env`、凭据、账号、账户、资金、持仓、委托、成交或日志原文读取。
- CP8 不得被解释为授权 submit/cancel、simulation/live。
- CR020 保持 `deleted-by-user`。
- CR089 和 CR091 保持 blocked，需独立用户授权和门禁。

## 15. 结论摘要

CR046 CP7 在允许范围内通过，结论为 `PASS_WITH_RISK`。已验证的能力是“框架、文档、契约、fixture-plan 和证据分级可用于后续 CP8 决策”；未验证且未授权的能力是任何真实交易终端、真实连接、真实数据/账号、真实下单或真实模拟 / 实盘运行。

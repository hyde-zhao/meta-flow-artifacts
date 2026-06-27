# Current Meta Flow State

Project: quant-lab
Workflow mode: standard
Phase: story-execution
Blocked: false
Active CR: none
Active Story: none
Pending gate: none
Next action: Stage 3 研究机交接文档已准备好，当前 active context 切换为 `process/context/RUNNER-QMT-SIMULATION-MULTIFACTOR-STAGE3-RESEARCH-MACHINE-HANDOFF-2026-06-26.md`。用户将在另一台可以连接数据湖的研究机实施 Stage 3：按 handoff 清单准备真实数据输入、生成 research evidence、构建 mature admission package 和 runner offline preflight。当前机器不连接数据湖，不执行 simulation runtime，不启动 gateway，不进入 `small_live` / `live`。

Refs:
- state: process/state/STATE.current.json
- CR ledger: process/state/CR-LEDGER.ndjson
- Story ledger: process/state/STORY-LEDGER.ndjson
- Checkpoint ledger: process/state/CHECKPOINT-LEDGER.ndjson
- Handoff ledger: process/state/HANDOFF-LEDGER.ndjson
- Agent dispatch ledger: process/state/AGENT-DISPATCH-LEDGER.ndjson
- Gate ledger: process/state/GATE-LEDGER.ndjson
- Run ledger: process/state/RUN-LEDGER.ndjson
- Read expansion ledger: process/state/READ-EXPANSION-LEDGER.ndjson
- routing: process/.meta-flow-process.yaml
- active context: process/context/RUNNER-QMT-SIMULATION-MULTIFACTOR-STAGE3-RESEARCH-MACHINE-HANDOFF-2026-06-26.md
- stage3 research machine handoff: process/context/RUNNER-QMT-SIMULATION-MULTIFACTOR-STAGE3-RESEARCH-MACHINE-HANDOFF-2026-06-26.md
- next phase goals and plan: process/context/RUNNER-QMT-SIMULATION-MULTIFACTOR-NEXT-PHASE-GOALS-AND-PLAN-2026-06-26.md
- project blueprint: docs/design/BLUEPRINT.md
- project domain map: docs/design/DOMAIN-MAP.md
- project dependency map: docs/design/DEPENDENCY-MAP.md
- archived project blueprint: process/archive/design-blueprints/BLUEPRINT-before-quant-lab-project-roadmap-2026-06-26.md
- stage2 framework module: engine/mature_multifactor_framework.py
- stage2 framework test: tests/test_stage2_mature_multifactor_framework.py
- stage2 framework fixture: tests/fixtures/stage2_multifactor_framework/mature_multifactor_stage2_fixture.json
- stage2 framework check: process/checks/STAGE2-MULTIFACTOR-FRAMEWORK-NO-LAKE-INITIAL-SLICE-2026-06-26.md
- phase goals: process/context/RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-PHASE-GOALS-2026-06-25.md
- P0 evidence: process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-P0-EVIDENCE-2026-06-25.json
- P1 check: process/checks/RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-P1-MULTIFACTOR-TARGET-PORTFOLIO-2026-06-25.md
- P1 evidence: process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-P1-EVIDENCE-2026-06-25.json
- P2 check: process/checks/RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-P2-SIMULATION-ORDER-PLAN-2026-06-25.md
- P2 evidence: process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-P2-EVIDENCE-2026-06-25.json
- P3 check: process/checks/RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-P3-SIMULATION-EXECUTION-ENGINE-2026-06-25.md
- P3 evidence: process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-P3-EVIDENCE-2026-06-25.json
- P4 check: process/checks/RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-P4-RECONCILIATION-2026-06-25.md
- P4 evidence: process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-P4-EVIDENCE-2026-06-25.json
- P5 check: process/checks/RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-P5-OPERATIONAL-RUNBOOK-2026-06-25.md
- P5 evidence: process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-P5-EVIDENCE-2026-06-25.json
- P5 runbook: process/runbooks/RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-RUNBOOK-2026-06-25.md
- P6 check: process/checks/RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-P6-DEFERRED-LIVE-SWITCH-SCENARIO-2026-06-25.md
- P6 evidence: process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-P6-EVIDENCE-2026-06-25.json
- P6 scenario pack: process/context/RUNNER-QMT-SIMULATION-TO-LIVE-SWITCH-DEFERRED-SCENARIO-PACK-2026-06-25.md
- real readonly evidence: process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-REAL-READONLY-EVIDENCE-2026-06-25.json
- real operator check: process/checks/RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-REAL-OPERATOR-2026-06-25.md
- real operator evidence: process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-REAL-OPERATOR-EVIDENCE-2026-06-25.json
- after-restart verification: process/checks/RUNNER-QMT-SIMULATION-MULTIFACTOR-AFTER-RESTART-VERIFICATION-2026-06-25.md
- after-restart readonly evidence: process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-AFTER-RESTART-READONLY-EVIDENCE-2026-06-25.json
- after-restart runtime evidence: process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-AFTER-RESTART-RUNTIME-EVIDENCE-2026-06-25.json
- next-session handoff: process/context/RUNNER-QMT-SIMULATION-MULTIFACTOR-NEXT-SESSION-HANDOFF-2026-06-25.md
- formal strategy admission package: process/context/RUNNER-QMT-SIMULATION-MULTIFACTOR-FORMAL-STRATEGY-ADMISSION-PACKAGE-2026-06-25.json
- formal operator spec: process/context/RUNNER-QMT-SIMULATION-MULTIFACTOR-FORMAL-OPERATOR-SPEC-2026-06-25.json
- non-trading completion: process/checks/RUNNER-QMT-SIMULATION-MULTIFACTOR-NON-TRADING-WINDOW-COMPLETION-2026-06-25.md
- formal preflight index: process/evidence/runner-simulation-formal-2026-06-25/preflight-only/runner-qmt-simulation-multifactor-formal-non-trading-2026-06-25/operator-evidence.index.json
- formal plan-only index: process/evidence/runner-simulation-formal-2026-06-25/plan-only/runner-qmt-simulation-multifactor-formal-non-trading-2026-06-25/operator-evidence.index.json
- formal fixture index: process/evidence/runner-simulation-formal-2026-06-25/fixture/runner-qmt-simulation-multifactor-formal-non-trading-2026-06-25/operator-evidence.index.json
- formal reconcile-only index: process/evidence/runner-simulation-formal-2026-06-25/reconcile-only/runner-qmt-simulation-multifactor-formal-non-trading-2026-06-25/operator-evidence.index.json
- trading-window acceptance handoff: process/context/RUNNER-QMT-SIMULATION-MULTIFACTOR-TRADING-WINDOW-ACCEPTANCE-HANDOFF-2026-06-26.md
- trading-window acceptance blocked: process/checks/RUNNER-QMT-SIMULATION-MULTIFACTOR-TRADING-WINDOW-ACCEPTANCE-2026-06-26.md
- trading-window readonly evidence: process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-TRADING-WINDOW-READONLY-EVIDENCE-2026-06-26.json
- trading-window runtime evidence: process/evidence/runner-simulation-runtime-2026-06-26/runner-qmt-simulation-multifactor-trading-window-2026-06-26-r1/operator-evidence.json
- trading-window r2 revalidation: process/checks/RUNNER-QMT-SIMULATION-MULTIFACTOR-TRADING-WINDOW-R2-REVALIDATION-2026-06-26.md
- trading-window r2 readonly evidence: process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-TRADING-WINDOW-READONLY-EVIDENCE-2026-06-26-r2.json
- trading-window r2 runtime evidence: process/evidence/runner-simulation-runtime-2026-06-26/runner-qmt-simulation-multifactor-trading-window-2026-06-26-r2/operator-evidence.json
- formal runtime input fix: process/checks/RUNNER-QMT-SIMULATION-MULTIFACTOR-FORMAL-RUNTIME-INPUT-FIX-2026-06-26.md
- runtime input builder: trading/strategy_runner/runtime_inputs.py
- runtime input builder CLI: scripts/build_qmt_multifactor_runtime_inputs.py
- private runtime r3 overlay: /home/hyde/.quant-lab/runtime/qmt/cr138-simulation/runner-qmt-simulation-multifactor-trading-window-2026-06-26-r3-runtime-overlay.json
- private runtime r3 spec: /home/hyde/.quant-lab/runtime/qmt/cr138-simulation/runner-qmt-simulation-multifactor-trading-window-2026-06-26-r3-runtime-spec.json
- private runtime r3 admission: /home/hyde/.quant-lab/runtime/qmt/cr138-simulation/runner-qmt-simulation-multifactor-trading-window-2026-06-26-r3-runtime-admission-package.json
- private runtime r3 plan-only evidence: process/evidence/runner-simulation-runtime-input-fix-2026-06-26/plan-only/runner-qmt-simulation-multifactor-trading-window-2026-06-26-r3/operator-evidence.json
- trading-window r3 runtime acceptance: process/checks/RUNNER-QMT-SIMULATION-MULTIFACTOR-TRADING-WINDOW-R3-RUNTIME-ACCEPTANCE-2026-06-26.md
- trading-window r3 readonly evidence: process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-TRADING-WINDOW-READONLY-EVIDENCE-2026-06-26-r3.json
- trading-window r3 runtime evidence: process/evidence/runner-simulation-runtime-2026-06-26/runner-qmt-simulation-multifactor-trading-window-2026-06-26-r3/operator-evidence.json
- trading-window r3 runtime evidence index: process/evidence/runner-simulation-runtime-2026-06-26/runner-qmt-simulation-multifactor-trading-window-2026-06-26-r3/operator-evidence.index.json
- trading-window r4 runtime acceptance: process/checks/RUNNER-QMT-SIMULATION-MULTIFACTOR-TRADING-WINDOW-R4-RUNTIME-ACCEPTANCE-2026-06-26.md
- trading-window r4 P0 health evidence: process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-TRADING-WINDOW-P0-HEALTH-EVIDENCE-2026-06-26-r4.json
- trading-window r4 P0 capabilities evidence: process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-TRADING-WINDOW-P0-CAPABILITIES-EVIDENCE-2026-06-26-r4.json
- trading-window r4 readonly evidence: process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-TRADING-WINDOW-READONLY-EVIDENCE-2026-06-26-r4.json
- trading-window r4 runtime evidence: process/evidence/runner-simulation-runtime-2026-06-26/runner-qmt-simulation-multifactor-trading-window-2026-06-26-r4/operator-evidence.json
- trading-window r4 runtime evidence index: process/evidence/runner-simulation-runtime-2026-06-26/runner-qmt-simulation-multifactor-trading-window-2026-06-26-r4/operator-evidence.index.json
- trading-window r4 stability window summary: process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-STABILITY-WINDOW-SUMMARY-2026-06-26-r4.json
- trading-window r5 runtime acceptance: process/checks/RUNNER-QMT-SIMULATION-MULTIFACTOR-TRADING-WINDOW-R5-RUNTIME-ACCEPTANCE-2026-06-26.md
- trading-window r5 P0 health evidence: process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-TRADING-WINDOW-P0-HEALTH-EVIDENCE-2026-06-26-r5.json
- trading-window r5 P0 capabilities evidence: process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-TRADING-WINDOW-P0-CAPABILITIES-EVIDENCE-2026-06-26-r5.json
- trading-window r5 readonly evidence: process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-TRADING-WINDOW-READONLY-EVIDENCE-2026-06-26-r5.json
- trading-window r5 runtime evidence: process/evidence/runner-simulation-runtime-2026-06-26/runner-qmt-simulation-multifactor-trading-window-2026-06-26-r5/operator-evidence.json
- trading-window r5 runtime evidence index: process/evidence/runner-simulation-runtime-2026-06-26/runner-qmt-simulation-multifactor-trading-window-2026-06-26-r5/operator-evidence.index.json
- trading-window r5 stability window summary: process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-STABILITY-WINDOW-SUMMARY-2026-06-26-r5.json
- trading-window r6 runtime acceptance: process/checks/RUNNER-QMT-SIMULATION-MULTIFACTOR-TRADING-WINDOW-R6-RUNTIME-ACCEPTANCE-2026-06-26.md
- trading-window r6 P0 health evidence: process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-TRADING-WINDOW-P0-HEALTH-EVIDENCE-2026-06-26-r6.json
- trading-window r6 P0 capabilities evidence: process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-TRADING-WINDOW-P0-CAPABILITIES-EVIDENCE-2026-06-26-r6.json
- trading-window r6 readonly evidence: process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-TRADING-WINDOW-READONLY-EVIDENCE-2026-06-26-r6.json
- trading-window r6 runtime evidence: process/evidence/runner-simulation-runtime-2026-06-26/runner-qmt-simulation-multifactor-trading-window-2026-06-26-r6/operator-evidence.json
- trading-window r6 runtime evidence index: process/evidence/runner-simulation-runtime-2026-06-26/runner-qmt-simulation-multifactor-trading-window-2026-06-26-r6/operator-evidence.index.json
- trading-window r6 stability window summary: process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-STABILITY-WINDOW-SUMMARY-2026-06-26-r6.json
- simulation readiness closure: process/checks/RUNNER-QMT-SIMULATION-MULTIFACTOR-SIMULATION-OPERATIONAL-READINESS-CLOSURE-2026-06-26.md
- simulation readiness release context: process/release/RELEASE-CONTEXT-RUNNER-QMT-SIMULATION-MULTIFACTOR-SIMULATION-READINESS-2026-06-26.yaml
- simulation readiness human decision: process/checkpoints/RUNNER-QMT-SIMULATION-MULTIFACTOR-SIMULATION-READINESS-HUMAN-DECISION-2026-06-26.md
- simulation readiness runbook addendum: process/runbooks/RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-RUNBOOK-2026-06-25.md
- simulation ops policy: process/policies/RUNNER-QMT-SIMULATION-MULTIFACTOR-RUNTIME-INPUT-GATEWAY-LIFECYCLE-POLICY-2026-06-26.md
- simulation ops policy closure: process/checks/RUNNER-QMT-SIMULATION-MULTIFACTOR-OPS-POLICY-CLOSURE-2026-06-26.md

Policy refs:
- SIMULATION_RUNTIME_AUTHORIZED_THIS_RUN
- NO_RAW_CREDENTIAL_PERSISTENCE
- NO_RAW_ACCOUNT_SYMBOL_ORDER_PERSISTENCE
- NO_NAS
- NO_SMALL_LIVE_OR_LIVE
- NO_PROVIDER_LAKE_CATALOG
- NO_REMOTE_WRITE

Open risks:
- {'id': 'RISK-CR138-RUNTIME-MISREAD', 'summary': 'CR138 fixture/static verified 可能被误读为 QMT / gateway runtime 或交易授权；当前 CP7 使用 PASS_WITH_RISK 并继续要求后续按需 runtime_authorization。', 'status': 'controlled'}
- {'id': 'RISK-RUNNER-MULTIFACTOR-REAL-SIMULATION-SESSION-EXPIRED', 'summary': '用户重启 Windows QMT gateway 后，CR138 signed positions readonly 与临时最小 spec 的真实 multifactor simulation runtime operator 已通过；旧 session_expired 阻断关闭。', 'status': 'closed-after-restart-verification'}
- {'id': 'RISK-RUNNER-MULTIFACTOR-FORMAL-SPEC-NOT-YET-STABLE', 'summary': 'r3 私有 runtime overlay 生成的正式 runtime spec/admission 已通过交易窗口 simulation runtime；长期稳定性窗口仍需继续逐次授权累计。', 'status': 'closed-after-r3-runtime-pass'}
- {'id': 'RISK-RUNNER-SIMULATION-SMOKE-RUNTIME-AUTHORIZATION', 'summary': 'RUNNER-QMT-SIMULATION-TRADING-MACHINE-SMOKE-2026-06-24 已获本轮模拟账户 smoke 授权，gateway runtime ready，WSL HMAC client env 可用，positions readonly、simulation buy submit/cancel 和 simulation sell submit/cancel 均通过；未授权 small_live/live。', 'status': 'closed'}
- {'id': 'RISK-RUNNER-MULTIFACTOR-FIXTURE-INSTRUMENT-RUNTIME-SUBMIT-TIMEOUT', 'summary': '已新增私有 runtime input builder 和 operator runtime_symbol_contract_invalid guard；当前 formal fixture spec 不会再触达 gateway。业务验收仍需用户提供私有 runtime overlay 后重新逐次授权。', 'status': 'mitigated-by-input-builder-and-runtime-guard'}
- {'id': 'RISK-RUNNER-MULTIFACTOR-STABILITY-WINDOW-COLLECTING', 'summary': '交易窗口 simulation runtime r6 已通过 P0/P0.5/P1/P2/P3/P4；稳定性窗口当前 completed_runs=5/5，Phase A 已完成。', 'status': 'closed'}
- {'id': 'RISK-RUNNER-MULTIFACTOR-STABILITY-COUNTER-SPEC-REFS', 'summary': 'r4/r5 operator evidence 内嵌 stability_window.completed_runs 与 Phase A rollup 不一致；已分别用独立 stability summary 记录 3/5 与 4/5，并修复 runtime_inputs.py 使 r6 stability_evidence_refs 只包含历史成功 refs。', 'status': 'mitigated'}
- {'id': 'RISK-RUNNER-MULTIFACTOR-SIMULATION-READINESS-CLOSURE-PENDING', 'summary': 'simulation stability window 已达到 5/5；Phase B readiness closure 已生成，结论 READY_WITH_RISK。', 'status': 'closed-ready-with-risk'}
- {'id': 'RISK-RUNNER-MULTIFACTOR-THREE-TRADING-DAY-OBSERVATION', 'summary': '5/5 runtime count gate 已通过，但 runbook 保留 3 trading days 观察口径；当前证据主要覆盖 2026-06-25 与 2026-06-26。用户已接受 READY_WITH_RISK；补第三个交易日观察仅作为未来升级 READY 的可选后续。', 'status': 'accepted-ready-with-risk'}
- {'id': 'RISK-RUNNER-MULTIFACTOR-RUNTIME-INPUT-OPERABILITY', 'summary': 'runtime input overlay 来源、刷新触发、字段边界、私有输出目录和脱敏 evidence 边界已在 SIM-OPS policy 中固化。', 'status': 'policy-defined'}
- {'id': 'RISK-RUNNER-MULTIFACTOR-GATEWAY-LIFECYCLE-DESYNC', 'summary': 'Windows gateway 启停、重启、WSL/Windows runtime code sync 和 P0/P0.5 复验规则已在 SIM-OPS policy 中固化。', 'status': 'policy-defined'}
- {'id': 'RISK-RUNNER-MULTIFACTOR-PROJECT-BLUEPRINT-DRIFT', 'summary': 'Stage 1 已在当前 quant-lab 项目蓝图三件套上合并五阶段路线和多策略类型适配边界，旧蓝图已归档，避免新增孤立端到端策略研究蓝图。', 'status': 'mitigated'}
- {'id': 'RISK-RUNNER-MULTIFACTOR-STAGE2-DATA-LAKE-COUPLING', 'summary': 'Stage 2 多因子研究框架升级不连接数据湖；桥接切片已用 no-lake counter validation、typed unavailable、CR030/CR039 bridge 和 Stage 3 handoff 测试固化边界。真实数据湖接入推迟到 Stage 3 由用户在研究机实施。', 'status': 'controlled'}
- {'id': 'RISK-RUNNER-MULTIFACTOR-MATURE-STRATEGY-GAP', 'summary': '当前可运行对象仍是 fixture / 综合 alpha score 注入包样例；Stage 2 已具备成熟策略生产框架和 handoff，Stage 3 仍需基于研究机数据湖生产真实策略研究、evidence 和准入包。', 'status': 'open'}
- {'id': 'RISK-RUNNER-MULTIFACTOR-UNIVERSE-SCALE-UP', 'summary': '从 2 个 ETF / fixture 扩展到真实股票池会引入 PIT 数据质量、流动性、停牌、ST、涨跌停、容量和调仓复杂度；不得直接扩大 runtime，必须先按 Stage 3 handoff 完成 universe policy、风险过滤和证据追溯。', 'status': 'open'}

Updated at: 2026-06-26T15:30:00+08:00

<!-- generated-by: meta-flow state render -->

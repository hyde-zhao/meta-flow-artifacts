---
status: "draft-current-index"
version: "1.4"
source_blueprint: "docs/design/BLUEPRINT.md"
change: "CR-138"
---

# Domain Map

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-06-07 | meta-po | 新增领域对象、状态机、术语和核心业务规则索引 |
| 1.1 | 2026-06-13 | meta-po | 按 CR-046 增补双目标策略交付框架领域对象、状态机和规则 |
| 1.2 | 2026-06-14 | host-orchestrator | 按 CR-051 增补策略研究生命周期、研究归档、项目身份迁移和硬件冷热分层规则 |
| 1.3 | 2026-06-24 | host-orchestrator | 按 CR138 增补 Runner Control Plane 与 QMT Gateway Service Layer 运营领域对象、状态机和 fail-closed 规则 |
| 1.4 | 2026-06-24 | host-orchestrator | 根据用户 CP3 反馈补齐 FEAT-12 查询领域对象：交易日历、交易窗口、佣金 / 费用模型、成本估算、收益 / PnL 快照和收益摘要；runtime policy 改为按需授权 |

## 术语表

| Term | 定义 | 来源 | 备注 |
|---|---|---|---|
| current truth | 经 Explicit Publish Gate 发布并由 catalog current pointer 指向的数据真相 | CR-014 / CR-018 | validate pass 不等于 current truth |
| candidate | 尚未发布的候选数据、候选报告或候选 release | CR-014 / CR-018 | 可验证、可审计，但不得默认消费为生产事实 |
| publish gate | 将 candidate 显式升级为 catalog current truth 的唯一门禁 | HLD-DATA-LAKE | 必须可回滚 |
| readiness | 数据或研究对象是否满足 production strict 消费的准入状态 | CR-010..018 | 与 quality 相关但不等同 |
| blocked claim | 因缺数据、未授权、未通过 gate 或后置能力未满足而禁止声明的能力 | CR-013..030 | 必须写解除条件 |
| allowed claim | 在证据和 gate 支持下允许对外声明的能力 | CR-013..030 | 必须可追溯 |
| order_intent_draft_v1 | 研究输出到交易治理的草稿合同 | CR-025 / CR-030 | 不等于真实订单或授权 |
| runtime authorization | 对真实运行、真实数据、真实账户、真实 gateway / QMT 操作的逐项授权 | CR-020 / CR-021..024 | CP approval 不自动授权 runtime |
| no-real-operation | 默认安全状态，真实 provider / lake / QMT / broker / credential 相关计数必须为 0 | 全部高风险 CR | 有授权时必须记录 authorization_id |
| simulation | QMT 模拟盘阶段 | CR-016 / CR-021 candidate | 需要独立 CR、stage gate 和 per-run authorization |
| live_readonly | 真实账户只读阶段 | CR-022 candidate | 不等于 small_live |
| small_live | 小资金实盘阶段 | CR-023 candidate | 需要更高门槛和回滚策略 |
| strategy core contract | 研究策略可复用核心的输入、输出、风控和证据合同 | CR-046 | 不得导入 QMT / XtQuant |
| QMT terminal target | 面向 QMT 终端内运行的策略包目标形态 | CR-046 | 本 CR 只设计，不执行终端验证 |
| MiniQMT runner target | 面向外部 Python runner 的策略包目标形态和安装设计 | CR-046 | 本 CR 只设计 install dry-run，不真实安装或连接 |
| strategy package contract | 双目标策略交付包的目录、schema、配置、文档和验证证据约束 | CR-046 | 后续 CR047 具体策略必须消费 |
| strategy validation evidence | 策略包静态、fixture、dry-run 和人工验证计划证据集合 | CR-046 | 不等于 QMT / MiniQMT runtime verified |
| quant-lab | 本项目迁移后的 canonical 项目名 | CR-051 | 新文档、新迁移目标和后续项目身份使用该名称 |
| local_backtest legacy alias | 本项目历史审计名和兼容别名 | CR-051 | 不批量重写旧 CR / process / handoff 证据 |
| research archive | 研究协议、运行证据、报告摘要、artifact 指针和归档 manifest 的外部归档域 | CR-051 | 不等于 Git 仓库，也不等于 market data lake |
| hot / warm / cold archive | 当前硬件下的冷热分层：研究主机 2T SSD、NAS 512G SSD、NAS 4T RAID、NAS 14T HDD | CR-051 | 交易主机 512G SSD 只消费 package，不做研究归档主机 |
| delivery_candidate | 研究输出满足策略交付合同所需证据，可进入后续策略交付 CR | CR-051 / CR-046 | 不等于 runtime_candidate 或 trade-ready |
| runtime_candidate | 已具备后续 runtime 验证条件的候选 | CR-051 | 需要独立 CR、授权和运行证据 |

## 领域对象

| Object ID | 对象 | Owner Feature | 关键字段 / 属性 | 状态 | 规则来源 |
|---|---|---|---|---|---|
| OBJ-01 | MarketDataRun | FEAT-02 | run_id、dataset、source、interface、date_range、lake_root、authorization_ref | planned / running / success / partial_success / failed / blocked | REQ-088..137、ADR-048..066 |
| OBJ-02 | ManifestBatch | FEAT-02 | batch_id、idempotency_key、raw_path、checksum、attempts、error_enum | pending / running / success / skipped / failed / resume_conflict | ADR-005、ADR-051 |
| OBJ-03 | DatasetCandidate | FEAT-02 | dataset、schema_version、coverage、lineage、quality_status、readiness_status | candidate / validated / rejected | CR-014 / CR-018 |
| OBJ-04 | CatalogCurrentTruth | FEAT-02 | dataset、release_id、current_pointer、published_at、rollback_target | unpublished / published / rolled_back | ADR-048、ADR-052、ADR-064 |
| OBJ-05 | ClaimBoundary | FEAT-02 / FEAT-07 | allowed_claims、blocked_claims、required_missing、release_condition | open / allowed / blocked / waived | CR-013..030 |
| OBJ-06 | AdjustmentView | FEAT-02 | prices_raw、adj_factor、prices_qfq、prices_hfq、returns_adjusted、as_of_trade_date | raw / derived / published / blocked | ADR-053、ADR-054 |
| OBJ-07 | ResearchDataset | FEAT-01 / FEAT-03 | research_input_v1、benchmark、universe、quality、lineage、blocked_claims | available / unavailable / required_missing | ADR-024..029、ADR-079..086 |
| OBJ-08 | FactorSpec | FEAT-03 | factor_id、direction、lookback、source_fields、preprocess、version | draft / valid / invalid | CR-030 |
| OBJ-09 | FactorRunSpec | FEAT-03 | run_id、factor_ids、dataset_release、label_window、cost_config、random_seed | planned / executed / failed | CR-030 |
| OBJ-10 | FactorPanelContract | FEAT-03 | raw、directional、winsorized、zscore、coverage、filter_reason | complete / incomplete / blocked | CR-011 / CR-030 |
| OBJ-11 | LabelWindowSpec | FEAT-03 | horizon、decision_time、available_at_rule、leakage_policy | valid / leakage_blocked | CR-030 |
| OBJ-12 | FactorEvaluationReport | FEAT-03 | IC、RankIC、ICIR、layered_returns、turnover、cost_sensitivity | pass / warn / fail | CR-030 |
| OBJ-13 | ExperimentManifest | FEAT-03 | config_hash、dataset_release、factor_version、code_version、report_path | recorded / incomplete | CR-030 |
| OBJ-14 | StrategyAdmissionPackage | FEAT-03 / FEAT-07 | gate_results、blocked_reasons、order_intent_draft_ref、no_real_op_counters | draft / pass / blocked / follow_up_required | CR-019 / CR-030 |
| OBJ-15 | SemanticDiffReport | FEAT-04 | lightweight_result、optional_backend_result、diff_reason、fallback_status | generated / backend_unavailable / blocked | CR-025 |
| OBJ-16 | OrderIntentDraft | FEAT-04 / FEAT-06 | strategy_id、target_trade_date、research_adjustment_policy、execution_price_policy | draft / invalid / handed_off | CR-025 / CR-030 |
| OBJ-17 | PairingToken | FEAT-05 | token_id、scope、nonce、approved_by、expires_at、redaction_status | requested / approved / completed / revoked / expired | CR-019 / CR-020 |
| OBJ-18 | QMTGatewaySession | FEAT-05 | bind_host、port、run_mode、session_ready、capabilities、redaction | not_started / healthy / session_ready / blocked | CR-019 / CR-020 |
| OBJ-19 | QueryPositionsResult | FEAT-05 | account_label、positions_count、readonly_scope、blocked_reason | available / blocked / unavailable | CR-020 |
| OBJ-20 | OMSOrder | FEAT-06 | order_id、intent_id、state、broker_ref、risk_result、idempotency_key | pending / accepted / partially_filled / filled / cancel_pending / canceled / rejected / failed / unknown / timeout | CR-015 / CR-016 |
| OBJ-21 | BrokerLakeRecord | FEAT-06 | run_id、strategy_id、event_type、redacted_account_label、retention_policy | planned / dry_run / written / blocked | CR-015 / CR-016 |
| OBJ-22 | StageGate | FEAT-06 / FEAT-07 | stage、entry_criteria、exit_criteria、rollback、authorization_ref | shadow / simulation / live_readonly / small_live / scale_up / blocked | CR-016 |
| OBJ-23 | AuthorizationRecord | FEAT-07 | authorization_id、scope、actor、approved_at、expires_at、operation_allowlist | missing / active / expired / revoked | 全部高风险 CR |
| OBJ-24 | NoRealOpCounter | FEAT-07 | provider_fetch、lake_write、catalog_publish、qmt_api_call、real_order、credential_read | zero / nonzero_authorized / violation | CR-014..030 |
| OBJ-25 | Runbook | FEAT-08 | runbook_id、scope、operator_steps、not_authorized_items、evidence_required | draft / active / stale | docs/QMT-*.md |
| OBJ-26 | StrategyCoreContract | FEAT-09 | strategy_id、input_schema、target_portfolio_schema、order_intent_schema、risk_assumption、evidence_required | draft / validated / blocked | CR-046 |
| OBJ-27 | StrategyPackageContract | FEAT-09 | package_id、layout_version、targets、validation_suite、docs_bundle、authorization_boundary | draft / review_ready / approved / blocked | CR-046 |
| OBJ-28 | QMTTerminalTargetContract | FEAT-09 | entry_file、config_schema、import_steps、shadow_report_schema、manual_evidence_required | draft / validated / runtime_deferred | CR-046 |
| OBJ-29 | MiniQMTRunnerTargetContract | FEAT-09 | install_layout、uv_python_version、dependency_policy、start_stop_contract、log_paths、kill_switch | draft / install_dry_run_ready / runtime_deferred | CR-046 |
| OBJ-30 | StrategyValidationEvidence | FEAT-09 / FEAT-07 | fixture_result、schema_result、static_guardrail_result、dry_run_plan、manual_validation_plan | missing / partial / pass / blocked | CR-046 |
| OBJ-31 | RunnerInstallPlan | FEAT-09 / FEAT-08 | windows_root、venv_policy、upgrade_plan、rollback_plan、uninstall_plan、redaction_policy | draft / dry_run_reviewed / blocked | CR-046 |
| OBJ-32 | FollowUpStrategyDeliveryGate | FEAT-09 / FEAT-07 | candidate_cr、required_strategy_selection、runtime_authorization_required、evidence_prerequisites | open / ready_for_cr / blocked | CR047 / CR049 / CR051 candidates |
| OBJ-33 | InformationSource | FEAT-10 | source_id、source_type、title、url_or_ref、license_note、credibility、captured_at | captured / triaged / rejected / archived | CR-051 |
| OBJ-34 | StrategyIdea | FEAT-10 | idea_id、hypothesis、source_refs、market_scope、expected_signal、risk_note | captured / triaged / chartered / rejected | CR-051 |
| OBJ-35 | ResearchProject | FEAT-10 | project_id、strategy_family、owner、protocol_ref、archive_root_ref、status | chartered / running / validation_ready / closed / retired | CR-051 |
| OBJ-36 | ResearchProtocol | FEAT-10 | protocol_id、universe、data_release_ref、metric_suite、cost_model、risk_assumptions | draft / protocol_ready / superseded | CR-051 |
| OBJ-37 | ResearchRun | FEAT-10 / FEAT-03 | run_id、commit、data_release、config_hash、seed、artifact_refs、archive_manifest_ref | planned / running / completed / failed / archived | CR-051 |
| OBJ-38 | ValidationEvidence | FEAT-10 / FEAT-07 | evidence_id、run_refs、metrics、bias_checks、claim_boundary、runtime_claim_level | missing / partial / pass / blocked | CR-051 |
| OBJ-39 | ResearchArchiveManifest | FEAT-10 | manifest_id、storage_tier、archive_root、artifact_refs、checksum、retention_policy | draft / indexed / archived / stale | CR-051 |
| OBJ-40 | StrategyTaxonomyEntry | FEAT-10 | taxonomy_id、strategy_family、timeframe、data_dependency、execution_dependency、risk_class | draft / active / deprecated | CR-051 |
| OBJ-41 | ProjectIdentity | FEAT-10 | canonical_name、legacy_aliases、repo_name_target、package_name_target、doc_alias_policy | local_backtest_legacy / quant_lab_canonical / alias_verified | CR-051 |
| OBJ-42 | MigrationInventory | FEAT-10 / FEAT-08 | inventory_id、path_class、owner_feature、move_action、verification_rule、rollback_ref | draft / inventory_ready / move_ready / verified | CR-051 |
| OBJ-43 | ArchivePointer | FEAT-10 | pointer_id、logical_type、external_root_ref、relative_path、checksum、redaction_status | planned / available / missing / redacted | CR-051 |

## 状态机

| State Machine ID | 对象 | 状态 | 合法转换 | 非法转换处理 |
|---|---|---|---|---|
| SM-01 | DatasetCandidate -> CatalogCurrentTruth | candidate -> validated -> published -> rolled_back | 只能由 validate / publish / rollback 触发 | validate pass 自动 current pointer 更新必须 blocked |
| SM-02 | ClaimBoundary | required_missing -> blocked_claim -> allowed_claim | 只有补齐证据、quality/readiness pass、授权满足后可 allowed | 缺证据的 allowed claim 必须 fail |
| SM-03 | FactorRunSpec | planned -> executed -> report_recorded | 任一输入 gate 失败则 blocked | 不得生成 admission pass |
| SM-04 | OrderIntentDraft | draft -> validated -> handed_off | 只传给 OMS / 后续 CR 审查 | 不得直接提交 QMT / adapter |
| SM-05 | PairingToken | requested -> approved -> completed -> revoked/expired | scope、timestamp、nonce 均需通过 | nonce replay / scope mismatch fail-closed |
| SM-06 | QMTGatewaySession | not_started -> healthy -> session_ready -> readonly_query_available | 需要本地启动、HMAC、QMT login/session ready | health pass 不得升级 simulation/live |
| SM-07 | OMSOrder | pending -> accepted -> partially_filled -> filled/cancel_pending/canceled/rejected/failed/unknown/timeout | 事件必须可审计 | unknown / timeout 不得静默成功 |
| SM-08 | StageGate | shadow -> simulation -> live_readonly -> small_live -> scale_up | 必须逐阶段满足 entry/exit 和 authorization | 跳阶段请求 blocked |
| SM-09 | AuthorizationRecord | missing -> active -> expired/revoked | active 需要 scope、actor、时间、操作白名单 | scope 外操作 blocked |
| SM-10 | StrategyPackageContract | draft -> review_ready -> approved -> delivered_by_followup | CP3/CP5/CP8 只批准合同；具体策略交付必须后续 CR | approved 不得自动生成可交易策略包 |
| SM-11 | QMTTerminalTargetContract | draft -> validated -> runtime_deferred | 本 CR 只能到设计和静态验证计划 | terminal shadow / 模拟盘运行请求必须 blocked |
| SM-12 | MiniQMTRunnerTargetContract | draft -> install_dry_run_ready -> runtime_deferred | 本 CR 只能到 install dry-run 设计 | 真实安装、连接、行情订阅或 submit/cancel 必须 blocked |
| SM-13 | StrategyIdea -> ResearchProject -> ValidationEvidence | captured -> triaged -> chartered -> protocol_ready -> research_running -> validation_ready -> admission_review -> research_only / paper_candidate / delivery_candidate / rejected -> packaged -> retired | 只有 protocol、run manifest、validation evidence 和 claim boundary 齐备后才能升级 | delivery_candidate 不得被解释为 runtime_candidate 或 trade-ready |
| SM-14 | MigrationInventory | baseline_archived -> design_approved -> inventory_ready -> mechanical_move_ready -> moved -> externalized -> verified -> released | 每一步必须有 Git 归档点、inventory、验证规则和用户授权 | CP3/CP4/CP5 approval 不得自动触发真实移动 |
| SM-15 | ProjectIdentity | local_backtest_legacy -> quant_lab_canonical -> alias_compatibility_verified | 新文档使用 `quant-lab`；历史证据保留 legacy alias | 批量改写历史 process / handoff / CR 证据必须 blocked |

## 业务规则

| Rule ID | 规则 | Owner | 影响场景 | 验证入口 |
|---|---|---|---|---|
| RULE-01 | 回测、扫描、候选和研究消费主路径不得联网、不得自动补数、不得读取凭据 | FEAT-01 / FEAT-07 | UC-01..UC-09、UC-20..27 | consumer boundary tests |
| RULE-02 | catalog current truth 只能由 Explicit Publish Gate 更新 | FEAT-02 | CR-014 / CR-018 | `tests/test_cr014_catalog_publish_gate.py` |
| RULE-03 | `prices_raw` 和 broker price 是 QMT 执行价格唯一来源，qfq/hfq 只能作为研究 metadata | FEAT-02 / FEAT-06 | CR-017 / CR-015 | `tests/test_cr017_adjustment_leakage_gates.py` |
| RULE-04 | Factor / Label / Report 任一 production gate 未通过时，StrategyAdmissionPackage 不得 pass | FEAT-03 | CR-030 | `tests/test_cr030_strategy_admission_package.py` |
| RULE-05 | Backtrader / Qlib / external projects 只能作为参考或后续 Spike，不默认成为依赖或 truth | FEAT-04 / FEAT-07 | CR-025 / CR-030 | external reference guardrail tests |
| RULE-06 | Linux C 侧不得导入 xtquant，只能通过受控 REST client 调 Windows gateway | FEAT-05 | CR-019 / CR-020 | QMT client tests |
| RULE-07 | gateway health / capabilities / endpoint matrix 可见不等于真实 QMT / simulation / live 授权 | FEAT-05 / FEAT-07 | CR-019 / CR-020 | run gate and docs tests |
| RULE-08 | pre-trade risk 失败时 adapter 调用次数必须为 0 | FEAT-06 | CR-015 / CR-016 | risk gate tests |
| RULE-09 | CP2 / CP3 / CP5 / CP8 approved、Story verified、runbook 存在均不得隐式授权真实运行 | FEAT-07 / FEAT-08 | 全部高风险 CR | no-real-operation safety tests |
| RULE-10 | 文档必须同时说明已验证能力和不授权项 | FEAT-08 / FEAT-07 | README、USER-MANUAL、QMT runbook | docs guardrail tests |
| RULE-11 | 策略核心合同不得导入或直接调用 QMT / XtQuant / MiniQMT API | FEAT-09 / FEAT-07 | CR-046、CR047-candidate | static guardrail |
| RULE-12 | QMT terminal target 本 CR 只能定义策略入口、配置、导入步骤和 shadow 证据格式，不执行终端运行 | FEAT-09 | CR-046 | CP3/CP5/CP7 evidence review |
| RULE-13 | MiniQMT runner target 本 CR 只能定义安装设计、uv 隔离、启动/停止合同、日志和 kill switch，不真实安装或连接 | FEAT-09 / FEAT-07 | CR-046 | install dry-run design review |
| RULE-14 | 策略包验证证据必须区分 fixture/static/design pass 与 runtime verified | FEAT-09 / FEAT-08 | CR-046、后续 CR047 | docs guardrail |
| RULE-15 | CR047 首个具体策略交付必须消费 CR046 策略包合同，不得在 CR046 内提前交付 | FEAT-09 | CR047-candidate | follow-up gate |
| RULE-16 | CR051 研究框架完善必须反向消费 StrategyCoreContract 和 StrategyValidationEvidence，不得扩大 CR046 范围 | FEAT-03 / FEAT-09 | CR051-candidate | follow-up tracking |
| RULE-17 | Git 仓库不得存放真实大体量行情、broker facts、凭据、账户标识或未脱敏运行日志 | FEAT-10 / FEAT-07 | CR-051、迁移 Story | repository guardrail |
| RULE-18 | ResearchRun / RunManifest 必须记录 commit、data release、config hash、seed、artifact refs 和 archive manifest ref | FEAT-10 | CR-051、CR052 | manifest validation |
| RULE-19 | `delivery_candidate` 只表示研究交付候选，不等于 runtime_candidate、simulation-ready 或 live-ready | FEAT-10 / FEAT-07 | CR-051..CR056 | claim boundary review |
| RULE-20 | 交易主机 512G SSD 只消费 strategy package / runner bundle，不作为研究开发或 archive 主机 | FEAT-10 / FEAT-07 | CR-051、后续 runner CR | migration plan review |
| RULE-21 | `quant-lab` 是 canonical 项目名；`local_backtest` 是 legacy alias，不批量重写历史审计材料 | FEAT-10 / FEAT-08 | CR-051 migration | alias compatibility review |
| RULE-22 | CP3 / CP4 / CP5 设计通过不授权目录重命名、NAS 操作、外部 archive 搬迁、远端仓库改名或 git push | FEAT-10 / FEAT-07 | CR-051 | no-real-operation safety review |
| RULE-23 | market data lake、research archive 和 broker lake 必须保持三类事实域隔离 | FEAT-02 / FEAT-06 / FEAT-10 | CR-051、后续迁移 | dependency / archive manifest review |

## CR138 增量：Runner / QMT Gateway 领域模型

### 术语增量

| Term | 定义 | 来源 | 备注 |
|---|---|---|---|
| Runner Control Plane | 负责策略运行计划、盘前确认、执行跟踪、事件/信号接入、再平衡、证据查询、复盘、异常恢复和策略生命周期变更的运营控制面 | CR-138 / UC-33..UC-43 | 不等于 offline `strategy-runner-core` implementation authority |
| QMT Gateway Service Layer | 负责 QMT / MiniQMT / XtQuant 服务生命周期、查询、订阅、订单命令门、回报、恢复、审计和配置变更的服务层 | CR-138 / UC-44..UC-50 | 不等于 QMT terminal target 或 readonly-only endpoint |
| GatewayHealth | Gateway 的可用性、会话、capabilities、degraded reason 和恢复状态 | UC-44 / UC-48 | health pass 不等于 runtime authorization |
| ExecutionReport | Gateway 标准化后的订单状态、成交、拒单、撤单和错误事件 | UC-47 | 不得在无授权时由真实 adapter 产生 |
| RunnerCommand | Runner 对 Gateway 或 OMS 发出的领域命令意图 | UC-35..UC-38 | 必须带 idempotency_key 和 authorization_ref |
| Reference / Account Query Service | Gateway 内负责交易日历、佣金 / 费用模型、收益 / PnL、账户、持仓、订单、成交等查询的服务能力 | UC-34 / UC-41 / UC-45 | 交易日历可走本地参考数据；账户级查询必须授权和脱敏 |

### 领域对象增量

| Object ID | 对象 | Owner Feature | 关键字段 / 属性 | 状态 | 规则来源 |
|---|---|---|---|---|---|
| OBJ-44 | RunPlan | FEAT-11 | run_id、strategy_id、strategy_version、data_release、target_date、mode_request、authorization_ref | draft / preflight_ready / blocked / approved_for_manual_review | UC-33 / UC-34 |
| OBJ-45 | PreflightResult | FEAT-11 / FEAT-07 | run_id、admission_status、data_status、gateway_health_ref、auth_status、blocked_reasons | pass / warn / blocked / manual_review | UC-34 |
| OBJ-46 | RunnerCommand | FEAT-11 | command_id、run_id、strategy_id、command_type、idempotency_key、authorization_ref、audit_id | accepted / duplicate / rejected / blocked | UC-35..UC-38 |
| OBJ-47 | RunState | FEAT-11 | run_id、state、active_orders、last_event_at、gateway_status、incident_refs | planned / preflight / running / degraded / paused / completed / blocked / manual_takeover | UC-35 / UC-39 / UC-42 |
| OBJ-48 | RunEvidence | FEAT-11 / FEAT-08 | run_id、evidence_refs、audit_ids、redaction_status、summary | indexed / redacted / unavailable / blocked | UC-40 / UC-49 |
| OBJ-49 | ReviewSummary | FEAT-11 / FEAT-08 | run_id、trade_date、metrics_summary、incidents、follow_up_candidates | draft / reviewed / follow_up_created | UC-41 |
| OBJ-50 | StrategyChangePlan | FEAT-11 | strategy_id、change_type、parameter_diff、version_diff、rollback_target、approval_ref | draft / approved / applied_by_followup / blocked | UC-43 |
| OBJ-51 | GatewayHealth | FEAT-12 | gateway_id、status、capabilities_ref、session_state、degraded_reason、last_heartbeat | not_started / healthy / degraded / cooldown / unavailable / disabled | UC-44 / UC-48 |
| OBJ-52 | CapabilitySnapshot | FEAT-12 | gateway_id、protocols、query_scopes、market_scopes、order_scopes、runtime_mode | discovered / stale / blocked | UC-44 |
| OBJ-53 | TradingSession | FEAT-12 | session_id、account_label、run_mode、scope、started_at、expires_at、redaction_status | missing / opening / ready / expired / revoked / blocked | UC-45 / UC-47 |
| OBJ-54 | ReadonlyQueryResult | FEAT-12 | request_id、scope、account_label、snapshot_time、summary、redaction_status | available / blocked / unavailable / stale | UC-45 |
| OBJ-55 | MarketSubscription | FEAT-12 | subscription_id、symbols、period、mode、cache_policy、recovery_state | requested / active / degraded / recovering / stopped / blocked | UC-46 |
| OBJ-56 | GatewayCommand | FEAT-12 / FEAT-06 | command_id、command_type、order_intent_id、scope、idempotency_key、risk_ref | accepted / hard_rejected / sent_by_future_runtime / blocked | UC-47 |
| OBJ-57 | ExecutionReport | FEAT-12 / FEAT-06 | execution_report_id、order_intent_id、broker_order_ref、state、filled_qty、error_code、audit_id | ack / partial / filled / canceled / rejected / unknown / stale | UC-47 / UC-48 |
| OBJ-58 | GatewayAuditRecord | FEAT-12 / FEAT-07 | audit_id、request_id、run_id、scope、redaction_status、event_refs | recorded / redacted / blocked | UC-49 |
| OBJ-59 | GatewayChangePlan | FEAT-12 / FEAT-08 | change_id、config_diff、protocol_diff、auth_diff、compatibility_result、rollback_target | draft / review_ready / approved_by_followup / rejected | UC-50 |
| OBJ-60 | TradingCalendar | FEAT-12 | market、date_range、trading_days、holidays、source、source_freshness | available / stale / unavailable | UC-34 / UC-45 |
| OBJ-61 | TradingWindow | FEAT-12 | trade_date、market、pre_open、open_sessions、close_time、after_hours、timezone | open / closed / pre_open / post_close / unknown | UC-33 / UC-34 / UC-45 |
| OBJ-62 | CommissionSchedule | FEAT-12 / FEAT-06 | account_label、instrument_type、commission_rate、min_fee、stamp_tax、transfer_fee、source、effective_from | configured / broker_confirmed / estimated / unavailable | UC-38 / UC-41 / UC-45 |
| OBJ-63 | CostEstimate | FEAT-06 / FEAT-12 | order_intent_id、notional、estimated_commission、taxes、slippage_assumption、source | estimated / unavailable / superseded | UC-38 / UC-41 |
| OBJ-64 | PnLSnapshot | FEAT-12 / FEAT-06 | account_label、run_id、period、realized_pnl、unrealized_pnl、cash_delta、positions_value、source、redaction_status | available / estimated / blocked / stale / unavailable | UC-39 / UC-41 / UC-45 |
| OBJ-65 | ReturnSummary | FEAT-11 / FEAT-12 | run_id、strategy_id、period、gross_return、net_return、benchmark_ref、cost_ref、pnl_ref | draft / reviewed / blocked | UC-41 |

### 状态机增量

| State Machine ID | 对象 | 状态 | 合法转换 | 非法转换处理 |
|---|---|---|---|---|
| SM-16 | RunPlan / PreflightResult | draft -> preflight_ready -> approved_for_manual_review / blocked | admission、data、auth、GatewayHealth 均可检查后转换 | 缺授权或关键输入时 blocked，不生成 runtime action |
| SM-17 | RunState | planned -> preflight -> running -> degraded/paused/completed/manual_takeover | 只能由 RunnerCommand、GatewayEvent、ExecutionReport 或人工动作推进 | unknown order / stale report 不得静默 completed |
| SM-18 | GatewayHealth | not_started -> healthy -> degraded/cooldown/unavailable/disabled -> healthy | health probe、heartbeat、recovery plan 或人工确认触发 | degraded 状态不得自动解除 kill switch 或重发订单 |
| SM-19 | MarketSubscription | requested -> active -> degraded -> recovering -> active/stopped | 授权、capabilities、订阅恢复通过后转换 | 无授权、xtdata 异常或 schema mismatch 时 blocked/degraded |
| SM-20 | GatewayChangePlan | draft -> review_ready -> approved_by_followup/rejected | diff、compatibility、rollback plan 齐备后进入 review | 缺 rollback target 或含明文凭据时 rejected |
| SM-21 | Reference / Account Query | requested -> authorized/local_source_selected -> available/estimated/blocked/unavailable | 本地日历可直接选择 local source；账户佣金 / 收益必须先授权 | 缺授权 blocked；QMT 不支持时 unavailable_with_reason，不得伪造 broker facts |

### 业务规则增量

| Rule ID | 规则 | Owner | 影响场景 | 验证入口 |
|---|---|---|---|---|
| RULE-24 | Runner Control Plane 不得导入 xtquant、读取凭据或直接查询账户 / 行情 / 订单 | FEAT-11 / FEAT-07 | UC-33..UC-43 | CR138 CP5 static guardrail |
| RULE-25 | Gateway health、capabilities、endpoint 可见不等于 account / quote / order / trading 授权 | FEAT-12 / FEAT-07 | UC-44..UC-50 | CP3 / docs guardrail |
| RULE-26 | Runner 与 Gateway 之间所有命令 / 事件必须携带 audit_id 或可追溯 request_id | FEAT-11 / FEAT-12 | UC-35..UC-49 | event schema tests |
| RULE-27 | 未授权 submit/cancel 时 GatewayCommand 必须 hard_rejected / blocked，adapter_calls=0 | FEAT-12 / FEAT-06 / FEAT-07 | UC-47 | no-real-operation tests |
| RULE-28 | Gateway 处于 degraded / unavailable 时 Runner 不得自动重发订单或解除 kill switch | FEAT-11 / FEAT-12 / FEAT-06 | UC-42 / UC-48 | recovery scenario tests |
| RULE-29 | GatewayChangePlan 必须包含 compatibility、auth diff 和 rollback target，且配置不得包含明文凭据 | FEAT-12 / FEAT-08 / FEAT-07 | UC-50 | static validation |
| RULE-30 | 交易日历优先使用本地参考数据；若需要 QMT / vendor 日历，必须记录来源和 freshness，不得把缺失日历推断为交易日 | FEAT-12 / FEAT-07 | UC-34 / UC-45 | calendar fixture |
| RULE-31 | 佣金 / 费用模型必须标注 source=broker_confirmed/configured/estimated/unavailable；QMT 不支持时不得伪造 broker confirmed 结果 | FEAT-12 / FEAT-06 | UC-38 / UC-41 / UC-45 | fee model fixture |
| RULE-32 | 账户级收益 / PnL、资金、持仓、委托、成交查询必须有账户只读授权、脱敏策略和 audit_id；Runner 只消费摘要和引用 | FEAT-11 / FEAT-12 / FEAT-07 | UC-39 / UC-41 / UC-45 | query auth guardrail |

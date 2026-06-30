---
status: "draft-current-index"
version: "1.13"
source_blueprint: "docs/design/BLUEPRINT.md"
change: "CR-139"
companion_hld_cr139: "process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md"
archived_previous: "process/archive/design-blueprints/DOMAIN-MAP-before-quant-lab-project-roadmap-2026-06-26.md"
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
| 1.5 | 2026-06-26 | host-orchestrator | 按 `quant-lab` 项目级五阶段路线增补策略类型适配、Stage 2 多因子框架升级、Stage 3 成熟策略生产和 Stage 4 模拟盘观察领域对象与规则。 |
| 1.6 | 2026-06-27 | codex | 增补统一因子目录领域对象、状态枚举和因子消费规则；mature runner / Chapter 消费必须引用目录登记项。 |
| 1.7 | 2026-06-27 | codex | 增补高级多因子模型评估领域对象、报告状态、门禁规则和 mature admission 消费约束。 |
| 1.8 | 2026-06-27 | codex | 收紧 Stage 3 领域目标：MatureStrategyDefinition 只有在高级评估和 mature admission 通过后才能成为 simulation_candidate；blocked 研究运行不得出阶段。 |
| 1.9 | 2026-06-27 | codex | 增补异象发现与异象研究领域对象、四阶段研究状态机和准入规则；无先验逻辑或未通过严格验证的异象不得升级为因子。 |
| 1.10 | 2026-06-27 | codex | 增补自动异象发现领域规则：ControlledAnomalyTemplate、搜索空间审计、多重检验控制、动态因子目录接入和 Stage 3 候选消费。 |
| 1.11 | 2026-06-28 | codex | 增补 ResearchEngineModule、CompatibilityWrapper 和 SharedResearchContract 领域对象；旧 chapter/stage/root 入口只作兼容或归档。 |
| 1.12 | 2026-06-28 | meta-se | 按 CR-139「Strategy Data Foundation」CP3 增补策略生产数据底座领域对象：写侧（PublishedRelease/LineageChecksum/IncrementalAppendPlan/ConfigFactRelease）、读侧（PITAsOfReader/PanelReader/ReadAuditLog/ReadinessGate/DuckDBReadOnlyAdapter）、ML feature 层（FeatureArtifact/FeatureVersionSchema/LabelArtifact/ModelArtifactHash/SplitManifest）、交易审计链（BrokerLakeAuditChain/BrokerEventRunIdLink/RunEvidenceRunIdLink）、配置类事实源（BenchmarkFactSource/CommissionFactSource/UniversePolicyFactSource/RiskPolicyFactSource/PolicyCycleFactSource/PITUniverseConstituentChain）、schema 契约（SchemaEvolutionRule/SchemaContractFreeze）；补状态机 SM-27..31 与业务规则 RULE-54..60。AGA-1/3/5 推荐方案 CP3 已确认 A1/C1/E1。 |
| 1.13 | 2026-06-28 | host-orchestrator | CP3 approved，AGA-1/3/5 确认 A1/C1/E1，pending-cp3 → confirmed-cp3。 |

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
| OBJ-14A | ResearchEngineModule | FEAT-03 | module_name、domain_role、stable_import_path、compatibility_wrappers、shared_contract_refs | active / compatibility_wrapper / archived | research-engine-stable-modules |
| OBJ-14B | SharedResearchContract | FEAT-03 | contract_name、provided_helpers、consumer_modules、schema_impact | active / deprecated / blocked | research-engine-stable-modules |
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
| strategy type adapter | 将某一类策略的研究输出归一为 `SignalSet`、`StrategyCandidate` 和可审计证据索引的适配合同 | Stage 1 | 多因子、事件型、机器学习和规则型策略都必须通过该边界接入 |
| SignalSet | 统一策略信号集合，包含 signal_id、trade_date、universe、score / signal、direction、confidence、available_at 和 lineage | Stage 1 / Stage 2 | 不等于订单，也不等于 target portfolio |
| StrategyCandidate | 经过研究评价但尚未获得 runtime 授权的策略候选 | Stage 1 / Stage 3 | 可进入 mature admission，不等于 simulation-ready 或 live-ready |
| EventSpec | 事件型策略的事件定义、披露时点、available_at、响应窗口和过滤规则 | Stage 1 后续扩展 | 当前只冻结适配边界，具体事件策略另起 CR |
| ModelSpec | 机器学习策略的特征、标签、模型、训练窗口、验证窗口、漂移和泄漏检查定义 | Stage 1 后续扩展 | 当前只冻结适配边界，具体 ML 策略另起 CR |
| mature multifactor strategy | 满足真实数据 lineage、FactorSpec、UniversePolicy、高级评估通过、组合风控、mature admission PASS 和 runner offline 证据的多因子策略 | Stage 3 | 不等于 fixture / alpha_score 注入包；blocked / insufficient_data 研究运行只是失败候选或诊断基线 |
| simulation observation | 对成熟策略在模拟盘中的长期运行观察、异常闭环和 small_live 入口条件评估 | Stage 4 | 必须逐次 runtime authorization，不自动升级 live |
| factor model validation report | 对多因子模型进行 GRS、因子溢价、经济显著性、时间切分、测试资产多样性、样本外、壳价值、做空可行性、政策周期和稳健性评估的统一报告 | factor-model-validation | Mature admission 必须引用；非核心缺口不得伪造成 pass |
| validation gate decision | 单个评估门禁的状态、理由、严重性和指标引用 | factor-model-validation | 状态固定为 pass / pass_with_risk / blocked / not_applicable / insufficient_data |
| anomaly candidate | 由经济理论、行为金融、市场微观结构、财务指标扩展或事件 / 政策假设生成的异象候选 | anomaly-discovery-research | 必须先有先验逻辑，不能只由数据挖掘产生 |
| anomaly research report | 对异象候选执行单变量排序、分组单调性、控制因子 alpha、时间切分、子样本 / A 股可行性、成本和经济逻辑审查的报告 | anomaly-discovery-research | 通过报告才允许进入 AnomalyAdmissionDecision |
| anomaly admission decision | 将异象研究结果分为 reject / watch / alpha_feature_candidate / factor_catalog_candidate 的准入决策 | anomaly-discovery-research | 不等于 QMT-ready、simulation-ready 或 Stage 3 策略通过 |

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
| OBJ-66 | StrategyTypeAdapter | FEAT-13 | adapter_id、strategy_family、input_contract、output_contract、evidence_required、unsupported_reason | draft / validated / blocked / deprecated | Stage 1 |
| OBJ-67 | SignalSet | FEAT-13 | signal_set_id、strategy_family、trade_date、universe_ref、signal_schema、available_at、lineage_ref | draft / complete / blocked / superseded | Stage 1 / Stage 2 |
| OBJ-68 | StrategyCandidate | FEAT-13 / FEAT-14 | candidate_id、strategy_family、signal_set_ref、backtest_ref、risk_ref、evidence_index_ref | research_only / admission_ready / blocked / retired | Stage 1 / Stage 3 |
| OBJ-69 | ResearchEvidenceIndex | FEAT-13 / FEAT-10 | index_id、data_release_ref、run_manifest_ref、metric_refs、lineage_refs、limitations | missing / partial / complete / blocked | Stage 1 / Stage 3 |
| OBJ-70 | MultifactorFrameworkUpgradePlan | FEAT-03 / FEAT-13 | plan_id、contract_changes、schema_changes、fixture_plan、test_plan、data_lake_dependency | draft / implementation_ready / verified / blocked | Stage 2 |
| OBJ-71 | UniversePolicy | FEAT-14 / FEAT-03 | universe_id、market_scope、PIT_rule、listing_filter、ST_filter、suspension_filter、limit_filter、liquidity_filter、industry_style_refs | draft / validated / blocked / superseded | Stage 3 |
| OBJ-72 | PortfolioRiskPolicy | FEAT-14 / FEAT-06 | policy_id、top_n、max_weight、turnover_limit、industry_limit、style_limit、capacity_assumption、fee_slippage_ref、stop_conditions | draft / validated / blocked / superseded | Stage 3 / Stage 4 |
| OBJ-73 | MatureStrategyDefinition | FEAT-14 | strategy_id、benchmark、holding_period、rebalance_rule、factor_refs、universe_policy_ref、risk_policy_ref、factor_model_validation_report_ref、admission_package_ref、runner_preflight_ref | draft / admission_ready / evaluation_passed / simulation_candidate / blocked | Stage 3 |
| OBJ-74 | SimulationObservationPlan | FEAT-14 / FEAT-11 | observation_id、strategy_id、calendar、run_frequency、metrics、incident_classes、stop_conditions、small_live_entry_criteria | draft / active / paused / completed / blocked | Stage 4 |
| OBJ-75 | ReadinessDecision | FEAT-14 / FEAT-07 | decision_id、scope、evidence_refs、risk_acceptance_refs、authorized_next_stage、non_authorized_items | not_ready / ready_with_risk / ready / blocked | Stage 3 / Stage 4 / Stage 5 |
| OBJ-76 | FactorCatalogEntry | FEAT-03 | factor_id、name、category、family、direction、input_fields、formula、source_refs、status、calculator_status、used_by、evidence_refs、notes | defined / calculable / stage3_active / proxy_only / deprecated / blocked | factor-catalog-cli |
| OBJ-77 | FactorAvailabilityStatus | FEAT-03 | status、meaning、allowed_transitions、blocked_reason | defined / calculable / stage3_active / proxy_only / deprecated / blocked | factor-catalog-cli |
| OBJ-78 | FactorUsageRef | FEAT-03 / FEAT-13 / FEAT-14 | factor_id、consumer、purpose、evidence_ref、run_or_chapter_ref | registered / stale / blocked | factor-catalog-cli |
| OBJ-79 | FactorModelValidationReport | FEAT-03 / FEAT-14 | run_id、schema_version、factor_count、observation_count、overall_status、gate_decisions、risk_warnings、blocked_reasons、evidence_refs、no_real_op_counters | pass / pass_with_risk / blocked / insufficient_data | factor-model-validation |
| OBJ-80 | ValidationGateDecision | FEAT-03 | gate_id、status、reason、severity、metric_refs | pass / pass_with_risk / blocked / not_applicable / insufficient_data | factor-model-validation |
| OBJ-81 | PolicyCycleCoverageSpec | FEAT-03 / FEAT-02 | cycle_id、start_date、end_date、source、coverage_ratio、missing_cycles | configured / covered / insufficient_data / blocked | factor-model-validation |
| OBJ-82 | ShortFeasibilityAssessment | FEAT-03 / FEAT-06 | strategy_type、shortable_coverage、borrow_cost_available、turnover_constraint、constraint_result | not_applicable / pass / pass_with_risk / blocked | factor-model-validation |
| OBJ-83 | TestAssetDiversityAssessment | FEAT-03 | asset_count、industry_count、date_count、concentration_metrics、diversity_result | pass / pass_with_risk / blocked / insufficient_data | factor-model-validation |
| OBJ-84 | OutOfSampleSplitSpec | FEAT-03 | split_method、train_period、test_period、purge_gap、metric_delta、reversal_flag | pass / pass_with_risk / blocked / insufficient_data | factor-model-validation |
| OBJ-85 | AnomalyCandidate | FEAT-03 | anomaly_id、name、source_type、hypothesis、economic_rationale、expected_direction、input_fields、required_factor_ids、formula、prior_logic_ref、a_share_adjustments | proposed / screened / research_ready / rejected / blocked | anomaly-discovery-research |
| OBJ-86 | AnomalyDiscoveryRun | FEAT-03 | run_id、candidate_ids、generation_sources、screening_policy、multiple_testing_policy、data_release_ref、no_real_op_counters | planned / executed / blocked | anomaly-discovery-research |
| OBJ-87 | AnomalyResearchReport | FEAT-03 | run_id、anomaly_id、sorting_t_stat、harvey_pass、monotonicity_score、factor_control_alpha_t、time_split_status、subsample_status、net_return_after_cost、turnover_status、a_share_controls、economic_story_status | pass / watch / rejected / blocked | anomaly-discovery-research |
| OBJ-88 | AnomalyAdmissionDecision | FEAT-03 / FEAT-14 | anomaly_id、decision、reason_codes、research_report_ref、factor_catalog_action、blocked_claims | reject / watch / alpha_feature_candidate / factor_catalog_candidate / blocked | anomaly-discovery-research |
| OBJ-89 | ControlledAnomalyTemplate | FEAT-03 | candidate_id、input_fields、transform、direction、formula、hypothesis、economic_rationale、prior_logic_ref | active / skipped_missing_fields | automatic-anomaly-discovery |
| OBJ-90 | MultipleTestingControl | FEAT-03 | candidate_count、raw_p_value、bonferroni_p_value、fdr_bh_q_value、multiple_testing_pass | pass / fail / insufficient_data | automatic-anomaly-discovery |

### 状态机增量

| State Machine ID | 对象 | 状态 | 合法转换 | 非法转换处理 |
|---|---|---|---|---|
| SM-16 | RunPlan / PreflightResult | draft -> preflight_ready -> approved_for_manual_review / blocked | admission、data、auth、GatewayHealth 均可检查后转换 | 缺授权或关键输入时 blocked，不生成 runtime action |
| SM-17 | RunState | planned -> preflight -> running -> degraded/paused/completed/manual_takeover | 只能由 RunnerCommand、GatewayEvent、ExecutionReport 或人工动作推进 | unknown order / stale report 不得静默 completed |
| SM-18 | GatewayHealth | not_started -> healthy -> degraded/cooldown/unavailable/disabled -> healthy | health probe、heartbeat、recovery plan 或人工确认触发 | degraded 状态不得自动解除 kill switch 或重发订单 |
| SM-19 | MarketSubscription | requested -> active -> degraded -> recovering -> active/stopped | 授权、capabilities、订阅恢复通过后转换 | 无授权、xtdata 异常或 schema mismatch 时 blocked/degraded |
| SM-20 | GatewayChangePlan | draft -> review_ready -> approved_by_followup/rejected | diff、compatibility、rollback plan 齐备后进入 review | 缺 rollback target 或含明文凭据时 rejected |
| SM-21 | Reference / Account Query | requested -> authorized/local_source_selected -> available/estimated/blocked/unavailable | 本地日历可直接选择 local source；账户佣金 / 收益必须先授权 | 缺授权 blocked；QMT 不支持时 unavailable_with_reason，不得伪造 broker facts |
| SM-22 | StrategyTypeAdapter -> SignalSet -> StrategyCandidate | adapter_draft -> adapter_validated -> signal_complete -> candidate_admission_ready / blocked | 只有适配合同、信号 lineage、available_at 和 evidence index 齐备后才能进入 mature admission | 缺 adapter 或 evidence 时 blocked，不允许 runner 直接消费策略类型私有输出 |
| SM-23 | MultifactorFrameworkUpgradePlan | draft -> implementation_ready -> verified -> stage3_ready | Stage 2 只允许用 fixture / schema / static / typed unavailable 验证框架能力 | 尝试连接数据湖、伪造真实 lineage 或把 unavailable 当 pass 时 blocked |
| SM-24 | MatureStrategyDefinition -> SimulationObservationPlan -> ReadinessDecision | admission_ready -> evaluation_passed -> simulation_candidate -> observation_active -> ready_with_risk / ready / blocked -> small_live_candidate | 必须先完成 mature research 真实数据证据、FactorModelValidationReport 核心门禁通过、mature admission PASS 和 runner offline/preflight，再逐次授权进入 Stage 4 runtime | validation / admission blocked 时不得成为 simulation_candidate；simulation pass 不得自动升级 live；异常未闭环时 blocked |
| SM-25 | FactorModelValidationReport -> MatureStrategyAdmission | draft -> evaluated -> pass / pass_with_risk / blocked -> admission_consumed | 只有评估输入、gate_decisions、risk_warnings 和 no-real-op counters 齐备后才能被 mature admission 消费 | 核心门禁 blocked 或报告缺失时 admission blocked；非核心缺口必须显式 risk warning / unavailable |
| SM-26 | AnomalyCandidate -> AnomalyResearchReport -> AnomalyAdmissionDecision | proposed -> screened -> research_ready -> investigated -> alpha_feature_candidate / factor_catalog_candidate / watch / rejected / blocked | 候选必须先完成先验逻辑登记；研究必须覆盖 Harvey t>=3、单调性、控制因子、时间切分、成本、A 股特殊处理和经济逻辑 | 无先验逻辑、数据泄漏、未披露多重检验或核心研究门禁失败时 blocked / rejected，不得写入可计算因子目录 |

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
| RULE-33 | `quant-lab` 项目蓝图是项目级能力边界入口，不得被替换为单一端到端策略研究蓝图 | FEAT-13 / FEAT-08 | Stage 1 | blueprint review |
| RULE-34 | 多因子、事件型、机器学习和规则型策略必须通过 StrategyTypeAdapter 输出统一 `SignalSet` / `StrategyCandidate`，不得让 runner 消费策略类型私有对象 | FEAT-13 / FEAT-14 | Stage 1、后续事件 / ML CR | adapter contract tests |
| RULE-35 | Stage 2 多因子研究框架升级不连接数据湖，不触达 provider / lake / catalog / publish；真实数据缺口必须 typed unavailable | FEAT-03 / FEAT-13 / FEAT-07 | Stage 2 | framework fixture / no-real-op tests |
| RULE-36 | Stage 3 成熟多因子策略必须记录真实输入字段、计算窗口、available_at、PIT universe、lineage、factor panel、label window、IC / RankIC、分层收益、换手、暴露、风险版本、FactorModelValidationReport 和 mature admission 结果 | FEAT-03 / FEAT-14 | Stage 3 | research evidence review |
| RULE-37 | Stage 3 可以在研究机连接数据湖，但必须由用户在研究机实施并记录数据 release、run manifest、config hash 和证据索引；只有评估通过且 mature admission PASS 的策略才能作为 Stage 3 输出 | FEAT-02 / FEAT-03 / FEAT-14 / FEAT-07 | Stage 3 | data lineage gate |
| RULE-38 | Stage 4 模拟盘运行必须逐次 simulation runtime authorization，从 P0 health / identity 开始，不得复用历史授权 | FEAT-11 / FEAT-14 / FEAT-07 | Stage 4 | runtime authorization gate |
| RULE-39 | Stage 4 日常调整只能在观察计划、风险策略和策略变更计划内进行；超出范围必须新建决策或 CR | FEAT-14 / FEAT-11 / FEAT-07 | Stage 4 | observation review |
| RULE-40 | Stage 5 small_live / live 必须独立 live switch CR、独立人工门禁和独立 runtime authorization；simulation readiness 不得自动升级为 live readiness | FEAT-14 / FEAT-07 / FEAT-08 | Stage 5 | live gate |
| RULE-41 | Mature runner、Chapter 3/5/6/7 脚本和成熟策略准入只能消费统一因子目录中的 factor_id；新增因子、mature runner 因子、查询 CLI 或目录 schema 变化必须更新 FactorCatalogEntry 并通过目录测试 | FEAT-03 / FEAT-13 / FEAT-14 | 因子研究、mature multifactor、章节复刻 / 异常代理 | `tests/test_factor_registry.py`、`tests/test_list_factors_cli.py`、`tests/test_stage3_mature_multifactor_research.py` |
| RULE-42 | Mature admission 必须引用 FactorModelValidationReport；核心门禁为因子溢价显著性、经济显著性、样本外验证和数据偏差审计，任一 blocked 时 admission blocked 且 Stage 3 不得出阶段 | FEAT-03 / FEAT-14 | Mature multifactor、成熟策略准入 | `tests/test_factor_model_validation_report.py`、`tests/test_stage3_factor_model_validation_integration.py` |
| RULE-43 | GRS、做空可行性、政策周期覆盖、壳价值控制、测试资产多样性、时间分割和稳健性评估缺少必要输入时必须返回 insufficient_data / not_applicable / risk warning，不得推断或伪造外部事实；若缺口影响核心门禁或 Stage 4 可行性，候选保持 blocked | FEAT-03 / FEAT-07 | 因子模型评估、报告审计、后续 Stage4 输入 | `tests/test_factor_model_validation_report.py`、`tests/test_policy_cycle_dataset_contract.py` |
| RULE-44 | 异象发现必须理论 / 行为 / 微观结构 / 财务扩展 / 事件政策假设驱动；无先验逻辑的纯数据挖掘结果不得进入 AnomalyResearchReport pass | FEAT-03 / FEAT-07 | 异象发现、候选生成 | `tests/test_chapter5_anomalies.py` |
| RULE-45 | 异象研究初筛必须报告 High-Low 单变量排序、Harvey t>=3、五分组单调性和样本数；低于阈值只能 watch / reject，不得直接升级为 factor_catalog_candidate | FEAT-03 | 异象初筛、Chapter5 复刻 | `tests/test_chapter5_anomalies.py` |
| RULE-46 | 异象严格验证必须报告控制现有因子后的 alpha、时间切分、子样本 / A 股控制、扣费后收益、换手上限、壳价值控制、做空可行性和政策周期覆盖；缺项必须进入 gap / risk warning | FEAT-03 / FEAT-14 | 异象研究、Stage 3 因子候选 | `tests/test_chapter5_anomalies.py`、`tests/test_stage3_factor_model_validation_integration.py` |
| RULE-47 | 异象经济逻辑必须给出风险故事或行为故事；只有统计显著但无经济解释的异象必须标记为 statistical_artifact_risk，不得进入 Stage 3 因子候选 | FEAT-03 / FEAT-10 | 异象研究归档、因子准入 | research report review |
| RULE-48 | 自动异象发现只能从 ControlledAnomalyTemplate 生成候选；模板必须声明公式、输入字段、方向和经济假设，禁止从收益结果反推新公式或阈值 | FEAT-03 | 自动候选生成、搜索空间审计 | `tests/test_anomaly_candidate_generator.py` |
| RULE-49 | 自动异象发现必须记录本轮 candidate_count 并执行多重检验控制；未通过 Bonferroni 或 BH-FDR 的候选不得成为 `stage3_candidate` | FEAT-03 / FEAT-14 | 自动异象准入、Stage 3 候选搜索 | `tests/test_anomaly_discovery.py` |
| RULE-50 | 自动异象发现通过项只能作为动态 FactorCatalogEntry extras 显式传入 CLI 或 Stage 3 candidate spec 构建，不得自动写入静态 registry、publish catalog 或触发 QMT / simulation / live | FEAT-03 / FEAT-07 | 因子目录、Stage 3 候选消费、安全边界 | `tests/test_anomaly_discovery.py`、`tests/test_factor_registry.py` |
| RULE-51 | 新增研究引擎主实现必须使用领域名模块；`engine/chapter*`、`engine/stage*`、`engine/cr*` 顶层文件不得存在，旧命名材料必须集中在 `docs/legacy/archive/engine/` | FEAT-03 / FEAT-08 | 研究引擎维护、测试引用、文档入口 | `tests/test_script_entrypoint_naming.py` |
| RULE-52 | 新增研究脚本主入口必须放入 `scripts/research/*` 或对应稳定目录；旧 root 实现只能归档到 `scripts/legacy/*` 并由稳定 wrapper 调用 | FEAT-03 / FEAT-07 | CLI 入口、运行手册、归档治理 | `scripts/quality/check_script_entrypoints.py` |
| RULE-53 | 重复的 JSON 序列化、矩阵转换、gate/admission 状态枚举必须优先复用 `engine.serialization`、`engine.factor_research_matrices` 和 `engine.admission_contracts`；不得在新模块私有复制 | FEAT-03 | 共享合同、报告输出、admission gate | compileall / targeted research tests |

## CR-139 增量：策略生产数据底座领域模型

> 来源：CR-139 companion HLD「Strategy Data Foundation」（写侧/读侧分离）+ handoff v0.7 §3 整改 45 项 + REQ-201..249。既有合同闭环非新建（REQ-249）。

### 术语增量

| Term | 定义 | 来源 | 备注 |
|---|---|---|---|
| 写侧 / 读侧分离 | 数据湖按写侧（normalize/publish/lineage/质量门禁）与读侧（PIT reader/panel reader/dedup/published selector/读审计）分层 | CR-139 D2 / AGA-1 | 同属 FEAT-02，companion HLD 分章（A1，CP3 已确认） |
| PIT as-of reader | 读财报/估值按 `available_at <= as_of` 取最新，杜绝前视 | C1 / REQ-204 | 四道 P0 防线之一 |
| panel reader | 统一 `read_panel(datasets, as_of)`，复用 `read_dataset` published 门禁，输出多表 as-of 宽表 | R1 / REQ-214 | 四道 P0 防线之一 |
| published current pointer | catalog 指向已发布 current truth 的指针，validate pass 不自动前移 | V1 / REQ-232 | 四道 P0 防线之一 |
| 读层去重 | 读侧按 source_run_id 取最新版本，保证 `(symbol,trade_date)` 唯一 | C2b / REQ-205 | 四道 P0 防线之一 |
| 读审计 log | reader 读操作审计记录，与 RunEvidenceIndex 同 run-id 贯通 | L3 / REQ-233 | d1 纯新建 |
| run-id 贯通 | 数据 run-id 贯穿 reader→RunEvidenceIndex→broker event | T6 / REQ-242 | 跨 FEAT-02/03/06/11 |
| ML feature 层 | lake `features/` 子层带版本 + schema，feature/label/artifact 审计链 | V3 / REQ-219 / D5 | 保留切换独立 store 条件（DEF-139-01） |
| 配置类事实源 | benchmark/commission/universe·risk policy/政策周期四类配置的事实源，版本化 + release 闭环 | F1-F4 / REQ-236..239 | 复用 V1 published pointer 语义（AGA-5 E1，CP3 已确认） |
| schema 契约冻结 | schema 演进规则 + reader 兼容回退，模拟盘前必过 | V4 / REQ-220 / HIGH3 | Wave2 |
| 整改回归基线 | 整改前后下游因子/回测黄金值快照对比，归因差异 | T7 / REQ-212 | d1 纯新建 |
| 整改对象清册 | 一行命令输出全 dataset 行数/覆盖/重复键/pit/published/lineage | T8 / REQ-213 | d1 纯新建 |

### 领域对象增量

| Object ID | 对象 | Owner Feature | 关键字段 / 属性 | 状态 | 规则来源 |
|---|---|---|---|---|---|
| OBJ-91 | PublishedRelease | FEAT-02 写侧 | release_id、dataset 清单、release_scope、as_of_trade_date、source_run_ids、quality 结果、blocked_claims、rollback_target、approver、approved_at | draft / published / rolled_back | REQ-232 / V1 |
| OBJ-92 | CatalogCurrentPointer | FEAT-02 写侧 | dataset、schema_version、current_release_id、published_at、lineage_checksum | unpublished / published / rolled_back | REQ-232 / M1 |
| OBJ-93 | LineageChecksum | FEAT-02 写侧 | raw_run_id、canonical_run_id、checksum、computed_at | missing / filled | REQ-209 / M2 |
| OBJ-94 | IncrementalAppendPlan | FEAT-02 写侧 | date、dataset、idempotency_key、available_at_stamp、dedup_key、status | planned / appended / skipped_conflict | REQ-243 / L1（既有 `incremental.py:248` 闭环） |
| OBJ-95 | PITAsOfReader | FEAT-02 读侧 | datasets、as_of、available_at_filter、source_run_id | ready / blocked | REQ-204 / C1 |
| OBJ-96 | PanelReader | FEAT-02 读侧 | datasets、as_of、published_gate、join_schema、dedup_policy | ready / blocked | REQ-214 / R1 |
| OBJ-97 | ReadAuditLog | FEAT-02 读侧 | read_id、run_id、dataset、as_of、reader、read_at、published_release_ref | recorded / missing | REQ-233 / L3（d1 新建） |
| OBJ-98 | ReadinessGate | FEAT-02 读侧 | coverage、freshness、pit_status、block_reason | pass / blocked | REQ-234 / L4（既有 `readiness.py:462` 闭环） |
| OBJ-99 | DuckDBReadOnlyAdapter | FEAT-02 读侧 | query、predicates、columns、readonly_enforced | available / unavailable | REQ-216 / R3（D4 只读引擎） |
| OBJ-100 | FeatureArtifact | FEAT-03 | feature_id、version、schema、values_path、lineage_ref、available_at | draft / versioned / deprecated | REQ-219 / V3（D5） |
| OBJ-101 | FeatureVersionSchema | FEAT-03 | feature_id、schema_version、fields、compute_spec、compatibility | draft / frozen / superseded | REQ-219 / V3 |
| OBJ-102 | LabelArtifact | FEAT-03 | label_id、horizon、decision_time、available_at、leakage_policy、values_path | draft / frozen / leakage_blocked | REQ-219 / V3 |
| OBJ-103 | ModelArtifactHash | FEAT-03 | model_id、artifact_hash、dataset_snapshot_ref、split_manifest_ref、code_version | recorded / missing | REQ-222 / E2（既有 StrategyAdmissionPackage 承载） |
| OBJ-104 | SplitManifest | FEAT-03 | split_id、cutoff、train/test/embargo、purge_policy、frozen_at | draft / frozen | REQ-225 / E5（既有 embargo/split 闭环） |
| OBJ-105 | ExperimentManifestClosure | FEAT-03/11 | manifest_id、dataset_snapshot、as_of、split、lineage、产出引用 | recorded / incomplete | REQ-221 / E1（既有 `research_manifest.py:152` 闭环） |
| OBJ-106 | BrokerLakeAuditChain | FEAT-06 | run_id、order/trade/position 事件、audit_link、redaction | planned / written / blocked | REQ-240 / T4（既有 `broker_lake.py` schema 闭环） |
| OBJ-107 | BrokerEventRunIdLink | FEAT-06 | broker_event_id、data_run_id、research_run_id、link_status | linked / broken | REQ-242 / T6 |
| OBJ-108 | RunEvidenceRunIdLink | FEAT-11 | evidence_id、data_run_id、read_audit_ref、link_status | linked / broken | REQ-233/242 / L3/T6（既有 `evidence_index.py:19` 闭环） |
| OBJ-109 | BenchmarkFactSource | FEAT-02 | benchmark_id、version、effective_from、release_id、risk_free_curve_ref | configured / versioned / released | REQ-236 / F1（既有 `benchmarks.py:99/114` 闭环） |
| OBJ-110 | CommissionFactSource | FEAT-12 | commission_id、version、effective_from、release_id、fee_model、slippage | configured / versioned / released | REQ-237 / F2（既有 `qmt_gateway_contracts.py:997` 闭环） |
| OBJ-111 | UniversePolicyFactSource | FEAT-14 | universe_id、version、effective_from、release_id、PIT_rule、退市/ST/容量约束 | configured / versioned / released | REQ-238 / F3（既有 `mature_multifactor_framework.py:228` 闭环） |
| OBJ-112 | RiskPolicyFactSource | FEAT-14 | policy_id、version、effective_from、release_id、top_n、max_weight、turnover_limit | configured / versioned / released | REQ-238 / F3 |
| OBJ-113 | PolicyCycleFactSource | FEAT-03 | cycle_id、version、effective_from、release_id、coverage、shortability | configured / versioned / released | REQ-239 / F4（既有 `factor_model_validation.py:444` 闭环，P2） |
| OBJ-114 | PITUniverseConstituentChain | FEAT-14 | as_of、index_members_snapshot、constituent_chain、survivorship_bias_flag | built / missing | REQ-231 / X4（既有 `contracts.py:270` 闭环） |
| OBJ-115 | ConfigFactRelease | FEAT-02 写侧 | release_id、config_type、version、effective_from、current_pointer | draft / published / rolled_back | REQ-236..239 / F1-F4（AGA-5 E1 复用 V1 pointer 语义） |
| OBJ-116 | SchemaEvolutionRule | FEAT-02/03 | schema_id、change_type（兼容/破坏）、reader_compat_policy、migration_path | draft / approved | REQ-220 / V4 |
| OBJ-117 | SchemaContractFreeze | FEAT-02/03 | freeze_id、frozen_at、scope、reader_fallback、unfreeze_condition | draft / frozen / unfrozen | REQ-220 / V4（HIGH3 模拟盘前必过） |
| OBJ-118 | RemediationBaseline | FEAT-02 | baseline_id、frozen_at、golden_value_snapshot、coverage | frozen / superseded | REQ-212 / T7（d1 新建） |
| OBJ-119 | RemediationInventory | FEAT-02 | inventory_id、dataset、rows、coverage、dup_keys、pit_status、published、lineage_missing | generated / stale | REQ-213 / T8（d1 新建） |

### 状态机增量

| State Machine ID | 对象 | 状态 | 合法转换 | 非法转换处理 |
|---|---|---|---|---|
| SM-27 | DatasetCandidate → PublishedRelease → CatalogCurrentPointer | candidate → validated → published → current_pointer_set → rolled_back | 只能由 validate/publish/rollback 触发 | validate pass 自动前移 current pointer 必须 blocked（RULE-02 强化） |
| SM-28 | ReadAuditLog → RunEvidenceRunIdLink → BrokerEventRunIdLink | read_recorded → run_id_linked → broker_linked / broken | run-id 贯通逐级链接 | 断链标注断点，不伪造贯通 |
| SM-29 | FeatureArtifact 版本化 | draft → versioned → frozen → superseded | schema 变更按 SchemaEvolutionRule | 缺 version/schema 阻断持久化 |
| SM-30 | ConfigFactRelease | draft → published → current_pointer_set → rolled_back | 复用 V1 published pointer 语义 | 配置缺 version 阻断 release |
| SM-31 | SchemaContractFreeze | draft → frozen（模拟盘前）→ unfrozen（仅破坏性变更+重跑） | 模拟盘前必过 HIGH3 | 未冻结不得进模拟盘 |

### 业务规则增量

| Rule ID | 规则 | Owner | 影响场景 | 验证入口 |
|---|---|---|---|---|
| RULE-54 | 读侧只能消费 published current truth；validate pass 不自动前移 current pointer；reader 不得读未发布 candidate | FEAT-02 读侧 | UC-52、四道 P0 防线 | V1/C1/R1 tests |
| RULE-55 | 读层去重按 source_run_id 取最新版本，保证 `(symbol,trade_date)` 唯一；不用 glob-concat 全量分区替代去重 | FEAT-02 读侧 | UC-52、C2 | C2b dedup tests |
| RULE-56 | 数据 run-id 必须贯穿 reader 读审计 → RunEvidenceIndex → broker event；断链标注断点，不伪造贯通 | FEAT-02/11/06 | UC-55、可审计目标 | L3/T6 run-id tests |
| RULE-57 | ML 必须接入 `read_panel_as_of`，废除 `--data-dir`/`load_local_frames` 旁路；feature/label/artifact 必须带版本 + schema + lineage | FEAT-03 | UC-53、R2/V3 | R2/V3 no-bypass tests |
| RULE-58 | 配置类事实源（benchmark/commission/universe·risk policy/政策周期）必须带 version + effective_from + release_id；缺版本阻断复现声明 | FEAT-02/12/14/03 | UC-56、F1-F4 | F1-F4 version tests |
| RULE-59 | schema 契约必须在模拟盘前冻结；reader 必须按 SchemaEvolutionRule 兼容回退；未冻结不得进模拟盘 | FEAT-02/03 | UC-57、HIGH3 | V4 freeze tests |
| RULE-60 | Wave1 任何结构性变更（V1/C1/R1/C2b/N1）必须在 T8 清册 → T7 黄金值基线 → C2a 重复画像三步完成后才允许执行；物理分区迁移后置到基线冻结之后 | FEAT-02 | UC-51/52、REQ-247 | Wave1 baseline gate tests |

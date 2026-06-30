---
checkpoint: CP2
cr_id: CR-139
title: "CP2 需求/场景/范围基线门 — CR-139 Strategy Data Foundation 自动预检（草稿）"
type: auto-precheck
status: draft
created_at: 2026-06-28T00:00:00+08:00
created_by: meta-pm
human_review_status: pending
conclusion: PASS_WITH_RISK
---

# CP2 Requirements Baseline Precheck — CR-139 Strategy Data Foundation

> meta-pm 自动预检草稿。**不写人工审查结果**（由 host-orchestrator 发起 CP2 门禁时回填）。终审由 host-orchestrator 运行后发起 CP2 Decision Brief 人工门禁。

## Entry Criteria

| 项 | 状态 | 证据 |
|---|---|---|
| CR-139 已建立审计边界 | PASS | `process/changes/CR-139-STRATEGY-DATA-FOUNDATION-2026-06-28.md` |
| handoff v0.7 五轮评审定稿 | PASS | `process/context/DATA-LAKE-REMEDIATION-HANDOFF.md` v0.7 |
| CP2 context capsule 存在 | PASS | `process/context/CP2-CR139-STRATEGY-DATA-FOUNDATION-CONTEXT.yaml` |
| 阶段零调研完成 | PASS | `process/discussions/CLARIFICATION-LOG.md` 调研发现段 |
| Scenario Gray Areas 识别并讨论 | PASS | `process/discussions/CP2-CR139-SCENARIO-DISCUSSION-LOG.md` + `process/checks/CP2-CR139-DISCUSSION-CHECKPOINT.json` |
| USE-CASES.md 增量完成 | PASS | `process/USE-CASES.md` v1.17（UC-51..UC-57） |
| REQUIREMENTS.md 增量完成 | PASS | `process/REQUIREMENTS.md` v1.17（REQ-201..REQ-249） |
| SCENARIOS.yaml 完成 | PASS | `process/SCENARIOS-CR139.yaml`（24 SCN） |
| TEST-MATRIX.md 完成 | PASS | `process/TEST-MATRIX-CR139.md` |
| STORY-MAP.md 完成 | PASS | `process/STORY-MAP-CR139.md` |
| MVP-SCOPE.md 完成 | PASS | `process/MVP-SCOPE-CR139.md` |
| RELEASE-SLICES.md 完成 | PASS | `process/RELEASE-SLICES-CR139.md` |
| BACKLOG.md 完成 | PASS | `process/BACKLOG-CR139.md` |

## Checklist

### 1. USE-CASES / REQUIREMENTS 增量完整性

| 项 | 状态 | 说明 |
|---|---|---|
| 旧基线保留（UC-01..50 / REQ-001..200） | PASS | 增量更新，编号不重排，修订记录追加 v1.17 |
| 新增 UC-51..UC-57 覆盖四阶段全流程 | PASS | 信息收集/回测/模拟盘/实盘 + ML feature + run evidence + 配置事实源 + schema 冻结；UC-51..57 已按用户 CP2 后反馈重写为 persona 驱动用户场景（触发条件为真实业务情境，处理逻辑为用户交互叙事 + 场景边界声明，整改项 C1/C2/V1/R1 等作为支撑证据在叙事中自然带出并回链 REQ-201..245，不再用 Given/When/Then 实现验证罗列；UC-01..50 与 frontmatter v1.17 不变，D8a/D8b 决策与范围基线不变） |
| 新增 REQ-201..REQ-245 映射 45 项整改 | PASS | 机械核验：45 REQ 1:1 映射 45 项，无重复，类别 (a)=6/(b)=7/(c)=12/(d1)=6/(d2)=14 一致 |
| 四道 P0 防线标注（C1/C2/V1/R1） | PASS | REQ-204/205/232/214 |
| 跨切约束 REQ-246..249 | PASS | 四组门禁/物理迁移后置/不授权范围/既有合同闭环原则 |
| 每条 REQ 有验收条件 + 来源场景 + Feature + Wave | PASS | REQUIREMENTS.md CR-139 增量节 |
| 风险与假设 + 里程碑建议 | PASS | RA-201..205 + M-CR139-1/2/3 |

### 2. SCENARIOS / TEST-MATRIX 覆盖摘要

| 项 | 状态 | 说明 |
|---|---|---|
| 正向场景 | PASS | 11 个 |
| 负向场景 | PASS | 5 个 |
| 边界场景 | PASS | 3 个 |
| 权限场景 | PASS | 1 个（不授权范围计数） |
| 失败恢复/外部失败 | PASS | 1 个（dry-run 合同未执行降级） |
| precheck 场景 | PASS | 2 个（四组门禁集 + 物理迁移后置） |
| 四道 P0 防线覆盖 | PASS | C1=SCN-002,003；C2=SCN-006,007；V1=SCN-004,005；R1=SCN-004 |
| 需求覆盖统计 | PASS_WITH_RISK | 49 REQ：33 直接覆盖，9 Wave2 planned，7 Wave3 deferred；0 BLOCKING gap |
| empty-data 场景 | N/A | CR-139 无空数据集场景（清册场景覆盖缺失状态） |

### 3. STORY-MAP / MVP-SCOPE 范围取舍

| 项 | 状态 | 说明 |
|---|---|---|
| Story 候选边界（不拆 Story） | PASS | STC-01..22 + N01..06，CP4 由 host-orchestrator+meta-se 拆 |
| Feature 归属 | PASS | FEAT-02/03/06/11/12/14（CP3 由 meta-se 刷新） |
| MVP = Wave1 P0 | PASS | IN-01..08（T8/T7/C2a/V1/C1/R1/C2b/N1 读层优先） |
| Out of Scope 明确 | PASS | Wave2/3/DuckDB 依赖/feature store/物理迁移/runtime 授权 |
| Deferred Ideas | PASS | DEF-139-01..04 |

### 4. D8a / D8b 纳入口径

| 项 | 状态 | 说明 |
|---|---|---|
| D8a（d1 纯新建 6 项）纳入 | PASS_WITH_RISK | 建议批准为 working basis；正式确认路由 CP2 人工门禁 |
| D8b（d2 既有合同闭环 14 项）纳入 | PASS_WITH_RISK | 建议批准为 working basis；既有合同已 grep 核验（15 类）；正式确认路由 CP2 人工门禁 |
| d2 不重复设计原则 | PASS | REQ-249 约束 + handoff §1.6 既有合同表 |
| E1/E2/T4 归 (c) 不计 D8 | PASS | handoff §3.12 口径一致 |

### 5. Scenario Gray Areas

| 项 | 状态 | 说明 |
|---|---|---|
| 4 个灰区识别 | PASS | SGA-139-01..04 |
| 用户选 1-3 个重点讨论 | PASS | SGA-01/02/04 selected，SGA-03 deferred |
| SGQ 交互记录 ≥1 | PASS_WITH_RISK | SGQ-139-01/02（来源 handoff v0.7 五轮评审 trail，interaction_mode=handoff-trail-plus-return-summary，因未提供 AskUserQuestion 工具） |
| 未选项入 Deferred | PASS | DEF-139-01..04 |
| 讨论日志 + checkpoint | PASS | CP2-CR139-SCENARIO-DISCUSSION-LOG.md + CP2-CR139-DISCUSSION-CHECKPOINT.json |

### 6. Deferred Ideas / 认知盲区

| 项 | 状态 | 说明 |
|---|---|---|
| Deferred Ideas 完整 | PASS | feature store 切换/物理迁移/Wave3 P2/存量重命名 |
| 认知盲区补充 | PASS | 既有合同闭环性质（避免"把已有写成无"）、配置类事实源非纯新建、V4 模拟盘前冻结、物理迁移归因 |
| 既有合同清单核验 | PASS | 15 类对象 grep 核验存在（memory `verify-existing-code-before-claiming-absent`） |
| 计数机械核验 | PASS | 脚本逐项点数 6+7+12+6+14=45 无重复（memory `mechanical-recount-with-script`） |

### 7. 变更追溯与文档处理

| 项 | 状态 | 说明 |
|---|---|---|
| USE-CASES/REQUIREMENTS 增量更新 | PASS | 保留旧基线 + 追加修订记录（规则 24） |
| CR 文档处理决策遵守 | PASS | CR-139 §8 文档处理决策表 |
| 不修改 BLUEPRINT/HLD/FEATURE-DESIGN-MATRIX | PASS | CP3 由 meta-se |
| 不修改 CR-139 本身 | PASS | 只读 |
| 不拆 Story / 不写 LLD / 不实现 | PASS | 规则 25 |

### 8. 不授权范围

| 项 | 状态 | 说明 |
|---|---|---|
| runtime/NAS/QMT/trading/provider-lake write | PASS | REQ-248 不授权，计数为 0 |
| 物理分区迁移 | PASS | REQ-247 Wave1 N1 后置 |
| Git remote write | PASS | 不授权 |
| CP2/CP3 通过 ≠ 授权 | PASS | CR-139 §9 |

## Exit Criteria

| 项 | 状态 | 说明 |
|---|---|---|
| BLOCKING 未决项 = 0 | PASS | D8a/D8b 建议批准为 working basis，0 BLOCKING |
| ready_for_design | PASS_WITH_RISK | CP2 人工门禁通过后由 host-orchestrator 设 true；当前 false |
| 四组门禁集落 CP3 门禁 | PASS（CP3） | REQ-246 约束，CP3 由 meta-se 落 |
| SCENARIOS/TEST-MATRIX/STORY-MAP/MVP-SCOPE 就绪 | PASS | 全部产出 |

## Deliverables

| 产物 | 路径 |
|---|---|
| USE-CASES.md v1.17 | `process/USE-CASES.md` |
| REQUIREMENTS.md v1.17 | `process/REQUIREMENTS.md` |
| SCENARIOS-CR139.yaml | `process/SCENARIOS-CR139.yaml` |
| TEST-MATRIX-CR139.md | `process/TEST-MATRIX-CR139.md` |
| STORY-MAP-CR139.md | `process/STORY-MAP-CR139.md` |
| MVP-SCOPE-CR139.md | `process/MVP-SCOPE-CR139.md` |
| RELEASE-SLICES-CR139.md | `process/RELEASE-SLICES-CR139.md` |
| BACKLOG-CR139.md | `process/BACKLOG-CR139.md` |
| CP2 讨论日志 | `process/discussions/CP2-CR139-SCENARIO-DISCUSSION-LOG.md` |
| CP2 讨论 checkpoint | `process/checks/CP2-CR139-DISCUSSION-CHECKPOINT.json` |
| CLARIFICATION-LOG | `process/discussions/CLARIFICATION-LOG.md` |
| CP2 自动预检（本文件） | `process/checks/CP2-CR139-REQUIREMENTS-BASELINE-PRECHECK.md` |

## 结论

**PASS_WITH_RISK**

- 0 BLOCKING 项；所有 CP2 产物已产出；45 项整改完整映射；四道 P0 防线覆盖；既有合同已核验；计数机械核验一致。
- ~~RISK 1：D8a/D8b 为"二轮建议批准"working basis~~ **已解除**：host-orchestrator live SGQ 用户已确认 D8a 纳入、D8b 纳入不重复设计（2026-06-28T15:50:00+08:00，见 discussion log）。
- ~~RISK 2：SGQ 交互证据来源为 handoff v0.7 五轮评审 trail（未提供 AskUserQuestion 工具）~~ **已解除**：host-orchestrator 补一轮 live SGQ 用户可见交互，3 项决策全部采纳推荐方案，0 项需决策修订再发布；`interaction_mode` 升级为 `handoff-trail-plus-live-sgq`。
- RISK 3：49 REQ 中 9 项 Wave2 planned、7 项 Wave3 deferred 无独立 SCN（CP5 落地测试），非 BLOCKING。

## 人工审查结果

> 由 host-orchestrator 发起 CP2 门禁时回填。meta-pm 不填写。

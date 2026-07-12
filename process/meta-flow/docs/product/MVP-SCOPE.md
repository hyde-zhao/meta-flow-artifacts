---
status: baseline
version: "1.3"
created_at: "2026-07-02"
owner: "meta-pm"
cr_ref: "CR-037"
active_change_ref: "CR-046"
source_story_map: "process/docs/product/STORY-MAP.md"
source_requirements: "process/docs/product/REQUIREMENTS.md"
---

# Meta Flow 项目治理与状态强制 MVP 范围

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 | 文档处理方式 |
|---|---|---|---|---|
| 1.0 | 2026-07-02 | meta-pm | 建立本轮实施的 MVP / Out of Scope / Deferred 范围 | 初始化长期可追踪范围基线 |
| 1.2 | 2026-07-11 | meta-pm | 增量加入 CR-046 evidence-integrity MVP、明确不授权项和 CR-163 append-only pilot；保留全部 PG scope ID | 原文档增量更新 |
| 1.3 | 2026-07-12 | meta-pm | CR-046 CP2 scope rework R2：把 compaction 语义保持、通用 post-close correction、机器 audit report、null-provenance dogfooding 和 dispatch 披露纳入既有 IN-EI scope | 原文档增量更新 |

## MVP 目标

本轮 MVP 的目标是先把 meta-flow 的轻量状态入口和项目治理状态边界变成可执行、可校验、可迁移的机制，再用 quant-lab 作为真实样本验证。MVP 不追求新增第二套治理体系，而是在既有 state、CR、ledger、context、gate 和 checker 体系上补齐缺口。

## In Scope

| Scope ID | 范围项 | 优先级 | 对应 Story | 推荐理由 |
|---|---|---|---|---|
| IN-PG-001 | `STATE.current.json` allowlist、字段预算、audit/enforce 灰度 | P0 | ST-PG-001 | 当前已出现 v2 约束失效，必须先阻断继续污染。 |
| IN-PG-002 | 受控 current state 更新 API 和内部写入路径收敛 | P0 | ST-PG-002 | 没有受控入口，schema 规则无法成为真实门禁。 |
| IN-PG-003 | Agent / Skill current state 写契约同步 | P0 | ST-PG-003 | 需要让人和 agent 都知道重型状态应写到何处。 |
| IN-PG-004 | `process/project/` scaffold、`PROJECT.current.json`、`PROJECT-SCALE.yaml` | P1 | ST-PG-004, ST-PG-005 | 长期项目治理需要独立 refs-only 项目状态，不能继续塞入 current state。 |
| IN-PG-005 | capability / feature registry 引用校验 | P1 | ST-PG-006 | 防止 migration 把自由字符串升级成新的隐性命名空间。 |
| IN-PG-006 | impact surface normalization 和 migration report | P1 | ST-PG-007 | 恢复冲突检测语义，避免路径和治理面混写。 |
| IN-PG-007 | ROADMAP-REFRESH result/checker、GATE-LEDGER event、过程归档库内 cascade | P1 | ST-PG-008, ST-PG-009 | 支持 roadmap refresh，同时保持跨仓写入边界清晰。 |
| IN-PG-008 | FU-RF / SP-RF / RA-RF tracking 和 project stale check | P1 | ST-PG-010 | 让 refresh 发现的后续影响可持续追踪。 |
| IN-PG-009 | quant-lab P2 迁移验证 | P2 | ST-PG-011 | 用真实长期项目验证机制完整性，但不混入机制实现 CR。 |
| IN-PG-010 | ledger compaction 命令边界和最小 CLI 契约 | P0.5 | ST-PG-012 | 明确 ledger compact 不复用 state compact，先冻结命令职责边界，完整 retention 策略可在 FEAT-PG-002 设计中细化。 |
| IN-PG-011 | second-system guardrail | P0 | ST-PG-013 | 所有 CR-037 子能力必须复用现有 state/context/result/ledger/registry 体系，新增平行机制必须被设计审查阻断。 |
| IN-EI-001 | Gate temporal invariant 与条件式批准重放 | P0 | ST-EI-001 | 关闭“schema PASS 但实际时序不可信”的核心缺口。 |
| IN-EI-002 | Dispatch receipt/attempt/retry/supersession/terminal closure contract，以及 compaction/restore 语义保持和 session/repository 证据层级披露 | P0 | ST-EI-002 | 平台调用必须可证明；平台不提供 receipt 时只能诚实披露 `session-observed/repository-unverifiable` 或 unavailable；压缩不得混淆标识或丢失 attempt chain。 |
| IN-EI-003 | CP check attempt、输入 hash、final correlation 与跨真相源一致性 | P0 | ST-EI-003 | 让最终结论关联真实执行并可检测 result/checkpoint/gate/state/ledger 冲突。 |
| IN-EI-004 | delivered finalization、workflow health、authorized read expansion 与 provenance-bearing machine audit report | P0 | ST-EI-004 | 防止 delivered 残留 active refs、悬空 health ref 和隐式全文读取，并让 event/attempt/thread/outcome/token 指标可机器复核。 |
| IN-EI-005 | 平台 token telemetry 与 unavailable/proxy 分层 | P0 | ST-EI-005 | 提供可信 workflow 成本，不把估算冒充实测。 |
| IN-EI-006 | checker identity、schema/policy hash、as-executed/current-replay、CR-046 CP1/CP2 null-provenance dogfooding | P0 | ST-EI-006 | 使历史证据在 checker 演进后仍可解释、可复现；缺 provenance 的原始结果不被静默美化。 |
| IN-EI-007 | 可复用 post-close correction lifecycle 与 quant-lab CR-163 append-only migration/replay fixture | P1 | ST-EI-007 | 先建立通用 correction schema/audit trail，再由 pilot 消费并达成 23/23 current replay，同时保持 lineage 业务源码 diff 为 0。 |

## Out of Scope

| Scope ID | 排除项 | 原因 | 影响 |
|---|---|---|---|
| OUT-PG-001 | 第二套上下文治理、影响分析、capability 命名、result 或 ledger 体系 | 已批准方案明确否决，新增体系会扩大维护面 | 复用现有体系并补 checker / schema 缺口 |
| OUT-PG-002 | 跨仓原子事务 | 成本高且授权 / 回滚边界不清 | roadmap refresh 只自动写过程归档库 |
| OUT-PG-003 | 自动修改 quant-lab 发布库代码、测试和正式文档 | 发布库需要独立 CR 和用户授权 | 输出 stale finding / follow-up，由用户后续决策 |
| OUT-PG-004 | 直接修改 `GATE-PROFILES.json` | project scale 只表达默认偏好，不改变全局 gate profile 真相源 | 使用 `PROJECT-SCALE.yaml` |
| OUT-PG-005 | 新增五档项目规模矩阵或 regulated 独立档 | 当前 `lite / standard / full` 与 runtime-high-risk 足够表达 | 需要真实项目证明不足后再开 CR |
| OUT-PG-006 | 在产品阶段定义实现模块、函数签名细节或代码文件 diff | 属于设计 / 实现阶段 | 由后续 HLD / LLD / implementation 处理 |
| OUT-EI-001 | 伪造或追溯补齐历史 platform receipt、签名、token telemetry、checker identity | 会把未知事实伪装为已验证事实 | 缺失项显式 unavailable/unknown，并按证据层级降级 |
| OUT-EI-002 | 修改 quant-lab lineage contract、recorder、producer、consumer 或 admission 业务代码 | pilot 只验证 process evidence migration | 业务源码 diff 必须为 0；若需业务变更另开 CR |
| OUT-EI-003 | credentials、runtime、production write、publish、交易、repository commit/push | 当前产品门不授予这些权限 | 需要独立明确授权和安全/回滚边界 |
| OUT-EI-004 | 把所有 Story QA 合并成单一 batch 结论 | 会丢失 Story 独立结论和路由 | 只允许共享 regression 批量运行，Story verdict 保持独立 |

## Deferred

| ID | 延后项 | 来源 | 延后原因 | 重启条件 |
|---|---|---|---|---|
| DEF-PG-001 | Ledger compaction 高级保留策略扩展 | P0 后段 | MVP 只冻结命令边界和最小 CLI 契约；复杂 retention 窗口、归档层级和索引策略可后续细化 | P0 enforcement 稳定后，ledger 体积或读取成本成为实际瓶颈 |
| DEF-PG-002 | 更细粒度 project scale 或 regulated 档 | SGA-04 | 当前三档足够，避免过早复杂化 | 新项目无法用现有 gate bias 和 high-risk profile 表达 |
| DEF-PG-003 | 跨仓自动修复 stale 发布文档 | SGA-02 | 自动写发布库越权 | 用户批准具体发布库 CR，且有明确回滚策略 |
| DEF-PG-004 | 长期消费非 YAML capability 来源 | SGA-03 | 不利于稳定 checker | 标准 registry 不能表达某些能力状态时重新设计 |
| DEF-EI-001 | 跨平台统一加密签名 receipt | SGA-05 | 平台能力不一致，当前 contract 先表达实际可用性 | 所有目标平台提供稳定签名接口后另行设计 |
| DEF-EI-002 | 基于估算 token 的强制计费/配额门 | SGA-06 | estimate 不能替代平台 measured telemetry | telemetry 覆盖和估算误差模型通过独立验证后重启 |

## 范围边界规则

- P0 必须先于 P1 / P2：没有 current state enforcement，项目治理对象会继续被污染。
- P1 中 project state、registry 归一、impact surface、roadmap refresh 可以分 CR 交付，但必须保留依赖顺序。
- P2 quant-lab 迁移是验证切片，不是机制设计切片；迁移不得自动改 quant-lab 发布库。
- Roadmap refresh 的 `UPDATED_WITH_DOC_IMPACTS` 不等于授权修改发布库，只表示产生 must_check / stale_items / follow_up_candidates。
- 缺失 registry ID、无法解析 impact surface 或发布库 stale 均应进入 blocked finding / follow-up，不得由实现静默补全。
- 缺失 platform receipt 或 token telemetry 不构成虚构值的理由；必须输出 unavailable，并保留可用的较低证据层级。
- CR-163 pilot 必须 append-only；任何历史事件改写或 lineage business-code diff 都应阻断并回退设计/实现门。

## 成功指标

| 指标 ID | 指标名称 | MVP 验收口径 |
|---|---|---|
| MVP-SM-01 | P0 污染阻断 | unknown / oversized current state 写入在 enforce 模式被拒绝。 |
| MVP-SM-02 | 项目状态瘦身 | `STATE.current.json` 仅通过 `project_state_ref` 引用项目级状态，`PROJECT.current.json` 不超过 16KB。 |
| MVP-SM-03 | 引用可解析 | capability / feature refs 缺失注册时不会通过检查，也不会自动生成 ID。 |
| MVP-SM-04 | 影响面可迁移 | 路径类 impact surface 被拆入 `affected_paths`，历史漂移有 migration report。 |
| MVP-SM-05 | refresh 不越权 | Roadmap refresh 自动写入只发生在过程归档库，发布库影响进入 follow-up。 |
| MVP-SM-06 | quant-lab 可验证 | P2 迁移后检查可复跑，发布库无自动写入。 |
| MVP-SM-EI-01 | 时序可信 | chronology negative fixtures 100% rejected；合法条件式批准可重放。 |
| MVP-SM-EI-02 | Attempt 闭环 | 适用 attempt terminal status、input hash 和 final correlation 覆盖率 100%。 |
| MVP-SM-EI-03 | Telemetry 诚实 | measurement status 覆盖率 100%，estimated/unavailable 不标 measured。 |
| MVP-SM-EI-04 | 重放与 pilot | replay provenance 完整或明确 unavailable；CR-163 current replay 23/23 PASS；业务源码 diff 为 0。 |
| MVP-SM-EI-05 | Compaction 与 correction 语义完整 | compact/restore 前后关系图、terminal selection、correction chain、workflow-health refs 语义差异为 0；非法 post-close 原位改写 100% rejected。 |
| MVP-SM-EI-06 | Audit report 与 dogfooding 可信 | machine audit report 逐维计数 100% 正确并携带 provenance/input hashes；CR-046 R1 null-provenance results 在 strict profile 下失败或标 legacy/unavailable。 |

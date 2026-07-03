---
status: baseline
version: "1.0"
created_at: "2026-07-02"
owner: "meta-pm"
cr_ref: "CR-037"
source_story_map: "process/docs/product/STORY-MAP.md"
source_requirements: "process/docs/product/REQUIREMENTS.md"
---

# Meta Flow 项目治理与状态强制 MVP 范围

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 | 文档处理方式 |
|---|---|---|---|---|
| 1.0 | 2026-07-02 | meta-pm | 建立本轮实施的 MVP / Out of Scope / Deferred 范围 | 初始化长期可追踪范围基线 |

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

## Out of Scope

| Scope ID | 排除项 | 原因 | 影响 |
|---|---|---|---|
| OUT-PG-001 | 第二套上下文治理、影响分析、capability 命名、result 或 ledger 体系 | 已批准方案明确否决，新增体系会扩大维护面 | 复用现有体系并补 checker / schema 缺口 |
| OUT-PG-002 | 跨仓原子事务 | 成本高且授权 / 回滚边界不清 | roadmap refresh 只自动写过程归档库 |
| OUT-PG-003 | 自动修改 quant-lab 发布库代码、测试和正式文档 | 发布库需要独立 CR 和用户授权 | 输出 stale finding / follow-up，由用户后续决策 |
| OUT-PG-004 | 直接修改 `GATE-PROFILES.json` | project scale 只表达默认偏好，不改变全局 gate profile 真相源 | 使用 `PROJECT-SCALE.yaml` |
| OUT-PG-005 | 新增五档项目规模矩阵或 regulated 独立档 | 当前 `lite / standard / full` 与 runtime-high-risk 足够表达 | 需要真实项目证明不足后再开 CR |
| OUT-PG-006 | 在产品阶段定义实现模块、函数签名细节或代码文件 diff | 属于设计 / 实现阶段 | 由后续 HLD / LLD / implementation 处理 |

## Deferred

| ID | 延后项 | 来源 | 延后原因 | 重启条件 |
|---|---|---|---|---|
| DEF-PG-001 | Ledger compaction 高级保留策略扩展 | P0 后段 | MVP 只冻结命令边界和最小 CLI 契约；复杂 retention 窗口、归档层级和索引策略可后续细化 | P0 enforcement 稳定后，ledger 体积或读取成本成为实际瓶颈 |
| DEF-PG-002 | 更细粒度 project scale 或 regulated 档 | SGA-04 | 当前三档足够，避免过早复杂化 | 新项目无法用现有 gate bias 和 high-risk profile 表达 |
| DEF-PG-003 | 跨仓自动修复 stale 发布文档 | SGA-02 | 自动写发布库越权 | 用户批准具体发布库 CR，且有明确回滚策略 |
| DEF-PG-004 | 长期消费非 YAML capability 来源 | SGA-03 | 不利于稳定 checker | 标准 registry 不能表达某些能力状态时重新设计 |

## 范围边界规则

- P0 必须先于 P1 / P2：没有 current state enforcement，项目治理对象会继续被污染。
- P1 中 project state、registry 归一、impact surface、roadmap refresh 可以分 CR 交付，但必须保留依赖顺序。
- P2 quant-lab 迁移是验证切片，不是机制设计切片；迁移不得自动改 quant-lab 发布库。
- Roadmap refresh 的 `UPDATED_WITH_DOC_IMPACTS` 不等于授权修改发布库，只表示产生 must_check / stale_items / follow_up_candidates。
- 缺失 registry ID、无法解析 impact surface 或发布库 stale 均应进入 blocked finding / follow-up，不得由实现静默补全。

## 成功指标

| 指标 ID | 指标名称 | MVP 验收口径 |
|---|---|---|
| MVP-SM-01 | P0 污染阻断 | unknown / oversized current state 写入在 enforce 模式被拒绝。 |
| MVP-SM-02 | 项目状态瘦身 | `STATE.current.json` 仅通过 `project_state_ref` 引用项目级状态，`PROJECT.current.json` 不超过 16KB。 |
| MVP-SM-03 | 引用可解析 | capability / feature refs 缺失注册时不会通过检查，也不会自动生成 ID。 |
| MVP-SM-04 | 影响面可迁移 | 路径类 impact surface 被拆入 `affected_paths`，历史漂移有 migration report。 |
| MVP-SM-05 | refresh 不越权 | Roadmap refresh 自动写入只发生在过程归档库，发布库影响进入 follow-up。 |
| MVP-SM-06 | quant-lab 可验证 | P2 迁移后检查可复跑，发布库无自动写入。 |

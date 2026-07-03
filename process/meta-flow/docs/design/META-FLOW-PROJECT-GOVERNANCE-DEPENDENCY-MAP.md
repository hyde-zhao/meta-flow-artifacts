---
status: baseline
version: "1.0"
cr_ref: "CR-037"
source_blueprint: "process/docs/design/META-FLOW-PROJECT-GOVERNANCE-BLUEPRINT.md"
source_domain_map: "process/docs/design/META-FLOW-PROJECT-GOVERNANCE-DOMAIN-MAP.md"
---

# Meta Flow Project Governance Dependency Map

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-02 | meta-se | 建立项目治理整改 Feature 间允许依赖、禁止依赖和循环风险 |

## 1. 依赖关系

| From | To | 依赖类型 | 允许方向 | 原因 | 验证 / 监控 |
|---|---|---|---|---|---|
| FEAT-PG-003 Project State Governance | FEAT-PG-001 Current State Enforcement | read / ref | allowed | `STATE.current.json` 通过 `project_state_ref` 指向 project state；project state 不反写重型字段 | state schema check、project state check |
| FEAT-PG-006 Roadmap Refresh Governance | FEAT-PG-003 Project State Governance | write / event | allowed | refresh 需要更新过程侧 `PROJECT.current.json`、ROADMAP、MILESTONES | roadmap-refresh checker、Gate Ledger event check |
| FEAT-PG-006 Roadmap Refresh Governance | FEAT-PG-007 FU-RF Candidate Support | write | allowed | 发布库陈旧项需要转 follow-up candidates | cr_tracking tests |
| FEAT-PG-006 Roadmap Refresh Governance | FEAT-PG-008 Project Stale Check | read / event | allowed | refresh 结果和 project state 是 stale-check 的输入之一 | stale-check tests |
| FEAT-PG-005 Impact Surface Normalization | FEAT-PG-004 Capability / Feature Registry | read | allowed | migration 必须用 registry 解析 feature/capability refs | capability / feature checks |
| FEAT-PG-009 Quant-lab Migration | FEAT-PG-001 Current State Enforcement | read / write-through | allowed | quant-lab current state 清理必须走 state checker / writer 约束 | quant-lab state check |
| FEAT-PG-009 Quant-lab Migration | FEAT-PG-004 Capability / Feature Registry | read / write process-side registry | allowed | quant-lab capability 来源归一需要标准 registry | capability migration report |
| FEAT-PG-009 Quant-lab Migration | FEAT-PG-005 Impact Surface Normalization | read / write migration report | allowed | quant-lab 历史 CR impact_surface 漂移需要迁移报告 | impact migration report |
| FEAT-PG-009 Quant-lab Migration | FEAT-PG-006 Roadmap Refresh Governance | trigger / read result | allowed | quant-lab roadmap refresh 和 stale report 是迁移验收的一部分 | ROADMAP-REFRESH result |
| FEAT-PG-002 Ledger Compaction | ledgers | read / archive / index | allowed | 压缩只作用于 ledger 和 archive/index，不影响 current state 语义 | event ledger check、archive index check |

## 2. 禁止依赖

| Forbidden ID | From | To | 禁止原因 | 替代路径 | 违反风险 |
|---|---|---|---|---|---|
| FD-PG-001 | 任意 Agent / Skill | 直接写 `process/state/STATE.current.json` | 会绕过 allowlist、预算和写前校验 | 通过受控 CLI、host-orchestrator 状态入口或 `update_current_state()` | current state 再次膨胀，CP/context 门禁失真 |
| FD-PG-002 | FEAT-PG-001 Current State Enforcement | roadmap / deferred / checkpoint 全文 | P0 目标是瘦身，不是搬运重型状态 | 写入 ledger、CP result、summary、index、legacy `STATE.md` 摘要 | 默认读取入口变重，违反 AGENT-SKILL-CONTRACT |
| FD-PG-003 | FEAT-PG-006 Roadmap Refresh Governance | quant-lab 发布库自动写入 | 已确认禁止跨仓原子事务和自动修改发布库 | 输出 `must_check`、`stale_items`、`follow_up_candidates` | 未授权生产写、跨仓撕裂、审计不可追溯 |
| FD-PG-004 | FEAT-PG-005 Impact Surface Normalization | 自由字符串 capability / feature refs | 会继续削弱冲突检测和迁移可信度 | 使用 FEAT-PG-004 registry resolver；缺失则 blocked finding | capability namespace 污染，quant-lab 迁移不可验证 |
| FD-PG-005 | FEAT-PG-006 Roadmap Refresh Governance | CP result checker | ROADMAP-REFRESH 有独立 result type 和 decision 枚举 | `meta-flow check roadmap-refresh` | CP result schema 被过度泛化，检查语义混乱 |
| FD-PG-006 | FEAT-PG-007 FU-RF Candidate Support | `RELEASE-CONTEXT` | roadmap follow-up 不是 release readiness evidence | CR tracking / FU-RF candidate | 发布上下文混入项目治理待办 |
| FD-PG-007 | FEAT-PG-008 Project Stale Check | 自动修复正式 HLD / TEST-STRATEGY / release docs | stale-check 是检测，不是 CR 实施 | 生成 finding 和 FU-RF / CR 候选 | 未经 CR / gate 修改正式产物 |
| FD-PG-008 | FEAT-PG-009 Quant-lab Migration | 机制实现 CR | P2 是真实样本迁移，不与机制实现混交 | 依赖 P0/P1 能力完成后单独迁移 CR | 样本问题反向污染机制边界 |

## 3. 循环风险

| Cycle ID | 涉及对象 | 风险 | 当前处理 |
|---|---|---|---|
| CYCLE-PG-001 | `STATE.current.json` <-> `PROJECT.current.json` | project state 若反向要求 current state 承载 roadmap / milestone，将破坏瘦身 | eliminated：current 只保留 `project_state_ref` |
| CYCLE-PG-002 | Roadmap Refresh <-> Project Stale Check | refresh 触发 stale，stale 又触发 refresh，可能产生无界循环 | accepted with guard：stale-check 只报告，follow-up 需 CR / human gate |
| CYCLE-PG-003 | Impact Migration <-> Capability Registry | migration 需要 registry，registry 又可能依赖 migration 发现 capability | accepted with block：缺 registry 时输出 blocked finding，不自动创造 ID |
| CYCLE-PG-004 | Quant-lab Migration <-> Mechanism Implementation | 用未完成机制迁移真实项目会导致返工，真实样本又可能要求机制变更 | eliminated by phase：P2 在 P0/P1 能力后执行；机制变更另开 CR |

## 4. 建议 Wave 依赖

| Wave | Feature | 必须前置 | 可并行 | 说明 |
|---|---|---|---|---|
| W1 | FEAT-PG-001 | 已批准实施计划 | 无 | P0 gate，后续 Feature 都依赖 current state 约束可信 |
| W2 | FEAT-PG-002、FEAT-PG-003 | FEAT-PG-001 audit 至少可用 | 可并行 | ledger compaction 与 project scaffold 写面不同 |
| W3 | FEAT-PG-004、FEAT-PG-005 | FEAT-PG-003 registry 路径和 project scaffold 明确 | 可并行但 FEAT-PG-005 enforce 依赖 FEAT-PG-004 | registry resolver 是 impact migration 的前提 |
| W4 | FEAT-PG-006、FEAT-PG-007、FEAT-PG-008 | FEAT-PG-003、FEAT-PG-004、FEAT-PG-005 audit 通过 | FEAT-PG-007 可与 FEAT-PG-006 checker 并行；FEAT-PG-008 依赖 refresh result shape | Roadmap refresh 形成项目治理闭环 |
| W5 | FEAT-PG-009 | W1-W4 全部达到验收条件 | 不建议并行 | quant-lab migration 作为真实样本验证 |

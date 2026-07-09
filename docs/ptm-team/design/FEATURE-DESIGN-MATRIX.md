# PTM Team Feature 设计矩阵

> 版本：v1.0 · 更新：2026-06-08 · 覆盖：全部 6 个 Agent

---

## Feature 总览

| Feature ID | Feature 名称 | Agent | 适用性 | lld_policy | 状态 |
|---|---|---|---|---|---|
| F-001 | 三阶段框架（KYM→MFQ→PPDCS） | ptm-tde | required | full-lld | ✅ delivered |
| F-002 | 场景发现与确认 | ptm-tde | required | full-lld | ✅ delivered |
| F-003 | M 分析（单功能拆分） | ptm-tde | required | full-lld | ✅ delivered |
| F-004 | F 分析（耦合分析） | ptm-tde | required | full-lld | ✅ delivered |
| F-005 | Q 分析（质量属性） | ptm-tde | required | full-lld | ✅ delivered |
| F-006 | PPDCS 五方法设计 | ptm-tde | required | full-lld | ✅ delivered |
| F-007 | 覆盖率验证与交付渲染 | ptm-tde | required | full-lld | ✅ delivered |
| F-008 | AskUserQuestion 交互 | ptm-tde | required | technical-note | ✅ delivered |
| F-009 | 原子操作集成 | ptm-tde | required | full-lld | ✅ delivered |
| F-010 | 因子库发现 | ptm-tde | required | full-lld | ✅ delivered |
| F-011 | 测试用例执行 | ptm-te | required | full-lld | ⬜ planned |
| F-012 | Topo 映射与环境管理 | ptm-te | required | full-lld | ⬜ planned |
| F-013 | 原子能力框架 | ptm-tae | required | full-lld | 🔄 in-progress |
| F-014 | 自动化翻译与回归 | ptm-tae | required | full-lld | ⬜ planned |
| F-015 | 公共 Skill 基础设施 | ptm-tae | required | full-lld | 🔄 in-progress |
| F-016 | 测试计划与调度 | ptm-tm | required | full-lld | ⬜ planned |
| F-017 | 需求分析与策略制定 | ptm-tse | required | full-lld | ⬜ planned |
| F-018 | 用例/执行/工具评审 | ptm-tse | required | full-lld | ⬜ planned |
| F-019 | 质量审计与度量 | ptm-qa | required | full-lld | ⬜ planned |

---

## LLD 策略说明

| 级别 | 适用条件 | Agent 分配 |
|---|---|---|
| `full-lld` | 跨模块契约、数据模型、权限、安全、外部接口 | F-001~007, F-009~019 |
| `technical-note` | 低复杂度、纯适配、Story 内技术说明足够 | F-008 |
| `waived` | 明确不需要独立设计文档 | 无 |

---

## Feature 设计文档索引

| Feature ID | DESIGN | TEST-PLAN | TASKS | 状态 |
|---|---|---|---|---|
| F-001~010 | CR 级 LLD（约 25 个 STORY-*-LLD.md） | CP5 checks | CP6 checks | ✅ delivered |
| F-011~012 | — | — | — | ⬜ planned |
| F-013~015 | — | — | — | 🔄 in-progress |
| F-016 | — | — | — | ⬜ planned |
| F-017~018 | — | — | — | ⬜ planned |
| F-019 | — | — | — | ⬜ planned |

---

## 统计

| Agent | Features | delivered | in-progress | planned |
|---|---|---|---|---|
| ptm-tde | 10 | 10 | 0 | 0 |
| ptm-te | 2 | 0 | 0 | 2 |
| ptm-tae | 3 | 0 | 2 | 1 |
| ptm-tm | 1 | 0 | 0 | 1 |
| ptm-tse | 2 | 0 | 0 | 2 |
| ptm-qa | 1 | 0 | 0 | 1 |
| **合计** | **19** | **10** | **2** | **7** |

---

*本矩阵随 Agent 开发进展动态更新。*

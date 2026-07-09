# PTM Team 发布切片

> 版本：v1.0 · 更新：2026-06-08

---

## 切片规划

```
Slice 1 (v1.0)     Slice 2 (v1.1)     Slice 3 (v2.0)     Slice 4 (v3.0)
✅ 已发布            ⬜ 规划中            ⬜ 规划中            ⬜ 规划中
ptm-tde 独立可用    ptm-te 执行就绪     tm+tse+qa 协同     Autopilot 全自动
```

---

## Slice 1 — ptm-tde v1.0 ✅

| 维度 | 内容 |
|---|---|
| 版本 | v1.0 |
| 日期 | 2026-06-06 |
| Agent | ptm-tde |
| CR | CR-010/011/012/013/015/016/017 |
| Stories | 23 |
| 核心能力 | 三阶段框架（KYM→MFQ→PPDCS）、12 步主流程、5 个人工检查点 |
| 发布决策 | READY |

## Slice 2 — ptm-te + tae 增强 ⬜

| 维度 | 内容 |
|---|---|
| 目标版本 | v1.1 |
| Agent | ptm-te、ptm-tae |
| 核心能力 | 用例执行（解析→配置→打流→判定）、自动化翻译、回归 |
| 前置依赖 | ptm-tae 原子能力 + Topo 管理就绪 |
| 预计 Stories | 5-7 |

## Slice 3 — 六 Agent 协同 ⬜

| 维度 | 内容 |
|---|---|
| 目标版本 | v2.0 |
| Agent | ptm-tm、ptm-tse、ptm-qa |
| 核心能力 | 项目调度、需求分析+策略、质量审计 |
| 前置依赖 | Slice 2 闭环跑通 |
| 预计 Stories | 8-12 |

## Slice 4 — Autopilot ⬜

| 维度 | 内容 |
|---|---|
| 目标版本 | v3.0 |
| Agent | 全部 |
| 核心能力 | 全自动运行、异常自处理、人工监督面板 |
| 前置依赖 | Slice 3 六 Agent 协同成熟 |
| 预计 Stories | 5-8 |

---

*切片规划随项目进展动态更新。*

---
feature_id: "CR102-NAS-REAL-PACKAGE-EXCHANGE-AUTHORIZATION"
title: "CR102 NAS Real Package Exchange Authorization Tasks"
status: "ready-for-cp5-review"
source_cr: "CR-102"
checkpoint: "CP5"
created_at: "2026-06-21T06:32:36+08:00"
owner: "host-orchestrator"
---

# CR102 NAS Real Package Exchange Authorization Tasks

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| v0.1 | 2026-06-21 | host-orchestrator | 初版，拆分 CP5 design-only 任务和后续 gate 任务边界。 |

## 1. CP5 Design-only Tasks

| TASK-ID | 任务 | 输入 | 输出 | 文件范围 | 验证入口 |
|---|---|---|---|---|---|
| CR102-T01 | 冻结 authorization matrix | CP3 HLD、CR102 policy | matrix 表，全部操作 false | `DESIGN.md` | CP5 precheck |
| CR102-T02 | 定义 execution plan shell | CP3 DQ、未授权项 | 非执行型阶段模板 | `DESIGN.md` / LLD | CP5 precheck |
| CR102-T03 | 定义 evidence schema | security boundary | allowed / forbidden fields | `DESIGN.md` / LLD / TEST-PLAN | CP5 precheck |
| CR102-T04 | 定义 fail-closed tests | TEST-PLAN | 负向场景 | `TEST-PLAN.md` | CP5 precheck |
| CR102-T05 | 生成 context / checkpoint | CP5 design package | context capsule、precheck、manual gate | `process/context` / `process/checks` / `process/checkpoints` | human-gate check |

## 2. Future Gate Tasks

以下任务不属于当前 CP5 执行范围，仅作为后续门禁候选：

| TASK-ID | 触发条件 | 当前状态 |
|---|---|---|
| CR102-FU01 | 用户提供 path/export/root/mount/identity | blocked |
| CR102-FU02 | 用户授权 check/list/read 等具体操作 | blocked |
| CR102-FU03 | 用户授权 publish/write/delete/copy/pull | high-risk blocked |
| CR102-FU04 | 用户要求 checker / CLI 实现 | requires new CP5 or runtime gate |
| CR102-FU05 | 用户要求真实运行证据 | requires future runtime authorization |

## 3. 明确不执行

- 不写 mount/list/check/copy/publish/pull/delete/read/write 命令。
- 不实现或运行真实 NAS checker。
- 不读取 env、凭据、账户、原始日志。
- 不启动 runtime。
- 不执行交易或 provider/lake/catalog publish。

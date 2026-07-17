---
checkpoint_id: "CP2-CR031"
checkpoint_name: "需求 / 场景 / 范围基线门"
type: "auto_then_manual"
status: "superseded"
owner: "host-orchestrator"
created_at: "2026-07-17T07:35:00Z"
reviewed_by: ""
reviewed_at: ""
approval_source: ""
auto_check_result: "process/checks/CP2-CR031-PRODUCT-BASELINE.result.json"
target:
  phase: "requirement-clarification"
  story_id: ""
  artifacts:
    - "docs/product/USE-CASES.md"
    - "docs/product/REQUIREMENTS.md"
    - "docs/product/SCENARIOS.yaml"
    - "docs/product/TEST-MATRIX.md"
    - "docs/product/STORY-MAP.md"
    - "docs/product/MVP-SCOPE.md"
---

# CP2 人工审查 — CR-031 项目级安装与运行数据隔离基线

> **已撤回，不得审批。** 2026-07-17 的事实核验确认 `/home/hyde/projects/ptm-tse` 已完成项目级安装并已有运行数据；本审查稿错误地把安装实现列为待决范围。CR-031 已重构为运行数据治理，后续将生成替代 CP1/CP2 工件。保留本文件仅用于审计，不代表当前基线。

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP1-CR031-PRODUCT-BASELINE.result.json` | PASS | 0 | 用户旅程、REQ-RA-023..026、成功/阻断场景、范围与 Story 候选已完整回链。 |
| `process/checks/CP2-CR031-PRODUCT-BASELINE.result.json` | PASS | 0 | 默认/显式项目根、运行数据路径、失败保护与清理时序一致；等待人工确认。 |
| `process/checks/CP2-CR031-DISCUSSION-CHECKPOINT.json` | PASS | 0 | SGQ-031-01 已由用户确认，明确默认安装根与 runtime root 语义。 |

## Decision Brief

### 审批者摘要

| 字段 | 内容 |
|---|---|
| 本次确认服务的整体目标 | 让测试架构师把 `ptm-tse` 安装到自己的项目目录后，可独立定位支持文件，并将后续运行数据隔离在该项目的 `<runtime-root>/data/`，不污染 `ptm-team` 源码仓库。 |
| 推荐动作 | `approve`：确认项目级默认安装根、显式目标覆盖、默认 runtime root 等于安装根、禁止源码/全局/CWD 回退，以及“新布局验证成功后才清理源码根 support 副本”的范围基线。 |
| approve 后会发生什么 | 进入 CP3 HLD：决定安装根下私有 `runtime-support` 的精确布局、版本/导入/权限契约、安装器参数、失败清理和回滚；随后才拆 Story、做 LLD 和实现。 |
| approve 不授权什么 | 不授权真实 ITR GET、凭据或 secret 读取、真实问题单/SQLite/快照写入、移动或删除现有 `data/`、外部写入、生产操作、自动 reviewer 动作、发布或 runtime 声明。 |
| 不确认会阻塞什么 | 阻塞 HLD、安装器改造、运行时支持文件迁移、源码根 `data/` 清理、Story 拆解与后续开发验证。 |

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP2-CR031.context.json` |
| capsule 状态 | ready，`meta-flow context check` 已通过 |
| read_profile | `architecture-major`，16,000 token 上限 |
| 默认读取策略 | capsule-first；全文档读取仅在 human-audit 下展开：完整 CR 和场景讨论记录已登记 `READ-EXPANSION-LEDGER`。 |
| 最小输入 | CR-031 summary/index、当前状态、读策略；按需读取产品基线与本 CP2 审查稿。 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---|---:|---:|---|
| 用户直接确认 | `SGQ-031-01` / 场景讨论记录 | complete | 1 | 1 | 安装根、默认 runtime root 和显式覆盖已 resolved-by-user，仍列入以供本门禁正式纳入基线。 |
| 产品范围 | `MVP-SCOPE.md`、`STORY-MAP.md` | complete | 1 | 1 | 源码根 support 清理时序影响范围与回退，因此列为 CP2 scope 决策。 |
| 安全与失败路径 | REQ-RA-024/025、SCN-RA-15 | complete | 1 | 1 | 隐式回退会突破项目数据隔离，列为高风险安全决策。 |
| HLD 细节 | AGQ-031-02 / AGQ-031-03 | deferred to CP3 | 2 | 0 | 私有 support 子目录、导入契约和精确清理实施不改变本次外部路径语义，由 CP3 设计。 |

### 决策分层

| 分类 | 数量 | 处理方式 |
|---|---:|---|
| 必须用户决策 | 1 | 确认 CR-031 的范围基线与源码根 support 的“验证后清理”时序。 |
| 高风险策略确认 | 1 | 接受没有任何隐式全局/源码/CWD 回退，目标无效时直接阻断。 |
| agent 默认处理 | 2 | HLD 决定 `runtime-support` 的私有子目录与安装器实现细节；CP5 决定文件变更和验证 fixture。 |
| 仅审计记录 | 2 | 用户已确认的安装根 / runtime root 语义，以及根目录盘点中不属于本 CR 的缓存和历史备份。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| CP2-DQ-CR031-01 | scope | 是否以用户已确认的项目级安装与运行根语义作为 CR-031 基线？ | 默认安装根为执行 `ptm-team` 安装命令的项目目录；显式目标覆盖；`--runtime-root` 默认安装根；数据只写 `<runtime-root>/data/`。 | 全局用户目录默认；或任意运行时 CWD 默认。 | 推荐路径可预测、项目隔离明确；全局/CWD 方案会产生泄漏和路径漂移。 | 改变安装器、Skill 路径解析与数据权限。 | 仅新的 CR 可改变默认根；HLD 可决定私有 support 子目录，不得改变外部语义。 |
| CP2-DQ-CR031-02 | security | 无效根或 support 缺失时是否拒绝所有回退？ | 是：阻断、清理未提交数据、说明修复方式；不访问 `ptm-team/data`、全局目录或任意 CWD。 | 自动回退到源码或全局目录。 | 推荐能避免跨项目数据泄漏；回退看似方便但会掩盖安装故障并破坏隔离。 | 影响首次运行失败体验和受限存储可靠性。 | 若需兼容旧布局，须独立 CR 定义显式迁移命令，而不是隐式回退。 |
| CP2-DQ-CR031-03 | implementation | 源码根旧 `data/` support 副本何时清理？ | 完成新安装、路径隔离、权限、卸载/回滚验证后清理；不迁移真实运行数据到源码仓库。 | 立即删除；或永久保留为运行时依赖。 | 推荐保留可回退能力且最终清除错误边界；立即删除有中断风险，永久依赖则无法独立安装。 | 影响 `data/schema.sql`、`data/dao.py` 和安装包设计。 | 新布局失败则暂停清理并保留当前副本，只用于修复验证。 |

### 用户需决策事项

请确认上表三项推荐方案：`CP2-DQ-CR031-01`（项目级根语义）、`CP2-DQ-CR031-02`（无隐式回退）和 `CP2-DQ-CR031-03`（验证后清理）。用户此前已直接确认 CP2-DQ-CR031-01 的路径语义；本门禁将三项一起固化为 CR-031 的正式产品基线。

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| CR-031 已创建且冲突预检通过 | PASS | `process/changes/CR-031.md`、CR summary/index | 当前唯一 active CR 是 CR-031。 |
| CP1 产品基线完整性通过 | PASS | `process/checks/CP1-CR031-PRODUCT-BASELINE.result.json` | 新 UC/REQ/SCN/MVP/Story 均有追溯。 |
| 场景灰区已处理 | PASS | `process/discussions/CP2-CR031-SCENARIO-DISCUSSION-LOG.md` | 用户已直接确认默认路径语义。 |
| 真实运行未被授权 | PASS | CR-031 不授权范围 | 本次仅为设计基线确认。 |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | 默认安装根是命令执行的项目目录 | pending human | UC-RA-09、REQ-RA-023 | 已由用户确认，等待纳入正式基线。 |
| 2 | 显式目标覆盖默认安装根 | pending human | UC-RA-09、REQ-RA-023 | 不应再隐式改变全局目录。 |
| 3 | 默认 runtime root 等于项目级安装根 | pending human | UC-RA-09、REQ-RA-024 | 数据路径只允许 `<runtime-root>/data/`。 |
| 4 | 无效根、缺失 support 与源码/全局回退均被阻断 | pending human | REQ-RA-024/025、SCN-RA-15 | 不生成部分 SQLite 或伪造分析。 |
| 5 | 旧 `data/` 仅在新布局验证后清理 | pending human | REQ-RA-026、CP2-DQ-CR031-03 | 当前不移动、不删除。 |
| 6 | 项目级边界不扩大运行授权 | pending human | CR-031 不授权范围 | 不授权任何真实外部访问或数据写入。 |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| 需求 / 场景 / 范围基线已确认 | pending human | 本文件及 CP2 result | 通过后才能进入 CP3。 |
| BLOCKING 产品未决项为 0 | pending human | CP2 决策项 | HLD 细节可保留为设计决策，不改变外部路径语义。 |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| 用户场景 | `docs/product/USE-CASES.md` | draft, pending human | v1.4；新增 UC-RA-09。 |
| 结构化需求 | `docs/product/REQUIREMENTS.md` | draft, pending human | v1.4；新增 REQ-RA-023..026。 |
| 工程场景 / 覆盖矩阵 | `docs/product/SCENARIOS.yaml`、`docs/product/TEST-MATRIX.md` | draft, pending human | v1.4/2.1；新增 SCN-RA-14/15、GAP-RA-07。 |
| 规划产物 | `docs/product/MVP-SCOPE.md`、`docs/product/STORY-MAP.md` | draft, pending human | v1.4；新增目标、范围和 Story 候选。 |
| 场景确认审计 | `process/discussions/CP2-CR031-SCENARIO-DISCUSSION-LOG.md` | complete | 记录用户的直接确认。 |

## 人工审查结果

待用户确认。`approve` 表示接受上述三项推荐方案；`修改: <具体修改点>` 表示保持本门禁 pending 并按指定范围修订。

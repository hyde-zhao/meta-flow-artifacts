---
checkpoint_id: "CP3-CR107-REMEDIATION-DESIGN-REVIEW"
checkpoint_name: "CR107 Remediation Design Review"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-06-21T21:28:55+08:00"
reviewed_by: "user"
reviewed_at: "2026-06-21T22:02:28+08:00"
auto_check_result: "process/checks/CP3-CR107-HLD-CONSISTENCY.md"
context_capsule: "process/context/CP3-CR107-REMEDIATION-DESIGN-CONTEXT.yaml"
target:
  phase: "solution-design"
  artifacts:
    - "docs/design/HLD-CR107-QUANT-LAB-REMEDIATION-DESIGN.md"
    - "process/DEVELOPMENT-PLAN-CR107.yaml"
---

# CP3 CR107 Remediation Design 人工审查

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP3-CR107-HLD-CONSISTENCY.md` | PASS | 0 | P2 推荐设计可审查；不授权 P3 实施。 |

## Decision Brief

推荐决策：批准 CR107 推荐设计，后续拆分为 CR108 layered docs/package identity current truth、CR109 CR tracking historical hygiene、CR110 non-runtime verification/readiness。CR108 必须覆盖 README、USER-MANUAL、`docs/design`、`docs/features`、`docs/quality`，并把 `process` 历史设计/readiness 材料纳入 inventory-only 分类；CR110 必须扫描 tests/scripts/config/tooling 中的旧身份引用。

备选方案：只修 README / USER-MANUAL；合并为单个大整改 CR；或只记录偏离不实施；或等待 CR102-CR104 关闭后再批准。

影响维度：用户文档准确性、`docs/design` / `docs/features` / `docs/quality` 设计链一致性、`process` 历史材料误读风险、package identity 一致性、CR tracking 可审计性、后续 P3/P4 验证成本、runtime/credential 安全边界。

风险与回退：若用户 reject，CR107 保持 active 并回到设计修订；若用户要求缩小范围，则保留 docs/package identity，CR tracking hygiene 转 follow-up；若 CR102-CR104 未关闭影响 P3，CR108/CR109 可保持 pending。

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP3-CR107-REMEDIATION-DESIGN-CONTEXT.yaml` |
| capsule 状态 | ready-for-human-gate |
| read_profile | compact |
| 默认读取策略 | 先读 capsule；仅在缺失、冲突、字段不足或人工审计时读取完整文档 |
| 全文档读取扩展 | 0 次 |
| discussion log | N/A：本轮由用户明确要求进入 P2 推荐设计，灰区已写入 HLD 表格。 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| P1 baseline | `process/changes/CR-106.md` | scanned | 5 | 3 | 范围、阶段、CR 拆分纳入。 |
| HLD | `docs/design/HLD-CR107-QUANT-LAB-REMEDIATION-DESIGN.md` | scanned | 10 | 4 | 方案、范围、风险接受纳入。 |
| README / USER-MANUAL quick scan | `README.md`; `docs/USER-MANUAL.md` | scanned | 3 | 2 | package identity / Host Orchestrator 偏离纳入。 |
| docs design/features/quality quick scan | `docs/design`; `docs/features`; `docs/quality` | scanned | 6 | 1 | 分层文档资产整改纳入 DQ-CP3-CR107-02。 |
| process historical design/readiness quick scan | `process/HLD.md`; `process/REQUIREMENTS.md`; `process/STORY-BACKLOG.md`; `process/checkpoints`; `process/context`; `process/release` | scanned | 4 | 1 | inventory-only 分类纳入 DQ-CP3-CR107-02；默认 legacy audit keep。 |
| tooling/test/config quick scan | `tests`; `scripts`; `config`; `.codex/agents`; `.agents/skills` | planned | 3 | 1 | P4 静态验证纳入 DQ-CP3-CR107-03；不授权业务代码整改。 |
| STATE / CR tracking | `process/state/STATE.current.json`; `process/changes/CR-INDEX.yaml` | scanned | 3 | 1 | CR102-CR104 active 风险纳入。 |
| 下游 LLD / implementation | process stories | n/a | 0 | 0 | P2 尚未进入 Story LLD。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP3-CR107-01 | architecture | 是否采用 CR108/CR109/CR110 拆分？ | 采用分层文档整改 + CR tracking + 验证收尾。 | 合并为单 CR；只记录不修。 | 推荐方案风险可控；合并复杂；只记录无法消除偏离。 | 影响后续 CR 数和审查成本。 | 用户要求缩短流程时可合并。 |
| DQ-CP3-CR107-02 | scope | CR108 是否覆盖 README、USER-MANUAL、`docs/design`、`docs/features`、`docs/quality`，并把 `process` 历史设计/readiness 材料作为 inventory-only 面？ | 是；先做分层偏离矩阵，再修 current-truth 项；`process` 历史材料默认 legacy audit keep。 | 只修 README / USER-MANUAL；或全仓替换旧口径。 | 推荐方案覆盖完整设计链且保留审计；只修用户文档会遗漏设计消费入口；全仓替换会破坏历史语义。 | 影响 CR108 范围和文档验证矩阵。 | 若范围过大，可把 docs/quality 或 process inventory 后移到 CR110。 |
| DQ-CP3-CR107-03 | implementation | P3/P4 是否允许修改 README / USER-MANUAL 和 current-truth 文档，并静态扫描 tests/scripts/config/tooling，但不改业务代码？ | 允许。 | 文档也延后到 P4；或只允许 inventory 不允许修订。 | 推荐方案先消除用户可见偏离并识别工具入口风险。 | P3 将触碰用户文档和 current-truth 文档；P4 扫描工具/测试引用；不触碰业务逻辑。 | 用户不同意则 CR108 改为 design-only，工具扫描改为 follow-up。 |
| DQ-CP3-CR107-04 | risk_acceptance | CR102-CR104 未关闭时是否允许 CR107 设计通过？ | 允许，但 P3 前重新检查。 | 阻塞直到主线关闭。 | 推荐方案不阻断设计；阻塞更保守但延后。 | 多 active WARN 仍存在。 | P3 前若仍影响检查，保持 pending。 |

### 用户需决策事项

| 决策 ID | 用户需决策事项 | 当前建议 |
|---|---|---|
| DQ-CP3-CR107-01 | 是否批准 CR108/CR109/CR110 后续拆分 | approve |
| DQ-CP3-CR107-02 | 是否批准 CR108 覆盖 README、USER-MANUAL、docs/design、docs/features、docs/quality，并 inventory-only 盘点 process 历史设计/readiness 面 | approve |
| DQ-CP3-CR107-03 | 是否允许 P3/P4 修改 current-truth 文档并静态扫描工具/测试引用，但不改业务代码 | approve |
| DQ-CP3-CR107-04 | 是否接受 CR102-CR104 不阻断 P2 设计 | approve |

### 不授权项

- 不授权读取凭据、`.env`、token、API key、cookie、私钥、账号。
- 不授权 runtime、SaaS、trace upload、production write、真实交易或真实发布。
- 不授权启动 CR-033。
- 不授权关闭或处理 CR102-CR104。
- 不授权业务代码整改；P3 实施前需独立 CR/CP。

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| HLD 已输出 | 待审查 | `docs/design/HLD-CR107-QUANT-LAB-REMEDIATION-DESIGN.md` | 待用户确认。 |
| 自动预检 PASS | 待审查 | `process/checks/CP3-CR107-HLD-CONSISTENCY.md` | 待用户确认。 |
| context ready | 待审查 | `process/context/CP3-CR107-REMEDIATION-DESIGN-CONTEXT.yaml` | 待用户确认。 |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | 推荐方案是否接受 | PASS | DQ-CP3-CR107-01 | 用户已批准。 |
| 2 | 分层文档资产整改边界是否接受 | PASS | DQ-CP3-CR107-02 | 用户已批准。 |
| 3 | P3 文档修改授权边界是否接受 | PASS | DQ-CP3-CR107-03 | 用户已批准。 |
| 4 | CR102-CR104 风险接受是否接受 | PASS | DQ-CP3-CR107-04 | 用户已批准。 |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| 用户明确 approve / 修改 / reject | PASS | 当前对话：批准 CR107 CP3 通过，继续推进项目 | 用户已 approve。 |
| approve 后仍不启动 P3 | PASS | 不授权项 | 仅允许进入 CR108 文档/package identity current-truth 整改入口；仍不授权业务代码整改。 |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| HLD | `docs/design/HLD-CR107-QUANT-LAB-REMEDIATION-DESIGN.md` | PASS | 推荐设计已批准。 |
| Development plan | `process/DEVELOPMENT-PLAN-CR107.yaml` | PASS | 阶段计划已批准。 |
| CP3 checkpoint | `process/checkpoints/CP3-CR107-REMEDIATION-DESIGN-REVIEW.md` | PASS | 本文件。 |

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-06-21T22:02:28+08:00
- 修改意见：无；批准 CR107 CP3 通过，继续推进项目。
- 风险接受项：接受 CR102-CR104 尚未由主线关闭时不阻断 CR107 设计通过；P3 前重新检查。

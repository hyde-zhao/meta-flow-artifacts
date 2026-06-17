---
checkpoint_id: "CP3-CR071-HLD-REVIEW"
checkpoint_name: "CR071 Copy-first Shadow Root Design Review"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-06-16T07:16:44+08:00"
reviewed_by: "user"
reviewed_at: "2026-06-16T07:33:48+08:00"
auto_check_result: "process/checks/CP3-CR071-HLD-CONSISTENCY.md"
context_capsule: "process/context/CP3-CR071-DESIGN-CONTEXT.yaml"
auto_final_authorization: false
target:
  phase: "solution-design"
  story_id: ""
  artifacts:
    - "process/changes/CR-071-COPY-FIRST-SHADOW-ROOT-MIGRATION-2026-06-16.md"
    - "docs/release/COPY-FIRST-SHADOW-ROOT-MIGRATION-PLAN-CR071.md"
    - "docs/release/COPY-FIRST-SHADOW-ROOT-MANIFEST-CR071.yaml"
---

# CP3 CR071 Copy-first Shadow Root Design Review 人工审查

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP3-CR071-HLD-CONSISTENCY.md` | PASS | 0 | copy-first 架构、`.git` 策略、target preflight、回退与不授权边界已定义。 |

## Decision Brief

推荐决策：批准 CR071 CP3 设计，以 copy-first shadow root 替代 rename/move；默认复制 `.git` 以保留本地 Git 工作树，不执行任何远端 Git 写动作；target root 必须在执行前不存在或为空。

备选方案：不复制 `.git` 只生成普通文件树；改为 rename/move；延后执行只保留设计。推荐方案优先，因为它保留可用工作树和回退路径，同时避免修改外部状态。

影响维度：目标目录是否可直接作为本地 Git 工作树、远端配置副本风险、回退方式、排除项验证和未来 root cutover。

风险与回退：如果用户不接受复制 `.git`，需要重写 manifest 和 CP5 验证；如果执行前 target 非空，必须阻断并回到人工确认；如果 copy 失败，不自动删除 partial target。

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP3-CR071-DESIGN-CONTEXT.yaml` |
| capsule 状态 | ready |
| read_profile | compact |
| 默认读取策略 | 先读 capsule；仅在缺失、冲突、字段不足、人工审计或深度评审时读取完整文档 |
| 全文档读取扩展 | 0 次 |
| 缺失 / waived 理由 | BLUEPRINT / DOMAIN-MAP / DEPENDENCY-MAP N/A：CR071 是 single-operation local migration gate。 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue | `process/STATE.md.human_gate_decisions.pending_human_decisions[]` | scanned | 0 | 0 | 当前队列尚未登记 CR071；本文件创建后回写。 |
| 委托 Agent 交还摘要 | `process/handoffs/*RETURN-SUMMARY.md` | n/a | 0 | 0 | 本轮未调度子 agent；设计由用户约束和前序迁移基线直接导出。 |
| 自动预检结果 | `process/checks/CP3-CR071-HLD-CONSISTENCY.md` | scanned | 5 | 5 | 架构、`.git`、target、回退和 no path rewrite 纳入 DQ。 |
| discussion log / checkpoint | `process/discussions/*` / `process/checks/*DISCUSSION-CHECKPOINT.json` | n/a | 0 | 0 | 架构灰区已直接转为 DQ。 |
| 下游正式产物 | CR071 plan / manifest | scanned | 5 | 5 | 命令形态、回退、验证和不授权项纳入 DQ。 |
| 用户显式选择题 | 当前对话 | scanned | 4 | 4 | 用户限定 copy-first、old root retained、no NAS/remote/runtime/lake。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP3-CR071-01 | architecture | copy-first 是否作为主架构？ | 采用 copy-first shadow root；不 rename/move old root。 | rename/move；只生成 tar/snapshot。 | 推荐方案可回退；rename/move 风险高；snapshot 不形成可用工作树。 | 影响目标目录可用性。 | 用户要求不可复制时重提。 |
| DQ-CP3-CR071-02 | implementation | 是否复制 `.git`？ | 复制 `.git`，使 target 是完整本地 Git 工作树；不执行 remote。 | 不复制 `.git`；在 target 后续重新 init/clone。 | 推荐方案保留当前本地状态；不复制会失去 Git 语境；重新 init/clone 涉远端风险。 | 复制本地 Git config，但不会网络写入。 | 用户不接受时修改 manifest。 |
| DQ-CP3-CR071-03 | implementation | target root 存在时如何处理？ | 执行前若 target 非空则阻断，不覆盖。 | 覆盖合并；改名旧 target；删除旧 target。 | 推荐方案保护已有内容；覆盖/删除有破坏风险。 | 避免误覆盖。 | target 非空时回到人工决策。 |
| DQ-CP3-CR071-04 | risk_acceptance | copy 失败或 partial target 如何回退？ | 保留旧 root，停止并记录 partial-copy；不自动删除 target。 | 自动删除 target；自动重试覆盖。 | 推荐方案避免二次破坏；自动删除/覆盖需要额外授权。 | 可能留下 partial target 需人工处理。 | 用户授权清理时另起动作。 |
| DQ-CP3-CR071-05 | architecture | 是否允许本轮改写路径引用或切换环境变量？ | 不允许；不改 README/process、shell、IDE、cron、systemd、MARKET_DATA_LAKE_ROOT。 | 同步切换工作流到 target；批量改写历史路径。 | 推荐方案把复制和 cutover 分离；备选会扩大影响。 | 复制完成后仍需人工选择是否切换工作根。 | 后续 CR072/root cutover 决策。 |

### CP3 追加 Decision Brief 字段

| 字段 | 内容 |
|---|---|
| 候选架构 | A: copy-first shadow root（推荐）；B: rename/move；C: snapshot-only；D: remote clone。 |
| 推荐方案优化项 | 保留旧 root、可静态验证、可后续切换。 |
| 牺牲项 | 初始 target 不含 data/reports/.env/.venv/node_modules；可能需要后续重建运行环境。 |
| When to switch | target 非空、需要 `.env` / data/report、要求 NAS/data lake/remote Git 时切换到新 CR。 |
| 场景模拟 | target absent happy path PASS；target non-empty blocked；excluded path absent 可验证。 |

### 不授权项

如果你回复 approve，表示接受上述 5 项推荐方案；不表示授权以下操作：

- 不授权 rename/move/delete old root。
- 不授权覆盖、删除或清理已存在 target root。
- 不授权 remote Git write、branch/default branch governance、history rewrite、force push、tag、remote deletion 或 set-url。
- 不授权 NAS/data lake/provider/catalog/runtime/credential、`.env` read/copy、QMT / MiniQMT runtime 或 CR046 recovery。
- 不授权 shell/IDE/cron/systemd/环境变量切到新 root。

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| CP3 context ready | 待审查 | `process/context/CP3-CR071-DESIGN-CONTEXT.yaml` |  |
| 自动预检 PASS | 待审查 | `process/checks/CP3-CR071-HLD-CONSISTENCY.md` |  |
| plan / manifest 可读 | 待审查 | `docs/release/COPY-FIRST-SHADOW-ROOT-*.{md,yaml}` |  |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | copy-first 是否作为主架构 | 待审查 | DQ-CP3-CR071-01 |  |
| 2 | `.git` 复制策略是否接受 | 待审查 | DQ-CP3-CR071-02 |  |
| 3 | target root preflight 是否严格 | 待审查 | DQ-CP3-CR071-03 |  |
| 4 | partial-copy 回退是否接受 | 待审查 | DQ-CP3-CR071-04 |  |
| 5 | 是否禁止路径/环境切换 | 待审查 | DQ-CP3-CR071-05 |  |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| 用户明确 approve / 修改 / reject | 待审查 | 本文件人工审查结果 |  |
| CP3 approve 不执行复制 | 待审查 | 不授权项 |  |
| 设计变更已回到 manifest | 待审查 | 待用户回复 |  |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| CP3 auto precheck | `process/checks/CP3-CR071-HLD-CONSISTENCY.md` | 待审查 |  |
| CP3 context capsule | `process/context/CP3-CR071-DESIGN-CONTEXT.yaml` | 待审查 |  |
| CP3 launch message | `process/checks/CP3-CR071-HUMAN-GATE-LAUNCH-MESSAGE.md` | 待审查 |  |

## 人工审查结果

- 结论：approved
- 审查人：user
- 审查时间：2026-06-16T07:33:48+08:00
- 修改意见：N/A
- 风险接受项：用户回复“同意”，按 `approve` 处理，接受 DQ-CP3-CR071-01..05 推荐方案；该批准确认 copy-first shadow root 架构、默认复制 `.git`、target 非空阻断、partial target 不自动清理和不做路径/环境切换，不授权任何远端 Git 写动作、NAS、data lake、runtime、凭据读取、old root rename/move/delete 或 CR046 recovery。

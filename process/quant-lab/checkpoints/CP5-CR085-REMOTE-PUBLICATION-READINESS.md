---
checkpoint_id: "CP5-CR085-REMOTE-PUBLICATION-READINESS"
checkpoint_name: "CR085 Remote Publication Readiness"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-06-17T14:20:48+08:00"
reviewed_by: "user"
reviewed_at: "2026-06-17T14:37:22+08:00"
auto_check_result: "process/checks/CP5-CR085-REMOTE-PUBLICATION-READINESS.md"
context_capsule: "process/context/CP5-CR085-REMOTE-CLEANUP-CONTEXT.yaml"
auto_final_authorization: false
target:
  phase: "documentation"
  story_id: ""
  artifacts:
    - "docs/release/CLEAN-SNAPSHOT-MATERIALIZATION-MANIFEST-CR085.yaml"
    - "process/checks/CR085-CLEAN-SNAPSHOT-MATERIALIZATION-SCAN-2026-06-17.md"
---

# CP5 CR085 Remote Publication Readiness 人工审查

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP5-CR085-REMOTE-PUBLICATION-READINESS.md` | PASS | 0 | A/B 候选和 sanitize-docs 路径已形成；远端写入仍未授权。 |

## Decision Brief

推荐决策：先选择 `sanitize-docs`，即暂不 push，先清理或替换 README.md 与 `docs/USER-MANUAL.md` 中的 topology blockers，再重新扫描。该方案保留远端文档能力，又避免 overlay A 的既有 blocker 和 overlay B 的文档删除。

备选方案：

- overlay A：保留远端 README 和 USER-MANUAL，接受 2 个既有 topology blockers。
- overlay B：使用 blocker=0 的严格候选，但远端将删除 README 和 `docs/USER-MANUAL.md`。
- stop：暂停 CR085，不做远端发布。

影响维度：远端文档可用性、public-safety、远端删除范围、下一轮扫描成本、push 风险。

风险与回退：本门禁不授权 push；若选择 sanitize-docs，只进入文档清理与重扫；若选择 A 或 B，也仍需单独远端写入授权。任何 remote write 前必须再次确认 remote URL、目标分支和候选树。

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP5-CR085-REMOTE-CLEANUP-CONTEXT.yaml` |
| capsule 状态 | ready |
| read_profile | compact |
| 默认读取策略 | 先读 capsule；本次因 materialization 结果已产生，读取 manifest / scan evidence |
| 全文档读取扩展 | 0 次 |
| 缺失 / waived 理由 | Story LLD N/A：CR085 是 remote publication governance gate。 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue | `process/STATE.md.human_gate_decisions.pending_human_decisions[]` | scanned | 0 | 0 | 当前队列无旧 pending。 |
| materialization manifest | `docs/release/CLEAN-SNAPSHOT-MATERIALIZATION-MANIFEST-CR085.yaml` | scanned | 3 | 3 | A/B/sanitize-docs、warning、remote write boundary 纳入。 |
| materialization scan | `process/checks/CR085-CLEAN-SNAPSHOT-MATERIALIZATION-SCAN-2026-06-17.md` | scanned | 3 | 3 | topology blockers 和 warnings 纳入。 |
| CR085 formal CR | `process/changes/CR-085-REMOTE-SYNCHRONIZATION-CLEANUP-GATE-2026-06-17.md` | scanned | 3 | 3 | 不授权项和下一门禁纳入。 |
| 用户显式输入 | 当前对话 `同意` | scanned | 1 | 1 | 只解释为 materialization / scan 授权，不解释为 push 授权。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP5-CR085-PUB-01 | implementation | README.md 和 `docs/USER-MANUAL.md` 如何处理？ | `sanitize-docs`：先清理 / 替换文档中的 topology blockers，再重扫。 | overlay A：保留现状并接受 2 个 blocker；overlay B：删除 README/USER-MANUAL；stop。 | 推荐兼顾文档可用性与安全；A 风险残留；B 文档缺失；stop 不发布。 | 影响远端文档、public-safety 和后续 push 候选。 | sanitize 后重扫；若用户明确接受删除可切换 B；接受风险可切换 A。 |
| DQ-CP5-CR085-PUB-02 | risk_acceptance | 4 个测试 fixture warning 是否可接受？ | 接受为非阻断 warning，保留测试。 | 删除相关测试；停止发布。 | 推荐保留测试覆盖；删除测试会降低验证能力。 | warning 仅在测试中，未发现精确密钥。 | 若用户要求零 warning，转为排除相关测试并重扫。 |
| DQ-CP5-CR085-PUB-03 | runtime_authorization | 本门禁是否授权远端写入？ | 不授权；只确认下一步文档处理 / 重扫路线。 | 直接授权 overlay A push；直接授权 overlay B push。 | 推荐保持 fail-closed；直接 push 需要更精确执行计划。 | 远端仍不变。 | 远端写入必须另起 exact push gate。 |

### 用户需决策事项

用户需决策事项：是否接受 DQ-CP5-CR085-PUB-01、DQ-CP5-CR085-PUB-02、DQ-CP5-CR085-PUB-03 的推荐方案。回复 `approve` 表示选择 sanitize-docs、接受测试 fixture warning、继续不授权远端写入；不表示授权“不授权项”中的任何操作。

### CP5 追加 Decision Brief 字段

| 字段 | 内容 |
|---|---|
| 设计证据类型分布 | Git publication governance evidence；Story LLD N/A。 |
| LLD clarification queue 收敛状态 | N/A。 |
| 已回答问题 | `.env.example` 保留、`.env` 禁止、direct push 阻断、materialization 扫描结果。 |
| 转 OPEN / Spike 的问题 | history purge、force push、default branch governance、tag。 |
| 未回答阻断项为 0 的证据 | 自动预检 PASS；但 remote write 仍未授权。 |
| 文件 owner | host-orchestrator 处理 governance；实际文档 sanitization 若执行需另行记录。 |
| merge order | 本门禁不 merge/rebase/amend/push。 |

### 不授权项

如果你回复 approve，表示接受上述 3 项推荐方案：选择 sanitize-docs、接受测试 fixture warning、继续不授权远端写入。

approve 不表示授权以下操作：

- 不授权 `git push`、`git push -u`、remote deletion、remote add / set-url / remove。
- 不授权 tag、force push、history rewrite、default branch governance。
- 不授权上传、读取、打印或保存 `.env`、token、password、private key、cookie、session。
- 不授权 data/reports 内容读取、NAS 内容操作、provider/lake/catalog、runtime、CR046 recovery、old root retirement 或 cleanup。

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| materialization scan 已完成 | 待审查 | `process/checks/CR085-CLEAN-SNAPSHOT-MATERIALIZATION-SCAN-2026-06-17.md` |  |
| A/B 候选可审查 | 待审查 | materialization manifest |  |
| 远端写入未执行 | 待审查 | 本 Decision Brief |  |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | README / USER-MANUAL 处理方式是否接受 | 待审查 | DQ-CP5-CR085-PUB-01 |  |
| 2 | 测试 fixture warning 是否接受 | 待审查 | DQ-CP5-CR085-PUB-02 |  |
| 3 | 本门禁继续不授权远端写入是否接受 | 待审查 | DQ-CP5-CR085-PUB-03 |  |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| 用户明确 approve / 修改 / reject | 待审查 | 本文件人工审查结果 |  |
| approve 后不执行远端写入 | 待审查 | 不授权项 |  |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| 自动预检 | `process/checks/CP5-CR085-REMOTE-PUBLICATION-READINESS.md` | 待审查 |  |
| materialization manifest | `docs/release/CLEAN-SNAPSHOT-MATERIALIZATION-MANIFEST-CR085.yaml` | 待审查 |  |
| materialization scan | `process/checks/CR085-CLEAN-SNAPSHOT-MATERIALIZATION-SCAN-2026-06-17.md` | 待审查 |  |

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-06-17T14:37:22+08:00
- 修改意见：用户回复“同意”，按 CP5 Remote Publication Readiness approve 处理；接受 DQ-CP5-CR085-PUB-01..03 的推荐方案。该批准选择 `sanitize-docs`、接受 4 个测试 fixture warning 作为非阻断项，并继续不授权任何远端写入。
- 风险接受项：接受本轮仅做本地文档清理候选和重扫；不授权 push、远端删除、force、history rewrite、tag、default branch governance 或 `.env` 读取 / 上传。

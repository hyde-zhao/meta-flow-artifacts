---
check_id: "CR073-TARGET-EVIDENCE-DIFF"
type: "evidence-diff"
status: "PASS"
owner: "host-orchestrator"
created_at: "2026-06-16T10:07:45+08:00"
source_root: "/home/hyde/workspace/local_backtest"
target_root: "/home/hyde/workspace/quant-lab"
---

# CR073 Target Evidence Diff

## Entry Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| source root 存在 | PASS | `/home/hyde/workspace/local_backtest` | 当前日常工作根，不切换。 |
| target root 存在 | PASS | `/home/hyde/workspace/quant-lab` | CR071 copy-first target。 |
| 排除项只做路径检查 | PASS | commands | 未读取 `.env`、`reports/`、`data/` 内容。 |
| dry-run 工具可用 | PASS | `rsync 3.2.7` | 后续使用 `rsync -ain --relative`。 |

## 差异分类

| 分类 | 数量 | 路径 |
|---|---:|---|
| MISSING | 14 | `docs/quality/VERIFICATION-REPORT-CR072.md`; `docs/release/DEPLOY-CHECKLIST-CR071.md`; `docs/release/FEEDBACK-CR071.md`; `docs/release/LOCAL-ASSETS-NAS-ARCHIVE-MANIFEST-CR072.yaml`; `docs/release/MIGRATION-CR071.md`; `docs/release/RELEASE-NOTES-CR071.md`; `docs/release/ROLLBACK-CR071.md`; `process/changes/CR-072-NAS-ARCHIVE-LOCAL-ASSETS-2026-06-16.md`; `process/checkpoints/CP8-CR071-DELIVERY-READINESS.md`; `process/checks/CP6-CR072-NAS-ARCHIVE-EXECUTION-DONE.md`; `process/checks/CP7-CR072-NAS-ARCHIVE-VERIFICATION-DONE.md`; `process/checks/CP8-CR071-DELIVERY-READINESS.md`; `process/context/CP8-CR071-DELIVERY-CONTEXT.yaml`; `process/release/RELEASE-CONTEXT-CR071.yaml` |
| DIFFERENT | 6 | `docs/quality/REVIEW-CR071.md`; `docs/quality/TEST-REPORT-CR071.md`; `docs/release/COPY-FIRST-SHADOW-ROOT-MANIFEST-CR071.yaml`; `process/STATE.md`; `process/changes/CR-071-COPY-FIRST-SHADOW-ROOT-MIGRATION-2026-06-16.md`; `process/changes/CR-INDEX.yaml` |
| SAME | 17 | CR071 CP2/CP3/CP5、CP6/CP7 和部分 docs/release/docs/quality 证据 |

## Initial Dry-run Itemize

```text
.d..t...... docs/quality/
>f.st...... docs/quality/REVIEW-CR071.md
>f.st...... docs/quality/TEST-REPORT-CR071.md
>f+++++++++ docs/quality/VERIFICATION-REPORT-CR072.md
.d..t...... docs/release/
>f.st...... docs/release/COPY-FIRST-SHADOW-ROOT-MANIFEST-CR071.yaml
>f+++++++++ docs/release/DEPLOY-CHECKLIST-CR071.md
>f+++++++++ docs/release/FEEDBACK-CR071.md
>f+++++++++ docs/release/LOCAL-ASSETS-NAS-ARCHIVE-MANIFEST-CR072.yaml
>f+++++++++ docs/release/MIGRATION-CR071.md
>f+++++++++ docs/release/RELEASE-NOTES-CR071.md
>f+++++++++ docs/release/ROLLBACK-CR071.md
>f.st...... process/STATE.md
.d..t...... process/changes/
>f.st...... process/changes/CR-071-COPY-FIRST-SHADOW-ROOT-MIGRATION-2026-06-16.md
>f+++++++++ process/changes/CR-072-NAS-ARCHIVE-LOCAL-ASSETS-2026-06-16.md
>f.st...... process/changes/CR-INDEX.yaml
.d..t...... process/checkpoints/
>f+++++++++ process/checkpoints/CP8-CR071-DELIVERY-READINESS.md
.d..t...... process/checks/
>f+++++++++ process/checks/CP6-CR072-NAS-ARCHIVE-EXECUTION-DONE.md
>f+++++++++ process/checks/CP7-CR072-NAS-ARCHIVE-VERIFICATION-DONE.md
>f+++++++++ process/checks/CP8-CR071-DELIVERY-READINESS.md
.d..t...... process/context/
>f+++++++++ process/context/CP8-CR071-DELIVERY-CONTEXT.yaml
.d..t...... process/release/
>f+++++++++ process/release/RELEASE-CONTEXT-CR071.yaml
```

## Post-CR073-Material Scoped Dry-run

生成 CR073 formal CR、context、checks、checkpoints 和 launch message 后，按同一 scoped files-from 重新 dry-run：

| 分类 | 数量 | 说明 |
|---|---:|---|
| MISSING | 29 | 原 14 个缺失证据 + CR073 新增证据 15 个 |
| DIFFERENT | 6 | 原 6 个过期证据，含 `process/STATE.md` 与 `process/changes/CR-INDEX.yaml` |
| SAME | 17 | 原一致证据未变化 |

```text
>f+++++++++ process/changes/CR-073-TARGET-ROOT-CONVERGENCE-CUTOVER-READINESS-GATE-2026-06-16.md
>f+++++++++ process/checkpoints/CP2-CR073-TARGET-EVIDENCE-BASELINE.md
>f+++++++++ process/checkpoints/CP3-CR073-SCOPED-SYNC-HLD-REVIEW.md
>f+++++++++ process/checkpoints/CP5-CR073-SCOPED-SYNC-READINESS.md
>f+++++++++ process/checks/CP2-CR073-HUMAN-GATE-LAUNCH-MESSAGE.md
>f+++++++++ process/checks/CP2-CR073-TARGET-EVIDENCE-BASELINE.md
>f+++++++++ process/checks/CP3-CR073-HUMAN-GATE-LAUNCH-MESSAGE.md
>f+++++++++ process/checks/CP3-CR073-SCOPED-SYNC-HLD-CONSISTENCY.md
>f+++++++++ process/checks/CP5-CR073-HUMAN-GATE-LAUNCH-MESSAGE.md
>f+++++++++ process/checks/CP5-CR073-SCOPED-SYNC-READINESS.md
>f+++++++++ process/checks/CR073-TARGET-EVIDENCE-DIFF-2026-06-16.md
>f+++++++++ process/context/CP2-CR073-REQUIREMENT-CONTEXT.yaml
>f+++++++++ process/context/CP3-CR073-DESIGN-CONTEXT.yaml
>f+++++++++ process/context/CP5-CR073-SCOPED-SYNC-CONTEXT.yaml
>f+++++++++ process/context/NEXT-SESSION-CR073-STARTUP-CONTEXT.md
```

完整 itemize 仍只包含 `process/` / `docs/` scoped evidence 文件；未出现 `.git/`、`reports/`、`data/`、`.venv/`、`node_modules/` 或 `.env`。

## Forbidden Path Baseline

| 检查 | 状态 | 证据 | 说明 |
|---|---|---|---|
| target 根目录查找 | PASS | `find /home/hyde/workspace/quant-lab -maxdepth 1 (...)` 输出为空 | 未发现 `reports data .venv node_modules .env`。 |
| target Git status | PASS | `git -C /home/hyde/workspace/quant-lab status --short -- reports data .venv node_modules .env` 输出为空 | 无排除项工作树状态。 |
| target Git ls-files | PASS | `git -C /home/hyde/workspace/quant-lab ls-files -- reports data .venv node_modules .env` 输出为空 | 无排除项被 Git 跟踪。 |

## 结论

- 结论：`PASS`
- 阻断项：无；存在 target 缺失/过期证据，正是 CR073 需要收敛的范围。
- 下一步：生成 CP2/CP3/CP5 门禁；通过后再执行 scoped local sync。

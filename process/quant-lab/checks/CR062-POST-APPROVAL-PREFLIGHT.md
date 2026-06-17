# CR062 Post-Approval Publication Preflight

## 结论

CR062 post-approval preflight 结论：BLOCKED。

用户已于 2026-06-15T09:59:22+08:00 回复“同意”，批准 CP2 / CP3 / CP5，并授权受控 Git publication 范围。但本次 preflight 发现当前分支已跟踪的 forbidden publication paths。由于 `git push current branch` 会发布当前分支既有 tracked 历史，而不只发布本次 staged manifest，按 CR062 fail-closed 策略必须停止，不执行 `git add`、`git commit`、`git remote add` 或 `git push`。

## Preflight Summary

| 检查项 | 结果 | 证据 |
|---|---|---|
| Git 仓库存在 | PASS | `git rev-parse --is-inside-work-tree` -> `true` |
| 当前分支 | PASS_WITH_RISK | `work/chapter3-factor-gap-remediation-20260608`；按 CP5 不 rename |
| 当前 HEAD | PASS | `e7a3a0d` |
| remote 配置 | PASS | `git remote -v` 无输出；remote absent，但未执行 add |
| 工作树状态 | PASS_WITH_RISK | 有 CR058-CR062 文档、CR061 package/import code、STATE / CR-INDEX 回填待提交 |
| 未跟踪 forbidden paths | PASS | `git ls-files -o --exclude-standard data reports runs .env .env.local .envrc` 无输出 |
| 已跟踪 forbidden paths | BLOCKED | `reports/**`、`runs/**` 和 `.env.example` 已在 Git index 中 |
| large file scan | PASS_WITH_RISK | `process/STATE.md` 超过 1MB，属于已跟踪过程状态文件 |
| NAS/lake 路径扫描 | PASS_WITH_RISK | 候选 CR059 证据中包含 NAS export / mount path 字符串；本轮不纳入 publish manifest |

## Blocking Findings

| Finding ID | 严重度 | 路径 | 原因 | 处理 |
|---|---|---|---|---|
| CR062-BLOCK-001 | BLOCKING | `reports/chapter3_factor_panel/**` | CR062 exclude policy 默认禁止 `reports/**` 进入 GitHub publication；这些文件已被当前 Git index 跟踪。 | 停止 publication；需要新决策选择 repository hygiene / clean publication strategy。 |
| CR062-BLOCK-002 | BLOCKING | `runs/RUN-EXEC-20260613-001.md` | CR062 exclude policy 默认禁止 `runs/**` 进入 GitHub publication；该文件已被当前 Git index 跟踪。 | 停止 publication；需要新决策。 |
| CR062-BLOCK-003 | HIGH | `.env.example` | CR062 manifest 将 `.env*` 列为 exclude pattern；该文件已被当前 Git index 跟踪。虽然可能是示例文件，但当前门禁未授予例外。 | 停止 publication；需要新决策明确 `.env.example` 是否允许。 |

## Non-Blocking Observations

| Observation ID | 严重度 | 路径 / 范围 | 说明 |
|---|---|---|---|
| CR062-OBS-001 | MEDIUM | `process/STATE.md` | 文件超过 1MB，且包含长期过程状态；不单独阻断，但增加 publication 体积和审计成本。 |
| CR062-OBS-002 | MEDIUM | `docs/release/NAS-MOUNT-PLAN-CR059.md`、`docs/release/MIGRATION-MANIFEST-SEED-CR059.yaml`、`ops/systemd/*.mount` | 候选未跟踪证据包含 NAS export / mount path 和内网 IP 字符串；本轮不应纳入 publication manifest，除非另行批准。 |

## Stop Condition

触发 stop condition：`forbidden path appears in tracked branch publication surface`。

本轮未执行：

- `git add`
- `git commit`
- `git remote add`
- `git push`
- `git tag`
- branch rename
- history rewrite
- force push
- remote deletion
- NAS/data lake/provider/catalog/runtime/凭据操作
- physical move / bulk import rewrite
- CR046 recovery

## Required Follow-Up Decision

用户已于 2026-06-15T10:24:37+08:00 回复“同意”，接受推荐处理方式。后续决策 / CR 已登记为：`CR066 Repository Hygiene / Clean Publication Strategy`。

编号说明：`CR063` / `CR064` / `CR065` 已在迁移序列中作为 NAS archive/data migration、data lake root/catalog migration、runtime/trading authorization 的语义候选保留；本次 blocker follow-up 使用 `CR066`，避免覆盖这些候选含义。

推荐方案：暂停 CR062 publication execution，先设计 clean publication strategy，至少处理：

- 已跟踪 `reports/**` / `runs/**` 是否允许发布、移出未来发布面，或接受风险。
- `.env.example` 是否作为示例文件例外允许。
- 是否采用新仓库快照、clean branch、历史保留、历史重写或私有仓库发布策略。
- 若需 history rewrite、branch creation/rename、force push 或清理 tracked history，必须另起 CR 并单独授权。

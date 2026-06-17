---
check_id: "CR074-ROOT-CUTOVER-EXECUTION"
cr_id: "CR-074"
type: "execution-evidence"
status: "PASS"
owner: "host-orchestrator"
created_at: "2026-06-16T22:41:34+08:00"
checked_at: "2026-06-16T22:48:47+08:00"
source_root: "/home/hyde/workspace/local_backtest"
target_root: "/home/hyde/workspace/quant-lab"
authoritative_root: "/home/hyde/workspace/quant-lab"
legacy_root: "/home/hyde/workspace/local_backtest"
---

# CR074 Root Cutover Execution Evidence

## Scope

CR074 执行范围只包含 logical root authority marker：

- 将 `/home/hyde/workspace/quant-lab` 标记为后续 authoritative daily workflow root。
- 保留 `/home/hyde/workspace/local_backtest` 为 legacy root。
- 将 CR074 状态、checkpoint、context、check evidence、STATE 和 CR-INDEX scoped sync 到 target。

## Non-Authorized Operations

| 操作 | 状态 | 说明 |
|---|---|---|
| 当前会话自动切换 cwd | NOT_AUTHORIZED | 当前命令仍在旧根执行，不自动 `cd`。 |
| 删除 / 重命名 / 移动旧根 | NOT_AUTHORIZED | 旧根保留。 |
| 复制 / 读取 / 比对 `reports/`、`data/`、`.venv/`、`node_modules/`、`.env` | NOT_AUTHORIZED | 白名单不包含这些路径。 |
| 凭据读取 / 打印 / 迁移 | NOT_AUTHORIZED | 未读取凭据。 |
| target env rebuild / Python dependency sync / smoke runtime | NOT_AUTHORIZED | 不执行运行时动作。 |
| 远端 Git 写入 / branch / tag / set-url / default branch governance | NOT_AUTHORIZED | 未做远端 Git 写入。 |
| NAS 新写入 / 删除 / promote、data lake、provider fetch、lake write、catalog publish | NOT_AUTHORIZED | 未触发外部数据动作。 |
| QMT / MiniQMT runtime / 账户查询 / 下单 / 撤单 / simulation/live | NOT_AUTHORIZED | 未触发 runtime。 |
| CR046 recovery | NOT_AUTHORIZED | CR046 仍 user-paused。 |

## Preflight

| 检查项 | 结果 | 命令 / 证据 | 说明 |
|---|---|---|---|
| 本地 CR tracking | PASS | `python3 -B scripts/check_cr_tracking_consistency.py --project-root .` | 输出 `CR tracking consistency: PASS`。 |
| target CR tracking | PASS | `python3 -B /home/hyde/workspace/quant-lab/scripts/check_cr_tracking_consistency.py --project-root /home/hyde/workspace/quant-lab` | 输出 `CR tracking consistency: PASS`。 |
| target CR073 evidence | PASS | `test -f` 白名单检查 | 无 `MISSING` 输出。 |
| target forbidden root find | PASS | `find /home/hyde/workspace/quant-lab -maxdepth 1 ...` | 空输出。 |
| target forbidden git status | PASS | `git -C /home/hyde/workspace/quant-lab status --short -- reports data .venv node_modules .env` | 空输出。 |
| target forbidden git ls-files | PASS | `git -C /home/hyde/workspace/quant-lab ls-files -- reports data .venv node_modules .env` | 空输出。 |
| root retained | PASS | `test -d /home/hyde/workspace/local_backtest && test -d /home/hyde/workspace/quant-lab` | old root 与 target 均存在。 |

## Sync Allowlist

| 路径 | 类别 | 说明 |
|---|---|---|
| `process/STATE.md` | state | 写入 `root_authority` marker。 |
| `process/changes/CR-INDEX.yaml` | index | 写入 root authority marker 和 CR074 状态。 |
| `process/changes/CR-074-ROOT-CUTOVER-DAILY-WORKFLOW-SWITCH-2026-06-16.md` | CR evidence | 正式 CR。 |
| `process/context/CP2-CR074-ROOT-CUTOVER-CONTEXT.yaml` | context | CP2 context。 |
| `process/context/CP3-CR074-CUTOVER-DESIGN-CONTEXT.yaml` | context | CP3 context。 |
| `process/context/CP5-CR074-CUTOVER-READINESS-CONTEXT.yaml` | context | CP5 context。 |
| `process/checks/CP2-CR074-ROOT-CUTOVER-BASELINE.md` | check | CP2 auto precheck。 |
| `process/checks/CP2-CR074-HUMAN-GATE-LAUNCH-MESSAGE.md` | check | CP2 launch message。 |
| `process/checks/CP3-CR074-CUTOVER-DESIGN-CONSISTENCY.md` | check | CP3 auto precheck。 |
| `process/checks/CP3-CR074-HUMAN-GATE-LAUNCH-MESSAGE.md` | check | CP3 launch message。 |
| `process/checks/CP5-CR074-CUTOVER-READINESS.md` | check | CP5 auto precheck。 |
| `process/checks/CP5-CR074-HUMAN-GATE-LAUNCH-MESSAGE.md` | check | CP5 launch message。 |
| `process/checks/CR074-ROOT-CUTOVER-EXECUTION-2026-06-16.md` | check | 本执行证据。 |
| `process/checks/CR074-ROOT-CUTOVER-VERIFICATION-2026-06-16.md` | check | 执行后验证证据。 |
| `process/checkpoints/CP2-CR074-ROOT-CUTOVER-BASELINE.md` | checkpoint | CP2 approved。 |
| `process/checkpoints/CP3-CR074-CUTOVER-DESIGN-REVIEW.md` | checkpoint | CP3 approved。 |
| `process/checkpoints/CP5-CR074-CUTOVER-READINESS.md` | checkpoint | CP5 approved。 |

## Conflict Classification Before Sync

| 分类 | 文件 |
|---|---|
| expected-different | `process/STATE.md` |
| expected-different | `process/changes/CR-INDEX.yaml` |
| target-missing | `process/changes/CR-074-ROOT-CUTOVER-DAILY-WORKFLOW-SWITCH-2026-06-16.md` |
| target-missing | `process/context/CP2-CR074-ROOT-CUTOVER-CONTEXT.yaml` |
| target-missing | `process/context/CP3-CR074-CUTOVER-DESIGN-CONTEXT.yaml` |
| target-missing | `process/context/CP5-CR074-CUTOVER-READINESS-CONTEXT.yaml` |
| target-missing | `process/checks/CP2-CR074-ROOT-CUTOVER-BASELINE.md` |
| target-missing | `process/checks/CP2-CR074-HUMAN-GATE-LAUNCH-MESSAGE.md` |
| target-missing | `process/checks/CP3-CR074-CUTOVER-DESIGN-CONSISTENCY.md` |
| target-missing | `process/checks/CP3-CR074-HUMAN-GATE-LAUNCH-MESSAGE.md` |
| target-missing | `process/checks/CP5-CR074-CUTOVER-READINESS.md` |
| target-missing | `process/checks/CP5-CR074-HUMAN-GATE-LAUNCH-MESSAGE.md` |
| target-missing | `process/checkpoints/CP2-CR074-ROOT-CUTOVER-BASELINE.md` |
| target-missing | `process/checkpoints/CP3-CR074-CUTOVER-DESIGN-REVIEW.md` |
| target-missing | `process/checkpoints/CP5-CR074-CUTOVER-READINESS.md` |

## Dry Run

命令形态：

```bash
printf '%s\n' <allowlist> | rsync -ani --relative --files-from=- \
  --exclude='.git/' --exclude='reports/' --exclude='data/' \
  --exclude='.venv/' --exclude='node_modules/' --exclude='.env' \
  ./ /home/hyde/workspace/quant-lab/
```

Dry-run output:

```text
>f.st...... process/STATE.md
.d..t...... process/changes/
>f+++++++++ process/changes/CR-074-ROOT-CUTOVER-DAILY-WORKFLOW-SWITCH-2026-06-16.md
>f.st...... process/changes/CR-INDEX.yaml
.d..t...... process/checkpoints/
>f+++++++++ process/checkpoints/CP2-CR074-ROOT-CUTOVER-BASELINE.md
>f+++++++++ process/checkpoints/CP3-CR074-CUTOVER-DESIGN-REVIEW.md
>f+++++++++ process/checkpoints/CP5-CR074-CUTOVER-READINESS.md
.d..t...... process/checks/
>f+++++++++ process/checks/CP2-CR074-HUMAN-GATE-LAUNCH-MESSAGE.md
>f+++++++++ process/checks/CP2-CR074-ROOT-CUTOVER-BASELINE.md
>f+++++++++ process/checks/CP3-CR074-CUTOVER-DESIGN-CONSISTENCY.md
>f+++++++++ process/checks/CP3-CR074-HUMAN-GATE-LAUNCH-MESSAGE.md
>f+++++++++ process/checks/CP5-CR074-CUTOVER-READINESS.md
>f+++++++++ process/checks/CP5-CR074-HUMAN-GATE-LAUNCH-MESSAGE.md
.d..t...... process/context/
>f+++++++++ process/context/CP2-CR074-ROOT-CUTOVER-CONTEXT.yaml
>f+++++++++ process/context/CP3-CR074-CUTOVER-DESIGN-CONTEXT.yaml
>f+++++++++ process/context/CP5-CR074-CUTOVER-READINESS-CONTEXT.yaml
```

## Execution Decision

- 结论：PASS
- 允许执行：是，限定为 allowlist scoped rsync。
- `--delete`：未使用。
- forbidden paths：未列入 allowlist，并通过 `--exclude` 明确排除。
- 后续验证：见 `process/checks/CR074-ROOT-CUTOVER-VERIFICATION-2026-06-16.md`。

## Execution Output

命令形态：

```bash
printf '%s\n' <allowlist> | rsync -ai --relative --files-from=- \
  --exclude='.git/' --exclude='reports/' --exclude='data/' \
  --exclude='.venv/' --exclude='node_modules/' --exclude='.env' \
  ./ /home/hyde/workspace/quant-lab/
```

执行输出：

```text
>f.st...... process/STATE.md
.d..t...... process/changes/
>f+++++++++ process/changes/CR-074-ROOT-CUTOVER-DAILY-WORKFLOW-SWITCH-2026-06-16.md
>f.st...... process/changes/CR-INDEX.yaml
.d..t...... process/checkpoints/
>f+++++++++ process/checkpoints/CP2-CR074-ROOT-CUTOVER-BASELINE.md
>f+++++++++ process/checkpoints/CP3-CR074-CUTOVER-DESIGN-REVIEW.md
>f+++++++++ process/checkpoints/CP5-CR074-CUTOVER-READINESS.md
.d..t...... process/checks/
>f+++++++++ process/checks/CP2-CR074-HUMAN-GATE-LAUNCH-MESSAGE.md
>f+++++++++ process/checks/CP2-CR074-ROOT-CUTOVER-BASELINE.md
>f+++++++++ process/checks/CP3-CR074-CUTOVER-DESIGN-CONSISTENCY.md
>f+++++++++ process/checks/CP3-CR074-HUMAN-GATE-LAUNCH-MESSAGE.md
>f+++++++++ process/checks/CP5-CR074-CUTOVER-READINESS.md
>f+++++++++ process/checks/CP5-CR074-HUMAN-GATE-LAUNCH-MESSAGE.md
>f+++++++++ process/checks/CR074-ROOT-CUTOVER-EXECUTION-2026-06-16.md
.d..t...... process/context/
>f+++++++++ process/context/CP2-CR074-ROOT-CUTOVER-CONTEXT.yaml
>f+++++++++ process/context/CP3-CR074-CUTOVER-DESIGN-CONTEXT.yaml
>f+++++++++ process/context/CP5-CR074-CUTOVER-READINESS-CONTEXT.yaml
```

# CR068 Test Report

## Commands

| 命令 | 结果 | 说明 |
|---|---|---|
| `git ls-files -- .gitignore` | PASS | 当前仓库 `.gitignore` 已跟踪。 |
| `git hash-object .gitignore` | PASS | hash `3bee76d19c3493b1d9550f23f3539c1e1ee55bed`。 |
| `git ls-remote git@github.com:hyde-zhao/quant-lab.git refs/heads/master` | PASS | push 前为 `495fcc...`，push 后为 `3ade165...`。 |
| `git add --dry-run .` | PASS | 仅输出 `add '.gitignore'`。 |
| `git check-ignore -v .env .env.local data/sample.parquet reports/report.md .venv/x.py __pycache__/x.pyc credentials/key.pem secret.key` | PASS | 目标敏感 / 本地产物路径均被忽略。 |
| `git check-ignore -v .env.example` | PASS | exit 1，无输出，表示 `.env.example` 未被忽略。 |
| `git diff --cached --name-status` | PASS | 仅 `A .gitignore`。 |
| `git diff --cached --check` | PASS | 无输出。 |
| `git merge-base --is-ancestor origin/master HEAD` | PASS | fast-forward 可成立。 |
| `git push origin HEAD:master` | PASS | `495fcc4..3ade165 HEAD -> master`。 |

## Not Run

| 项目 | 原因 |
|---|---|
| Full pytest | CR068 只发布 `.gitignore`，不改 Python runtime 或业务代码。 |
| NAS / data lake / provider / catalog / runtime checks | 明确不授权且不属于 CR068。 |
| Branch/default branch settings check | branch governance out of scope。 |

## Result

- 结论：`PASS_WITH_RISK`
- 阻断项：0

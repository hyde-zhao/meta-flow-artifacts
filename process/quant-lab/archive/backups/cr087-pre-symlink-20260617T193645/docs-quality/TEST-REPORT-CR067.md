# CR067 Test Report

## Commands

| 命令 / 检查 | 结果 |
|---|---|
| snapshot scan gate | PASS, `FINDING_COUNT=0` |
| import smoke | PASS |
| `python -m compileall -q quant_lab engine market_data strategies trading tests` | PASS |
| `pytest -q tests/test_cr061_package_import_layout.py` | PASS, `4 passed in 0.02s` |
| `git ls-remote --heads origin master` | PASS, remote commit matches local commit |

## Not Run

| 项目 | 原因 |
|---|---|
| Full pytest | CP5 DQ-CP5-CR067-04 scoped validation accepted; publication gate only requires package/import smoke. |
| Runtime / provider / lake tests | Explicitly out of CR067 authorization. |
| GitHub branch/default-branch settings | Branch governance not authorized in CR067. |

## Result

`PASS_WITH_RISK`.

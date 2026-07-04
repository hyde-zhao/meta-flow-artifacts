# CR075 Migration Notes

CR075 performs no schema, data, source tree or external system migration.

| Area | Decision | Reason |
|---|---|---|
| Source code | N/A | No code changes in CR075 execution. |
| Data / reports | N/A | Access and recovery are explicitly out of scope. |
| Python environment | Local target `.venv` created | Approved `uv sync --python 3.11` created `/home/hyde/workspace/quant-lab/.venv`. |
| Legacy root | Retained | `/home/hyde/workspace/local_backtest` remains legacy root. |
| Remote Git | N/A | No remote write or governance operation authorized. |

Cleanup, old-root retirement, data/report handling and remote Git governance require separate CRs.

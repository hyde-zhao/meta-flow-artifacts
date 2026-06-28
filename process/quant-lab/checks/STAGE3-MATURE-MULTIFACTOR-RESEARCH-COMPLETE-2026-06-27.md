# Stage 3 Mature Multifactor Research Complete

Date: 2026-06-27
Run ID: `stage3-mature-mf-20260627-real-lake-v2`
Conclusion: PASS, ready for Stage 4 review input only.

## Scope

Completed Stage 3 research on the data-capable machine using the canonical data lake at `/home/hyde/data/quant-lab/lake`.

This run produced a mature multifactor research package, mature strategy admission package, runner offline preflight, risk policy, factor specs, research evidence index, and Stage 4 observation plan.

## Authorization Boundary

Allowed:

- Read canonical data lake inputs.
- Write research artifacts under `/home/hyde/data/quant-lab/research/runs/stage3_mature_multifactor`.
- Write process evidence under `process/evidence/stage3-mature-multifactor`.

Not authorized and not performed:

- provider fetch
- lake write
- catalog publish
- QMT operation
- gateway start
- simulation or live runtime
- account or order operation
- credential read

Operation counters in `stage3-research-summary.json` are all `0` for the prohibited operations above.

## Inputs

Data range: `2021-01-01` to `2026-05-28`

Canonical datasets:

- `prices`
- `trade_status`
- `liquidity_capacity`
- `market_cap`
- `prices_limit`
- `stock_basic`
- `industry_classification`
- `hs300_index` as benchmark lineage ref

Reader behavior: the Stage 3 runner reads full `canonical/<dataset>/1.0` roots and treats catalog `canonical_path` values as lineage refs. This avoids partial reads where catalog entries point to a single parquet shard.

## Output Artifacts

Research directory:

`/home/hyde/data/quant-lab/research/runs/stage3_mature_multifactor/stage3-mature-mf-20260627-real-lake-v2`

Key files:

- `STAGE3-MATURE-RESEARCH-PACKAGE.json`
- `MATURE-STRATEGY-ADMISSION-PACKAGE.json`
- `RUNNER-OFFLINE-PREFLIGHT.json`
- `STAGE4-OBSERVATION-PLAN.json`
- `RESEARCH-EVIDENCE-INDEX.json`
- `RUN-MANIFEST.json`
- `FACTOR-SPECS.json`
- `SIGNAL-SET.json`
- `PORTFOLIO-RISK-POLICY.json`
- `factor_panel.parquet`
- `label_window.parquet`
- `ic_rankic.csv`
- `layered_returns.csv`
- `turnover_cost.csv`
- `exposure.csv`

Process evidence:

`process/evidence/stage3-mature-multifactor/stage3-mature-mf-20260627-real-lake-v2/stage3-research-summary.json`

## Metrics

| Metric | Value |
|---|---:|
| factor_panel_rows | 6646097 |
| valid_research_rows | 256055 |
| rebalance_count | 256 |
| portfolio_rows | 20283 |
| mean_composite_rank_ic | 0.05691211494913351 |
| mean_net_forward_return | -0.0007326291504067516 |
| mean_turnover | 0.3339663192975936 |
| top_quantile_mean_return | -0.0047652017115113994 |

Package status: `stage3_research_ready_for_stage4_review`

## Validation

Commands executed:

```bash
uv run --python 3.11 python -m py_compile engine/stage3_mature_multifactor_research.py scripts/run_stage3_mature_multifactor_research.py
uv run --python 3.11 pytest -q tests/test_stage3_mature_multifactor_research.py tests/test_stage2_mature_multifactor_framework.py
uv run --python 3.11 python scripts/run_stage3_mature_multifactor_research.py --run-id stage3-mature-mf-20260627-real-lake-v2 --lake-root /home/hyde/data/quant-lab/lake --research-root /home/hyde/data/quant-lab/research/runs/stage3_mature_multifactor --process-evidence-root process/evidence/stage3-mature-multifactor --start 2021-01-01 --end 2026-05-28 --top-n 80 --min-adv-amount 1000000
```

Results:

- Py compile: PASS
- Regression tests: `16 passed`
- Stage 3 runner: PASS
- Stage 3 package validation: PASS, no blocked reasons

## Corrections Made During Stage 3

- Fixed reader behavior to prefer complete canonical dataset roots over catalog single-shard paths.
- Corrected liquidity capacity threshold from an assumed `20000000` amount unit to `1000000` in data lake native `adv20_amount` units.
- Removed scipy dependency from RankIC by computing Pearson correlation on ranked factor and label values.
- Kept `stage3-mature-mf-20260627-real-lake-v1` as a superseded diagnostic run; v2 is the accepted Stage 3 result.

## Remaining Limits

- Stage 3 output is research/admission evidence only.
- Stage 4 still requires independent simulation runtime authorization before any gateway or QMT runtime use.
- No `small_live` or `live` claim is made.
- `prices_limit` catalog lineage remains imperfect, although the runner reads the complete canonical root.
- Style exposure is a proxy built from market cap, PB, volatility, and momentum.

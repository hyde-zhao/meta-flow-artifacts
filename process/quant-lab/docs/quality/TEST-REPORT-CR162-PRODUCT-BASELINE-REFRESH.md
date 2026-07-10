# CR162 Test Report

| Check | Result | Notes |
|---|---|---|
| YAML parse: `SCENARIOS.yaml` | PASS | Parsed with `uv run --python 3.11 python` and `yaml.safe_load`. |
| Nine-document CR162 marker scan | PASS | `rg -l 'CR162'` returned all six product and three feature targets. |
| Seven-object matrix scan | PASS | All seven named objects found in feature DESIGN overlay. |
| Fail-closed / CR155 / no-overclaim scan | PASS | Required wording found across product and feature baselines. |
| CP2 result JSON | PASS | `meta-flow cp result-check` passed after user approval. |
| CP3-CP6 N/A result JSON | PASS | All four route records passed `meta-flow cp result-check`. |
| Dispatch ledger | PASS | `meta-flow event check --type dispatch` passed. |
| Diff whitespace | PASS | `git diff --check` passed. |

No runtime, provider, data, broker, simulation, paper, live, trading, remote or publish test was authorized or executed.

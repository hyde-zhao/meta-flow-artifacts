请审查人工门禁 `CP2-CR161-STRATEGY-ADMISSION-EVIDENCE-PIPELINE-HARDENING-SCOPE`。

checklist 路径: `process/checkpoints/CP2-CR161-STRATEGY-ADMISSION-EVIDENCE-PIPELINE-HARDENING-SCOPE.md`
自动预检结论: PASS，阻断项 0。

推荐回复: `approve`

如果你回复 `approve`，表示接受以下 5 项推荐方案：

| 决策 ID | 推荐方案 |
|---|---|
| `DQ-CP2-CR161-WAVE1-TRIAL-LINEAGE-SOURCE` | Wave 1 采用 contract-first；先定义 `ExperimentFamilyManifest` contract，trial lineage 缺失输出 `typed_unavailable`。 |
| `DQ-CP2-CR161-TYPED-UNAVAILABLE-FAIL-CLOSED` | mandatory evidence `typed_unavailable` 时阻断 statistical significance / robustness / paper candidate / production-like claims，不得 silent PASS。 |
| `DQ-CP2-CR161-CR155-NEGATIVE-REGRESSION` | CR155 只作为 negative regression seed；验收目标是仍 `blocked_admission_failed`、`paper_candidate=false`，不要求历史 C1/C2 全部可计算。 |
| `DQ-CP2-CR161-FOLLOWUP-SPLIT` | research-engine forward instrumentation 和 impact/capacity computation 作为 follow-up / later wave，不进当前 CP2 最小切片。 |
| `DQ-CP2-CR161-AUTHORIZATION-BOUNDARY` | 不授权 source/test implementation、新 real lake 读取、runtime、paper/simulation/live/trading、broker、NAS/provider/credential、external framework、Git remote write 或 publish。 |

不表示授权: source/test implementation、research-engine instrumentation、新 real lake read/write、NAS/provider/credential、runtime/paper/simulation/live/trading/broker、external framework、catalog/store/registry write、Git remote write 或 publish。

可选回复:

- `approve`
- `修改: <具体修改点>`
- `reject`

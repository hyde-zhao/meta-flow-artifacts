---
story_id: "CR163-S05-integrity-recovery-permission-regression"
change_id: "CR-163"
stage: "CP6"
status: "implemented"
created_by: "meta-dev-debugger"
created_at: "2026-07-11T17:15:00+08:00"
context_ref: "process/context/stories/STORY-CR163-S05.CP6.work-packet.json"
design_ref: "process/stories/STORY-CR163-S05-integrity-recovery-permission-regression-LLD.md"
---

# CR163-S05 Integrity, Recovery, Permission and CR155 Regression Implementation

## 修订记录

| 轮次 | 日期 | 修订人 | 变更 |
|---|---|---|---|
| CP6 initial | 2026-07-11 | meta-dev-debugger | 新增且仅新增三组 synthetic regression tests；覆盖全量 trace、完整性/恢复、授权边界和 CR155 负回归。 |
| CP7 rework 1 | 2026-07-11 | meta-dev-debugger | 关闭 QA-CR163-S05-001..003：13 类 deny-by-default exercised-path hooks、authorization exact schema、12 条可解析 structured trace。 |
| CP7 rework 2 | 2026-07-11 | meta-dev-debugger | 最终关闭 QA-CR163-S05-001：13 类全部安装到 concrete/path-aware 标准库边界，安全 marker 从真实安装入口验证 pre-original 阻断。 |

## 实现前置检查

| 条目 | 结论 | 证据 |
|---|---|---|
| CP5 / Story 门控 | PASS | Story `status=dev-ready`、`design_evidence_confirmed=true`、LLD `confirmed=true` |
| S01-S04 runtime 依赖 | PASS_WITH_RISK | `process/evidence/STORY-CR163-S01..S04.CP7.index.json`；S01 PASS，S02-S04 PASS_WITH_RISK |
| 文件所有权 | PASS | 只创建 packet `write_scope` 的三组新 tests 与 S05 IMPLEMENTATION/return/evidence |
| 授权边界 | PASS | 全部 fixture 为内存或 `tmp_path`；无真实 data/lake/NAS/provider/credential/research/runtime/trading/backfill/release/external 操作 |

## 实现对象与设计契约映射

| 对象 | 实现 | 契约与验证 |
|---|---|---|
| Integrity / recovery suite | `tests/test_cr163_trial_lineage_integrity.py` | 精确冻结 requirements 8/8、scenarios 12/12、CPI mappings 4/4、producer chains 2/2；验证 raw membership、10→1 hash、target binding、claim ceiling、合法 supersession、broken/cyclic fail closed 与五类 negative 5/5 |
| Authorization suite | `tests/test_cr163_trial_lineage_authorization.py` | `BOUNDARY_INSTALLATION_MANIFEST` 精确列出 13 类 concrete target+predicate：Path.open read/write、os.scandir、socket.create_connection/socket.socket、os.getenv、五个 subprocess 入口、urllib.request.urlopen、Path.glob；全部 guard 激活时 S01→S05 synthetic path observed=0；每类通过实际安装入口的安全 marker 证明 counter++ 且 original spy 未执行 |
| CR155 regression suite | `tests/test_cr163_trial_lineage_cr155_regression.py` | synthetic dict 与实际 CR155 public contract object 两条路径均在 native ledger 缺失时 blocked、paper false、backfill/reconstruction 0；effective/C1/runtime-ready ceiling 保持关闭 |
| Structured handoff | 本 IMPLEMENTATION、CP6 return/evidence | 记录实际 touched files、量化退出、边界、验证和下一路由；无 design delta |

## 定量退出结果

| 指标 | 结果 |
|---|---|
| Requirements / P0 scenarios | 8/8、12/12；每条含 scenario_id/requirements/feature_scope/test_function，12 个唯一函数均可解析，错误/删除引用 fail closed |
| CPI mappings / producer chains | 4/4、2/2 |
| Logical-equivalent seal hashes | 10 seals → 1 distinct hash |
| Negative fixture classes | 5/5 blocked，均保留 non-empty reason；validator cases 保留 target ref binding |
| Supersession recovery | valid v1→v2 通过且 v1 bytes/ref/hash 不变；broken 与 cyclic 100% blocked |
| Forbidden operation categories | exercised public path 13/13 zero；13/13 sentinel deliberate hit 在真实操作前 fail closed；missing/unknown/bool/negative/nonzero evidence 全部 fail closed |
| CR155 actual/synthetic | 2/2 blocked；paper false；historical backfill/family/trial reconstruction 均 0 |
| Claim ceiling | effective count/ref/method available claims 0；C1 computed claims 0；runtime-ready claims 0 |

## 最小实现切片与验证

| Slice | 内容 | 结果 |
|---|---|---|
| S05-01 | Trace matrix、raw membership、deterministic seal、target/tamper、supersession 与 five-negative | PASS |
| S05-02 | 13-category authorization contract、zero baseline、逐类 nonzero fail-closed | PASS |
| S05-03 | actual public CR155 object + synthetic CR155 negative regression 与 overclaim ceiling | PASS |
| S05-04 | packet 指定 S01-S05 full suite | PASS，253 passed |
| S05-R1-01 | deny-by-default sentinel registry 包围 synthetic public path，并做 13 类 hit matrix | PASS |
| S05-R1-02 | authorization mapping exact-key/type/range/nonzero fail-closed matrix | PASS |
| S05-R1-03 | structured trace + AST function resolver + wrong/deleted ref negative test | PASS |
| S05-R1-04 | S05 focused 与 packet 七套全量回归 | PASS，33 / 262 passed |
| S05-R2-01 | 13 类 concrete/path-aware monkeypatch wrappers 与 exact installation manifest | PASS，13/13 installed |
| S05-R2-02 | 每类先替换安全 original spy，再从实际安装 API 调用 marker | PASS，13/13 blocked；original marker calls=0 |
| S05-R2-03 | 全 guard active 期间重跑 S01→S05 synthetic public path | PASS，observed counters 13/13 zero |
| S05-R2-04 | authorization/focused/full packet 回归 | PASS，19 / 33 / 262 passed |

## 验证结果

| 命令 | 结果 | 摘要 |
|---|---|---|
| `uv run --python 3.11 pytest -q tests/test_cr163_trial_lineage_integrity.py tests/test_cr163_trial_lineage_authorization.py tests/test_cr163_trial_lineage_cr155_regression.py tests/test_experiment_family_lineage_contracts.py tests/test_experiment_family_lineage_store.py tests/test_cr163_trial_lineage_producer_adapters.py tests/test_cr163_trial_lineage_admission_projection.py` | PASS | `253 passed in 28.98s` |
| `git diff --check` | PASS | 无 whitespace error |
| `uv run --python 3.11 pytest -q tests/test_cr163_trial_lineage_integrity.py tests/test_cr163_trial_lineage_authorization.py tests/test_cr163_trial_lineage_cr155_regression.py` | PASS | Rework 1 focused：`33 passed in 0.64s` |
| `uv run --python 3.11 pytest -q tests/test_cr163_trial_lineage_integrity.py tests/test_cr163_trial_lineage_authorization.py tests/test_cr163_trial_lineage_cr155_regression.py tests/test_experiment_family_lineage_contracts.py tests/test_experiment_family_lineage_store.py tests/test_cr163_trial_lineage_producer_adapters.py tests/test_cr163_trial_lineage_admission_projection.py` | PASS | Rework 1 full packet：`262 passed in 29.39s` |
| `uv run --python 3.11 pytest -q tests/test_cr163_trial_lineage_authorization.py` | PASS | Rework 2 concrete boundary suite：`19 passed in 0.63s` |
| `uv run --python 3.11 pytest -q tests/test_cr163_trial_lineage_integrity.py tests/test_cr163_trial_lineage_authorization.py tests/test_cr163_trial_lineage_cr155_regression.py` | PASS | Rework 2 focused：`33 passed in 0.66s` |
| `uv run --python 3.11 pytest -q tests/test_cr163_trial_lineage_integrity.py tests/test_cr163_trial_lineage_authorization.py tests/test_cr163_trial_lineage_cr155_regression.py tests/test_experiment_family_lineage_contracts.py tests/test_experiment_family_lineage_store.py tests/test_cr163_trial_lineage_producer_adapters.py tests/test_cr163_trial_lineage_admission_projection.py` | PASS | Rework 2 full packet：`262 passed in 29.28s` |

## Rework 1 findings closure

| Finding | 结论 | 证据 |
|---|---|---|
| QA-CR163-S05-001 | RESOLVED (rework 2) | 13 类均有 concrete/path-aware monkeypatch target+predicate，不再是 registry-only；全 guard 下实际执行 S01 recorder/seal/project、S03 inventory、S04 attachment 与 S05 zero guard，observed 13/13 zero；hit matrix 先将对应 original 替换为安全 spy，再从 Path/os/socket/subprocess/urllib 的实际安装入口调用 marker，13/13 counter++ 并在 original 前抛错，spy calls=0，无真实 I/O/network/subprocess |
| QA-CR163-S05-002 | RESOLVED | `validate_authorization_evidence()` 先比较 exact key set，再拒绝 bool/negative，最后由 zero guard 拒绝任意 nonzero；missing/unknown/bool/negative/nonzero 五类 matrix 全部 fail closed，不再用 dataclass defaults 补零 |
| QA-CR163-S05-003 | RESOLVED | 12 条 `SCENARIO_TRACE` 均含四个 exact fields，requirement union=8/8、scenario IDs=12/12、test functions 唯一；AST resolver 校验三文件中的函数真实存在，错误/删除引用测试稳定失败 |

## 边界与剩余风险

- 未修改任何 production source、script、已有 test、Host state/check/handoff 或历史 CR155 artifact；工作树中这些路径的既有/并行改动不属于 S05。
- 测试不扫描真实 CR155 文件，不读取 `.env` 或 credential，不探测 lake/NAS/network/provider，不运行研究、simulation/paper/live/broker/trading，不执行 backfill/reconstruction 或 Git remote/publish。
- S03 已接受的 retry-loop 风险保持不变：本 suite 验证三 attempt 不增加 stable raw membership，但未来 producer 新增真实 retry loop 时仍应增加其端到端 fixture。
- effective trial 方法未获授权，故 C1/statistical/runtime-ready claims 按设计保持不可用；无长期设计变化，design delta 不需要。

## QA 交接重点

- 独立核验 trace 常量的 8/8、12/12、4/4、2/2 精确集合，而非只依赖测试总数。
- 复核 10 seal hashes、五类 negative reasons、v1 immutable + valid v2、broken/cyclic 两类恢复失败。
- 复核 13 个 counter 的零基线与逐类 nonzero 参数化 fail-closed。
- 复核 actual/synthetic CR155 均 blocked、paper false、backfill/reconstruction 0，且 effective/C1/runtime authorization 不被 lineage 提升。

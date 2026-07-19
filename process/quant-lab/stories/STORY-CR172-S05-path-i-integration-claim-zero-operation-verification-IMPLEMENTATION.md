# STORY-CR172-S05 实现执行证据

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| v1.0 | 2026-07-19 | meta-dev / dev-lv | 完成 S05 repository-local、test-only fixture 与三套 QAC；记录 CP6 验证和零操作边界。 |

## 1. 阶段决策

`PASS`。Story 的 CP5 设计证据已确认，四项串行依赖 S01-S04 均已通过 CP7，process route health=`ok`，文件所有权无冲突。实现未发现上游合同缺陷，也未修改任何 production source、上游测试、设计文档、状态文件或真实运行环境。

下一路由：`CP7 verification`。

## 2. 实现前置检查

| 检查项 | 结果 | 证据 |
|---|---|---|
| process 路由健康 | PASS | `process/current/CURRENT.json`：`health=ok` |
| CP5 / LLD 门控 | PASS | Story：`design_evidence_confirmed=true`、`lld_confirmed=true` |
| S01-S04 依赖 | PASS | packet dependency_inputs 指向四个 CP7 reverify PASS result/evidence |
| 文件所有权 | PASS | 仅写 packet `allowed_write_paths`，forbidden writes=`0` |
| 授权边界 | PASS | 仅 repository fixture；真实 action authorized/executed=`0/6`,`0/6` |
| 上游 hash 稳定 | PASS | 四个 production module 与四个上游测试均精确匹配 CP7 hash |

## 3. 实现对象清单

| 对象 | 文件 | 作用 | 规则真相源 |
|---|---|---|---|
| fixture 说明 | `tests/fixtures/cr172_path_i/README.md` | 声明 test-only 边界与非双真相原则 | S01-S04 production contracts |
| trace catalog | `tests/fixtures/cr172_path_i/scenario_catalog.json` | 15/27/11 精确映射 | 已确认 S05 acceptance |
| sealed chain | `tests/fixtures/cr172_path_i/sealed_chain_v1.json` | 固定 identity/time/observation/source-selection digest | S02/S03 verifier |
| failure oracle | `tests/fixtures/cr172_path_i/failure_mutations_v1.json` | tamper/partial/failure/revoke 预期 | S01-S04 fail-closed results |
| zero-op oracle | `tests/fixtures/cr172_path_i/zero_operation_oracle_v1.json` | 真实动作、claim 与 deferred 零值 | S01 claim/authz contracts |
| test adapter | `tests/fixtures/cr172_path_i/path_i_fixture.py` | 装配 public dataclass；实现纯内存 port fixture | S01-S04 public contracts |
| integration QAC | `tests/research/test_cr172_path_i_integration.py` | 三段 artifact 链、verifier、tamper、失败恢复 | S02-S04 public contracts |
| authorization QAC | `tests/research/test_cr172_path_i_authorization.py` | 六动作、DAG、caller 自报、revoke/context | S01 public contract |
| claim QAC | `tests/research/test_cr172_path_i_claim_regression.py` | trace、REQ-013、claim、signal/deferred/zero-op | S01 public contract + S05 catalog |

不适用对象：production code、Prompt/Skill、Schema、安装器、真实 adapter、用户文档、迁移和部署均为 N/A；本 Story 明确为 test-only。

## 4. fixture inventory 与非双真相证明

LLD §4/§11 要求的五个精确文件已落地：`README.md`、`scenario_catalog.json`、`sealed_chain_v1.json`、`failure_mutations_v1.json`、`zero_operation_oracle_v1.json`。`path_i_fixture.py` 是附加的 test-only adapter，已计入实现文件清单。

该 adapter 只执行三类动作：读取仓库内 JSON、构造 S01-S04 public dataclass、实现 production port 抽象要求的纯内存存取 seam。它不实现或复制以下规则：

- action eligibility、DAG predecessor、reason-code 或 claim ceiling；
- payload/manifest/seal canonicalization 与 digest；
- S02 verifier、S03/S04 verification vector、receipt self-hash、freshness 或 CAS 判定；
- 真实 lake/NAS/执行机、credential、network、signal、trading、deploy 或 Git remote adapter。

因此 fixture 仅是输入/oracle，规则判定仍由四个 production module 唯一提供；design delta=`N/A`，因为实现严格对齐已批准文件形态和边界，没有改变长期设计。

## 5. 设计契约映射

| 合同 | 实现/验证 | 结果 |
|---|---|---|
| 15/15 requirements、27/27 scenarios、11/11 outcomes | scenario catalog + collected test-name exact set | PASS；unknown/duplicate/uncovered=`0/0/0` |
| canonical payload 仅 timestamp/simple_return | Arrow schema 读取 + S02 verifier | `2/2` 列；forward proxy trial-return/empirical-R/effective-count accepted=`0/0/0` |
| S02 canonical seal bytes/digest/verifier | public canonicalizer + verifier 三项断言 | `1/1/1` |
| S03/S04 secondary digest | receipt dataclass field inventory | `0/0` |
| S04 每个 selected candidate 调 S02 verifier | 仅 monkeypatch S04 imported verifier binding 计数 | exactly once=`1` |
| 三段 artifact 链 | source selection→replica receipt/selection→material receipt/selection/handle | PASS |
| 六动作与 DAG | public Enum/record/enforcement/DIRECT_PREREQUISITE | `6/6/6`，nodes/edges=`6/5` |
| runtime without read | S01 decision + pre-side-effect guard | eligible=`0`，guard reject=`1` |
| approved ledger caller 自报 | fixture/real target 两种请求 | accepted/authorized/eligible=`0/0/0`；reason=`APPROVED_LEDGER_ADAPTER_UNAVAILABLE` |
| failure/tamper/revoke | failure oracle + public results | 全部 fail closed；previous selection preservation=`100%` |
| REQ-013 | public `decide_run_path` + zero-op oracle | contract_ready/runtime-enforcement/default-switch/runtime-delivered/future-prerequisite=`1/0/0/0/1` |
| 五项 claim | public `enforce_path_i_claim_ceiling` | true=`0/5` |
| deferred boundaries | AST import surface + zero-op oracle | Signal detailed/FU-v2/public C1/legacy migration/real runtime=`0` |
| 真实动作 | zero-op oracle + import/boundary scan | authorized/executed=`0/6`,`0/6`；真实操作=`0` |

## 6. 测试与 fixture 计划落实

- integration：10 tests，覆盖 canonical chain、exact verifier count、forward proxy、tamper、partial-lineage、replica/materialization preservation、digest 和 provenance surface。
- authorization：9 tests，覆盖六动作/DAG、完整 fixture 链、runtime-without-read、missing/revoke/context、approved-ledger、real-target escape。
- claim regression：8 tests，覆盖 15/27/11 trace、REQ-013、五 claim、8-slot signal boundary、deferred、zero-op 和 import boundary。
- 总计：27 tests，与 `scenario_catalog.json` 一一对应；duplicate/unknown/uncovered=`0/0/0`。

## 7. 验证结果

| 命令 | 结果 |
|---|---|
| S05 三模块 scoped pytest | PASS：`27 passed / 0 failed / 0 skipped`，最终 0.55s |
| S01-S05 combined pytest | PASS：`154 passed / 0 failed / 0 skipped`，最终 1.03s |
| 三模块 `py_compile` | PASS：exit 0 |
| 三模块 `git diff --no-index --check` | PASS：exit 1 为新增文件预期 diff，whitespace diagnostics=`0/3` |
| 四 production + 四 upstream test hash correlation | PASS：`8/8` 与 CP7 evidence 完全一致 |

首轮 scoped 曾为 `18 passed / 9 failed`；9 项均由 S05 fixture 复用 preflight decision 作为 commit decision 导致正确的 `COMMIT_DECISION_NOT_FRESH`，以及测试侧 Arrow bytes 输入和精确 reason oracle 两处问题。修正仅发生在 S05 fixture/test，未修改上游合同。最终失败=`0`。

## 8. 权限、真实操作与副作用计数

| 维度 | 计数 |
|---|---:|
| data lake read authorized/executed | 0/0 |
| multi-trial runtime authorized/executed | 0/0 |
| trial-return generation real authorized/executed | 0/0 |
| empirical-R real authorized/executed | 0/0 |
| NAS sync real authorized/executed | 0/0 |
| execution materialization real authorized/executed | 0/0 |
| credential/network/subprocess/mount/signal/trading/deploy/Git remote | 0 |
| production source/upstream test/design/state write | 0 |

测试中的 `repository_fixture` eligibility 和纯内存 port call 不计作真实授权、真实运行、Stage 3 readiness 或 C1 readiness。

## 9. 影响文件与边界检查

- touched files：15（fixture 6、test 3、Story/IMPLEMENTATION/return/evidence/result/summary 6）。
- forbidden touched files：0。
- unexpected imports：0。
- production source diff：0；mature runner diff：0；lineage diff：0；上游测试 diff：0。
- cache artifact：最终 `__pycache__` / `.pyc`=`0`。

## 10. 剩余风险与交接

非阻塞剩余风险保持原状态：真实 trial-return source、真实 runtime authorization、真实 adapter durability/performance、FU-CR173-001 empirical-R positive path、Signal transport、public C1 projection 与 legacy migration 均未实现、未验证且未获授权。repository fixture 证据不能上推为上述 readiness。

QA/Review 重点：

1. 独立复跑 27 scoped 与 154 combined；
2. 核对 scenario catalog 与实际 27 个 test name exact set；
3. 核对 S04 verifier exactly once、approved-ledger exact reason、previous-selection preservation；
4. 核对四 production/四 upstream hashes 与 CP7 evidence；
5. 核对 allowed/forbidden paths、zero real operations 与无缓存产物。

未运行项：真实 adapter、真实 lake/NAS/runtime、Signal、交易、部署和 Git remote 验证均因 packet 明令禁止而 N/A，不构成 waiver。

## CP6R1 最小回修（2026-07-19）

### Finding 闭环

- `F-001=CLOSED`：根因是 `scenario_catalog.json` 中 `SC-CR172-021` 的 requirement 语义误绑；实现合同、生产代码与上游测试均无缺陷。
- `SC-CR172-021 / test_req013_is_contract_ready_only` 已从 `REQ-CR172-006` 精确改绑为 `REQ-CR172-013`。
- `test_traceability_is_exact_and_total` 新增两项显式语义绑定断言：场景 021 → `REQ-CR172-013`，权威场景 006 → `REQ-CR172-006`；`semantic_mismatch` 从 `1` 收敛为 `0`。

### 实现与验证证据

- 实现资产仅修改 `tests/fixtures/cr172_path_i/scenario_catalog.json` 与 `tests/research/test_cr172_path_i_claim_regression.py`；未修改 production、其他 fixture/test、设计、状态或 ledger。
- S05 scoped：`27 passed / 0 failed / 0 skipped`；S01-S05 combined：`154 passed / 0 failed / 0 skipped`。
- `py_compile=PASS`；两项 `git diff --no-index --check=PASS`。
- requirement/scenario/outcome=`15/15`、`27/27`、`11/11`；unknown/duplicate/uncovered=`0/0/0`。
- production modules 与 upstream tests SHA-256=`8/8` 与 CP7 基线一致；六类真实动作 authorized/executed=`0/6`,`0/6`，五项高阶 claim true=`0/5`，真实外部操作=`0`。
- 行为合同没有变化，不需要 design delta；下一路由为 CP7R1 独立复验。

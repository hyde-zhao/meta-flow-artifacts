---
story_id: "CR172-S02-trial-return-artifact-native-hook"
cr_id: "CR-172"
stage: "CP6"
status: "reworked-2"
implementation_version: "1.2"
design_evidence_ref: "process/stories/STORY-CR172-S02-trial-return-artifact-native-hook-LLD.md"
work_packet_ref: "process/context/stories/STORY-CR172-S02.CP6.rework-2.work-packet.json"
implemented_by: "meta-dev"
implemented_at: "2026-07-18T20:09:51+08:00"
design_delta_required: false
open_items: 0
---

# CR172-S02 — IMPLEMENTATION

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-18 | meta-dev | 按 confirmed LLD v1.3 实现 pure trial-return artifact、fixture producer port、唯一 seal verifier 与定向合同测试。 |
| 1.1 | 2026-07-18 | meta-dev | CP6R1 关闭 F-CR172-S02-CP7-001：将结构兼容端口收敛为 repository-owned nominal capability，增加双 pre-call binding guard 与模块自有原子失败后置条件。 |
| 1.2 | 2026-07-18 | meta-dev | CP6R2 完整关闭 F-CR172-S02-CP7-001：在既有 verified-to-binding guard 中加入 seal authorization refs 有序精确关联，补齐五类偏差和 publisher 共享 guard 回归。 |

## 1. 实现结论

本 Story 已在独占路径 `2/2` 内完成，结论为 `implemented`。实现没有连接或修改 mature runner、lineage/store、真实湖、NAS、runtime、R、signal、trading、deploy 或 Git remote；设计差异、越界写入、waiver、开放实现问题均为 `0`。

量化结果：payload columns=`2/2`；manifest/seal semantics=`15/15`,`8/8`；跨 Story public types/functions=`3/3`,`3/3`；canonical seal bytes/digest/verifier definitions=`1/1/1`；fixture+real 与 forward proxy accepted=`0/0`；partial lineage erase/fake rollback/canonical advance=`0/0/0`。

## 2. 实现前置检查

| 检查项 | 结果 | 证据 |
|---|---|---|
| process route health | PASS | `process/current/CURRENT.json` health=`ok` |
| Story / CP5 设计门 | PASS | Story=`dev-ready`（实现前）；LLD status/confirmed/implementation_allowed=`confirmed/true/true` |
| S01 dependency | PASS | S01 CP7R1=`PASS`；S01 governance contract hash=`bc6abc...82abf` |
| 文件 owner / 冲突 | PASS | primary=`2/2`；shared=`0`；file_conflict_free=`true` |
| pyarrow writer profile | PASS | runtime pyarrow=`16.1.0`；LLD profile=`trial-return-parquet-v1` |
| clarification / design gap | PASS | active implementation questions=`0`；无需 runner/lineage/真实 producer 变更 |

## 3. 实现对象清单

| 对象 | 实现位置 | 动作 | 验证 |
|---|---|---|---|
| exact payload/schema | `engine/trial_return_artifact.py` | 创建 UTC interval 校验、exact two-column deterministic Parquet serializer 与 content hash | P01/P02/N03 |
| 15-field manifest / 8-field seal | 同上 | 创建 frozen/slots value objects、canonical manifest body 与 domain-separated hashes | P01/P02/F01/F02 |
| exact public verifier contract | 同上 | 导出 3 types + 3 functions；`VerifiedTrialReturnBundleV1` 禁止直接构造 | public-contract / receipt-only negative |
| fixture producer port | 同上 | 固定 guard→candidate→seal→verify→commit；只调用 injected `commit_verified` 一次 | P01/N02/F03 |
| partial-lineage audit value | 同上 | 只输出 `partial_lineage_blocked_audit`，三个 mutation flag 固定 false | F04 |
| contract / failure / static tests | `tests/research/test_cr172_trial_return_artifact.py` | 创建 12 个 test nodes，覆盖 Feature 10 cases、public contract、zero-operation surface | targeted / combined pytest |
| CP6 交接证据 | 本文件 + return/evidence/result/summary | 固化边界、命令、风险和 QA 关注点 | return-check / result-check |

## 4. 设计契约映射

| LLD 合同 | 实现动作 | 验证结果 |
|---|---|---|
| F-01/F-02：only trial-return + exact columns | `TrialReturnSourceKindV1` hard deny proxy；Arrow schema固定 `timestamp,simple_return` | columns=`2/2`；proxy accepted=`0` |
| F-03：显式 interval、UTC、finite、non-overlap | observation constructor + `O(n log n)` ordering/overlap guard | missing/wrong/overlap/NaN/Inf path fail closed |
| F-04/F-05：S01 fixture origin/target binding | first operation 调用 `require_action_eligible`，校验 family/run/release/URI | fixture+real accepted/port call=`0/0` |
| F-06：15/15、8/8、domain separation | manifest/seal exact dataclass fields；payload/manifest/seal 分域 hash | field counts=`15/8`；tamper BLOCKED |
| F-07～F-09：唯一 seal bytes/digest/verifier | `__all__` 精确 6 项；唯一三个 function definitions；verifier复算全部 binding | definitions=`1/1/1`；digest regex=`100%` |
| F-10：verified-then-commit | verified value 仅由 verifier 内部 factory 产生；随后 port commit一次 | direct verified construction=0；valid commit=`1` |
| F-11：partial lineage audit only | pure classifier；不 import/call lineage | state=`partial_lineage_blocked_audit`；mutation=`0/3` |
| F-12/F-13：runner/lineage zero diff；S04 不绕 S03 selection | production module无 forbidden imports；verifier拒绝 receipt-only mapping | owner paths=`2/2`；receipt-only accepted=`0` |

## 5. 测试 / Fixture 计划与实现

- 正向 fixture：3 个显式、不重叠 UTC period observations；repository fixture decision/context/URI；in-memory port。
- 完整性负例：truncated payload、manifest mutation、seal binding mutation、selection digest mutation、uppercase/非法 digest。
- 授权负例：fixture decision + real target 在 serializer/port 前拒绝；non-fixture URI 拒绝。
- 语义负例：`forward_label_proxy@v1`、unknown kind/basis、overlap interval 永远不进入 selection/R/effective-count。
- 故障负例：injected port exception 不产生 commit；receipt-only mapping 不能绕过 exact selection type。
- 静态 guard：禁止 runner/lineage imports，禁止 filesystem/network/subprocess adapters；唯一 seal function definition 与唯一 commit call count。

## 6. 最小实现切片与局部验证

| Slice | 产物 | 局部验证 | 结果 |
|---|---|---|---|
| S1 value/schema | definition/identity/observation/payload/manifest/seal values | py_compile | PASS |
| S2 canonical seal/verifier | canonical bytes/digest、bundle/selection/verified | S02 targeted pytest | `12 passed` |
| S3 producer port/audit | authorization binding、verified-then-commit、partial audit | negative/spy/static tests | PASS |
| S4 regression/evidence | S01+S02 combined、structured CP6 artifacts | combined pytest + guards | `58 passed` |

回滚点是删除两个新增 owner 文件；现有 runner/lineage behavior delta=`0`，无数据迁移、pointer 或补偿动作。

## 7. 本地验证

| 命令 | 结果 |
|---|---|
| `PYTHONDONTWRITEBYTECODE=1 PYTEST_ADDOPTS='-p no:cacheprovider' uv run --python 3.11 pytest -q tests/research/test_cr172_trial_return_artifact.py` | PASS：`12 passed, 0 failed, 0 skipped` |
| `PYTHONDONTWRITEBYTECODE=1 PYTEST_ADDOPTS='-p no:cacheprovider' uv run --python 3.11 pytest -q tests/research/test_cr172_path_i_governance.py tests/research/test_cr172_trial_return_artifact.py` | PASS：`58 passed, 0 failed, 0 skipped` |
| `PYTHONPYCACHEPREFIX=/tmp/cr172-s02-pycompile uv run --python 3.11 python -m py_compile engine/path_i_governance.py engine/trial_return_artifact.py tests/research/test_cr172_trial_return_artifact.py` | PASS：exit=`0` |
| `git diff --no-index --check /dev/null engine/trial_return_artifact.py` | PASS：新增文件预期 exit=`1`；whitespace diagnostics=`0` |
| `git diff --no-index --check /dev/null tests/research/test_cr172_trial_return_artifact.py` | PASS：新增文件预期 exit=`1`；whitespace diagnostics=`0` |

## 8. 平台差异与未运行项

Claude/Codex/OpenClaw、安装器和平台发现路径均为 N/A：本 Story 是 repository-local Python pure contract。未运行真实 runner、lake、NAS、runtime、lineage、R、signal、trading、deploy 或 remote operation，原因是 packet 明确禁止且它们不属于本 Story；skipped authorization 不是 waiver，授权/执行计数=`0/6`,`0/6`。

## 9. 文件边界与设计差异

实际代码/测试 touch=`2/2`：

- `engine/trial_return_artifact.py`（created）
- `tests/research/test_cr172_trial_return_artifact.py`（created）

其余允许写入仅为 Story/CP6 证据。forbidden paths touched=`0`；unexpected imports=`0`；design delta=`0`，因此不生成 `process/design-deltas/*`。

## 10. QA / Review / Doc 交接

- QA 必须独立复核 Parquet schema nullable/type/order、3/3 determinism、digest single-truth、tamper 与 proxy/real-target hard deny。
- QA 必须把 fixture PASS 解释为合同证据，不得升级为真实 producer、真实 canonical、empirical R、positive effective count 或 public C1 证据。
- S03/S04 只能复用 exact verifier/result；S04 数据仍须来自 S03 selected replica staging，receipt-only metadata accepted=`0`。
- 剩余风险保持 fail-closed：`R-CR172-TRIAL-RETURN-SOURCE-ABSENT`、`R-CR172-RETURN-DEFINITION-DRIFT`、`R-CR172-PATH-IDENTITY-DRIFT`。真实 native producer 与 atomic lineage 仍是独立前置。

## 11. 阶段建议

CP6 决策建议=`PASS`，next route=`host_dispatch_meta_qa_for_cp7`。如果 QA 发现实现缺陷，返回 `NEEDS_REWORK`；只有需要 runner/lineage/schema 变更时返回 `NEEDS_DESIGN_CLARIFICATION`。

## 12. CP6R1 Finding 回修

### 12.1 Finding 与根因

`F-CR172-S02-CP7-001` 的根因是旧实现把 `RepositoryFixtureTrialReturnPortV1` 定义为只含 `commit_verified` 的结构 Protocol，并以 `hasattr` 接受任意对象。异常包装只能报告失败，不能阻止一个非合规 adapter 先记录再抛错。

### 12.2 最小修复

- 将端口改为模块自有 exact nominal class；publisher 使用 `type(port) is RepositoryFixtureTrialReturnPortV1`，duck typing、任意 subclass 和 record-then-fail 冒充对象调用数=`0`。
- 增加 immutable capability binding，字段覆盖 capability version、repository-owned、decision origin、target kind、scope revision/hash、release/run/family、logical URI、authorization/approval/evidence refs。
- Publisher 在 candidate 生成前与 verifier 返回后、commit 前执行同一完整 binding guard；构建期间发生 binding drift 仍在首次端口调用前拒绝。
- 模块自有 commit 先校验 verified bundle 与 binding，再记录 call attempt；受控失败在 selection/commit-count mutation 前抛出稳定 typed error。初始 selection 为 `None` 或已有 verified selection 时，失败后均保持原值。

### 12.3 回归证据

- 未绑定结构端口：accepted/called/records/selection advance=`0/0/0/0`。
- 非合规 record-then-fail 端口：called/retained records=`0/0`。
- 13 个 capability 单字段 drift：port calls/commit/selection advance=`0/0/0`。
- candidate 生成期间 drift：第二道 pre-call guard 拒绝，port call=`0`。
- 合法 nominal port：call/commit/selection=`1/1/1`。
- 受控原子失败：call=`1`，commit/selection advance=`0/0`；已有 selection 场景保持 object identity 不变。
- S02=`29 passed`；S01+S02=`75 passed`；py_compile 与 whitespace/static guards PASS。

### 12.4 设计与边界

本次只强化既有 repository fixture port 的实现合同；payload `2/2`、manifest `15/15`、seal `8/8`、public `3/3+3/3`、唯一 verifier、partial-lineage 与真实操作 ceiling 均不变。设计 delta=`0`，无需读取或修改 HLD、ADR、Feature、LLD、S01 或 S03-S05。

CP6R1 决策建议=`PASS`，next route=`host_dispatch_meta_qa_for_cp7r1`。

## 13. CP6R2 Authorization Evidence Binding 回修

### 13.1 Finding 与合同判定

CP7R1 证明 nominal capability 已持有 `approval_ref`、`evidence_ref`，但 `_require_verified_matches_port_binding` 只比较 selection 与 manifest identity，未把 sealed authorization evidence 与端口 capability 相关联。该缺口属于既有接口内的实现遗漏；LLD 扩展审查结论为无需新增字段、公共接口或设计差异。

### 13.2 最小修复

- 在既有 `_require_verified_matches_port_binding` 的同一布尔 guard 中增加精确比较：`verified.bundle.seal.authorization_evidence_refs == (binding.approval_ref, binding.evidence_ref)`。
- 使用 tuple 原生等值语义同时约束两个值、顺序和长度；集合化、排序、别名、前缀匹配或容错逻辑=`0`。
- `RepositoryFixtureTrialReturnPortV1.commit_verified` 仍在 `call_count`、selection 和 commit mutation 前调用该 guard；稳定错误码仍为 `FIXTURE_PORT_VERIFIED_BINDING_MISMATCH`。
- Publisher 不增加旁路 validator，继续通过同一 nominal `commit_verified` 路径获得保护；重复真相源=`0`。

### 13.3 回归证据

- 直接 nominal commit 五类负例：approval mismatch、evidence mismatch、swapped refs、extra ref、missing ref，blocked=`5/5`。
- 每个直接负例的 call/commit/selection mutation=`0/0/0`。
- Publisher verifier 后授权引用漂移：对外错误=`FIXTURE_PORT_COMMIT_FAILED`，底层同一 guard 拒绝，call/commit/selection=`0/0/0`。
- 合法 nominal 路径仍为 call/commit/selection=`1/1/1`；受控失败、13/13 capability drift、未绑定结构端口与 record-then-fail 冒充回归全部通过。
- S02=`35 passed`；S01+S02=`81 passed`；py_compile exit=`0`；两个 no-index whitespace diagnostics=`0/2`。

### 13.4 设计、边界与阶段建议

Public API、payload `2/2`、manifest/seal `15/15` 与 `8/8`、唯一 seal bytes/digest/verifier、partial-lineage、runner/lineage/真实资源边界均不变。代码/测试 touch=`2/2`，设计 delta、forbidden touch、unexpected import、真实操作和 waiver=`0`。

`F-CR172-S02-CP7-001` 状态建议=`fully_closed_pending_cp7r2`；CP6R2 决策建议=`PASS`，next route=`host_dispatch_meta_qa_for_cp7r2`。若 CP7R2 仍判定 `NEEDS_REWORK`，workflow health 将超过默认阈值，必须停止自动第三次回修并升级设计澄清或人工仲裁。

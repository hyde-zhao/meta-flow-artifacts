---
story_id: "CR173-S03-golden-failure-boundary-verification"
stage: "CP6"
status: "implemented"
implemented_by: "meta-dev"
implemented_at: "2026-07-17"
design_ref: "process/stories/STORY-CR173-S03-golden-failure-boundary-verification-LLD.md"
design_version: "1.4"
---

# CR173-S03 实现交接

## 1. 实现结论

S03 已按批准的 LLD v1.4 完成 fixture/test-only 实现。新增资产严格限定为一个 synthetic fixture、一个 QAC 模块和一个授权边界模块；`engine/**`、8 个 public production 文件、12 个既有 public regression expected 和 `docs/**` 的修改数均为 0。

本 Story 只证明 standalone offline spectral effective dimensionality 方法的确定性、fail-closed、审计恢复与 estimator-only 边界，不产生 public C1 projection、Gate1 改善、admission PASS、Stage 3 ready 或 CR-172 自动恢复/关闭。

## 2. 实现前置检查

| 检查项 | 结果 | 证据 |
|---|---|---|
| process route health | PASS | `process/state/STATE.current.json` health routing；S03 work packet 已生效 |
| CP5 / LLD | PASS | CP5 已批准；S03 LLD v1.4 `approved/confirmed=true` |
| 依赖门 | PASS | S01 CP7R2 PASS；S02 CP7 PASS |
| Story / 文件所有权 | PASS | `dev-ready`；仅 S03 三个 primary asset 可写 |
| 阻塞 clarification | PASS | `open_items=0`；无需 public contract 修改 |
| 授权 | PASS | fixture-only；真实数据、credential、provider、lake/NAS、runtime、trading、publish/deploy、Git remote 均未授权且操作数为 0 |

## 3. 实现对象与切片

| TASK | 对象 | 实现内容 | 局部结果 |
|---|---|---|---|
| CR173-F01-T09 | `golden_vectors_v1.json` | 6/6 declared-exact synthetic cases；raw/expected view 分离；6 个固定七字段 golden oracle；F01-F08 固定七字段 failure oracle | schema/case/method/hash/integrity guard PASS |
| CR173-F01-T10 | QAC | 6 类各 3 次 raw-only 执行；actual outcome 后置 basis；每组 stable computation/evidence=1/1、audit=3/3；F01-F08 `8×7`；7-field delete/mutation/orphan/forgery；两类 append-only recovery | S03 QAC PASS |
| CR173-F01-T11 | authorization | AST 只读扫描；NP-01..09 唯一 9/9 且各 0；public new-code edge/call/diff/write `0/0/0/0`；public/non-public 无重复 | authorization PASS |
| CR173-F01-T12 | public/claim | 权威 public regression path 12/12 精确存在；existing expected edits=0；standalone true；public C1/computable/CR172 auto route 均 false | 12-path selected regression PASS |

## 4. 关键合同落地

### 4.1 Oracle 隔离

fixture loader 先把 case 拆为 `raw` 与 `expected_*` 两个不相交 view。只有 raw view进入 S01/S02；actual outcome、returned basis、computation ref 和 evidence hash 全部完成后才读取固定 oracle。golden `6×7` 与 failure `8×7` 的 expected 值进入 estimator/finalizer 次数为 0。

### 4.2 稳定 identity 与外置 audit

每个 case 三次执行得到唯一 computation ref 和唯一 evidence hash；ordinal 只进入外置 audit，三个 attempt audit ref 均唯一且链接同一 stable identity。audit collection 仅使用 immutable repository-local in-memory `AttemptAuditLog`，persistence/retention 均为 N/A。

### 4.3 Failure 与完整性

F01-F08 全部返回非 present、count=null；F03 noncanonical token 在 matrix-domain 前停止；F04 只接受 finite exact-rational 后的 domain failure；F08 attempted-evidence digest 只进入第四个 snapshot digest，不扩张三位 presence bitmap。七个 evidence 顶层字段分别删除均拒绝；method/lineage/computation/hash mutation/orphan/forgery 均无法通过 trusted builder/serialization comparison。

### 4.4 Recovery

失败 A 修正为成功 B 时，A 的 basis/computation/evidence/audit 保留；B 的 identity 全新，parent/supersedes 同时指向 A。同一失败重试保持 computation/evidence 不变，仅追加不同 ordinal 的 audit；覆盖旧 attempt 数为 0。

### 4.5 Public 与权限边界

CR173 新代码到 8 个 public production 模块的静态 dependency/call 为 0；S03 changed manifest 与 public production、12 个 existing regression 文件交集均为空。12 个既有路径以只读方式执行 current expected，255/255 PASS；未修改任何 expected。NP-01..09 与 public 六指标分别建账，重复分类数为 0。

## 5. 验证结果

| 层级 | 命令摘要 | 结果 |
|---|---|---|
| S03 局部 | QAC + authorization | 34/34 PASS |
| CR173 targeted | S01 contracts + S02 estimator + S03 QAC/auth | 121/121 PASS |
| public read-only | 权威 12 个 regression/authorization modules | 255/255 PASS |
| combined | targeted + 12/12 public read-only | 376/376 PASS |
| syntax | `py_compile` 两个 S03 test modules | 2/2 PASS |
| diff hygiene | 三个 S03 asset 的 `git diff --check` | PASS；whitespace error=0 |
| write boundary | S03 三文件 untracked；public production/expected modified=0 | PASS |

## 6. 量化验收

| 指标 | 实际 | 结论 |
|---|---:|---|
| golden classes / repeats | `6/6 × 3/3` | PASS |
| stable computation/evidence / external audits | 每组 `1/1 / 3/3` | PASS |
| failure / returned basis | `8/8 / 8×7` | PASS |
| evidence required fields | `7/7` | PASS |
| F03 matrix-domain/estimator calls | `0/0` | PASS |
| NP inventory / duplicate / non-zero | `9/9 / 0 / 0` | PASS |
| public new-code edge/call/diff/write | `0/0/0/0` | PASS |
| public read-only inventory / expected edits | `12/12 / 0` | PASS |
| standalone evidence | `1/1` | PASS |
| public populatable / C1 computable | `false/false` | PASS |
| CR172 auto resume/close | `0/0` | PASS |

## 7. 文件清单与回滚

新增文件：

- `tests/fixtures/effective_trial/golden_vectors_v1.json`
- `tests/research/test_effective_trial_cr173_qac.py`
- `tests/research/test_effective_trial_authorization.py`

回滚仅需删除以上三个 test assets；S01/S02 source 与现有 public C1 不需迁移。没有长期 HLD/ADR/Feature contract 变化，因此 design delta=N/A。

## 8. 剩余风险与 QA 关注点

- `R-CR173-EXACT-INTEGER-GROWTH` 仍按 fixture-only 接受：当前 golden 最大 n=4，不声明 production SLA；CR-172 PATH-C/A 或真实规模目标前重评。
- participation ratio 仍只表示二阶相关结构有效维度，不是 Li-Ji/FWER/DSR calibration 数。
- CP7 应独立重跑 376 项，并重点攻击 fixture oracle 是否可能进入执行路径、authorization counter 是否存在静态漏计、F08 attempted-evidence linkage、append-only recovery 与 public 12/12 的 current-unavailable 语义。
- `offline_method_ready` 只能由后续 CP8 人工终验确认；本 Story CP6/CP7 不得提前声明。

## 9. CP6R1 定向回修：F-CP7-S03-001

### 9.1 Finding 与范围

CP7 首轮发现原静态守卫只保存未绑定的 import/call leaf，导致 `urllib.request.urlopen`、`os.environ.get/getenv`、`os.replace`、`subprocess.run` argv、Git push 以及 public module alias 可形成假阴性。回修只修改 `tests/research/test_effective_trial_authorization.py`；fixture、QAC、S01/S02、`engine/**`、public production 和既有 expected 修改数均为 0。

### 9.2 修复设计

- 第一遍 AST 建立 `import` / `from ... import ...` 的 local-name → qualified-name binding，覆盖显式 alias 与普通 dotted import。
- 第二遍把 `Name`、`Attribute` 和构造器方法调用解析为绑定后的 qualified call；public dependency 与 call 使用完整模块前缀，不再使用不可靠的首段/leaf 推测。
- 对 `subprocess.run/call/check_*`、`Popen`、`os.system/popen` 只解析执行调用的静态 argv；`git push` 唯一归 NP-09，其他 subprocess runtime 唯一归 NP-06。
- 文件写只识别执行语义：write-mode `open`、`os.replace/rename/remove/unlink`、`shutil` 写操作、`Path(...).write_text/write_bytes/unlink/rename/replace`；普通 `str.replace` 不进入 NP-05。
- Credential 识别绑定后的 `os.getenv`、`os.environ.get`、dotenv/keyring 调用；network 识别绑定后的 urllib/requests/httpx/urllib3/socket 调用。
- 普通字符串中的危险词不参与判定；没有 AST execution semantics 的 token 计数保持 0。

### 9.3 对抗性与回归结果

| 验证 | 结果 |
|---|---|
| `urllib.request.urlopen` 普通/alias | NP-04，2/2 命中 |
| `os.environ.get` / aliased `getenv` | NP-01，2/2 命中 |
| `os.replace` / `Path.write_text` / write-mode `open` | NP-05，3/3 命中 |
| 普通 subprocess runtime | NP-06，1/1 命中 |
| `subprocess.run(['git','push',...])` | NP-09，1/1 命中且 NP-06=0 |
| public module import alias / aliased call | dependency/call 均非 0，3/3 命中 |
| 危险字符串 + `str.replace` + read-only `Path.read_text` | 全部 NP/public counter=0，误报=0 |
| authorization module | 9/9 PASS |
| CR173 targeted | 124/124 PASS |
| targeted + public 12 modules | 379/379 PASS |
| `py_compile` | 2/2 PASS |
| diff hygiene | PASS |

F-CP7-S03-001 在实现层已整改，等待独立 CP7R1 复验后才能关闭；本轮不改变 estimator-only claim ceiling 或任何授权范围。

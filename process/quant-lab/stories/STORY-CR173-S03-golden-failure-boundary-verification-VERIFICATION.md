---
story_id: "CR173-S03-golden-failure-boundary-verification"
cr_id: "CR-173"
stage: "CP7"
validation_mode: "mixed"
status: "NEEDS_REWORK"
reviewer: "meta-qa-critical"
verified_at: "2026-07-17T11:58:27+08:00"
source_packet_ref: "process/context/stories/STORY-CR173-S03.CP7.verify-packet.json"
read_expansion_ref: "RE-20260717T035453Z0000-c03q1101"
---

# CR-173 S03 CP7 独立验证报告

## 1. 阶段结论

结论：`NEEDS_REWORK`。

四个 CR-173 targeted 模块与权威 12 个 public read-only 模块组合执行 `376/376 PASS`，`py_compile=2/2`，diff whitespace error=`0`。Golden `6/6×3/3`、F01-F08、七字段 mutation、append-only recovery、public C1 unavailable claim ceiling 的现有测试均通过；8 个 public production path 与 12 个 existing expected path 的实际 Git 变更数也都是 `0`。

但独立对抗性审查确认静态授权守卫存在一个阻断缺陷：真实 network、subprocess/runtime、filesystem write、credential/env、Git remote 以及 public import alias 形态可被简单写入扫描输入，却仍得到 NP-01..09 全零与 public dependency/call 全零。该结果不能支撑批准 LLD 的“任一 forbidden operation/new-code public counter 非零即 fail-closed”结论。S03 不得标记 `verified`，必须回 meta-dev 收紧 guard 并补负向回归。

## 2. 验证范围与非范围

验证范围：

- fixture schema、raw/expected 双 view 隔离与 oracle 非自证。
- Golden `6/6×3/3`，每组稳定 computation/evidence `1/1` 与 audit `3/3`。
- F01-F08 returned basis `8×7`、七字段 delete/mutate/orphan/forged、F03/F04 precedence。
- append-only audit identity 与 recovery A/B parent/supersedes linkage。
- non-public NP-01..09、public 双 lane 六指标、8+12 inventory 与 claim ceiling。
- 四个 CR-173 targeted + 12 个 public read-only regression 模块、语法和 diff hygiene。

非范围：不修改 `tests/**`、`engine/**`、`docs/**`；不创建 public adapter/projection/migration；不执行真实数据、credential、provider、runtime、lake/NAS、交易、publish/deploy 或 Git remote 操作。

按 verify packet 要求完整读取批准 S03 LLD，全文扩展理由=`deep_review`，Host 记录 event=`RE-20260717T035453Z0000-c03q1101`。

## 3. 前置条件与验证对象

| 对象 / 前置 | 验证方式 | 结果 |
|---|---|---|
| process route health | `STATE.current.json` / `CURRENT.json` 指向 S03 CP7，health=`ok` | PASS |
| CP6 implementation gate | CP6 result/return/evidence index | PASS；decision=`PASS` |
| 独立 QA dispatch | `AD-CR173-S03-CP7-META-QA-CRITICAL-20260717T115246+0800` | PASS |
| `golden_vectors_v1.json` | schema/key/case/oracle 结构审查 | PASS |
| `test_effective_trial_cr173_qac.py` | 全文审查、pytest、oracle/recovery 语义审查 | PASS |
| `test_effective_trial_authorization.py` | 全文审查、pytest、独立对抗 AST probe | **FAIL** |
| S01/S02 standalone SUT | 四模块 targeted 回归、只读源代码审查 | PASS |
| 12 个 public regression/authorization 模块 | 精确清单只读组合执行 | PASS，255/255 |

## 4. 验证追踪矩阵

| Requirement / 合同 | 实现 / 验证 | 结果 | 风险 / finding |
|---|---|---|---|
| S03-FR-01 fixture schema 与双 view | `_load_fixture` / `_split_case`；6 case key inventory | PASS | expected key 未进入 raw view |
| S03-FR-02/03 6×3 stable identity/audit | `test_golden_vectors_repeat_three...` | PASS | 每组 computation/evidence=1/1，audit=3/3 |
| S03-FR-04 F01-F08 returned basis | 8 参数化 failure + fixture oracle | PASS | 8×7 全字段 equality；count=null |
| S03-FR-05 七字段完整性 | 7 delete + mutation/orphan/forged tests | PASS | accepted present=0 |
| S03-FR-06 append-only recovery | A/B 与 same-failure repeat tests | PASS | A retained；B linked；overwrite=0 |
| S03-FR-07 public 双 lane | 8 production + 12 regression inventory、Git status、组合回归 | PASS | public path/expected edits=0 |
| S03-FR-08 deny-default authorization | `_scan_python` / `_non_public_operation_counters` + adversarial probe | **FAIL** | `F-CP7-S03-001` |
| S03-NFR-04 current public C1 unavailable | 12/12 existing regressions | PASS | 只证明 unavailable/worst-state 未改善 |

## 5. Oracle、failure 与 recovery 审查

- Fixture 每个 case 的 `expected_*` 与 raw key 集严格分离；`_execute_raw` 只接收 raw mapping。expected oracle 仅在 execution 返回后由 `_assert_fixture_oracle` 逐字段比较，进入 estimator/finalizer 次数=`0`。
- 六个 golden case 都以相同 raw components 重复三次；稳定 computation ref/evidence hash 各 `1`，attempt audit ref 各 `3`，ordinal 只进入外置 audit。
- F01-F08 从 raw injection 获得 actual execution，再与 fixture 中的七字段 expected basis 比较；8 类全部非 present，`effective_trial_count=null`。
- F03 noncanonical token 通过 monkeypatch 计数证明 matrix-domain validator/estimator 调用=`0/0`；F04 仅接收已解析 finite exact-rational 输入。
- 七个 evidence 顶层字段逐一删除全部拒绝；orphan lineage、forged method/hash/computation 与 count/status mutation 全部拒绝。
- Recovery A/B 保留 A，B 生成新 computation/evidence/audit，并令 parent/supersedes 同时指向 A；同一失败重试保持 evidence hash 稳定并追加不同 audit ref。

这些正向与负向合同没有发现 oracle 自证或 recovery 新缺陷。

## 6. 自动化与静态执行证据

| 层 | 命令 / 检查 | 结果 |
|---|---|---|
| Combined regression | `uv run --python 3.11 pytest -q`，4 个 targeted + 12 个权威 public 模块 | PASS，`376 passed in 1.85s` |
| Syntax | `uv run --python 3.11 python -m py_compile`，S03 两个 Python 模块 | PASS，2/2 |
| Diff hygiene | `git diff --check --`，S03 三项资产 | PASS，whitespace errors=0 |
| Path truth | `git status --short --`，7 个 CR173 新资产、8 production、12 expected | PASS；只有 7 个 CR173 新资产为 untracked，8+12 均 0 修改 |
| Current-source forbidden scan | `rg` 检查 network/subprocess/write/credential/runtime/trade/remote 形态 | PASS；SUT 无真实越权实现，测试只有 repository-local `read_text` 与 guard 字面量 |
| Adversarial authorization probe | 以内存 AST 构造真实越权与普通字符串/replace，对当前 scanner/counter 函数计数 | **FAIL**；真实越权全部未命中 |

## 7. Findings（严重度优先）

### F-CP7-S03-001 — HIGH / BLOCKING — authorization AST guard 可被真实越权形态简单绕过

- 位置：`tests/research/test_effective_trial_authorization.py:83-141`，以及 public edge/call 统计 `:151-165`。
- 复现输入：
  - `from urllib import request; request.urlopen(...)`
  - `subprocess.run(["git", "push"])`
  - `os.replace("a", "b")`
  - `os.environ.get("TOKEN")`
  - `from engine import experiment_family_lineage as ef; ef.consume()`
  - 同时加入普通字符串 `"Path.write_text and git push"` 与普通 `"x".replace(...)` 作为非误报对照。
- 实际结果：imports=`engine, os, subprocess, urllib`；calls=`request.urlopen, subprocess.run, os.replace, os.environ.get, ef.consume, replace`；NP-01..09 全部为 `0`；public dependency edges=`0`；public calls=`0`。
- 正确的非误报行为：普通字符串与普通 `replace` 没有被计为写入，结果为 `0`。
- 根因：
  - network 只枚举 `requests/httpx/urllib3/socket`，漏掉 `urllib` 等标准网络入口；
  - write 只识别少数 `Path.*`/`open(mode)`，漏掉 `os.replace/remove/rename`、`shutil.move` 等；
  - runtime/Git 只匹配业务化 leaf 名，漏掉 `subprocess.run/Popen`、`os.system` 及命令参数；
  - credential 只匹配 `getenv/load_dotenv/get_keyring`，漏掉 `os.environ` 等；
  - public import/call 未解析 `from engine import <public_module> as alias` 的 alias 绑定；
  - production diff/write 使用硬编码 `CR173_NEW_CODE_PATHS` 集合交集，而不是由受信 changed-path manifest 输入，新增未登记路径可能不被观察。
- 影响：当前代码确实未执行越权操作，但 guard 无法证明未来/回修 diff 中的 forbidden operation 与 public dependency/call 必然被发现；CP6 声明的 NP-01..09 与 public four-zero 是易绕过的假阴性证据。
- 必须修复：
  1. 建立基于 AST alias binding 的 import/call 解析，覆盖 `Import`、`ImportFrom`、`asname` 与 public module alias；
  2. 对 filesystem/network/subprocess/env/credential/runtime/trading/publish/remote 使用可审计的禁止模块与调用族，至少覆盖上述复现及相邻变体；
  3. 保留 context-aware call classification，确保字符串与普通 `str.replace` 不误计；
  4. changed-path/public-diff 由外部受信 manifest 或 Git 只读结果驱动，并拒绝 missing/extra CR173 path；不得仅自报固定集合；
  5. 新增 adversarial negative tests：上述真实形态全部非零命中，普通字符串/普通 replace 继续为零。

该缺陷是已批准 LLD 的实现缺口，不需要重开 CP5 或修改规范合同。

## 8. 公共边界与 claim ceiling

当前仓库事实仍是：

- `public_effective_trial_count_populatable=false`
- `c1_computable=false`
- `cr172_auto_resume_count=0`
- `cr172_auto_close_count=0`
- public C1 / Gate1 / DSR / admission / Stage 3 positive claim=`0`

12/12 read-only regression 的 PASS 只证明 public C1 继续 `typed_unavailable`、Gate1 blocker 与 worst-state 未改善；不证明 effective count 已 public-populated 或 C1 computable。

## 9. 剩余风险与复验范围

| 风险 | 状态 | Owner / 下一步 |
|---|---|---|
| `F-CP7-S03-001` | OPEN / BLOCKING | meta-dev 修复 authorization guard；meta-qa-critical 独立复验 |
| `R-CR173-EXACT-INTEGER-GROWTH` | 已接受 fixture-only | 最大 n=4；CR-172 PATH-C/A 或真实规模/SLA 前重评 |
| `R-CR173-PUBLIC-MIGRATION` | deferred | future projection CR；本 CR 不接 public C1 |
| `R-CR173-SECOND-ORDER-MODEL-BIAS` | claim-limited | 仅二阶有效维度，不用于 FWER/DSR/admission calibration |

最小复验集：

1. 原 combined 376 项全部通过；
2. network/subprocess/write/credential/git/public alias 的上述复现及相邻变体全部被正确计为非零；
3. 普通字符串、普通 `str.replace`、repository-local read-only fixture 不误报；
4. changed-path manifest 缺失/extra/public intersection 必须 fail-closed；
5. py_compile、diff-check、8 production 与 12 expected 零修改继续通过；
6. claim ceiling 四项继续为 `false/false/0/0`。

## 10. 路由

`NEEDS_REWORK → meta-dev（S03 同 Story） → CP6R1 → meta-qa-critical CP7R1`。

本轮 waiver=`0`，design delta=`0`，无需重开 CP5。S03 完成回修与独立复验前不得标记 `verified`，CR-173 不得进入 CP8 人工门禁。

---

# CP7 复验 1（CP7R1）

## 11. 复验结论

结论：`PASS`。

CP6R1 将 import/from-import alias、qualified call、filesystem write、network、credential/env、subprocess argv、Git push 与 public module alias 纳入 AST 语义绑定。独立复验重放原始绕过并增加 13 个相邻 operation 变体、2 个额外 public alias 变体，未发现同因旁路：13/13 均唯一命中正确 NP 类，public dependency/call 均非零；危险词字符串、普通 `str.replace`、`Path.read_text`、默认只读 `open` 和仅 import subprocess 的误报总数为 0。

自动化结果为 authorization `9/9 PASS`、CR173 targeted `124/124 PASS`、targeted + 12 个 public read-only `379/379 PASS`；`py_compile=2/2`、diff-check PASS。8 个 public production 与 12 个 existing expected 路径实际修改数仍为 0。因此 `F-CP7-S03-001` 关闭，S03 可标记 `verified`。

## 12. 复验范围与对象

- 权威入口：`process/context/stories/STORY-CR173-S03.CP7.reverify-1.packet.json`。
- Finding 输入：CP7 首轮 result；回修输入：CP6R1 result/return/evidence。
- 工程对象：`tests/research/test_effective_trial_authorization.py`；QAC、fixture、S01/S02 与 public 模块均只读回归。
- 非范围：不修改 tests/engine/docs，不执行真实 network、filesystem write、credential、runtime、provider、交易、publish/deploy 或 Git remote。
- 本轮没有全文扩展读取；只读取 packet 的 must_read / allowed_reads。

## 13. 回修设计与对抗性审查

| 合同 | 回修机制 | 独立复验证据 | 结果 |
|---|---|---|---|
| Import / from-import alias | 建立 local-name → qualified-name binding | `urllib.request`、`httpx.get`、`socket.create_connection` 等 alias/qualified 变体 | PASS |
| Credential/env | 匹配 resolved `os.getenv`、`os.environ.get`、dotenv/keyring | `env.get`、keyring alias 共 2 个相邻变体 | PASS；唯一 NP-01 |
| Network | resolved qualified call + network prefix | urllib plain、httpx from-import、socket alias 共 3 个相邻变体 | PASS；唯一 NP-04 |
| Filesystem write | open mode、Path method、os/shutil/io write family | os.rename、Path.replace、io append、动态 open mode 共 4 个相邻变体 | PASS；唯一 NP-05 |
| Runtime / Git | subprocess/os command target + literal argv | dynamic subprocess、git string、`/usr/bin/git push`、git status 共 4 个相邻变体 | PASS；runtime/git 唯一归 NP-06/NP-09 |
| Public import/call | qualified public module prefix | strategy admission package 的 module/function alias 2 例 | PASS；结果 `(1,1)` / `(2,1)` |
| False positive | 不扫描纯字符串；按 resolved call 语义区分 read/write | 危险字符串、`str.replace`、`Path.read_text`、默认只读 `open`、仅 import subprocess | PASS；NP/public/write=0 |

原始 finding 的五类关键绕过均被现有 9 个回归样例覆盖；独立 13+2 变体未复用这些源码样例，证明回修不是只针对固定拼写。

## 14. 自动化与边界证据

| 层 | 结果 |
|---|---|
| Authorization adversarial | `9 passed in 0.10s` |
| CR173 targeted 4 modules | `124 passed in 0.31s` |
| Targeted + public read-only 12 modules | `379 passed in 1.76s` |
| 独立相邻 operation probe | 13/13 唯一分类；旁路=0 |
| 独立 public alias probe | 2/2 dependency/call 非零 |
| 独立 safe probe | false positive=0 |
| Syntax | `py_compile=2/2 PASS` |
| Diff hygiene | whitespace errors=0 |
| Path truth | 8 production + 12 expected 修改=0；仅 CR173 七个新增资产出现于 status |

## 15. Finding 关闭审计

### F-CP7-S03-001 — CLOSED

- 原因已关闭：scanner 现在以 AST binding 解析 import alias 与 qualified call；subprocess argv 将普通 runtime 与 Git push 分开；filesystem/network/env/public 模块均按调用语义计数。
- 原始复现：`urllib.request.urlopen`、`subprocess.run(["git","push"])`、`os.replace`、`os.environ.get` 与 public import alias 均被识别。
- 同类扩展：13 个 operation 变体全部唯一分类，2 个 public alias 变体全部检测，假阴性=`0`。
- 误报约束：普通字符串、普通 `str.replace`、repository-local `read_text` 与只读 `open` 均为 0，假阳性=`0`。
- Path inventory：本轮以独立 Git status 对 7 个 CR173 新资产、8 个 public production、12 个 existing expected 做事实交叉验证；production/expected 修改=`0/0`。
- 结论：满足批准 LLD 的本 CR 静态授权验证边界；不产生 runtime authorization，也不把测试守卫提升为通用安全沙箱。

## 16. Claim ceiling 与剩余风险

- `public_effective_trial_count_populatable=false`
- `c1_computable=false`
- CR-172 auto-resume/auto-close=`0/0`
- 12/12 public regression 继续只证明 current C1 unavailable / worst-state 未改善。
- `R-CR173-EXACT-INTEGER-GROWTH` 保持 fixture-only 已接受风险；真实规模、性能目标或 CR-172 PATH-C/A 前必须重评。
- `R-CR173-PUBLIC-MIGRATION` 继续 deferred 到 future projection CR。

本轮 blocker=`0`、waiver=`0`、design delta=`0`。

## 17. 路由

`PASS → S03 verified → CR-173 release-readiness / CP8 preparation`。

该路由只允许准备 CP8 人工门禁；不自动批准 CP8，不授权 public C1、真实数据、runtime、Stage 3、admission、部署或远程写入。

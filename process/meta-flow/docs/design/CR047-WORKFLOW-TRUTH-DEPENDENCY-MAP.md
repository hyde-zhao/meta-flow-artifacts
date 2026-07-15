---
status: accepted
version: "1.1"
cr_ref: "CR-047"
---

# CR-047 Workflow Truth Dependency Map

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.1 | 2026-07-14 | host-orchestrator-inline / meta-se | CP3 R2：增加 CR-046 protected-object 身份解析、CP6/CP7 hash 校验依赖及禁止 prefix-only/原件写入方向。 |
| 1.0 | 2026-07-13 | host-orchestrator-inline / meta-se | 建立 Feature/模块单向依赖、禁止方向和循环消除规则。 |

## 依赖关系

| From | To | 依赖类型 | 允许方向 | 原因 | 验证 / 监控 |
|---|---|---|---|---|---|
| FEAT-WT-01 Truth Consistency | workspace routing health | precondition/read | allowed | 真相检查前须确认 process 路由有效 | workspace check fixture |
| CURRENT builder | State v2 + CR catalog + existing refs | read/projection | allowed | CURRENT 是派生发现层 | current projection tests |
| CR tracking | State v2 reader + formal CR/index/follow-up | read/validation | allowed | 检测 active/lifecycle/candidate 冲突 | cr-tracking tests |
| FEAT-WT-02 Artifact Routing | filesystem/Git layout | read/write metadata | allowed | 路由 owner 只写 link metadata | workspace routing tests |
| FEAT-WT-03 Quality Governance | policies + CP results + ledgers | read/derived result | allowed | 所有 metric 由 machine evidence 派生 | quality/token/context doctor tests |
| FEAT-WT-04 Delivery Integrity | tracked delivery rule + installer + Git/package classifier | read/validation | allowed | clean clone 只能依赖 tracked/package input | installer/guardrail tests |
| FEAT-WT-05 Operator & Status | FEAT-WT-01/03/04 results | read/documentation | allowed | README/status 是派生说明 | docs contract tests |
| ST-WT-007 firewall | CR-046 formal CR + evidence index + CP/Story refs + ledger attribution | read/identity/hash | allowed | 按对象身份捕获 protected originals 与原始 SHA256 | CP6 manifest + CP6/CP7 hash fixtures |
| Release preflight | Truth → Doctor → guardrail → Ruff → pytest → installer dry-run | serial orchestration | allowed | 前置 blocker 必须先暴露，Run event 按实际执行写入 | TC-WT-001..006 |

## 禁止依赖

| Forbidden ID | From | To | 禁止原因 | 替代路径 | 违反风险 |
|---|---|---|---|---|---|
| FD-WT-01 | State/CR owner | CURRENT projection | 派生视图不得反向覆盖真相 | 受控 state writer / CR status-sync | 双向同步产生不可解冲突 |
| FD-WT-02 | Internal docs writer | root `docs/product|design|quality` | 根路径不是 artifact canonical | `process/docs/**` | clean clone 再现本机副本漂移 |
| FD-WT-03 | Doctor | manual dashboard/ranking | 质量结论必须 derived-only | CP results + ledgers + policies | 手工数值覆盖真实 blocker |
| FD-WT-04 | Budget remediation | original historical CP/result/ledger overwrite | 破坏 hash 和时序 | summary/index/hash/correction/archive | 历史伪造为 PASS |
| FD-WT-05 | Clean-clone guardrail | ignored root `AGENTS.md` | ignored 本机文件不可成为必需输入 | tracked `delivery/rules/AGENTS.md` + installer dry-run | clean clone 必然失败 |
| FD-WT-06 | Guardrail | all local cache as one class | ignored local 与 tracked/package 风险不同 | Git/package classifier | 要么永久红灯，要么泄漏交付缓存 |
| FD-WT-07 | CR-047 | prelink backup or quant-lab business files | 用户显式排除，且与目标无关 | 保持 untouched | 越权修改用户数据 |
| FD-WT-08 | CR-046 status convergence | platform/runtime claims | 仓库无该证据 | unavailable/READY_WITH_RISK + follow-up | 虚假证明 |
| FD-WT-09 | CR-047 / ST-WT-007 | CR-046 protected originals | 本 CR 只允许 read/reference 或新增 append-only correction/projection | 新 correction/status projection 或独立子 CR replacement/supersession | 原 hash/时序被静默改写 |
| FD-WT-10 | Protected-object resolver | directory prefix as sole identity/allowlist | CR-046/047 对象在同目录混居，prefix 会误伤或漏判 | source CR + object type + canonical/provenance ref；SHA256 仅作完整性锚 | firewall 假阴性或假阳性 |

## 顺序与文件所有权原则

| 顺序 | 输入 | 输出 | 合并规则 |
|---:|---|---|---|
| 1 | workspace route metadata | healthy canonical route | 路由未通过不允许后续写 process |
| 2 | State/CR/index | truth consistency + CURRENT rebuild | State writer、CR status-sync、projection builder 各自单写 |
| 3 | truth result + retention/quality policies | Doctor result | 诊断不修改历史输入 |
| 4 | tracked source + package classifier | guardrail result | generated wrapper 只在 installer target 中验证 |
| 5 | Python source/tests/docs | Ruff/pytest/installer/status result | 共享 regression 可串行，Story verdict 独立 |
| 6 | CP6 pre-implementation manifest | CP6 首验 → CP7 再验 | 任一身份不明或 hash 变化即停止 ST-WT-007；不得进入后续状态投影关闭 |

## 循环风险

| Cycle ID | 涉及对象 | 风险 | 当前处理 |
|---|---|---|---|
| CYCLE-WT-01 | State ↔ CURRENT | projection 反写会产生不可判定 owner | eliminated；只允许 State → CURRENT |
| CYCLE-WT-02 | CR index ↔ Markdown STATE summary | legacy parser 可将摘要当真相 | eliminated；State v2 + JSON index 优先，Markdown 仅 render |
| CYCLE-WT-03 | Doctor finding → history rewrite → Doctor PASS | 为了“修绿”改写输入 | eliminated；append-only correction + scoped classification |
| CYCLE-WT-04 | root wrapper ↔ delivery rule | 双向同步无法判定 canonical | eliminated；delivery rule 单向生成 wrapper |
| CYCLE-WT-05 | CR-046 original → CR-047 projection → original rewrite | current 说明反向污染历史证据 | eliminated；projection/correction 单向追加，原件由 manifest/hash firewall 保护 |

## Gotchas

- 共享文件冲突不能只靠 Wave 解决；后续 Story plan 必须给 `cr_tracking.py`、`current.py`、guardrail 等文件指定单一 merge owner。
- Doctor 对归档分类的依赖不可反向修改 CR lifecycle；lifecycle 仍由 CR owner 写入。
- 整个 release preflight 串行不意味着实现 Story 必须串行；Story 并行由文件 owner 和 DAG 决定。

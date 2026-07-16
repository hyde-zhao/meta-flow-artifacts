---
status: archived-changes-requested
version: "1.0.1"
created_at: "2026-07-16"
owner: "host-orchestrator-inline / meta-se"
cr_ref: "CR-050"
review_gate: "CP3"
superseded_by_scope_change: "CP2 R2 explicit merge baseline; ADR revision pending after approval"
---

# CR-050 Git Branch Lifecycle Architecture Decisions

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-16 | host-orchestrator-inline / meta-se | 提出原生 Git/coordinator、跨仓单调执行、evidence/recovery 与 proof/risk ceiling 四项决策，等待 CP3。 |
| 1.0.1 | 2026-07-16 | host-orchestrator | 仅更新审查状态：ADR-GB-004基于verify-only产品边界，已被用户要求的显式merge范围改变；保留原决策正文，待CP2 R2后生成新ADR提案。 |

## ADR-GB-001：使用原生 Git lifecycle service 与显式 CR Start Coordinator

- 状态：proposed / CP3-DQ-01
- 背景：现有 `workspace git-status/push` 已有 stdlib subprocess与双仓发现，但 CR branch需要 identity/default/proof/policy；若 formal CR先写artifact，clean precheck会失败；若直接修改旧 `cr bootstrap` 默认行为，旧调用会意外产生远端写。
- 决策：复用/扩展 native Git adapter，新增 branch lifecycle planner/executor；增加显式 governed start入口，由 coordinator执行 intent无写校验、local branch prepare、现有 CR bootstrap、remote tracking push。旧 bootstrap和workspace push默认行为不变。
- 备选：安装 Git Town/`gb`；Host 手工Git；让旧bootstrap默认创建/push branch。
- 理由：不增加必需依赖，保持argv-only和bare remote测试；显式入口承载远端授权，解决clean/CR-first顺序冲突。
- 影响：需新的operation types/result、CLI与coordinator integration；start可能留下可恢复local/remote partial。
- 回退/切换：若 coordinator无法复用writer，退回“显式 prepare + bootstrap + publish”三步并保存同一intent/result链；不得放宽clean gate或让旧bootstrap静默写远端。真实stacked需求出现时另开adapter CR。

## ADR-GB-002：跨仓采用 preflight-all、单调执行与观察式恢复

- 状态：proposed / CP3-DQ-02
- 背景：Git不提供project/artifact跨仓事务；网络或remote policy可在任一步变化。用delete/recreate/force补偿会扩大损害并破坏真实历史。
- 决策：所有仓先完成适用precheck；按固定project→artifact顺序执行；每个step立即post-check；成功fact不自动撤销；失败输出per-repo PASS/NO_CHANGE/BLOCKED/PARTIAL/FAILED/SKIPPED与resume入口。finish中先两仓proof，再两仓recovery/default准备，再remote deletes，最后local deletes。
- 备选：自动补偿/force回滚；只操作一个仓；并行执行两仓。
- 理由：最大化审计与恢复确定性；并行不能降低事务风险，反而让partial顺序不稳定。
- 影响：overall可能PARTIAL，需要operator重试；某仓成功ref可早于另一仓存在。
- 回退/切换：只有底层提供可验证的真正跨仓transaction才重评。当前任何partial都保留并重新观察，不以旧plan盲目继续。

## ADR-GB-003：Git refs 持有事实，append-only attempt 持有证据

- 状态：proposed / CP3-DQ-03
- 背景：finish需要known published tip处理remote auto-delete；只打印文本不可跨设备恢复，而新branch state数据库会与Git ref/CR state形成第三套truth。
- 决策：local/remote refs始终是branch事实；每次operation生成versioned、append-only result，记录intent/plan digest、before/after OID、steps、authorization、per-repo terminal status与recovery route，并引用现有RUN/CR ledgers。重试写新attempt/supersession，不覆盖。删除前建立local-only `refs/meta-flow/recovery/<cr>/<fingerprint>`。
- 备选：只stdout；仅OID字符串无ref；新branch state DB；自动push recovery tag。
- 理由：result能证明“当时观察/执行”，recovery ref能抵抗local branch删除后的对象不可达；两者都不冒充merge proof或远端truth。
- 影响：artifact repo增加小型result/ledger事件；local recovery refs需未来retention治理；early start failure可能只有terminal JSON，必须诚实披露未持久化。
- 回退/切换：recovery ref collision且OID不同即BLOCKED；retention需求另立CR。不得在finish自动删除或push recovery ref。

## ADR-GB-004：只接受 Git ancestry 正证明，并叠加计算风险上限

- 状态：proposed / CP3-DQ-04
- 背景：remote branch absence、CR close、patch相似或squash/rebase均不能证明exact CR tip进入default。用户禁用子Agent，当前验证也缺独立QA。
- 决策：MVP positive cleanup proof仅为known exact tip对fresh remote default tip的`merge-base --is-ancestor`成功，并同时满足identity/tip/protected policy；否则删除数为0。有效inline fallback下CP7最高`PASS_WITH_RISK`、CP8最高`READY_WITH_RISK`；无有效验证证据则BLOCKED。
- 备选：patch-id/内容相似推断；remote absence即PASS；CP7前取得独立QA/forge receipt；要求READY时阻断。
- 理由：fail closed避免不可恢复误删；风险上限诚实反映证据来源，不把fixture写成forge/runtime attestation。
- 影响：squash/rebase merge不能由MVP清理；可能需要人工保留branch并创建future forge adapter follow-up。
- 回退/切换：目标forge提供可验证identity/token/PR merge receipt且用户批准独立CR后，可新增receipt proof adapter；新证据只解除对应风险，不自动关闭partial/real-pilot/independent-QA等维度。

## 决策一致性

| HLD/Blueprint 契约 | ADR |
|---|---|
| native argv-only、显式 start、旧命令兼容 | ADR-GB-001 |
| deterministic paired operation、partial/resume | ADR-GB-002 |
| ref truth、attempt evidence、recovery ref | ADR-GB-003 |
| proof-gated delete、squash fail closed、verdict ceiling | ADR-GB-004 |

四项决策同时成立；若 CP3 修改任一项，必须同步 Blueprint、Domain Map、Dependency Map与HLD修订记录，不能只改本文件。

## Gotchas

1. coordinator不是跨仓事务或新release orchestrator，只负责这一项CR start顺序。
2. operation result记录的OID是证据，不保证对象永久可达；recovery ref承担本地可达性。
3. local recovery ref不同于用户可见release tag，不得自动push。
4. ancestry positive仍需先通过exact identity/tip/protected检查，不能单独授权删除任意ref。

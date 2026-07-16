---
status: accepted
version: "1.2"
created_at: "2026-07-16"
owner: "host-orchestrator-inline / meta-se"
cr_ref: "CR-050"
review_gate: "CP3-R3"
supersedes: "process/archive/CR-050/design/CR050-GIT-BRANCH-ARCHITECTURE-DECISION.v1.0.1.md"
---

# CR-050 Architecture Decisions

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-16 | host-orchestrator-inline / meta-se | 提出native lifecycle、non-atomic pair、append-only evidence/recovery、ancestry-only cleanup。 |
| 1.0.1 | 2026-07-16 | host-orchestrator | 显式merge改变产品边界，标记changes-requested并归档。 |
| 1.1 | 2026-07-16 | host-orchestrator-inline / meta-se | 重述为ADR-GB-R2-001..005：同一service的独立merge、artifact-first PARTIAL、typed authz+exact FF、current merge evidence+finish reproof、叠加风险上限。 |
| 1.2 | 2026-07-16 | host-orchestrator-inline / meta-se | CP3 R3补强ADR-GB-R2-002：artifact-first采用治理预写+2/2聚合投影门；PARTIAL不得推进paired projection/CR close；无法实现该门时切project-first并重开CP2/CP3。 |

## ADR-GB-R2-001：扩展既有原生Git lifecycle，不新建总编排器

- 状态：Accepted / CP3-R3-DQ-01
- Context：需要四个显式动作，又必须保留旧bootstrap/workspace命令兼容和source-owner边界。
- Decision：采用Option A；CLI/Host显式调用同一typed lifecycle service的open/publish/merge/finish handlers。open coordinator复用existing bootstrap writer；旧入口不新增隐式remote write。Git Town/`gb`不是必需依赖。
- Consequences：公共types/policy/result可复用；模块需处理PARTIAL。不得建立branch DB或release orchestrator。
- Alternatives：外部脚本、新orchestrator、finish隐式merge，均拒绝。
- Switch：出现独立forge/queue owner、credential、receipt和release cadence时，另立adapter CR/HLD。

## ADR-GB-R2-002：paired merge采用preflight-all、artifact→project、非补偿PARTIAL

- 状态：Accepted / CP3-R3-DQ-02
- Context：Git没有跨仓ref transaction；两次remote update间可能发生竞态。artifact-first的PARTIAL若被状态投影误读，会形成“过程真相领先源码”的假完成；project-first则会形成“源码已进default但治理证据滞后”的窗口。
- Decision：保留CP2批准的artifact→project，但把它定义为受约束治理预写：所有仓先fresh只读preflight；artifact先保存已批准治理/证据基线，project后写高风险源码default；每仓即时post-check。只有2/2 repo outcome均为PASS/NO_CHANGE时，独立Paired Projection Gate才允许现有CR/state/current writer投影overall PASS、finish eligibility或close。artifact成功/project失败只追加PARTIAL attempt，`paired_projection_advanced=false`、CR保持active、2/2 CR branches保留、禁止rollback。
- Consequences：避免源码default领先canonical治理基线，同时把单仓过程事实与跨仓workflow truth严格分离；代价是必须有可fixture证明的2/2聚合门，且默认分支仍可能短暂不一致。resume必须fresh observe并写新attempt。
- Alternatives：project-first可降低truth-ahead风险但允许payload-ahead-governance；并行不可确定；delete/reset/force补偿扩大风险，均不作为当前主选。
- Switch：若CP4/CP5无法证明`PARTIAL => paired_projection_advanced=false && cr_closed=false && finish_allowed=false`，立即切project-first并重开CP2/CP3；若未来有可验证跨仓transaction可再次重评顺序和原子性。

## ADR-GB-R2-003：default write要求typed operation authz与exact ordinary fast-forward

- 状态：Accepted / CP3-R3-DQ-03
- Context：CP gate approval不能长期授权真实默认分支写；远端可能要求保护规则/PR queue。
- Decision：每次actual merge提供绑定operation/repo/remote/default/branch/expected OID的explicit authz。写入只能是普通exact ref update；fresh default必须是published tip祖先/相等；禁止merge commit、rebase、force/force-with-lease与策略绕过。远端拒绝保留为BLOCKED/PARTIAL。
- Consequences：native路径无法处理强制PR/queue，但不会伪装成功或扩大权限。
- Alternatives：gate blanket authz、force-with-lease、静默forge fallback，拒绝。
- Switch：受保护平台需求通过独立forge adapter CR，定义credential和receipt契约。

## ADR-GB-R2-004：attempt evidence只建立eligibility，finish必须fresh reproof

- 状态：Accepted / CP3-R3-DQ-04
- Context：merge result可能过时，且default write authority与branch delete authority不同。
- Decision：operation attempts append-only并关联existing ledgers；只有current matching 2/2 merge PASS使finish eligible。finish仍需独立delete authz、fresh fetch、identity/tip/protected/ancestry重证与local recovery refs；不得由merge handler直接删除。
- Consequences：安全边界清晰，重复检查增加少量成本；不建立branch state DB。
- Alternatives：merge成功即删、只stdout、新DB、CR closed即proof，拒绝。
- Switch：future trusted forge receipt只能作为额外proof source，不能替代identity/authz/protected校验。

## ADR-GB-R2-005：no-subagent下采用叠加式验证/发布上限

- 状态：Accepted / CP3-R3-DQ-05
- Context：用户要求不使用子Agent；Host inline fixture不是独立QA或平台runtime attestation。
- Decision：若CP7没有可验证独立QA dispatch/return/receipt，CP7最高PASS_WITH_RISK、CP8最高READY_WITH_RISK；不得声称independent-QA-verified、platform-attested或real-default-write-verified。此上限与branch protection、forge receipt、real pilot风险叠加独立。
- Consequences：能够诚实交付确定fixture，但不能达到无风险READY。
- Alternatives：CP7前解除限制并取得独立QA；若用户要求READY则阻断。
- Switch：补齐某项证据仅解除对应风险维度。

## 一致性映射

| HLD/Blueprint契约 | ADR |
|---|---|
| 四动作显式、同一service、无新orchestrator/DB | ADR-GB-R2-001 |
| artifact-first治理预写、2/2聚合投影门、PARTIAL/no rollback | ADR-GB-R2-002 |
| operation/OID authz、ordinary exact ff、remote policy fail closed | ADR-GB-R2-003 |
| current 2/2 merge→finish eligibility；fresh reproof/recovery | ADR-GB-R2-004 |
| CP7/CP8叠加式风险上限 | ADR-GB-R2-005 |

五项必须整体一致；任一CP3修改都要同步Blueprint、Domain、Dependency、HLD与本ADR的修订记录。

## Gotchas

1. ADR-002的顺序不是“artifact更重要”；它以2/2聚合门为安全前提，先固化治理基线再写源码default。单仓artifact ref绝不等于paired merge PASS。
2. ADR-003不承诺远端接受direct push；拒绝是受治理结果。
3. ADR-004中的merge PASS不是删除授权，只是finish入口条件之一。
4. ADR-005不会因CP3由用户批准而解除；架构人工决策与独立运行验证是不同证据。
5. 若实现只能从artifact default推导workflow状态、不能延迟投影到2/2 post-check之后，则当前顺序不成立，必须按Switch回退而不是在LLD中弱化不变量。

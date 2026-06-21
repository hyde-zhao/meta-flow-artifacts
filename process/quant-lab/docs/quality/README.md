---
status: "current-index"
version: "1.1"
change: "CR-109"
created_at: "2026-06-21T22:05:41+08:00"
created_by: "host-orchestrator"
---

# Quality Current Index

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-06-21 | host-orchestrator | 按 CR-108 新增质量文档 current index，区分历史 CR 质量证据、当前可消费 guardrail 和不授权边界。 |
| 1.1 | 2026-06-21 | host-orchestrator | 按 CR109 将历史 CR 质量证据从顶层平铺迁入 `by-feature/<feature>/by-cr/<CR>/`。 |

## 定位

`docs/quality/` 保存质量评审、测试报告、验证报告、修复记录和 guardrail 文档。多数 `REVIEW-*`、`TEST-REPORT-*`、`VERIFICATION-REPORT-*`、`FIXES-*` 文件是对应 CR 的历史证据，不应被批量改写，也不应被解释为当前 runtime、NAS、QMT、provider、lake、publish 或交易授权。

当前读取顺序：

1. 需要当前质量边界时，先读本文件。
2. 需要 factor research guardrail 时，读 `docs/quality/FACTOR-RESEARCH-GUARDRAILS.md`。
3. 需要某个 CR 的历史验证事实时，读对应 `docs/quality/by-feature/<feature>/by-cr/<CR>/` 文件，并按该 CR 的授权边界解释。

## 文档分类

| 类别 | 路径模式 | 当前含义 | 默认处理 |
|---|---|---|---|
| Current quality index | `docs/quality/README.md` | 当前质量文档入口和边界说明。 | 可随 current truth 修订。 |
| Current guardrail | `docs/quality/FACTOR-RESEARCH-GUARDRAILS.md` | 当前 factor research 相关 guardrail。 | 作为当前 guardrail 消费；修改需走 CR。 |
| Historical review | `docs/quality/by-feature/<feature>/by-cr/<CR>/*REVIEW*.md` | 对应 CR 的历史质量评审。 | legacy audit keep。 |
| Historical test report | `docs/quality/by-feature/<feature>/by-cr/<CR>/*TEST-REPORT*.md` | 对应 CR 的历史测试结果。 | legacy audit keep。 |
| Historical verification report | `docs/quality/by-feature/<feature>/by-cr/<CR>/*VERIFICATION-REPORT*.md` | 对应 CR 的历史验证结论。 | legacy audit keep。 |
| Historical fixes | `docs/quality/by-feature/<feature>/by-cr/<CR>/*FIXES*.md` | 对应 CR 的历史修复摘要。 | legacy audit keep。 |

## 历史质量证据入口

| 功能域 | 路径 |
|---|---|
| package-identity-governance | `docs/quality/by-feature/package-identity-governance/by-cr/` |
| process-ledger-tooling-hygiene | `docs/quality/by-feature/process-ledger-tooling-hygiene/by-cr/` |
| qmt-gateway-readonly | `docs/quality/by-feature/qmt-gateway-readonly/by-cr/` |
| qmt-miniqmt-dual-target-framework | `docs/quality/by-feature/qmt-miniqmt-dual-target-framework/by-cr/` |
| quant-lab-migration-dry-run | `docs/quality/by-feature/quant-lab-migration-dry-run/by-cr/` |
| runtime-authorization-safety | `docs/quality/by-feature/runtime-authorization-safety/by-cr/` |
| strategy-research-lifecycle | `docs/quality/by-feature/strategy-research-lifecycle/by-cr/` |

## 不授权边界

读取或引用 `docs/quality/` 历史报告不授权以下动作：

- 读取 `.env`、token、API key、cookie、私钥、账号或任何凭据。
- 启动 runtime、SaaS、trace upload、production write、真实交易或真实发布。
- 打开 QMT / MiniQMT / GUI、调用 broker API、查询账户、发单、撤单。
- 拉取真实 snapshot、写真实 broker lake、provider fetch、写真实 lake 或 publish。
- 启动 CR-033，或关闭 / 处理 CR102-CR104。

## 历史引用解释规则

| 历史内容 | 当前解释 |
|---|---|
| `project_id: local_backtest` | 历史项目身份事实；当前 canonical 为 `quant-lab`，legacy alias 保留。 |
| `meta-po` | 历史编排角色事实；当前由 Host Orchestrator 主进程承担 CR / CP 编排。 |
| runtime authorization gate | 历史 CR 的授权边界；当前真实运行仍需新的 Host Orchestrator CR / CP / human-gate。 |
| real / readonly / NAS / QMT smoke | 仅表示对应 CR 当时的证据；不得自动升级为当前授权或当前 readiness。 |

## 当前 CR108 结论

CR109 不批量重写历史质量报告正文。若后续发现某个历史质量报告被当前 README、USER-MANUAL、HLD、Feature DESIGN 或 release context 当作当前事实源，应只修当前引用或新增明确边界说明。

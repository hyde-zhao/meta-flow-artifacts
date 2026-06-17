---
project_id: "meta-flow"
release_scope: "meta-flow-artifact-routing-and-eval-governance"
created_at: "2026-06-17T13:49:25+08:00"
---

# Deploy Checklist

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-06-11 | host-orchestrator | 新增 workflow eval governance 部署检查 |
| 1.1 | 2026-06-17 | host-orchestrator | 增加 artifact repo、workspace health 和 advanced eval runner 检查 |

## 检查项

| 项 | 命令 / 证据 | 状态 |
|---|---|---|
| Source repository status | `git status --short --branch` in `/home/hyde/projects/meta-flow` | PASS |
| Artifact repository status | `git status --short --branch` in `/home/hyde/projects/meta-flow-artifacts` | PASS |
| Workspace health | `meta-flow workspace check` | PASS |
| CR tracking | `meta-flow check cr-tracking --project-root .` | PASS |
| Guardrail | `scripts/check_delivery_guardrails.py` | PASS |
| Basic eval validate | `meta-flow eval validate --eval evals/fixtures/generated-workflow-basic/WORKFLOW-EVAL.yaml` | PASS |
| Basic eval run | `meta-flow eval run --eval evals/fixtures/generated-workflow-basic/WORKFLOW-EVAL.yaml --out /tmp/meta-flow-eval-existing` | PASS |
| Advanced eval validate | `meta-flow eval validate --eval evals/fixtures/generated-workflow-advanced/WORKFLOW-EVAL.yaml` | PASS |
| Advanced eval run | `meta-flow eval run --eval evals/fixtures/generated-workflow-advanced/WORKFLOW-EVAL.yaml --out /tmp/meta-flow-eval-advanced` | PASS |
| External adapters | `evals/adapters/ADAPTERS.md` | DISABLED_BY_DEFAULT |

## 不授权项

- 不执行外部 Promptfoo / DeepEval / Langfuse / Garak。
- 不读取凭据。
- 不上传 trace。
- 不执行 publish / live / production 写入。
- 不替换或删除用户未明确确认的 artifact 目录。

---
story_id: "ST-WT-004"
cr_id: "CR-047"
stage: "CP6"
status: "implemented_with_risk"
execution_mode: "inline-fallback"
implemented_at: "2026-07-15"
---

# ST-WT-004 Implementation

## 实现对象

- `scripts/check_delivery_guardrails.py`：tracked `delivery/rules/AGENTS.md` 为 canonical；ignored 根 wrapper 可选。tracked/package-input cache 为 blocker，ignored local cache 为 warning，并去重 cache 目录下 pyc。
- `.gitignore`：明确根 `AGENTS.md`/`CLAUDE.md` 是生成 wrapper。
- `tests/test_cr047_delivery_gate.py`：覆盖 wrapper 缺失、canonical 缺失和 cache 优先级。

## 契约映射

clean clone 不依赖 ignored 文件；发布输入按 Git tracked/clean staging 判定。根 wrapper 存在时仍可检查漂移，但缺失本身不阻断。

## 验证与边界

从 `git archive HEAD` 构造的 clean staging 在不补根 `AGENTS.md` 时 guardrail 通过。当前工作树 ignored cache 只输出 warning；任何 tracked cache 仍会失败。未删除用户 cache，未修改安装目标或发布远端。

## 设计差异与交接

无设计差异。ignored cache warning 是明确风险分类，不代表可把 cache 打进交付包。

---
story_id: "CR037-S13"
cr_ref: "CR-037"
title: "quant-lab migration dry-run and reports"
status: "dev-ready"
wave: "CR037-W5"
priority: "P2"
depends_on: ["CR037-S01", "CR037-S05", "CR037-S07", "CR037-S08", "CR037-S09", "CR037-S10", "CR037-S11", "CR037-S12"]
feature_design_refs:
  - "process/docs/features/quant-lab-migration-readiness/DESIGN.md"
  - "process/docs/features/quant-lab-migration-readiness/TEST-PLAN.md"
  - "process/docs/features/quant-lab-migration-readiness/TASKS.md"
lld_policy:
  required_level: "full-lld"
  trigger_reasons: ["migration", "security", "external-project-boundary"]
file_ownership:
  primary: ["process/archive/CR-037/**", "process/checks/**"]
  shared: ["process/quant-lab/** read-only references only after explicit migration authorization"]
  forbidden: ["quant-lab release repo writes", "credentials", "runtime", "publish", "live", "production write"]
---

# CR037-S13 quant-lab migration dry-run and reports

## 用户价值

作为迁移执行者，我要用新治理机制迁移 quant-lab，以验证机制能服务真实长期项目。

## 验收标准

- migration 默认 dry-run，只清理过程侧状态和 artifact。
- 对 quant-lab 发布库只读，不自动修改代码、tests、正式 docs 或发布文档。
- 输出 state cleanup、registry migration、impact migration、stale report 和 FU-RF / formal CR candidate。

## 设计证据要求

必须生成 full LLD，明确授权边界、dry-run 输入输出、回滚和人工审批路径。

## LLD 阶段关注风险

| Risk ID | 风险 | 影响 | LLD 必须明确 |
|---------|------|------|--------------|
| R-CR037-S13-LONG-CHAIN | S13 依赖 CR037-S01、S05、S07、S08、S09、S10、S11、S12 共 8 个上游 Story，覆盖 P0 current-state enforcement 与 P1 project governance 主链。 | 任一 P1 Story 延期都会让 quant-lab migration 验证切片成为长尾；若强行推进，可能绕过 registry refs、impact migration、roadmap refresh、FU-RF 或 stale-check 前置能力。 | CP5 前的 full LLD 必须写明：上游依赖未完成时 S13 保持 blocked、拆分为只读 inventory/report 子切片，或缩窄为不触碰缺失能力的 dry-run；不得在依赖缺失时执行 process-side apply。 |

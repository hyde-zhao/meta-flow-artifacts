---
story_id: "ST-WT-006"
cr_id: "CR-047"
stage: "CP6"
status: "implemented"
execution_mode: "inline-fallback"
implemented_at: "2026-07-15"
---

# ST-WT-006 Implementation

## 实现对象

- `README.md`、`delivery/README.md`、`delivery/doc/USER-MANUAL.md`：新增 Codex/Claude/Qoder 三平台非交互 project/full dry-run，全部显式 `--project-dir .`。
- `docs/USER-MANUAL.md`：同步公开派生手册。
- `tests/test_cr047_operator_status.py`：解析文档命令、执行 3/3 dry-run、断言临时目标零写入，并覆盖缺 project-dir 的非交互失败。

## 契约映射

示例直接调用既有 `meta-flow install`，没有新增 orchestrator。五门 preflight 保留逐门退出码、warning 与风险；`OK_WITH_WARNINGS` 不可改写为全绿。

## 验证与边界

三平台 subprocess dry-run 3/3 通过且临时项目写入数=0；缺 `--project-dir` 返回非零并提示修复。未真实安装到用户目录，未读取凭据或访问外部平台。

## 设计差异与交接

CLI/installer 功能契约无需修改；仅修正文档第一入口并增加防漂移测试。

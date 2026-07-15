---
story_id: "ST-WT-005"
cr_id: "CR-047"
stage: "CP6"
status: "implemented"
execution_mode: "inline-fallback"
implemented_at: "2026-07-15"
---

# ST-WT-005 Implementation

## 实现对象

- 对源码、脚本与测试执行 Ruff safe fixes：import 排序、Python 3.11 modernize、无用导入清理。
- 人工修复剩余 B/F 项：异常链、未使用局部变量/循环变量、subprocess 捕获方式。
- README/USER-MANUAL 固化 pytest、Ruff、guardrail、Doctor、CR tracking 五门 preflight。

## 契约映射

Ruff 是发布前独立门，与 pytest 互不替代；任何非零 lint 或测试退出码阻断 CP7。机械 modernize 不改变公开 CLI/schema/状态契约。

## 验证与边界

`ruff check .` 已为零错误；全量回归第一轮为 395 passed + 70 subtests。所有修复均留在当前工作树，未 commit/push，未执行 runtime/credentials/SaaS 操作。

## 设计差异与交接

无设计差异。CP7 在最终过程证据生成后必须再次运行 Ruff 与全量 pytest，不能只引用本轮中间结果。

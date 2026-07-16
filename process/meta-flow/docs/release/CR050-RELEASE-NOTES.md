---
cr_id: "CR-050"
release_decision: "READY_WITH_RISK"
release_artifact_profile: "full"
---

# CR-050 Release Notes

CR-050 为 Meta Flow 增加显式、可审计的双仓 CR Git 分支生命周期：

1. `meta-flow cr branch-open`：从两仓 fresh remote default exact OID 创建同名 CR branch 并建立 upstream；不自动提交。
2. operator 显式提交 bootstrap/业务改动。
3. `meta-flow cr branch-publish`：只推送调用前已存在的 clean committed OID，并生成 matching evidence。
4. `meta-flow cr branch-merge`：以独立 typed authorization，按 `artifact→project` 对 remote default 做 ordinary exact-OID fast-forward；2/2 前不推进 paired truth。
5. `meta-flow cr branch-finish`：fresh reproof 后先创建 local recovery refs，再按 `artifact→project` 删除 remote CR refs，最后删除 local branches。

兼容性：现有 `cr bootstrap`、workspace sync/push 默认行为不变；新命令为 additive。实现复用原生 Git subprocess，不安装 `gb`/Git Town，不调用 forge API。建议在下一次正式包发布采用 MINOR 版本升级；本 CR 不自行修改 `0.4.0` 版本号或创建 tag/release。

质量基线：四个 Story CP6 PASS、CP7 `PASS_WITH_RISK`；447 tests + 70 subtests；Ruff 0；Doctor/guardrail/State/route/event checks 通过；三平台安装 dry-run 3/3。两仓普通 main 留存已完成，但真实 branch lifecycle、branch protection、平台 receipt 与独立 QA 未验证，因此不得声称 production-ready 或 platform-attested。

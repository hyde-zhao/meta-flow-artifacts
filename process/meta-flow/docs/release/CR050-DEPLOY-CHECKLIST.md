# CR-050 Deploy Checklist

- [x] 四个 Story 的 CP6/CP7 R2 result、return、evidence index 与 checkpoint event 可校验。
- [x] TC-GB-001..017 全部回链；13/13 lifecycle fixture 通过。
- [x] 全量 pytest 447 + 70；Ruff 0；`git diff --check` 0。
- [x] Doctor、delivery guardrail、workspace route、State v2 enforce、CR tracking 通过。
- [x] checkpoint/gate ledger check 通过；R2 event ID 唯一。
- [x] Codex/Claude/Qoder project/full dry-run 3/3；真实安装写入=0。
- [x] remote mutation 命令无 force、force-with-lease、local merge、rebase、force-delete。
- [x] release notes、rollback、migration、feedback 与 release context 已生成。
- [x] CP8 用户已接受 `READY_WITH_RISK` 与 DQ-01..04 推荐方案，并为 DQ-05 选择配对 Git 留存备选。
- [x] 当前两仓 `main` 已完成普通 commit/push 和远端 OID 核验；配对结果为 PASS。
- [x] 源码提交：`1373df565417092304ae7b523f529ae878f6931d`；artifact 交付提交：`95c7b7eff7c944a8a905637b4a49f57f21f900f5`。
- [x] 双仓留存证据：`process/checks/CR050-PAIRED-PUBLICATION-EVIDENCE.json`；该证据不代表 branch lifecycle real pilot。
- [x] CR branch lifecycle 真实试运行、force/history rewrite、tag/release、凭据/forge/runtime、`process/quant-lab/**` 与 prelink backup 仍未授权。

真实使用每次都必须提供与 operation/repository/ref/OID 匹配的 fresh authorization；任何 branch protection 或非 FF 拒绝均保持 BLOCKED/PARTIAL，不得绕过。

# CR-047 Deploy Checklist

- [x] process symlink health=ok，元数据重复 link 字节稳定。
- [x] State v2 enforce、CURRENT projection、CR tracking 通过。
- [x] 7/7 CP6 与 7/7 CP7 result/evidence/ledger 可校验。
- [x] pytest 401 + 70、Ruff 0、guardrail/Doctor/CR tracking exit 0。
- [x] Codex/Claude/Qoder dry-run 3/3，真实目标写入=0。
- [x] CR-046 protected object CP7 recheck findings=0。
- [x] Post-approval publication preflight：精确事件身份 selector 回归通过，57 objects / 0 findings。
- [x] 用户批准 CP8 `READY_WITH_RISK`。
- [x] 用户另行授权 `meta-flow` 与 `meta-flow-artifacts` 配对 commit/push；远端事实在执行后复核。
- [x] GitHub `main` 已验证：`meta-flow=3e303215692840ec8ac1920a1d3c21c14ea2907b`；artifact delivery=`7c383661abad6aff8b2e94e91f2910ec42e25a11`。
- [ ] 真实安装、runtime、tag/release 或外部发布仍需独立授权。

本清单不授权 credentials、runtime/SaaS、production write/publish/trading、quant-lab 写入或 prelink backup 操作。

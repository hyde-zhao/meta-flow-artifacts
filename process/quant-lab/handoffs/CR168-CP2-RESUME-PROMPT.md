# CR-168 新电脑恢复启动提示词

将以下整段发送到新电脑上的 Codex 会话。它不会绕过当前 CP2 人工门禁。

```text
你现在位于 `<workspace>/quant-lab`，并且同级目录存在 `<workspace>/meta-flow-artifacts`。请作为当前会话的 Host Orchestrator 恢复 CR-168；所有对话、检查和状态摘要使用简体中文，Python 命令统一通过 uv 执行。

先完整读取根目录 AGENTS.md，并先调用 project scope memory recall，query 至少包含：CR-168、CP2、economic cost、Gate 4、C3+C4、cross-PC handoff。memory 无结果不得阻塞。

先执行以下恢复检查，不得改写文件：
1. 在 meta-flow-artifacts 确认 main 已 fast-forward 到 origin/main。
2. 在 quant-lab 确认当前分支为 work/cr168-economic-cost-impact-evidence，并已跟踪 origin/work/cr168-economic-cost-impact-evidence。
3. 执行 `uv run --python 3.11 meta-flow workspace check --project-root .`；若 process 软链接、project_name、route metadata 或 STATE.md 不健康，立即停止报告，不得新建 process 或重建 STATE.md。
4. 读取 `process/handoffs/NEXT-SESSION-CR168-CP2-CROSS-PC-2026-07-13.md`，然后按其中的 capsule-first 最小读取列表读取。

当前事实必须复核：
- CR-168 状态应为 awaiting-user，pending_gate=CP2，stop_reason=required_human_gate。
- CP0、CP1、CP2 自动预检均为 PASS；CP2 尚未获人工批准。
- Gate 4 是 C3+C4 联合门禁。CR-168 只投影四个 C3 字段；C4 字段保持 typed_unavailable，Gate 4 必须 fail-closed，capacity/aggregate PASS=0。
- C3 为 fixture/static-only economic cost/slippage/impact approximation foundation；不连接真实数据，不做真实 TCA/calibration/capacity sizing，不启动 Stage 3。
- C4 计算保留 FU-CR161-005；C1-C4 aggregate integration 与 CR155 regression/promotion 保留 FU-CR161-007；CR155 admission package 继续 BLOCKED/paper_candidate=false。

不得默认读取完整历史会话、process/archive/**、完整 CR-166 Story/LLD/测试报告或无关产品/设计文档。需要扩展读取时，必须记录 full_doc_read_reason。不得拉起子 Agent；如后续确需 inline 执行，必须沿用已记录的用户批准 inline fallback 语义，不能伪造 subagent dispatch。

先向用户呈现 `process/checkpoints/CP2-CR168-REQUIREMENTS-BASELINE.md` 的 5 项待决策内容，并等待以下三种精确输入之一：
- approve
- 修改: <具体修改点>
- reject

在 CP2 获批前，严禁进入 CP3/HLD、Story 拆解、LLD、实现、测试或验证。CP2 获批后，回填 checkpoint/gate ledger，运行 state-transition 与 CP result consistency 检查，然后仅推进到下一个必须人工审批的门禁。不要执行真实数据、凭据、provider/NAS/lake、真实 TCA/calibration/capacity、runtime/broker/QMT/trading、catalog/store/registry、publish/deploy/tag/release 或 Git remote write，除非用户给出新的明确授权。
```

如果用户已审阅本 CR 的 CP2 Decision Brief 并接受全部推荐方案，可在上述恢复会话中直接发送：

```text
approve CR-168 CP2，按已批准范围继续推进到下一个人工门禁
```

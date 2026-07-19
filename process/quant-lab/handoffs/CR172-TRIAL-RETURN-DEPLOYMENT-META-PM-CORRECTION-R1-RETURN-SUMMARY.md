---
handoff_id: "CR172-TRIAL-RETURN-DEPLOYMENT-META-PM-CORRECTION-R1-RETURN"
from_agent: "meta-pm"
to_agent: "host-orchestrator"
change_id: "CR-172"
phase: "requirement-clarification"
checkpoint: "CP2"
status: "completed"
created_at: "2026-07-17T15:20:00+08:00"
context_ref: "process/context/CP2-CR172-TRIAL-RETURN-DEPLOYMENT-CONTEXT.yaml"
correction_ref: "process/handoffs/CR172-TRIAL-RETURN-DEPLOYMENT-META-PM-CORRECTION-R1-2026-07-17.md"
cp1_result_ref: "process/checks/CP1-CR172-TRIAL-RETURN-DEPLOYMENT-USE-CASE-COMPLETENESS.result.json"
cp2_result_ref: "process/checks/CP2-CR172-TRIAL-RETURN-DEPLOYMENT-REQUIREMENTS-BASELINE.result.json"
checkpoint_ref: "process/checkpoints/CP2-CR172-TRIAL-RETURN-DEPLOYMENT-REQUIREMENTS-BASELINE.md"
---

# CR172 trial-return / deployment correction R1 return summary

## 纠正结论

- 部署主权已统一为：研究机本地 active canonical → NAS verified replica/backup/distribution → 执行机按需 pull 到临时目录，校验 release/manifest/hash 后原子物化为本地 immutable cache；运行时直接读取 NAS 为 0。
- REQ/DQ-CR172-015 已纳入 CP2：默认在执行机本地生成信号；低频/EOD 可使用 NAS immutable `SignalBatch` / `TargetPositionBatch` mailbox；intraday realtime transport 必须另立 CR。
- batch 合同固定覆盖 strategy/package hash、sequence、validity、content hash/signature、source refs、idempotency key、ack 共 8/8；credential、account secret、broker order command 三类载荷 0/3 允许。
- 授权动作从 5 类修订为 6 类：lake read、multi-trial runtime、workspace write、trial-return/R generation、R compute、NAS replica sync；本次真实 sync/pull/signal/runtime 等执行计数均为 0。
- CR172 场景为 27/27 P0：保留原 8 个，前轮新增 13 个经纠正，R1 新增 6 个 signal/sync 场景；REQ-009~015 正向与负向语义覆盖均为 7/7。

## 修改范围

1. `docs/product/USE-CASES.md`
2. `docs/product/REQUIREMENTS.md`
3. `docs/product/SCENARIOS.yaml`
4. `docs/product/TEST-MATRIX.md`
5. `docs/product/STORY-MAP.md`
6. `docs/product/MVP-SCOPE.md`
7. `docs/product/RELEASE-SLICES.md`
8. `docs/product/BACKLOG.md`
9. CP1 result 与人类摘要
10. CP2 result 与人类摘要
11. CP2 人工 checkpoint Decision Brief
12. 原 return summary 与本 correction R1 return summary

未修改设计文档、代码、README、USER-MANUAL、状态机、ledger；未执行真实 sync、pull、signal generation/transport 或 runtime。

## 自动检查

| 检查 | 结果 |
|---|---|
| 8 个产品文档结构与修订记录 | PASS |
| `SCENARIOS.yaml` 解析、场景 ID 唯一与 coverage 总数 | PASS |
| REQ/DQ-009~015 与 scenario/matrix 追踪 | PASS |
| CP1/CP2 result JSON 解析与 `meta-flow cp result-check` | PASS |
| 旧部署主权表述残留扫描 | PASS，0 |
| 真实 sync/pull/signal/runtime 操作 | 0 |

## 待人工确认

- DQ-CR172-009~015 共 7 项仍为 scope-delta CP2 `pending`；correction R1 是用户意图纠正，不代替正式 CP2 approval。
- host-orchestrator 可据修订后的 Decision Brief 发起 CP2 人工门禁；批准后由 host 负责 checkpoint 人工结果、事件 ledger、CR status-sync 与状态推进。

## 阻塞

- meta-pm 阻塞：无。

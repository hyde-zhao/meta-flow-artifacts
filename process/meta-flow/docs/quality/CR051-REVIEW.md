---
cr_id: CR-051
stage: CP8-preparation
review_decision: PASS_WITH_RISK
generated_at: "2026-07-18T16:55:34Z"
---

# CR-051 质量评审

## 1. Findings Summary

| 严重度 | Open | Closed | 结论 |
|---|---:|---:|---|
| BLOCKER | 0 | 0 | 无阻断 |
| HIGH | 0 | 2 | 两项均由 R2 关闭 |
| MEDIUM | 5 | 0 | 风险接受、not-authorized 或 follow-up |
| LOW | 2 | 0 | fail-closed / 平台 pilot 后续项 |

## 2. 开放 findings / 风险

### CR051-REV-CP8-001 — 独立 QA 覆盖不完整（MEDIUM）

ST-AW-001 R2 与 ST-AW-002 在 QA timeout 后由 Host 执行 bounded fallback。证据可复跑且不构成缺失证据，但交付声明不得写成“五个 Story 全部独立 QA”。推荐 CP8 接受该上限；若发布场景要求全量独立性，则另行复验这两个 Story。

### CR051-REV-CP8-002 — capability registry 缺失（MEDIUM）

`docs/design/CAPABILITY-STATUS.yaml` 不存在，导致 capability-claims checker 无法验证 README 与 USER-MANUAL。两份文档已经明确使用 offline fixture / not-authorized 标签，未发现 false runtime-ready 声明，因此不阻断 CP8，但恢复 registry 后必须重跑检查。

### CR051-REV-CP8-003 — 真实托管环境未验证（MEDIUM）

remote、凭据、branch protection、真实 worktree/ref 与 publication 未授权。推荐维持不授权状态；首次真实接入前按项目执行独立授权与 pilot。

### CR051-REV-CP8-004 — 真实迁移与并发目录威胁未验证（MEDIUM）

当前只交付只读 migration preflight，没有 CLI 或迁移执行。真实 cutover 及不可信并发目录 rename/replace 不在本轮验证范围；未来迁移 CR 需 full-LLD、dirfd/openat 或等价防护。

### CR051-REV-CP8-005 — canonical module boundary manifest 缺失（MEDIUM）

当前依赖边界由代码、测试与 forbidden-import 审计证明，缺少 canonical manifest。推荐作为 follow-up，不把局部静态证据包装为完整架构边界注册。

### CR051-REV-CP8-006 — Windows native pilots 未执行（LOW）

Git argv/path、durability、cross-process lock 与 no-follow 仅在 Linux/fixture 或 fail-closed 语义下验证。首次 Windows 支持前应执行原生 pilot。

### CR051-REV-CP8-007 — selector lock crash 残留（LOW）

writer crash 可能遗留 lock directory；后续 writer 会 fail closed。需要人工诊断或未来受控恢复机制，不应自动删除无法证明 stale 的锁。

## 3. 已关闭 HIGH findings

| Finding | 关闭方式 | 独立性 |
|---|---|---|
| REV-AW003-001 | 修复 default-clock 基准并验证合法/过期/未来路径 | actual independent QA R2 |
| REV-AW005-001 | 每对象 deny guard 与 relative symlink denied-target 归一化；3 个独立 probe | actual QA 首轮发现；Host 有界落盘首轮证据；同 QA lane 独立验证 R2 |

## 4. 架构一致性结论

实现与文档均采用三层 artifact 拓扑：shared `main` → per-project integration → per-CR short branch。CR 分支只从最新项目 integration 派生并回合到该 integration；shared main 的双向同步在 CR 外由人工完成。源码仓库仍采用 default branch → CR branch → default branch。两种仓库生命周期没有被错误合并。

## 5. 授权边界

本评审仅判定离线实现与证据可进入 CP8。它不批准真实 Git/worktree/ref/remote、凭据、网络、main↔integration 同步、迁移、软链接、commit、push、publish 或 release。CP8 `approve` 也不应扩大为上述授权。

## 6. 评审结论

`PASS_WITH_RISK`。无阻断 finding，建议进入 CP8，并由人工明确接受风险、保持真实运行不授权、将平台/manifest/TOCTOU 项留作 follow-up tracking candidate。

## 7. CP8 R2 Finding Closure（2026-07-19）

| Finding | 结果 |
|---|---|
| ST-AW-002 保守偏离未记录 | CLOSED：design delta、Feature 1.2、LLD 1.2.0 |
| ST-AW-002 owner/calibration/proof 持久化 | CLOSED：实现与故障测试 |
| ST-AW-003 四个高优测试与 cleanup proof | CLOSED：TP-AW03-004/006/008/014 + containment/exact CAS |
| ST-AW-004 target policy/PARTIAL/dependency-DAG | CLOSED：consumer validation、staged projector、retry 与 spy/order tests |

新增实现 finding 全部关闭；剩余为 runtime authorization、平台、historical ledger/format baseline 和 Host inline-fallback 独立性风险。最新详细评审见 `process/docs/quality/CR051-CP8-REWORK-REVIEW.md`。
